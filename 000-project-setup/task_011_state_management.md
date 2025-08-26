# TASK-011: State Management Setup

## Task Overview

### Goal
**What to Build:** Configure comprehensive state management system using Zustand for global application state and TanStack Query for server state management, optimized for healthcare data workflows and clinical operations.

**Why:** Provides efficient, scalable state management that handles both client-side application state and server-side healthcare data with proper caching, synchronization, and offline capabilities for clinical environments.

**Definition of Done:** Zustand stores configured for global state, TanStack Query setup for API data fetching with healthcare-specific configurations, and proper integration with authentication and error handling systems.

### Scope
```yaml
task_type: "FRONTEND"
complexity: "MEDIUM"
```

## Requirements

### Functional Requirements
1. **Global State Management with Zustand**
   - Description: Configure Zustand stores for authentication, user preferences, application settings, and global UI state
   - Acceptance Criteria: Global state accessible throughout app, persistence configured, and proper state updates
   - Priority: HIGH

2. **Server State Management with TanStack Query**
   - Description: Setup TanStack Query for API data fetching, caching, and synchronization with healthcare-optimized configurations
   - Acceptance Criteria: API data properly cached, background updates working, and offline-first capabilities
   - Priority: HIGH

3. **Healthcare Data Optimizations**
   - Description: Specialized configurations for patient data, appointment scheduling, and clinical workflows
   - Acceptance Criteria: Stale-while-revalidate patterns for critical data, proper invalidation strategies
   - Priority: MEDIUM

4. **Error Handling and Loading States**
   - Description: Consistent error handling and loading state management across all data operations
   - Acceptance Criteria: User-friendly error messages, retry mechanisms, and proper loading indicators
   - Priority: MEDIUM

### Technical Requirements
```yaml
performance:
  - requirement: "State update performance"
    target: "< 16ms for state updates to maintain 60fps UI"
    
security:
  - requirement: "Sensitive data handling in state"
    implementation: "No sensitive patient data persisted in local storage"
    
compatibility:
  - requirement: "Cross-tab synchronization"
    scope: "State synchronization across multiple browser tabs"
```

## Implementation Details

### Technical Approach
```yaml
state_management_architecture:
  zustand_stores:
    - auth_store: "Authentication state and user context"
    - ui_store: "Global UI state, modals, notifications"
    - settings_store: "Application settings and user preferences"
    - clinic_store: "Current clinic context and settings"
    
  tanstack_query_setup:
    - query_client: "Global query client with healthcare configurations"
    - cache_config: "Optimized caching for healthcare data patterns"
    - mutation_handling: "Optimistic updates for clinical workflows"
    - offline_support: "Offline-first approach for critical operations"
    
  data_patterns:
    - patient_queries: "Patient data fetching with proper invalidation"
    - appointment_queries: "Real-time appointment data synchronization"
    - clinical_mutations: "Clinical data updates with conflict resolution"
    - cache_invalidation: "Smart invalidation strategies for related data"

healthcare_optimizations:
  cache_strategies:
    - patient_data: "Long-term caching with background updates"
    - appointment_data: "Short-term caching with real-time invalidation"
    - clinical_data: "Immediate updates with optimistic UI"
    - system_data: "Extended caching for reference data"
    
  offline_capabilities:
    - read_operations: "Cached data available offline"
    - write_operations: "Queue mutations for online sync"
    - conflict_resolution: "Handle data conflicts on reconnection"
```

### Integration Points
```yaml
dependencies:
  - dependency: "Authentication system for API calls"
    type: "FRONTEND"
    status: "IN_PROGRESS"
    
  - dependency: "API client configuration"
    type: "FRONTEND"
    status: "PLANNED"
    
  - dependency: "React application with routing"
    type: "FRONTEND"
    status: "AVAILABLE"
    
provides:
  - deliverable: "Global state management system"
    interface: "Zustand stores and hooks for application state"
    consumers: "All frontend components requiring global state"
    
  - deliverable: "Server state management system"
    interface: "TanStack Query hooks for API data"
    consumers: "All components requiring server data"
    
  - deliverable: "Healthcare-optimized data patterns"
    interface: "Specialized hooks for patient, appointment, and clinical data"
    consumers: "Healthcare workflow components"
```

### Code Patterns to Follow

**Zustand Global Stores:**
```typescript
// src/lib/stores/auth.store.ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import { User, UserPreferences } from '@/types/auth.types';

interface AuthStore {
  user: User | null;
  isAuthenticated: boolean;
  preferences: UserPreferences | null;
  setUser: (user: User | null) => void;
  updatePreferences: (preferences: Partial<UserPreferences>) => void;
  clearAuth: () => void;
}

export const useAuthStore = create<AuthStore>()(
  persist(
    (set, get) => ({
      user: null,
      isAuthenticated: false,
      preferences: null,

      setUser: (user) => set({ 
        user, 
        isAuthenticated: !!user,
        preferences: user?.preferences || null 
      }),

      updatePreferences: (newPreferences) => set((state) => {
        if (!state.user) return state;
        
        const updatedUser = {
          ...state.user,
          preferences: { ...state.user.preferences, ...newPreferences }
        };
        
        return {
          user: updatedUser,
          preferences: updatedUser.preferences
        };
      }),

      clearAuth: () => set({ 
        user: null, 
        isAuthenticated: false, 
        preferences: null 
      }),
    }),
    {
      name: 'medflow-auth',
      storage: createJSONStorage(() => localStorage),
      partialize: (state) => ({
        user: state.user,
        preferences: state.preferences,
        isAuthenticated: state.isAuthenticated,
      }),
    }
  )
);

// src/lib/stores/ui.store.ts
import { create } from 'zustand';

interface Notification {
  id: string;
  type: 'success' | 'error' | 'warning' | 'info';
  title: string;
  message?: string;
  duration?: number;
}

interface UIStore {
  notifications: Notification[];
  sidebarCollapsed: boolean;
  currentModal: string | null;
  loading: Record<string, boolean>;
  
  addNotification: (notification: Omit<Notification, 'id'>) => void;
  removeNotification: (id: string) => void;
  toggleSidebar: () => void;
  setSidebarCollapsed: (collapsed: boolean) => void;
  openModal: (modalId: string) => void;
  closeModal: () => void;
  setLoading: (key: string, isLoading: boolean) => void;
  clearNotifications: () => void;
}

export const useUIStore = create<UIStore>((set, get) => ({
  notifications: [],
  sidebarCollapsed: false,
  currentModal: null,
  loading: {},

  addNotification: (notification) => {
    const id = Date.now().toString();
    const newNotification = { ...notification, id };
    
    set((state) => ({
      notifications: [...state.notifications, newNotification]
    }));

    // Auto-remove notification after duration
    if (notification.duration !== 0) {
      setTimeout(() => {
        get().removeNotification(id);
      }, notification.duration || 5000);
    }
  },

  removeNotification: (id) => set((state) => ({
    notifications: state.notifications.filter(n => n.id !== id)
  })),

  toggleSidebar: () => set((state) => ({
    sidebarCollapsed: !state.sidebarCollapsed
  })),

  setSidebarCollapsed: (collapsed) => set({ sidebarCollapsed: collapsed }),

  openModal: (modalId) => set({ currentModal: modalId }),
  
  closeModal: () => set({ currentModal: null }),

  setLoading: (key, isLoading) => set((state) => ({
    loading: { ...state.loading, [key]: isLoading }
  })),

  clearNotifications: () => set({ notifications: [] }),
}));

// src/lib/stores/clinic.store.ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';

interface ClinicContext {
  clinicId: string;
  clinicName: string;
  mode: 'single_doctor' | 'multi_provider';
  settings: {
    timeZone: string;
    dateFormat: string;
    workingHours: {
      start: string;
      end: string;
    };
    defaultAppointmentDuration: number;
  };
}

interface ClinicStore {
  currentClinic: ClinicContext | null;
  recentPatients: string[];
  quickActions: string[];
  
  setCurrentClinic: (clinic: ClinicContext) => void;
  updateClinicSettings: (settings: Partial<ClinicContext['settings']>) => void;
  addRecentPatient: (patientId: string) => void;
  setQuickActions: (actions: string[]) => void;
  clearClinicContext: () => void;
}

export const useClinicStore = create<ClinicStore>()(
  persist(
    (set, get) => ({
      currentClinic: null,
      recentPatients: [],
      quickActions: [],

      setCurrentClinic: (clinic) => set({ currentClinic: clinic }),

      updateClinicSettings: (newSettings) => set((state) => {
        if (!state.currentClinic) return state;
        
        return {
          currentClinic: {
            ...state.currentClinic,
            settings: { ...state.currentClinic.settings, ...newSettings }
          }
        };
      }),

      addRecentPatient: (patientId) => set((state) => {
        const updated = [patientId, ...state.recentPatients.filter(id => id !== patientId)];
        return { recentPatients: updated.slice(0, 10) }; // Keep last 10
      }),

      setQuickActions: (actions) => set({ quickActions: actions }),

      clearClinicContext: () => set({
        currentClinic: null,
        recentPatients: [],
        quickActions: []
      }),
    }),
    {
      name: 'medflow-clinic',
      storage: createJSONStorage(() => localStorage),
    }
  )
);
```

**TanStack Query Setup:**
```typescript
// src/lib/query.ts
import { QueryClient, QueryCache, MutationCache } from '@tanstack/react-query';
import { useUIStore } from './stores/ui.store';
import { toast } from '@/components/ui/use-toast';

// Global error handler
const handleQueryError = (error: unknown) => {
  const { addNotification } = useUIStore.getState();
  
  if (error instanceof Error) {
    addNotification({
      type: 'error',
      title: 'Data Error',
      message: error.message,
    });
  }
};

// Global mutation error handler
const handleMutationError = (error: unknown) => {
  const { addNotification } = useUIStore.getState();
  
  if (error instanceof Error) {
    addNotification({
      type: 'error',
      title: 'Operation Failed',
      message: error.message,
    });
  }
};

// Create query client with healthcare-optimized settings
export const queryClient = new QueryClient({
  queryCache: new QueryCache({
    onError: handleQueryError,
  }),
  mutationCache: new MutationCache({
    onError: handleMutationError,
  }),
  defaultOptions: {
    queries: {
      // Healthcare data patterns
      staleTime: 5 * 60 * 1000, // 5 minutes
      gcTime: 10 * 60 * 1000, // 10 minutes
      retry: (failureCount, error: any) => {
        // Don't retry on authentication errors
        if (error?.status === 401 || error?.status === 403) {
          return false;
        }
        // Retry up to 3 times for other errors
        return failureCount < 3;
      },
      retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
      refetchOnWindowFocus: true,
      refetchOnReconnect: true,
    },
    mutations: {
      retry: 1,
      onError: handleMutationError,
    },
  },
});

// Healthcare-specific query configurations
export const queryConfig = {
  patients: {
    staleTime: 2 * 60 * 1000, // 2 minutes - patient data changes frequently
    gcTime: 5 * 60 * 1000,
  },
  appointments: {
    staleTime: 30 * 1000, // 30 seconds - appointments change in real-time
    gcTime: 2 * 60 * 1000,
    refetchInterval: 60 * 1000, // Poll every minute
  },
  clinicalData: {
    staleTime: 10 * 60 * 1000, // 10 minutes - clinical data is more stable
    gcTime: 30 * 60 * 1000,
  },
  referenceData: {
    staleTime: 60 * 60 * 1000, // 1 hour - reference data rarely changes
    gcTime: 24 * 60 * 60 * 1000, // 24 hours
  },
};
```

**Healthcare Data Hooks:**
```typescript
// src/hooks/usePatients.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { patientApi } from '@/lib/api/patients.api';
import { useClinicStore } from '@/lib/stores/clinic.store';
import { useUIStore } from '@/lib/stores/ui.store';
import { queryConfig } from '@/lib/query';
import type { Patient, CreatePatientRequest, UpdatePatientRequest } from '@/types/patient.types';

export const usePatients = (searchParams?: any) => {
  const { currentClinic } = useClinicStore();
  
  return useQuery({
    queryKey: ['patients', currentClinic?.clinicId, searchParams],
    queryFn: () => patientApi.list(searchParams),
    enabled: !!currentClinic?.clinicId,
    ...queryConfig.patients,
  });
};

export const usePatient = (patientId: string) => {
  const { addRecentPatient } = useClinicStore();
  
  return useQuery({
    queryKey: ['patients', patientId],
    queryFn: () => patientApi.getById(patientId),
    enabled: !!patientId,
    onSuccess: () => {
      // Add to recent patients when accessed
      addRecentPatient(patientId);
    },
    ...queryConfig.patients,
  });
};

export const useCreatePatient = () => {
  const queryClient = useQueryClient();
  const { addNotification } = useUIStore();
  
  return useMutation({
    mutationFn: (data: CreatePatientRequest) => patientApi.create(data),
    onSuccess: (newPatient: Patient) => {
      // Optimistically update cache
      queryClient.setQueryData(['patients', newPatient.id], newPatient);
      
      // Invalidate patient lists
      queryClient.invalidateQueries({ queryKey: ['patients'] });
      
      addNotification({
        type: 'success',
        title: 'Patient Created',
        message: `${newPatient.firstName} ${newPatient.lastName} has been added to the system.`,
      });
    },
    onError: (error: any) => {
      addNotification({
        type: 'error',
        title: 'Failed to Create Patient',
        message: error.message || 'An error occurred while creating the patient.',
      });
    },
  });
};

export const useUpdatePatient = (patientId: string) => {
  const queryClient = useQueryClient();
  const { addNotification } = useUIStore();
  
  return useMutation({
    mutationFn: (data: UpdatePatientRequest) => patientApi.update(patientId, data),
    onMutate: async (newData) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: ['patients', patientId] });
      
      // Snapshot previous value
      const previousPatient = queryClient.getQueryData(['patients', patientId]);
      
      // Optimistically update
      queryClient.setQueryData(['patients', patientId], (old: Patient) => ({
        ...old,
        ...newData,
        updatedAt: new Date().toISOString(),
      }));
      
      return { previousPatient };
    },
    onError: (error, newData, context) => {
      // Rollback on error
      if (context?.previousPatient) {
        queryClient.setQueryData(['patients', patientId], context.previousPatient);
      }
      
      addNotification({
        type: 'error',
        title: 'Failed to Update Patient',
        message: error.message || 'An error occurred while updating the patient.',
      });
    },
    onSettled: () => {
      // Refetch to ensure consistency
      queryClient.invalidateQueries({ queryKey: ['patients', patientId] });
      queryClient.invalidateQueries({ queryKey: ['patients'] });
    },
    onSuccess: (updatedPatient: Patient) => {
      addNotification({
        type: 'success',
        title: 'Patient Updated',
        message: `${updatedPatient.firstName} ${updatedPatient.lastName}'s information has been updated.`,
      });
    },
  });
};

// src/hooks/useAppointments.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { appointmentApi } from '@/lib/api/appointments.api';
import { useClinicStore } from '@/lib/stores/clinic.store';
import { useUIStore } from '@/lib/stores/ui.store';
import { queryConfig } from '@/lib/query';

export const useAppointments = (filters?: any) => {
  const { currentClinic } = useClinicStore();
  
  return useQuery({
    queryKey: ['appointments', currentClinic?.clinicId, filters],
    queryFn: () => appointmentApi.list(filters),
    enabled: !!currentClinic?.clinicId,
    ...queryConfig.appointments,
  });
};

export const useDailySchedule = (date: string) => {
  const { currentClinic } = useClinicStore();
  
  return useQuery({
    queryKey: ['appointments', 'daily', currentClinic?.clinicId, date],
    queryFn: () => appointmentApi.getDailySchedule(date),
    enabled: !!currentClinic?.clinicId && !!date,
    ...queryConfig.appointments,
  });
};

export const useAppointmentMutations = () => {
  const queryClient = useQueryClient();
  const { addNotification } = useUIStore();
  
  const checkIn = useMutation({
    mutationFn: ({ appointmentId, data }: { appointmentId: string; data: any }) =>
      appointmentApi.checkIn(appointmentId, data),
    onSuccess: (appointment) => {
      // Update appointment cache
      queryClient.setQueryData(['appointments', appointment.id], appointment);
      
      // Invalidate related queries
      queryClient.invalidateQueries({ queryKey: ['appointments'] });
      queryClient.invalidateQueries({ queryKey: ['appointments', 'daily'] });
      
      addNotification({
        type: 'success',
        title: 'Patient Checked In',
        message: 'Patient has been successfully checked in.',
      });
    },
  });
  
  const updateStatus = useMutation({
    mutationFn: ({ appointmentId, status }: { appointmentId: string; status: string }) =>
      appointmentApi.updateStatus(appointmentId, status),
    onSuccess: (appointment) => {
      queryClient.setQueryData(['appointments', appointment.id], appointment);
      queryClient.invalidateQueries({ queryKey: ['appointments'] });
      
      addNotification({
        type: 'info',
        title: 'Status Updated',
        message: `Appointment status changed to ${status.replace('_', ' ').toUpperCase()}.`,
      });
    },
  });
  
  return { checkIn, updateStatus };
};
```

**Query Provider Setup:**
```typescript
// src/providers/QueryProvider.tsx
import React from 'react';
import { QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';
import { queryClient } from '@/lib/query';

interface QueryProviderProps {
  children: React.ReactNode;
}

export const QueryProvider: React.FC<QueryProviderProps> = ({ children }) => {
  return (
    <QueryClientProvider client={queryClient}>
      {children}
      {import.meta.env.DEV && (
        <ReactQueryDevtools 
          initialIsOpen={false}
          position="bottom-right"
        />
      )}
    </QueryClientProvider>
  );
};
```

**Store Provider Integration:**
```typescript
// src/providers/AppProvider.tsx
import React from 'react';
import { QueryProvider } from './QueryProvider';
import { AuthProvider } from './AuthProvider';
import { ThemeProvider } from './ThemeProvider';
import { I18nextProvider } from 'react-i18next';
import { i18n } from '@/lib/i18n';

interface AppProviderProps {
  children: React.ReactNode;
}

export const AppProvider: React.FC<AppProviderProps> = ({ children }) => {
  return (
    <I18nextProvider i18n={i18n}>
      <QueryProvider>
        <AuthProvider>
          <ThemeProvider>
            {children}
          </ThemeProvider>
        </AuthProvider>
      </QueryProvider>
    </I18nextProvider>
  );
};
```

## Testing Requirements

### Test Coverage
```yaml
unit_tests:
  - focus: "Zustand stores and custom hooks"
  - coverage_target: "90%"
  - key_scenarios: ["state_updates", "store_persistence", "query_configurations"]
  
integration_tests:
  - focus: "State management integration with components"
  - test_environment: "React Testing Library with mock providers"
  - key_flows: ["data_fetching", "optimistic_updates", "error_handling"]
  
performance_tests:
  - focus: "State update performance and memory usage"
  - test_environment: "Performance testing tools"
  - key_scenarios: ["large_dataset_handling", "concurrent_updates", "cache_efficiency"]
```

### Test Scenarios
```yaml
happy_path:
  - scenario: "Successful data fetching and caching"
    steps: ["Fetch patient data", "Data cached properly", "Background refetch working"]
    expected_result: "Data available immediately on subsequent requests"
    
error_cases:
  - scenario: "Network error during data fetch"
    trigger: "API endpoint unavailable"
    expected_behavior: "Cached data served, error notification shown, retry mechanism active"
    
edge_cases:
  - scenario: "Concurrent mutations on same data"
    conditions: "Multiple users updating same patient record"
    expected_behavior: "Optimistic updates with conflict resolution"
```

## Context References

### From Project
```yaml
tech_stack_references:
  - "Zustand for lightweight global state management"
  - "TanStack Query for server state management"
  - "Healthcare-optimized caching strategies"
  - "Offline-first approach for clinical environments"
  
architecture_patterns:
  - "Store pattern with Zustand for global state"
  - "Query pattern with TanStack Query for server data"
  - "Optimistic updates for better user experience"
  - "Cache invalidation strategies for data consistency"
  
code_standards:
  - "Consistent hook naming patterns"
  - "Proper error handling and user feedback"
  - "Performance-optimized state updates"
  - "TypeScript integration for type safety"
```

### From Epic
```yaml
business_rules:
  - "Patient data requires frequent updates and synchronization"
  - "Appointment data needs real-time synchronization"
  - "Clinical data should be immediately available offline"
  - "Cross-clinic data isolation maintained in state management"
  
integration_points:
  - "Required by all data-driven components"
  - "Integrates with authentication system for user context"
  - "Enables optimistic UI updates for clinical workflows"
```

## Acceptance Criteria

### Functional Acceptance
- [ ] Zustand stores configured for global application state
- [ ] TanStack Query setup with healthcare-optimized configurations
- [ ] Patient and appointment data hooks working with proper caching
- [ ] Optimistic updates implemented for critical operations
- [ ] Error handling and loading states consistent across all operations

### Technical Acceptance  
- [ ] State updates meet performance requirements (< 16ms)
- [ ] Offline capabilities working for critical data
- [ ] Cross-tab state synchronization functional
- [ ] Memory usage optimized for large datasets
- [ ] Cache invalidation strategies prevent stale data

### Quality Acceptance
- [ ] All stores and hooks thoroughly tested
- [ ] Error scenarios handled gracefully
- [ ] Performance testing shows acceptable state management overhead
- [ ] Integration testing confirms proper data flow
- [ ] Documentation includes state management patterns and best practices

---

**Last Updated:** 2025-01-24