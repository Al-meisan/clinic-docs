# TASK-009: Frontend Routing System

## Task Overview

### Goal
**What to Build:** Implement comprehensive React Router system with protected routes, role-based navigation, healthcare workflow routing patterns, and deep linking support for the MedFlow healthcare management application.

**Why:** Provides structured navigation that supports healthcare workflows, enforces proper access control based on user roles and permissions, and enables bookmarkable URLs for efficient clinical operations.

**Definition of Done:** Complete routing system operational with protected routes, role-based navigation guards, healthcare-specific URL patterns, and proper breadcrumb navigation throughout the application.

### Scope
```yaml
task_type: "FRONTEND"
complexity: "MEDIUM"
```

## Requirements

### Functional Requirements
1. **React Router Integration**
   - Description: Complete React Router v6 setup with nested routes, lazy loading, and error boundaries
   - Acceptance Criteria: All routes functional, lazy loading working, and proper error handling for missing routes
   - Priority: HIGH

2. **Protected Route System**
   - Description: Route guards that enforce authentication and authorization based on OAuth2 scopes
   - Acceptance Criteria: Unauthenticated users redirected to login, insufficient permissions show appropriate error
   - Priority: HIGH

3. **Healthcare Workflow Routes**
   - Description: URL patterns following healthcare workflow conventions with proper parameter handling
   - Acceptance Criteria: Routes support patient IDs, appointment IDs, and other healthcare-specific parameters
   - Priority: HIGH

4. **Navigation Structure**
   - Description: Hierarchical navigation with breadcrumbs, contextual navigation, and workflow-aware routing
   - Acceptance Criteria: Users can navigate efficiently through clinical workflows with clear context
   - Priority: MEDIUM

### Technical Requirements
```yaml
performance:
  - requirement: "Route transition performance"
    target: "< 200ms for route transitions with lazy loading"
    
security:
  - requirement: "Route-based access control"
    implementation: "Scope-based route protection with proper redirection"
    
compatibility:
  - requirement: "Deep linking and bookmarking support"
    scope: "All routes accessible via direct URL with proper authentication flow"
```

## Implementation Details

### Technical Approach
```yaml
routing_architecture:
  router_configuration:
    - react_router_dom: "v6+ for modern routing patterns"
    - lazy_loading: "Route-based code splitting for performance"
    - error_boundaries: "Route-specific error handling"
    - navigation_guards: "Authentication and authorization checks"
    
  route_structure:
    - public_routes: "Login, registration, public pages"
    - protected_routes: "All authenticated user routes"
    - role_based_routes: "Routes restricted by user scopes"
    - clinic_routes: "Multi-tenant clinic-specific routes"
    
  navigation_patterns:
    - hierarchical_breadcrumbs: "Workflow-aware navigation context"
    - contextual_navigation: "Patient/appointment context preservation"
    - deep_linking: "Direct access to specific workflow states"

healthcare_url_patterns:
  patient_routes:
    - /patients (patient list and search)
    - /patients/new (new patient registration)
    - /patients/:patientId (patient profile)
    - /patients/:patientId/edit (patient editing)
    - /patients/:patientId/medical-history
    - /patients/:patientId/appointments
    
  appointment_routes:
    - /appointments (appointments dashboard)
    - /appointments/new (schedule appointment)
    - /appointments/daily (daily schedule view)
    - /appointments/status (status tracking)
    - /appointments/:appointmentId (appointment details)
    - /appointments/:appointmentId/edit
    - /appointments/:appointmentId/check-in
    
  clinical_routes:
    - /clinical/documentation/:patientId/:appointmentId
    - /prescriptions (prescription management)
    - /prescriptions/new
    - /prescriptions/renewals
```

### Integration Points
```yaml
dependencies:
  - dependency: "React application foundation"
    type: "FRONTEND"
    status: "AVAILABLE"
    
  - dependency: "Authentication system integration"
    type: "FRONTEND"
    status: "PLANNED"
    
  - dependency: "UI framework components"
    type: "FRONTEND"
    status: "AVAILABLE"
    
provides:
  - deliverable: "Routing framework"
    interface: "React Router configuration and route components"
    consumers: "All frontend feature modules and navigation components"
    
  - deliverable: "Protected route system"
    interface: "Route guards and authentication flow"
    consumers: "Authentication system and all protected features"
    
  - deliverable: "Navigation utilities"
    interface: "Breadcrumb generation and context management"
    consumers: "Layout components and navigation elements"
```

### Code Patterns to Follow

**Main Router Configuration:**
```typescript
// src/routes/AppRoutes.tsx
import React, { Suspense } from 'react';
import { BrowserRouter, Routes, Route, Navigate } from 'react-router-dom';
import { ErrorBoundary } from '@/components/common/ErrorBoundary';
import { LoadingSpinner } from '@/components/common/LoadingSpinner';
import { ProtectedRoute } from './ProtectedRoute';
import { PublicRoute } from './PublicRoute';
import { DashboardLayout } from '@/components/layouts/DashboardLayout';
import { AuthLayout } from '@/components/layouts/AuthLayout';

// Lazy load page components for better performance
const LoginPage = React.lazy(() => import('@/features/auth/pages/LoginPage'));
const DashboardPage = React.lazy(() => import('@/features/dashboard/pages/DashboardPage'));
const PatientsPage = React.lazy(() => import('@/features/patients/pages/PatientListPage'));
const PatientDetailPage = React.lazy(() => import('@/features/patients/pages/PatientDetailPage'));
const AppointmentsPage = React.lazy(() => import('@/features/appointments/pages/AppointmentListPage'));
const SchedulePage = React.lazy(() => import('@/features/appointments/pages/SchedulePage'));

export const AppRoutes: React.FC = () => {
  return (
    <BrowserRouter>
      <ErrorBoundary>
        <Suspense fallback={<LoadingSpinner />}>
          <Routes>
            {/* Public Routes */}
            <Route
              path="/login"
              element={
                <PublicRoute>
                  <AuthLayout>
                    <LoginPage />
                  </AuthLayout>
                </PublicRoute>
              }
            />
            
            {/* Protected Routes */}
            <Route
              path="/"
              element={
                <ProtectedRoute>
                  <DashboardLayout />
                </ProtectedRoute>
              }
            >
              {/* Dashboard */}
              <Route index element={<Navigate to="/dashboard" replace />} />
              <Route path="dashboard" element={<DashboardPage />} />
              
              {/* Patient Management Routes */}
              <Route
                path="patients"
                element={
                  <ProtectedRoute requiredScopes={['patient:read']}>
                    <PatientsPage />
                  </ProtectedRoute>
                }
              />
              <Route
                path="patients/new"
                element={
                  <ProtectedRoute requiredScopes={['patient:write']}>
                    <PatientCreatePage />
                  </ProtectedRoute>
                }
              />
              <Route
                path="patients/:patientId"
                element={
                  <ProtectedRoute requiredScopes={['patient:read']}>
                    <PatientDetailPage />
                  </ProtectedRoute>
                }
              />
              <Route
                path="patients/:patientId/edit"
                element={
                  <ProtectedRoute requiredScopes={['patient:write']}>
                    <PatientEditPage />
                  </ProtectedRoute>
                }
              />
              
              {/* Appointment Management Routes */}
              <Route
                path="appointments"
                element={
                  <ProtectedRoute requiredScopes={['appointment:read']}>
                    <AppointmentsPage />
                  </ProtectedRoute>
                }
              />
              <Route
                path="appointments/new"
                element={
                  <ProtectedRoute requiredScopes={['appointment:write']}>
                    <SchedulePage />
                  </ProtectedRoute>
                }
              />
            </Route>
            
            {/* Catch-all route */}
            <Route path="*" element={<NotFoundPage />} />
          </Routes>
        </Suspense>
      </ErrorBoundary>
    </BrowserRouter>
  );
};
```

**Protected Route Component:**
```typescript
// src/routes/ProtectedRoute.tsx
import React from 'react';
import { Navigate, useLocation } from 'react-router-dom';
import { useAuth } from '@/hooks/useAuth';
import { LoadingSpinner } from '@/components/common/LoadingSpinner';

interface ProtectedRouteProps {
  children: React.ReactNode;
  requiredScopes?: string[];
  fallbackPath?: string;
}

export const ProtectedRoute: React.FC<ProtectedRouteProps> = ({
  children,
  requiredScopes = [],
  fallbackPath = '/login',
}) => {
  const { user, isAuthenticated, isLoading } = useAuth();
  const location = useLocation();

  // Show loading spinner while authentication status is being determined
  if (isLoading) {
    return <LoadingSpinner />;
  }

  // Redirect to login if not authenticated
  if (!isAuthenticated) {
    return (
      <Navigate 
        to={fallbackPath} 
        state={{ from: location.pathname }} 
        replace 
      />
    );
  }

  // Check required scopes if specified
  if (requiredScopes.length > 0 && user) {
    const hasRequiredScopes = requiredScopes.every(scope => 
      user.scopes.includes(scope) || user.scopes.includes('admin:full')
    );

    if (!hasRequiredScopes) {
      return (
        <Navigate 
          to="/unauthorized" 
          state={{ requiredScopes, currentPath: location.pathname }} 
          replace 
        />
      );
    }
  }

  return <>{children}</>;
};
```

**Public Route Component:**
```typescript
// src/routes/PublicRoute.tsx
import React from 'react';
import { Navigate } from 'react-router-dom';
import { useAuth } from '@/hooks/useAuth';
import { LoadingSpinner } from '@/components/common/LoadingSpinner';

interface PublicRouteProps {
  children: React.ReactNode;
  redirectTo?: string;
}

export const PublicRoute: React.FC<PublicRouteProps> = ({
  children,
  redirectTo = '/dashboard',
}) => {
  const { isAuthenticated, isLoading } = useAuth();

  if (isLoading) {
    return <LoadingSpinner />;
  }

  // Redirect authenticated users away from public routes like login
  if (isAuthenticated) {
    return <Navigate to={redirectTo} replace />;
  }

  return <>{children}</>;
};
```

**Navigation Utilities:**
```typescript
// src/lib/navigation.ts
import { useLocation, useParams } from 'react-router-dom';

export interface BreadcrumbItem {
  label: string;
  path: string;
  isActive: boolean;
}

export const useBreadcrumbs = (): BreadcrumbItem[] => {
  const location = useLocation();
  const params = useParams();
  
  const generateBreadcrumbs = (pathname: string): BreadcrumbItem[] => {
    const segments = pathname.split('/').filter(Boolean);
    const breadcrumbs: BreadcrumbItem[] = [
      { label: 'Home', path: '/dashboard', isActive: false }
    ];

    let currentPath = '';
    
    segments.forEach((segment, index) => {
      currentPath += `/${segment}`;
      const isLast = index === segments.length - 1;
      
      // Handle parameterized routes
      let label = segment;
      if (segment.startsWith(':')) {
        const paramKey = segment.slice(1);
        label = params[paramKey] || segment;
      }
      
      // Custom labels for specific routes
      const customLabels: Record<string, string> = {
        'patients': 'Patients',
        'appointments': 'Appointments',
        'new': 'New',
        'edit': 'Edit',
        'medical-history': 'Medical History',
        'daily': 'Daily Schedule',
        'status': 'Status Tracking',
      };
      
      if (customLabels[segment]) {
        label = customLabels[segment];
      }
      
      breadcrumbs.push({
        label: formatLabel(label),
        path: currentPath,
        isActive: isLast,
      });
    });

    return breadcrumbs;
  };

  return generateBreadcrumbs(location.pathname);
};

export const formatLabel = (text: string): string => {
  return text
    .split('-')
    .map(word => word.charAt(0).toUpperCase() + word.slice(1))
    .join(' ');
};

// Navigation context hook for maintaining workflow state
export const useNavigationContext = () => {
  const location = useLocation();
  const params = useParams();

  const getPatientId = (): string | null => {
    return params.patientId || null;
  };

  const getAppointmentId = (): string | null => {
    return params.appointmentId || null;
  };

  const isInPatientWorkflow = (): boolean => {
    return location.pathname.includes('/patients/');
  };

  const isInAppointmentWorkflow = (): boolean => {
    return location.pathname.includes('/appointments/');
  };

  const getWorkflowContext = () => {
    return {
      patientId: getPatientId(),
      appointmentId: getAppointmentId(),
      isPatientWorkflow: isInPatientWorkflow(),
      isAppointmentWorkflow: isInAppointmentWorkflow(),
      currentPath: location.pathname,
    };
  };

  return {
    getPatientId,
    getAppointmentId,
    isInPatientWorkflow,
    isInAppointmentWorkflow,
    getWorkflowContext,
  };
};
```

**Route Configuration Types:**
```typescript
// src/types/navigation.types.ts
export interface RouteConfig {
  path: string;
  component: React.ComponentType<any>;
  requiresAuth: boolean;
  requiredScopes?: string[];
  layout?: 'dashboard' | 'auth' | 'minimal';
  title?: string;
  breadcrumbLabel?: string;
}

export interface NavigationItem {
  id: string;
  label: string;
  path: string;
  icon?: React.ComponentType;
  requiredScopes?: string[];
  children?: NavigationItem[];
}

export interface WorkflowContext {
  patientId?: string;
  appointmentId?: string;
  clinicId?: string;
  providerId?: string;
  workflowType: 'patient' | 'appointment' | 'clinical' | 'admin';
  currentStep?: string;
}
```

**Navigation Hook:**
```typescript
// src/hooks/useNavigation.ts
import { useNavigate, useLocation } from 'react-router-dom';
import { useAuth } from './useAuth';

export const useNavigation = () => {
  const navigate = useNavigate();
  const location = useLocation();
  const { user } = useAuth();

  const navigateToPatient = (patientId: string, tab?: string) => {
    const basePath = `/patients/${patientId}`;
    const path = tab ? `${basePath}/${tab}` : basePath;
    navigate(path);
  };

  const navigateToAppointment = (appointmentId: string, action?: string) => {
    const basePath = `/appointments/${appointmentId}`;
    const path = action ? `${basePath}/${action}` : basePath;
    navigate(path);
  };

  const navigateWithWorkflowContext = (path: string, context?: Record<string, string>) => {
    const searchParams = new URLSearchParams();
    if (context) {
      Object.entries(context).forEach(([key, value]) => {
        searchParams.set(key, value);
      });
    }
    
    const fullPath = searchParams.toString() ? `${path}?${searchParams}` : path;
    navigate(fullPath);
  };

  const goBack = (fallbackPath = '/dashboard') => {
    if (window.history.length > 1) {
      navigate(-1);
    } else {
      navigate(fallbackPath);
    }
  };

  const canNavigateTo = (path: string, requiredScopes: string[] = []): boolean => {
    if (!user) return false;
    
    if (requiredScopes.length === 0) return true;
    
    return requiredScopes.every(scope => 
      user.scopes.includes(scope) || user.scopes.includes('admin:full')
    );
  };

  return {
    navigateToPatient,
    navigateToAppointment,
    navigateWithWorkflowContext,
    goBack,
    canNavigateTo,
    currentPath: location.pathname,
  };
};
```

## Testing Requirements

### Test Coverage
```yaml
unit_tests:
  - focus: "Route components and navigation utilities"
  - coverage_target: "90%"
  - key_scenarios: ["protected_route_logic", "breadcrumb_generation", "navigation_context"]
  
integration_tests:
  - focus: "Route transitions and authentication flow"
  - test_environment: "React Testing Library with Router"
  - key_flows: ["authenticated_navigation", "route_protection", "deep_linking"]
  
e2e_tests:
  - focus: "Complete navigation workflows"
  - user_scenarios: ["login_and_navigate", "unauthorized_access_attempt", "workflow_navigation"]
```

### Test Scenarios
```yaml
happy_path:
  - scenario: "Authenticated user navigating through patient workflow"
    steps: ["Login", "Navigate to patients", "Select patient", "Navigate to patient details"]
    expected_result: "Smooth navigation with proper breadcrumbs and context"
    
error_cases:
  - scenario: "Unauthenticated user accessing protected route"
    trigger: "Direct URL access without authentication"
    expected_behavior: "Redirect to login with return URL preserved"
    
  - scenario: "Insufficient permissions for route"
    trigger: "User accessing route requiring higher-level scopes"
    expected_behavior: "Redirect to unauthorized page with clear message"
    
edge_cases:
  - scenario: "Deep linking to specific workflow state"
    conditions: "User bookmarks and accesses specific patient or appointment URL"
    expected_behavior: "Proper authentication flow then navigation to intended page"
```

## Context References

### From Project
```yaml
tech_stack_references:
  - "React Router v6 for modern routing patterns"
  - "Lazy loading for performance optimization"
  - "OAuth2 scope-based route protection"
  - "Healthcare workflow URL patterns"
  
architecture_patterns:
  - "Protected route pattern with authentication guards"
  - "Hierarchical routing with nested layouts"
  - "Context-aware navigation with workflow preservation"
  - "Route-based code splitting for performance"
  
code_standards:
  - "RESTful URL patterns for healthcare resources"
  - "Consistent route parameter naming"
  - "Breadcrumb navigation for complex workflows"
  - "Deep linking support for all routes"
```

### From Epic
```yaml
business_rules:
  - "Route access controlled by OAuth2 scopes"
  - "Cross-clinic navigation restricted by user clinic association"
  - "Healthcare workflow context preserved during navigation"
  - "Patient and appointment URLs support direct access"
  
integration_points:
  - "Required by authentication system for login flow"
  - "Foundation for all feature navigation"
  - "Enables layout system with contextual navigation"
```

## Acceptance Criteria

### Functional Acceptance
- [ ] React Router configured with all required routes
- [ ] Protected routes enforce authentication and authorization
- [ ] Healthcare URL patterns implemented with proper parameter handling
- [ ] Breadcrumb navigation working throughout application
- [ ] Deep linking functional for all routes

### Technical Acceptance  
- [ ] Route-based code splitting working with lazy loading
- [ ] Route transitions meet performance requirements
- [ ] Error boundaries handle routing errors gracefully
- [ ] Navigation context preserved during workflow transitions
- [ ] Authentication flow properly integrated with routing

### Quality Acceptance
- [ ] All routes tested with authentication scenarios
- [ ] Navigation utilities provide consistent behavior
- [ ] Route protection tested for all permission levels
- [ ] Deep linking tested with various user scenarios
- [ ] Performance testing shows acceptable route transition times

---

**Last Updated:** 2025-01-24