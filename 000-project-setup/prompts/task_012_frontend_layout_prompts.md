# PROMPTS: TASK-012 - Frontend Layout System

# PROMPT 1: Core Layout Components Foundation

Create the foundational layout components structure including DashboardLayout, AuthLayout, and the base layout composition pattern that will serve as the foundation for the clinic management application's user interface.

## PROJECT CONTEXT

**Tech Stack:**
- Frontend: React with TypeScript
- UI Components: shadcn/ui
- Architecture: Feature-based folder structure
- State Management: Zustand stores
- Routing: React Router
- Styling: Tailwind CSS with responsive design

**Current Project State:**
- React application setup complete (TASK-007)
- Authentication system in progress (TASK-010)
- Routing system available
- State management infrastructure ready

## EPIC CONTEXT

**From Healthcare Management Epic:**
- Layout must be professional and appropriate for medical environments
- Support both single-doctor and multi-provider clinic modes
- Responsive design optimized for clinical workstations and tablets
- Integration with authentication system for role-based navigation

## TASK CONTEXT

**Goal:** Create comprehensive responsive layout system supporting both single-doctor and multi-provider clinic modes, with healthcare-optimized navigation, contextual breadcrumbs, and workflow-aware interface adaptations.

**Scope:** HIGH complexity FRONTEND task focused on layout architecture and component composition.

## BUSINESS DOMAIN CONTEXT

**Clinic Mode Adaptations:**
- Single-doctor mode: Simplified navigation with essential features only
- Multi-provider mode: Comprehensive navigation with full feature access
- Professional medical interface styling with efficient space utilization
- WCAG 2.1 AA accessibility compliance required

## FUNCTIONAL REQUIREMENTS

1. **DashboardLayout Component**
   - Main authenticated user layout with sidebar and content areas
   - Support for collapsible sidebar functionality
   - Integration with authentication context for user information
   - Responsive behavior for desktop and tablet devices

2. **AuthLayout Component**
   - Clean, minimal layout for authentication pages (login, register, forgot password)
   - Centered content with healthcare branding
   - No navigation elements, focused on authentication flow

3. **Layout Composition Pattern**
   - Outlet integration for React Router content rendering
   - Consistent layout structure across all application areas
   - Support for nested layouts and content areas

## TECHNICAL REQUIREMENTS

**Performance:**
- Layout rendering performance < 100ms for transitions and responsive breakpoint changes
- Optimized component re-rendering with React.memo where appropriate

**Accessibility:**
- WCAG 2.1 AA compliance for all layout elements
- Keyboard navigation support
- Screen reader compatibility with proper ARIA attributes

**Compatibility:**
- Desktop (1920x1080) optimal experience
- Tablet (1024x768) touch-friendly adaptation
- Mobile (390x844) basic functionality fallback

## TESTING REQUIREMENTS

**Unit Tests:**
- Layout component rendering with different props
- Responsive behavior at various breakpoints
- Authentication context integration
- Component composition and outlet rendering

**Integration Tests:**
- Layout integration with React Router
- Authentication state changes affecting layout
- Responsive transitions between breakpoints

**Manual Tests:**
- Layout functionality across target device sizes
- Accessibility testing with keyboard navigation
- Screen reader compatibility verification

## DOCUMENTATIONS REQUIREMENTS

- Component documentation with props and usage examples
- Layout composition patterns and best practices guide
- Responsive design implementation notes

## VALIDATION CRITERIA
- [ ] DashboardLayout component renders with sidebar and main content areas
- [ ] AuthLayout component provides clean authentication page structure
- [ ] Layout components integrate properly with React Router Outlet
- [ ] Responsive behavior works at desktop and tablet breakpoints
- [ ] Accessibility requirements met with keyboard navigation
- [ ] Component composition pattern allows for flexible content rendering
- [ ] Performance targets met for layout rendering and transitions
- [ ] All unit and integration tests pass with >85% coverage

---

# PROMPT 2: Navigation System with shadcn/ui Sidebar

Implement the main navigation system using shadcn/ui sidebar component with role-based menu items, collapsible functionality, and support for simplified and comprehensive navigation modes.

## PROJECT CONTEXT

**Tech Stack:**
- shadcn/ui sidebar component (https://ui.shadcn.com/docs/components/sidebar)
- React Router for navigation
- Zustand for UI state management
- TypeScript for type safety
- Tailwind CSS for styling

**Current Project State:**
- Core layout components foundation established (Prompt 1)
- Authentication system provides user context and scopes
- React Router configuration available
- UI state management store ready

## EPIC CONTEXT

**Healthcare Workflow Integration:**
- Navigation must reflect healthcare workflows (Patients → Appointments → Clinical → Billing)
- Role-based access with OAuth2 scope-based permissions
- Professional medical interface with intuitive navigation patterns

## TASK CONTEXT

**Focus:** Main navigation sidebar with role-based menu items and clinic mode adaptation using shadcn/ui sidebar component.

**Integration:** Works within DashboardLayout component established in Prompt 1.

## BUSINESS DOMAIN CONTEXT

**Navigation Structure:**
- Dashboard (always visible)
- Patients (patient:read scope required)
- Appointments (appointment:read scope required)  
- Clinical Documentation (clinical:read scope required)
- Prescriptions (prescription:read scope required)
- Billing (billing:read scope, multi-provider mode only)
- Settings (clinic:manage scope, multi-provider mode only)

**Role-Based Visibility:**
- System Administrator: Full access to all menu items
- Clinic Manager: All operational menus
- Healthcare Provider: Clinical and patient-focused menus
- Clinical Staff: Limited clinical and patient menus
- Administrative Staff: Patient and appointment menus

## FUNCTIONAL REQUIREMENTS

1. **MainNavigation Component**
   - Implement using shadcn/ui sidebar component
   - Support collapsed and expanded states
   - Role-based menu item visibility using OAuth2 scopes
   - Simplified vs comprehensive mode switching

2. **Navigation Menu Structure**
   - Hierarchical navigation with expandable sections
   - Active state indication for current route
   - Icon-based navigation with text labels
   - Support for nested menu items where appropriate

3. **Responsive Behavior**
   - Auto-collapse on mobile devices
   - Touch-friendly interaction for tablet users
   - Overlay mode for mobile navigation

## TECHNICAL REQUIREMENTS

**State Management:**
- Integration with UI store for sidebar collapsed state
- Navigation state persistence across page loads
- Route-based active state management

**Performance:**
- Efficient re-rendering when navigation state changes
- Lazy loading of navigation icons
- Smooth transition animations

**Accessibility:**
- Keyboard navigation between menu items
- Screen reader announcements for navigation state changes
- Focus management during expand/collapse operations

## TESTING REQUIREMENTS

**Unit Tests:**
- Navigation component rendering with different user scopes
- Menu item visibility based on authentication context
- Collapsed/expanded state transitions
- Active route highlighting

**Integration Tests:**
- Navigation integration with React Router
- Role-based menu filtering with authentication context
- Clinic mode switching affecting menu visibility

**Manual Tests:**
- Navigation accessibility with keyboard and screen reader
- Touch interaction testing on tablet devices
- Visual testing across different user roles

## DOCUMENTATIONS REQUIREMENTS

- Navigation component API documentation
- Role-based navigation configuration guide
- shadcn/ui sidebar customization notes

## VALIDATION CRITERIA
- [ ] MainNavigation component implemented using shadcn/ui sidebar
- [ ] Navigation menu items filter based on user OAuth2 scopes
- [ ] Collapsed and expanded states work properly with smooth transitions
- [ ] Active route highlighting shows current page location
- [ ] Simplified mode hides multi-provider specific menu items
- [ ] Responsive behavior works on mobile and tablet devices
- [ ] Keyboard navigation between menu items functions correctly
- [ ] Role-based visibility correctly shows/hides menu items
- [ ] Integration with UI store maintains navigation state
- [ ] All accessibility requirements met with ARIA attributes

---

# PROMPT 3: Top Bar Component with User Profile and Quick Actions

Create the top bar component featuring user profile access, notification system, quick actions menu, and global search functionality integrated with the healthcare workflow context.

## PROJECT CONTEXT

**Tech Stack:**
- shadcn/ui components (Button, Input, Badge, DropdownMenu)
- Lucide React icons for consistent iconography
- React hooks for state management
- Authentication context for user information

**Current Project State:**
- Core layout components and navigation system established (Prompts 1-2)
- Authentication system provides user context
- UI state management for layout preferences available

## EPIC CONTEXT

**Healthcare Workflow Requirements:**
- Quick access to frequently used actions (New Patient, Schedule Appointment)
- Notification system for clinical alerts and reminders
- User profile with role and clinic information
- Global search for patients and appointments

## TASK CONTEXT

**Focus:** Top bar component that provides global navigation, user context, and quick access to common healthcare workflows.

**Integration:** Integrates with DashboardLayout and MainNavigation components from previous prompts.

## BUSINESS DOMAIN CONTEXT

**Quick Actions by Role:**
- Healthcare Provider: New Patient, Schedule Appointment, Clinical Documentation
- Clinical Staff: Schedule Appointment, Patient Search
- Administrative Staff: New Patient, Schedule Appointment, Payment Processing
- Clinic Manager: All actions plus reports and settings access

**Notification Types:**
- Appointment reminders and changes
- Clinical alerts (prescription interactions, allergies)
- System notifications (failed payments, overdue tasks)
- Administrative alerts (pending approvals, staff notifications)

## FUNCTIONAL REQUIREMENTS

1. **TopBar Component Structure**
   - Mobile-responsive header bar spanning full width
   - Left side: hamburger menu for mobile navigation toggle
   - Center: global search functionality
   - Right side: notifications, quick actions, user profile menu

2. **User Profile Dropdown**
   - Display user name, role, and current clinic
   - Access to user settings and preferences
   - Logout functionality
   - Role and permission indicator

3. **Notification System**
   - Notification badge with unread count
   - Dropdown with categorized notifications
   - Mark as read/unread functionality
   - Priority-based notification ordering

4. **Quick Actions Menu**
   - Role-based action visibility
   - Common workflow shortcuts
   - Keyboard shortcut indicators
   - Recent actions history

5. **Global Search**
   - Patient search by name, ID, or phone
   - Appointment search functionality
   - Real-time search suggestions
   - Search history for frequent searches

## TECHNICAL REQUIREMENTS

**Performance:**
- Debounced search input to prevent excessive API calls
- Efficient notification polling without blocking UI
- Optimized dropdown rendering for large notification lists

**State Management:**
- Integration with authentication context for user data
- UI state management for dropdown open/closed states
- Search state management with history persistence

**Accessibility:**
- Keyboard navigation through all interactive elements
- Screen reader announcements for notifications
- Focus management in dropdown menus

## TESTING REQUIREMENTS

**Unit Tests:**
- TopBar component rendering with different user roles
- Notification dropdown functionality
- Quick actions menu filtering based on user permissions
- Global search input handling and debouncing

**Integration Tests:**
- Integration with authentication context for user data
- Notification system integration with backend services
- Search functionality integration with patient/appointment APIs

**Manual Tests:**
- Accessibility testing with keyboard navigation
- Mobile responsiveness testing
- User experience testing for quick action workflows

## DOCUMENTATIONS REQUIREMENTS

- TopBar component API and props documentation
- Quick actions configuration guide
- Notification system integration documentation

## VALIDATION CRITERIA
- [ ] TopBar component renders with all required sections (menu, search, actions, profile)
- [ ] User profile dropdown displays correct user information and role
- [ ] Notification system shows unread count and categorized notifications
- [ ] Quick actions menu filters based on user role and permissions
- [ ] Global search provides real-time suggestions with debouncing
- [ ] Mobile hamburger menu toggles navigation properly
- [ ] All dropdown menus have proper keyboard navigation
- [ ] Notification badge updates in real-time with new notifications
- [ ] Search functionality integrates with patient/appointment data
- [ ] Component integrates properly with existing layout structure

---

# PROMPT 4: Contextual Breadcrumb Navigation System

Implement a workflow-aware breadcrumb navigation system that shows current context (patient, appointment, clinical workflow) and provides clickable navigation throughout healthcare workflows.

## PROJECT CONTEXT

**Tech Stack:**
- React Router for route context and navigation
- TypeScript for type safety with breadcrumb definitions
- shadcn/ui components for consistent styling
- Context providers for patient/appointment state

**Current Project State:**
- Layout foundation and navigation established (Prompts 1-3)
- React Router configuration provides route context
- Patient and appointment context providers available

## EPIC CONTEXT

**Healthcare Workflow Context:**
- Breadcrumbs must show clinical workflow progression
- Patient context preservation across different views
- Appointment context during clinical documentation
- Clear navigation hierarchy for complex medical workflows

## TASK CONTEXT

**Focus:** Contextual breadcrumb system that adapts to healthcare workflows and maintains patient/appointment context throughout navigation.

**Integration:** Works within DashboardLayout below TopBar and above main content area.

## BUSINESS DOMAIN CONTEXT

**Healthcare Navigation Patterns:**
- Home → Patients → John Doe → Medical History
- Home → Appointments → Daily Schedule → Appointment Details → Clinical Documentation
- Home → Patients → Jane Smith → Prescriptions → New Prescription
- Home → Billing → Patient Billing → Payment History

**Context Preservation:**
- Patient context: Maintain patient information across different views
- Appointment context: Show appointment details during clinical workflows
- Clinical workflow: Track progression through documentation steps

## FUNCTIONAL REQUIREMENTS

1. **AppBreadcrumbs Component**
   - Dynamic breadcrumb generation based on current route
   - Patient and appointment context integration
   - Clickable navigation to previous levels
   - Workflow-aware breadcrumb formatting

2. **Context Integration**
   - Patient name display in breadcrumbs when in patient context
   - Appointment date/time in clinical workflow breadcrumbs
   - Provider context in multi-provider environments
   - Clinic context in system administration areas

3. **Breadcrumb Generation Logic**
   - Route-based breadcrumb building
   - Context injection for dynamic segments
   - Simplified breadcrumbs in single-doctor mode
   - Permissions-aware breadcrumb visibility

4. **Navigation Functionality**
   - Clickable breadcrumb segments for quick navigation
   - Preserve context when navigating to parent levels
   - Handle invalid navigation attempts gracefully
   - Maintain current page state during breadcrumb navigation

## TECHNICAL REQUIREMENTS

**Route Integration:**
- React Router location and params integration
- Dynamic route parameter resolution
- Nested route breadcrumb support

**State Management:**
- Integration with patient and appointment context providers
- Navigation state preservation
- Breadcrumb history for complex workflows

**Performance:**
- Efficient breadcrumb generation without excessive re-rendering
- Cached context lookups for frequently accessed data

## TESTING REQUIREMENTS

**Unit Tests:**
- Breadcrumb generation logic with different route scenarios
- Context integration with patient/appointment data
- Click navigation functionality
- Simplified mode breadcrumb filtering

**Integration Tests:**
- Breadcrumb integration with React Router
- Patient/appointment context provider integration
- Navigation preservation across route changes

**Manual Tests:**
- Breadcrumb navigation across different healthcare workflows
- Context preservation testing in complex navigation scenarios
- Accessibility testing with screen readers

## DOCUMENTATIONS REQUIREMENTS

- Breadcrumb component configuration documentation
- Healthcare workflow breadcrumb patterns guide
- Context integration implementation notes

## VALIDATION CRITERIA
- [ ] AppBreadcrumbs component generates dynamic breadcrumbs based on current route
- [ ] Patient context displays correctly in breadcrumbs (patient name, ID)
- [ ] Appointment context shows in clinical workflow breadcrumbs
- [ ] All breadcrumb segments are clickable and navigate properly
- [ ] Simplified mode shows appropriate breadcrumb levels
- [ ] Context is preserved when navigating via breadcrumbs
- [ ] Invalid navigation attempts are handled gracefully
- [ ] Breadcrumbs integrate properly with existing layout structure
- [ ] Accessibility requirements met with keyboard navigation
- [ ] Performance requirements met for breadcrumb generation

---

# PROMPT 5: Responsive Design Integration and Mobile Adaptations

Implement responsive design features including mobile sidebar overlay, touch-friendly interactions, and adaptive layout behavior across desktop, tablet, and mobile device sizes.

## PROJECT CONTEXT

**Tech Stack:**
- Tailwind CSS for responsive utilities and breakpoints
- CSS media queries for advanced responsive behavior
- React hooks (useEffect, useState) for responsive state management
- Touch event handling for mobile interactions

**Current Project State:**
- Core layout components established (Prompts 1-4)
- Navigation and breadcrumb systems implemented
- Desktop layout functionality complete

## EPIC CONTEXT

**Cross-Device Healthcare Access:**
- Clinical workstations (desktop) primary interface
- Tablet access for bedside and mobile clinical use
- Emergency mobile access for critical healthcare functions
- Touch-friendly interfaces for clinical staff mobility

## TASK CONTEXT

**Focus:** Responsive design implementation ensuring optimal user experience across all device sizes while maintaining healthcare workflow efficiency.

**Integration:** Enhances existing layout components with responsive behavior and mobile adaptations.

## BUSINESS DOMAIN CONTEXT

**Device Usage Patterns:**
- Desktop (1920x1080): Primary clinical workstation interface
- Tablet (1024x768): Mobile clinical use, bedside documentation
- Mobile (390x844): Emergency access, basic task completion

**Responsive Priorities:**
- Desktop: Full feature access, multi-panel layouts
- Tablet: Touch-optimized, streamlined workflows
- Mobile: Essential functions only, simplified navigation

## FUNCTIONAL REQUIREMENTS

1. **Mobile Sidebar Overlay**
   - Slide-out navigation overlay for mobile devices
   - Touch-friendly interaction with swipe gestures
   - Backdrop overlay with tap-to-close functionality
   - Smooth animation transitions

2. **Responsive Layout Adaptations**
   - Desktop: Fixed sidebar with collapsible functionality
   - Tablet: Overlay sidebar with touch optimization
   - Mobile: Full-screen overlay with simplified menu

3. **Touch Interaction Optimization**
   - Increased touch target sizes for mobile devices
   - Touch-friendly spacing and padding
   - Swipe gesture support for navigation
   - Optimized scroll behavior

4. **Breakpoint Management**
   - Consistent breakpoint definitions across components
   - Responsive state management with React hooks
   - Performance-optimized resize handling

## TECHNICAL REQUIREMENTS

**Responsive Breakpoints:**
- Mobile: < 768px
- Tablet: 768px - 1024px  
- Desktop: > 1024px

**Performance:**
- Efficient resize event handling with debouncing
- Optimized responsive image loading
- Minimal layout shift during responsive transitions

**Touch Optimization:**
- Minimum 44px touch targets
- Appropriate spacing for thumb navigation
- Optimized scrolling performance on mobile devices

## TESTING REQUIREMENTS

**Unit Tests:**
- Responsive hook functionality across breakpoints
- Mobile overlay state management
- Touch event handling logic

**Integration Tests:**
- Responsive behavior integration with existing layout components
- Navigation state management across device sizes
- Performance testing for responsive transitions

**Manual Tests:**
- Cross-device responsive behavior testing
- Touch interaction testing on actual mobile devices
- Accessibility testing with mobile screen readers

## DOCUMENTATIONS REQUIREMENTS

- Responsive design implementation guide
- Mobile interaction patterns documentation
- Breakpoint and media query reference

## VALIDATION CRITERIA
- [ ] Mobile sidebar overlay functions with smooth animations
- [ ] Touch targets meet minimum size requirements (44px)
- [ ] Responsive breakpoints work correctly across target devices
- [ ] Navigation automatically adapts based on screen size
- [ ] Swipe gestures work for mobile navigation
- [ ] Performance targets met for responsive transitions
- [ ] Layout maintains functionality across all device sizes
- [ ] Touch interactions are optimized for mobile use
- [ ] Backdrop overlay closes navigation on tap/click
- [ ] All responsive features integrate with existing layout components

---

# PROMPT 6: Multi-Mode Layout System (Single Doctor vs Multi-Provider)

Implement the clinic mode adaptation system that switches between simplified single-doctor mode and comprehensive multi-provider mode layouts based on clinic configuration.

## PROJECT CONTEXT

**Tech Stack:**
- Zustand store for clinic configuration state
- React context for mode-aware component behavior
- TypeScript interfaces for mode-specific configurations
- Conditional rendering patterns for feature visibility

**Current Project State:**
- Complete layout system with responsive design (Prompts 1-5)
- Authentication system with role-based permissions
- Navigation system with role-based menu items
- State management infrastructure ready

## EPIC CONTEXT

**Clinic Operational Modes:**
- Single-doctor mode: Solo practitioner with simplified workflows
- Multi-provider mode: Clinic with multiple providers and advanced features
- Mode affects navigation, dashboard, reporting, and administrative features

## TASK CONTEXT

**Focus:** Implementation of clinic mode switching that adapts the entire layout system and feature visibility based on clinic operational configuration.

**Integration:** Enhances all existing layout components with mode-aware behavior.

## BUSINESS DOMAIN CONTEXT

**Single-Doctor Mode Features:**
- Simplified navigation (Dashboard, Patients, Appointments, Clinical)
- Streamlined workflows optimized for solo practice
- Reduced administrative complexity
- Focus on essential clinical functions

**Multi-Provider Mode Features:**
- Comprehensive navigation including provider management
- Advanced reporting and analytics
- Role-based access with granular permissions
- Multi-provider scheduling and coordination tools

**Mode Switching Rules:**
- Clinic configuration determines default mode
- Admin users can switch between modes for testing
- Mode changes affect navigation, dashboard widgets, and feature availability
- User permissions still apply within selected mode

## FUNCTIONAL REQUIREMENTS

1. **Clinic Mode Store Integration**
   - Zustand store for current clinic mode state
   - Mode switching functionality with persistence
   - Integration with clinic configuration API

2. **Mode-Aware Layout Components**
   - Navigation menu filtering based on clinic mode
   - Dashboard widget visibility adaptation
   - Feature availability conditional rendering
   - Simplified UI patterns in single-doctor mode

3. **Mode Switching Interface**
   - Admin interface for mode switching (development/testing)
   - Visual indicators of current mode
   - Smooth transitions between modes
   - User session persistence of mode preference

4. **Feature Visibility Logic**
   - Centralized mode-based feature flags
   - Granular control over component visibility
   - Graceful degradation for unsupported features
   - Consistent mode behavior across application

## TECHNICAL REQUIREMENTS

**State Management:**
- Clinic mode state in Zustand store
- Mode persistence in localStorage/user preferences
- Real-time mode switching without page reload

**Performance:**
- Efficient component re-rendering during mode switches
- Lazy loading of mode-specific components
- Optimized conditional rendering logic

**Configuration:**
- Centralized mode configuration definitions
- Type-safe mode switching with TypeScript
- Environment-specific mode defaults

## TESTING REQUIREMENTS

**Unit Tests:**
- Mode switching logic and state management
- Component visibility based on clinic mode
- Mode-specific navigation menu generation
- Feature flag evaluation logic

**Integration Tests:**
- Mode switching integration across layout components
- Persistence of mode preferences
- API integration for clinic configuration

**Manual Tests:**
- Mode switching user experience testing
- Feature visibility verification in both modes
- Performance testing for mode transitions

## DOCUMENTATIONS REQUIREMENTS

- Clinic mode configuration guide
- Mode-specific feature documentation
- Mode switching implementation patterns

## VALIDATION CRITERIA
- [ ] Clinic mode store manages current mode state correctly
- [ ] Navigation menu adapts based on selected clinic mode
- [ ] Single-doctor mode shows simplified interface with essential features
- [ ] Multi-provider mode displays comprehensive feature set
- [ ] Mode switching works smoothly without page reload
- [ ] Feature visibility correctly filters based on current mode
- [ ] Mode preferences persist across user sessions
- [ ] Admin interface allows mode switching for testing
- [ ] All layout components respond to mode changes appropriately
- [ ] Performance requirements met for mode transitions
- [ ] Mode indicators clearly show current operational mode
- [ ] Integration with existing authentication and role system works correctly