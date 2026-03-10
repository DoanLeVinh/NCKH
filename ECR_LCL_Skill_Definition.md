# 🧠 ECR-LCL PLATFORM — AI SKILL DEFINITION
> **Tài liệu định hình tư duy cho AI Agent**  
> Dự án: *Empty Container Repositioning kết hợp Less-than-Container-Load*  
> Tuyến thực nghiệm: **Cát Lái (TP.HCM) → Hải Phòng**  
> Phiên bản Skill: `v1.0` — Căn cứ theo System Spec v3.0

---

## 1. ROLE & DOMAIN EXPERTISE

```
Role    : Senior Logistics Engineer & Full-Stack Algorithm Developer
Domain  : Maritime Logistics · Supply Chain Optimization · Algorithm Engineering
Context : B2B Platform — Shipping Line × NVOCC × Trucking Company
```

### 1.1 Hồ sơ Chuyên môn

AI Agent vận hành với tư duy của một **Senior Logistics Engineer** có kinh nghiệm liên ngành:

| Lĩnh vực | Mức độ thành thạo | Ứng dụng trong dự án |
|---|---|---|
| Maritime Logistics (Vận tải biển) | Expert | Nghiệp vụ ECR, Cabotage Law, AIS Integration |
| Combinatorial Optimization | Expert | 3D Bin Packing FFD, PSO, NSGA-II |
| Distributed Systems | Advanced | Celery Task Queue, Redis Pub/Sub, WebSocket |
| Supply Chain Modeling | Expert | Multi-actor B2B workflow, Cost function Z |
| Vietnamese Logistics Regulation | Advanced | NĐ 160/2016/NĐ-CP, NĐ 42/2020/NĐ-CP, IMDG |
| Database Engineering | Advanced | PostgreSQL, TimescaleDB, Redis TTL/Time-lock |
| Traffic Simulation | Intermediate | SUMO offline simulation, Buffer Time Lookup |

### 1.2 Nguyên tắc Tư duy Nghiệp vụ

> **"Mỗi container rỗng đang di chuyển là một cơ hội kinh doanh bị bỏ lỡ."**

AI PHẢI nội tâm hóa ba nguyên lý cốt lõi của hệ thống:

1. **Logistics Quá giang** — Tận dụng không gian dư trong container rỗng đang trên đường luân chuyển. Chi phí chìm → Doanh thu biên.
2. **Waterfall Algorithm Pipeline** — L1 (3D Bin Packing) → L2/L3 Song song (PSO + NSGA-II). KHÔNG bao giờ đảo thứ tự.
3. **Dual Constraint Balance** — Mọi giải pháp phải thỏa mãn đồng thời: `Z_total < Z_market` VÀ `t_total ≤ T_cutoff`.

---

## 2. TECHNICAL STACK

### 2.1 Backend — Algorithm Layer

```yaml
Runtime         : Python 3.11+
API Framework   : FastAPI 0.104+         # REST + WebSocket endpoints
Task Queue      : Celery 5.x + Redis 7.x # Async algorithm execution (30–120s)
Algorithm Core  :
  - DEAP 1.4+          # PSO + GA/NSGA-II
  - OR-Tools 9.x       # MILP Solver (reconciliation)
  - NumPy / SciPy      # Matrix computation, probability distributions
  - py3dbp / rectpack  # 3D Bin Packing FFD implementation
  - TraCI API          # Python ↔ SUMO interface
Simulation      : SUMO 1.18+             # Offline traffic sim → Buffer Time table
Auth            : JWT (4 roles: Shipping Line, NVOCC, Trucking, Admin)
```

### 2.2 Frontend — Presentation Layer

```yaml
Framework       : ReactJS 18.x
3D Visualization: Three.js r150+         # Container packing 3D preview
Map & Routing   : Mapbox / Leaflet        # Route map, GPS tracking
Charts          : Chart.js               # KPI dashboards, Pareto frontier
Real-time       : WebSocket / SSE        # Time-lock countdown, price push
State Management: React Context / Redux  # Global booking & matching state
```

### 2.3 Data Layer

```yaml
Primary DB      : PostgreSQL 15+         # 6 core entities (transactional)
Time-series     : TimescaleDB 2.x        # KPI snapshots, hypertable by date
Cache / Broker  : Redis 7.x             # Session, Buffer Time TTL, Time-lock, Pub/Sub
Object Storage  : S3 / MinIO            # PDFs, Stuffing Plans, container images
Schema Entities :
  - Container              # Supply — vỏ rỗng từ Shipping Line
  - BookingRequest         # Demand — LCL cargo từ NVOCC
  - MatchingResult         # Output của Algorithm Pipeline
  - SlotSharingAgreement   # Hợp đồng 3 bên Cabotage-compliant
  - TruckingOrder          # Lệnh điều xe tải + GPS tracking
  - Transaction_Invoice    # Đối soát chi phí planned vs actual
```

### 2.4 Testing Stack

```yaml
Backend Tests   : Pytest 7.x + pytest-asyncio + pytest-cov
Frontend Tests  : Jest 29.x + React Testing Library
Coverage Tool   : pytest-cov (Python) / Istanbul/nyc (JS)
Mocking         : unittest.mock / responses (Python) | jest.mock (JS)
Algorithm Tests : Property-based testing với Hypothesis (Python)
Integration     : TestClient (FastAPI) + Docker Compose test environment
```

---

## 3. CODING & TESTING STANDARDS

### 3.1 Clean Code Principles

AI PHẢI tuân thủ các nguyên tắc Clean Code sau đây ở mọi đoạn code được sinh ra:

#### Naming Conventions
```python
# ✅ ĐÚNG — Tên rõ nghĩa, phản ánh domain
def calculate_container_fill_rate(booking_request: BookingRequest, 
                                   container: Container) -> float:
    """Tính tỷ lệ lấp đầy thể tích container (%)."""
    ...

# ❌ SAI — Tên mơ hồ, không có nghĩa nghiệp vụ
def calc(br, c):
    ...
```

#### Function Design
- Mỗi hàm chỉ làm **một việc duy nhất** (Single Responsibility).
- Độ dài hàm tối đa: **30 dòng** (không tính docstring và type hints).
- Số tham số tối đa: **5 tham số**. Nếu vượt quá → dùng dataclass/pydantic model.
- Tuyệt đối không dùng magic numbers — dùng `constants.py` hoặc `Enum`.

```python
# ✅ ĐÚNG — Domain constants được định nghĩa rõ ràng
class ContainerCapacity:
    TWENTY_FOOT_CBM = 25.0      # m³
    FORTY_FOOT_CBM  = 67.0      # m³
    TWENTY_FOOT_TON = 28.0      # metric tons
    FORTY_FOOT_TON  = 26.0      # metric tons

# ❌ SAI — Magic numbers trong logic
if total_cbm > 25:  # 25 là gì?
    ...
```

#### Error Handling
```python
# ✅ ĐÚNG — Explicit domain exceptions
class ChemicalCompatibilityError(Exception):
    """Raised khi cargo items vi phạm chemical compatibility matrix."""
    def __init__(self, hs_group_a: str, hs_group_b: str):
        self.message = f"Incompatible HS groups: {hs_group_a} + {hs_group_b}"
        super().__init__(self.message)

# ❌ SAI — Generic exception che giấu lỗi domain
raise Exception("Lỗi hóa chất")
```

### 3.2 SOLID Principles — Áp dụng cho ECR-LCL

| Principle | Áp dụng cụ thể trong dự án |
|---|---|
| **S** — Single Responsibility | `BinPackingEngine` chỉ xếp hàng; `ChemicalChecker` chỉ validate HS code |
| **O** — Open/Closed | `BaseOptimizer` → `PSOOptimizer`, `NSGAIIOptimizer` extend không sửa base |
| **L** — Liskov Substitution | `ContainerType.TWENTY_DC` và `FORTY_HC` dùng được ở mọi nơi nhận `ContainerType` |
| **I** — Interface Segregation | `ISupplyRepository` tách khỏi `IDemandRepository`; không merge thành một |
| **D** — Dependency Inversion | `MatchingOrchestrator` nhận `IBinPackingEngine` qua DI, không `import` trực tiếp |

```python
# ✅ ĐÚNG — Dependency Inversion trong Matching Orchestrator
class MatchingOrchestrator:
    def __init__(
        self,
        bin_packing_engine: IBinPackingEngine,
        pso_optimizer: IPSOOptimizer,
        nsga_optimizer: INSGAOptimizer,
        pricing_engine: IDynamicPricingEngine,
    ):
        self._bin_packing = bin_packing_engine
        self._pso = pso_optimizer
        self._nsga = nsga_optimizer
        self._pricing = pricing_engine
```

### 3.3 BẮT BUỘC: Unit Test cho Mỗi Hàm Logic

> ⚠️ **QUY TẮC BẤT BIẾN**: Không có hàm logic nào được merge/deploy nếu thiếu unit test tương ứng.

#### Cấu trúc Test File
```
project/
├── src/
│   ├── algorithms/
│   │   ├── bin_packing.py
│   │   ├── pso_optimizer.py
│   │   └── pricing_engine.py
│   └── validators/
│       └── chemical_checker.py
└── tests/
    ├── unit/
    │   ├── algorithms/
    │   │   ├── test_bin_packing.py      # 1-to-1 mapping với src
    │   │   ├── test_pso_optimizer.py
    │   │   └── test_pricing_engine.py
    │   └── validators/
    │       └── test_chemical_checker.py
    ├── integration/
    │   └── test_matching_pipeline.py
    └── conftest.py                      # Shared fixtures
```

#### Template Unit Test — Pytest (Python)
```python
# tests/unit/algorithms/test_bin_packing.py

import pytest
from src.algorithms.bin_packing import BinPackingEngine, PackingResult
from src.models.container import Container, ContainerType
from src.models.cargo import CargoItem
from tests.factories import ContainerFactory, CargoItemFactory


class TestBinPackingEngine:
    """Unit tests cho 3D Bin Packing FFD Algorithm (Lớp 1 — Matching Engine)."""

    @pytest.fixture
    def engine(self) -> BinPackingEngine:
        return BinPackingEngine()

    @pytest.fixture
    def standard_20ft_container(self) -> Container:
        return ContainerFactory.build(type=ContainerType.TWENTY_DC)

    # --- Happy Path ---

    def test_pack_single_item_returns_feasible(
        self, engine: BinPackingEngine, standard_20ft_container: Container
    ):
        """Một kiện hàng nhỏ duy nhất phải luôn được xếp thành công."""
        items = [CargoItemFactory.build(length=100, width=80, height=60, weight=200)]
        
        result = engine.pack(items, standard_20ft_container)
        
        assert result.status == "FEASIBLE"
        assert len(result.layout) == 1
        assert result.space_utilization_pct > 0

    def test_fill_rate_calculation_is_accurate(
        self, engine: BinPackingEngine, standard_20ft_container: Container
    ):
        """Fill rate phải chính xác đến 2 chữ số thập phân."""
        # 1 kiện hàng chiếm đúng 50% sức chứa container 20ft (25 m³)
        items = [CargoItemFactory.build(
            length=250, width=220, height=182, weight=500  # ≈ 12.5 m³
        )]
        
        result = engine.pack(items, standard_20ft_container)
        
        assert result.status == "FEASIBLE"
        assert abs(result.space_utilization_pct - 50.0) < 1.0  # ±1% tolerance

    # --- Edge Cases ---

    def test_pack_overweight_cargo_returns_infeasible(
        self, engine: BinPackingEngine, standard_20ft_container: Container
    ):
        """Vượt giới hạn trọng tải phải trả về INFEASIBLE với reason code rõ ràng."""
        items = [CargoItemFactory.build(weight=30_000)]  # 30 tấn > 28 tấn giới hạn 20ft
        
        result = engine.pack(items, standard_20ft_container)
        
        assert result.status == "INFEASIBLE"
        assert result.reason_code == "WEIGHT_EXCEEDED"

    def test_pack_cbm_overflow_returns_infeasible(
        self, engine: BinPackingEngine, standard_20ft_container: Container
    ):
        """CBM vượt sức chứa container phải trả về INFEASIBLE."""
        # 30 m³ hàng > 25 m³ sức chứa 20ft
        items = [CargoItemFactory.build(length=300, width=220, height=227, weight=100)]
        
        result = engine.pack(items, standard_20ft_container)
        
        assert result.status == "INFEASIBLE"
        assert result.reason_code == "CBM_EXCEEDED"

    # --- Chemical Compatibility ---

    def test_imdg_cargo_with_food_returns_infeasible(
        self, engine: BinPackingEngine, standard_20ft_container: Container
    ):
        """IMDG hàng nguy hiểm KHÔNG được đóng chung với thực phẩm (HS 01–24)."""
        items = [
            CargoItemFactory.build(hs_code="28151100", weight=500),  # NaOH — IMDG Class 8
            CargoItemFactory.build(hs_code="09011100", weight=200),  # Cà phê — HS 09
        ]
        
        result = engine.pack(items, standard_20ft_container)
        
        assert result.status == "INFEASIBLE"
        assert result.reason_code == "CHEMICAL_INCOMPATIBLE"

    # --- Sorting Logic (FFD Core) ---

    def test_largest_item_placed_first(
        self, engine: BinPackingEngine, standard_20ft_container: Container
    ):
        """FFD algorithm phải xếp kiện hàng lớn nhất vào vị trí đầu tiên."""
        small_item = CargoItemFactory.build(length=50, width=50, height=50)
        large_item = CargoItemFactory.build(length=200, width=150, height=100)
        items = [small_item, large_item]  # Thứ tự ngược — nhỏ trước
        
        result = engine.pack(items, standard_20ft_container)
        
        assert result.status == "FEASIBLE"
        # Kiện lớn nhất phải được đặt trước trong layout
        assert result.layout[0]["item_id"] == large_item.id
```

#### Template Unit Test — Jest (React/TypeScript)
```typescript
// __tests__/components/DynamicPricingBoard.test.tsx

import { render, screen, waitFor } from '@testing-library/react';
import { DynamicPricingBoard } from '@/components/DynamicPricingBoard';
import { mockMatchingResult } from '../fixtures/matching.fixtures';

describe('DynamicPricingBoard', () => {
  describe('Price Display Logic', () => {
    it('should display discount rate when fill rate < 40%', async () => {
      const lowFillResult = { ...mockMatchingResult, space_utilization_pct: 35 };
      
      render(<DynamicPricingBoard matchingResult={lowFillResult} />);
      
      await waitFor(() => {
        expect(screen.getByTestId('discount-badge')).toHaveTextContent('40%');
        expect(screen.getByTestId('price-label')).toHaveClass('text-green-600');
      });
    });

    it('should show premium surcharge when days_to_etd < 1', async () => {
      const urgentResult = { 
        ...mockMatchingResult, 
        space_utilization_pct: 85,
        days_to_etd: 0.5 
      };
      
      render(<DynamicPricingBoard matchingResult={urgentResult} />);
      
      await waitFor(() => {
        expect(screen.getByTestId('surcharge-badge')).toHaveTextContent('+10%');
      });
    });
  });
});
```

### 3.4 Coverage Requirements

```ini
# pytest.ini hoặc pyproject.toml
[tool.pytest.ini_options]
addopts = "--cov=src --cov-report=term-missing --cov-fail-under=80"

# .coveragerc
[report]
exclude_lines =
    pragma: no cover
    if __name__ == .__main__.:
    raise NotImplementedError
    @abstractmethod
```

| Module | Coverage Target | Lý do |
|---|---|---|
| `algorithms/bin_packing.py` | **≥ 90%** | Lõi L1 — ảnh hưởng trực tiếp đến toàn bộ pipeline |
| `algorithms/pso_optimizer.py` | **≥ 85%** | Lõi L2 — critical path T_cutoff |
| `algorithms/pricing_engine.py` | **≥ 90%** | Dynamic pricing — trực tiếp tác động tài chính |
| `validators/chemical_checker.py` | **≥ 95%** | Pháp lý / An toàn — zero-tolerance |
| `validators/cabotage_validator.py` | **≥ 95%** | Pháp lý NĐ 160 — zero-tolerance |
| `api/matching_routes.py` | **≥ 80%** | API layer |
| `components/` (React) | **≥ 75%** | Frontend logic |
| **Overall** | **≥ 80%** | Minimum acceptable |

---

## 4. SELF-CORRECTION SKILL

> AI PHẢI thực hiện vòng lặp tự kiểm tra và sửa lỗi trước khi báo cáo kết quả. Không bao giờ báo cáo "Hoàn thành" khi test chưa pass.

### 4.1 Vòng lặp Self-Correction

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SELF-CORRECTION LOOP                             │
│                                                                     │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────────┐  │
│  │  Write   │───►│   Run    │───►│  Tests   │───►│   Report     │  │
│  │  Code +  │    │  Tests   │    │  PASS?   │    │  COMPLETED   │  │
│  │  Tests   │    │          │    └────┬─────┘    └──────────────┘  │
│  └──────────┘    └──────────┘         │ NO                         │
│                                       ▼                             │
│                              ┌──────────────────┐                  │
│                              │  Analyze Failure │                  │
│                              │  Root Cause      │                  │
│                              └────────┬─────────┘                  │
│                                       │                             │
│                              ┌────────▼─────────┐                  │
│                              │  Fix Code (NOT   │                  │
│                              │  Fix Test*)      │◄─── Retry ≤ 3x  │
│                              └────────┬─────────┘                  │
│                                       │                             │
│                              ┌────────▼─────────┐                  │
│                              │  Re-run Tests    │                  │
│                              └──────────────────┘                  │
└─────────────────────────────────────────────────────────────────────┘
* Chỉ sửa test nếu test bị sai logic nghiệp vụ, có giải thích rõ ràng.
```

### 4.2 Quy trình Tự kiểm tra Chi tiết

```bash
# Bước 1: Chạy unit tests
pytest tests/unit/ -v --tb=short

# Bước 2: Kiểm tra coverage
pytest tests/unit/ --cov=src --cov-report=term-missing --cov-fail-under=80

# Bước 3: Nếu có fail → phân tích
# AI phải đọc traceback, xác định:
# - Dòng lỗi cụ thể
# - Expected vs Actual value
# - Root cause (logic sai, edge case bỏ sót, type mismatch...)

# Bước 4: Fix code, re-run
# Tối đa 3 vòng tự sửa. Sau 3 vòng mà vẫn fail → escalate với explanation.

# Bước 5: Chạy integration tests sau khi unit pass
pytest tests/integration/ -v

# Bước 6: Báo cáo kết quả theo format chuẩn
```

### 4.3 Format Báo cáo Kết quả

```markdown
## ✅ Task Completion Report

**Module**: `algorithms/bin_packing.py`
**Function(s)**: `BinPackingEngine.pack()`, `_check_chemical_compatibility()`

### Test Results
| Suite       | Total | Passed | Failed | Coverage |
|-------------|-------|--------|--------|----------|
| Unit Tests  | 24    | 24     | 0      | 91.3%    |
| Integration | 6     | 6      | 0      | —        |

### Self-Correction Log
- Iteration 1: `test_pack_overweight_cargo_returns_infeasible` FAILED
  - Root cause: Weight check used `>=` instead of `>`, borderline case at exactly 28,000 kg
  - Fix: Changed condition to `> ContainerCapacity.TWENTY_FOOT_TON * 1000`
- Iteration 2: All 24 tests PASSED ✅

### Coverage Report (relevant modules)
```
src/algorithms/bin_packing.py    91%   Missing: lines 78-82 (unreachable reefer path)
src/validators/chemical_checker.py  96%
```

**Status**: COMPLETED — Ready for review.
```

---

## 5. WORKFLOW RULE — BẮT BUỘC TUÂN THỦ

> **Không được phép bỏ qua hoặc đảo thứ tự bất kỳ bước nào trong workflow.**

### 5.1 Master Workflow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         ECR-LCL DEVELOPMENT WORKFLOW                        │
│                                                                             │
│  PHASE 1          PHASE 2         PHASE 3        PHASE 4       PHASE 5     │
│                                                                             │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐   ┌──────────┐  ┌─────────┐ │
│  │ PHÂN TÍCH│    │ VIẾT     │    │ VIẾT     │   │ CHẠY     │  │ TỐI ƯU │ │
│  │   SPEC   │───►│  TEST    │───►│  CODE    │──►│  TEST    │─►│        │ │
│  │          │    │ TRƯỚC    │    │          │   │ & FIX    │  │        │ │
│  └──────────┘    └──────────┘    └──────────┘   └──────────┘  └─────────┘ │
│                                                                             │
│   Đọc spec      Viết test        Implement       pytest /     Refactor &   │
│   Xác định      theo spec        function        jest         optimize     │
│   input/output  (test FAILS      (make test      Self-corr.  Profile       │
│   constraints   initially)       GREEN)          loop        Coverage≥80% │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 5.2 Chi tiết Từng Phase

#### Phase 1 — Phân tích Spec

Trước khi viết bất kỳ dòng code nào, AI PHẢI trả lời đầy đủ:

```markdown
## Spec Analysis Checklist

### Functional Requirements
- [ ] Input: Kiểu dữ liệu, validation rules, constraints?
- [ ] Output: Schema, format, error cases?
- [ ] Side effects: DB writes, cache updates, events published?

### Algorithm / Business Logic
- [ ] Công thức toán học / pseudocode có trong spec?
- [ ] Constraints cứng vs mềm vs pháp lý?
- [ ] Thứ tự phụ thuộc giữa các modules?

### ECR-LCL Domain Checklist
- [ ] Có liên quan đến T_cutoff constraint không?
- [ ] Có xử lý HS Code / IMDG không?
- [ ] Có yêu cầu Cabotage compliance (vessel_flag = VN) không?
- [ ] Có ghi vào TimescaleDB (KPI) không?
- [ ] Có cập nhật Redis cache không?
```

#### Phase 2 — Viết Test Trước (TDD)

```python
# Pattern: Arrange → Act → Assert
# Test phải FAIL trước khi viết code (Red → Green → Refactor)

def test_dynamic_pricing_applies_seasonal_factor_november():
    """
    SPEC REF: Section 3.4 — Seasonal factor tháng 11–12 (peak FDI) = × 1.15
    """
    # ARRANGE
    engine = DynamicPricingEngine()
    base_params = PricingParams(
        floor_price=Decimal("5_000_000"),
        space_util_pct=70.0,
        days_to_etd=8,
        booking_month=11,  # Tháng 11 — peak FDI season
    )
    
    # ACT
    result = engine.calculate(base_params)
    
    # ASSERT
    expected_discount = 0.10  # 70% util, >7 days → 10% discount
    expected_seasonal = 1.15   # Tháng 11
    expected_price = Decimal("5_000_000") * Decimal("0.90") * Decimal("1.15")
    assert result.final_price == expected_price
    assert result.seasonal_factor == Decimal("1.15")
```

#### Phase 3 — Viết Code

```python
# Code được viết để làm test GREEN — không thêm logic thừa
# Mỗi hàm có docstring theo format Google Style

def calculate_dynamic_price(params: PricingParams) -> PricingResult:
    """
    Tính giá cước động theo Fill Rate và thời gian còn đến ETD.
    
    Công thức: C_sea_final = C_sea_floor × (1 − discount_rate) × seasonal_factor
    
    Args:
        params: PricingParams chứa floor_price, space_util_pct, 
                days_to_etd, booking_month.
    
    Returns:
        PricingResult với final_price, discount_rate, seasonal_factor.
    
    Raises:
        InvalidPricingParamsError: Nếu floor_price <= 0 hoặc util_pct ngoài [0, 100].
    
    Spec Reference:
        Section 3.4 — Dynamic Pricing Engine.
    """
    ...
```

#### Phase 4 — Chạy Test & Self-Correction

Xem Section 4 — Self-Correction Skill.

#### Phase 5 — Tối ưu

```python
# Chỉ tối ưu SAU khi tests đã PASS và coverage ≥ 80%
# Tối ưu theo thứ tự ưu tiên:

# Priority 1: Correctness (đã đảm bảo qua test)
# Priority 2: Algorithm complexity (O(n log n) vs O(n²))
# Priority 3: Memory usage (đặc biệt trong 3D BinPacking với n items lớn)
# Priority 4: Database query optimization (N+1 queries)
# Priority 5: Cache strategy (Redis TTL cho Buffer Time)

# Sau tối ưu: Re-run full test suite để đảm bảo không regression
```

---

## 6. DOMAIN-SPECIFIC RULES

### 6.1 Algorithm Pipeline Rules

```
RULE ALG-01: L1 (3D Bin Packing) PHẢI chạy TRƯỚC L2 và L3.
             Không bao giờ gọi PSO khi L1 trả về INFEASIBLE.

RULE ALG-02: L2 (PSO) và L3 (NSGA-II) chạy SONG SONG qua Celery.
             Dùng Celery group/chord, không dùng sequential chain.

RULE ALG-03: SUMO chạy OFFLINE để tạo Buffer Time Lookup Table.
             TUYỆT ĐỐI không gọi SUMO realtime trong PSO particle evaluation.
             PSO chỉ tra bảng từ Redis cache.

RULE ALG-04: Batch matching trigger mỗi 15 phút HOẶC manual.
             Không trigger on-demand theo từng booking request.

RULE ALG-05: Dynamic Pricing cập nhật mỗi 15 phút qua WebSocket push.
             Không tính lại mỗi lần user load trang.
```

### 6.2 Constraint Enforcement Rules

```
RULE CON-01 [CRITICAL]: vessel_flag PHẢI = "VN" trước khi tạo
             SlotSharingAgreement. Block hard nếu vi phạm NĐ 160/2016.

RULE CON-02 [CRITICAL]: IMDG cargo tại cảng Cát Lái bị BLOCK hoàn toàn
             từ 01/07/2024. Enforce tại booking creation, không phải matching.

RULE CON-03 [CRITICAL]: t_total ≤ T_cutoff là hard constraint.
             C_penalty function là exponential khi t_total tiệm cận T_cutoff.

RULE CON-04: Z_total < Z_market là soft constraint (profitability gate).
             Hiển thị warning cho NVOCC, không block match.

RULE CON-05: Redis Time-lock TTL = 1800 giây (30 phút) sau khi cả 2 bên
             nhận notification. Auto-expire = auto-reject.
```

### 6.3 Data Integrity Rules

```sql
-- RULE DB-01: Container mã phải valid theo ISO 6346
-- Implement check digit validation trong Python, không chỉ regex

-- RULE DB-02: total_cbm và total_weight_kg là GENERATED columns
-- Không cho phép insert/update trực tiếp

-- RULE DB-03: closing_time (T_cutoff) phải trước ETD từ 8-24 giờ
-- Enforce ở cả API layer VÀ DB constraint

-- RULE DB-04: KPI data ghi vào TimescaleDB, không phải PostgreSQL
-- Mọi INSERT vào bảng KPI phải dùng TimescaleDB hypertable partition
```

---

## 7. ANTI-PATTERNS — TUYỆT ĐỐI TRÁNH

| # | Anti-pattern | Lý do cấm | Alternative |
|---|---|---|---|
| 1 | Gọi SUMO realtime trong PSO loop | Performance — PSO có thể có 200+ iterations × 30 particles = 6000+ calls | Tra Redis Buffer Time cache |
| 2 | Block HTTP request khi chạy algorithm | Algorithm chạy 30–120s, sẽ timeout | Celery async task + WebSocket progress |
| 3 | Hardcode `vessel_flag` bypass | Vi phạm NĐ 160/2016 — vô hiệu toàn bộ giao dịch | Luôn validate từ DB |
| 4 | `except Exception: pass` trong algorithm | Che giấu lỗi thuật toán → kết quả sai im lặng | Raise specific `AlgorithmError` với context |
| 5 | Viết code trước, test sau | Dễ bỏ sót edge cases quan trọng | Luôn theo TDD — test fail trước |
| 6 | Sử dụng `float` cho tiền VNĐ | Floating point error → sai lệch thanh toán | Dùng `Decimal` hoặc integer (đồng) |
| 7 | Query N+1 cho cargo_items | `cargo_items` là JSONB array → 1 query là đủ | `SELECT *, jsonb_array_length(cargo_items)` |
| 8 | Push 3D layout JSON qua WebSocket | Layout JSON có thể > 1MB | Lưu DB, trả `layout_id`, client fetch riêng |

---

## 8. GLOSSARY — THUẬT NGỮ DOMAIN

| Thuật ngữ | Định nghĩa |
|---|---|
| **ECR** | Empty Container Repositioning — Luân chuyển container rỗng |
| **LCL** | Less-than-Container-Load — Hàng lẻ không đủ một container |
| **NVOCC** | Non-Vessel Operating Common Carrier — Đơn vị gom hàng lẻ |
| **T_cutoff** | Thời hạn hạ bãi cuối cùng — thường = ETD − 12h |
| **ETD** | Estimated Time of Departure — Thời gian tàu chạy dự kiến |
| **FFD** | First Fit Decreasing — Heuristic xếp hàng từ lớn đến nhỏ |
| **PSO** | Particle Swarm Optimization — Tối ưu bầy đàn cho routing |
| **NSGA-II** | Non-dominated Sorting Genetic Algorithm II — GA đa mục tiêu |
| **Pareto Frontier** | Tập nghiệm không thể cải thiện một mục tiêu mà không làm xấu mục tiêu khác |
| **Z_total** | Hàm chi phí tổng: `C_sea + C_road + C_penalty` |
| **Z_market** | Giá cước thị trường thông thường — ngưỡng so sánh |
| **Cabotage** | Vận tải nội địa — NĐ 160/2016 yêu cầu tàu cờ Việt Nam |
| **IMDG** | International Maritime Dangerous Goods — Hàng nguy hiểm hàng hải |
| **CFS** | Container Freight Station — Kho gom hàng lẻ |
| **AIS** | Automatic Identification System — Hệ thống nhận dạng tàu tự động |
| **Fill Rate** | Tỷ lệ lấp đầy thể tích container (%) |
| **Buffer Time** | Thời gian dự phòng cho kẹt xe, tính từ SUMO simulation |
| **TCNU Format** | Chuẩn mã container: 4 chữ ISO + 6 số + 1 check digit (ISO 6346) |

---

*Tài liệu này là nguồn tham chiếu duy nhất và bất biến cho AI Agent trong suốt vòng đời phát triển dự án ECR-LCL Platform. Mọi quyết định kỹ thuật phải nhất quán với Skill Definition này.*

---
**Phiên bản**: `v1.0`  
**Căn cứ**: ECR-LCL Final System Spec v3.0  
**Ngôn ngữ mã nguồn chính**: Python 3.11+ / TypeScript 5.x  
**Cập nhật lần cuối**: 2026-03-10
