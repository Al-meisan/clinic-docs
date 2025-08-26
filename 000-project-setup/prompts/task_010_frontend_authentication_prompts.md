# PROMPTS: TASK-010 - Frontend Authentication Implementation

# PROMPT 1: Authentication Types and Interfaces Setup

Create comprehensive TypeScript type definitions for the authentication system including user interfaces, authentication states, AWS Cognito payloads, and all related types needed for the MedFlow healthcare management application.

## PROJECT CONTEXT

**Tech Stack:**
- Frontend: React with TypeScript
- Authentication: AWS Cognito
- State Management: Zustand with React Context
- Architecture: Domain-driven design with feature-based folder structure

**Current State:**
- React application is set up with TypeScript support
- Routing system is available  
- UI framework (shadcn/ui) is integrated
- AWS Cognito user pool will be configured

## EPIC CONTEXT

This task is part of the Project Setup Epic focusing on establishing foundational authentication infrastructure. The authentication system must support:
- Healthcare-specific user roles and attributes
- Multi-language support (Arabic and French)
- Role-based access control with OAuth2 scopes
- Clinic association for multi-tenant data isolation

## TASK CONTEXT

**Goal:** Define all TypeScript interfaces and types needed for authentication system including user management, token handling, and AWS Cognito integration.

**Scope:** Create type definitions that will be used across authentication context, hooks, services, and UI components.

## BUSINESS DOMAIN CONTEXT

**Healthcare-Specific Requirements:**
- User roles: System Administrator, Clinic Manager, Healthcare Provider, Clinical Staff, Administrative Staff
- OAuth2 scopes following healthcare workflow patterns (patient:read, clinical:write, etc.)
- Clinic association for proper data isolation
- User preferences including language, timezone, and clinical workflow settings

**Multi-Language Support:**
- Arabic (RTL) and French language preferences
- Internationalized user interface elements
- Regional date/time formatting preferences

## TECHNICAL REQUIREMENTS

**Type Safety:**
- Strict TypeScript configuration with comprehensive type definitions
- No use of 'any' types - all interfaces must be properly typed
- Generic types where appropriate for reusability

**Integration Requirements:**
- Compatibility with AWS Cognito user attributes structure
- Support for JWT token structure and claims
- Integration with React Context and custom hooks patterns

## TESTING REQUIREMENTS

**Type Validation:**
- TypeScript compilation without errors
- Type compatibility with AWS Cognito SDK
- Proper generic constraints and utility types

## VALIDATION CRITERIA

- [ ] User interface includes all required healthcare-specific attributes
- [ ] Authentication state interface covers all possible auth states
- [ ] AWS Cognito payload interface matches expected JWT structure
- [ ] User role enum includes all standardized healthcare roles
- [ ] OAuth2 scopes follow healthcare workflow patterns
- [ ] User preferences interface supports multi-language requirements
- [ ] All interfaces have proper JSDoc documentation
- [ ] Type definitions export from central index file for easy imports

---

# PROMPT 2: AWS Cognito Service Integration Layer

Implement the service layer for AWS Cognito integration including authentication operations, token management, and user profile handling with proper error handling and TypeScript support.

## PROJECT CONTEXT

**Tech Stack:**
- AWS Cognito for user authentication and management
- TypeScript with strict type checking
- React application with modern hooks pattern
- Healthcare domain with specific security requirements

**Dependencies:**
- Authentication types from Prompt 1 are implemented
- AWS Cognito user pool is configured
- AWS SDK or AWS Amplify libraries are available

## EPIC CONTEXT

Core authentication infrastructure requiring robust Cognito integration for:
- Secure user authentication in healthcare environment
- Token lifecycle management with automatic refresh
- Healthcare-specific user attribute handling
- Multi-tenant clinic association management

## TASK CONTEXT

**Goal:** Create a comprehensive service layer that handles all AWS Cognito operations including login, logout, token refresh, and user profile management.

**Scope:** Service class with methods for all authentication operations, proper error handling, and integration with healthcare-specific user attributes.

## BUSINESS DOMAIN CONTEXT

**Healthcare Security Requirements:**
- Secure token storage appropriate for clinical environment
- Automatic logout for security in shared clinical workstations
- Healthcare compliance considerations for user data handling
- Clinic association validation for multi-tenant access control

**User Attribute Management:**
- Custom Cognito attributes for medical license information
- Role-based attribute retrieval and validation
- Clinic association and specialty information
- User preference management (language, timezone, workflow settings)

## FUNCTIONAL REQUIREMENTS

1. **Authentication Operations**
   - User login with email/password credentials
   - Secure logout with complete session cleanup
   - Password reset and change functionality
   - Multi-factor authentication support preparation

2. **Token Management**
   - JWT token parsing and validation
   - Automatic token refresh before expiration
   - Secure token storage with encryption considerations
   - Token cleanup on logout

3. **User Profile Management**
   - User attribute retrieval from Cognito
   - Healthcare-specific custom attribute handling
   - User preference synchronization
   - Profile update operations

## TECHNICAL REQUIREMENTS

**Error Handling:**
- Comprehensive error types for all Cognito operations
- User-friendly error messages without exposing system details
- Network error handling with retry logic
- Authentication failure scenarios

**Performance:**
- Efficient token refresh mechanism
- Minimal API calls through proper caching
- Async/await patterns for all operations
- Memory management for token storage

**Security:**
- Secure credential handling
- Token encryption for storage
- Proper cleanup of sensitive data
- OWASP security best practices

## TESTING REQUIREMENTS

**Unit Tests:**
- Authentication service method testing
- Token management operations
- Error handling scenarios
- User profile operations

**Integration Tests:**
- Mock Cognito service integration
- Token refresh workflow testing
- Complete authentication flow validation

## VALIDATION CRITERIA

- [ ] CognitoAuth class implements all required authentication methods
- [ ] Token management includes automatic refresh before expiration
- [ ] Error handling provides user-friendly messages for all scenarios
- [ ] User profile operations handle healthcare-specific attributes
- [ ] Service integrates properly with TypeScript types from Prompt 1
- [ ] Secure token storage implementation included
- [ ] Service methods return consistent Result<T, Error> patterns
- [ ] All methods include proper JSDoc documentation
- [ ] Service handles network errors and connectivity issues gracefully

---

# PROMPT 3: Authentication Context Provider Implementation

Create the React Context provider for global authentication state management including user state, authentication status, and auth operations with proper integration with the Cognito service layer.

## PROJECT CONTEXT

**Architecture:**
- React application with Context API for global state
- Feature-based folder structure with shared providers
- TypeScript with strict typing requirements
- Integration with custom hooks pattern

**Dependencies:**
- Authentication types from Prompt 1
- AWS Cognito service from Prompt 2
- React application with routing system available

## EPIC CONTEXT

Authentication context serves as the foundation for all authenticated features in the healthcare management system, providing:
- Global user state across all components
- Centralized authentication operations
- Role-based access control data
- Session persistence across browser refreshes

## TASK CONTEXT

**Goal:** Implement AuthProvider component that manages global authentication state and provides authentication operations to the entire application.

**Scope:** React Context provider with authentication state, operations, and proper lifecycle management including initialization and cleanup.

## BUSINESS DOMAIN CONTEXT

**Healthcare Workflow Integration:**
- User context includes role and scope information for UI adaptation
- Clinic association data for multi-tenant functionality
- User preferences for personalized healthcare workflows
- Session management appropriate for clinical environment security

**Multi-Language Support:**
- User language preference management
- Context includes localization state
- Regional formatting preferences

## FUNCTIONAL REQUIREMENTS

1. **Authentication State Management**
   - Current user information and authentication status
   - Loading states for authentication operations
   - Error state management for authentication failures
   - Session persistence across browser refreshes

2. **Authentication Operations**
   - Login operation with credential validation
   - Logout with complete session cleanup
   - Token refresh management
   - Authentication status checking

3. **User Context Data**
   - User profile information including healthcare-specific attributes
   - Role and scope information for access control
   - Clinic association data for multi-tenant features
   - User preferences for workflow customization

## TECHNICAL REQUIREMENTS

**State Management:**
- React Context with useReducer for complex state operations
- Proper state updates with immutable patterns
- Loading and error state handling
- State persistence with localStorage integration

**Performance:**
- Minimize re-renders through proper context value memoization
- Efficient state updates without unnecessary re-computations
- Lazy loading of user profile data
- Optimized context provider hierarchy

**Integration:**
- Seamless integration with Cognito service layer
- Token refresh integration with automatic user session extension
- Error boundary compatibility
- Router integration for redirect handling

## TESTING REQUIREMENTS

**Unit Tests:**
- AuthProvider state management testing
- Authentication operation handling
- Context value updates and memoization
- Error state management

**Integration Tests:**
- Complete authentication flow with service integration
- Token refresh scenario testing
- Session persistence testing
- Error recovery scenarios

## VALIDATION CRITERIA

- [ ] AuthProvider manages all authentication state correctly
- [ ] Authentication operations integrate with Cognito service layer
- [ ] Context provides user, authentication status, and operations
- [ ] Loading states properly managed during async operations
- [ ] Error handling provides clear feedback for authentication failures
- [ ] Session persistence works across browser refreshes
- [ ] Token refresh happens automatically without user interaction
- [ ] Context values are properly memoized to prevent unnecessary re-renders
- [ ] Provider includes proper TypeScript typing with context interface
- [ ] Integration with application routing for redirect handling

---

# PROMPT 4: Authentication Custom Hooks Implementation

Create custom React hooks for authentication operations including useAuth, useUser, and specialized hooks for login/logout operations with proper error handling and loading states.

## PROJECT CONTEXT

**Architecture:**
- Custom hooks pattern for React application
- Integration with authentication context from Prompt 3
- TypeScript with comprehensive type checking
- Healthcare application with specific workflow requirements

**Dependencies:**
- Authentication types from Prompt 1
- Authentication context from Prompt 3
- React hooks and modern patterns

## EPIC CONTEXT

Custom authentication hooks provide the interface between authentication context and application components, enabling:
- Clean component integration with authentication state
- Reusable authentication operations across components
- Standardized error handling and loading states
- Type-safe authentication operations

## TASK CONTEXT

**Goal:** Implement custom React hooks that provide convenient, type-safe access to authentication operations and state for use throughout the application.

**Scope:** Set of custom hooks including useAuth for general authentication, useUser for user-specific operations, and specialized hooks for specific authentication workflows.

## BUSINESS DOMAIN CONTEXT

**Healthcare Workflow Optimization:**
- Hooks optimized for clinical workflow patterns
- Role-based hook behavior for different user types
- Quick access to user permissions and scopes
- Clinic context integration for multi-tenant features

**Security Considerations:**
- Automatic cleanup of sensitive data
- Secure operation handling with proper validation
- Session timeout handling for clinical environment security

## FUNCTIONAL REQUIREMENTS

1. **Primary Authentication Hook (useAuth)**
   - Access to authentication state and user information
   - Authentication operations (login, logout, refresh)
   - Loading and error states for all operations
   - Session status and authentication checks

2. **User-Specific Hook (useUser)**
   - Current user profile information
   - User preferences and settings
   - Role and permission checking utilities
   - Profile update operations

3. **Specialized Operation Hooks**
   - Login operation hook with form integration support
   - Logout hook with cleanup confirmation
   - Token refresh hook for manual refresh scenarios
   - Authentication status checking hook

## TECHNICAL REQUIREMENTS

**Hook Design Patterns:**
- Consistent return value patterns across all hooks
- Proper dependency arrays for useEffect operations
- Custom hook composition for code reuse
- Error boundary compatibility

**Performance Optimization:**
- Memoization of complex computed values
- Efficient re-render prevention
- Selective context subscription patterns
- Lazy evaluation of expensive operations

**Type Safety:**
- Comprehensive TypeScript interfaces for hook returns
- Generic constraints where appropriate
- Proper error type definitions
- Type guards for user role checking

## TESTING REQUIREMENTS

**Unit Tests:**
- Individual hook behavior testing
- Hook return value validation
- Error state handling testing
- Loading state management testing

**Integration Tests:**
- Hook integration with authentication context
- Complete workflow testing with hooks
- Error recovery scenario testing

**Custom Hook Testing:**
- React Testing Library with renderHook
- Mock authentication context for isolated testing
- Async operation testing with proper cleanup

## VALIDATION CRITERIA

- [ ] useAuth hook provides complete authentication interface
- [ ] useUser hook gives access to user profile and permissions
- [ ] All hooks return consistent object shapes with loading/error states
- [ ] Specialized hooks handle specific authentication operations correctly
- [ ] Hook implementations are properly memoized to prevent unnecessary re-renders
- [ ] Error handling in hooks provides clear, actionable error information
- [ ] Hooks integrate seamlessly with authentication context
- [ ] TypeScript types are properly implemented for all hook returns
- [ ] Role and permission checking utilities work with healthcare role definitions
- [ ] Session management hooks handle automatic logout scenarios properly

---

# PROMPT 5: Login Form UI Components Implementation

Create comprehensive login form components including LoginPage, LoginForm, and supporting UI elements with proper validation, error handling, accessibility, and multi-language support.

## PROJECT CONTEXT

**UI Framework:**
- React with TypeScript
- shadcn/ui component library
- Tailwind CSS for styling  
- Multi-language support with i18next
- Accessibility compliance requirements (WCAG 2.1 AA)

**Dependencies:**
- Authentication hooks from Prompt 4
- UI components library (shadcn/ui)
- Form validation library integration
- Internationalization system setup

## EPIC CONTEXT

Login interface serves as the entry point for healthcare professionals accessing the MedFlow system, requiring:
- Professional, trustworthy design appropriate for healthcare setting
- Accessibility for healthcare workers with different abilities
- Multi-language support for Arabic and French
- Robust error handling for various authentication scenarios

## TASK CONTEXT

**Goal:** Create a complete login interface including page layout, form components, validation, and user feedback systems that integrate with the authentication system.

**Scope:** LoginPage component, LoginForm component, and related UI elements including error displays, loading states, and success feedback.

## BUSINESS DOMAIN CONTEXT

**Healthcare Professional Requirements:**
- Clean, professional interface appropriate for clinical setting
- Fast, efficient login process for busy healthcare workers
- Clear error messages that don't expose sensitive system information
- Support for various input methods and accessibility needs

**Security and Compliance:**
- Input validation to prevent common attacks
- Secure credential handling in the UI
- No sensitive data exposure in error messages
- Proper form submission handling for authentication

## FUNCTIONAL REQUIREMENTS

1. **LoginPage Component**
   - Full-page layout for unauthenticated users
   - Professional healthcare-themed design
   - Responsive design for desktop and mobile access
   - Integration with application routing system

2. **LoginForm Component**
   - Email and password input fields with validation
   - Form submission handling with authentication integration
   - Loading states during authentication process
   - Error display for authentication failures
   - Success feedback and redirect handling

3. **Form Validation and UX**
   - Real-time input validation with user-friendly messages
   - Proper form accessibility with labels and ARIA attributes
   - Keyboard navigation support
   - Mobile-responsive form design

## TECHNICAL REQUIREMENTS

**Form Management:**
- React Hook Form for efficient form handling
- Zod schema validation for type-safe form validation
- Integration with authentication hooks from Prompt 4
- Proper form state management and cleanup

**Accessibility (WCAG 2.1 AA):**
- Proper form labels and associations
- ARIA attributes for screen reader support
- Keyboard navigation functionality
- Sufficient color contrast ratios
- Focus management and visual indicators

**Internationalization:**
- Multi-language form labels and messages
- RTL layout support for Arabic
- Localized error messages and validation feedback
- Regional input format support

**Responsive Design:**
- Mobile-first responsive layout
- Touch-friendly form inputs on mobile devices
- Proper viewport handling
- Consistent design across device sizes

## TESTING REQUIREMENTS

**Component Testing:**
- Login form rendering and interaction testing
- Form validation behavior testing
- Error state display testing
- Loading state management testing

**Accessibility Testing:**
- Screen reader compatibility testing
- Keyboard navigation testing
- Color contrast validation
- ARIA attribute correctness

**Integration Testing:**
- Form submission with authentication hooks
- Error handling with authentication service
- Redirect behavior after successful login
- Multi-language functionality testing

## VALIDATION CRITERIA

- [ ] LoginPage provides complete unauthenticated user layout
- [ ] LoginForm handles email/password input with proper validation
- [ ] Form integrates with authentication hooks for login operations
- [ ] Loading states provide clear feedback during authentication
- [ ] Error messages are user-friendly and don't expose system details
- [ ] Form is fully accessible with proper ARIA attributes and keyboard navigation
- [ ] Multi-language support works correctly for Arabic and French
- [ ] Responsive design works properly on mobile and desktop devices
- [ ] Form validation prevents common security vulnerabilities
- [ ] Success states handle proper redirect after authentication
- [ ] Component styling matches healthcare application design system
- [ ] Form performance is optimized with minimal re-renders

---

# PROMPT 6: User Profile Menu and Navigation Components

Create user profile dropdown menu, navigation components, and authentication-related UI elements including user avatar, logout functionality, and role-based navigation visibility.

## PROJECT CONTEXT

**UI Framework:**
- React with TypeScript and shadcn/ui components
- Multi-language support with i18next
- Responsive design with Tailwind CSS
- Integration with routing system for navigation

**Dependencies:**
- Authentication hooks from Prompt 4
- User interface components from shadcn/ui
- Navigation routing system
- Internationalization setup

## EPIC CONTEXT

User navigation components provide authenticated users with:
- Quick access to profile information and settings
- Secure logout functionality
- Role-appropriate navigation options
- Consistent user experience across the healthcare application

## TASK CONTEXT

**Goal:** Implement user profile menu, authentication status indicators, and navigation components that integrate with the authentication system and provide role-based interface elements.

**Scope:** UserMenu dropdown component, user avatar display, navigation elements, and logout confirmation handling with proper integration into application header/navigation areas.

## BUSINESS DOMAIN CONTEXT

**Healthcare Professional Workflow:**
- Quick access to user profile and role information for verification
- Easy logout for security in shared clinical environments
- Role-based navigation showing only relevant features
- Clear indication of current user and clinic association

**Multi-User Environment Considerations:**
- Clear user identification for audit trail purposes
- Quick user switching capabilities preparation
- Role visibility for compliance and workflow clarity
- Clinic context display for multi-tenant awareness

## FUNCTIONAL REQUIREMENTS

1. **User Profile Menu Component**
   - Dropdown menu accessible from application header
   - User avatar with initials or profile image
   - Display of user name, role, and email
   - Navigation to profile and settings pages
   - Secure logout functionality with confirmation

2. **User Avatar and Status Display**
   - User initials or profile picture display
   - Online/offline status indication
   - Role badge or indicator
   - Clinic association display for multi-tenant scenarios

3. **Authentication Navigation Elements**
   - Role-based visibility for navigation items
   - Authentication status integration with routing
   - Breadcrumb integration with user context
   - Quick action buttons for common user operations

## TECHNICAL REQUIREMENTS

**Component Architecture:**
- Reusable components following atomic design principles
- Integration with shadcn/ui dropdown and menu components
- Proper state management for menu open/close states
- TypeScript interfaces for all component props

**Navigation Integration:**
- React Router integration for navigation actions
- Programmatic navigation after logout
- Route protection awareness in navigation elements
- Dynamic menu generation based on user permissions

**User Experience:**
- Smooth animations and transitions
- Accessible dropdown behavior
- Mobile-responsive menu design
- Consistent visual hierarchy

## TESTING REQUIREMENTS

**Component Testing:**
- User menu rendering with different user roles
- Dropdown behavior and interaction testing
- Avatar display with various user data scenarios
- Navigation action testing

**Accessibility Testing:**
- Keyboard navigation for dropdown menus
- Screen reader compatibility
- Focus management in dropdown interactions
- ARIA attributes for menu components

**Integration Testing:**
- Logout flow with authentication system integration
- Navigation integration with routing system
- Role-based visibility testing
- Multi-language menu content testing

## VALIDATION CRITERIA

- [ ] UserMenu dropdown displays user information correctly
- [ ] User avatar shows appropriate initials or profile image
- [ ] Role and clinic information displayed clearly for user verification
- [ ] Logout functionality includes confirmation and proper session cleanup
- [ ] Navigation elements show/hide based on user roles and permissions
- [ ] Menu is fully accessible with keyboard navigation and screen readers
- [ ] Multi-language support works for all menu text and labels
- [ ] Mobile responsive design works properly for touch interactions
- [ ] Integration with authentication hooks provides real-time user state
- [ ] Navigation integrates properly with React Router for page transitions
- [ ] Component styling matches application design system
- [ ] Performance optimized with proper memoization and render optimization

---

# PROMPT 7: Token Management System Implementation

Implement comprehensive JWT token management including secure storage, automatic refresh, expiration handling, and cleanup mechanisms with integration into the authentication system.

## PROJECT CONTEXT

**Security Requirements:**
- Healthcare application requiring secure token handling
- Client-side token storage with security considerations
- Automatic token refresh to maintain user sessions
- Proper cleanup for security in shared clinical environments

**Dependencies:**
- Authentication types from Prompt 1
- AWS Cognito service from Prompt 2
- React application with authentication context

## EPIC CONTEXT

Token management is critical for healthcare application security, providing:
- Seamless user experience with automatic session extension
- Secure token storage appropriate for clinical environment
- Proper session management for compliance requirements
- Integration with API calls for authenticated requests

## TASK CONTEXT

**Goal:** Create a comprehensive token management system that handles JWT tokens securely, provides automatic refresh functionality, and integrates with the authentication system for seamless user experience.

**Scope:** TokenManager service class, secure storage utilities, automatic refresh mechanism, and integration hooks for API requests.

## BUSINESS DOMAIN CONTEXT

**Healthcare Security Requirements:**
- Tokens must be handled securely to protect patient data access
- Automatic logout on token expiration for compliance
- Session management appropriate for shared clinical workstations
- Audit trail considerations for token operations

**Clinical Workflow Considerations:**
- Minimal interruption to clinical workflows during token refresh
- Graceful handling of token expiration during critical operations
- Multi-tab session coordination for clinical efficiency
- Proper cleanup when healthcare workers switch stations

## FUNCTIONAL REQUIREMENTS

1. **Token Storage and Retrieval**
   - Secure storage of access, refresh, and ID tokens
   - Token retrieval with validation and expiration checking
   - Storage cleanup on logout or token invalidation
   - Cross-tab synchronization for consistent session state

2. **Automatic Token Refresh**
   - Background refresh before token expiration
   - Retry logic for failed refresh attempts
   - Graceful fallback to login when refresh fails
   - Event-driven refresh triggering for optimal timing

3. **Token Validation and Security**
   - JWT token parsing and validation
   - Expiration checking with buffer time
   - Token integrity verification
   - Secure cleanup of expired or invalid tokens

## TECHNICAL REQUIREMENTS

**Storage Security:**
- Encrypted token storage where possible
- httpOnly cookie considerations vs localStorage trade-offs
- Secure token transmission and handling
- Memory-only storage for sensitive operations

**Refresh Mechanism:**
- Automatic refresh timing optimization
- Network error handling for refresh operations
- Concurrent refresh request prevention
- Token refresh state management

**Integration Patterns:**
- API client integration for automatic token attachment
- Axios/fetch interceptors for token handling
- Authentication context integration
- Error handling for token-related failures

## TESTING REQUIREMENTS

**Unit Tests:**
- Token storage and retrieval operations
- Token validation and expiration checking
- Automatic refresh mechanism testing
- Security cleanup operations

**Integration Tests:**
- Token refresh workflow with Cognito service
- API request integration with token attachment
- Cross-tab session synchronization testing
- Error recovery scenarios

**Security Testing:**
- Token storage security validation
- Token transmission security testing
- Session cleanup verification
- Token expiration handling

## VALIDATION CRITERIA

- [ ] TokenManager class handles all token lifecycle operations
- [ ] Secure token storage implemented with appropriate security measures
- [ ] Automatic refresh works seamlessly without user interruption
- [ ] Token validation prevents use of expired or invalid tokens
- [ ] API integration automatically attaches valid tokens to requests
- [ ] Cross-tab session synchronization maintains consistent authentication state
- [ ] Token cleanup on logout removes all stored authentication data
- [ ] Error handling gracefully manages token refresh failures
- [ ] Network error resilience included for refresh operations
- [ ] Performance optimized with minimal token operations overhead
- [ ] Integration with authentication context for state synchronization
- [ ] Security best practices followed for token handling in browser environment

---

# PROMPT 8: Authentication Guards and Route Protection Integration

Implement authentication guards, protected route components, and integration with the routing system to ensure proper access control and user redirection based on authentication status and user roles.

## PROJECT CONTEXT

**Routing Architecture:**
- React Router for client-side routing
- Protected routes requiring authentication
- Role-based access control for specific features
- Redirect handling for unauthenticated users

**Dependencies:**
- Authentication context and hooks from previous prompts
- Token management system from Prompt 7
- React Router setup in application
- User role definitions and permission system

## EPIC CONTEXT

Route protection ensures healthcare application security by:
- Preventing unauthorized access to patient data and clinical features
- Implementing role-based access control for different user types
- Providing proper user experience with appropriate redirects
- Maintaining security compliance for healthcare data protection

## TASK CONTEXT

**Goal:** Create authentication guards and protected route components that integrate with the routing system to enforce access control based on authentication status and user roles.

**Scope:** ProtectedRoute component, authentication guards, role-based route protection, and redirect handling for various authentication scenarios.

## BUSINESS DOMAIN CONTEXT

**Healthcare Access Control:**
- Clinical features require appropriate healthcare professional roles
- Administrative functions restricted to management roles
- Patient data access controlled by role and clinic association
- Emergency access considerations for critical healthcare scenarios

**Compliance Requirements:**
- Audit trail for access attempts and denials
- Proper session handling for healthcare compliance
- Role-based access logging for security monitoring
- Clear access control boundaries for different user types

## FUNCTIONAL REQUIREMENTS

1. **Protected Route Component**
   - Authentication status checking before route access
   - Loading states during authentication verification
   - Automatic redirect to login for unauthenticated users
   - Integration with existing routing structure

2. **Role-Based Route Protection**
   - User role validation for protected routes
   - Permission scope checking for granular access control
   - Clinic association validation for multi-tenant routes
   - Graceful handling of insufficient permissions

3. **Authentication Guards**
   - Route guard functions for programmatic protection
   - Higher-order components for route wrapping
   - Custom hooks for component-level protection
   - Guard composition for complex permission scenarios

## TECHNICAL REQUIREMENTS

**Route Integration:**
- Seamless integration with React Router v6
- Proper handling of route parameters and state
- Location preservation for post-login redirects
- Dynamic route generation based on user permissions

**Performance Optimization:**
- Lazy loading of protected components
- Efficient authentication status checking
- Minimal re-renders during auth state changes
- Proper cleanup of route-related subscriptions

**Error Handling:**
- Graceful handling of authentication failures
- Clear user feedback for access denials
- Error boundaries for route protection failures
- Fallback routes for error scenarios

## TESTING REQUIREMENTS

**Route Protection Testing:**
- Unauthenticated user redirect testing
- Role-based access control validation
- Protected route loading behavior testing
- Authentication state change handling

**Integration Testing:**
- Complete authentication flow with route protection
- Role change scenario testing with route re-evaluation
- Multi-tab session handling with route protection
- Error scenario testing with fallback behavior

**User Experience Testing:**
- Login redirect flow testing
- Intended destination preservation
- Loading state behavior during auth verification
- Error message display for access denials

## VALIDATION CRITERIA

- [ ] ProtectedRoute component prevents access for unauthenticated users
- [ ] Role-based protection works correctly with healthcare role definitions
- [ ] Automatic redirect to login preserves intended destination
- [ ] Loading states provide appropriate feedback during auth verification
- [ ] Permission denied scenarios provide clear user feedback
- [ ] Integration with React Router maintains proper navigation behavior
- [ ] Authentication state changes trigger proper route re-evaluation
- [ ] Multi-tenant clinic association checking works for protected routes
- [ ] Performance optimized with minimal authentication checks
- [ ] Error handling provides graceful fallbacks for protection failures
- [ ] Route protection integrates with token management for session validation
- [ ] Accessibility maintained in protected route loading and error states