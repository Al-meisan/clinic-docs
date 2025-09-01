# TASK-014: Development Environment Configuration

## Task Overview

### Goal
**What to Build:** Configure comprehensive development environment with ESLint, Prettier, Jest testing framework, development scripts, Git hooks, and quality assurance tools optimized for healthcare application development.

**Why:** Establishes consistent development practices, enforces code quality standards, prevents bugs through automated testing, and ensures maintainable codebase for the healthcare management system.

**Definition of Done:** Complete development toolchain operational with automated code quality checks, testing framework, Git hooks enforcement, and development scripts for efficient workflow.

### Scope
```yaml
task_type: "DEVELOPMENT"
complexity: "MEDIUM"
```

## Requirements

### Functional Requirements
1. **Code Quality Tools Configuration**
   - Description: ESLint and Prettier setup with healthcare-specific rules and consistent formatting
   - Acceptance Criteria: Linting rules enforce project standards, formatting is automatic, and rules appropriate for medical software
   - Priority: HIGH

2. **Testing Framework Setup**
   - Description: Jest and React Testing Library configuration with healthcare-specific testing patterns
   - Acceptance Criteria: Unit tests run efficiently, testing utilities available, coverage reporting functional
   - Priority: HIGH

3. **Git Hooks and Quality Gates**
   - Description: Pre-commit hooks, commit message validation, and automated quality checks
   - Acceptance Criteria: Code quality enforced before commits, consistent commit messages, failing tests prevent commits
   - Priority: MEDIUM

4. **Development Scripts and Automation**
   - Description: Package.json scripts for development workflow, build processes, and maintenance tasks
   - Acceptance Criteria: Common development tasks automated, build process optimized, maintenance scripts available
   - Priority: MEDIUM

### Technical Requirements
```yaml
performance:
  - requirement: "Development tool performance"
    target: "< 5 seconds for linting, < 10 seconds for test suite"
    
quality:
  - requirement: "Code quality enforcement"
    implementation: "Automated quality gates preventing low-quality code commits"
    
compatibility:
  - requirement: "Cross-platform development support"
    scope: "Tools work consistently across Windows, macOS, and Linux"
```

## Implementation Details

### Technical Approach
```yaml
development_tools:
  code_quality:
    - eslint: "Code linting with TypeScript and React rules"
    - prettier: "Code formatting with healthcare-appropriate settings"
    - typescript: "Strict type checking configuration"
    - husky: "Git hooks management"
    - lint_staged: "Pre-commit linting for staged files"
    
  testing_framework:
    - jest: "JavaScript testing framework"
    - react_testing_library: "React component testing utilities"
    - jest_dom: "Extended Jest matchers for DOM testing"
    - msw: "API mocking for testing"
    - coverage_reporting: "Test coverage analysis and reporting"
    
  development_scripts:
    - build_scripts: "Production and development build automation"
    - test_scripts: "Various testing scenarios and coverage"
    - maintenance_scripts: "Database migrations, cleanup tasks"
    - deployment_scripts: "Environment-specific deployment automation"

healthcare_specific_rules:
  eslint_rules:
    - medical_data_handling: "Rules for sensitive healthcare data"
    - accessibility: "WCAG compliance enforcement"
    - performance: "Performance-critical code patterns"
    - security: "Security best practices for medical applications"
    
  testing_patterns:
    - patient_data_mocking: "Secure patient data mocking patterns"
    - clinical_workflow_testing: "Healthcare workflow testing utilities"
    - accessibility_testing: "Automated accessibility test patterns"
    - integration_testing: "Medical system integration test patterns"
```

### Integration Points
```yaml
dependencies:
  - dependency: "React and TypeScript application setup"
    type: "DEVELOPMENT"
    status: "AVAILABLE"
    
  - dependency: "Git repository with proper structure"
    type: "DEVELOPMENT"
    status: "AVAILABLE"
    
  - dependency: "Node.js development environment"
    type: "DEVELOPMENT"
    status: "AVAILABLE"
    
provides:
  - deliverable: "Development quality tools"
    interface: "ESLint, Prettier, and testing configurations"
    consumers: "All development team members"
    
  - deliverable: "Testing framework"
    interface: "Jest configuration and testing utilities"
    consumers: "All code requiring automated testing"
    
  - deliverable: "Development automation scripts"
    interface: "npm scripts for common development tasks"
    consumers: "Development workflow and deployment processes"
```

### Code Patterns to Follow

**ESLint Configuration:**
```javascript
// .eslintrc.js
module.exports = {
  root: true,
  env: {
    browser: true,
    es2022: true,
    node: true,
    jest: true,
  },
  extends: [
    'eslint:recommended',
    '@typescript-eslint/recommended',
    '@typescript-eslint/recommended-requiring-type-checking',
    'plugin:react/recommended',
    'plugin:react/jsx-runtime',
    'plugin:react-hooks/recommended',
    'plugin:jsx-a11y/recommended',
    'plugin:@typescript-eslint/strict',
    'prettier',
  ],
  parser: '@typescript-eslint/parser',
  parserOptions: {
    ecmaVersion: 'latest',
    sourceType: 'module',
    project: ['./tsconfig.json', './tsconfig.node.json'],
    tsconfigRootDir: __dirname,
    ecmaFeatures: {
      jsx: true,
    },
  },
  plugins: [
    '@typescript-eslint',
    'react',
    'react-hooks',
    'jsx-a11y',
    'import',
    'unused-imports',
  ],
  settings: {
    react: {
      version: 'detect',
    },
    'import/resolver': {
      typescript: {
        alwaysTryTypes: true,
        project: './tsconfig.json',
      },
    },
  },
  rules: {
    // TypeScript specific rules
    '@typescript-eslint/no-unused-vars': [
      'error',
      { argsIgnorePattern: '^_', varsIgnorePattern: '^_' },
    ],
    '@typescript-eslint/explicit-function-return-type': 'off',
    '@typescript-eslint/explicit-module-boundary-types': 'off',
    '@typescript-eslint/no-explicit-any': 'warn',
    '@typescript-eslint/prefer-nullish-coalescing': 'error',
    '@typescript-eslint/prefer-optional-chain': 'error',
    '@typescript-eslint/no-floating-promises': 'error',
    '@typescript-eslint/await-thenable': 'error',
    
    // React specific rules
    'react/react-in-jsx-scope': 'off',
    'react/prop-types': 'off',
    'react/jsx-uses-react': 'off',
    'react/jsx-uses-vars': 'error',
    'react-hooks/rules-of-hooks': 'error',
    'react-hooks/exhaustive-deps': 'warn',
    
    // Healthcare-specific rules
    'no-console': 'warn', // Prevent accidental console.log in production
    'no-debugger': 'error',
    'no-alert': 'error',
    
    // Security rules for healthcare data
    'no-eval': 'error',
    'no-implied-eval': 'error',
    'no-script-url': 'error',
    
    // Accessibility rules
    'jsx-a11y/alt-text': 'error',
    'jsx-a11y/anchor-has-content': 'error',
    'jsx-a11y/aria-props': 'error',
    'jsx-a11y/aria-proptypes': 'error',
    'jsx-a11y/aria-unsupported-elements': 'error',
    'jsx-a11y/click-events-have-key-events': 'error',
    'jsx-a11y/heading-has-content': 'error',
    'jsx-a11y/label-has-associated-control': 'error',
    
    // Import organization
    'import/order': [
      'error',
      {
        groups: [
          'builtin',
          'external',
          'internal',
          'parent',
          'sibling',
          'index',
        ],
        'newlines-between': 'always',
        alphabetize: {
          order: 'asc',
          caseInsensitive: true,
        },
      },
    ],
    'import/no-duplicates': 'error',
    'import/no-unresolved': 'error',
    
    // Code quality
    'prefer-const': 'error',
    'no-var': 'error',
    'object-shorthand': 'error',
    'prefer-template': 'error',
    
    // Medical data handling rules
    'no-console': ['warn', { allow: ['warn', 'error'] }],
    'no-restricted-syntax': [
      'error',
      {
        selector: 'CallExpression[callee.name="localStorage"][arguments.0.value=/patient|medical|clinical/i]',
        message: 'Direct localStorage usage with medical data is prohibited. Use secure storage utilities.',
      },
      {
        selector: 'CallExpression[callee.name="sessionStorage"][arguments.0.value=/patient|medical|clinical/i]',
        message: 'Direct sessionStorage usage with medical data is prohibited. Use secure storage utilities.',
      },
    ],
    
    // Unused imports cleanup
    'unused-imports/no-unused-imports': 'error',
    'unused-imports/no-unused-vars': [
      'warn',
      {
        vars: 'all',
        varsIgnorePattern: '^_',
        args: 'after-used',
        argsIgnorePattern: '^_',
      },
    ],
  },
  overrides: [
    {
      files: ['**/*.test.{js,jsx,ts,tsx}', '**/*.spec.{js,jsx,ts,tsx}'],
      env: {
        jest: true,
      },
      rules: {
        // Test files can be more lenient
        '@typescript-eslint/no-explicit-any': 'off',
        '@typescript-eslint/no-non-null-assertion': 'off',
      },
    },
    {
      files: ['vite.config.ts', '*.config.js'],
      env: {
        node: true,
      },
    },
  ],
  ignorePatterns: [
    'dist/',
    'build/',
    'node_modules/',
    '*.min.js',
    'coverage/',
    'public/',
  ],
};
```

**Prettier Configuration:**
```javascript
// .prettierrc.js
module.exports = {
  // Core formatting options
  printWidth: 100,
  tabWidth: 2,
  useTabs: false,
  semi: true,
  singleQuote: true,
  quoteProps: 'as-needed',
  trailingComma: 'es5',
  bracketSpacing: true,
  bracketSameLine: false,
  arrowParens: 'always',
  endOfLine: 'lf',
  
  // Healthcare-specific formatting preferences
  proseWrap: 'always', // For documentation
  htmlWhitespaceSensitivity: 'css',
  
  // File-specific overrides
  overrides: [
    {
      files: '*.json',
      options: {
        tabWidth: 2,
      },
    },
    {
      files: '*.md',
      options: {
        proseWrap: 'always',
        printWidth: 80,
      },
    },
    {
      files: '*.{css,scss}',
      options: {
        singleQuote: false,
      },
    },
    {
      files: '*.{yml,yaml}',
      options: {
        tabWidth: 2,
        singleQuote: false,
      },
    },
  ],
};
```

**Jest Configuration:**
```javascript
// jest.config.js
module.exports = {
  testEnvironment: 'jsdom',
  roots: ['<rootDir>/src'],
  setupFilesAfterEnv: ['<rootDir>/src/setupTests.ts'],
  moduleNameMapping: {
    '^@/(.*)$': '<rootDir>/src/$1',
    '\\.(css|less|scss|sass)$': 'identity-obj-proxy',
    '\\.(jpg|jpeg|png|gif|eot|otf|webp|svg|ttf|woff|woff2|mp4|webm|wav|mp3|m4a|aac|oga)$':
      '<rootDir>/src/__mocks__/fileMock.js',
  },
  transform: {
    '^.+\\.(ts|tsx)$': ['ts-jest', {
      useESM: true,
    }],
    '^.+\\.(js|jsx)$': ['babel-jest', {
      presets: [
        ['@babel/preset-env', { targets: { node: 'current' } }],
        ['@babel/preset-react', { runtime: 'automatic' }],
      ],
    }],
  },
  transformIgnorePatterns: [
    'node_modules/(?!(.*\\.mjs$|@testing-library|@tanstack/react-query))',
  ],
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
    '!src/main.tsx',
    '!src/vite-env.d.ts',
    '!src/**/*.stories.{js,jsx,ts,tsx}',
    '!src/**/__tests__/**',
    '!src/**/__mocks__/**',
  ],
  coverageReporters: ['text', 'lcov', 'html'],
  coverageDirectory: 'coverage',
  coverageThreshold: {
    global: {
      branches: 70,
      functions: 70,
      lines: 70,
      statements: 70,
    },
    // Healthcare-critical modules should have higher coverage
    './src/lib/auth/**/*.ts': {
      branches: 90,
      functions: 90,
      lines: 90,
      statements: 90,
    },
    './src/hooks/**/*.ts': {
      branches: 85,
      functions: 85,
      lines: 85,
      statements: 85,
    },
  },
  testMatch: [
    '<rootDir>/src/**/__tests__/**/*.{js,jsx,ts,tsx}',
    '<rootDir>/src/**/*.(test|spec).{js,jsx,ts,tsx}',
  ],
  watchPlugins: [
    'jest-watch-typeahead/filename',
    'jest-watch-typeahead/testname',
  ],
  globalSetup: '<rootDir>/src/__tests__/setup/globalSetup.ts',
  globalTeardown: '<rootDir>/src/__tests__/setup/globalTeardown.ts',
  
  // Healthcare-specific test configurations
  testTimeout: 10000, // Longer timeout for integration tests
  maxWorkers: '50%', // Conservative for CI environments
  
  // Mock service worker for API testing
  setupFiles: ['<rootDir>/src/__mocks__/setupMSW.ts'],
};
```

**Test Setup Files:**
```typescript
// src/setupTests.ts
import '@testing-library/jest-dom';
import 'jest-canvas-mock';

// Mock IntersectionObserver
global.IntersectionObserver = jest.fn().mockImplementation(() => ({
  observe: jest.fn(),
  unobserve: jest.fn(),
  disconnect: jest.fn(),
}));

// Mock ResizeObserver
global.ResizeObserver = jest.fn().mockImplementation(() => ({
  observe: jest.fn(),
  unobserve: jest.fn(),
  disconnect: jest.fn(),
}));

// Mock window.matchMedia
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: jest.fn().mockImplementation(query => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: jest.fn(), // deprecated
    removeListener: jest.fn(), // deprecated
    addEventListener: jest.fn(),
    removeEventListener: jest.fn(),
    dispatchEvent: jest.fn(),
  })),
});

// Mock window.scrollTo
Object.defineProperty(window, 'scrollTo', {
  writable: true,
  value: jest.fn(),
});

// Healthcare-specific test utilities
export const mockPatient = {
  id: 'patient-1',
  firstName: 'Ahmed',
  lastName: 'Benali',
  dateOfBirth: '1985-03-15',
  gender: 'male',
  phone: '+213-555-0123',
  email: 'ahmed.benali@email.com',
  status: 'active',
};

export const mockAppointment = {
  id: 'appointment-1',
  patientId: 'patient-1',
  providerId: 'provider-1',
  dateTime: '2024-12-01T10:00:00Z',
  duration: 30,
  type: 'consultation',
  status: 'scheduled',
  reason: 'Routine checkup',
};

// Mock authentication context for tests
export const mockAuthUser = {
  id: 'user-1',
  email: 'doctor@clinic.dz',
  firstName: 'Dr. Amina',
  lastName: 'Khoury',
  role: 'healthcare_provider',
  scopes: ['patient:read', 'patient:write', 'appointment:read', 'appointment:write'],
  clinicId: 'clinic-1',
  isActive: true,
};

// Global test error handler
window.addEventListener('error', (event) => {
  console.error('Test window error:', event.error);
});

window.addEventListener('unhandledrejection', (event) => {
  console.error('Test unhandled promise rejection:', event.reason);
});
```

**Package.json Scripts:**
```json
{
  "scripts": {
    "dev": "vite --port 3000",
    "build": "tsc && vite build",
    "build:analyze": "npm run build && npx vite-bundle-analyzer",
    "preview": "vite preview",
    "serve": "vite preview --port 3000",
    
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "test:ci": "jest --coverage --watchAll=false --ci",
    "test:debug": "node --inspect-brk node_modules/.bin/jest --runInBand --no-cache",
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    
    "lint": "eslint . --ext .ts,.tsx,.js,.jsx --report-unused-disable-directives --max-warnings 0",
    "lint:fix": "eslint . --ext .ts,.tsx,.js,.jsx --fix",
    "lint:staged": "lint-staged",
    
    "format": "prettier --write \"src/**/*.{ts,tsx,js,jsx,json,css,md}\"",
    "format:check": "prettier --check \"src/**/*.{ts,tsx,js,jsx,json,css,md}\"",
    
    "type-check": "tsc --noEmit",
    "type-check:watch": "tsc --noEmit --watch",
    
    "clean": "rm -rf dist coverage node_modules/.cache",
    "clean:deps": "rm -rf node_modules package-lock.json && npm install",
    
    "prepare": "husky install",
    "postinstall": "husky install",
    
    "docker:build": "docker build -t medflow-frontend .",
    "docker:run": "docker run -p 3000:3000 medflow-frontend",
    
    "storybook": "storybook dev -p 6006",
    "build-storybook": "storybook build",
    
    "translate:extract": "i18next-parser",
    "translate:check": "i18next-parser --fail-on-warnings",
    
    "security:audit": "npm audit --audit-level high",
    "security:fix": "npm audit fix",
    
    "deps:check": "npm-check-updates",
    "deps:update": "npm-check-updates -u && npm install",
    
    "precommit": "lint-staged && npm run type-check",
    "prepush": "npm run test:ci && npm run build"
  }
}
```

**Husky Git Hooks:**
```bash
#!/usr/bin/env sh
# .husky/pre-commit
. "$(dirname -- "$0")/_/husky.sh"

echo "🔍 Running pre-commit checks..."

# Run lint-staged for staged files
npm run lint:staged

# Type check
echo "📝 Type checking..."
npm run type-check

# Run tests for changed files
echo "🧪 Running tests..."
npm run test -- --bail --findRelatedTests --passWithNoTests

echo "✅ Pre-commit checks passed!"
```

```bash
#!/usr/bin/env sh
# .husky/commit-msg
. "$(dirname -- "$0")/_/husky.sh"

# Commit message validation for healthcare project
echo "📝 Validating commit message..."

# Check commit message format: type(scope): description
commit_regex='^(feat|fix|docs|style|refactor|test|chore|ci|perf|build|revert)(\(.+\))?: .{1,72}'

if ! grep -qE "$commit_regex" "$1"; then
    echo "❌ Invalid commit message format!"
    echo ""
    echo "Format: type(scope): description"
    echo ""
    echo "Types:"
    echo "  feat:     New feature"
    echo "  fix:      Bug fix" 
    echo "  docs:     Documentation changes"
    echo "  style:    Code style changes"
    echo "  refactor: Code refactoring"
    echo "  test:     Test changes"
    echo "  chore:    Maintenance tasks"
    echo "  perf:     Performance improvements"
    echo "  build:    Build system changes"
    echo "  revert:   Revert previous commit"
    echo ""
    echo "Examples:"
    echo "  feat(patients): add patient search functionality"
    echo "  fix(auth): resolve login token expiration issue"
    echo "  docs(api): update appointment API documentation"
    echo ""
    exit 1
fi

# Check for medical data references in commit messages
if grep -qiE "(patient data|medical record|phi|healthcare privacy)" "$1"; then
    echo "⚠️  Warning: Commit message references sensitive medical data."
    echo "   Please ensure no actual patient information is included."
fi

echo "✅ Commit message format is valid!"
```

**Lint-staged Configuration:**
```javascript
// .lintstagedrc.js
module.exports = {
  '*.{ts,tsx,js,jsx}': [
    'eslint --fix',
    'prettier --write',
    () => 'tsc --noEmit', // Type check all files
  ],
  '*.{json,css,md,yml,yaml}': ['prettier --write'],
  '*.{ts,tsx}': [
    () => 'jest --bail --findRelatedTests --passWithNoTests',
  ],
  // Healthcare-specific file validations
  '**/patient*.{ts,tsx}': [
    () => 'echo "🏥 Checking patient-related files for security compliance..."',
    'eslint --rule "no-console: error"', // No console.log in patient files
  ],
  '**/clinical*.{ts,tsx}': [
    () => 'echo "🩺 Validating clinical module compliance..."',
    'eslint --rule "no-debugger: error"',
  ],
};
```

**VSCode Settings for Team:**
```json
// .vscode/settings.json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true,
    "source.organizeImports": true
  },
  "typescript.preferences.importModuleSpecifier": "relative",
  "typescript.suggest.autoImports": true,
  "typescript.updateImportsOnFileMove.enabled": "always",
  
  "files.associations": {
    "*.css": "tailwindcss"
  },
  
  "emmet.includeLanguages": {
    "typescript": "html",
    "typescriptreact": "html"
  },
  
  "tailwindCSS.includeLanguages": {
    "typescript": "html",
    "typescriptreact": "html"
  },
  
  "files.exclude": {
    "**/node_modules": true,
    "**/dist": true,
    "**/coverage": true,
    "**/.next": true
  },
  
  "search.exclude": {
    "**/node_modules": true,
    "**/dist": true,
    "**/coverage": true
  },
  
  "jest.jestCommandLine": "npm test --",
  "jest.autoRun": "off",
  
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[json]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

## Testing Requirements

### Test Coverage
```yaml
development_tools_tests:
  - focus: "Development tool configurations and scripts"
  - coverage_target: "100% of configuration files validated"
  - key_scenarios: ["linting_rules", "formatting_consistency", "git_hooks_execution"]
  
integration_tests:
  - focus: "Tool integration and workflow testing"
  - test_environment: "Fresh repository with clean setup"
  - key_flows: ["new_developer_setup", "commit_workflow", "build_process"]
  
automation_tests:
  - focus: "Automated quality gate testing"
  - test_environment: "Development environment simulation"
  - key_scenarios: ["pre_commit_validation", "test_execution", "build_verification"]
```

### Test Scenarios
```yaml
happy_path:
  - scenario: "Developer workflow with quality tools"
    steps: ["Write code", "Auto-format on save", "Run tests", "Commit with validation"]
    expected_result: "Smooth development experience with automatic quality enforcement"
    
error_cases:
  - scenario: "Code quality violations"
    trigger: "Code with linting errors, failing tests, or formatting issues"
    expected_behavior: "Clear error messages and prevention of problematic commits"
    
edge_cases:
  - scenario: "Large codebase performance"
    conditions: "Running quality tools on entire codebase"
    expected_behavior: "Acceptable performance without workflow interruption"
```

## Context References

### From Project
```yaml
tech_stack_references:
  - "TypeScript with strict configuration for type safety"
  - "React with comprehensive linting and testing"
  - "Healthcare-specific code quality standards"
  - "Node.js development environment optimization"
  
architecture_patterns:
  - "Quality-first development approach"
  - "Automated testing and validation patterns"
  - "Git workflow with quality gates"
  - "Development environment standardization"
  
code_standards:
  - "Consistent code formatting across team"
  - "Comprehensive linting rules for healthcare applications"
  - "Test-driven development practices"
  - "Security-focused development practices"
```

### From Epic
```yaml
business_rules:
  - "Code quality critical for healthcare application reliability"
  - "Automated testing essential for patient safety"
  - "Security standards enforced through development tools"
  - "Consistent development practices across team"
  
integration_points:
  - "Required for all development activities"
  - "Enables consistent code quality across features"
```

## Acceptance Criteria

### Functional Acceptance
- [ ] ESLint configuration enforces project coding standards
- [ ] Prettier automatically formats code consistently
- [ ] Jest testing framework runs unit and integration tests
- [ ] Git hooks prevent commits that don't meet quality standards
- [ ] Development scripts automate common workflow tasks

### Technical Acceptance  
- [ ] Development tools meet performance requirements
- [ ] Quality gates effectively prevent low-quality code
- [ ] Tool configurations work across different operating systems
- [ ] Test coverage reporting provides meaningful insights
- [ ] Healthcare-specific rules properly enforced

### Quality Acceptance
- [ ] Development environment setup tested with new developers
- [ ] Quality tool effectiveness validated through problematic code testing
- [ ] Performance impact of development tools acceptable
- [ ] Documentation provides clear setup and usage instructions
- [ ] Team adoption of development practices successful

---

**Last Updated:** 2025-01-24