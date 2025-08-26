# TASK-012: Frontend Layout System

## Task Overview

### Goal
**What to Build:** Create comprehensive responsive layout system supporting both single-doctor and multi-provider clinic modes, with healthcare-optimized navigation, contextual breadcrumbs, and workflow-aware interface adaptations.

**Why:** Provides consistent, professional, and efficient user interface layouts that adapt to different clinic configurations and support healthcare workflows with proper navigation, accessibility, and responsive design.

**Definition of Done:** Complete layout system operational with single-doctor and multi-provider modes, responsive navigation, contextual breadcrumbs, and proper integration with authentication and routing systems.

### Scope
```yaml
task_type: "FRONTEND"
complexity: "HIGH"
```

## Requirements

### Functional Requirements
1. **Multi-Mode Layout System**
   - Description: Adaptive layouts supporting simplified single-doctor mode and comprehensive multi-provider mode
   - Acceptance Criteria: Layout automatically adapts based on clinic mode, appropriate feature visibility, simplified navigation for solo practitioners
   - Priority: HIGH

2. **Responsive Navigation System**
   - Description: Professional healthcare navigation with sidebar, top bar, and contextual navigation elements
   - Acceptance Criteria: Navigation works on desktop and tablet, collapsible sidebar, role-based menu items
   - Priority: HIGH

3. **Contextual Breadcrumb System**
   - Description: Workflow-aware breadcrumb navigation showing patient/appointment context
   - Acceptance Criteria: Breadcrumbs reflect current workflow state, clickable navigation, proper context display
   - Priority: MEDIUM

4. **Healthcare-Optimized Components**
   - Description: Layout components optimized for clinical workflows and medical data presentation
   - Acceptance Criteria: Professional medical interface styling, efficient space utilization, accessibility compliance
   - Priority: MEDIUM

### Technical Requirements
```yaml
performance:
  - requirement: "Layout rendering performance"
    target: "< 100ms for layout transitions and responsive breakpoint changes"
    
accessibility:
  - requirement: "WCAG 2.1 AA compliance"
    implementation: "All layout elements accessible via keyboard and screen reader"
    
compatibility:
  - requirement: "Cross-device responsive design"
    scope: "Optimal experience on desktop (1920x1080), tablet (1024x768), and mobile (390x844)"
```

## Implementation Details

### Technical Approach
```yaml
layout_architecture:
  layout_components:
    - DashboardLayout: "Main authenticated user layout"
    - AuthLayout: "Authentication pages layout"
    - SingleDoctorLayout: "Simplified layout for solo practitioners"
    - MultiProviderLayout: "Full-featured layout for clinics"
    - ModalLayout: "Modal and overlay layout system"
    
  navigation_system:
    - MainNavigation: "Primary navigation sidebar use shadcn/ui sidebare (https://ui.shadcn.com/docs/components/sidebar, https://ui.shadcn.com/blocks/sidebar#sidebar-02)"
    - TopBar: "User profile, notifications, and quick actions"
    - Breadcrumbs: "Contextual navigation breadcrumbs"
    - QuickActions: "Workflow-specific quick action buttons"
    
  responsive_design:
    - desktop_first: "Optimized for clinical workstations"
    - tablet_adaptation: "Touch-friendly for mobile clinical use"
    - mobile_fallback: "Basic functionality for emergency mobile access"

clinic_mode_adaptations:
  single_doctor_mode:
    - simplified_navigation: "Essential features only"
    - streamlined_workflows: "Optimized for solo practice patterns"
    - reduced_complexity: "Minimal configuration options"
    - focused_dashboard: "Single provider dashboard view"
    
  multi_provider_mode:
    - comprehensive_navigation: "Full feature access"
    - provider_management: "Multi-provider scheduling and management"
    - advanced_reporting: "Clinic-wide analytics and reports"
    - role_based_access: "Granular permission-based interface"
```

### Integration Points
```yaml
dependencies:
  - dependency: "Authentication system for user context"
    type: "FRONTEND"
    status: "IN_PROGRESS"
    
  - dependency: "Routing system for navigation integration"
    type: "FRONTEND"
    status: "AVAILABLE"
    
  - dependency: "State management for layout preferences"
    type: "FRONTEND"
    status: "IN_PROGRESS"
    
provides:
  - deliverable: "Layout component system"
    interface: "Reusable layout components for all application pages"
    consumers: "All page components and route definitions"
    
  - deliverable: "Navigation framework"
    interface: "Responsive navigation with role-based visibility"
    consumers: "All authenticated application areas"
    
  - deliverable: "Contextual UI patterns"
    interface: "Workflow-aware interface adaptations"
    consumers: "Patient, appointment, and clinical workflow components"
```

### Code Patterns to Follow

**Main Dashboard Layout:**
```typescript
// src/components/layouts/DashboardLayout.tsx
import React, { useState, useEffect } from 'react';
import { Outlet, useLocation } from 'react-router-dom';
import { useAuth } from '@/hooks/useAuth';
import { useClinicStore } from '@/lib/stores/clinic.store';
import { useUIStore } from '@/lib/stores/ui.store';
import { MainNavigation } from './components/MainNavigation';
import { TopBar } from './components/TopBar';
import { AppBreadcrumbs } from './components/AppBreadcrumbs';
import { NotificationCenter } from '@/components/common/NotificationCenter';
import { cn } from '@/lib/utils';

export const DashboardLayout: React.FC = () => {
  const { user } = useAuth();
  const { currentClinic } = useClinicStore();
  const { sidebarCollapsed, setSidebarCollapsed } = useUIStore();
  // replace this with the useSidebar hook from "@/components/ui/sidebar"
  const [isMobile, setIsMobile] = useState(false);
  const location = useLocation();

  // Handle responsive design
  useEffect(() => {
    const handleResize = () => {
      const mobile = window.innerWidth < 768;
      setIsMobile(mobile);
      
      // Auto-collapse sidebar on mobile
      if (mobile && !sidebarCollapsed) {
        setSidebarCollapsed(true);
      }
    };

    handleResize();
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, [sidebarCollapsed, setSidebarCollapsed]);

  // Determine layout mode based on clinic configuration
  const layoutMode = currentClinic?.mode || 'single_doctor';
  const isSimplified = layoutMode === 'single_doctor';

  return (
    <div className="flex h-screen bg-gray-50">
      {/* Main Navigation Sidebar */}
      <div
        className={cn(
          'bg-white border-r border-gray-200 transition-all duration-300 flex flex-col',
          sidebarCollapsed ? 'w-16' : 'w-64',
          isMobile && 'fixed inset-y-0 left-0 z-50 shadow-lg',
          isMobile && sidebarCollapsed && '-translate-x-full'
        )}
      >
        <MainNavigation 
          collapsed={sidebarCollapsed}
          simplified={isSimplified}
        />
      </div>

      {/* Mobile sidebar overlay */}
      {isMobile && !sidebarCollapsed && (
        <div
          className="fixed inset-0 bg-black bg-opacity-50 z-40"
          onClick={() => setSidebarCollapsed(true)}
        />
      )}

      {/* Main Content Area */}
      <div className="flex-1 flex flex-col overflow-hidden">
        {/* Top Bar */}
        <TopBar 
          onToggleSidebar={() => setSidebarCollapsed(!sidebarCollapsed)}
          simplified={isSimplified}
        />

        {/* Breadcrumb Navigation */}
        <div className="bg-white border-b border-gray-200 px-6 py-3">
          <AppBreadcrumbs simplified={isSimplified} />
        </div>

        {/* Page Content */}
        <main className="flex-1 overflow-y-auto bg-gray-50 p-6">
          <div className="max-w-7xl mx-auto">
            <Outlet />
          </div>
        </main>
      </div>

      {/* Notification Center */}
      <NotificationCenter />
    </div>
  );
};
```

**Main Navigation Component:**
```typescript
// src/components/layouts/components/MainNavigation.tsx
import React from 'react';
import { Link, useLocation } from 'react-router-dom';
import { useAuth } from '@/hooks/useAuth';
import { useTranslation } from 'react-i18next';
import {
  Users,
  Calendar,
  FileText,
  Pill,
  CreditCard,
  Settings,
  Home,
  ChevronDown,
  Activity,
} from 'lucide-react';
import { cn } from '@/lib/utils';
import { Button } from '@/components/ui/button';
import { Collapsible, CollapsibleContent, CollapsibleTrigger } from '@/components/ui/collapsible';

interface NavigationItem {
  id: string;
  label: string;
  path: string;
  icon: React.ComponentType<{ className?: string }>;
  requiredScopes?: string[];
  children?: NavigationItem[];
  simplified?: boolean; // Show in simplified mode
}

const navigationItems: NavigationItem[] = [
  {
    id: 'dashboard',
    label: 'Dashboard',
    path: '/dashboard',
    icon: Home,
    simplified: true,
  },
  {
    id: 'patients',
    label: 'Patients',
    path: '/patients',
    icon: Users,
    requiredScopes: ['patient:read'],
    simplified: true,
    children: [
      {
        id: 'patients-list',
        label: 'Patient List',
        path: '/patients',
        icon: Users,
        requiredScopes: ['patient:read'],
      },
      {
        id: 'patients-new',
        label: 'New Patient',
        path: '/patients/new',
        icon: Users,
        requiredScopes: ['patient:write'],
      },
    ],
  },
  {
    id: 'appointments',
    label: 'Appointments',
    path: '/appointments',
    icon: Calendar,
    requiredScopes: ['appointment:read'],
    simplified: true,
    children: [
      {
        id: 'appointments-schedule',
        label: 'Schedule',
        path: '/appointments/new',
        icon: Calendar,
        requiredScopes: ['appointment:write'],
      },
      {
        id: 'appointments-daily',
        label: 'Daily View',
        path: '/appointments/daily',
        icon: Calendar,
        requiredScopes: ['appointment:read'],
      },
      {
        id: 'appointments-status',
        label: 'Status Tracking',
        path: '/appointments/status',
        icon: Activity,
        requiredScopes: ['appointment:read'],
      },
    ],
  },
  {
    id: 'clinical',
    label: 'Clinical',
    path: '/clinical',
    icon: FileText,
    requiredScopes: ['clinical:read'],
    children: [
      {
        id: 'clinical-documentation',
        label: 'Documentation',
        path: '/clinical/documentation',
        icon: FileText,
        requiredScopes: ['clinical:write'],
      },
      {
        id: 'prescriptions',
        label: 'Prescriptions',
        path: '/prescriptions',
        icon: Pill,
        requiredScopes: ['prescription:read'],
      },
    ],
  },
  {
    id: 'billing',
    label: 'Billing',
    path: '/billing',
    icon: CreditCard,
    requiredScopes: ['billing:read'],
    // Only show in multi-provider mode
    simplified: false,
  },
  {
    id: 'settings',
    label: 'Settings',
    path: '/settings',
    icon: Settings,
    requiredScopes: ['clinic:manage'],
    simplified: false,
  },
];

interface MainNavigationProps {
  collapsed: boolean;
  simplified: boolean;
  isMobile: boolean;
}

export const MainNavigation: React.FC<MainNavigationProps> = ({
  collapsed,
  simplified
}) => {
  //...build this using the shadcn/ui sidebare component(https://ui.shadcn.com/docs/components/sidebar, https://ui.shadcn.com/blocks/sidebar#sidebar-02)
};
```

**Top Bar Component:**
```typescript
// src/components/layouts/components/TopBar.tsx
import React from 'react';
import { Menu, Bell, Search, Plus } from 'lucide-react';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Badge } from '@/components/ui/badge';
import { UserProfile } from '@/components/common/UserProfile';
import { NotificationDropdown } from '@/components/common/NotificationDropdown';
import { QuickActions } from './QuickActions';
import { useAuth } from '@/hooks/useAuth';
import { useUIStore } from '@/lib/stores/ui.store';
import { cn } from '@/lib/utils';

interface TopBarProps {
  onToggleSidebar: () => void;
  simplified: boolean;
}

export const TopBar: React.FC<TopBarProps> = ({ onToggleSidebar, simplified }) => {
  const { user } = useAuth();
  const { notifications } = useUIStore();
  const [searchQuery, setSearchQuery] = React.useState('');

  const unreadCount = notifications.filter(n => !n.readAt).length;

  return (
    <header className="bg-white border-b border-gray-200 px-6 py-4">
      <div className="flex items-center justify-between">
        {/* Left Section */}
        <div className="flex items-center space-x-4">
          {/* Sidebar Toggle */}
          <Button
            variant="ghost"
            size="sm"
            onClick={onToggleSidebar}
            className="p-2"
          >
            <Menu className="h-5 w-5" />
          </Button>

          {/* Search Bar */}
          <div className="relative">
            <Search className="absolute left-3 top-1/2 transform -translate-y-1/2 h-4 w-4 text-gray-400" />
            <Input
              type="search"
              placeholder="Search patients, appointments..."
              value={searchQuery}
              onChange={(e) => setSearchQuery(e.target.value)}
              className="pl-10 w-80"
            />
          </div>
        </div>

        {/* Center Section - Quick Actions */}
        {!simplified && (
          <div className="flex items-center space-x-2">
            <QuickActions />
          </div>
        )}

        {/* Right Section */}
        <div className="flex items-center space-x-4">
          {/* Quick Add Button */}
          <Button size="sm" className="gap-2">
            <Plus className="h-4 w-4" />
            {simplified ? 'New Patient' : 'Quick Add'}
          </Button>

          {/* Notifications */}
          <div className="relative">
            <NotificationDropdown>
              <Button variant="ghost" size="sm" className="relative p-2">
                <Bell className="h-5 w-5" />
                {unreadCount > 0 && (
                  <Badge
                    variant="destructive"
                    className="absolute -top-1 -right-1 h-5 w-5 rounded-full p-0 flex items-center justify-center text-xs"
                  >
                    {unreadCount > 9 ? '9+' : unreadCount}
                  </Badge>
                )}
              </Button>
            </NotificationDropdown>
          </div>

          {/* User Profile */}
          <UserProfile />
        </div>
      </div>
    </header>
  );
};
```

**Breadcrumb Component:**
```typescript
// src/components/layouts/components/AppBreadcrumbs.tsx
import React from 'react';
import { Link, useLocation, useParams } from 'react-router-dom';
import { ChevronRight, Home } from 'lucide-react';
import { useTranslation } from 'react-i18next';
import { usePatient } from '@/hooks/usePatients';
import { useAppointment } from '@/hooks/useAppointments';
import { cn } from '@/lib/utils';

import {
  Breadcrumb,
  BreadcrumbEllipsis,
  BreadcrumbItem,
  BreadcrumbLink,
  BreadcrumbList,
  BreadcrumbPage,
  BreadcrumbSeparator,
} from "@/components/ui/breadcrumb"

interface BreadcrumbItem {
  label: string;
  path?: string;
  isActive: boolean;
}

interface AppBreadcrumbsProps {
  simplified: boolean;
}

export const AppBreadcrumbs: React.FC<AppBreadcrumbsProps> = ({ simplified }) => {
  const location = useLocation();
  const params = useParams();
  const { t } = useTranslation('navigation');

  // Fetch contextual data for breadcrumbs
  const { data: patient } = usePatient(params.patientId || '', {
    enabled: !!params.patientId,
  });
  const { data: appointment } = useAppointment(params.appointmentId || '', {
    enabled: !!params.appointmentId,
  });

  const generateBreadcrumbs = (): BreadcrumbItem[] => {
    const segments = location.pathname.split('/').filter(Boolean);
    const breadcrumbs: BreadcrumbItem[] = [
      { label: t('dashboard'), path: '/dashboard', isActive: false }
    ];

    let currentPath = '';
    
    segments.forEach((segment, index) => {
      currentPath += `/${segment}`;
      const isLast = index === segments.length - 1;
      
      let label = segment;
      let path: string | undefined = currentPath;
      
      // Handle specific route patterns
      switch (segment) {
        case 'patients':
          label = t('patients.title');
          break;
        case 'appointments':
          label = t('appointments.title');
          break;
        case 'clinical':
          label = t('clinical.title');
          break;
        case 'prescriptions':
          label = t('prescriptions.title');
          break;
        case 'new':
          label = t('common.new');
          break;
        case 'edit':
          label = t('common.edit');
          break;
        case 'daily':
          label = t('appointments.daily_view');
          break;
        case 'status':
          label = t('appointments.status_tracking');
          break;
        case 'medical-history':
          label = t('patients.medical_history');
          break;
        default:
          // Handle dynamic segments (IDs)
          if (segment === params.patientId && patient) {
            label = `${patient.firstName} ${patient.lastName}`;
          } else if (segment === params.appointmentId && appointment) {
            label = `Appointment - ${new Date(appointment.dateTime).toLocaleDateString()}`;
          } else {
            // Capitalize and format segment
            label = segment.replace(/-/g, ' ').replace(/\b\w/g, c => c.toUpperCase());
          }
      }
      
      // Don't show link for last item
      if (isLast) {
        path = undefined;
      }
      
      breadcrumbs.push({
        label,
        path,
        isActive: isLast,
      });
    });

    return breadcrumbs;
  };

  const breadcrumbs = generateBreadcrumbs();

  // In simplified mode, show fewer breadcrumb levels
  const displayBreadcrumbs = simplified 
    ? breadcrumbs.slice(-2) // Show only last 2 levels
    : breadcrumbs;

  return (
    <nav className="flex items-center space-x-1 text-sm">
      <Home className="h-4 w-4 text-gray-400" />
      <Breadcrumb>
      <BreadcrumbList>
        <BreadcrumbItem>
          <BreadcrumbLink asChild>
            <Link href="/"><Home className="h-4 w-4 text-gray-400" /></Link>
          </BreadcrumbLink>
        </BreadcrumbItem>
        
        <BreadcrumbSeparator />
          {displayBreadcrumbs.map((item, index) => (
            // use https://ui.shadcn.com/docs/components/breadcrumb (do the neccessary to adapte generateBreadcrumbs())
            <BreadcrumbItem>
              <BreadcrumbLink asChild>
                <Link href="/docs/components">{// link to page}</Link>
              </BreadcrumbLink>
            </BreadcrumbItem>
            <BreadcrumbSeparator />
          ))}
      </BreadcrumbList>
    </Breadcrumb>  
    </nav>
  );
};
```

**Single Doctor Layout:**
```typescript
// src/components/layouts/SingleDoctorLayout.tsx
import React from 'react';
import { Outlet } from 'react-router-dom';
import { useAuth } from '@/hooks/useAuth';
import { SimplifiedNavigation } from './components/SimplifiedNavigation';
import { SimplifiedTopBar } from './components/SimplifiedTopBar';
import { PatientQuickAccess } from './components/PatientQuickAccess';

export const SingleDoctorLayout: React.FC = () => {
  const { user } = useAuth();

  return (
    <div className="flex h-screen bg-gray-50">
      {/* Simplified Sidebar */}
      <div className="w-20 bg-white border-r border-gray-200 flex flex-col">
        <SimplifiedNavigation />
      </div>

      {/* Main Content */}
      <div className="flex-1 flex flex-col">
        {/* Simplified Top Bar */}
        <SimplifiedTopBar />

        {/* Content Area */}
        <div className="flex-1 flex">
          {/* Main Content */}
          <main className="flex-1 overflow-y-auto p-6">
            <Outlet />
          </main>

          {/* Quick Access Panel */}
          <div className="w-80 bg-white border-l border-gray-200 p-4">
            <PatientQuickAccess />
          </div>
        </div>
      </div>
    </div>
  );
};
```

## Testing Requirements

### Test Coverage
```yaml
unit_tests:
  - focus: "Layout components and navigation logic"
  - coverage_target: "85%"
  - key_scenarios: ["responsive_behavior", "navigation_state", "breadcrumb_generation"]
  
integration_tests:
  - focus: "Layout integration with routing and authentication"
  - test_environment: "React Testing Library with Router and Auth providers"
  - key_flows: ["navigation_permissions", "layout_mode_switching", "responsive_transitions"]
  
accessibility_tests:
  - focus: "Layout accessibility and keyboard navigation"
  - test_environment: "Automated accessibility testing tools"
  - key_scenarios: ["keyboard_navigation", "screen_reader_compatibility", "focus_management"]
```

### Test Scenarios
```yaml
happy_path:
  - scenario: "Navigation and layout adaptation based on clinic mode"
    steps: ["Login to single-doctor clinic", "Verify simplified layout", "Switch to multi-provider", "Verify full layout"]
    expected_result: "Layout correctly adapts to clinic mode with appropriate feature visibility"
    
error_cases:
  - scenario: "Navigation with insufficient permissions"
    trigger: "User attempts to access restricted navigation item"
    expected_behavior: "Navigation item hidden or disabled with appropriate feedback"
    
edge_cases:
  - scenario: "Responsive layout on extreme viewport sizes"
    conditions: "Very narrow mobile viewport or ultra-wide desktop"
    expected_behavior: "Layout remains functional with appropriate adaptations"
```

## Context References

### From Project
```yaml
tech_stack_references:
  - "React Router for navigation integration"
  - "Tailwind CSS for responsive design utilities"
  - "shadcn/ui for consistent component styling"
  - "Healthcare-optimized layout patterns"
  
architecture_patterns:
  - "Layout component pattern with composition"
  - "Responsive design with mobile-first approach"
  - "Role-based navigation with conditional rendering"
  - "Context-aware breadcrumb generation"
  
code_standards:
  - "Consistent layout component structure"
  - "Accessible navigation with ARIA attributes"
  - "Performance-optimized responsive transitions"
  - "Clean separation between layout and content"
```

### From Epic
```yaml
business_rules:
  - "Single-doctor mode provides simplified interface with reduced complexity"
  - "Multi-provider mode offers full feature access with role-based navigation"
  - "Layout must be professional and appropriate for medical environments"
  - "Responsive design optimized for clinical workstations and tablets"
  
integration_points:
  - "Required by all authenticated application routes"
  - "Integrates with authentication system for role-based navigation"
  - "Foundation for healthcare workflow user interfaces"
```

## Acceptance Criteria

### Functional Acceptance
- [ ] Layout system supports both single-doctor and multi-provider modes
- [ ] Responsive navigation works across all target device sizes
- [ ] Breadcrumb navigation shows proper workflow context
- [ ] Role-based navigation items display correctly
- [ ] Layout transitions and mode switching functional

### Technical Acceptance  
- [ ] Layout rendering meets performance requirements
- [ ] All layout elements meet WCAG 2.1 AA accessibility standards
- [ ] Responsive breakpoints work properly across devices
- [ ] Navigation state properly integrated with routing system
- [ ] Cross-tab layout synchronization working

### Quality Acceptance
- [ ] All layout components thoroughly tested
- [ ] Accessibility testing passes for navigation and layout
- [ ] Responsive design tested across target devices
- [ ] Performance testing shows acceptable layout render times
- [ ] User experience testing confirms intuitive navigation

---

**Last Updated:** 2025-01-24
