# - Frontend Specifications

> **Horaion Workforce Management Platform - Frontend Specifications**

## 5.1 Runtime Environment {#id-5.1-runtime-environment}
- **Node Version**: Latest LTS
- **Package Manager**: pnpm
- **Build Tool**: Vite 6.3.5 (SWC)

## 5.2 Core Dependencies {#id-5.2-core-dependencies}
- **Framework**: React 19.1.0
- **Routing**: @tanstack/react-router
- **State Management & Data Fetching**: @tanstack/react-query
- **HTTP Client**: Axios

## 5.3 UI/UX Stack {#id-5.3-ui-ux-stack}
- **Styling Engine**: Tailwind CSS 4.x (via `@tailwindcss/vite`)
- **Component Primitives**: Radix UI (@radix-ui/react-*) & Headless UI
- **Icons**: Lucide React
- **Utility Libraries**: 
  - `clsx` & `tailwind-merge` (Class composition)
  - `class-variance-authority` (Variant management)
- **Toasts**: Sonner

## 5.4 Forms & Validation {#id-5.4-forms-validation}
- **Form Management**: React Hook Form
- **Validation Schema**: Zod
- **Resolver**: @hookform/resolvers

## 5.5 Development Dependencies {#id-5.5-development-dependencies}
- **Language**: TypeScript 5.8.3
- **Linting**: ESLint 9.x (with typescript-eslint)
- **Formatting**: Prettier
- **Sorting**: @trivago/prettier-plugin-sort-imports

## 5.6 Utilities {#id-5.6-utilities}
- **Date Handling**: date-fns, dayjs, react-day-picker
- **Internationalization**: i18next, react-i18next
- **Analytics**: @vercel/analytics

## 5.7 Deployment {#id-5.7-deployment}
- **Platform**: Vercel
- **Config**: `vercel.json` present in root

## 5.8 Directory Structure {#id-5.8-directory-structure}
```
horaion-app/
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

## 5.9 Environment Configuration {#id-5.9-environment-configuration}
Required environment variables in `.env`:

| Variable | Description | Example |
|----------|-------------|---------|
| `VITE_API_URL` | Base URL for the Horaion API | `https://api.horaion.com` |

## 5.10 API Integration {#id-5.10-api-integration}
- **Client**: Axios instance configured in `src/services/api.ts`
- **Configuration**:
  - `baseURL`: Loaded from `VITE_API_URL`
  - `timeout`: 100s
  - `headers`: `Content-Type: application/json`
- **Pattern**: Service modules (e.g., `todo.ts`) export methods that use this shared instance.
