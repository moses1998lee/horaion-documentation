# 03 Frontend & ETL Engineering Specifications

## 5. Frontend Engineering Specifications

### 5.1 Runtime Environment
*   **Runtime**: Browser-based Single Page Application (SPA).
*   **Build System**: Vite 6+ (Lightning fast HMR and bundling).
*   **Language**: TypeScript 5.8+.
*   **Node.js**: Developed on Node v20+, deployed as static assets.

### 5.2 Application Architecture
The frontend is built using **React 19** and follows a modular, feature-based architecture.

*   **Routing**: **TanStack Router** (File-based routing generated from `routeTree.gen.ts`).
*   **Data Fetching**: **TanStack Query** (React Query) v5 for managing server state, caching, and background updates.
*   **Styling**: **Tailwind CSS v4** with a Shadcn/UI-based component library.
*   **Forms**: `react-hook-form` with `zod` schema validation.
*   **Date Handling**: `date-fns` and `dayjs`.

### 5.3 Core Configuration
*   **Vite Plugins**:
    *   `@tanstack/router-plugin/vite`: Logic for auto-generating routes.
    *   `@vitejs/plugin-react-swc`: High-performance React compilation.
    *   `@tailwindcss/vite`: Tailwind v4 integration.
*   **Path Aliases**: `@/` maps to `./src/` for clean imports.

### 5.4 State Management & Data Flow
*   **Server State (API Data)**: Managed efficiently by TanStack Query.
    *   Automatic cache invalidation and refetching.
    *   Optimistic updates for smoother UI interactions.
*   **Client State (Form/UI)**:
    *   Local component state (`useState`, `useReducer`).
    *   Form state via `react-hook-form` contexts.
    *   Global UI state (e.g., Toasts) via Providers (e.g., `sonner` for notifications).

### 5.5 UI/UX Functional Requirements
*   **Component Library**: Reusable UI components located in `src/components/ui` (likely Shadcn).
*   **Responsiveness**: Mobile-first design principles using Tailwind's utility classes.
*   **Theme**: Supports Light/Dark modes (`next-themes`).

---

## 5.9 ETL Service Engineering Specifications
The **Genesis ETL Service** is a specialized microservice designed to handle high-compute tasks, specifically the parsing and transformation of complex Excel files.

### 5.9.1 Runtime Environment
*   **Language**: Python 3.10+
*   **Framework**: Flask 2.3.3
*   **Server**: Gunicorn (Production) / Werkzeug (Dev)
*   **Database**: MongoDB (via `flask-pymongo`)

### 5.9.2 Architecture & Responsibilities
This service acts as an "Anti-Corruption Layer" for data ingestion. It takes messy human-generated data (Excel) and sanitizes it before it reaches the core system.

*   **Input**: Raw Excel/CSV files via HTTP Upload.
*   **Processing**:
    1.  **Parsing**: Uses **Pandas** to read Excel files into DataFrames.
    2.  **Dynamic Mapping**: Uses `ExcelModelFactory` to map Excel columns to internal data structures dynamically.
    3.  **Expansion**: Logic to parse unstructured `ADDITIONAL_FIELDS` (JSON-like strings) into distinct columns.
    4.  **Enrichment**: Logic to infer fields (e.g., `AVAILABILITY` based on `IS_PART_TIME`).
*   **Output**: Cleaned JSON documents stored in **MongoDB**.

### 5.9.3 Data Flow: Excel Upload
1.  **Endpoint**: `POST /api/excel/upload`
2.  **Logic**:
    *   Validates file extension.
    *   Saves file to local `resource_dir`.
    *   Reads file using `ExcelModelFactory`.
    *   **Upsert Strategy**:
        *   Keys: `EMAIL_ADDRESS` + `PHONE_NUMBER`.
        *   Action: Updates existing records or inserts new ones.
    *   **Storage**: Saves processed records to `mongo.db.employee`.
3.  **Response**: Returns success status and file statistics (row count, column names).

### 5.9.4 Key Libraries
*   **Pandas**: Core data manipulation engine.
*   **OpenPyXL**: Excel file reading/writing backend.
*   **PyMongo**: MongoDB driver.
*   **Flask-CORS**: Handles Cross-Origin requests from the React Frontend.
