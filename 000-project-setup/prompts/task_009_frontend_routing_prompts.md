# PROMPTS: TASK-009 - Frontend Routing System

# PROMPT 1: React Router Foundation and Route Configuration

**Implement the core React Router v6 setup with lazy loading, error boundaries, and basic route structure for the MedFlow healthcare application. Create the main routing configuration that will serve as the foundation for all navigation workflows.**

## PROJECT CONTEXT
**MedFlow Healthcare Management Application:**
- Vision: Streamlined healthcare management platform for single-doctor practices in Algeria with scalability for multi-provider clinics
- Target Users: Healthcare providers, clinical staff, administrative staff, and system administrators
- Tech Stack: React with TypeScript, Tailwind CSS, shadcn/ui components
- Architecture: Feature-based folder structure with domain-driven design
- Languages: Multi-language support (Arabic, French)

**Current State:**
- React application foundation is established (TASK-007 completed)
- UI framework with shadcn/ui is available (TASK-008 completed)
- Authentication system implementation is planned (TASK-010 pending)
- Backend authentication system ready (TASK-006 completed)

## EPIC CONTEXT
**EPIC-000 Project Setup and Infrastructure:**
- Goal: Establish foundational technical infrastructure for rapid healthcare feature development
- Current Phase: Frontend foundation setup and routing implementation
- Dependencies: React app setup, UI framework integration
- Next Phase: Authentication implementation and layout system

## TASK CONTEXT
**TASK-009: Frontend Routing System**
- Goal: Implement comprehensive React Router system with protected routes and healthcare workflows
- Priority: HIGH (blocking authentication and all feature development)
- Expected Outcome: Complete routing foundation ready for protected routes and feature integration
- Integration Points: Authentication system, UI layouts, navigation components

## FUNCTIONAL REQUIREMENTS
1. **React Router v6 Integration**
   - Browser router setup with nested route support
   - Lazy loading implementation for route-based code splitting
   - Suspense boundaries with appropriate loading states
   - Error boundaries for route-specific error handling

2. **Route Structure Foundation**
   - Public routes (login, registration, password reset)
   - Protected route wrapper structure (ready for authentication)
   - Nested routes supporting dashboard layout
   - Healthcare workflow route patterns preparation

3. **Performance Optimization**
   - Route-based lazy loading with React.lazy()
   - Suspense fallbacks matching final component dimensions
   - Preloading strategies for critical paths

## TECHNICAL REQUIREMENTS
- Use React Router v6+ with latest routing patterns
- Implement TypeScript interfaces for route configuration
- Follow feature-based folder structure: `src/routes/`
- Support both dashboard and authentication layouts
- Maintain accessibility with proper focus management
- Route transition performance < 200ms

## TESTING REQUIREMENTS
**Unit Tests:**
- Route component rendering and lazy loading
- Error boundary functionality
- Route parameter extraction
- Navigation utilities

**Integration Tests:**
- Route transitions between public and protected areas
- Lazy loading with Suspense boundaries
- Error boundary activation and fallback display

## VALIDATION CRITERIA
- [ ] React Router v6 successfully configured with BrowserRouter
- [ ] All route components lazy load properly with Suspense
- [ ] Error boundaries catch and display route errors appropriately
- [ ] Route transitions work smoothly with proper loading states
- [ ] TypeScript types defined for route configuration
- [ ] Route structure supports future protected route implementation
- [ ] Performance targets met for route transitions < 200ms
- [ ] Accessibility maintained during route transitions

---

# PROMPT 2: Protected Route System Implementation

**Create a comprehensive protected route system that enforces authentication and authorization using OAuth2 scopes. Implement route guards that redirect unauthenticated users and handle insufficient permissions scenarios for healthcare workflows.**

## PROJECT CONTEXT
**MedFlow Healthcare Management Application:**
- OAuth2 scope-based authorization system with healthcare-specific roles
- User Roles: System Administrator, Clinic Manager, Healthcare Provider, Clinical Staff, Administrative Staff
- Security Requirements: Route-based access control for medical data and administrative functions
- Healthcare Compliance: Proper access control for patient data protection

**Standardized OAuth2 Scopes:**
- `admin:full` - Complete system administration
- `clinic:manage` - Clinic operations management  
- `patient:write/read` - Patient record access
- `appointment:write/read` - Appointment management
- `clinical:write/read` - Clinical documentation access
- `billing:write/read` - Financial information access
- `prescription:write/read` - Prescription management

## EPIC CONTEXT
**EPIC-000 Project Setup:**
- Authentication foundation ready with backend authentication integration planned
- Role-based access control critical for healthcare data security
- Protected routes block all feature development until implemented

## TASK CONTEXT  
**TASK-009: Frontend Routing System**
- Dependency: Basic route structure from Prompt 1 completed
- Integration Point: Authentication context and user session management
- Next Step: Navigation utilities and healthcare workflow patterns

## FUNCTIONAL REQUIREMENTS
1. **Protected Route Component**
   - Authentication status checking with loading states
   - OAuth2 scope validation for route access
   - Redirect logic for unauthenticated users
   - Unauthorized access handling with appropriate error pages

2. **Public Route Component** 
   - Redirect authenticated users away from login/register pages
   - Maintain clean separation between public and protected areas
   - Support authentication flow redirects with state preservation

3. **Route Guard Integration**
   - Compose protected routes with existing route structure
   - Support multiple scope requirements per route
   - Hierarchical permission checking (admin scope inheritance)
   - Loading states during authentication verification

## TECHNICAL REQUIREMENTS
- Integrate with useAuth hook (to be provided by authentication system)
- TypeScript interfaces for route protection configuration
- React.memo optimization for route guard components
- Proper error boundaries for authentication failures
- Support for route-level scope requirements
- State preservation during authentication redirects

## TESTING REQUIREMENTS
**Unit Tests:**
- Protected route with valid authentication and scopes
- Protected route with invalid/missing authentication
- Protected route with insufficient scopes
- Public route redirect behavior for authenticated users
- Loading state handling during authentication checks

**Integration Tests:**
- Complete authentication flow through route protection
- Scope-based access control validation
- Redirect behavior with state preservation
- Error page display for insufficient permissions

## VALIDATION CRITERIA
- [ ] ProtectedRoute component successfully guards authenticated routes
- [ ] Unauthenticated users redirected to login with proper state
- [ ] PublicRoute component redirects authenticated users appropriately
- [ ] OAuth2 scope validation works correctly for all healthcare roles
- [ ] Insufficient permissions show clear error messages
- [ ] Loading states display during authentication verification
- [ ] Route state preserved during authentication redirects
- [ ] TypeScript types properly defined for route protection
- [ ] Performance maintained during route guard checks

---

# PROMPT 3: Navigation Utilities and Workflow Hooks

**Implement navigation utilities, custom hooks, and helper functions that support healthcare-specific workflows. Create reusable navigation patterns for patient-centric and appointment-centric workflows with proper context management.**

## PROJECT CONTEXT
**MedFlow Healthcare Management Application:**
- Healthcare Workflow Focus: Patient management, appointment scheduling, clinical documentation
- Navigation Patterns: Patient-centric workflows, appointment workflows, clinical documentation flows
- Context Preservation: Patient ID, appointment ID, provider ID maintained across navigation
- User Experience: Streamlined navigation for busy healthcare providers

**Healthcare Workflow Requirements:**
- Patient Context: Navigate within patient profile preserving patient ID
- Appointment Context: Move through appointment workflow maintaining appointment context
- Clinical Workflow: Documentation flows with patient and appointment context
- Provider Context: Multi-provider mode support with provider filtering

## EPIC CONTEXT
**EPIC-000 Project Setup:**
- Foundation for all healthcare feature development
- Navigation utilities will be used by patient management, appointments, and clinical features
- Support both single-doctor and multi-provider operational modes

## TASK CONTEXT
**TASK-009: Frontend Routing System**
- Dependency: Protected route system from Prompt 2 completed  
- Core utilities needed before breadcrumb and URL pattern implementation
- Foundation for healthcare-specific navigation patterns

## FUNCTIONAL REQUIREMENTS
1. **Navigation Hook (useNavigation)**
   - Patient-focused navigation helpers (navigateToPatient with tab support)
   - Appointment workflow navigation (navigateToAppointment with action context)
   - Workflow context management with search parameters
   - Back navigation with intelligent fallback paths
   - Permission-based navigation checking

2. **Workflow Context Management**
   - Context preservation across route transitions
   - Search parameter handling for workflow state
   - Deep linking support with context reconstruction
   - Multi-context support (patient + appointment simultaneously)

3. **Permission-Based Navigation**
   - Route accessibility checking based on user scopes
   - Dynamic navigation menu generation
   - Conditional navigation options based on user roles
   - Healthcare role hierarchy support

## TECHNICAL REQUIREMENTS
- Custom React hooks using useNavigate and useLocation
- TypeScript interfaces for workflow context types
- URL search parameter management for context preservation
- Integration with authentication system for permission checking
- Performance optimized with React.useCallback for navigation functions
- Support for both programmatic and declarative navigation

## TESTING REQUIREMENTS
**Unit Tests:**
- Navigation hook functions (navigateToPatient, navigateToAppointment)
- Workflow context parameter management
- Permission checking logic
- Back navigation with fallback behavior
- Search parameter construction and parsing

**Integration Tests:**
- Navigation with context preservation across routes
- Permission-based navigation restrictions
- Workflow context reconstruction from URLs
- Multi-context navigation scenarios

## VALIDATION CRITERIA
- [ ] useNavigation hook provides all required navigation helpers
- [ ] Patient navigation maintains patient context across workflows
- [ ] Appointment navigation preserves appointment and related context
- [ ] Workflow context properly managed with URL search parameters
- [ ] Permission-based navigation checking works for all healthcare roles
- [ ] Back navigation intelligent with appropriate fallback paths
- [ ] Deep linking reconstructs workflow context from URLs
- [ ] TypeScript types defined for all workflow context interfaces
- [ ] Performance optimized with proper React hook patterns

---

# PROMPT 4: Breadcrumb Navigation System

**Create an intelligent breadcrumb navigation system that understands healthcare workflows and provides contextual navigation paths. Implement automatic breadcrumb generation based on route patterns and healthcare entities (patients, appointments, providers).**

## PROJECT CONTEXT  
**MedFlow Healthcare Management Application:**
- Complex Healthcare Workflows: Multi-step processes requiring clear navigation context
- User Base: Healthcare providers need quick orientation during busy clinical workflows  
- Workflow Depth: Patient → Medical History → Medications → Prescription flow examples
- Context Requirements: Display meaningful labels with entity names (patient names, appointment times)

**Healthcare Navigation Patterns:**
- Patient Workflows: Home → Patients → [Patient Name] → Medical History
- Appointment Workflows: Home → Appointments → Daily View → [Patient Name - 2:30 PM]  
- Clinical Workflows: Home → Clinical → Documentation → [Patient Name] → [Appointment Date]
- Administrative: Home → Billing → Patients → [Patient Name] → Payment History

## EPIC CONTEXT
**EPIC-000 Project Setup:**
- Navigation foundation supporting all future healthcare features
- User experience critical for healthcare provider efficiency
- Breadcrumbs essential for complex multi-step clinical workflows

## TASK CONTEXT
**TASK-009: Frontend Routing System**
- Dependency: Navigation utilities from Prompt 3 completed
- Integration: Route configuration and workflow context management
- Usage: Will be integrated into layout components and page headers

## FUNCTIONAL REQUIREMENTS
1. **Dynamic Breadcrumb Generation**
   - Route-based breadcrumb creation using path segments
   - Healthcare entity resolution (patient names, appointment details)
   - Contextual labeling based on workflow type
   - Active/inactive breadcrumb state management

2. **Healthcare Entity Integration**
   - Patient name resolution from patient ID in URL
   - Appointment details display (time, date) from appointment context
   - Provider name display in multi-provider scenarios
   - Workflow type detection (patient, appointment, clinical, administrative)

3. **Interactive Navigation**
   - Clickable breadcrumb segments for quick navigation
   - Skip-level navigation support (jump from patient to appointments)
   - Breadcrumb truncation for mobile devices
   - Loading states during entity name resolution

## TECHNICAL REQUIREMENTS
- Custom React hook (useBreadcrumbs) for breadcrumb generation
- Integration with navigation utilities and route parameters
- Asynchronous entity name resolution with caching
- Responsive design for mobile breadcrumb display
- TypeScript interfaces for breadcrumb configuration
- Performance optimization with memoization

## TESTING REQUIREMENTS
**Unit Tests:**
- Breadcrumb generation for various route patterns
- Healthcare entity name resolution
- Breadcrumb click navigation behavior
- Mobile truncation logic
- Loading and error states

**Integration Tests:**  
- Complete breadcrumb workflow across healthcare routes
- Entity name resolution with real data
- Breadcrumb navigation with workflow context preservation
- Mobile responsive behavior

## VALIDATION CRITERIA
- [ ] useBreadcrumbs hook generates accurate breadcrumbs for all routes
- [ ] Healthcare entities (patients, appointments) properly resolved and displayed
- [ ] Breadcrumb navigation preserves workflow context
- [ ] Mobile responsive design with appropriate truncation
- [ ] Loading states shown during entity name resolution
- [ ] Error handling for missing/invalid entities
- [ ] Performance optimized with proper memoization
- [ ] TypeScript interfaces defined for breadcrumb configuration
- [ ] Accessibility support for screen readers and keyboard navigation

---

# PROMPT 5: Healthcare URL Patterns Implementation

**Implement the standardized URL patterns and route definitions specific to healthcare workflows. Create the comprehensive routing structure supporting patient management, appointment scheduling, clinical documentation, and billing workflows with proper parameter handling.**

## PROJECT CONTEXT
**MedFlow Healthcare Management Application:**
- Healthcare Workflows: Patient management, appointments, clinical documentation, prescriptions, billing
- Multi-Mode Support: Single-doctor practices and multi-provider clinics
- Deep Linking Requirements: Bookmarkable URLs for quick access to patient records and appointments
- Workflow Efficiency: URLs that support healthcare provider daily routines

**Standardized URL Patterns:**
- Patient Management: `/patients`, `/patients/:patientId`, `/patients/:patientId/medical-history`
- Appointment Management: `/appointments`, `/appointments/:appointmentId/check-in`, `/appointments/daily`
- Clinical Documentation: `/clinical/documentation/:patientId/:appointmentId`, `/prescriptions`
- Billing: `/billing`, `/payments/check-in/:patientId`, `/billing/patients/:patientId`

## EPIC CONTEXT
**EPIC-000 Project Setup:**
- URL structure foundation for all future healthcare features
- Supports both single-doctor and multi-provider deployment modes
- Critical foundation for feature development starting with patient management

## TASK CONTEXT
**TASK-009: Frontend Routing System**
- Dependency: Breadcrumb system from Prompt 4 completed
- Integration: All previous routing components (protected routes, navigation utilities)
- Completion: Final routing system integration ready for authentication and layout systems

## FUNCTIONAL REQUIREMENTS
1. **Patient Management Routes**
   - Patient list/search routes with filtering and pagination support
   - Patient profile routes with tab navigation (overview, history, medications)
   - Patient creation and editing workflows
   - Family relationship management routes

2. **Appointment Workflow Routes**  
   - Appointment dashboard and scheduling routes
   - Daily/weekly schedule views with date parameters
   - Appointment check-in and status tracking routes
   - Waiting list and status management routes

3. **Clinical and Administrative Routes**
   - Clinical documentation routes with patient and appointment context
   - Prescription management and renewal workflows
   - Billing and payment processing routes
   - Provider management routes (multi-provider mode)

4. **Route Parameter Handling**
   - Patient ID, appointment ID, provider ID parameter validation
   - Date parameter parsing and formatting
   - Query parameter handling for filters, pagination, and view modes
   - Modal and overlay state management through URL parameters

## TECHNICAL REQUIREMENTS
- Complete React Router route configuration with nested routes
- Parameter validation and type safety with TypeScript
- Query parameter parsing and state synchronization
- Modal state management through URL parameters
- Route prefetching for improved performance
- SEO-friendly URLs with proper metadata

## TESTING REQUIREMENTS
**Unit Tests:**
- Route parameter extraction and validation
- Query parameter parsing and encoding
- Route matching for all healthcare workflow patterns
- Modal state management through URL parameters

**Integration Tests:**
- Complete healthcare workflow navigation
- Deep linking to specific patient/appointment contexts
- URL parameter persistence across navigation
- Modal and overlay URL state management

**End-to-End Tests:**
- Healthcare provider workflow navigation scenarios
- Bookmark and direct access functionality  
- URL sharing between healthcare team members
- Mobile navigation and URL handling

## VALIDATION CRITERIA
- [ ] All healthcare workflow routes properly configured and accessible
- [ ] Patient ID, appointment ID, and other parameters correctly validated
- [ ] Query parameters support filtering, pagination, and view modes
- [ ] Deep linking works for all healthcare workflow contexts
- [ ] Modal and overlay states managed through URL parameters
- [ ] Route transitions maintain proper loading and error states
- [ ] URL patterns follow healthcare workflow conventions
- [ ] TypeScript types defined for all route parameters
- [ ] SEO and accessibility considerations implemented
- [ ] Performance targets met for route loading and transitions

---

# PROMPT 6: Route System Integration and Testing

**Complete the integration of all routing components and implement comprehensive testing suite. Ensure the entire routing system works cohesively with proper error handling, performance optimization, and preparation for authentication integration.**

## PROJECT CONTEXT
**MedFlow Healthcare Management Application:**
- Production-Ready Requirements: Robust error handling and performance for healthcare environments
- Integration Readiness: Routing system must integrate seamlessly with upcoming authentication and layout systems
- Healthcare Reliability: Mission-critical reliability for patient care workflows

**Quality Standards:**
- Route transition performance < 200ms
- Error recovery without data loss
- Accessibility compliance for healthcare users
- Mobile responsiveness for tablet use in clinical settings

## EPIC CONTEXT  
**EPIC-000 Project Setup:**
- Final routing system deliverable before authentication and layout implementation
- Foundation quality determines success of all future feature development
- Integration testing critical for project momentum

## TASK CONTEXT
**TASK-009: Frontend Routing System - Final Integration**
- Dependencies: All previous prompts (1-5) completed successfully
- Integration: Combine all routing components into cohesive system  
- Deliverable: Complete routing system ready for TASK-010 (Authentication) integration
- Testing: Comprehensive test coverage ensuring system reliability

## FUNCTIONAL REQUIREMENTS
1. **Complete Route System Integration**
   - Combine all routing components (router config, protected routes, navigation utilities, breadcrumbs, URL patterns)
   - Error boundary integration with route-specific error handling
   - Loading state coordination across all routing components
   - Performance optimization with route preloading and caching

2. **Error Handling and Recovery**
   - 404 error pages with helpful navigation options
   - Route error recovery with user context preservation
   - Network error handling during route transitions
   - Graceful degradation for slow connections

3. **Performance Optimization**
   - Route-based code splitting optimization
   - Prefetching strategies for critical healthcare workflows
   - Bundle size analysis and optimization
   - Route transition performance monitoring

## TECHNICAL REQUIREMENTS
- Integration of all routing TypeScript interfaces and types
- Comprehensive error boundary implementation
- Performance monitoring and optimization
- Accessibility testing and compliance
- Mobile responsiveness verification
- Memory leak prevention in route transitions

## TESTING REQUIREMENTS
**Unit Tests:**
- All routing utility functions
- Protected route logic with various authentication states
- Breadcrumb generation for all route patterns  
- Navigation hook functionality
- Error boundary activation and recovery

**Integration Tests:**
- Complete routing system workflow testing
- Authentication integration readiness
- Layout system integration points
- Cross-component state management
- Mobile and desktop navigation flows

**End-to-End Tests:**
- Complete healthcare workflow navigation scenarios
- Error recovery and user experience flows
- Performance testing under various network conditions
- Accessibility compliance verification
- Mobile device testing across healthcare workflow

**Performance Tests:**
- Route transition timing (< 200ms target)
- Bundle size optimization verification
- Memory usage during navigation
- Network request optimization

## VALIDATION CRITERIA
- [ ] All routing components integrated and working cohesively
- [ ] Comprehensive error handling with user-friendly error pages
- [ ] Route transition performance meets < 200ms target
- [ ] All TypeScript types and interfaces properly integrated
- [ ] Complete test suite passes with > 90% coverage
- [ ] Accessibility compliance verified (WCAG 2.1 AA)
- [ ] Mobile responsiveness working across all healthcare workflows  
- [ ] Integration readiness verified for authentication system (TASK-010)
- [ ] Bundle size optimized and acceptable for production
- [ ] Memory leaks prevented and performance monitored
- [ ] Documentation complete for all routing system components