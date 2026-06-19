# PROJECT_CONTEXT.md

> Tài liệu này dành cho AI agent đọc để hiểu nhanh toàn bộ dự án trước khi thực hiện bất kỳ tác vụ nào.
> Cập nhật lần cuối: 2026-06-17

---

## 1. Tổng quan dự án

**Tên sản phẩm**: SmartHandover AI (PHWM)

**Mô tả ngắn**:  
Hệ thống quản lý bàn giao căn hộ và bảo hành tự động tích hợp AI (Gemini API + Computer Vision) cho **Vinhomes**. Hệ thống số hóa toàn bộ quy trình: Cư dân chụp ảnh lỗi → AI tự động phân tích, phân loại và chuẩn hóa thuật ngữ → Tạo phiếu sửa chữa → Phân công thợ → Xuất báo cáo PDF.

**Vấn đề giải quyết**:  
- Quy trình ghi nhận lỗi bàn giao căn hộ hiện tại hoàn toàn thủ công: chụp ảnh, tự phân tích, gõ biên bản
- Thuật ngữ không đồng nhất giữa các nhân viên kiểm định
- Mất hàng giờ cho công việc hành chính thay vì chuyên môn

**Mục tiêu**:
- Rút ngắn thời gian xử lý lỗi từ **5 ngày xuống dưới 2 ngày**
- Giảm **≥60%** thời gian làm báo cáo so với thủ công
- Độ chính xác nhận diện lỗi **>90%** (Precision & Recall)
- Độ chính xác thuật ngữ chuyên ngành **>95%**
- Latency xử lý ảnh **<5 giây/ảnh**

---

## 2. Kiến trúc hệ thống

**Tech stack** (ADR-0002):
- **Frontend**: Vite + React 19 + TypeScript (`strict: true`)
- **AI**: Gemini API (Vision + Text), provider-agnostic interface
- **Database**: Supabase (PostgreSQL + Storage + Auth + Realtime) — ADR-0008
- **PDF Export**: Client-side (ADR-0006)
- **Testing**: Vitest (ADR-0003)

**Kiến trúc tổng thể** (ADR-0001): Monolith frontend-first — toàn bộ logic nằm trong một React app, không microservices.

**Cấu trúc thư mục mã nguồn** (`/agent-management/src`):
```
/src
  /types          ← TypeScript types, enums, constants (barrel export tại /types/index.ts)
  /domain         ← Business logic thuần (State Machine, SLA, RBAC) — KHÔNG import React
  /data           ← Mock data, seed JSON, repository layer
  /services       ← AI service interface + Gemini implementation + Supabase client
  /pages          ← 8 màn hình (7 MVP + 1 Standard)
  /components     ← Shared UI components
  /context        ← React context providers
  /images         ← Static assets
  App.tsx         ← App shell với routing
  main.tsx        ← Entry point
  styles.css      ← Global styles (CSS variables, no Tailwind)
```

---

## 3. Các module chức năng chính

| Module | Mô tả | Task liên quan |
|---|---|---|
| **Đăng nhập** | Xác thực 3 vai trò: Cư dân (User), BQL (Manager), Thợ (Worker) | PHWM-001 |
| **Báo lỗi + AI** | Upload ảnh → Gemini Vision phân tích → sinh mã ERR-xxx | PHWM-003 |
| **Danh sách lỗi** | Cư dân xem lỗi của mình, ký xác nhận/từ chối | PHWM-004 |
| **Work Order** | Worker xem và xử lý công việc được phân công | PHWM-005, PHWM-006 |
| **Dashboard BQL** | KPI: SLA, Reopen Rate, FTFR, AI Analytics | PHWM-007 |
| **Xuất PDF** | Biên bản sửa chữa + KPI report | PHWM-008 |
| **Notification** | In-app theo Notification Matrix (7 sự kiện × 3 vai trò) | PHWM-009 |
| **Bảo hành** | AI Predictive Maintenance, lịch sử bảo hành (Standard) | PHWM-010 |
| **RAG Glossary** | Chuẩn hóa thuật ngữ chuyên ngành qua LLM + RAG | PHWM-017 |
| **Object Detection** | Khoanh vùng lỗi trên ảnh (bounding box) | PHWM-018 |

---

## 4. Vai trò người dùng

| Vai trò | Tên | Quyền chính |
|---|---|---|
| **Cư dân (User)** | Người sở hữu căn hộ | Báo lỗi, xem tiến độ, ký xác nhận |
| **BQL (Manager)** | Ban quản lý tòa nhà | Phê duyệt, phân công thợ, xem Dashboard, xuất PDF |
| **Thợ (Worker)** | Nhân viên kỹ thuật sửa chữa | Xem/xử lý Work Order, upload minh chứng hoàn thành |

---

## 5. Trạng thái hiện tại (Scaffold)

Dự án đang ở giai đoạn **scaffold** — cấu trúc đã được dựng, chưa implement đầy đủ:

- ✅ Vite + React 19 + TypeScript setup
- ✅ 8 page placeholders (7 MVP + 1 Standard)
- ✅ Domain logic: Ticket State Machine, SLA Calculator, RBAC
- ✅ AI Service Interface (mock implementation)
- ✅ Seed data cho demo (3 users, property hierarchy, 3 sample defects)
- ✅ 4 test suites (state machine, SLA, RBAC, AI service)
- ✅ Supabase schema + migrations (ADR-0008)
- 🔲 Tích hợp Gemini API thực
- 🔲 Kết nối Supabase thực (đang dùng mock)
- 🔲 Object Detection thực
- 🔲 RAG Engine thực

---

## 6. Sprint & Backlog hiện tại

### Sprint Plan (3 sprint x ~1 tuần)

| Sprint | Mục tiêu | Trạng thái |
|---|---|---|
| **Sprint 1** | MVP cốt lõi: Login, Data Model, Defect Report + AI, Defect List, Work Order List | Planned |
| **Sprint 2** | MVP hoàn thiện: Worker Detail, Dashboard BQL, PDF Report, Notification | Planned |
| **Sprint 3** | Standard: Bảo hành & Predictive Maintenance | Planned |

### Backlog tóm tắt (PB-001 → PB-010)

| ID | Hạng mục | Sprint | Trạng thái |
|---|---|---|---|
| PB-001 | Đăng nhập 3 vai trò | S1 | Sprint Ready |
| PB-002 | Data Model & Mock Data | S1 | Sprint Ready |
| PB-003 | Báo lỗi & AI phân tích | S1 | Sprint Ready |
| PB-004 | Danh sách lỗi của tôi | S1 | Sprint Ready |
| PB-005 | Danh sách Work Order | S1 | Sprint Ready |
| PB-006 | Chi tiết khắc phục | S2 | Sprint Ready |
| PB-007 | Dashboard BQL | S2 | Sprint Ready |
| PB-008 | Xuất báo cáo PDF | S2 | Sprint Ready |
| PB-009 | Notification In-app | S2 | Sprint Ready |
| PB-010 | Bảo hành & Giám sát lỗi lặp | S3 | Chưa làm |

---

## 7. Tasks hiện có (PHWM-001 → PHWM-023)

Tất cả tasks nằm trong `/agent-management/tasks/`. ID format: `PHWM-NNN`.

**Sprint 1 tasks**: PHWM-001 (Login), PHWM-002 (Data Model), PHWM-003 (Defect Report + AI), PHWM-004 (Defect List), PHWM-005 (Work Order List)

**Sprint 2 tasks**: PHWM-006 (Work Order Detail), PHWM-007 (Dashboard), PHWM-008 (PDF Report), PHWM-009 (Notification)

**Sprint 3 tasks**: PHWM-010 (Warranty)

**Extended tasks** (backlog mở rộng): PHWM-011 (DB Persistence), PHWM-012 (Auth RBAC), PHWM-013 (Image Storage), PHWM-014 (Account Management), PHWM-015 (Apartment Ownership), PHWM-016 (AI Transparency), PHWM-017 (RAG Glossary), PHWM-018 (Object Detection), PHWM-019 (Async AI Batch), PHWM-020 (Multichannel Notifications), PHWM-021 (SLA Escalation), PHWM-022 (Mobile Camera), PHWM-023 (Offline Support)

---

## 8. Quy tắc coding (ADR-0007)

| Quy tắc | Chi tiết |
|---|---|
| **Naming** | PascalCase (components, interfaces, enums), camelCase (functions/vars), SCREAMING_SNAKE (constants), kebab-case (files/CSS) |
| **TypeScript** | `strict: true`, không dùng `any`, dùng `enum` cho domain values, `interface` cho entities |
| **React** | Function declaration + `export default`, props qua `interface`, tối đa 150 lines/component |
| **CSS** | CSS variables cho design tokens, plain CSS, scope `.page-name`, không inline styles |
| **Files** | Pages → `/pages/Name/index.tsx`, domain logic → `/domain/` (không import React), barrel exports → `/types/index.ts` |
| **Imports** | Thứ tự: React → third-party → domain/services → components → data (ngăn bằng blank line) |
| **Comments** | Header nguồn sự thật, JSDoc cho exports, `// TODO: PHWM-XXX`, tiếng Anh |
| **Tests** | Mirror `src/`, `describe/it`, behavior-focused, 1 assert/test |
| **Errors** | Không silent catch, message rõ ràng, trả `null` cho expected failures |

---

## 9. Workflow làm việc cho AI agent

Khi nhận một task mới, đọc theo thứ tự sau:

1. **`AGENTS.md`** — `/agent-management/AGENTS.md` (luật chơi tổng thể)
2. **Task file** — `/agent-management/tasks/PHWM-NNN-name.md`
3. **Spec chính** — `/agent-management/specs/phwm-spec.md`
4. **ADR liên quan** — `/agent-management/adrs/ADR-XXXX-*.md`
5. **Sprint plan** — `/agent-management/planning/sprints/sprint-N.md`
6. **Source code** — `/agent-management/src/`
7. **Tests** — `/agent-management/tests/`

### Git workflow

```
Branch:  feature/<task-id>-short-name
Commit:  PHWM-001: implement login flow with 3 roles
PR → merge → main
```

### Definition of Done

- [ ] Implementation khớp spec
- [ ] ADR liên quan được tôn trọng
- [ ] Coding conventions (ADR-0007) được tuân thủ
- [ ] Tests pass
- [ ] Docs/contracts được cập nhật
- [ ] PR đã được review và merge
- [ ] Issue đã được đóng

---

## 10. Chạy dự án

```bash
# Cài đặt (tại /agent-management)
npm install

# Dev server
npm run dev

# Build production
npm run build

# Chạy tests
npm run test

# Type check
npm run check
```

**Yêu cầu**: Node.js 22+, npm 10+

**Environment**: Copy `/agent-management/.env.local.example` → `/agent-management/.env.local` và điền các API keys (Gemini, Supabase).

---

## 11. Tài liệu tham khảo

| Tài liệu | Đường dẫn | Mô tả |
|---|---|---|
| AGENTS.md | `/agent-management/AGENTS.md` | Luật chơi đầy đủ cho AI agent |
| Spec đầy đủ | `/agent-management/specs/phwm-spec.md` | **Nguồn sự thật chính** |
| RFP gốc | `/agent-management/specs/phwm-rfp.md` | §1–§22 yêu cầu từ Vinhomes |
| ADRs | `/agent-management/adrs/` | 8 quyết định kiến trúc |
| Backlog | `/agent-management/planning/backlog.md` | Danh sách tính năng ưu tiên |
| Coding Rules | `/agent-management/docs/coding-rules.md` | Quick reference quy ước coding |
| Brief gốc | `/brief.md` | Tóm tắt bài toán ban đầu |
| PRD | `/prd.md` | Product Requirements Document |
