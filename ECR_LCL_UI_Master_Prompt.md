# 🎨 ECR-LCL PLATFORM — MASTER UI/UX PROMPT
> **Tài liệu Prompt Thiết kế Giao diện Tối ưu**
> Dự án: *B2B Logistics Quá giang — Empty Container × LCL Platform*
> Tuyến: Cát Lái (TP.HCM) → Hải Phòng | Phiên bản: `v2.0`

---

## 🧭 DESIGN SYSTEM FOUNDATION

### Triết lý Thiết kế
> *"Logistics không cần đẹp — nhưng một nền tảng B2B đẳng cấp phải khiến người dùng chuyên nghiệp thấy mình đang dùng công cụ của tương lai."*

Thiết kế theo hướng **Industrial Precision** — giao thoa giữa Bloomberg Terminal (data-dense, authoritative) và modern SaaS (clean, actionable). Tối tăm, sắc nét, đáng tin cậy.

---

### Design Tokens (CSS Variables)

```css
:root {
  /* === COLOR PALETTE === */
  /* Background hierarchy */
  --bg-base:        #0A0E1A;   /* Màu nền cơ bản — xanh đen đậm */
  --bg-surface:     #111827;   /* Card / Panel */
  --bg-elevated:    #1C2333;   /* Dropdown / Modal / Popover */
  --bg-overlay:     #242D3D;   /* Hover states */

  /* Brand & Accent */
  --accent-primary: #00D4FF;   /* Cyan điện — interactive elements, links */
  --accent-secondary:#F59E0B;  /* Amber — warnings, countdowns, alerts VÀNG */
  --accent-success: #10B981;   /* Emerald — confirmed, feasible, on-time */
  --accent-danger:  #EF4444;   /* Red — blocked, infeasible, violation */
  --accent-purple:  #8B5CF6;   /* Violet — algorithm output, AI-generated */

  /* Text hierarchy */
  --text-primary:   #F1F5F9;   /* Heading, critical data */
  --text-secondary: #94A3B8;   /* Labels, metadata */
  --text-muted:     #475569;   /* Placeholder, disabled */
  --text-inverse:   #0A0E1A;   /* Text on light backgrounds */

  /* Status colors */
  --status-available: #10B981;
  --status-locked:    #F59E0B;
  --status-confirmed: #00D4FF;
  --status-on-vessel: #8B5CF6;
  --status-returned:  #475569;

  /* === TYPOGRAPHY === */
  --font-display:  'Syne', sans-serif;       /* Headers, KPI numbers — geometric, strong */
  --font-body:     'IBM Plex Sans', sans-serif; /* Body text — technical, readable */
  --font-mono:     'JetBrains Mono', monospace; /* Code, IDs, hashes, GPS coords */
  --font-numeric:  'Syne', sans-serif;       /* Large numbers — tabular figures */

  /* === SPACING === */
  --space-xs:  4px;
  --space-sm:  8px;
  --space-md:  16px;
  --space-lg:  24px;
  --space-xl:  32px;
  --space-2xl: 48px;
  --space-3xl: 64px;

  /* === BORDERS === */
  --border-subtle:  1px solid rgba(255,255,255,0.06);
  --border-default: 1px solid rgba(255,255,255,0.12);
  --border-strong:  1px solid rgba(0,212,255,0.3);
  --border-danger:  1px solid rgba(239,68,68,0.5);
  --radius-sm:  6px;
  --radius-md:  10px;
  --radius-lg:  16px;
  --radius-xl:  24px;

  /* === SHADOWS === */
  --shadow-card:   0 4px 24px rgba(0,0,0,0.4);
  --shadow-modal:  0 24px 64px rgba(0,0,0,0.7);
  --shadow-glow:   0 0 24px rgba(0,212,255,0.15);
  --shadow-danger: 0 0 24px rgba(239,68,68,0.2);

  /* === ANIMATION === */
  --ease-snap:   cubic-bezier(0.4, 0, 0.2, 1);
  --ease-spring: cubic-bezier(0.34, 1.56, 0.64, 1);
  --dur-fast:    150ms;
  --dur-normal:  250ms;
  --dur-slow:    400ms;
}
```

---

### Component Library Chuẩn

```
Shared Components (dùng xuyên suốt toàn bộ roles):

┌─ StatusBadge       → AVAILABLE|LOCKED|CONFIRMED|ON_VESSEL|RETURNED
├─ CountdownTimer    → Redis TTL 30 phút — dạng MM:SS, màu đỏ khi < 5 phút
├─ KPIScorecard      → Icon + Label + Value lớn + Delta % + sparkline
├─ DataTable         → Sort, filter, pagination, row selection, export CSV
├─ ContainerTypeTag  → 20DC | 40DC | 40HC | 20RF | 40RF với màu phân biệt
├─ CurrencyDisplay   → VNĐ format với font tabular (Syne Mono)
├─ GPSCoord          → lat/lng format monospace với copy-to-clipboard
├─ ProgressBar       → CBM fill rate với gradient cyan→amber→red
├─ AlertBanner       → INFO|WARNING|DANGER với icon và dismiss
├─ SplitView         → Resizable split layout (drag divider)
├─ WebSocketStatus   → Indicator kết nối WS (xanh nhấp nháy = live)
└─ RoleAvatar        → Avatar có badge role (SL|NVOCC|TR|AD)
```

---

### Navigation Architecture

```
Sidebar Navigation (64px collapsed / 240px expanded):
├─ Logo ECR-LCL (animate trên hover)
├─ Role indicator badge (màu theo role)
├─ Navigation items (icon + label)
├─ WebSocket live indicator (góc dưới)
└─ User avatar + quick logout

Top Bar (56px fixed):
├─ Breadcrumb động
├─ Global search (Cmd+K)
├─ Notification bell (badge count, WebSocket push)
├─ Role switcher (nếu multi-role)
└─ Countdown Timer (xuất hiện khi có giao dịch đang chờ xác nhận)
```

---

### Responsive Breakpoints

```
Desktop  : ≥ 1280px — Full layout, sidebar expanded
Laptop   : 1024–1279px — Sidebar collapsed (icon only)
Tablet   : 768–1023px — Sidebar overlay
Mobile   : < 768px — Bottom nav (chỉ cho TR-04, TR-05, TR-06)
```

---

---

## 🌐 PHẦN 1: GIAO DIỆN CHUNG & XÁC THỰC (4 MÀN HÌNH)

---

### [COM-01] — Màn hình Login

**Purpose:** Kiểm soát truy cập, xác thực JWT, định tuyến đến 4 roles khác nhau.

**Layout:**
```
┌────────────────────────────────────────────────────────────────────┐
│  LEFT PANEL (55%)                  │  RIGHT PANEL (45%)            │
│  ─────────────────────────────────  ─────────────────────────────  │
│  Hero Visual:                       Form Container:                 │
│  • Animated SVG map illustration    • Logo + Tagline                │
│    tuyến Cát Lái → Hải Phòng        • "Email / Domain" input        │
│  • Animated dashed route line       • "Password" input + toggle     │
│  • 3 moving ship icons (looping)    • "Forgot Password" link        │
│  • Port markers với pulse effect    • [ĐĂNG NHẬP] button (full-w)  │
│  • Overlay text:                    • Divider "──── hoặc ────"      │
│    "Logistics Quá giang"            • [ĐĂNG KÝ DOANH NGHIỆP]       │
│    "Cát Lái × Hải Phòng"           • Footer: Version + Status page  │
└────────────────────────────────────────────────────────────────────┘
```

**Data & State:**
```
POST /auth/login
  → body: { email, password }
  → response: {
      jwt_token: string,
      user: { id, name, role: "SHIPPING_LINE"|"NVOCC"|"TRUCKING"|"ADMIN" },
      organization: { id, name, type }
    }

Routing logic sau login:
  SHIPPING_LINE → /sl/dashboard
  NVOCC         → /nv/dashboard
  TRUCKING      → /tr/dispatch
  ADMIN         → /ad/dashboard
```

**UX Requirements:**
- Error state: Banner đỏ "Sai thông tin đăng nhập" — shake animation
- Loading state: Button spinner + disabled
- Auto-fill support cho password managers
- "Remember me" checkbox → JWT refresh token 30 ngày
- Bên trái: particle effect nhẹ trên nền, route line animate theo interval 4s

**Accessibility:** `aria-live` cho error messages, keyboard navigation đầy đủ

---

### [COM-02] — B2B Onboarding (Đăng ký Doanh nghiệp)

**Purpose:** Thu thập thông tin tổ chức, tạo tài khoản B2B, chờ Admin duyệt.

**Layout:**
```
┌─────────────────────────────────────────────────────────────────┐
│  STEPPER HEADER                                                  │
│  ① Loại tổ chức → ② Thông tin cơ bản → ③ Xác minh → ④ Hoàn thành│
│  [━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━░░░░░░░░░░░░░░░░░░░░░░]  │
├─────────────────────────────────────────────────────────────────┤
│  STEP CONTENT (centered, max-width 640px)                        │
│  Step 1: Role Selection Cards (4 cards ngang)                    │
│    ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐          │
│    │ 🚢 Hãng  │ │ 📦 NVOCC │ │ 🚚 Truck │ │ ⛴ Feeder│          │
│    │  tàu     │ │  Consol  │ │  Company │ │ Operator │          │
│    └──────────┘ └──────────┘ └──────────┘ └──────────┘          │
│  Step 2: Form fields (animated slide-in)                         │
│    • Tên tổ chức (required)                                      │
│    • Mã số thuế (MST - 10 digits Vietnam)                        │
│    • Địa chỉ đăng ký kinh doanh                                  │
│    • Số điện thoại / Email liên hệ chính                         │
│    • Upload Giấy phép kinh doanh (PDF/JPG ≤ 10MB)               │
│  Step 3: Verify email + upload documents                         │
│  Step 4: Success screen + "Chờ Admin xét duyệt (1-2 ngày)"      │
├─────────────────────────────────────────────────────────────────┤
│  [QUAY LẠI]                              [TIẾP THEO →]           │
└─────────────────────────────────────────────────────────────────┘
```

**Data:**
```
POST /auth/register
  → body: {
      organization_name, tax_code, role_type,
      contact_email, contact_phone, address,
      business_license_url  ← S3/MinIO pre-signed upload
    }
  → response: { application_id, status: "PENDING_APPROVAL" }
```

**UX Requirements:**
- Inline validation realtime cho MST (10 số, không dấu)
- File upload: drag-and-drop zone với preview thumbnail
- Step persistence: localStorage lưu progress nếu thoát ngang
- Step 4: Animation confetti nhẹ + countdown "Thường mất 1-2 ngày làm việc"

---

### [COM-03] — Notification Center

**Purpose:** Nhận push notification realtime về matching mới, cảnh báo T_cutoff, xác nhận giao dịch.

**Layout:**
```
Side Drawer (trượt từ phải, width 420px):
┌────────────────────────────────────────────────┐
│ 🔔 Thông báo            [Đọc tất cả] [✕ Đóng]  │
│ ──────────────────────────────────────────────  │
│ FILTER: [Tất cả ▼] [Matching] [Cảnh báo] [Hệ thống]│
│ ══════════════════════════════════════════════  │
│ [●] Mới — 2 phút trước                 URGENT  │
│ 🔴 XE TẢI BIỂN SỐ 51C-123.45 CÓ NGUY CƠ       │
│    TRỄ T_CUTOFF — ETA còn 45 phút, cutoff     │
│    lúc 14:00. [Xem bản đồ →]                  │
│ ──────────────────────────────────────────────  │
│ [●] Mới — 15 phút trước               MATCH   │
│ 🟢 MATCHING MỚI: TCKU123456 × Booking #4521    │
│    Fill Rate 87% · Tiết kiệm 23% so TT        │
│    [Xem chi tiết →]    ⏱ Còn 28:45 để xác nhận│
│ ──────────────────────────────────────────────  │
│ Đã đọc — Hôm qua                               │
│ 🔵 Giao dịch #3841 đã hoàn thành               │
│    Invoice INV-202603-00421 đã phát            │
│ ──────────────────────────────────────────────  │
│ [Xem thêm thông báo cũ]                        │
└────────────────────────────────────────────────┘
```

**Data:**
```
WebSocket: ws://api/notifications/stream
  → Event types:
      NEW_MATCH_PROPOSED  → amber pulse on bell icon
      T_CUTOFF_WARNING    → red pulse + vibrate (mobile)
      MATCH_CONFIRMED     → green notification
      MATCH_EXPIRED       → grey notification
      GATE_IN_COMPLETED   → blue notification
      INVOICE_ISSUED      → blue notification

Notification schema:
{
  id, type, title, body, severity: "INFO"|"WARNING"|"URGENT",
  action_url, related_id, created_at, is_read
}
```

**UX Requirements:**
- Bell icon trên TopBar: badge số đỏ (số notification chưa đọc)
- Notification URGENT: pulse đỏ + âm thanh ping (user-configurable)
- Mark as read khi hover > 2 giây hoặc click
- Group theo ngày: "Hôm nay", "Hôm qua", "7 ngày trước"
- Infinite scroll (load 20 items/batch)

---

### [COM-04] — Global Settings

**Purpose:** Cấu hình tài khoản, bảo mật, API key cho tích hợp bên thứ 3.

**Layout:**
```
┌──────────────┬──────────────────────────────────────────────────┐
│ TABS         │ TAB CONTENT                                       │
│ ──────────── │ ─────────────────────────────────────────────── │
│ 👤 Hồ sơ    │ Tab: Hồ sơ Cá nhân & Tổ chức                     │
│ 🔒 Bảo mật  │   • Avatar upload (crop tool)                     │
│ 🔑 API Keys  │   • Tên hiển thị, Email, Phone                   │
│ 🔔 Thông báo │   • Thông tin tổ chức (readonly cho non-admin)   │
│ 🌐 Ngôn ngữ  │   • [LƯU THAY ĐỔI] button                       │
│              │                                                   │
│              │ Tab: Bảo mật                                      │
│              │   • Đổi mật khẩu (current + new + confirm)       │
│              │   • 2FA toggle (TOTP - Google Authenticator)      │
│              │   • Active sessions list + [Đăng xuất tất cả]    │
│              │   • Login history (last 10 events)                │
│              │                                                   │
│              │ Tab: API Keys (chỉ Admin & Shipping Line)         │
│              │   • [+ TẠO API KEY MỚI] button                   │
│              │   • Table: Key prefix, Created, Last used,        │
│              │     Permissions, [Revoke]                         │
│              │   • Copy key popup (shown once on create)         │
└──────────────┴──────────────────────────────────────────────────┘
```

---

---

## 🚢 PHẦN 2: ROLE — HÃNG TÀU / SHIPPING LINE (7 MÀN HÌNH)

**Role color accent:** `#00D4FF` (Cyan)
**Sidebar icon:** 🚢

---

### [SL-01] — Shipping Line Dashboard

**Purpose:** Tổng quan hiệu quả khai thác container rỗng, doanh thu biên, KPI vận hành.

**Layout:**
```
┌─────────────────────────────────────────────────────────────────┐
│ TOP BAR: "Tổng quan — Hãng tàu"   [Khoảng thời gian ▼] [Export]│
├──────┬──────┬──────┬──────┬──────┬──────┬──────┬───────────────┤
│SCORE │SCORE │SCORE │SCORE │SCORE │SCORE │SCORE │  SCORE CARD   │
│CARD  │CARD  │CARD  │CARD  │CARD  │CARD  │CARD  │  (8 cards     │
│ #1   │ #2   │ #3   │ #4   │ #5   │ #6   │ #7   │   responsive  │
│      │      │      │      │      │      │      │   grid)       │
├──────┴──────┴──────┴──────┴──────┴──────┴──────┴───────────────┤
│ ROW 2: CHARTS                                                    │
│ ┌──────────────────────────┐  ┌────────────────────────────────┐│
│ │ Doanh thu biên theo ngày │  │ Container Status Distribution  ││
│ │ (Line Chart - Chart.js)  │  │ (Donut Chart)                  ││
│ │ C_sea_actual vs planned  │  │ AVAILABLE/LOCKED/ON_VESSEL/DONE││
│ └──────────────────────────┘  └────────────────────────────────┘│
├──────────────────────────────────────────────────────────────────┤
│ ROW 3:                                                           │
│ ┌────────────────────────────┐ ┌───────────────────────────────┐│
│ │ Fill Rate Heatmap (tuần)   │ │ Recent Matching Inbox          ││
│ │ X: ngày, Y: container type │ │ (3 giao dịch gần nhất)         ││
│ │ color: fill rate %         │ │ với badge status + CTA         ││
│ └────────────────────────────┘ └───────────────────────────────┘│
└──────────────────────────────────────────────────────────────────┘
```

**Scorecard Metrics (8 cards):**
```
┌─────────────────────────────────────────────────────────────────────────┐
│ #1 Tổng vỏ rỗng   │ #2 Đang khớp lệnh │ #3 Fill Rate TB   │ #4 Rev biên │
│ [N] units          │ [M] containers     │ [87.3]%           │ [₫2.4T]     │
│ icon: Container    │ icon: Sync         │ icon: Package     │ icon: Chart↑│
│ delta: +12 vs 7d   │ LOCKED status      │ vs target 80%     │ MTD         │
├────────────────────┼────────────────────┼───────────────────┼─────────────┤
│ #5 Giao dịch HT   │ #6 T_cutoff rate   │ #7 CO₂ Reduction  │ #8 Avg Z vs │
│ [241] this month   │ [96.2]%            │ [1,847] kg        │ Z_market    │
│ icon: CheckCircle  │ on-time gate-in    │ icon: Leaf        │ [-24.1]%    │
└─────────────────────────────────────────────────────────────────────────┘
```

**Data:**
```
GET /api/sl/dashboard?period=7d|30d|90d
→ {
    kpis: { total_containers, matched_count, avg_fill_rate,
            marginal_revenue_vnd, completed_transactions,
            cutoff_compliance_rate, co2_reduction_kg, avg_z_savings_pct },
    revenue_chart: [{ date, planned_vnd, actual_vnd }],
    status_distribution: { available, locked, on_vessel, returned },
    fill_rate_heatmap: [{ date, container_type, fill_rate }],
    recent_matches: [{ match_id, booking_id, status, proposed_at }]
  }
```

---

### [SL-02] — Supply Management List

**Purpose:** Quản lý toàn bộ kho container rỗng đang đăng ký — lọc, tìm kiếm, theo dõi trạng thái.

**Layout:**
```
┌─────────────────────────────────────────────────────────────────┐
│ HEADER: "Quản lý Vỏ rỗng"              [+ ĐĂNG KÝ VỎ RỖNG]    │
├─────────────────────────────────────────────────────────────────┤
│ FILTER BAR:                                                      │
│ [🔍 Tìm mã container...]  [Loại ▼] [Status ▼] [ETD Range ▼]    │
│ [Depot location ▼]        Active filters: ✕ 20DC  ✕ AVAILABLE   │
├─────────────────────────────────────────────────────────────────┤
│ DATA TABLE:                                                      │
│ ☐ │ Container ID  │ Loại  │ Depot        │ ETD         │ T_cutoff    │ Giá sàn    │ Status     │ Còn lại │ Actions    │
│ ──┼───────────────┼───────┼──────────────┼─────────────┼─────────────┼────────────┼────────────┼─────────┼────────────│
│ ☐ │ TCKU123456 5  │ [20DC]│ Cát Lái D1  │ 05/04 08:00 │ 04/04 20:00 │ 3,500,000₫ │[AVAILABLE] │ 3d 12h  │ [⋯] Menu   │
│ ☐ │ CMAU987654 3  │ [40HC]│ Đồng Nai D3 │ 06/04 14:00 │ 06/04 02:00 │ 5,200,000₫ │[LOCKED]    │ 4d 6h   │ [⋯] Menu   │
│ ☐ │ MAEU555432 1  │ [40DC]│ Cát Lái D2  │ 03/04 06:00 │ 02/04 18:00 │ 4,800,000₫ │[ON_VESSEL] │ –       │ [⋯] Menu   │
├─────────────────────────────────────────────────────────────────┤
│ Bulk actions: [Xóa đã chọn] [Export CSV]      Trang: 1/14 [→]  │
└─────────────────────────────────────────────────────────────────┘
```

**Data:**
```
GET /api/sl/containers?type=&status=&etd_from=&etd_to=&page=&limit=20
→ {
    data: [{
      container_id, type, shipping_line_id,
      depot_location: { address, lat, lng },
      etd, closing_time, floor_price_vnd,
      status: "AVAILABLE"|"MATCHED"|"LOCKED"|"ON_VESSEL"|"RETURNED",
      days_remaining, matched_booking_id?
    }],
    pagination: { total, page, limit }
  }
```

**Row Action Menu (⋯):**
- Xem chi tiết
- Chỉnh sửa (chỉ khi status = AVAILABLE)
- Xem matching đề xuất
- Hủy đăng ký (chỉ khi AVAILABLE, xác nhận dialog)

**Status Badge Colors:**
```
AVAILABLE  → Green  (#10B981) — có thể nhận hàng
MATCHED    → Cyan   (#00D4FF) — đang trong quá trình xác nhận
LOCKED     → Amber  (#F59E0B) — đang time-lock 30 phút
ON_VESSEL  → Purple (#8B5CF6) — đã lên tàu
RETURNED   → Gray   (#475569) — đã hoàn trả depot HP
```

---

### [SL-03] — Supply Registration Form

**Purpose:** Đăng ký container rỗng mới vào hệ thống. Validate ISO 6346, logic thời gian ETD/T_cutoff, cảnh báo lưu bãi.

**Layout:**
```
┌─────────────────────────────────────────────────────────────────┐
│ MODAL HEADER: "Đăng ký Container Rỗng"                    [✕]  │
├─────────────────────────────────────────────────────────────────┤
│ SECTION 1: THÔNG TIN CONTAINER                                   │
│ ┌─────────────────────────────────────────────────────────────┐ │
│ │ Mã container *    [TCKU    ] [123456] [5]  ← 3 input fields │ │
│ │                   ↑ 4 chữ   ↑ 6 số  ↑ check digit auto     │ │
│ │                   [✅ Hợp lệ ISO 6346]                       │ │
│ │                                                             │ │
│ │ Loại vỏ *        [20DC ▼]   Số lượng *  [__] units          │ │
│ └─────────────────────────────────────────────────────────────┘ │
├─────────────────────────────────────────────────────────────────┤
│ SECTION 2: VỊ TRÍ & THỜI GIAN                                   │
│ ┌─────────────────────────────────────────────────────────────┐ │
│ │ Vị trí Depot *   [Chọn từ danh sách ▼] hoặc [📍 Nhập GPS]  │ │
│ │ Cảng đích         VNHPH — Hải Phòng (cố định, readonly)     │ │
│ │                                                             │ │
│ │ ETD (Tàu chạy) *  [────────────] Date+Time picker          │ │
│ │ T_cutoff *        [────────────] Auto = ETD − 12h (editable)│ │
│ │                   ⚠️ Phải trong khoảng ETD − 8h đến ETD − 24h│ │
│ └─────────────────────────────────────────────────────────────┘ │
├─────────────────────────────────────────────────────────────────┤
│ SECTION 3: GIÁ & CẢNH BÁO                                       │
│ ┌─────────────────────────────────────────────────────────────┐ │
│ │ Giá sàn (VNĐ/TEU) * [_______________] ₫                    │ │
│ │                                                             │ │
│ │ ── DEMURRAGE RISK CALCULATOR ──────────────────────────────  │ │
│ │ Số ngày tồn bãi dự kiến: [4.5 ngày] > Free Time (3 ngày)   │ │
│ │ ⚠️ CẢNH BÁO: Nguy cơ phí lưu bãi!                          │ │
│ │  Ngày 4:  +500,000₫/container/ngày                         │ │
│ │  Ngày 5:  +750,000₫/container/ngày (lũy tiến)              │ │
│ │  Phí ước tính: ~2,500,000₫ nếu không matching kịp          │ │
│ └─────────────────────────────────────────────────────────────┘ │
├─────────────────────────────────────────────────────────────────┤
│ [HỦY]                                    [✅ ĐĂNG KÝ NGAY]    │
└─────────────────────────────────────────────────────────────────┘
```

**Data & Validation Logic:**
```
Realtime validations:
  1. Container ID: format XXXX999999X → auto-compute check digit
     → Show ✅/❌ inline ngay khi nhập xong 10 ký tự đầu
  2. ETD: phải ≥ NOW() + 24h → error "ETD phải cách hiện tại ít nhất 24 giờ"
  3. T_cutoff: phải trong [ETD-24h, ETD-8h] → slider range hint
  4. Demurrage: (ETD - NOW) > 3 ngày → hiện WARNING box với bảng phí
  5. Floor price: > 0, phải số nguyên (VNĐ không có xu)

POST /api/sl/containers
```

---

### [SL-04] — Matching Inbox

**Purpose:** Xem tất cả đề xuất khớp lệnh từ Matching Engine, theo dõi trạng thái xác nhận.

**Layout:**
```
┌─────────────────────────────────────────────────────────────────┐
│ HEADER: "Hộp thư Matching"         [Filter: Tất cả ▼] [Refresh]│
├─────────────────────────────────────────────────────────────────┤
│ STATUS TABS: [Chờ xác nhận (3)] [Đã xác nhận] [Hết hạn] [Tất cả]│
├─────────────────────────────────────────────────────────────────┤
│ CARD LIST (dạng cards, không phải table)                         │
│                                                                  │
│ ┌─────────────────────────────────────────────────────────────┐ │
│ │ 🔴 URGENT — Còn ⏱ 24:31 để xác nhận           [PROPOSED]   │ │
│ │ Container: TCKU123456 5 [20DC] × Booking #4521 (NVOCC: Vinafco)│ │
│ │ Fill Rate: ████████░░ 87%  │ Z_total: 3,240,000₫ (-23% vs TT)│ │
│ │ Pareto Rank: #1/5          │ ETD: 05/04/2026 08:00           │ │
│ │ [XEM CHI TIẾT & XÁC NHẬN →]              [TỪ CHỐI ✕]        │ │
│ └─────────────────────────────────────────────────────────────┘ │
│                                                                  │
│ ┌─────────────────────────────────────────────────────────────┐ │
│ │ 🟡 Còn ⏱ 18:05 để xác nhận               [PROPOSED]         │ │
│ │ Container: CMAU654321 8 [40HC] × Booking #4498 (NVOCC: ITACO)│ │
│ │ Fill Rate: ██████████ 94%  │ Z_total: 7,100,000₫ (-18% vs TT)│ │
│ │ [XEM CHI TIẾT & XÁC NHẬN →]              [TỪ CHỐI ✕]        │ │
│ └─────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

**Data:**
```
GET /api/sl/matches?status=PROPOSED|CONFIRMED|EXPIRED
→ [{
    match_id, container_id, booking_id, nvocc_name,
    fill_rate_pct, z_total_vnd, z_market_vnd, z_savings_pct,
    pareto_rank, time_lock_expires_at,
    status: "PROPOSED"|"SL_CONFIRMED"|"CONFIRMED"|"REJECTED"|"EXPIRED"
  }]

WebSocket: nhận event NEW_MATCH_PROPOSED → thêm card mới realtime
```

---

### [SL-05] — Dual Confirmation (Chi tiết Khớp lệnh)

**Purpose:** Màn hình quan trọng nhất — Hãng tàu xem xét đề xuất matching 3D, so sánh tài chính và xác nhận/từ chối trong 30 phút.

**Layout:**
```
┌─────────────────────────────────────────────────────────────────┐
│ TOP BAR: "Xem xét Matching #M-4521"   ⏱ TIME LOCK: [27:43] 🔴  │
│          (Countdown timer đổi sang đỏ nhấp nháy khi < 5 phút)  │
├────────────────────────────┬────────────────────────────────────┤
│ LEFT PANEL (55%)            │ RIGHT PANEL (45%)                  │
│ ─────────────────────────── │ ──────────────────────────────── │
│ 🧊 3D CONTAINER VISUALIZER  │ 📋 CHI TIẾT GIAO DỊCH             │
│ (Three.js canvas)           │                                    │
│                             │ Container: TCKU123456 [20DC]      │
│  [Three.js 3D render]       │ Booking: #4521 — Vinafco          │
│  - Khung container rỗng     │ ETD: 05/04/2026 08:00             │
│  - Hàng hóa màu theo        │ T_cutoff: 04/04/2026 20:00        │
│    HS Group (JSONB layout)  │                                    │
│  - Animate rotate nhẹ       │ ── PHÂN TÍCH TÀI CHÍNH ──────── │
│  - Controls: Rotate/Zoom/   │ Z_total:    3,240,000₫ ✅          │
│    Reset View               │ Z_market:   4,200,000₫            │
│                             │ Tiết kiệm:  ██ -23.1%             │
│ Fill Rate: ████████░░ 87%   │                                    │
│ CBM: 21.7/25.0 m³           │ C_sea:  2,100,000₫                │
│ GW:  12,450/28,000 kg       │ C_road:   890,000₫                │
│                             │ C_penalty risk: 250,000₫          │
│ [CONTROLS: 🔄 Reset | 🔍+/-]│                                    │
│                             │ ── LỘ TRÌNH XE TẢI ────────────  │
│                             │ [Mini Leaflet map]                 │
│                             │ Depot A → CFS Đồng Nai → Cảng     │
│                             │ ETA Gate-in: 04/04 17:45          │
│                             │ Buffer: +45 phút (peak 17:00)     │
│                             │ Slack T_cutoff: 2h 15m ✅          │
│                             │                                    │
│                             │ ── SLOT SHARING AGREEMENT ──────  │
│                             │ [Preview PDF] vessel_flag: 🇻🇳 VN │
│                             │ Feeder: Hải An Shipping — VN      │
│                             │                                    │
│                             │ ┌─────────────┐ ┌──────────────┐  │
│                             │ │ ✅ XÁC NHẬN │ │ ✕ TỪ CHỐI   │  │
│                             │ │  (primary)  │ │  (outlined)  │  │
│                             │ └─────────────┘ └──────────────┘  │
└────────────────────────────┴────────────────────────────────────┘
```

**Data:**
```
GET /api/sl/matches/:match_id
→ {
    match_id, container_id, booking_id,
    packing_layout_json: [{item_id, x, y, z, L, W, H, color_group}],
    space_utilization_cbm_pct, space_utilization_weight_pct,
    c_sea_vnd, c_road_vnd, c_penalty_risk_vnd, z_total_vnd, z_market_vnd,
    route_plan_json: [{location, eta, leg_distance_km}],
    time_lock_expires_at,
    slot_sharing_agreement_preview_url
  }

POST /api/sl/matches/:match_id/confirm  → { status: "SL_CONFIRMED" }
POST /api/sl/matches/:match_id/reject   → { status: "REJECTED", reason }
```

**UX Requirements:**
- Three.js: Load layout_json, render mỗi item là BoxGeometry với màu theo `color_group`
- Container skeleton render trước (wireframe), items animate xuất hiện lần lượt
- Countdown timer: amber < 10 phút, đỏ nhấp nháy < 5 phút, sound alert
- Confirm dialog: "Bạn chắc chắn muốn xác nhận? Giao dịch sẽ không thể hoàn tác."
- Reject dialog: Dropdown lý do từ chối (price, timing, other) + textarea

---

### [SL-06] — Slot Sharing Agreement Manager

**Purpose:** Quản lý và tra cứu hợp đồng 3 bên đã ký — pháp lý Cabotage NĐ 160/2016.

**Layout:**
```
┌─────────────────────────────────────────────────────────────────┐
│ HEADER: "Hợp đồng Chia sẻ Slot"        [🔍 Tìm] [Export PDF]   │
├─────────────────────────────────────────────────────────────────┤
│ FILTER: [Status ▼] [Feeder ▼] [Tháng ▼]                         │
├─────────────────────────────────────────────────────────────────┤
│ TABLE:                                                           │
│ Agreement ID │ Container   │ NVOCC      │ Feeder VN  │ Signed   │ Status   │ Vessel 🇻🇳 │
│ SSA-240401-01│ TCKU123456  │ Vinafco    │ Hải An     │ 01/04    │[SIGNED]  │ ✅ VN     │
│ SSA-240402-02│ CMAU654321  │ ITACO      │ VIMC       │ 02/04    │[PENDING] │ ✅ VN     │
├─────────────────────────────────────────────────────────────────┤
│ SIDE PANEL (khi click vào row):                                  │
│ ┌─────────────────────────────────────────────────────────────┐ │
│ │ Agreement SSA-240401-01                                      │ │
│ │ vessel_flag: 🇻🇳 VN (Tuân thủ NĐ 160/2016/NĐ-CP)           │ │
│ │ Bên 1: Evergreen Line (Chủ vỏ)                              │ │
│ │ Bên 2: Vinafco NVOCC (Đặt slot)                             │ │
│ │ Bên 3: Hải An Shipping (Tàu VN)                             │ │
│ │ Blockchain Hash: 0x4f3a...c9b1 [Copy]                       │ │
│ │ [📄 Tải hợp đồng PDF] [🔗 Verify on-chain]                  │ │
│ └─────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

**Validation UI:** Row có `vessel_flag ≠ "VN"` → highlight đỏ + icon ⚠️ "Vi phạm Cabotage"

---

### [SL-07] — Invoice & Reconciliation

**Purpose:** Đối soát chi phí thực tế vs kế hoạch sau khi giao dịch hoàn thành — nguồn dữ liệu model learning.

**Layout:**
```
┌─────────────────────────────────────────────────────────────────┐
│ HEADER: "Đối soát Hóa đơn"      [Tháng 04/2026 ▼] [Export]    │
├─────────────────────────────────────────────────────────────────┤
│ SUMMARY CARDS:                                                   │
│ [Tổng hóa đơn: 241] [Đã TT: 198] [Chờ TT: 43] [Tranh chấp: 2] │
├─────────────────────────────────────────────────────────────────┤
│ RECONCILIATION TABLE:                                            │
│ Invoice ID    │ Container   │ C_sea Kế hoạch │ C_sea Thực tế │ Chênh lệch │ Status   │
│ INV-2604-0042 │ TCKU123456  │ 2,100,000₫     │ 2,240,000₫    │ +140,000₫  │[PAID]    │
│ INV-2604-0041 │ CMAU654321  │ 4,800,000₫     │ 4,800,000₫    │ 0₫         │[PENDING] │
├─────────────────────────────────────────────────────────────────┤
│ VARIANCE CHART: Bar chart dọc — Planned vs Actual mỗi tuần      │
└─────────────────────────────────────────────────────────────────┘
```

---

---

## 📦 PHẦN 3: ROLE — NVOCC / CONSOLIDATOR (8 MÀN HÌNH)

**Role color accent:** `#10B981` (Emerald)
**Sidebar icon:** 📦

---

### [NV-01] — NVOCC Dashboard

**Purpose:** Tổng quan hoạt động gom hàng, chi phí, tiết kiệm so với thị trường.

**Layout:**
```
┌─────────────────────────────────────────────────────────────────┐
│ SCORECARDS (6 cards):                                            │
│ [Booking đang xử lý] [Lô IN_TRANSIT] [Fill Rate TB] [Tiết kiệm TB%] [Chi phí MTD] [Booking hoàn thành]│
├─────────────────────────────────────────────────────────────────┤
│ ROW 2:                                                           │
│ ┌──────────────────────────────┐ ┌────────────────────────────┐ │
│ │ Savings vs Market (Line)     │ │ Booking Status Pipeline     │ │
│ │ % tiết kiệm theo ngày        │ │ Funnel: PENDING→MATCHED→    │ │
│ │ Benchmark: target 20%        │ │ CONFIRMED→IN_TRANSIT→DONE   │ │
│ └──────────────────────────────┘ └────────────────────────────┘ │
├─────────────────────────────────────────────────────────────────┤
│ QUICK ACTIONS:                                                   │
│ [+ Tạo Booking Mới]  [📦 Xem lô IN_TRANSIT]  [📄 Chứng từ]     │
└─────────────────────────────────────────────────────────────────┘
```

---

### [NV-02] — LCL Booking List

**Purpose:** Danh sách tất cả booking requests, quản lý trạng thái lô hàng.

**Layout:**
```
┌─────────────────────────────────────────────────────────────────┐
│ HEADER: "Danh sách Booking LCL"           [+ TẠO BOOKING MỚI]  │
├─────────────────────────────────────────────────────────────────┤
│ STATUS TABS: [Tất cả] [PENDING (5)] [MATCHED (2)] [IN_TRANSIT (8)] [COMPLETED]│
├─────────────────────────────────────────────────────────────────┤
│ TABLE:                                                           │
│ Booking ID │ Tổng CBM │ Tổng GW  │ Deadline    │ Budget       │ Status        │ Actions  │
│ BK-240401  │ 18.7 m³  │ 8,450 kg │ 05/04 08:00 │ 5,000,000₫   │[MATCHED] 🔵   │ [Xem]    │
│ BK-240399  │ 32.4 m³  │ 15,200 kg│ 07/04 00:00 │ 8,500,000₫   │[IN_TRANSIT] 🟣│ [Track]  │
│ BK-240388  │ 8.2 m³   │ 2,100 kg │ 10/04 00:00 │ 3,000,000₫   │[PENDING] ⚪    │ [Xem]    │
└─────────────────────────────────────────────────────────────────┘
```

---

### [NV-03] — Booking Form (Thông tin Chung)

**Purpose:** Bước 1/2 tạo booking — nhập thông tin logistics cơ bản.

**Layout:**
```
┌─────────────────────────────────────────────────────────────────┐
│ PROGRESS: Bước 1/2 — Thông tin chung ════════░░░░ Bước 2: Hàng hóa│
├─────────────────────────────────────────────────────────────────┤
│ FORM (max-width 680px, centered):                                │
│                                                                  │
│ CFS xuất phát *                                                  │
│ [🗺️ Chọn trên bản đồ hoặc nhập địa chỉ              ]          │
│ Mini Leaflet map preview (256px height)                          │
│                                                                  │
│ Cảng xuất phát    [Cát Lái — VNCLI] (readonly, tuyến cố định)   │
│ Cảng đích         [Hải Phòng — VNHPH] (readonly)               │
│                                                                  │
│ Hạn giao hàng *   [────────────────────] Date+Time picker       │
│ Ngân sách tối đa * [__________________] ₫/TEU                   │
│                    Hint: Thị trường hiện tại ~4,200,000₫/TEU    │
│                                                                  │
│ Flag đặc biệt:                                                   │
│ ☐ Hàng cần làm lạnh (Reefer)                                    │
│ ☐ Có hàng dễ vỡ (Fragile)                                       │
│ ☐ Cần giấy phép đặc biệt (Dangerous)                           │
│                                                                  │
│ Ghi chú cho Hãng tàu [____________________________________]     │
│                                                                  │
├─────────────────────────────────────────────────────────────────┤
│ [HỦY]                                  [TIẾP THEO: Nhập Hàng →]│
└─────────────────────────────────────────────────────────────────┘
```

---

### [NV-04] — Booking Form (Cargo Items — Lõi nhập liệu) ⭐

**Purpose:** Bước 2/2 — Nhập chi tiết từng kiện hàng. Đây là màn hình quan trọng nhất, cung cấp data cho thuật toán FFD Lớp 1.

**Layout:**
```
┌─────────────────────────────────────────────────────────────────────────────┐
│ PROGRESS: Bước 2/2 — Thông tin Hàng hóa ████████████████ 100%               │
├──────────────────────────────────────────┬──────────────────────────────────┤
│ LEFT: DYNAMIC DATA GRID (70%)             │ RIGHT: SIDEBAR TÓM TẮT (30%)     │
│ ─────────────────────────────────────── │ ────────────────────────────────  │
│ [+ THÊM KIỆN HÀNG]  [Import CSV]         │ 📦 TỔNG KẾT LÔ HÀNG              │
│                                           │                                   │
│ ┌────┬──────┬────┬────┬────┬────┬───┬──┐ │ Tổng CBM:                         │
│ │ #  │ Dài  │ R  │ C  │GW  │HS  │Qty│⋯ │ │ ██████████░░░░░░░░░░ 18.7/25 m³   │
│ │    │  cm  │cm  │cm  │ kg │Code│   │  │ │ 74.8% — Phù hợp 20DC ✅            │
│ ├────┼──────┼────┼────┼────┼────┼───┼──┤ │                                   │
│ │ 1  │ 120  │100 │ 80 │250 │6109│10 │⋮ │ │ Tổng trọng lượng:                 │
│ │    │      │    │    │    │1000│   │  │ │ ████░░░░░░░░░░░░░░░░ 8,450/28,000 │
│ ├────┼──────┼────┼────┼────┼────┼───┼──┤ │ 30.2% — OK ✅                     │
│ │ 2  │  80  │ 60 │ 40 │ 80 │8471│ 5 │⋮ │ │                                   │
│ │    │      │    │    │    │5000│   │  │ │ Gợi ý Container:                  │
│ ├────┼──────┼────┼────┼────┼────┼───┼──┤ │ 🟢 [20DC] — 74.8% fill            │
│ │ 3  │[WARN]│    │    │    │[HS │   │  │ │ hoặc ghép 40DC cho batch tiếp     │
│ │    │265cm │    │    │    │2807│   │  │ │                                   │
│ │    │> max │    │    │    │🔴] │   │  │ │ ── CẢNH BÁO HS CODE ──────────  │
│ ├────┴──────┴────┴────┴────┴────┴───┴──┤ │ ⚠️ Item #3: H2SO4 (HS 28070010)  │
│ │ Dài quá 234cm — Vượt chiều rộng 20DC │ │ IMDG Class 8 — Hàng nguy hiểm   │
│ └───────────────────────────────────────┘ │ 🔴 BLOCK: Cát Lái cấm IMDG       │
│                                           │    từ 01/07/2024                  │
│ [+ Thêm dòng]                             │ [Xóa item #3 để tiếp tục]         │
│                                           │                                   │
│ HS Code Lookup (realtime):               │ ── DỰ KIẾN PHÍ ─────────────── │
│ Gõ HS Code → fetch tên hàng & IMDG class│ Z_est: ~3,500,000₫               │
│                                           │ vs TT: ~4,200,000₫ (-16%)        │
├──────────────────────────────────────────┴──────────────────────────────────┤
│ [← QUAY LẠI]          [LƯU NHÁP]            [🚀 GỬI VÀO QUEUE MATCHING]     │
│                                    ↑ Disabled khi có IMDG BLOCK             │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Data & Validation:**
```
Realtime per-item validations:
  - L/W/H: positive numbers, W ≤ 234cm (container inner width), H ≤ 238cm
  - HS Code: 8 digits → GET /api/lookup/hs-code/:code
    → { description, imdg_class, imdg_group, is_food_category }
  - IMDG check: if imdg_class NOT NULL AND origin = VNCLI → 🔴 BLOCK
  - Fragile + Stackable: conflict validation
  - Weight: per-item × qty, cumulative weight check

Auto-calculations (realtime):
  - CBM per row: L×W×H/1,000,000 × qty
  - Total CBM, Total GW
  - Progress bar color: green (< 80%), amber (80-95%), red (> 95%)
  - Container suggestion: CBM ≤ 22 → 20DC, 22-60 → 40DC/HC

Submit button:
  - ENABLED: all items valid, no IMDG block
  - DISABLED + tooltip: if any IMDG block present
```

---

### [NV-05] — Matching Result & 3D Visualizer

**Purpose:** NVOCC xem kết quả matching, phân tích tài chính, preview 3D container và xác nhận giao dịch.

*(Layout tương tự [SL-05] nhưng từ góc nhìn NVOCC)*

**Layout:**
```
┌─────────────────────────────────────────────────────────────────┐
│ TOP BAR: "Kết quả Matching — Booking #4521"  ⏱ Còn [26:12] 🟡  │
├────────────────────────────┬────────────────────────────────────┤
│ LEFT: 3D VISUALIZER (55%)  │ RIGHT: PHÂN TÍCH NVOCC (45%)       │
│                             │                                    │
│ [Three.js 3D render]        │ ── SO SÁNH CHI PHÍ ─────────────  │
│ Container + hàng của bạn    │                                    │
│ highlighted màu xanh        │ 💰 Z_total của bạn:               │
│                             │    3,240,000₫/TEU                 │
│ [Bottom tabs]:              │                                    │
│ [Rank 1 ★] [Rank 2] [Rank 3]│ 🏪 Giá thị trường:               │
│ Chuyển giữa 5 options Pareto│    4,200,000₫/TEU                 │
│                             │                                    │
│ Metrics:                    │ 📉 Bạn tiết kiệm:                 │
│ Fill Rate:  87% ████████░░  │    960,000₫/TEU (-22.9%)          │
│ CBM:        21.7/25.0 m³    │                                    │
│ ETD:        05/04 08:00     │ ── LỘ TRÌNH XE TẢI ──────────── │
│ T_cutoff:   04/04 20:00     │ [Leaflet map — route xe tải]      │
│                             │ ETA Gate-in: 04/04 17:45          │
│                             │ ✅ Kịp T_cutoff (2h 15m trước)    │
│                             │                                    │
│                             │ Feeder tàu: Hải An — 🇻🇳 VN flag  │
│                             │                                    │
│                             │ ┌──────────────┐ ┌─────────────┐  │
│                             │ │ ✅ XÁC NHẬN  │ │ ✕ TỪ CHỐI  │  │
│                             │ │    GIAO DỊCH │ │             │  │
│                             │ └──────────────┘ └─────────────┘  │
└────────────────────────────┴────────────────────────────────────┘
```

**Pareto Tabs:** Cho phép NVOCC xem và chọn từ top-5 Pareto options (mỗi tab là một phương án khác nhau về cost/time trade-off)

---

### [NV-06] — In-Transit Shipment Tracking

**Purpose:** Theo dõi hành trình lô hàng từ khi xác nhận đến khi dỡ hàng tại Hải Phòng.

**Layout:**
```
┌─────────────────────────────────────────────────────────────────┐
│ HEADER: "Theo dõi Lô hàng — Booking #4521 / Container TCKU123456"│
├─────────────────────────────────────────────────────────────────┤
│ TIMELINE (vertical, trái) + MAP (phải, 55%)                     │
│ ┌──────────────────┐  ┌─────────────────────────────────────┐  │
│ │ TIMELINE:        │  │ Leaflet Map — Live tracking          │  │
│ │                  │  │ (cập nhật GPS mỗi 30 giây)           │  │
│ │ ✅ XONG          │  │ Route: Depot → CFS → Cát Lái → Biển │  │
│ │ 📍 Lấy vỏ        │  │ → Hải Phòng                         │  │
│ │ 01/04 09:15      │  │                                      │  │
│ │       │          │  │ 🚚 Xe tải: 51C-123.45               │  │
│ │ ✅ XONG          │  │ Vị trí: Đang trên QL1A, km 45       │  │
│ │ 📦 Đóng hàng CFS │  │ Tốc độ: 72 km/h                     │  │
│ │ 01/04 13:30      │  └─────────────────────────────────────┘  │
│ │       │          │                                            │
│ │ ✅ XONG          │  SHIP TRACKER (AIS):                       │
│ │ ⚓ Gate-in Cảng  │  Tàu: HAI AN 05                           │
│ │ 01/04 17:45      │  ETA Hải Phòng: 03/04 06:00 (AIS API)     │
│ │       │          │  [Map tàu trên biển — vị trí realtime]    │
│ │ 🔵 ĐANG XỬ LÝ    │                                            │
│ │ 🚢 Trên biển     │  ── ALERTS ─────────────────────────────  │
│ │ ETA: 03/04 06:00 │  ✅ Tất cả đúng tiến độ                   │
│ │       │          │  Không có cảnh báo nào                    │
│ │ ⬜ CHỜ           │                                            │
│ │ 📤 Dỡ hàng HP    │                                            │
│ └──────────────────┘                                            │
└─────────────────────────────────────────────────────────────────┘
```

**Data:**
```
GET /api/nv/bookings/:booking_id/tracking
→ {
    trucking_order: { status, actual_pickup_ts, actual_gate_in_ts,
                      gps_track_json: [{lat, lng, speed, ts}] },
    vessel: { name, ais_position: {lat, lng}, eta_haiphong },
    timeline: [{ step, status: "DONE"|"ACTIVE"|"PENDING", timestamp }]
  }

WebSocket: GPS updates mỗi 30 giây → cập nhật marker trên map
```

---

### [NV-07] — Document Management

**Purpose:** Quản lý và tải chứng từ vận tải — Stuffing Plan, HBL, Invoice.

**Layout:**
```
┌─────────────────────────────────────────────────────────────────┐
│ HEADER: "Chứng từ — Booking #4521"              [📁 Tải tất cả] │
├─────────────────────────────────────────────────────────────────┤
│ DOCUMENT GRID (3 columns):                                       │
│ ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────────┐ │
│ │ 📄 Stuffing Plan│ │ 📋 HBL (House   │ │ 🧾 Invoice          │ │
│ │ Generated auto  │ │    Bill Lading) │ │ INV-2604-0042       │ │
│ │ từ Bin Packing  │ │ HBL-4521        │ │ 3,240,000₫          │ │
│ │ 01/04/2026      │ │ 01/04/2026      │ │ 03/04/2026          │ │
│ │ PDF 2.4 MB      │ │ PDF 1.1 MB      │ │ PDF 0.8 MB          │ │
│ │ [👁 Xem] [⬇ Tải]│ │ [👁 Xem] [⬇ Tải]│ │ [👁 Xem] [⬇ Tải]   │ │
│ └─────────────────┘ └─────────────────┘ └─────────────────────┘ │
│ ┌─────────────────┐ ┌─────────────────┐                         │
│ │ 📜 Customs      │ │ 🤝 Slot Sharing │                         │
│ │ Declaration     │ │ Agreement       │                         │
│ │ HQ-4521-VNCLI   │ │ SSA-240401-01   │                         │
│ │ [👁 Xem] [⬇ Tải]│ │ [👁 Xem] [⬇ Tải]│                         │
│ └─────────────────┘ └─────────────────┘                         │
└─────────────────────────────────────────────────────────────────┘
```

---

### [NV-08] — Unstuffing & Final Invoice

**Purpose:** Xác nhận nhận hàng tại Hải Phòng, xem hóa đơn cuối, đánh giá % tiết kiệm thực tế.

**Layout:**
```
┌─────────────────────────────────────────────────────────────────┐
│ HEADER: "Dỡ hàng & Thanh toán — Booking #4521"                  │
├─────────────────────────────────────────────────────────────────┤
│ SUCCESS BANNER (nếu đã hoàn thành):                             │
│ ✅ Lô hàng đã giao thành công — Container TCKU123456             │
│ Dỡ hàng lúc: 03/04/2026 09:30 tại CFS Đình Vũ, Hải Phòng      │
├─────────────────────────────────────────────────────────────────┤
│ ┌──────────────────────────────┐ ┌─────────────────────────────┐│
│ │ INVOICE DETAIL               │ │ SAVINGS REPORT               ││
│ │                              │ │                              ││
│ │ Invoice: INV-2604-0042       │ │  22.9% 💰                    ││
│ │ Ngày: 03/04/2026             │ │  Tiết kiệm thực tế           ││
│ │                              │ │                              ││
│ │ C_sea thực tế: 2,240,000₫    │ │  ┌────────────────────────┐  ││
│ │ C_road thực tế:  890,000₫    │ │  │ Z_actual:  3,380,000₫  │  ││
│ │ C_penalty:            0₫    │ │  │ Z_market:  4,380,000₫  │  ││
│ │ ─────────────────────────── │ │  │ Tiết kiệm:  1,000,000₫  │  ││
│ │ TỔNG:           3,130,000₫   │ │  └────────────────────────┘  ││
│ │                              │ │                              ││
│ │ Fill Rate thực tế: 91.2%     │ │ So với dự báo: +4.2% 🎉      ││
│ │ (vs kế hoạch 87%)            │ │                              ││
│ │                              │ │ [📊 Xem báo cáo đầy đủ]     ││
│ │ [💳 THANH TOÁN NGAY]         │ │                              ││
│ └──────────────────────────────┘ └─────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

---

---

## 🚚 PHẦN 4: ROLE — TRUCKING COMPANY (6 MÀN HÌNH)

**Role color accent:** `#F59E0B` (Amber)
**Sidebar icon:** 🚚

---

### [TR-01] — Route Map & Dispatch Dashboard ⭐

**Purpose:** Màn hình điều phối cốt lõi — dispatcher theo dõi toàn bộ xe tải realtime, cảnh báo T_cutoff.

**Layout:**
```
┌─────────────────────────────────────────────────────────────────────────────┐
│ HEADER: "Trung tâm Điều phối"    Đang hoạt động: 8 xe  ⚠️ Cảnh báo: 1     │
├──────────────────────────────────────────┬──────────────────────────────────┤
│ MAP FULL SCREEN (Leaflet/Mapbox) — 70%   │ SIDEBAR FLEET STATUS — 30%       │
│                                           │                                   │
│ [Live map với GPS markers]                │ FILTER: [Tất cả] [⚠️ Nguy hiểm]  │
│                                           │                                   │
│ Truck markers:                            │ ┌────────────────────────────────┐│
│ 🔴 51C-123.45 (URGENT: sắp trễ)          │ │ 🔴 51C-123.45                  ││
│ 🟡 51C-678.90 (canh chừng)               │ │ TCKU123456 [20DC]              ││
│ 🟢 51C-456.78 (an toàn)                  │ │ T_cutoff: 04/04 20:00          ││
│ 🟢 51C-234.56 (an toàn)                  │ │ ETA: 04/04 19:12 ⚠️            ││
│                                           │ │ Slack: chỉ còn 48 phút!        ││
│ Route lines:                              │ │ [📞 Gọi tài xế] [Xem route]   ││
│ Solid line = kế hoạch PSO                │ └────────────────────────────────┘│
│ Dashed line = thực tế di chuyển          │                                   │
│                                           │ ┌────────────────────────────────┐│
│ [Layer controls]:                         │ │ 🟡 51C-678.90                  ││
│ ☑ Depots  ☑ CFS  ☑ Ports                │ │ CMAU654321 [40HC]              ││
│ ☑ Current GPS  ☑ Planned route           │ │ ETA: 04/04 15:30               ││
│                                           │ │ Slack: 4h 30m 🟡               ││
│ Map controls: +/- zoom, fullscreen        │ │ [Xem route]                    ││
│                                           │ └────────────────────────────────┘│
│                                           │                                   │
│                                           │ ┌────────────────────────────────┐│
│                                           │ │ 🟢 51C-456.78                  ││
│                                           │ │ MAEU555432 [40DC]              ││
│                                           │ │ ETA: 05/04 14:00               ││
│                                           │ │ Slack: 18h ✅                   ││
│                                           │ └────────────────────────────────┘│
├──────────────────────────────────────────┴──────────────────────────────────┤
│ BOTTOM ALERT STRIP (sticky):                                                  │
│ 🔴 ALERT: Xe 51C-123.45 — ETA 19:12 vs T_cutoff 20:00. Còn 48 phút!        │
│    Tài xế: Nguyễn Văn A | [📞 Gọi ngay] [🗺️ Xem phương án thay thế]        │
└──────────────────────────────────────────────────────────────────────────────┘
```

**Data:**
```
WebSocket: ws://api/tr/fleet/live
→ Event every 30s: { truck_plate, lat, lng, speed, status, eta_gate_in }

Alert logic (frontend):
  IF (eta_gate_in > t_cutoff × 0.9) → 🔴 URGENT marker + alert strip
  IF (eta_gate_in > t_cutoff × 0.95 AND < t_cutoff) → 🟡 WARNING
  IF (eta_gate_in ≤ t_cutoff × 0.9) → 🟢 SAFE

Buffer Time displayed from Redis cache:
  Current hour → buffer_time_min → shown on each truck card
```

---

### [TR-02] — Trucking Order Management

**Purpose:** Quản lý danh sách lệnh xe tải — Kanban view cho dispatcher.

**Layout:**
```
┌─────────────────────────────────────────────────────────────────┐
│ HEADER: "Quản lý Lệnh Xe"         [+ TẠO LỆNH MỚI] [📊 Export]│
├─────────────────────────────────────────────────────────────────┤
│ VIEW TOGGLE: [📋 Danh sách] [📌 Kanban] ← Default: Table       │
├─────────────────────────────────────────────────────────────────┤
│ TABLE VIEW:                                                      │
│ Order ID  │ Container   │ Biển số  │ Tài xế       │ ETA Pickup  │ ETA Gate-in │ Status          │ Actions  │
│ TO-240401 │ TCKU123456  │51C-123.45│ Nguyễn Văn A │ 01/04 09:00 │ 01/04 17:45 │[EN_ROUTE_CFS]🔵 │ [Xem] [⋯]│
│ TO-240402 │ CMAU654321  │51C-678.90│ Trần Văn B   │ 02/04 08:00 │ 02/04 16:00 │[ISSUED] ⚪       │ [Xem] [⋯]│
│ TO-240399 │ MAEU555432  │51C-456.78│ Lê Thị C     │ 01/04 07:00 │ 01/04 14:30 │[GATE_IN_DONE]✅  │ [Xem]    │
├─────────────────────────────────────────────────────────────────┤
│ KANBAN VIEW:                                                     │
│ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────┐ │
│ │ ISSUED   │ │EN_ROUTE  │ │ AT_CFS   │ │EN_ROUTE  │ │GATE_IN │ │
│ │ (2 cards)│ │ DEPOT(1) │ │ (3 cards)│ │ PORT(2)  │ │ DONE   │ │
│ │          │ │          │ │          │ │          │ │(8cards)│ │
│ │[Card]    │ │[Card]    │ │[Card]    │ │[Card]    │ │[Card]  │ │
│ └──────────┘ └──────────┘ └──────────┘ └──────────┘ └────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

---

### [TR-03] — Fleet & Driver Settings

**Purpose:** Quản lý danh sách xe và tài xế, phân công, cập nhật thông tin.

**Layout:**
```
┌──────────────────┬──────────────────────────────────────────────┐
│ TABS:            │ TAB CONTENT                                   │
│ 🚚 Xe đầu kéo   │                                               │
│ 👨‍✈️ Tài xế     │ TAB Xe đầu kéo:                              │
│ 📋 Phân công     │ [+ Thêm xe]                                  │
│                  │ Table: Biển số | Model | Tải trọng | Status   │
│                  │ 51C-123.45 | Isuzu FVZ | 15T | [ACTIVE] ✅   │
│                  │                                               │
│                  │ TAB Tài xế:                                   │
│                  │ [+ Thêm tài xế]                               │
│                  │ Table: Họ tên | Phone | GPLX | Status         │
│                  │ Nguyễn Văn A | 0901... | B2-HP | [ACTIVE]    │
│                  │                                               │
│                  │ TAB Phân công:                                │
│                  │ Assign tài xế → xe → lệnh                    │
│                  │ Calendar/Schedule view                        │
└──────────────────┴──────────────────────────────────────────────┘
```

---

### [TR-04] — Driver App — Task List (Mobile UI)

**Purpose:** Giao diện mobile cho tài xế — xem danh sách lệnh được giao, rõ ràng dễ bấm.

**Layout (Mobile — 390px width):**
```
┌───────────────────────────────────┐
│ ← ECR-LCL Driver      👤 Văn A   │
│ ─────────────────────────────────│
│ HÔM NAY — 01/04/2026             │
│ ─────────────────────────────────│
│ ┌─────────────────────────────┐  │
│ │ LỆNH ĐANG THỰC HIỆN         │  │
│ │ TO-240401                   │  │
│ │ 📦 TCKU123456 [20DC]        │  │
│ │ 🚩 Depot: Cát Lái Depot 1   │  │
│ │ 📦 CFS: KCN Nhơn Trạch      │  │
│ │ ⚓ Cảng Cát Lái             │  │
│ │                              │  │
│ │ T_cutoff: ⏱ 04/04 20:00    │  │
│ │ ─────────────────────────── │  │
│ │ [🗺️ XEM LỘ TRÌNH]           │  │
│ │ [📸 CHECK-IN / GPS]         │  │
│ └─────────────────────────────┘  │
│                                   │
│ LỆNH TIẾP THEO:                  │
│ ┌─────────────────────────────┐  │
│ │ TO-240403  (02/04 08:00)    │  │
│ │ CMAU654321 [40HC]           │  │
│ └─────────────────────────────┘  │
│ ─────────────────────────────────│
│ [🏠 Home] [📋 Lệnh] [🗺️ Map] [👤]│
└───────────────────────────────────┘
```

**UX Requirements:**
- Font size minimum 16px cho body, 20px cho labels quan trọng
- Tất cả tap targets ≥ 48×48px
- High contrast (đảm bảo đọc ngoài trời)
- Offline mode cho xem lệnh (cache localStorage)

---

### [TR-05] — Driver App — Check-in & GPS (Mobile UI)

**Purpose:** Tài xế check-in từng điểm, bắn GPS liên tục, nhận alert khi có nguy cơ trễ.

**Layout (Mobile):**
```
┌───────────────────────────────────┐
│ ← Lệnh TO-240401                 │
│ ─────────────────────────────────│
│                                   │
│ TRẠNG THÁI HIỆN TẠI:             │
│ 📍 Đang di chuyển đến CFS        │
│ GPS: 10.7769°N, 106.7009°E       │
│ Tốc độ: 68 km/h                  │
│                                   │
│ [Mini map — 200px height]        │
│ Route + current position          │
│                                   │
│ ─────────────────────────────────│
│ T_cutoff: 04/04 20:00            │
│ ETA hiện tại: 04/04 17:30 ✅     │
│ ─────────────────────────────────│
│                                   │
│ HÀNH ĐỘNG:                       │
│                                   │
│ ┌─────────────────────────────┐  │
│ │                             │  │
│ │      ✅ ĐÃ ĐẾN DEPOT         │  │
│ │    (Xác nhận lấy vỏ)        │  │
│ │                             │  │
│ └─────────────────────────────┘  │
│                                   │
│ ┌─────────────────────────────┐  │
│ │  📦 ĐÃ ĐÓNG HÀNG TẠI CFS    │  │
│ └─────────────────────────────┘  │
│                                   │
│ ┌─────────────────────────────┐  │
│ │  ⚓ ĐÃ VÀO CỔNG CẢNG         │  │
│ └─────────────────────────────┘  │
│ ─────────────────────────────────│
│ GPS đang phát: ● LIVE (30s/lần)  │
│ [🏠 Home] [📋 Lệnh] [🗺️ Map] [👤]│
└───────────────────────────────────┘
```

**Alert States:**
```
🟡 ALERT VÀNG: Lấy vỏ trễ > 2 giờ so với kế hoạch
  → Full-screen amber banner với vibrate
  → Text: "LẤY VỎ TRỄ 2G 15P — Cần đẩy nhanh tiến độ!"

🔴 ALERT ĐỎ: ETA_thực > T_cutoff × 0.9
  → Pulsing red banner + sound alert
  → Text: "NGUY CƠ TRỄ T_CUTOFF — Liên hệ dispatcher ngay!"
  → [📞 GỌI DISPATCHER] button lớn
```

---

### [TR-06] — Driver App — QR/Barcode Scanner (Mobile UI)

**Purpose:** Scan mã QR Release Order tại Depot và EDO tại Gate-in để ghi timestamp thực tế.

**Layout (Mobile):**
```
┌───────────────────────────────────┐
│ ← Quét mã                        │
│ ─────────────────────────────────│
│ ĐANG QUÉT: Release Order         │
│ (Lấy vỏ tại Depot)               │
│ ─────────────────────────────────│
│                                   │
│ ┌─────────────────────────────┐  │
│ │                             │  │
│ │   [CAMERA VIEWFINDER]       │  │
│ │                             │  │
│ │   ┌──────────────────┐      │  │
│ │   │                  │      │  │
│ │   │   QR Target      │      │  │
│ │   │   Frame (scan)   │      │  │
│ │   │                  │      │  │
│ │   └──────────────────┘      │  │
│ │                             │  │
│ └─────────────────────────────┘  │
│                                   │
│ ─────────────────────────────────│
│ ✅ SCAN THÀNH CÔNG!              │
│ Container: TCKU1234565           │
│ Thời gian: 01/04/2026 09:15:32  │
│ GPS: 10.7769°N, 106.7009°E      │
│ [XÁC NHẬN ĐÃ LẤY VỎ ✅]         │
│ ─────────────────────────────────│
│ [🏠 Home] [📋 Lệnh] [🗺️ Map] [👤]│
└───────────────────────────────────┘
```

**Data:**
```
Scan types:
  1. Release Order (tại Depot) → POST /api/tr/orders/:id/pickup-confirm
     → body: { actual_pickup_ts: timestamp, gps: { lat, lng } }
  2. EDO (Electronic Delivery Order, tại Gate-in)
     → POST /api/tr/orders/:id/gate-in-confirm
     → body: { actual_gate_in_ts: timestamp, gps: { lat, lng } }

Response sau scan:
  SUCCESS → green checkmark + timestamp ghi nhận
  WRONG_CONTAINER → red error "Sai container, vui lòng quét lại"
  ALREADY_SCANNED → amber warning "Đã quét trước đó lúc HH:MM"
```

---

---

## ⚙️ PHẦN 5: ROLE — ADMIN PLATFORM (6 MÀN HÌNH)

**Role color accent:** `#8B5CF6` (Violet)
**Sidebar icon:** ⚙️

---

### [AD-01] — System Master Dashboard & KPI Reports

**Purpose:** Tổng quan toàn hệ thống — KPI research, Pareto Chart, CO₂ tracking, algorithm performance.

**Layout:**
```
┌─────────────────────────────────────────────────────────────────┐
│ HEADER: "Master Dashboard"    [Period ▼] [📊 Export PDF] [🔄 ]  │
├─────────────────────────────────────────────────────────────────┤
│ KPI SCORECARDS (3 rows × 3 cols = 9 cards):                     │
│ ┌────────────┐ ┌────────────┐ ┌────────────────────────────────┐│
│ │Fill Rate TB│ │Giao dịch HT│ │CO₂ Reduction (Sustainability) ││
│ │ 87.3%      │ │ 241 / month│ │ 18,450 kg / month             ││
│ │ Target:80% │ │ ↑ +12%     │ │ = 184.5 tCO₂ YTD              ││
│ ├────────────┤ ├────────────┤ ├────────────────────────────────┤│
│ │T_cutoff OK │ │Rev biên    │ │Matching Engine Runtime         ││
│ │ 96.2%      │ │ ₫12.4T MTD │ │ Avg 47s/batch ✅ (<120s)       ││
│ ├────────────┤ ├────────────┤ ├────────────────────────────────┤│
│ │Pareto rank1│ │NVOCC active│ │SUMO Buffer Time age            ││
│ │ Rate: 78%  │ │ 34 partners│ │ Last update: 2h ago            ││
│ └────────────┘ └────────────┘ └────────────────────────────────┘│
├─────────────────────────────────────────────────────────────────┤
│ CHARTS ROW 2:                                                    │
│ ┌──────────────────────────┐  ┌────────────────────────────────┐│
│ │ Pareto Frontier Chart     │  │ Fill Rate Trend (TimescaleDB)  ││
│ │ (NSGA-II output)          │  │ Line chart — 90 ngày           ││
│ │ X: C_road, Y: C_sea       │  │ Benchmark line: 80%            ││
│ │ Scatter: Pareto rank 1-5  │  │                                ││
│ └──────────────────────────┘  └────────────────────────────────┘│
│ ┌──────────────────────────┐  ┌────────────────────────────────┐│
│ │ Algorithm Performance     │  │ Route Distribution Map         ││
│ │ Runtime per batch (Bar)   │  │ Heatmap: matching density      ││
│ │ PSO vs NSGA-II time       │  │ Depot clusters in Đồng Nai     ││
│ └──────────────────────────┘  └────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

---

### [AD-02] — Dynamic Pricing Board

**Purpose:** Cấu hình và giám sát bảng giá động — ma trận discount, seasonal factors, cập nhật realtime qua WebSocket.

**Layout:**
```
┌─────────────────────────────────────────────────────────────────┐
│ HEADER: "Bảng Giá Động"    ⟳ Cập nhật lúc: 14:00 [🔄 Manual]  │
│ WebSocket: ● LIVE — Push mỗi 15 phút                           │
├─────────────────────────────────────────────────────────────────┤
│ FORMULA DISPLAY:                                                 │
│ C_sea_final = C_sea_floor × (1 − discount_rate) × seasonal_factor│
│ ─────────────────────────────────────────────────────────────── │
│                                                                  │
│ DISCOUNT RATE MATRIX (editable grid):                           │
│                                                                  │
│         │ Days > 7 │ Days 3–7 │ Days 1–3 │ Days < 1 │          │
│ ─────── │──────────│──────────│──────────│──────────│          │
│ Util>80%│    0%    │   -5%    │  -15%    │  -10%    │          │
│ Util60–80│  10%    │   12%    │   15%    │   20%    │          │
│ Util40–60│  20%    │   22%    │   25%    │   30%    │          │
│ Util<40% │  40%    │   42%    │   45%    │   50%    │          │
│ (cells editable, click để sửa)                                   │
├─────────────────────────────────────────────────────────────────┤
│ SEASONAL FACTORS:                                                │
│ Tháng 11–12 (Peak FDI): ×1.15  [Edit]                          │
│ Tháng 2–3 (Sau Tết):    ×0.85  [Edit]                          │
│ Các tháng còn lại:       ×1.00                                  │
├─────────────────────────────────────────────────────────────────┤
│ CURRENT PRICING PREVIEW (realtime):                             │
│ Floor price mẫu: 3,500,000₫ — Container 20DC, Fill 72%, 5 ngày │
│ → discount = 10% → seasonal = 1.00 → Final: 3,150,000₫         │
├─────────────────────────────────────────────────────────────────┤
│ [💾 LƯU CẤU HÌNH] [↩️ Khôi phục mặc định]                      │
└─────────────────────────────────────────────────────────────────┘
```

---

### [AD-03] — Algorithm Engine Monitor

**Purpose:** Giám sát và vận hành Matching Engine — Celery queues, algorithm stats, manual trigger.

**Layout:**
```
┌─────────────────────────────────────────────────────────────────┐
│ HEADER: "Algorithm Engine Monitor"       [🔴/🟢 Engine Status]  │
├──────────────────────────────────┬──────────────────────────────┤
│ LEFT: CELERY QUEUE MONITOR (50%) │ RIGHT: ALGORITHM STATS (50%) │
│ ────────────────────────────────  ────────────────────────────  │
│ Queue: matching_queue            │ Last batch run: 14:15:32     │
│ ● Active tasks: 3                │ Duration: 47.3 seconds       │
│ ● Pending: 8                     │ Containers processed: 23     │
│ ● Failed: 0                      │ Bookings processed: 41       │
│                                  │                              │
│ [Celery Flower embed/iframe]     │ ── L1: 3D Bin Packing ───── │
│ hoặc custom log stream           │ FEASIBLE: 38 / INFEASIBLE: 3 │
│                                  │ Avg runtime: 1.2s            │
│ TASK LOG (console style):        │                              │
│ ┌──────────────────────────────┐ │ ── L2: PSO Routing ─────── │
│ │ 14:15:32 [INFO] Batch start  │ │ Converge iter: 87/200       │
│ │ 14:15:33 [INFO] L1 running   │ │ Avg runtime: 18.4s          │
│ │ 14:15:34 [OK]   L1 done 38F  │ │                              │
│ │ 14:15:35 [INFO] L2+L3 para   │ ── L3: NSGA-II ────────── │
│ │ 14:15:52 [OK]   L2 done      │ │ Pareto solutions: 5         │
│ │ 14:15:52 [OK]   L3 done      │ Avg runtime: 26.7s           │
│ │ 14:15:52 [INFO] Notify 11    │                              │
│ │ 14:15:52 [OK]   Batch done   │ ── MANUAL CONTROLS ─────── │
│ └──────────────────────────────┘ │ [▶️ TRIGGER BATCH NOW]       │
│                                  │ [⚙️ Edit PSO params]         │
│                                  │ [📋 View config JSON]        │
└──────────────────────────────────┴──────────────────────────────┘
```

---

### [AD-04] — SUMO Lookup Table Manager

**Purpose:** Quản lý bảng Buffer Time từ SUMO simulation — redis cache, lịch chạy, kết quả.

**Layout:**
```
┌─────────────────────────────────────────────────────────────────┐
│ HEADER: "SUMO Buffer Time Manager"    Redis TTL: 1 giờ          │
│ Cập nhật gần nhất: 02/04/2026 06:00 (tuần trước)              │
├─────────────────────────────────────────────────────────────────┤
│ BUFFER TIME TABLE (editable):                                    │
│                                                                  │
│ Khung giờ │ Buffer (phút) │ Nguồn    │ Cập nhật gần nhất       │
│ ──────────┼───────────────┼──────────┼─────────────────────── │
│ 00:00–05:59│ +10 phút     │ SUMO     │ 02/04/2026              │
│ 06:00–06:59│ +20 phút     │ SUMO     │ 02/04/2026              │
│ 07:00–09:00│ +65 phút 🔴  │ SUMO     │ 02/04/2026  (Peak)      │
│ 09:01–11:59│ +25 phút     │ SUMO     │ 02/04/2026              │
│ 12:00–16:59│ +15 phút     │ SUMO     │ 02/04/2026              │
│ 17:00–19:00│ +45 phút 🟡  │ SUMO     │ 02/04/2026  (Peak)      │
│ 19:01–23:59│ +10 phút     │ SUMO     │ 02/04/2026              │
│ (cells editable — manual override)                              │
├─────────────────────────────────────────────────────────────────┤
│ VISUALIZATION:                                                   │
│ [Bar chart: giờ (x-axis) vs buffer time phút (y-axis)]          │
│ Màu đỏ = peak hours, màu xanh = off-peak                        │
├─────────────────────────────────────────────────────────────────┤
│ SCHEDULE: SUMO Re-run                                            │
│ Lịch tự động: Chủ nhật 02:00 hàng tuần                         │
│ [▶️ CHẠY SUMO NGAY (mất ~20 phút)]  [📋 Xem log lần trước]     │
│ [💾 LƯU MANUAL OVERRIDE → Redis]                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

### [AD-05] — Rule Engine Configuration

**Purpose:** Cấu hình Chemical Compatibility Matrix và các rule nghiệp vụ — IMDG, HS Code groups.

**Layout:**
```
┌──────────────────────────────────────────────────────────────────┐
│ HEADER: "Rule Engine — Chemical Compatibility"                    │
├──────────────────────────────────────────────────────────────────┤
│ MATRIX VIEW (hover cell = tooltip mô tả):                         │
│                                                                   │
│          │ Food(01-24) │ Acid   │ Base   │ Oxidizer│ Flammable │  │
│ ─────────┼────────────┼────────┼────────┼─────────┼───────────┤  │
│ IMDG 1-9 │    🔴 BLOCK│ 🔴     │ 🔴     │ 🔴      │ 🔴        │  │
│ Acid     │    🔴 BLOCK│ 🟢 OK  │ 🔴     │ 🟡 WARN │ 🔴        │  │
│ Base     │    🔴 BLOCK│ 🔴     │ 🟢 OK  │ 🟡 WARN │ 🔴        │  │
│ Oxidizer │    🔴 BLOCK│ 🟡 WARN│ 🟡 WARN│ 🟢 OK   │ 🔴 BLOCK  │  │
│ Flammable│    🔴 BLOCK│ 🔴     │ 🔴     │ 🔴 BLOCK│ 🟢 OK     │  │
│ Dry cargo│    🟢 OK   │ 🟢 OK  │ 🟢 OK  │ 🟢 OK   │ 🟢 OK     │  │
│                                                                   │
│ Legend: 🔴 BLOCK (auto-reject) | 🟡 WARN (cảnh báo) | 🟢 OK      │
├──────────────────────────────────────────────────────────────────┤
│ CAT LÁI IMDG BAN (từ 01/07/2024):                                │
│ ● Cảng xuất = VNCLI → Block TẤT CẢ IMDG Class 1-9              │
│ [Toggle: ON ● / OFF] [Effective date: 01/07/2024]               │
├──────────────────────────────────────────────────────────────────┤
│ HS CODE → IMDG CLASS MAPPING:                                     │
│ [Data Table + Search] [+ Thêm mapping] [Import CSV]              │
│ HS Code    │ Mô tả       │ IMDG Class │ Group       │ Actions   │
│ 28070010   │ H2SO4       │ 8          │ Acid        │ [Sửa] [Xóa]│
│ 28151100   │ NaOH        │ 8          │ Base        │ [Sửa] [Xóa]│
└──────────────────────────────────────────────────────────────────┘
```

---

### [AD-06] — Audit Log & User Management

**Purpose:** Quản lý tài khoản B2B, phê duyệt đăng ký, tra cứu audit log.

**Layout:**
```
┌──────────────────┬───────────────────────────────────────────────┐
│ TABS:            │ TAB CONTENT                                    │
│ 👥 Người dùng   │                                                │
│ 🏢 Tổ chức      │ TAB: Người dùng                               │
│ 📋 Audit Log    │ [🔍 Tìm] [Role ▼] [Status ▼]                  │
│ ⏳ Chờ duyệt    │ Table: Tên | Email | Role | Org | Status | [⋯]│
│                  │ → [Duyệt] [Khóa] [Reset PW] [View log]       │
│                  │                                                │
│                  │ TAB: Chờ duyệt (badge count)                 │
│                  │ Cards: Tên tổ chức | Role | MST | Ngày nộp   │
│                  │ [✅ DUYỆT] [❌ TỪ CHỐI + lý do]              │
│                  │                                                │
│                  │ TAB: Audit Log                                │
│                  │ [🔍 Filter] [Export]                          │
│                  │ Table: Thời gian | User | IP | Action | Detail│
│                  │ 14:15:32 | admin@ecr | 1.2.3.4 | LOGIN | OK  │
│                  │ 14:10:01 | sl@haian | 5.6.7.8 | CONFIRM_MATCH│
│                  │ ...                                            │
└──────────────────┴───────────────────────────────────────────────┘
```

---

---

## 📐 COMPONENT INTERACTION PATTERNS

### Pattern 1: Countdown Timer (Critical — xuất hiện trong SL-05, NV-05)
```jsx
// Behavior:
// > 10 min: amber color, smooth countdown
// 5-10 min: amber + pulse animation
// < 5 min:  RED + fast pulse + sound ping every 30s
// 0:00:    auto-reject, redirect to Matching Inbox

<CountdownTimer
  expiresAt={timelock.expires_at}
  onExpire={() => handleAutoReject()}
  warningThreshold={600}  // 10 minutes in seconds
  criticalThreshold={300} // 5 minutes
/>
```

### Pattern 2: Three.js Container Visualizer (SL-05, NV-05)
```jsx
// Render pipeline:
// 1. Load layout_json from API
// 2. Create BoxGeometry per item (L, W, H in scene units)
// 3. Color by color_group (HS category)
// 4. Animate: items fade-in sequentially (stagger 50ms each)
// 5. Container wireframe: always visible
// 6. Camera: OrbitControls, initial front-45deg view
// 7. Labels: floating text (TextGeometry or CSS2DRenderer)

// Performance: Use instanced mesh for >50 items
// Mobile: Reduce geometry segments
```

### Pattern 3: GPS Tracking Update (TR-01, NV-06)
```jsx
// WebSocket handler:
ws.onmessage = (event) => {
  const { truck_plate, lat, lng, speed, eta_gate_in } = JSON.parse(event.data);
  
  // Update Leaflet marker position
  markers[truck_plate].setLatLng([lat, lng]);
  
  // Check T_cutoff breach
  const slack = (t_cutoff - eta_gate_in) / 60000; // minutes
  if (slack < t_cutoff * 0.1) setAlertLevel('CRITICAL');
  else if (slack < t_cutoff * 0.2) setAlertLevel('WARNING');
  
  // Append to GPS track polyline
  gpsTrack[truck_plate].addLatLng([lat, lng]);
};
```

### Pattern 4: HS Code Chemical Validator (NV-04)
```jsx
// On HS Code input (debounce 500ms):
const validateHSCode = async (hsCode, existingItems) => {
  const { imdg_class, imdg_group, is_food } = await api.get(`/hs-code/${hsCode}`);
  
  // Rule B: Cát Lái IMDG ban
  if (imdg_class && originPort === 'VNCLI') {
    return { blocked: true, reason: 'IMDG_BLOCKED_CAT_LAI' };
  }
  
  // Rule A: IMDG + Food incompatibility
  if (imdg_class && existingItems.some(i => i.is_food)) {
    return { warning: true, reason: 'IMDG_WITH_FOOD' };
  }
  
  // Rule C: Chemical matrix cross-check
  return checkCompatibilityMatrix(imdg_group, existingItems);
};
```

---

## 🏗️ IMPLEMENTATION NOTES

### Tech Stack Imports (CDN-ready cho Artifacts)
```html
<!-- Charts -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.0/chart.umd.min.js">

<!-- Maps -->
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/leaflet.min.css">
<script src="https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/leaflet.min.js">

<!-- 3D -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js">

<!-- Fonts (Google Fonts) -->
<link href="https://fonts.googleapis.com/css2?family=Syne:wght@400;600;700;800&family=IBM+Plex+Sans:wght@300;400;500;600&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet">
```

### React Component Architecture
```
src/
├── components/
│   ├── common/           ← Shared components (StatusBadge, CountdownTimer, etc.)
│   ├── charts/           ← Chart.js wrappers
│   ├── map/              ← Leaflet components
│   ├── three/            ← Three.js container visualizer
│   └── forms/            ← Form components with validation
├── pages/
│   ├── auth/             ← COM-01, COM-02
│   ├── shipping-line/    ← SL-01 to SL-07
│   ├── nvocc/            ← NV-01 to NV-08
│   ├── trucking/         ← TR-01 to TR-06
│   └── admin/            ← AD-01 to AD-06
├── hooks/
│   ├── useWebSocket.ts   ← WS connection & reconnect
│   ├── useCountdown.ts   ← Timer logic
│   └── useGPS.ts         ← GPS tracking state
└── services/
    ├── api.ts            ← Axios instance + JWT interceptors
    └── websocket.ts      ← WS singleton
```

---

*Tài liệu này là Master UI Prompt hoàn chỉnh. Mỗi màn hình có thể được implement độc lập dưới dạng React component hoặc HTML Artifact, tuân thủ Design System Foundation ở đầu tài liệu.*

---
**Màn hình tổng cộng:** 30 screens | **Roles:** 4 | **Phiên bản:** v2.0 | **Cập nhật:** 2026-03-10
