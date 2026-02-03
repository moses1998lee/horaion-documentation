# 05 - Frontend Specifications

> **Genesis Workforce Management Platform - Frontend Specifications**

## 1. Runtime Environment
- **Node Version**: Latest LTS
- **Package Manager**: pnpm
- **Build Tool**: Vite 6.3.5 (SWC)

## 2. Core Dependencies
- **Framework**: React 19.1.0
- **Routing**: @tanstack/react-router
- **State Management & Data Fetching**: @tanstack/react-query
- **HTTP Client**: Axios

## 3. UI/UX Stack
- **Styling Engine**: Tailwind CSS 4.x (via `@tailwindcss/vite`)
- **Component Primitives**: Radix UI (@radix-ui/react-*) & Headless UI
- **Icons**: Lucide React
- **Utility Libraries**: 
  - `clsx` & `tailwind-merge` (Class composition)
  - `class-variance-authority` (Variant management)
- **Toasts**: Sonner

## 4. Forms & Validation
- **Form Management**: React Hook Form
- **Validation Schema**: Zod
- **Resolver**: @hookform/resolvers

## 5. Development Dependencies
- **Language**: TypeScript 5.8.3
- **Linting**: ESLint 9.x (with typescript-eslint)
- **Formatting**: Prettier
- **Sorting**: @trivago/prettier-plugin-sort-imports

## 6. Utilities
- **Date Handling**: date-fns, dayjs, react-day-picker
- **Internationalization**: i18next, react-i18next
- **Analytics**: @vercel/analytics

## 7. Deployment
- **Platform**: Vercel
- **Config**: `vercel.json` present in root

## 8. Directory Structure
```
genesis-app/
├── src/
│   ├── components/         # Reusable UI components (Radix/Headless)
│   ├── hooks/              # Custom React hooks
│   ├── lib/                # Utility libraries (utils, cn)
│   ├── pages/              # Page components
│   ├── provider/           # Context providers
│   ├── routes/             # TanStack Router definitions
│   ├── services/           # API service modules (Axios)
│   ├── types/              # TypeScript type definitions
│   └── utils/              # Helper functions
├── public/                 # Static assets
├── index.html              # Entry point
├── vite.config.ts          # Vite configuration
├── tsconfig.json           # TypeScript configuration
└── package.json            # Dependencies
```

## 9. Environment Configuration
Required environment variables in `.env`:

| Variable | Description | Example |
|----------|-------------|---------|
| `VITE_API_URL` | Base URL for the Genesis API | `https://api.genesis.com` |

## 10. API Integration
- **Client**: Axios instance configured in `src/services/api.ts`
- **Configuration**:
  - `baseURL`: Loaded from `VITE_API_URL`
  - `timeout`: 100s
  - `headers`: `Content-Type: application/json`
- **Pattern**: Service modules (e.g., `todo.ts`) export methods that use this shared instance.
