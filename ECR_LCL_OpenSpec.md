# 📋 ECR-LCL PLATFORM — OPEN SPECIFICATION
> **Tài liệu Đặc tả Kỹ thuật Mở cho AI Thực thi**
> Dự án: *Empty Container Repositioning × Less-than-Container-Load*
> Căn cứ: Final System Spec v3.0 | Phiên bản OpenSpec: `v1.0`

---

## MỤC LỤC

| # | Feature | Module | Priority |
|---|---|---|---|
| [F-01](#f-01) | Container Registration & Validation | Supply Management | 🔴 Critical |
| [F-02](#f-02) | LCL Booking Request & HS Code Checker | Demand Management | 🔴 Critical |
| [F-03](#f-03) | 3D Bin Packing Engine (FFD — Layer 1) | Matching Engine | 🔴 Critical |
| [F-04](#f-04) | PSO Routing Optimizer (ECHDL-PSO — Layer 2) | Matching Engine | 🔴 Critical |
| [F-05](#f-05) | NSGA-II Multi-objective + Dynamic Pricing (Layer 3) | Matching Engine | 🔴 Critical |
| [F-06](#f-06) | Time-lock Dual Confirmation (Redis TTL) | Transaction | 🔴 Critical |
| [F-07](#f-07) | Dynamic Pricing Engine | Pricing | 🟠 High |
| [F-08](#f-08) | Cabotage Compliance Validator | Legal | 🔴 Critical |

---

<a id="f-01"></a>
## F-01 — Container Registration & Validation

### 1. Feature Overview

Cho phép **Hãng tàu (Shipping Line / MLO)** đăng ký container rỗng vào hệ thống. Đây là bước khởi tạo nguồn cung (Supply) cho toàn bộ Matching Engine. Hệ thống phải validate mã container theo chuẩn **ISO 6346**, kiểm tra logic thời gian ETD/T_cutoff, và broadcast notification đến NVOCC đang theo dõi tuyến.

---

### 2. Data Schema

#### 2.1 Input Schema

| Field | Type | Required | Constraints | Example |
|---|---|---|---|---|
| `container_id` | `VARCHAR(11)` | ✅ | Format: `[A-Z]{4}[0-9]{6}[0-9]{1}` · ISO 6346 check digit | `TCKU1234565` |
| `type` | `ENUM` | ✅ | `20DC` \| `40DC` \| `40HC` \| `20RF` \| `40RF` | `20DC` |
| `shipping_line_id` | `UUID` | ✅ | FK → Organization · Must exist in DB | `uuid-v4` |
| `depot_location` | `POINT(lat, lng)` | ✅ | lat ∈ [−90, 90] · lng ∈ [−180, 180] | `(10.7769, 106.7009)` |
| `destination_port` | `VARCHAR(5)` | ✅ | Must be `VNHPH` (Hải Phòng) for this route | `VNHPH` |
| `etd` | `TIMESTAMP` | ✅ | Must be ≥ NOW() + 24h | `2026-04-01T08:00:00Z` |
| `closing_time` | `TIMESTAMP` | ✅ | Must satisfy: `ETD − 24h ≤ closing_time ≤ ETD − 8h` | `2026-03-31T20:00:00Z` |
| `floor_price` | `DECIMAL(12,0)` | ✅ | > 0 · Unit: VNĐ/TEU | `3500000` |
| `quantity` | `INTEGER` | ✅ | ≥ 1 · ≤ 500 | `10` |

#### 2.2 Output Schema

| Field | Type | Description |
|---|---|---|
| `container_id` | `VARCHAR(11)` | Mã container đã validate |
| `status` | `ENUM` | Luôn = `AVAILABLE` khi tạo thành công |
| `days_until_etd` | `FLOAT` | `(ETD − NOW()) / 86400` |
| `demurrage_risk` | `BOOLEAN` | `TRUE` nếu `days_until_etd > 3` (Free Time thường = 3 ngày) |
| `demurrage_warning` | `STRING \| NULL` | Bảng phí lũy tiến nếu `demurrage_risk = TRUE` |
| `created_at` | `TIMESTAMP` | Server timestamp lúc tạo |
| `notification_sent` | `BOOLEAN` | `TRUE` nếu broadcast đến NVOCC thành công |

#### 2.3 ISO 6346 Check Digit Algorithm

```python
def calculate_iso6346_check_digit(container_code: str) -> int:
    """
    Tính check digit theo ISO 6346.
    Input: 10 ký tự đầu (4 chữ + 6 số), e.g. "TCKU123456"
    Output: check digit (0-9, hoặc 10 → biểu diễn là 0)
    
    Bảng giá trị chữ cái:
    A=10, B=12, C=13, D=14, E=15, F=16, G=17, H=18, I=19, J=20,
    K=21, L=23, M=24, N=25, O=26, P=27, Q=28, R=29, S=30, T=31,
    U=32, V=34, W=35, X=36, Y=37, Z=38
    """
    LETTER_VALUES = {
        'A':10,'B':12,'C':13,'D':14,'E':15,'F':16,'G':17,'H':18,
        'I':19,'J':20,'K':21,'L':23,'M':24,'N':25,'O':26,'P':27,
        'Q':28,'R':29,'S':30,'T':31,'U':32,'V':34,'W':35,'X':36,
        'Y':37,'Z':38
    }
    total = 0
    for i, char in enumerate(container_code[:10]):
        value = LETTER_VALUES[char] if char.isalpha() else int(char)
        total += value * (2 ** i)
    remainder = total % 11
    return 0 if remainder == 10 else remainder
```

---

### 3. Core Logic

```
STEP 1 — Validate Container ID Format
  regex_match = re.fullmatch(r'[A-Z]{4}[0-9]{6}[0-9]', container_id)
  IF NOT regex_match → raise ContainerFormatError(container_id)

STEP 2 — Validate ISO 6346 Check Digit
  expected_check = calculate_iso6346_check_digit(container_id[:10])
  actual_check   = int(container_id[10])
  IF expected_check != actual_check → raise ISO6346CheckDigitError(container_id)

STEP 3 — Validate Time Constraints
  now = datetime.utcnow()
  IF etd < now + timedelta(hours=24)    → raise ETDTooSoonError(etd)
  IF closing_time > etd - timedelta(hours=8)  → raise CutoffTooLateError()
  IF closing_time < etd - timedelta(hours=24) → raise CutoffTooEarlyError()

STEP 4 — Validate Destination Port
  IF destination_port != "VNHPH" → raise InvalidDestinationError(destination_port)
  # Route hiện tại chỉ hỗ trợ Cát Lái → Hải Phòng

STEP 5 — Demurrage Risk Check
  days_until_etd = (etd - now).total_seconds() / 86400
  demurrage_risk = days_until_etd > 3
  IF demurrage_risk:
      warning = build_demurrage_table(days_until_etd)  # Bảng phí lũy tiến

STEP 6 — Persist to Database
  container = Container(
      container_id=container_id, type=type,
      shipping_line_id=shipping_line_id,
      depot_location=depot_location,
      destination_port=destination_port,
      etd=etd, closing_time=closing_time,
      floor_price=floor_price, status="AVAILABLE"
  )
  db.session.add(container); db.session.commit()

STEP 7 — Broadcast Notification
  event = ContainerAvailableEvent(container_id, etd, type, floor_price)
  redis.publish("route:VNCLI-VNHPH:new_supply", event.to_json())
  → All subscribed NVOCCs receive push notification
```

---

### 4. Test Scenarios

#### 4.1 Happy Path

```python
# tests/unit/supply/test_container_registration.py

class TestContainerRegistrationHappyPath:

    def test_valid_20dc_container_registers_successfully(self):
        """Container 20DC hợp lệ, ETD đủ xa, T_cutoff đúng chuẩn."""
        payload = {
            "container_id": "TCKU1234565",   # Check digit hợp lệ
            "type": "20DC",
            "shipping_line_id": "550e8400-e29b-41d4-a716-446655440000",
            "depot_location": {"lat": 10.7769, "lng": 106.7009},
            "destination_port": "VNHPH",
            "etd": (now() + timedelta(days=5)).isoformat(),
            "closing_time": (now() + timedelta(days=4, hours=14)).isoformat(),
            "floor_price": 3_500_000,
            "quantity": 5,
        }
        response = client.post("/api/supply/containers", json=payload)
        assert response.status_code == 201
        assert response.json()["status"] == "AVAILABLE"
        assert response.json()["notification_sent"] is True

    def test_container_with_demurrage_risk_returns_warning(self):
        """Container ETD > 3 ngày → cảnh báo phí lưu bãi."""
        payload = {**valid_payload, "etd": (now() + timedelta(days=10)).isoformat()}
        response = client.post("/api/supply/containers", json=payload)
        assert response.status_code == 201
        assert response.json()["demurrage_risk"] is True
        assert response.json()["demurrage_warning"] is not None
```

#### 4.2 Edge Cases

```python
class TestContainerRegistrationEdgeCases:

    def test_etd_exactly_24h_from_now_is_accepted(self):
        """ETD = NOW + 24h là giá trị biên hợp lệ."""
        etd = now() + timedelta(hours=24, seconds=1)  # +1s buffer
        payload = {**valid_payload, "etd": etd.isoformat()}
        response = client.post("/api/supply/containers", json=payload)
        assert response.status_code == 201

    def test_etd_23h59m_from_now_is_rejected(self):
        """ETD = NOW + 23h59m phải bị từ chối."""
        etd = now() + timedelta(hours=23, minutes=59)
        payload = {**valid_payload, "etd": etd.isoformat()}
        response = client.post("/api/supply/containers", json=payload)
        assert response.status_code == 422
        assert "ETD_TOO_SOON" in response.json()["error_code"]

    def test_closing_time_exactly_8h_before_etd_is_accepted(self):
        """T_cutoff = ETD − 8h là giá trị biên hợp lệ."""
        etd = now() + timedelta(days=5)
        closing = etd - timedelta(hours=8)
        payload = {**valid_payload, "etd": etd.isoformat(), "closing_time": closing.isoformat()}
        response = client.post("/api/supply/containers", json=payload)
        assert response.status_code == 201

    def test_duplicate_container_id_returns_409(self):
        """Container ID đã tồn tại trong DB → 409 Conflict."""
        client.post("/api/supply/containers", json=valid_payload)  # First insert
        response = client.post("/api/supply/containers", json=valid_payload)  # Duplicate
        assert response.status_code == 409
        assert "DUPLICATE_CONTAINER_ID" in response.json()["error_code"]

    def test_quantity_zero_is_rejected(self):
        """Số lượng = 0 không hợp lệ."""
        payload = {**valid_payload, "quantity": 0}
        response = client.post("/api/supply/containers", json=payload)
        assert response.status_code == 422
```

#### 4.3 Error Handling

```python
class TestContainerRegistrationErrors:

    @pytest.mark.parametrize("bad_id, expected_error", [
        ("TCKU123456X",  "INVALID_FORMAT"),        # Ký tự cuối không phải số
        ("TCK1234567",   "INVALID_FORMAT"),        # Chỉ 3 chữ cái thay vì 4
        ("TCKU1234560",  "INVALID_CHECK_DIGIT"),   # Check digit sai (đúng phải là 5)
        ("",             "INVALID_FORMAT"),        # Chuỗi rỗng
        ("TCKU12345678", "INVALID_FORMAT"),        # Quá dài (12 ký tự)
    ])
    def test_invalid_container_id_formats(self, bad_id, expected_error):
        payload = {**valid_payload, "container_id": bad_id}
        response = client.post("/api/supply/containers", json=payload)
        assert response.status_code == 422
        assert expected_error in response.json()["error_code"]

    def test_non_vnhph_destination_is_blocked(self):
        """Chỉ chấp nhận destination_port = VNHPH (tuyến Cát Lái → Hải Phòng)."""
        payload = {**valid_payload, "destination_port": "VNSSG"}
        response = client.post("/api/supply/containers", json=payload)
        assert response.status_code == 422
        assert "INVALID_DESTINATION" in response.json()["error_code"]

    def test_floor_price_zero_is_rejected(self):
        payload = {**valid_payload, "floor_price": 0}
        response = client.post("/api/supply/containers", json=payload)
        assert response.status_code == 422
```

---

### 5. Verification

```bash
# Chạy toàn bộ test suite cho F-01
pytest tests/unit/supply/test_container_registration.py -v

# Kiểm tra coverage
pytest tests/unit/supply/ --cov=src/supply --cov-report=term-missing --cov-fail-under=90

# Chạy integration test (yêu cầu Docker Compose)
docker-compose -f docker-compose.test.yml up -d
pytest tests/integration/test_supply_api.py -v

# Kiểm tra ISO 6346 check digit riêng
pytest tests/unit/supply/test_iso6346.py -v -k "check_digit"
```

---

---

<a id="f-02"></a>
## F-02 — LCL Booking Request & HS Code Checker

### 1. Feature Overview

Cho phép **NVOCC / Consolidator** tạo yêu cầu gom hàng lẻ LCL. Hệ thống tự động tính CBM tổng, kiểm tra tương thích hóa chất theo **Chemical Compatibility Matrix** và quy định IMDG, đặc biệt áp dụng lệnh **cấm tuyệt đối hàng IMDG tại Cát Lái từ 01/07/2024**. Booking Request được đưa vào Celery Queue cho Matching Engine.

---

### 2. Data Schema

#### 2.1 Input — CargoItem (Per Item)

| Field | Type | Required | Constraints |
|---|---|---|---|
| `length` | `DECIMAL(8,2)` | ✅ | > 0 · Unit: cm · ≤ 1200 (container max) |
| `width` | `DECIMAL(8,2)` | ✅ | > 0 · Unit: cm · ≤ 234 (container inner width) |
| `height` | `DECIMAL(8,2)` | ✅ | > 0 · Unit: cm · ≤ 238 (container inner height) |
| `weight` | `DECIMAL(10,2)` | ✅ | > 0 · Unit: kg · Per item |
| `hs_code` | `VARCHAR(10)` | ✅ | 8 digits HS Code (Vietnam Tariff Schedule) |
| `qty` | `INTEGER` | ✅ | ≥ 1 |
| `fragile` | `BOOLEAN` | ✅ | `TRUE` → không được chất hàng lên trên |
| `stackable` | `BOOLEAN` | ✅ | `FALSE` nếu `fragile = TRUE` |

#### 2.2 Input — BookingRequest

| Field | Type | Required | Constraints |
|---|---|---|---|
| `nvocc_id` | `UUID` | ✅ | FK → Organization |
| `cargo_items` | `JSONB[]` | ✅ | ≥ 1 item · ≤ 500 items |
| `cfs_origin` | `POINT(lat, lng)` | ✅ | GPS của kho CFS xuất phát |
| `deadline` | `TIMESTAMP` | ✅ | > NOW() + 48h |
| `max_budget_vnd` | `DECIMAL(12,0)` | ✅ | > 0 |
| `origin_port` | `VARCHAR(5)` | ✅ | Must be `VNCLI` (Cát Lái) |

#### 2.3 Output Schema

| Field | Type | Description |
|---|---|---|
| `booking_id` | `UUID` | PK tự sinh |
| `total_cbm` | `DECIMAL(8,3)` | `Σ(L×W×H/1_000_000 × qty)` |
| `total_weight_kg` | `DECIMAL(10,2)` | `Σ(weight × qty)` |
| `suggested_container_type` | `ENUM` | `20DC` nếu CBM ≤ 22 · `40DC/HC` nếu CBM 22–60 |
| `fill_rate_preview` | `DECIMAL(5,2)` | `total_cbm / container_capacity × 100` |
| `has_imdg` | `BOOLEAN` | TRUE nếu bất kỳ item nào thuộc IMDG Class 1–9 |
| `chemical_warnings` | `STRING[]` | Danh sách cảnh báo tương thích hóa chất |
| `is_blocked` | `BOOLEAN` | TRUE nếu IMDG tại Cát Lái (block hoàn toàn) |
| `status` | `ENUM` | `PENDING` \| `BLOCKED` |
| `queue_position` | `INTEGER` | Vị trí trong Celery Matching Queue |

#### 2.4 Chemical Compatibility Matrix

```python
# Nhóm HS Code → IMDG Class / Chemical Group
HS_GROUP_MAP = {
    "GROUP_FOOD":      range_union(["01-24"]),   # Thực phẩm
    "GROUP_ACID":      ["28XX"],                 # Axit vô cơ
    "GROUP_BASE":      ["28151100", "28152000"], # Kiềm (NaOH, KOH)
    "GROUP_OXIDIZER":  ["28XX", "29XX"],         # Chất oxy hóa
    "GROUP_FLAMMABLE": ["27XX", "29XX"],         # Chất dễ cháy
}

INCOMPATIBLE_PAIRS = [
    ("GROUP_IMDG",    "GROUP_FOOD"),     # IMDG không đóng chung thực phẩm
    ("GROUP_ACID",    "GROUP_BASE"),     # Phản ứng tỏa nhiệt
    ("GROUP_OXIDIZER","GROUP_FLAMMABLE"), # Nguy cơ cháy nổ
]
```

---

### 3. Core Logic

```
STEP 1 — Validate Origin Port
  IF origin_port != "VNCLI":
      raise InvalidOriginPortError("Chỉ hỗ trợ tuyến từ Cát Lái")

STEP 2 — Validate Each CargoItem
  FOR each item in cargo_items:
      IF item.fragile AND item.stackable:
          raise ConflictingFlagsError(item_id, "fragile=TRUE requires stackable=FALSE")
      IF item.length > 1200 OR item.width > 234 OR item.height > 238:
          raise ItemExceedsContainerDimensionError(item_id)

STEP 3 — Calculate CBM & Weight
  total_cbm = Σ(item.length × item.width × item.height / 1_000_000 × item.qty)
  total_weight = Σ(item.weight × item.qty)

STEP 4 — HS Code Chemical Check (Rule Engine)
  FOR each item in cargo_items:
      imdg_class = lookup_imdg_class(item.hs_code)  # DB lookup
      
      # RULE A: IMDG + Food → reject
      IF imdg_class IS NOT NULL AND any_item_is_food(cargo_items):
          add_warning("IMDG_WITH_FOOD", item.hs_code)
          set has_imdg = TRUE

      # RULE B: Cát Lái IMDG ban (from 2024-07-01)
      IF imdg_class IS NOT NULL AND origin_port == "VNCLI" AND TODAY >= 2024-07-01:
          set is_blocked = TRUE
          raise IMDGBlockedAtCatLaiError(item.hs_code, imdg_class)

      # RULE C: Cross-compatibility Matrix
      FOR each pair in INCOMPATIBLE_PAIRS:
          IF has_group(cargo_items, pair[0]) AND has_group(cargo_items, pair[1]):
              add_warning("CHEMICAL_INCOMPATIBLE", pair)
              set is_blocked = TRUE

STEP 5 — Container Suggestion
  IF total_cbm <= 22:   suggested = "20DC"
  ELIF total_cbm <= 60: suggested = "40DC" or "40HC"
  ELSE:                 raise CBMExceedsSingleContainerError(total_cbm)
  fill_rate_preview = total_cbm / CONTAINER_CAPACITY[suggested] * 100

STEP 6 — Persist & Enqueue
  IF is_blocked: status = "BLOCKED"; return early (do not enqueue)
  booking = BookingRequest(..., status="PENDING")
  db.save(booking)
  celery.send_task("matching.run_batch", kwargs={"trigger": "new_booking"})
  queue_position = get_queue_position(booking.booking_id)
```

---

### 4. Test Scenarios

#### 4.1 Happy Path

```python
class TestBookingRequestHappyPath:

    def test_valid_dry_cargo_booking_created_successfully(self):
        """Hàng khô thông thường, không IMDG, CBM phù hợp 20ft."""
        payload = {
            "nvocc_id": VALID_NVOCC_UUID,
            "origin_port": "VNCLI",
            "cargo_items": [
                {"length": 100, "width": 80, "height": 60, "weight": 200,
                 "hs_code": "61091000", "qty": 10, "fragile": False, "stackable": True},
            ],
            "cfs_origin": {"lat": 10.9447, "lng": 106.8243},
            "deadline": (now() + timedelta(days=5)).isoformat(),
            "max_budget_vnd": 5_000_000,
        }
        response = client.post("/api/demand/bookings", json=payload)
        assert response.status_code == 201
        data = response.json()
        assert data["status"] == "PENDING"
        assert data["has_imdg"] is False
        assert data["is_blocked"] is False
        assert data["total_cbm"] == pytest.approx(0.048 * 10, rel=1e-3)
        assert data["suggested_container_type"] == "20DC"

    def test_cbm_auto_calculation_accuracy(self):
        """CBM tính tự động phải chính xác đến 3 chữ số thập phân."""
        items = [{"length": 120, "width": 100, "height": 80, "weight": 300,
                  "hs_code": "84715000", "qty": 5, "fragile": False, "stackable": True}]
        # Expected: 120×100×80/1_000_000 × 5 = 0.096 × 5 = 0.480 m³
        response = client.post("/api/demand/bookings", json={**valid_booking, "cargo_items": items})
        assert response.json()["total_cbm"] == pytest.approx(0.480, abs=0.001)
```

#### 4.2 Edge Cases

```python
class TestBookingRequestEdgeCases:

    def test_single_item_exactly_at_20ft_cbm_limit(self):
        """CBM = 22.0 m³ → gợi ý 20DC (giá trị biên trên)."""
        # 220cm × 200cm × 50cm / 1_000_000 × 1 = 2.2m³ × 10 items = 22m³
        items = [{"length": 220, "width": 200, "height": 50, "weight": 100,
                  "hs_code": "61091000", "qty": 10, "fragile": False, "stackable": True}]
        response = client.post("/api/demand/bookings", json={**valid_booking, "cargo_items": items})
        assert response.json()["suggested_container_type"] == "20DC"

    def test_cbm_22_001_suggests_40ft(self):
        """CBM = 22.001 m³ → gợi ý 40DC (vượt biên 20ft)."""
        items = make_items_with_cbm(22.001)
        response = client.post("/api/demand/bookings", json={**valid_booking, "cargo_items": items})
        assert response.json()["suggested_container_type"] in ["40DC", "40HC"]

    def test_empty_cargo_items_is_rejected(self):
        """Danh sách hàng rỗng phải bị từ chối."""
        payload = {**valid_booking, "cargo_items": []}
        response = client.post("/api/demand/bookings", json=payload)
        assert response.status_code == 422
        assert "EMPTY_CARGO_LIST" in response.json()["error_code"]

    def test_500_items_at_max_capacity(self):
        """500 items nhỏ vẫn phải xử lý được (performance test)."""
        items = [small_item() for _ in range(500)]
        response = client.post("/api/demand/bookings", json={**valid_booking, "cargo_items": items})
        assert response.status_code in [201, 422]  # 422 nếu tổng CBM vượt container

    def test_fragile_item_with_stackable_true_raises_error(self):
        """fragile=TRUE và stackable=TRUE là trạng thái mâu thuẫn."""
        items = [{"fragile": True, "stackable": True, **other_fields}]
        response = client.post("/api/demand/bookings", json={**valid_booking, "cargo_items": items})
        assert response.status_code == 422
        assert "CONFLICTING_FLAGS" in response.json()["error_code"]
```

#### 4.3 Error Handling

```python
class TestBookingRequestErrors:

    def test_imdg_cargo_at_cat_lai_is_blocked(self):
        """IMDG Class 8 (NaOH) tại cảng Cát Lái → BLOCKED hoàn toàn."""
        items = [{"hs_code": "28151100", "qty": 1, **dim_fields}]  # NaOH
        response = client.post("/api/demand/bookings",
                               json={**valid_booking, "cargo_items": items, "origin_port": "VNCLI"})
        assert response.status_code == 422
        data = response.json()
        assert data["is_blocked"] is True
        assert "IMDG_BLOCKED_CAT_LAI" in data["error_code"]

    def test_imdg_mixed_with_food_triggers_chemical_warning(self):
        """IMDG hàng nguy hiểm + HS thực phẩm (HS 09) → cảnh báo hóa chất."""
        items = [
            {"hs_code": "28151100", "qty": 1, **dim_fields},  # NaOH
            {"hs_code": "09011100", "qty": 2, **dim_fields},  # Cà phê
        ]
        response = client.post("/api/demand/bookings", json={**valid_booking, "cargo_items": items})
        assert response.status_code == 422
        assert "IMDG_WITH_FOOD" in response.json()["chemical_warnings"]

    def test_acid_plus_base_triggers_incompatibility(self):
        """Axit (HS 2807) + Kiềm (NaOH HS 28151100) → hóa chất không tương thích."""
        items = [
            {"hs_code": "28070010", "qty": 1, **dim_fields},  # H2SO4
            {"hs_code": "28151100", "qty": 1, **dim_fields},  # NaOH
        ]
        response = client.post("/api/demand/bookings", json={**valid_booking, "cargo_items": items})
        assert response.status_code == 422
        assert "CHEMICAL_INCOMPATIBLE" in response.json()["chemical_warnings"]

    def test_cbm_exceeds_single_40hc_container_returns_error(self):
        """CBM > 67 m³ (40HC max) → không thể đặt một booking, gợi ý tách."""
        items = make_items_with_cbm(70.0)
        response = client.post("/api/demand/bookings", json={**valid_booking, "cargo_items": items})
        assert response.status_code == 422
        assert "CBM_EXCEEDS_SINGLE_CONTAINER" in response.json()["error_code"]
        assert "split_suggestion" in response.json()
```

---

### 5. Verification

```bash
# Unit tests F-02
pytest tests/unit/demand/test_booking_request.py -v
pytest tests/unit/demand/test_chemical_checker.py -v
pytest tests/unit/demand/test_cbm_calculator.py -v

# Coverage check (validators phải ≥ 95%)
pytest tests/unit/demand/ --cov=src/demand \
    --cov=src/validators/chemical_checker \
    --cov-report=term-missing \
    --cov-fail-under=90

# Kiểm tra chemical matrix riêng
pytest tests/unit/demand/ -k "chemical or imdg or hs_code" -v

# Test với mock DB (không cần PostgreSQL)
pytest tests/unit/demand/ --mock-db -v
```

---

---

<a id="f-03"></a>
## F-03 — 3D Bin Packing Engine (FFD — Layer 1)

### 1. Feature Overview

**Lớp 1 của Matching Engine**. Kiểm tra khả năng đóng gói: xếp tất cả kiện hàng từ `BookingRequest` vào `Container` trong không gian 3 chiều theo thuật toán **First Fit Decreasing (FFD)**. Đây là cổng lọc (feasibility gate) — chỉ khi `FEASIBLE` mới kích hoạt L2 và L3. Output JSON layout là nguồn render cho **Three.js 3D Visualizer**.

---

### 2. Data Schema

#### 2.1 Input

| Field | Type | Source | Description |
|---|---|---|---|
| `items` | `CargoItem[]` | BookingRequest | Danh sách kiện hàng với dims + weight + flags |
| `container` | `ContainerSpec` | Container DB | `{L_max, W_max, H_max, max_weight_kg}` |
| `chemical_ok` | `BOOLEAN` | F-02 output | Pre-validated flag từ HS Code Checker |

#### 2.2 ContainerSpec Constants

| Type | Inner L (cm) | Inner W (cm) | Inner H (cm) | Max Weight (kg) | Volume (m³) |
|---|---|---|---|---|---|
| `20DC` | 589 | 234 | 238 | 28,000 | 32.8 |
| `40DC` | 1200 | 234 | 238 | 26,500 | 66.9 |
| `40HC` | 1200 | 234 | 269 | 26,480 | 75.5 |
| `20RF` | 537 | 225 | 218 | 27,400 | 26.4 |
| `40RF` | 1145 | 225 | 218 | 29,000 | 56.2 |

#### 2.3 Output

| Field | Type | Description |
|---|---|---|
| `status` | `ENUM` | `FEASIBLE` \| `INFEASIBLE` |
| `reason_code` | `STRING \| NULL` | `CBM_EXCEEDED` \| `WEIGHT_EXCEEDED` \| `ITEM_DOES_NOT_FIT` \| `CHEMICAL_INCOMPATIBLE` \| `STACK_VIOLATION` |
| `layout` | `JSON[]` | `[{item_id, x, y, z, L, W, H, color_group, label}]` · null nếu INFEASIBLE |
| `space_utilization_cbm_pct` | `DECIMAL(5,2)` | `total_cargo_cbm / container_volume × 100` |
| `space_utilization_weight_pct` | `DECIMAL(5,2)` | `total_cargo_weight / container_max_weight × 100` |
| `split_suggestion` | `BOOLEAN` | `TRUE` nếu INFEASIBLE do CBM — gợi ý tách 2 booking |

---

### 3. Core Logic (FFD Algorithm)

```python
def pack(items: list[CargoItem], container: ContainerSpec) -> PackingResult:
    """
    First Fit Decreasing 3D Bin Packing.
    Time complexity: O(n² × m) với n = items, m = available spaces.
    """
    # STEP 1: Sort items by volume DESCENDING (FFD heuristic)
    sorted_items = sorted(items, key=lambda i: i.volume, reverse=True)

    placed = []
    space_used_cbm = Decimal("0")
    space_used_gw   = Decimal("0")
    available_spaces = [Position(x=0, y=0, z=0)]  # Start với góc dưới-trái-trước

    # STEP 2: Try to place each item
    for item in sorted_items:
        placed_flag = False

        for position in available_spaces:
            # Check tất cả constraints theo thứ tự từ rẻ → đắt (fail-fast)
            if not fits_in_container(item, position, container):
                continue
            if not no_overlap(item, position, placed):
                continue
            if not chemical_compatible(item, placed):  # Matrix check
                return PackingResult(status="INFEASIBLE", reason_code="CHEMICAL_INCOMPATIBLE")
            if not stack_constraint_ok(item, position, placed):  # fragile check
                continue
            if space_used_gw + item.total_weight > container.max_weight_kg:
                return PackingResult(status="INFEASIBLE", reason_code="WEIGHT_EXCEEDED")

            # Place the item
            placed.append(PlacedItem(item=item, position=position))
            space_used_cbm += item.volume_m3 * item.qty
            space_used_gw   += item.total_weight
            available_spaces = update_available_spaces(available_spaces, item, position)
            placed_flag = True
            break

        if not placed_flag:
            reason = "CBM_EXCEEDED" if space_used_cbm + item.volume_m3 > container.volume_m3 \
                     else "ITEM_DOES_NOT_FIT"
            return PackingResult(
                status="INFEASIBLE",
                reason_code=reason,
                split_suggestion=(reason == "CBM_EXCEEDED")
            )

    # STEP 3: Build layout JSON for Three.js
    layout = [
        {
            "item_id": p.item.id, "x": p.position.x, "y": p.position.y, "z": p.position.z,
            "L": p.item.length, "W": p.item.width, "H": p.item.height,
            "color_group": get_hs_color_group(p.item.hs_code),
            "label": p.item.description,
        }
        for p in placed
    ]

    return PackingResult(
        status="FEASIBLE",
        layout=layout,
        space_utilization_cbm_pct=space_used_cbm / container.volume_m3 * 100,
        space_utilization_weight_pct=space_used_gw / container.max_weight_kg * 100,
    )
```

---

### 4. Test Scenarios

#### 4.1 Happy Path

```python
class TestBinPackingHappyPath:

    def test_single_small_item_is_feasible(self, engine, container_20dc):
        items = [CargoItem(length=50, width=40, height=30, weight=100, qty=1)]
        result = engine.pack(items, container_20dc)
        assert result.status == "FEASIBLE"
        assert len(result.layout) == 1
        assert 0 < result.space_utilization_cbm_pct < 1  # Rất nhỏ so với container

    def test_largest_item_placed_first_in_layout(self, engine, container_20dc):
        """FFD phải xếp item lớn nhất vào vị trí đầu tiên trong layout."""
        small = CargoItem(length=50, width=50, height=50, weight=50, qty=1, id="small")
        large = CargoItem(length=200, width=150, height=100, weight=500, qty=1, id="large")
        result = engine.pack([small, large], container_20dc)
        assert result.status == "FEASIBLE"
        assert result.layout[0]["item_id"] == "large"

    def test_fill_rate_calculation_accuracy(self, engine, container_20dc):
        """Fill rate phải chính xác ±0.5%."""
        # Container 20DC: 589×234×238 = 32.80 m³ thực dụng ≈ 25 m³ usable
        # 1 item = 250×220×180 / 1e6 = 9.9 m³
        item = CargoItem(length=250, width=220, height=180, weight=500, qty=1)
        result = engine.pack([item], container_20dc)
        expected_util = (0.250 * 0.220 * 0.180) / container_20dc.volume_m3 * 100
        assert abs(result.space_utilization_cbm_pct - expected_util) < 0.5
```

#### 4.2 Edge Cases

```python
class TestBinPackingEdgeCases:

    def test_item_exactly_fills_container_100pct(self, engine, container_20dc):
        """Item có kích thước bằng đúng inner dimensions → FEASIBLE, util ≈ 100%."""
        item = CargoItem(
            length=container_20dc.inner_length,
            width=container_20dc.inner_width,
            height=container_20dc.inner_height,
            weight=1000, qty=1
        )
        result = engine.pack([item], container_20dc)
        assert result.status == "FEASIBLE"
        assert result.space_utilization_cbm_pct > 99.0

    def test_item_1cm_too_long_returns_infeasible(self, engine, container_20dc):
        """Item dài hơn container 1cm → INFEASIBLE."""
        item = CargoItem(length=container_20dc.inner_length + 1,
                         width=100, height=100, weight=100, qty=1)
        result = engine.pack([item], container_20dc)
        assert result.status == "INFEASIBLE"
        assert result.reason_code == "ITEM_DOES_NOT_FIT"

    def test_fragile_item_cannot_have_items_above(self, engine, container_20dc):
        """Hàng fragile phải ở tầng trên cùng — không cho xếp chồng lên."""
        fragile = CargoItem(length=100, width=100, height=100, weight=100, qty=1,
                            fragile=True, stackable=False, id="fragile")
        heavy   = CargoItem(length=100, width=100, height=100, weight=500, qty=1,
                            fragile=False, stackable=True, id="heavy")
        result = engine.pack([fragile, heavy], container_20dc)
        # Nếu cả hai được xếp, fragile phải ở tầng cao hơn heavy
        if result.status == "FEASIBLE":
            fragile_z = next(p["z"] for p in result.layout if p["item_id"] == "fragile")
            heavy_z   = next(p["z"] for p in result.layout if p["item_id"] == "heavy")
            assert fragile_z >= heavy_z

    def test_empty_items_list_is_infeasible(self, engine, container_20dc):
        """Không có hàng hóa → INFEASIBLE (không có gì để xếp)."""
        result = engine.pack([], container_20dc)
        assert result.status == "INFEASIBLE"
        assert result.reason_code == "EMPTY_CARGO_LIST"

    def test_weight_exactly_at_limit_is_feasible(self, engine, container_20dc):
        """Tổng trọng lượng = 28,000 kg (đúng giới hạn 20DC) → FEASIBLE."""
        item = CargoItem(length=100, width=100, height=100, weight=28_000, qty=1)
        result = engine.pack([item], container_20dc)
        assert result.status == "FEASIBLE"

    def test_weight_1kg_over_limit_is_infeasible(self, engine, container_20dc):
        """Tổng trọng lượng = 28,001 kg → INFEASIBLE, reason = WEIGHT_EXCEEDED."""
        item = CargoItem(length=100, width=100, height=100, weight=28_001, qty=1)
        result = engine.pack([item], container_20dc)
        assert result.status == "INFEASIBLE"
        assert result.reason_code == "WEIGHT_EXCEEDED"
```

#### 4.3 Error Handling

```python
class TestBinPackingErrors:

    def test_chemical_incompatible_returns_immediately(self, engine, container_20dc):
        """Phát hiện hóa chất không tương thích → dừng ngay, không tiếp tục xếp."""
        items = [
            CargoItem(hs_code="28070010", qty=1, **acid_dims),    # H2SO4
            CargoItem(hs_code="28151100", qty=1, **base_dims),    # NaOH
        ]
        result = engine.pack(items, container_20dc)
        assert result.status == "INFEASIBLE"
        assert result.reason_code == "CHEMICAL_INCOMPATIBLE"
        assert result.layout is None  # Layout không được tạo

    def test_cbm_overflow_suggests_split(self, engine, container_20dc):
        """CBM > sức chứa → split_suggestion = TRUE."""
        items = make_items_with_total_cbm(30.0)  # > 25 m³ usable
        result = engine.pack(items, container_20dc)
        assert result.status == "INFEASIBLE"
        assert result.split_suggestion is True

    def test_negative_dimensions_raise_validation_error(self, engine, container_20dc):
        with pytest.raises(ValueError, match="Dimensions must be positive"):
            CargoItem(length=-10, width=50, height=50, weight=100, qty=1)
```

---

### 5. Verification

```bash
# Full test suite F-03
pytest tests/unit/algorithms/test_bin_packing.py -v --tb=short

# Coverage (target ≥ 90%)
pytest tests/unit/algorithms/test_bin_packing.py \
    --cov=src/algorithms/bin_packing \
    --cov-report=html \
    --cov-fail-under=90

# Property-based testing (Hypothesis)
pytest tests/property/test_bin_packing_properties.py -v
# Hypothesis tự generate random inputs, kiểm tra bất biến:
# - util_pct luôn ∈ [0, 100]
# - Tổng volume placed ≤ container volume
# - Không có 2 items overlap

# Performance benchmark
pytest tests/benchmark/test_bin_packing_perf.py -v
# Expected: pack(50 items, 20DC) < 500ms
```

---

---

<a id="f-04"></a>
## F-04 — PSO Routing Optimizer (ECHDL-PSO — Layer 2)

### 1. Feature Overview

**Lớp 2 của Matching Engine**, chạy **song song** với L3 sau khi L1 trả về `FEASIBLE`. Tìm lộ trình xe tải tối ưu trên hành trình `Depot → CFS → Gate-in (Port)`. Tích hợp **Buffer Time từ Redis cache** (được tạo bởi SUMO offline). Đảm bảo ràng buộc cứng `t_total ≤ T_cutoff` với hàm phạt exponential.

---

### 2. Data Schema

#### 2.1 Input

| Field | Type | Source | Description |
|---|---|---|---|
| `depot_gps` | `POINT` | Container.depot_location | Tọa độ depot xuất phát |
| `cfs_gps` | `POINT` | BookingRequest.cfs_origin | Tọa độ kho CFS |
| `port_gps` | `POINT` | Static config | Cổng Gate-in cảng Cát Lái |
| `t_cutoff` | `TIMESTAMP` | Container.closing_time | Hạn chót Gate-in |
| `departure_time` | `TIMESTAMP` | Scheduling input | Thời điểm xe rời Depot |
| `stuffing_cbm` | `DECIMAL` | L1 output | Dùng tính thời gian đóng hàng |

#### 2.2 PSO Parameters

```python
PSO_CONFIG = {
    "n_particles":    30,     # Số lượng hạt trong đàn
    "max_iterations": 200,    # Số vòng lặp tối đa
    "w_initial":      0.9,    # Inertia weight ban đầu
    "w_final":        0.4,    # Inertia weight cuối (giảm tuyến tính)
    "c1":             2.0,    # Cognitive coefficient
    "c2":             2.0,    # Social coefficient
    "convergence_patience": 20,     # Dừng nếu gbest không cải thiện
    "convergence_threshold": 1e-6,  # Ngưỡng thay đổi gbest
}
```

#### 2.3 Output

| Field | Type | Description |
|---|---|---|
| `optimal_route` | `Waypoint[]` | `[{location, eta, leg_distance_km, leg_duration_min}]` |
| `t_total_min` | `FLOAT` | Tổng thời gian từ Depot → Gate-in (phút) |
| `eta_gate_in` | `TIMESTAMP` | ETA Gate-in thực tế |
| `t_cutoff_slack_min` | `FLOAT` | `(T_cutoff − eta_gate_in)` tính bằng phút · < 0 = vi phạm |
| `c_road_vnd` | `DECIMAL(12,0)` | Chi phí xe tải ước tính (VNĐ) |
| `c_penalty_risk` | `DECIMAL(12,0)` | Giá trị kỳ vọng phí phạt trễ |
| `convergence_iter` | `INTEGER` | Số vòng lặp thực tế đến hội tụ |
| `buffer_time_applied` | `INTEGER` | Buffer Time (phút) được tra từ Redis |

---

### 3. Core Logic

```python
def optimize_route(params: PSOInput) -> PSOResult:
    """ECHDL-PSO Variant — Enhanced Constrained Handling with Dynamic Learning."""
    
    # STEP 1: Initialize particles
    particles = [Particle.random_init() for _ in range(PSO_CONFIG["n_particles"])]
    gbest = None
    stagnation_count = 0

    # STEP 2: Get Buffer Time from Redis (NOT from SUMO realtime)
    hour_of_day = params.departure_time.hour
    buffer_time_min = redis.get(f"buffer_time:{hour_of_day}") or DEFAULT_BUFFER[hour_of_day]

    # STEP 3: PSO Main Loop
    for iteration in range(PSO_CONFIG["max_iterations"]):
        w = linear_decay(PSO_CONFIG["w_initial"], PSO_CONFIG["w_final"], iteration,
                          PSO_CONFIG["max_iterations"])

        for particle in particles:
            # Evaluate fitness
            t_total = calculate_travel_time(particle.position, params, buffer_time_min)
            c_road  = calculate_road_cost(particle.position, params)
            c_penalty = calculate_penalty(t_total, params.t_cutoff, params.departure_time)
            
            fitness = c_road + c_penalty  # Minimize total on-ground cost

            # Update personal best
            if fitness < particle.pbest_fitness:
                particle.pbest = particle.position.copy()
                particle.pbest_fitness = fitness

        # Update global best
        current_best = min(particles, key=lambda p: p.pbest_fitness)
        if gbest is None or current_best.pbest_fitness < gbest.fitness:
            gbest = GlobalBest(position=current_best.pbest, fitness=current_best.pbest_fitness)
            stagnation_count = 0
        else:
            stagnation_count += 1

        # Early stop (convergence)
        if stagnation_count >= PSO_CONFIG["convergence_patience"]:
            break

        # Velocity update: v(t+1) = w*v(t) + c1*rand*(pbest-x) + c2*rand*(gbest-x)
        for particle in particles:
            r1, r2 = random(), random()
            particle.velocity = (
                w * particle.velocity
                + PSO_CONFIG["c1"] * r1 * (particle.pbest - particle.position)
                + PSO_CONFIG["c2"] * r2 * (gbest.position - particle.position)
            )
            particle.position = particle.position + particle.velocity

    return build_result(gbest, params, buffer_time_min, iteration)


def calculate_penalty(t_total_min: float, t_cutoff: datetime, departure: datetime) -> Decimal:
    """
    Hàm phạt exponential khi t_total tiệm cận T_cutoff.
    C_penalty = 0                              nếu slack > 60 phút
    C_penalty = BASE_PENALTY × e^(k × deficit) nếu slack ≤ 60 phút
    """
    eta_gate_in = departure + timedelta(minutes=t_total_min)
    slack_min = (t_cutoff - eta_gate_in).total_seconds() / 60

    if slack_min > 60:
        return Decimal("0")
    elif slack_min > 0:
        k = 0.05  # Decay constant
        return PENALTY_BASE * Decimal(str(math.exp(k * (60 - slack_min))))
    else:
        return PENALTY_HARD_VIOLATION  # = 999_999_999 VNĐ
```

---

### 4. Test Scenarios

#### 4.1 Happy Path

```python
class TestPSORoutingHappyPath:

    def test_optimal_route_found_within_200_iterations(self, optimizer, sample_route_params):
        result = optimizer.optimize_route(sample_route_params)
        assert result.convergence_iter <= 200
        assert result.t_cutoff_slack_min > 0
        assert result.c_road_vnd > 0

    def test_off_peak_route_has_less_buffer_time(self, optimizer):
        """Khung giờ 10:00 (off-peak) phải có buffer time < khung giờ 8:00 (peak)."""
        peak_params    = make_params(departure_hour=8)
        offpeak_params = make_params(departure_hour=10)
        peak_result    = optimizer.optimize_route(peak_params)
        offpeak_result = optimizer.optimize_route(offpeak_params)
        assert offpeak_result.buffer_time_applied < peak_result.buffer_time_applied

    def test_result_includes_waypoints_in_correct_order(self, optimizer, sample_route_params):
        """Lộ trình phải đi theo thứ tự: Depot → CFS → Gate-in."""
        result = optimizer.optimize_route(sample_route_params)
        assert result.optimal_route[0]["location_type"] == "DEPOT"
        assert result.optimal_route[1]["location_type"] == "CFS"
        assert result.optimal_route[-1]["location_type"] == "GATE_IN"
```

#### 4.2 Edge Cases

```python
class TestPSORoutingEdgeCases:

    def test_depot_same_location_as_cfs(self, optimizer):
        """CFS cùng địa chỉ với Depot → leg_distance = 0."""
        params = make_params(depot_gps=CFS_GPS, cfs_gps=CFS_GPS)
        result = optimizer.optimize_route(params)
        assert result.optimal_route[0]["leg_distance_km"] == pytest.approx(0, abs=0.1)

    def test_t_cutoff_exactly_at_minimum_feasible_window(self, optimizer):
        """T_cutoff = NOW + t_total_minimum → slack ≈ 0, penalty rất cao."""
        min_travel = optimizer.estimate_minimum_travel_time(sample_route_params)
        params = make_params(t_cutoff=now() + timedelta(minutes=min_travel + 5))
        result = optimizer.optimize_route(params)
        assert result.t_cutoff_slack_min < 10  # Chỉ còn < 10 phút đệm
        assert result.c_penalty_risk > 0

    def test_convergence_before_max_iterations(self, optimizer, sample_route_params):
        """PSO phải hội tụ trước 200 vòng lặp với input bình thường."""
        result = optimizer.optimize_route(sample_route_params)
        assert result.convergence_iter < 200

    def test_redis_buffer_time_unavailable_uses_default(self, optimizer, mock_redis_down):
        """Khi Redis không khả dụng → dùng default buffer time (không crash)."""
        result = optimizer.optimize_route(sample_route_params)
        assert result.status == "SUCCESS"
        assert result.buffer_time_applied == DEFAULT_BUFFER[sample_route_params.departure_time.hour]
```

#### 4.3 Error Handling

```python
class TestPSORoutingErrors:

    def test_impossible_t_cutoff_returns_hard_violation(self, optimizer):
        """T_cutoff trong quá khứ → penalty = HARD_VIOLATION."""
        params = make_params(t_cutoff=now() - timedelta(hours=1))
        result = optimizer.optimize_route(params)
        assert result.c_penalty_risk == PENALTY_HARD_VIOLATION
        assert result.t_cutoff_slack_min < 0

    def test_negative_coordinates_raise_validation_error(self, optimizer):
        """GPS coordinates ngoài range hợp lệ."""
        with pytest.raises(ValidationError, match="Invalid GPS coordinates"):
            optimizer.optimize_route(make_params(depot_gps=(200, 200)))

    def test_c_penalty_increases_exponentially_near_cutoff(self, optimizer):
        """Hàm phạt phải tăng exponential khi slack giảm về 0."""
        slacks = [60, 45, 30, 15, 5]
        penalties = [
            optimizer.calculate_penalty(
                t_total_min=BASE_TRAVEL + (60 - s),
                t_cutoff=BASE_CUTOFF,
                departure=BASE_DEPARTURE
            )
            for s in slacks
        ]
        # Kiểm tra tính đơn điệu tăng và exponential ratio
        for i in range(len(penalties) - 1):
            assert penalties[i+1] > penalties[i]  # Monotonically increasing
        growth_ratios = [penalties[i+1] / penalties[i] for i in range(len(penalties)-1)]
        # Exponential → các ratio phải xấp xỉ nhau
        assert max(growth_ratios) / min(growth_ratios) < 3.0
```

---

### 5. Verification

```bash
# Unit tests F-04
pytest tests/unit/algorithms/test_pso_optimizer.py -v

# Coverage (target ≥ 85%)
pytest tests/unit/algorithms/test_pso_optimizer.py \
    --cov=src/algorithms/pso_optimizer \
    --cov-report=term-missing \
    --cov-fail-under=85

# Test penalty function riêng
pytest tests/unit/algorithms/test_pso_optimizer.py -k "penalty" -v

# Performance: PSO phải hoàn thành trong 30 giây
pytest tests/benchmark/test_pso_performance.py -v
# Assertion: optimizer.optimize_route(params) runs in < 30,000ms
```

---

---

<a id="f-05"></a>
## F-05 — Dynamic Pricing Engine (Layer 3 Component)

### 1. Feature Overview

Tính giá cước biển động `C_sea_final` dựa trên **Space Utilization %** và **Days to ETD**, nhân với hệ số mùa vụ `seasonal_factor`. Cập nhật tự động mỗi 15 phút và push qua WebSocket đến Dashboard. Là thành phần cốt lõi của Layer 3 (NSGA-II) và là nguồn dữ liệu cho Dynamic Pricing Board UI.

---

### 2. Data Schema

#### 2.1 Input

| Field | Type | Constraints |
|---|---|---|
| `floor_price` | `DECIMAL(12,0)` | > 0 · VNĐ/TEU · từ Container.floor_price |
| `space_util_pct` | `DECIMAL(5,2)` | ∈ [0.0, 100.0] · từ L1 output |
| `days_to_etd` | `FLOAT` | ≥ 0 · tính từ `(ETD - NOW()) / 86400` |
| `booking_month` | `INTEGER` | ∈ [1, 12] |

#### 2.2 Pricing Matrix

```python
# discount_rate(space_util_pct, days_to_etd) → float
# Giá trị âm = premium (giá cao hơn sàn), dương = discount

PRICING_MATRIX = {
    # (util_range, days_range): discount_rate
    ((80, 100), (7, INF)):  0.00,   # Giá sàn gốc
    ((80, 100), (3,   7)): -0.05,   # Premium +5%
    ((80, 100), (1,   3)): -0.15,   # Premium +15%
    ((80, 100), (0,   1)): -0.10,   # Giá thị trường +10%
    ((60,  80), (7, INF)):  0.10,   # Discount 10%
    ((40,  60), (7, INF)):  0.20,   # Discount 20%
    ((0,   40), (7, INF)):  0.40,   # Discount 40% (đẩy nhanh gom hàng)
}

SEASONAL_FACTORS = {
    11: 1.15,  # Tháng 11 — peak FDI
    12: 1.15,  # Tháng 12 — peak FDI
    2:  0.85,  # Tháng 2 — sau Tết
    3:  0.85,  # Tháng 3 — sau Tết
    # Các tháng còn lại: 1.00
}
```

#### 2.3 Output

| Field | Type | Description |
|---|---|---|
| `final_price` | `DECIMAL(12,0)` | `floor_price × (1 − discount_rate) × seasonal_factor` |
| `discount_rate` | `DECIMAL(5,4)` | Tỷ lệ discount áp dụng (âm = premium) |
| `seasonal_factor` | `DECIMAL(4,2)` | Hệ số mùa vụ |
| `price_category` | `STRING` | `FLOOR` \| `PREMIUM` \| `DISCOUNT_10` \| `DISCOUNT_20` \| `DISCOUNT_40` |
| `savings_vs_market_pct` | `DECIMAL(5,2)` | % tiết kiệm so với Z_market |

---

### 3. Core Logic

```python
def calculate(params: PricingParams) -> PricingResult:
    """
    Công thức: C_sea_final = C_sea_floor × (1 − discount_rate) × seasonal_factor
    """
    # STEP 1: Lookup discount rate
    discount_rate = lookup_discount(params.space_util_pct, params.days_to_etd)

    # STEP 2: Lookup seasonal factor
    seasonal_factor = Decimal(str(SEASONAL_FACTORS.get(params.booking_month, 1.00)))

    # STEP 3: Calculate final price
    multiplier  = Decimal("1") - Decimal(str(discount_rate))
    final_price = (params.floor_price * multiplier * seasonal_factor).quantize(
        Decimal("1"),  # Round to VNĐ (no decimals)
        rounding=ROUND_HALF_UP
    )

    # STEP 4: Determine price category
    if discount_rate < 0:
        category = "PREMIUM"
    elif discount_rate == 0:
        category = "FLOOR"
    elif discount_rate <= 0.10:
        category = "DISCOUNT_10"
    elif discount_rate <= 0.20:
        category = "DISCOUNT_20"
    else:
        category = "DISCOUNT_40"

    return PricingResult(
        final_price=final_price,
        discount_rate=Decimal(str(discount_rate)),
        seasonal_factor=seasonal_factor,
        price_category=category,
    )
```

---

### 4. Test Scenarios

#### 4.1 Happy Path

```python
class TestDynamicPricingHappyPath:

    @pytest.mark.parametrize("util_pct, days, month, expected_discount, expected_seasonal", [
        (85.0, 8.0,  6,  0.00, 1.00),  # High util, far ETD, normal month
        (85.0, 5.0,  6, -0.05, 1.00),  # High util, 3-7 days → premium
        (85.0, 2.0, 11, -0.15, 1.15),  # Premium + peak FDI season
        (70.0, 8.0,  2,  0.10, 0.85),  # Discount + post-Tet
        (30.0, 8.0,  6,  0.40, 1.00),  # Low fill → max discount
    ])
    def test_pricing_matrix_correctness(self, engine, util_pct, days, month,
                                         expected_discount, expected_seasonal):
        params = PricingParams(
            floor_price=Decimal("5_000_000"),
            space_util_pct=util_pct, days_to_etd=days, booking_month=month
        )
        result = engine.calculate(params)
        expected_price = (Decimal("5000000")
                          * (Decimal("1") - Decimal(str(expected_discount)))
                          * Decimal(str(expected_seasonal)))
        assert result.final_price == expected_price.quantize(Decimal("1"), ROUND_HALF_UP)
        assert result.seasonal_factor == Decimal(str(expected_seasonal))
```

#### 4.2 Edge Cases

```python
class TestDynamicPricingEdgeCases:

    def test_util_exactly_80pct_with_days_7_uses_floor_price(self, engine):
        """Biên util=80%, days=7 → chính xác dùng FLOOR rate."""
        params = PricingParams(floor_price=Decimal("5_000_000"),
                               space_util_pct=80.0, days_to_etd=7.0, booking_month=6)
        result = engine.calculate(params)
        assert result.discount_rate == Decimal("0.00")
        assert result.final_price == Decimal("5_000_000")

    def test_util_79_99_with_days_7_uses_discount_10(self, engine):
        """util=79.99% → vào bracket 60-80% → 10% discount."""
        params = PricingParams(floor_price=Decimal("5_000_000"),
                               space_util_pct=79.99, days_to_etd=7.0, booking_month=6)
        result = engine.calculate(params)
        assert result.discount_rate == Decimal("0.10")
        assert result.final_price == Decimal("4_500_000")

    def test_days_to_etd_zero_applies_correct_rate(self, engine):
        """days_to_etd = 0 (tàu chạy hôm nay) với util cao → premium +10%."""
        params = PricingParams(floor_price=Decimal("5_000_000"),
                               space_util_pct=85.0, days_to_etd=0.0, booking_month=6)
        result = engine.calculate(params)
        assert result.discount_rate == Decimal("-0.10")

    def test_price_uses_decimal_not_float(self, engine):
        """final_price phải là Decimal để tránh floating-point error."""
        params = PricingParams(floor_price=Decimal("3_750_000"),
                               space_util_pct=50.0, days_to_etd=10.0, booking_month=6)
        result = engine.calculate(params)
        assert isinstance(result.final_price, Decimal)
```

#### 4.3 Error Handling

```python
class TestDynamicPricingErrors:

    def test_floor_price_zero_raises_error(self, engine):
        with pytest.raises(InvalidPricingParamsError, match="floor_price must be > 0"):
            engine.calculate(PricingParams(floor_price=Decimal("0"), ...))

    def test_util_pct_above_100_raises_error(self, engine):
        with pytest.raises(InvalidPricingParamsError, match="space_util_pct ∈ \\[0, 100\\]"):
            engine.calculate(PricingParams(space_util_pct=100.1, ...))

    def test_negative_days_to_etd_raises_error(self, engine):
        """ETD trong quá khứ không hợp lệ cho pricing."""
        with pytest.raises(InvalidPricingParamsError, match="days_to_etd must be >= 0"):
            engine.calculate(PricingParams(days_to_etd=-1.0, ...))
```

---

### 5. Verification

```bash
# Unit tests F-05 (phải đạt ≥ 90% coverage)
pytest tests/unit/algorithms/test_pricing_engine.py -v \
    --cov=src/algorithms/pricing_engine \
    --cov-fail-under=90

# Test toàn bộ pricing matrix
pytest tests/unit/algorithms/test_pricing_engine.py -k "pricing_matrix" -v

# Kiểm tra Decimal precision (không dùng float)
pytest tests/unit/algorithms/test_pricing_engine.py -k "decimal" -v

# Integration: WebSocket push mỗi 15 phút
pytest tests/integration/test_pricing_websocket.py -v
```

---

---

<a id="f-06"></a>
## F-06 — Time-lock Dual Confirmation (Redis TTL)

### 1. Feature Overview

Cơ chế **Time-lock 30 phút** đảm bảo giao dịch không bị "treo" vô thời hạn. Sau khi Matching Engine đề xuất phương án, **cả Hãng tàu và NVOCC** phải xác nhận trong 30 phút. Redis key với TTL=1800s là nguồn sự thật duy nhất. Hết hạn → auto-reject, container → AVAILABLE, Pareto rank 2 được đề xuất.

---

### 2. Data Schema

#### 2.1 Redis Key Schema

```
Key   : "container:{container_id}:lock"
Value : JSON { match_id, nvocc_id, shipping_line_id, proposed_at, pareto_rank }
TTL   : 1800 seconds (30 phút)
Type  : STRING (JSON encoded)
```

#### 2.2 Confirmation States

```
PROPOSED  → (Shipping Line confirms) → SHIPPING_LINE_CONFIRMED
PROPOSED  → (NVOCC confirms)         → NVOCC_CONFIRMED
SHIPPING_LINE_CONFIRMED + NVOCC_CONFIRMED → CONFIRMED (Transaction created)
Any state → (Reject or TTL expire)   → RELEASED (container back to AVAILABLE)
```

---

### 3. Core Logic

```python
def initiate_time_lock(match_id: UUID, container_id: str,
                        nvocc_id: UUID, shipping_line_id: UUID) -> TimeLockResult:
    lock_key = f"container:{container_id}:lock"
    lock_value = json.dumps({
        "match_id": str(match_id),
        "nvocc_id": str(nvocc_id),
        "shipping_line_id": str(shipping_line_id),
        "proposed_at": datetime.utcnow().isoformat(),
        "pareto_rank": 1,
        "confirmations": {"shipping_line": False, "nvocc": False},
    })
    success = redis.set(lock_key, lock_value, ex=1800, nx=True)  # NX = only set if not exists
    if not success:
        raise ContainerAlreadyLockedException(container_id)
    db.update_container_status(container_id, "LOCKED")
    notify_both_parties(match_id, nvocc_id, shipping_line_id)
    return TimeLockResult(lock_key=lock_key, expires_at=now() + timedelta(seconds=1800))


def confirm_match(match_id: UUID, confirming_party: str, container_id: str) -> ConfirmResult:
    lock_key = f"container:{container_id}:lock"
    lock_data = json.loads(redis.get(lock_key) or "null")

    if lock_data is None:
        raise LockExpiredOrNotFoundError(container_id)  # TTL expired

    lock_data["confirmations"][confirming_party] = True
    redis.set(lock_key, json.dumps(lock_data), keepttl=True)  # Preserve remaining TTL

    if all(lock_data["confirmations"].values()):  # Cả hai bên đã xác nhận
        redis.delete(lock_key)
        db.create_transaction(match_id)
        db.update_container_status(container_id, "CONFIRMED")
        auto_generate_trucking_order(match_id)
        auto_generate_slot_sharing_agreement(match_id)
        return ConfirmResult(status="FULLY_CONFIRMED")

    return ConfirmResult(status="AWAITING_OTHER_PARTY")


def handle_rejection_or_expiry(container_id: str, match_id: UUID, reason: str):
    """Gọi khi 1 bên từ chối HOẶC Redis TTL hết hạn (keyspace notification)."""
    redis.delete(f"container:{container_id}:lock")
    db.update_container_status(container_id, "AVAILABLE")
    db.update_match_status(match_id, "REJECTED", reason=reason)
    
    # Tự động đề xuất Pareto rank 2 (trong 5 phút)
    celery.send_task("matching.propose_next_pareto", 
                      args=[match_id], countdown=0)
```

---

### 4. Test Scenarios

#### 4.1 Happy Path

```python
class TestTimeLockHappyPath:

    def test_both_parties_confirm_creates_transaction(self, redis_mock, db_mock):
        lock = initiate_time_lock(MATCH_ID, CONTAINER_ID, NVOCC_ID, SHIPPING_LINE_ID)
        assert lock is not None

        result1 = confirm_match(MATCH_ID, "shipping_line", CONTAINER_ID)
        assert result1.status == "AWAITING_OTHER_PARTY"

        result2 = confirm_match(MATCH_ID, "nvocc", CONTAINER_ID)
        assert result2.status == "FULLY_CONFIRMED"
        db_mock.create_transaction.assert_called_once_with(MATCH_ID)
        assert redis_mock.exists(f"container:{CONTAINER_ID}:lock") == 0  # Key deleted

    def test_ttl_is_set_to_1800_seconds(self, redis_mock):
        initiate_time_lock(MATCH_ID, CONTAINER_ID, NVOCC_ID, SHIPPING_LINE_ID)
        ttl = redis_mock.ttl(f"container:{CONTAINER_ID}:lock")
        assert 1795 <= ttl <= 1800  # ±5s tolerance
```

#### 4.2 Edge Cases

```python
class TestTimeLockEdgeCases:

    def test_lock_already_exists_raises_exception(self, redis_mock):
        """Container đã bị lock không thể lock lại (NX flag)."""
        initiate_time_lock(MATCH_ID, CONTAINER_ID, NVOCC_ID, SHIPPING_LINE_ID)
        with pytest.raises(ContainerAlreadyLockedException):
            initiate_time_lock(uuid4(), CONTAINER_ID, uuid4(), SHIPPING_LINE_ID)

    def test_confirm_after_ttl_expired_raises_error(self, redis_mock, freeze_time):
        """Xác nhận sau khi TTL hết hạn → LockExpiredOrNotFoundError."""
        initiate_time_lock(MATCH_ID, CONTAINER_ID, NVOCC_ID, SHIPPING_LINE_ID)
        freeze_time.advance(seconds=1801)  # Vượt 30 phút
        with pytest.raises(LockExpiredOrNotFoundError):
            confirm_match(MATCH_ID, "nvocc", CONTAINER_ID)

    def test_rejection_by_one_party_releases_container(self, redis_mock, db_mock):
        """1 bên từ chối → container quay về AVAILABLE, đề xuất Pareto rank 2."""
        initiate_time_lock(MATCH_ID, CONTAINER_ID, NVOCC_ID, SHIPPING_LINE_ID)
        handle_rejection_or_expiry(CONTAINER_ID, MATCH_ID, reason="NVOCC_REJECTED")
        assert redis_mock.exists(f"container:{CONTAINER_ID}:lock") == 0
        db_mock.update_container_status.assert_called_with(CONTAINER_ID, "AVAILABLE")
```

#### 4.3 Error Handling

```python
class TestTimeLockErrors:

    def test_redis_down_falls_back_gracefully(self, mock_redis_unavailable):
        """Redis không khả dụng → raise ServiceUnavailableError (không crash silently)."""
        with pytest.raises(ServiceUnavailableError, match="Redis"):
            initiate_time_lock(MATCH_ID, CONTAINER_ID, NVOCC_ID, SHIPPING_LINE_ID)

    def test_confirm_with_invalid_party_raises_validation_error(self, redis_mock):
        """Party name không hợp lệ → ValidationError."""
        initiate_time_lock(MATCH_ID, CONTAINER_ID, NVOCC_ID, SHIPPING_LINE_ID)
        with pytest.raises(ValidationError, match="Invalid party"):
            confirm_match(MATCH_ID, "trucking_company", CONTAINER_ID)
```

---

### 5. Verification

```bash
# Unit tests F-06
pytest tests/unit/transaction/test_timelock.py -v \
    --cov=src/transaction/timelock \
    --cov-fail-under=85

# Test với fakeredis (không cần Redis server thật)
pip install fakeredis
pytest tests/unit/transaction/test_timelock.py -v --use-fakeredis

# Integration test: kiểm tra Redis keyspace expiry notification
pytest tests/integration/test_timelock_expiry.py -v
# Requires: redis-server với notify-keyspace-events=Ex

# Kiểm tra race condition (concurrent lock attempts)
pytest tests/concurrency/test_timelock_race.py -v --timeout=30
```

---

---

<a id="f-08"></a>
## F-08 — Cabotage Compliance Validator

### 1. Feature Overview

Validator pháp lý bắt buộc theo **NĐ 160/2016/NĐ-CP**. Hàng nội địa tuyến Cát Lái → Hải Phòng **BẮT BUỘC** đi tàu mang cờ Việt Nam. Validator phải block hoàn toàn mọi `SlotSharingAgreement` có `vessel_flag ≠ "VN"`. Đây là ràng buộc **pháp lý** — vi phạm làm vô hiệu toàn bộ giao dịch.

---

### 2. Data Schema

#### 2.1 Input

| Field | Type | Constraints |
|---|---|---|
| `vessel_flag` | `VARCHAR(3)` | Must = `"VN"` · ISO 3166-1 alpha-2 |
| `vessel_name` | `VARCHAR(100)` | Not null · Not empty |
| `feeder_operator_id` | `UUID` | Must be in whitelist (Hải An, VIMC, Vietnam Feeder) |
| `route_origin` | `VARCHAR(5)` | `VNCLI` |
| `route_destination` | `VARCHAR(5)` | `VNHPH` |

#### 2.2 Whitelisted Feeder Operators

```python
APPROVED_DOMESTIC_FEEDERS = {
    "hai-an-shipping-uuid":    "Hải An Shipping Lines",
    "vimc-uuid":               "VIMC (Vietnam Maritime Corporation)",
    "vietnam-feeder-uuid":     "Vietnam Feeder Lines",
}
```

---

### 3. Core Logic

```python
def validate_cabotage(agreement_draft: SlotSharingAgreementDraft) -> ValidationResult:
    errors = []

    # CHECK 1: vessel_flag phải là "VN" (hard constraint)
    if agreement_draft.vessel_flag != "VN":
        errors.append(CabotageViolation(
            code="NON_VN_FLAG",
            message=f"vessel_flag='{agreement_draft.vessel_flag}' violates NĐ 160/2016. "
                    f"Domestic cargo MUST use Vietnamese-flagged vessels.",
            severity="LEGAL_BLOCK"
        ))

    # CHECK 2: feeder_operator phải trong whitelist
    if agreement_draft.feeder_operator_id not in APPROVED_DOMESTIC_FEEDERS:
        errors.append(CabotageViolation(
            code="UNAPPROVED_FEEDER",
            message=f"Feeder operator not in approved domestic feeder whitelist.",
            severity="LEGAL_BLOCK"
        ))

    # CHECK 3: Route phải là domestic (Cát Lái → Hải Phòng)
    if not (agreement_draft.route_origin == "VNCLI" and 
            agreement_draft.route_destination == "VNHPH"):
        errors.append(CabotageViolation(
            code="INVALID_DOMESTIC_ROUTE",
            message="Cabotage rule applies to VNCLI→VNHPH route only.",
            severity="LEGAL_BLOCK"
        ))

    if errors:
        return ValidationResult(is_valid=False, violations=errors)
    return ValidationResult(is_valid=True, violations=[])
```

---

### 4. Test Scenarios

#### 4.1 Happy Path

```python
def test_vn_flagged_vessel_with_approved_feeder_passes():
    draft = SlotSharingAgreementDraft(
        vessel_flag="VN", vessel_name="Hai An 05",
        feeder_operator_id="hai-an-shipping-uuid",
        route_origin="VNCLI", route_destination="VNHPH"
    )
    result = validate_cabotage(draft)
    assert result.is_valid is True
    assert len(result.violations) == 0
```

#### 4.2 Edge Cases & Error Handling

```python
@pytest.mark.parametrize("vessel_flag, expected_code", [
    ("SG", "NON_VN_FLAG"),    # Singapore flag
    ("CN", "NON_VN_FLAG"),    # China flag
    ("",   "NON_VN_FLAG"),    # Empty
    ("vn", "NON_VN_FLAG"),    # Lowercase — must be exact "VN"
    ("VNM", "NON_VN_FLAG"),   # 3-letter code (ISO 3166-1 alpha-3) không chấp nhận
])
def test_non_vn_flags_are_blocked(vessel_flag, expected_code):
    draft = SlotSharingAgreementDraft(vessel_flag=vessel_flag, **valid_other_fields)
    result = validate_cabotage(draft)
    assert result.is_valid is False
    assert any(v.code == expected_code for v in result.violations)

def test_unapproved_feeder_operator_is_blocked():
    draft = SlotSharingAgreementDraft(
        vessel_flag="VN", feeder_operator_id="unknown-feeder-uuid", **valid_other_fields
    )
    result = validate_cabotage(draft)
    assert result.is_valid is False
    assert any(v.code == "UNAPPROVED_FEEDER" for v in result.violations)
```

---

### 5. Verification

```bash
# Coverage target: ≥ 95% (pháp lý — zero tolerance)
pytest tests/unit/validators/test_cabotage_validator.py -v \
    --cov=src/validators/cabotage_validator \
    --cov-fail-under=95

pytest tests/unit/validators/ -k "cabotage or vessel_flag" -v
```

---

---

## APPENDIX — Master Verification Script

```bash
#!/bin/bash
# run_all_specs.sh — Chạy toàn bộ test suite theo spec

set -e  # Dừng ngay khi có lỗi

echo "=== ECR-LCL Platform — Full Spec Verification ==="

echo "--- F-01: Container Registration ---"
pytest tests/unit/supply/ -v --cov=src/supply --cov-fail-under=90 -q

echo "--- F-02: LCL Booking & HS Code Checker ---"
pytest tests/unit/demand/ -v --cov=src/demand --cov-fail-under=90 -q

echo "--- F-03: 3D Bin Packing (FFD L1) ---"
pytest tests/unit/algorithms/test_bin_packing.py -v \
    --cov=src/algorithms/bin_packing --cov-fail-under=90 -q

echo "--- F-04: PSO Routing Optimizer (L2) ---"
pytest tests/unit/algorithms/test_pso_optimizer.py -v \
    --cov=src/algorithms/pso_optimizer --cov-fail-under=85 -q

echo "--- F-05: Dynamic Pricing Engine ---"
pytest tests/unit/algorithms/test_pricing_engine.py -v \
    --cov=src/algorithms/pricing_engine --cov-fail-under=90 -q

echo "--- F-06: Time-lock Dual Confirmation ---"
pytest tests/unit/transaction/test_timelock.py -v \
    --cov=src/transaction/timelock --cov-fail-under=85 -q

echo "--- F-08: Cabotage Compliance Validator ---"
pytest tests/unit/validators/test_cabotage_validator.py -v \
    --cov=src/validators/cabotage_validator --cov-fail-under=95 -q

echo "--- Overall Coverage Check ---"
pytest tests/unit/ --cov=src --cov-report=term-missing \
    --cov-report=html:coverage_report --cov-fail-under=80 -q

echo ""
echo "✅ All spec verifications PASSED"
echo "📊 Coverage report: ./coverage_report/index.html"
```

---

*Tài liệu OpenSpec này là nguồn tham chiếu duy nhất để AI thực thi từng feature. Mỗi test scenario là một tiêu chí chấp nhận (Acceptance Criteria) bắt buộc phải pass trước khi feature được đánh dấu hoàn thành.*

---
**Phiên bản**: `v1.0` | **Căn cứ**: ECR-LCL System Spec v3.0 | **Cập nhật**: 2026-03-10
