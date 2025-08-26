# PROMPTS: TASK-011 - State Management Setup

# PROMPT 1: Zustand Global Stores Configuration

Create Zustand stores for global application state management including authentication, UI state, user preferences, and clinic context. Each store should be properly typed, include persistence where appropriate, and follow the established patterns for healthcare applications.

## PROJECT CONTEXT
**Tech Stack**: Frontend uses React with TypeScript, Zustand for lightweight global state management, and localStorage for persistence with proper sanitization of sensitive data.

**Architecture Patterns**: Feature-based folder structure with stores in `src/lib/stores/`, follows store pattern with clear separation of concerns, and uses create() with middleware for persistence.

**Code Standards**: PascalCase for interfaces, camelCase for methods, store files named `*.store.ts`, proper TypeScript typing for all store methods and state properties.

## EPIC CONTEXT  
This is part of the foundational project setup epic that establishes core infrastructure for the clinic management system. State management is critical for maintaining authentication context, UI preferences, and clinic-specific configurations across the application.

## TASK CONTEXT
Task-011 focuses on comprehensive state management setup using Zustand for global state and TanStack Query for server state. This prompt specifically handles the Zustand global stores which provide the foundation for authentication state, UI management, and clinic context.

**Dependencies**: React application with TypeScript (available), routing system (available), folder structure established.

**Provides**: Global state management system accessible throughout the application via custom hooks.

## BUSINESS DOMAIN CONTEXT
**Healthcare Requirements**: 
- Patient data security - no sensitive information in persisted state
- Clinic context switching for multi-provider environments
- User preferences for clinical workflows and interface customization
- Cross-tab synchronization for clinical environments

**Role-Based Access**: State should accommodate different user roles (System Administrator, Clinic Manager, Healthcare Provider, Clinical Staff, Administrative Staff) with appropriate permissions and context.

## FUNCTIONAL REQUIREMENTS
1. **Authentication Store (useAuthStore)**
   - Manages user authentication state, user profile, and preferences
   - Persists non-sensitive authentication state to localStorage
   - Provides methods for login, logout, and preference updates
   - Integrates with AWS Cognito user context

2. **UI Store (useUIStore)**
   - Manages global UI state including notifications, modals, sidebar state, and loading indicators
   - Handles notification queue with auto-removal and different severity levels
   - Provides loading state management for different operations
   - Controls modal and sidebar visibility

3. **Clinic Store (useClinicStore)**
   - Manages current clinic context, settings, and mode (single-doctor vs multi-provider)
   - Handles clinic-specific configurations like timezone, date format, working hours
   - Maintains recent patients list and quick actions
   - Persists clinic preferences and context

## TECHNICAL REQUIREMENTS
**Performance**: State updates must complete in < 16ms to maintain 60fps UI responsiveness
**Security**: No sensitive patient data in localStorage, proper sanitization of persisted state
**Cross-tab Sync**: State synchronization across browser tabs for clinical environments
**Memory Management**: Efficient state updates with minimal re-renders

## TESTING REQUIREMENTS
**Unit Tests**:
- Test all store actions and state updates
- Test persistence middleware functionality  
- Test state cleanup and reset methods
- Verify proper TypeScript typing

**Integration Tests**:
- Test store integration with React components
- Test cross-tab synchronization behavior
- Test state persistence and rehydration
- Verify error handling in store operations

## DOCUMENTATION REQUIREMENTS
- JSDoc comments for all store methods and interfaces
- Usage examples for each store
- Integration patterns with React components
- Performance considerations and best practices

## VALIDATION CRITERIA
- [ ] Authentication store properly manages user state with persistence
- [ ] UI store handles notifications, modals, and loading states correctly
- [ ] Clinic store manages clinic context and preferences
- [ ] All stores include proper TypeScript interfaces and typing
- [ ] Persistence middleware configured correctly with security considerations
- [ ] State updates meet performance requirements (< 16ms)
- [ ] Cross-tab synchronization works for persisted state
- [ ] Unit tests cover all store functionality with >90% coverage
- [ ] No sensitive data persisted in localStorage
- [ ] Error handling implemented for all store operations

# PROMPT 2: TanStack Query Client Configuration

Set up TanStack Query (React Query) client with healthcare-optimized configurations for efficient server state management, caching strategies, and offline capabilities tailored for clinical environments.

## PROJECT CONTEXT
**Tech Stack**: React with TypeScript, TanStack Query for server state management, Axios for API client, and AWS Cognito for authentication.

**Architecture Patterns**: Centralized query client configuration in `src/lib/query.ts`, query hooks organized by feature domains, and consistent error handling across all queries.

**Code Standards**: Query keys follow hierarchical structure, query hooks named `use{Resource}{Action}`, error handling through consistent error types.

## EPIC CONTEXT  
Core infrastructure setup for clinic management system requiring robust server state management for patient data, appointments, and clinical workflows with offline capabilities and real-time synchronization.

## TASK CONTEXT
Task-011 state management setup focusing on server state management. This prompt establishes the TanStack Query foundation that will be used by all feature-specific query hooks for data fetching, caching, and synchronization.

**Dependencies**: Authentication system (in progress), API client configuration (planned), React application with TypeScript (available).

**Provides**: Configured QueryClient and provider setup for server state management throughout the application.

## BUSINESS DOMAIN CONTEXT
**Healthcare Data Patterns**:
- Patient data requires frequent updates and background synchronization  
- Appointment data needs real-time synchronization for scheduling conflicts
- Clinical data should support offline access with conflict resolution
- Reference data (medications, procedures) can be cached for extended periods

**Performance Requirements**: Healthcare apps need immediate data availability during patient care with graceful degradation when offline.

## FUNCTIONAL REQUIREMENTS
1. **QueryClient Configuration**
   - Healthcare-optimized default configurations for different data types
   - Stale-while-revalidate patterns for critical patient data
   - Proper cache invalidation strategies for real-time data
   - Offline-first capabilities with background sync

2. **Query Provider Setup**
   - Global QueryClient provider wrapping the application
   - Integration with authentication for automatic token refresh
   - Error boundary integration for query errors
   - Development tools configuration for debugging

3. **Cache Management**
   - Hierarchical query keys for efficient invalidation
   - Different cache durations for different data types
   - Memory management for large datasets
   - Cache persistence for offline capabilities

## TECHNICAL REQUIREMENTS
**Performance**: 
- Background refetch for stale data without blocking UI
- Query deduplication for concurrent requests
- Optimistic updates for better user experience

**Offline Support**:
- Cache persistence using browser storage
- Queue mutations for online sync
- Conflict resolution for concurrent updates

**Error Handling**:
- Retry strategies for network failures
- Error boundaries for query failures
- User-friendly error messages

## TESTING REQUIREMENTS
**Unit Tests**:
- Test QueryClient configuration options
- Test cache invalidation strategies
- Test query key hierarchies
- Test error handling configurations

**Integration Tests**:
- Test provider setup with authentication
- Test offline/online state transitions
- Test background refetch behavior
- Test error boundary integration

## DOCUMENTATION REQUIREMENTS
- Configuration explanations for healthcare-specific settings
- Query key conventions and patterns
- Cache invalidation strategies
- Error handling patterns and examples

## VALIDATION CRITERIA
- [ ] QueryClient configured with healthcare-optimized settings
- [ ] Query provider properly wraps application with authentication integration
- [ ] Cache strategies configured for different data types (patient, appointment, clinical, reference)
- [ ] Offline capabilities implemented with background sync
- [ ] Error handling configured with retry strategies and user feedback
- [ ] Query key hierarchies established for efficient invalidation
- [ ] Development tools configured for debugging
- [ ] Integration with authentication system working
- [ ] Performance optimization settings applied
- [ ] Unit tests cover QueryClient configuration >85% coverage

# PROMPT 3: Healthcare-Specific Query Hooks

Create feature-specific query hooks for patient data, appointments, and clinical information with healthcare-optimized caching, real-time updates, and offline capabilities.

## PROJECT CONTEXT
**Tech Stack**: React with TypeScript, TanStack Query for server state, custom API client with proper authentication integration.

**Architecture Patterns**: Query hooks organized by feature in `src/features/{feature}/hooks/`, consistent naming patterns, separation of queries and mutations.

**Code Standards**: Hooks named `use{Resource}{Action}`, consistent error handling, proper TypeScript return types, comprehensive loading and error states.

## EPIC CONTEXT
Foundation for all data-driven healthcare workflows requiring optimized data fetching patterns for patient management, appointment scheduling, and clinical documentation.

## TASK CONTEXT
Task-011 state management setup focusing on healthcare-specific query patterns. This builds on the configured QueryClient to provide feature-specific data access patterns optimized for clinical workflows.

**Dependencies**: TanStack Query client configured (from prompt 2), authentication system (in progress), API endpoints defined.

**Provides**: Healthcare-optimized query hooks for patient, appointment, and clinical data access.

## BUSINESS DOMAIN CONTEXT
**Clinical Workflow Patterns**:
- Patient data accessed frequently during appointments with real-time updates
- Appointment scheduling requires conflict detection and real-time availability
- Clinical documentation needs immediate saves with offline support
- Medical history and medications require fast lookup during patient care

**Data Relationships**:
- Patient → Appointments → Clinical Documentation → Prescriptions
- Provider schedules affect appointment availability
- Insurance information affects billing and appointment booking

## FUNCTIONAL REQUIREMENTS
1. **Patient Data Hooks**
   - `usePatients()` for patient search and listing with pagination
   - `usePatient(id)` for individual patient details with automatic refetch
   - `usePatientMedications(patientId)` for current medications
   - `usePatientHistory(patientId)` for medical history with lazy loading

2. **Appointment Data Hooks**
   - `useAppointments(filters)` for appointment listing with real-time updates
   - `useAppointment(id)` for appointment details with status synchronization
   - `useProviderSchedule(providerId, date)` for availability checking
   - `useAppointmentMutations()` for scheduling, updating, and canceling

3. **Clinical Data Hooks**
   - `useClinicalDocumentation(patientId, appointmentId)` for clinical notes
   - `usePrescriptions(patientId)` for prescription management
   - `useVitalSigns(patientId)` for patient vital signs history

## TECHNICAL REQUIREMENTS
**Cache Optimization**:
- Patient data: 5-minute stale time with background updates
- Appointment data: 30-second stale time with real-time invalidation
- Clinical data: Immediate invalidation on updates
- Reference data: 24-hour cache duration

**Real-time Updates**:
- Automatic invalidation on data changes
- Optimistic updates for immediate feedback
- Conflict resolution for concurrent edits

**Performance**:
- Query deduplication for concurrent requests
- Prefetching for predictable navigation patterns
- Infinite queries for large datasets

## TESTING REQUIREMENTS
**Unit Tests**:
- Test all query hooks with mock data
- Test error handling and loading states
- Test cache invalidation patterns
- Test optimistic updates behavior

**Integration Tests**:
- Test hooks with actual API integration
- Test real-time update scenarios
- Test offline/online transitions
- Test concurrent user scenarios

## DOCUMENTATION REQUIREMENTS
- Usage examples for each hook
- Cache behavior documentation
- Error handling patterns
- Performance optimization tips

## VALIDATION CRITERIA
- [ ] Patient query hooks implemented with proper caching and real-time updates
- [ ] Appointment query hooks support scheduling workflows with conflict detection
- [ ] Clinical data hooks provide immediate saves and offline support
- [ ] All hooks include comprehensive error handling and loading states
- [ ] Cache strategies optimized for healthcare data patterns
- [ ] Optimistic updates implemented for critical user interactions
- [ ] Query key hierarchies enable efficient invalidation
- [ ] Infinite queries implemented for large datasets (patient lists, history)
- [ ] Real-time invalidation working for concurrent user scenarios
- [ ] Unit tests cover all hooks with >90% coverage
- [ ] Performance optimizations applied (prefetching, deduplication)
- [ ] TypeScript interfaces properly defined for all query returns

# PROMPT 4: Error Handling and Loading States Integration

Implement comprehensive error handling and loading state management that integrates with both Zustand global state and TanStack Query, providing consistent user feedback across all data operations.

## PROJECT CONTEXT
**Tech Stack**: React with TypeScript, Zustand for global state, TanStack Query for server state, consistent error handling patterns across all features.

**Architecture Patterns**: Centralized error handling through global interceptors, loading states managed through both global UI store and query-specific states, consistent user feedback patterns.

**Code Standards**: Standardized error types, consistent loading state names, user-friendly error messages with localization support.

## EPIC CONTEXT
Critical infrastructure for healthcare application requiring reliable error handling and user feedback during clinical workflows where data accuracy and availability are crucial.

## TASK CONTEXT
Task-011 state management setup focusing on error handling and loading states. This integrates with the Zustand stores and TanStack Query configuration to provide comprehensive user feedback systems.

**Dependencies**: Zustand stores configured (from prompt 1), TanStack Query client configured (from prompt 2), healthcare query hooks (from prompt 3).

**Provides**: Integrated error handling and loading state management across global and server state.

## BUSINESS DOMAIN CONTEXT
**Healthcare Error Scenarios**:
- Network interruptions during patient data access
- Concurrent appointment scheduling conflicts  
- Clinical data validation errors during documentation
- Authentication token expiration during active sessions
- Offline mode transitions during patient care

**User Experience Requirements**:
- Non-disruptive error handling during clinical workflows
- Clear feedback for data validation errors
- Graceful degradation for network issues
- Retry mechanisms for critical operations

## FUNCTIONAL REQUIREMENTS
1. **Global Error Handler**
   - Centralized error processing through UI store
   - Different error types with appropriate user messaging
   - Error logging and reporting for clinical audit requirements
   - Integration with notification system for user feedback

2. **Loading State Management**  
   - Global loading indicators for application-wide operations
   - Query-specific loading states for individual data operations
   - Loading state coordination to prevent UI conflicts
   - Progress indicators for long-running operations

3. **Retry and Recovery Mechanisms**
   - Automatic retry for network failures
   - Manual retry options for user-initiated recovery
   - Optimistic update rollback for failed mutations
   - Cache fallback for offline scenarios

4. **User Feedback Integration**
   - Toast notifications for operation feedback
   - Modal dialogs for critical errors requiring user action
   - Inline error messages for form validation
   - Loading spinners and progress indicators

## TECHNICAL REQUIREMENTS
**Error Handling**:
- HTTP error classification (4xx client errors, 5xx server errors, network errors)
- Business logic error handling with specific error codes
- Authentication error handling with automatic token refresh
- Validation error display with field-specific messaging

**Loading States**:
- Debounced loading indicators to prevent flickering
- Loading state priority system for concurrent operations
- Progress tracking for file uploads and data synchronization
- Skeleton states for content loading

**Performance**:
- Error boundary implementation to prevent application crashes
- Memory-efficient error state management
- Loading state cleanup to prevent memory leaks

## TESTING REQUIREMENTS
**Unit Tests**:
- Test error handler for different error types
- Test loading state coordination
- Test retry mechanisms
- Test notification system integration

**Integration Tests**:
- Test error handling with actual API failures
- Test loading states during real data operations
- Test offline/online error scenarios
- Test user interaction with error recovery

**Error Scenario Tests**:
- Network failure during critical operations
- Authentication expiration during active sessions
- Concurrent data modification conflicts
- Invalid data submission scenarios

## DOCUMENTATION REQUIREMENTS
- Error handling patterns and best practices
- Loading state management guidelines
- User feedback design patterns
- Troubleshooting guide for common errors

## VALIDATION CRITERIA
- [ ] Global error handler processes all error types with appropriate user messaging
- [ ] Loading states managed consistently across global and query states
- [ ] Retry mechanisms implemented for recoverable errors
- [ ] Toast notification system integrated with error and success states
- [ ] Modal error dialogs for critical errors requiring user action
- [ ] Inline error messages for form validation and field-specific errors
- [ ] Loading indicators implemented with debouncing to prevent flickering
- [ ] Error boundaries prevent application crashes from unhandled errors
- [ ] Offline error scenarios handled gracefully with cache fallbacks
- [ ] Authentication errors trigger appropriate token refresh or re-login
- [ ] Progress indicators implemented for long-running operations
- [ ] Error logging configured for audit and debugging purposes
- [ ] User feedback provides clear next steps for error recovery
- [ ] Unit tests cover error handling scenarios >85% coverage

# PROMPT 5: Provider Setup and Integration Testing

Create the provider hierarchy that integrates all state management components and implement comprehensive integration tests to ensure proper functionality across the entire state management system.

## PROJECT CONTEXT
**Tech Stack**: React with TypeScript, provider pattern for dependency injection, comprehensive testing with React Testing Library, Jest for unit testing.

**Architecture Patterns**: Provider composition pattern in `src/providers/`, hierarchical provider structure, proper provider ordering for dependencies.

**Code Standards**: Providers named `{Feature}Provider`, consistent prop interfaces, proper error boundary integration, comprehensive test coverage.

## EPIC CONTEXT
Completes the foundational state management infrastructure by ensuring all components work together seamlessly and providing the testing foundation for reliable healthcare application operation.

## TASK CONTEXT
Task-011 state management setup focusing on provider integration and comprehensive testing. This finalizes the state management system by ensuring all components work together and are thoroughly tested.

**Dependencies**: Zustand stores (prompt 1), TanStack Query client (prompt 2), query hooks (prompt 3), error handling (prompt 4).

**Provides**: Complete state management system ready for use by all application features.

## BUSINESS DOMAIN CONTEXT
**Healthcare Integration Requirements**:
- State management must support multiple concurrent users in clinical settings
- Data consistency across different workflows (patient care, scheduling, billing)
- Reliable state persistence for clinical documentation and patient safety
- Proper error handling to prevent data loss during critical operations

**Clinical Workflow Integration**:
- Patient context maintained across appointment workflows
- Provider switching without data loss
- Clinic context preservation during multi-location operations

## FUNCTIONAL REQUIREMENTS
1. **AppProvider Setup**
   - Root provider that composes all state management providers
   - Proper provider ordering to handle dependencies
   - Error boundary integration for provider failures
   - Development vs production configuration handling

2. **Provider Composition**
   - QueryClient provider with authentication integration
   - I18n provider for internationalization support
   - Theme provider for UI consistency
   - Error boundary providers for graceful error handling

3. **Integration Testing Suite**
   - Component integration with all providers
   - Cross-provider communication testing
   - State persistence and rehydration testing
   - Error handling integration testing

4. **Development Tools Integration**
   - React Query DevTools for development
   - Zustand DevTools integration
   - State inspection capabilities for debugging

## TECHNICAL REQUIREMENTS
**Provider Architecture**:
- Minimal provider re-renders through proper memoization
- Provider dependency management and ordering
- Proper cleanup on provider unmount
- Error isolation between providers

**Testing Infrastructure**:
- Custom test utilities with provider wrapping
- Mock implementations for testing scenarios
- Test data factories for consistent test setup
- Integration test environment configuration

**Development Experience**:
- Hot module replacement support for state changes
- DevTools integration for state inspection
- Performance profiling capabilities
- Error tracking and debugging support

## TESTING REQUIREMENTS
**Integration Tests**:
- Test complete provider hierarchy setup
- Test state management across provider boundaries
- Test authentication integration with query client
- Test error handling across the entire stack

**End-to-End State Tests**:
- Test complete user workflows with state management
- Test offline/online state transitions
- Test concurrent user scenarios
- Test data consistency across browser tabs

**Performance Tests**:
- Test provider rendering performance
- Test state update performance under load
- Test memory usage with large datasets
- Test cleanup and garbage collection

## DOCUMENTATION REQUIREMENTS
- Provider setup and configuration guide
- Integration testing patterns and examples
- State management architecture overview
- Troubleshooting guide for common issues

## VALIDATION CRITERIA
- [ ] AppProvider properly composes all state management providers
- [ ] Provider dependencies handled correctly with proper ordering
- [ ] QueryClient integrated with authentication and error handling
- [ ] I18n provider configured for Arabic and French language support
- [ ] Theme provider integrated with UI components
- [ ] Error boundaries prevent provider failures from crashing application
- [ ] Development tools configured for state inspection and debugging
- [ ] Custom test utilities created for provider-wrapped component testing
- [ ] Integration tests cover cross-provider communication scenarios
- [ ] State persistence and rehydration tested across browser sessions
- [ ] Authentication integration tested with automatic token refresh
- [ ] Error handling integration tested across global and query states
- [ ] Performance requirements met (provider setup < 100ms)
- [ ] Memory management tested with cleanup verification
- [ ] Concurrent user scenarios tested and working
- [ ] Documentation complete with setup and troubleshooting guides