# PROMPTS: TASK-007 - Frontend React Application Setup

# PROMPT 1: React Application Foundation with TypeScript and Vite

Create a new React 18+ application using Vite as the build tool with TypeScript configuration optimized for a healthcare management system. Initialize the project with proper dependencies, basic configuration, and ensure the development server can start successfully.

## PROJECT CONTEXT

**Tech Stack Requirements:**
- Frontend: React with TypeScript
- Build Tool: Vite for fast development and optimized builds
- Architecture: Component-based architecture with feature module organization
- Package Manager: npm (preferred) or yarn

**Project Structure Philosophy:**
- Feature-based folder structure supporting domain-driven design
- Clear separation of concerns between UI, business logic, and data
- Provider pattern for global state and context management
- Hook-based logic sharing and reusability

## EPIC CONTEXT

This task is part of **EPIC-000: Project Setup and Foundation** which establishes the core technical infrastructure for the MedFlow healthcare management system. The React application will serve as the foundation for all frontend feature development including patient management, appointment scheduling, and clinical workflows.

## TASK Context

**Goal:** Initialize a complete React application with TypeScript, Vite build system, and foundational architecture supporting the MedFlow healthcare management system requirements.

**Definition of Done:** React application runs in development mode, builds successfully for production, includes proper TypeScript configuration, development tools working, and basic project structure ready for feature development.

**Priority:** HIGH - This is a foundational task required by all frontend development.

## FUNCTIONAL REQUIREMENTS

1. **React Application Foundation**
   - React 18+ with TypeScript support
   - Fast refresh and hot module replacement working
   - Production build capability
   - Environment variable support

2. **Vite Build System**
   - Development server with port 3000
   - Source maps for debugging
   - Asset optimization for production
   - Code splitting configuration

3. **TypeScript Configuration**
   - Strict mode enabled
   - Path aliases for clean imports
   - Type checking integration
   - React JSX support

## TECHNICAL REQUIREMENTS

**Performance:**
- Development server startup time under 5 seconds
- Hot reload response time under 1 second
- Production build completion under 2 minutes

**Vite Configuration Requirements:**
```typescript
// Expected vite.config.ts structure
export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@/components': path.resolve(__dirname, './src/components'),
      '@/features': path.resolve(__dirname, './src/features'),
      '@/lib': path.resolve(__dirname, './src/lib'),
    },
  },
  server: {
    port: 3000,
    open: true,
  },
  build: {
    sourcemap: true,
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
        },
      },
    },
  },
});
```

**TypeScript Configuration Requirements:**
- Target ES2020
- Strict mode enabled
- JSX as react-jsx
- Module resolution as node
- Base URL and path mapping configured

## TESTING REQUIREMENTS

**Manual Tests:**
- Verify `npm run dev` starts development server on port 3000
- Verify application loads with React default content
- Verify `npm run build` creates production bundle without errors
- Verify `npm run preview` serves production build successfully
- Verify TypeScript compilation with `npm run type-check`

**Integration Tests:**
- Build process completes successfully in clean environment
- All path aliases resolve correctly
- Source maps are generated for development
- Production bundle includes proper code splitting

## DOCUMENTATION REQUIREMENTS

Create or update:
- README.md with setup instructions and available scripts
- Package.json with proper scripts and metadata
- Basic project structure documentation

## VALIDATION CRITERIA

- [ ] React application initializes successfully with `npm create vite@latest`
- [ ] Development server starts on port 3000 with `npm run dev`
- [ ] TypeScript compilation passes with strict mode enabled
- [ ] Production build completes without errors with `npm run build`
- [ ] Hot module replacement works during development
- [ ] Path aliases (@, @/components, etc.) resolve correctly
- [ ] Environment variables are properly configured
- [ ] Package.json includes all required scripts and dependencies
- [ ] Basic folder structure is established in src/ directory
- [ ] Vite configuration includes all required optimizations

---

# PROMPT 2: Development Tools Configuration and Code Quality Setup

Configure ESLint, Prettier, and TypeScript integration for consistent code quality, formatting, and error detection. Set up development tooling that enforces project coding standards automatically.

## PROJECT CONTEXT

**Code Quality Standards:**
- TypeScript strict mode for type safety
- Consistent code formatting with Prettier
- ESLint rules enforcing React and TypeScript best practices
- Pre-commit hooks for automated quality checks

**Development Philosophy:**
- Code quality standards enforced automatically
- Consistent formatting across all team members
- Early error detection and prevention
- Zero-tolerance for type errors in strict mode

## EPIC CONTEXT

As part of **EPIC-000: Project Setup and Foundation**, establishing robust development tooling ensures code consistency and quality from the start of the project, reducing technical debt and maintenance overhead.

## TASK Context

**Goal:** Configure comprehensive development tools including ESLint, Prettier, and TypeScript to maintain code quality standards throughout the project lifecycle.

**Definition of Done:** All development tools are configured, working together seamlessly, and enforcing project standards automatically.

## FUNCTIONAL REQUIREMENTS

1. **ESLint Configuration**
   - React-specific rules enabled
   - TypeScript integration configured
   - Import ordering and unused import detection
   - React hooks rules enforcement

2. **Prettier Configuration**
   - Consistent formatting rules
   - Integration with ESLint (no conflicts)
   - Automatic formatting on save
   - Support for TypeScript, JSX, JSON, and CSS

3. **TypeScript Integration**
   - Type checking as separate script
   - Integration with build process
   - Path mapping validation
   - Strict mode compliance

## TECHNICAL REQUIREMENTS

**ESLint Configuration:**
```json
{
  "extends": [
    "eslint:recommended",
    "@typescript-eslint/recommended",
    "plugin:react/recommended",
    "plugin:react-hooks/recommended",
    "plugin:@typescript-eslint/recommended"
  ],
  "rules": {
    "react/react-in-jsx-scope": "off",
    "@typescript-eslint/no-unused-vars": "error",
    "prefer-const": "error"
  }
}
```

**Prettier Configuration:**
```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2,
  "useTabs": false
}
```

**Package Scripts Required:**
- `lint`: Run ESLint on all TypeScript/React files
- `lint:fix`: Run ESLint with auto-fix
- `format`: Format all files with Prettier
- `format:check`: Check if files are properly formatted
- `type-check`: Run TypeScript compiler without emitting files

## TESTING REQUIREMENTS

**Manual Tests:**
- Run `npm run lint` and verify it catches TypeScript errors
- Run `npm run lint:fix` and verify it fixes auto-fixable issues
- Run `npm run format` and verify consistent formatting
- Run `npm run type-check` and verify TypeScript compilation
- Create a file with formatting issues and verify Prettier fixes them

**Integration Tests:**
- ESLint and Prettier work together without conflicts
- TypeScript path aliases are recognized by ESLint
- All development scripts run successfully in CI environment

## DOCUMENTATION REQUIREMENTS

Update:
- README.md with development workflow and tooling commands
- Package.json with proper script descriptions
- Add .eslintignore and .prettierignore files with appropriate exclusions

## VALIDATION CRITERIA

- [ ] ESLint configuration file exists and includes React/TypeScript rules
- [ ] Prettier configuration file exists with consistent formatting rules
- [ ] TypeScript configuration enables strict mode and proper path mapping
- [ ] `npm run lint` executes successfully and reports issues
- [ ] `npm run lint:fix` automatically fixes correctable issues
- [ ] `npm run format` formats all source files consistently
- [ ] `npm run type-check` validates TypeScript without building
- [ ] No conflicts between ESLint and Prettier rules
- [ ] Editor integration works (format on save, error highlighting)
- [ ] All configuration files are properly documented

---

# PROMPT 3: Project Architecture and Folder Structure Setup

Establish the foundational folder structure and architectural patterns that will support feature-based development, domain-driven design, and scalable React application organization.

## PROJECT CONTEXT

**Architectural Principles:**
- Feature-based folder structure supporting domain-driven design
- Clear separation between shared components and feature-specific code
- Centralized configuration and utilities
- Scalable patterns for hooks, providers, and services

**Folder Structure Philosophy:**
```
src/
├── components/          # Shared UI components
├── features/           # Feature-based modules
├── hooks/              # Shared custom hooks
├── lib/                # Core utilities and services
├── providers/          # React context providers
├── routes/             # Routing configuration
├── types/              # Global TypeScript types
└── config/             # Application configuration
```

## EPIC CONTEXT

This architectural foundation supports all future development in **EPIC-000: Project Setup and Foundation** and enables clean implementation of healthcare workflows in subsequent epics.

## TASK Context

**Goal:** Create a well-organized folder structure with proper index files, basic architectural patterns, and foundational components that support scalable feature development.

**Definition of Done:** Complete folder structure exists with proper exports, basic components are implemented, and the architecture supports feature module development.

## FUNCTIONAL REQUIREMENTS

1. **Core Folder Structure**
   - Shared components directory with proper organization
   - Feature modules structure ready for domain-specific code
   - Centralized utilities and configuration
   - Proper TypeScript type organization

2. **Architectural Components**
   - App provider for global context
   - Basic layout components structure
   - Routing foundation (placeholder)
   - Configuration management setup

3. **Development Support**
   - Index files for clean imports
   - TypeScript type definitions
   - Utility functions structure
   - Asset organization

## TECHNICAL REQUIREMENTS

**Folder Structure Implementation:**
```
src/
├── assets/
│   ├── fonts/
│   ├── images/
│   └── icons/
├── components/
│   ├── layouts/
│   │   ├── AuthLayout.tsx
│   │   └── DashboardLayout.tsx
│   ├── ui/              # Future: shadcn/ui components
│   └── index.ts         # Export all shared components
├── config/
│   ├── app.config.ts
│   ├── routes.config.ts
│   └── index.ts
├── features/
│   └── README.md        # Feature module guidelines
├── hooks/
│   ├── useLocalStorage.ts
│   └── index.ts
├── lib/
│   ├── utils/
│   │   ├── cn.ts        # className utility
│   │   └── index.ts
│   └── store.ts         # Future: Global store setup
├── providers/
│   ├── AppProvider.tsx
│   └── index.ts
├── routes/
│   ├── AppRoutes.tsx
│   └── index.ts
├── types/
│   ├── global.types.ts
│   └── index.ts
├── App.tsx
├── main.tsx
└── index.css
```

**Component Implementations Required:**
- Basic AppProvider with theme context
- Placeholder layout components
- Basic routing structure
- Utility functions (className helper, etc.)

## BUSINESS DOMAIN CONTEXT

The folder structure must accommodate healthcare-specific features including:
- Patient management workflows
- Appointment scheduling systems
- Clinical documentation interfaces
- Provider management tools
- Multi-language support (Arabic/French)

## TESTING REQUIREMENTS

**Manual Tests:**
- Verify all index files export components properly
- Test that path aliases work with new folder structure
- Verify App.tsx renders with AppProvider
- Confirm TypeScript compilation with new structure

**Integration Tests:**
- All imports resolve correctly with path aliases
- Feature module structure supports future development
- Layout components render without errors
- Configuration files are properly typed

## DOCUMENTATION REQUIREMENTS

Create:
- src/features/README.md with feature module guidelines
- Update main README.md with folder structure explanation
- Add JSDoc comments to utility functions
- Document architectural patterns and conventions

## VALIDATION CRITERIA

- [ ] All required folders exist with proper structure
- [ ] Index files provide clean exports for each directory
- [ ] App.tsx uses AppProvider and renders successfully
- [ ] Path aliases work for all new folder structures
- [ ] Basic layout components exist and are importable
- [ ] Configuration files are properly structured and typed
- [ ] TypeScript compilation passes with new structure
- [ ] All placeholder components render without errors
- [ ] Utility functions are properly implemented and tested
- [ ] Feature module structure is documented and ready for use

---

# PROMPT 4: Build Optimization and Production Configuration

Configure Vite build optimizations, bundle analysis, environment configuration, and production-ready settings to ensure optimal performance and deployment capabilities.

## PROJECT CONTEXT

**Performance Requirements:**
- Fast development server startup (under 5 seconds)
- Optimized production bundles with code splitting
- Asset optimization and compression
- Source map generation for debugging

**Deployment Considerations:**
- Environment-specific configuration support
- Build artifacts ready for CI/CD deployment
- Bundle size monitoring and optimization
- Browser compatibility requirements

## EPIC CONTEXT

As part of **EPIC-000: Project Setup and Foundation**, proper build configuration ensures the application can be efficiently deployed and performs well in production environments.

## TASK Context

**Goal:** Configure comprehensive build optimization, environment management, and production settings to ensure optimal application performance and deployment readiness.

**Definition of Done:** Build system produces optimized bundles, supports multiple environments, includes proper source maps, and meets performance requirements.

## FUNCTIONAL REQUIREMENTS

1. **Build Optimization**
   - Code splitting for vendor and application code
   - Asset optimization (images, fonts, etc.)
   - CSS optimization and purging
   - Tree shaking for unused code elimination

2. **Environment Configuration**
   - Support for development, staging, and production environments
   - Environment variable management
   - Configuration validation and type safety
   - Secure handling of sensitive variables

3. **Bundle Analysis**
   - Bundle size reporting
   - Dependency analysis
   - Performance monitoring
   - Size limit warnings

## TECHNICAL REQUIREMENTS

**Enhanced Vite Configuration:**
```typescript
export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@/components': path.resolve(__dirname, './src/components'),
      '@/features': path.resolve(__dirname, './src/features'),
      '@/lib': path.resolve(__dirname, './src/lib'),
      '@/hooks': path.resolve(__dirname, './src/hooks'),
      '@/types': path.resolve(__dirname, './src/types'),
    },
  },
  server: {
    port: 3000,
    open: true,
    cors: true,
  },
  build: {
    outDir: 'dist',
    sourcemap: true,
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          router: ['react-router-dom'],
        },
      },
    },
    chunkSizeWarningLimit: 1000,
  },
  define: {
    __APP_VERSION__: JSON.stringify(process.env.npm_package_version),
  },
  optimizeDeps: {
    include: ['react', 'react-dom'],
  },
});
```

**Environment Files:**
- `.env` (shared variables)
- `.env.local` (local overrides, ignored by git)
- `.env.development` (development-specific)
- `.env.production` (production-specific)

**Required Environment Variables:**
```
VITE_API_URL=http://localhost:3001/api
VITE_APP_NAME=MedFlow
VITE_APP_VERSION=1.0.0
VITE_ENVIRONMENT=development
```

## TESTING REQUIREMENTS

**Build Tests:**
- Development build completes in under 30 seconds
- Production build completes in under 2 minutes
- Bundle size stays under warning limits
- Source maps are generated correctly

**Environment Tests:**
- Environment variables are properly loaded
- Different environment configurations work correctly
- Build artifacts are properly structured
- Assets are optimized and compressed

**Performance Tests:**
- Development server startup time measurement
- Build time benchmarking
- Bundle size analysis
- Runtime performance validation

## DOCUMENTATION REQUIREMENTS

Update:
- README.md with build commands and environment setup
- Add comments to vite.config.ts explaining optimizations
- Document environment variable requirements
- Create deployment documentation

## VALIDATION CRITERIA

- [ ] Vite configuration includes all required optimizations
- [ ] Code splitting works correctly (vendor/app chunks)
- [ ] Environment variables are properly configured and typed
- [ ] Production build generates optimized assets
- [ ] Source maps are available for debugging
- [ ] Bundle size warnings are configured appropriately
- [ ] Development server starts quickly (under 5 seconds)
- [ ] Asset optimization reduces file sizes
- [ ] Build artifacts are structured correctly for deployment
- [ ] Performance metrics meet specified requirements

---

# PROMPT 5: Git Hooks and Code Quality Automation

Set up Git hooks using Husky and lint-staged to automatically enforce code quality standards, run tests, and ensure all commits meet project requirements before they're committed.

## PROJECT CONTEXT

**Quality Assurance Philosophy:**
- Prevent low-quality code from entering the repository
- Automate repetitive quality checks
- Enforce consistent standards across all contributors
- Early detection and correction of issues

**Automation Requirements:**
- Pre-commit hooks for linting and formatting
- Pre-push hooks for tests and builds
- Automatic code formatting when possible
- Type checking enforcement

## EPIC CONTEXT

This automation supports the quality standards established in **EPIC-000: Project Setup and Foundation** and ensures consistent code quality throughout the project lifecycle.

## TASK Context

**Goal:** Configure Git hooks with Husky and lint-staged to automatically enforce code quality standards, formatting, and testing before commits and pushes.

**Definition of Done:** Git hooks are configured and working, preventing commits that don't meet quality standards, and automatically fixing issues where possible.

## FUNCTIONAL REQUIREMENTS

1. **Pre-commit Hooks**
   - Lint staged files with ESLint
   - Format staged files with Prettier
   - Run TypeScript type checking
   - Prevent commits with linting errors

2. **Pre-push Hooks**
   - Run full test suite (when available)
   - Verify production build succeeds
   - Ensure no TypeScript errors
   - Validate bundle size limits

3. **Automated Fixes**
   - Auto-format code when possible
   - Fix auto-correctable ESLint issues
   - Organize imports automatically
   - Update file permissions as needed

## TECHNICAL REQUIREMENTS

**Package Dependencies:**
```json
{
  "devDependencies": {
    "husky": "^8.0.3",
    "lint-staged": "^15.2.0"
  }
}
```

**Husky Configuration:**
```bash
# .husky/pre-commit
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

npx lint-staged
```

**Lint-staged Configuration:**
```json
{
  "lint-staged": {
    "*.{ts,tsx}": [
      "eslint --fix",
      "prettier --write",
      "git add"
    ],
    "*.{json,md,css}": [
      "prettier --write",
      "git add"
    ]
  }
}
```

**Package Scripts:**
```json
{
  "scripts": {
    "prepare": "husky install",
    "pre-commit": "lint-staged",
    "pre-push": "npm run type-check && npm run build"
  }
}
```

## TESTING REQUIREMENTS

**Hook Functionality Tests:**
- Create a file with linting errors and attempt to commit
- Verify ESLint errors prevent commit
- Test that auto-fixable issues are corrected automatically
- Confirm Prettier formatting occurs on staged files
- Verify pre-push hooks run before git push

**Integration Tests:**
- Test hooks work in different Git workflows
- Verify hooks work with various file types
- Test performance impact of hooks
- Confirm hooks work in CI/CD environment

## DOCUMENTATION REQUIREMENTS

Create or update:
- README.md with Git workflow and quality standards
- Contributing guidelines with hook information
- Troubleshooting guide for hook issues
- Development setup instructions

## VALIDATION CRITERIA

- [ ] Husky is installed and configured correctly
- [ ] lint-staged configuration covers all relevant file types
- [ ] Pre-commit hook prevents commits with linting errors
- [ ] Pre-commit hook automatically formats staged files
- [ ] Pre-commit hook runs TypeScript type checking
- [ ] Pre-push hook validates production build
- [ ] Auto-fixable ESLint issues are corrected automatically
- [ ] Hooks perform efficiently (under 30 seconds for typical changes)
- [ ] Hooks can be bypassed when necessary (emergency commits)
- [ ] All team members can set up hooks successfully