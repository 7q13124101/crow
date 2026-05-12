Dưới đây là báo cáo chi tiết về cấu trúc, kiến trúc và công nghệ của dự án Crow (AI Agent Full-stack) để bạn phục vụ công việc phân tích và thiết kế hệ thống mới.

---

## 1. Tổng quan kiến trúc hệ thống

Dự án được thiết kế theo kiến trúc Micro-Monolith (Monolith hiện đại), tách biệt rõ ràng giữa Frontend, Backend và các dịch vụ bổ trợ qua Docker.

- **Kiểu kiến trúc:** N-Tier Architecture (Phân lớp).
- **Giao tiếp:** REST API (cho dữ liệu tĩnh) & WebSockets (cho streaming AI).
- **Xử lý bất đồng bộ:** Sử dụng Celery + Redis cho các tác vụ nặng (xử lý file, sync dữ liệu).
- **Quan sát (Observability):** Tích hợp sâu với Logfire (Pydantic) để tracing và monitoring.

---

## 2. Cấu trúc thư mục (Directory Structure)

```text
crow/
├── backend/                # FastAPI Application
│   ├── alembic/            # Database Migrations
│   ├── app/
│   │   ├── agents/         # AI Logic (PydanticAI, Prompts, Tools)
│   │   ├── api/            # API Endpoints (v1) & Dependencies
│   │   ├── core/           # Config, Security, Middleware, Logging
│   │   ├── db/             # SQLAlchemy Models & Session management
│   │   ├── repositories/   # Data Access Layer (CRUD)
│   │   ├── services/       # Business Logic Layer
│   │   ├── schemas/        # Pydantic Models (Request/Response)
│   │   └── worker/         # Celery Task definitions
│   ├── cli/                # Management CLI (crow commands)
│   └── tests/              # Pytest suite
├── frontend/               # Next.js Application (App Router)
│   ├── src/
│   │   ├── app/            # Pages & Layouts
│   │   ├── components/     # UI Components (RadixUI, Shadcn-like)
│   │   ├── hooks/          # Custom React Hooks (React Query)
│   │   ├── lib/            # Utilities (API clients, constants)
│   │   └── stores/         # State management (Zustand)
│   ├── messages/           # I18n (Internationalization)
│   └── e2e/                # Playwright End-to-End tests
├── kubernetes/             # K8s Deployment manifests
├── docker-compose.yml      # Local development setup
└── Makefile                # Automation shortcuts
```

---

## 3. Stack Công nghệ & Thư viện chính

**Backend (Python 3.12+)**

- **Framework:** FastAPI (Async).
- **AI Engine:** PydanticAI (Framework mới của Pydantic để build Agent mạnh mẽ).
- **Database ORM:** SQLAlchemy 2.0 (Async) + PostgreSQL.
- **Migration:** Alembic.
- **Caching/Task Queue:** Redis & Celery.
- **Dependency Management:** uv (Công cụ quản lý package siêu nhanh).
- **Observability:** Logfire (Tracing), Prometheus (Metrics), Sentry (Error Tracking).

**Frontend (TypeScript)**

- **Framework:** Next.js 15 (App Router) + React 19.
- **Runtime:** Bun (Package manager & Bundler).
- **Styling:** Tailwind CSS 4.0.
- **State Management:** Zustand.
- **Data Fetching:** Tanstack React Query.
- **UI Components:** Radix UI (Unstyled primitives) + Lucide React (Icons).
- **I18n:** next-intl.

---

## 4. Các luồng xử lý cốt lõi (Core Workflows)

**A. Luồng AI Chat (WebSocket Streaming)**

1. FE kết nối tới `ws://.../api/v1/ws/agent`.
2. Backend sử dụng AssistantAgent (bao bọc PydanticAI Agent).
3. Agent sử dụng OllamaModel (hoặc Gemini/OpenAI) để xử lý.
4. Dữ liệu trả về qua WebSocket theo dạng Events: `text_delta` -> `tool_call` -> `final_result`.
5. Tin nhắn được lưu vào Postgres ngay sau khi hoàn tất stream.

**B. Luồng RAG (Retrieval Augmented Generation)**

- **Mức độ 1 (Hiện tại):** Chat-with-file. Text được trích xuất từ file (PDF, Docx bằng PyMuPDF, python-docx) và đẩy thẳng vào System Prompt.
- **Mức độ 2 (Placeholder):** Có sẵn cấu trúc cho Vector DB (ChromaDB) nhưng đang chờ cài đặt logic tìm kiếm ngữ nghĩa (Semantic Search).

**C. Luồng Security & Auth**

- **Auth:** JWT (Access Token + Refresh Token).
- **Role-based Access:** Phân quyền Admin và User thông qua Middleware.
- **File Storage:** Files được lưu trữ theo cấu trúc `media/{user_id}/` để đảm bảo cô lập dữ liệu.

---

## 5. Điểm mạnh và lưu ý thiết kế

- **Type Safety:** Sử dụng Pydantic v2 xuyên suốt từ API cho đến AI Agent.
- **Scalability:** Tách biệt Worker (Celery) giúp xử lý các tác vụ AI/File nặng mà không làm nghẽn API.
- **Modern DevX:** Sử dụng uv, bun, ruff giúp tốc độ phát triển và CI cực nhanh.
- **Infrastructure:** Đã sẵn sàng cho Kubernetes (có đầy đủ Kustomize manifests).

> Dữ liệu này sẽ giúp bạn có cái nhìn tổng thể để thiết kế một hệ thống tương tự hoặc nâng cấp hệ thống hiện tại. Bạn có muốn đi sâu vào chi tiết code của một Module cụ thể nào không?
