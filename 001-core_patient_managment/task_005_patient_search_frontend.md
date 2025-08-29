# TASK: TASK-005 - Patient Search Frontend Interface

## Task Overview

### Goal

**What to Build:** Advanced patient search interface with real-time search, filtering capabilities, recent patients
quick access, and optimal user experience for healthcare providers to quickly locate patient records.

**Why:** Enable healthcare providers to efficiently find patients during consultations and administrative tasks,
supporting fast-paced clinic operations with quick search results.

**Definition of Done:**

- Real-time patient search with instant results display
- Advanced filtering options with persistent filter state
- Recent patients quick access functionality
- Responsive design optimized for various screen sizes
- Integration with patient search API with proper error handling

### Scope

```yaml
task_type: "FRONTEND"
complexity: "MEDIUM"
```

## Requirements

### Functional Requirements

1. **Real-Time Search Interface**
    - Description: Instant search with debounced input and live results display
    - Acceptance Criteria:
        - Search results update as user types with 300ms debounce
        - Supports search by name, phone, DOB, and patient ID
        - Displays search results quickly
        - Highlights matching text in search results
    - Priority: HIGH

2. **Advanced Filtering System**
    - Description: Multiple filter options for refined patient search
    - Acceptance Criteria:
        - Filter by patient status (active, inactive, etc.)
        - Filter by age ranges and gender
        - Filter by registration date ranges
        - Persistent filter state across sessions
    - Priority: HIGH

3. **Recent Patients Access**
    - Description: Quick access to recently viewed patients
    - Acceptance Criteria:
        - Display last 10 accessed patients
        - One-click access to recent patient profiles
        - Updates automatically when patients are accessed
        - Persists across user sessions
    - Priority: MEDIUM

4. **Search Results Display**
    - Description: Optimized patient search results with relevant information
    - Acceptance Criteria:
        - Patient card layout with photo, name, and key details
        - Pagination for large result sets
        - Sorting options (name, last visit, registration date)
        - Empty state guidance for no results
    - Priority: HIGH

5. **Search Performance Optimization**
    - Description: Optimized search experience with loading states and caching
    - Acceptance Criteria:
        - Loading indicators during search operations
        - Result caching for improved performance
        - Infinite scroll or pagination for large datasets
        - Search query persistence in URL
    - Priority: MEDIUM

### Technical Requirements

```yaml
security:
  - requirement: "Search query sanitization"
    implementation: "Prevent XSS through proper input handling"

  - requirement: "Access control"
    implementation: "Respect user permissions and clinic boundaries"

compatibility:
  - requirement: "Responsive design"
    scope: "Optimized for desktop, tablet, and mobile devices"
```

## Implementation Details

### Technical Approach

```yaml
# Frontend Search Implementation Strategy
ui_components:
  - component: "PatientSearchInterface"
    purpose: "Main search container with input and results"
    props: "onPatientSelect, initialQuery, filters"

  - component: "SearchInput"
    purpose: "Debounced search input with auto-complete"
    props: "value, onChange, placeholder, loading"

  - component: "SearchFilters"
    purpose: "Advanced filtering options panel"
    props: "filters, onFilterChange, onReset"

  - component: "PatientSearchResults"
    purpose: "Search results display with pagination"
    props: "patients, loading, onLoadMore, onPatientSelect"

  - component: "RecentPatientsPanel"
    purpose: "Quick access to recently viewed patients"
    props: "recentPatients, onPatientSelect"

state_management:
  - approach: "TanStack Query for search data with Zustand for UI state"
  - data_flow: "Search input -> debounced query -> API call -> results display"

integration_points:
  - api: "Patient Search API (TASK-002)"
    endpoints: "GET /api/patients, GET /api/patients/recent"
    error_handling: "Network errors, empty results, validation errors"
```

### Integration Points

```yaml
dependencies:
  - dependency: "Patient Search Implementation (TASK-002)"
    type: "API"
    status: "IN_PROGRESS"

  - dependency: "Patient Profile Frontend Interface (TASK-006)"
    type: "COMPONENT"
    status: "PLANNED"

provides:
  - deliverable: "Patient search UI components"
    interface: "React components for patient selection"
    consumers: "Dashboard, appointment scheduling, clinical workflows"
```

### Code Patterns to Follow

```typescript
// src/features/patients/components/PatientSearchInterface.tsx
import React, {useState, useCallback, useEffect} from 'react';
import {useTranslation} from 'react-i18next';
import {Search, Filter, Users, Clock} from 'lucide-react';
import {Input} from '@/components/ui/input';
import {Button} from '@/components/ui/button';
import {Card, CardContent, CardHeader, CardTitle} from '@/components/ui/card';
import {Tabs, TabsContent, TabsList, TabsTrigger} from '@/components/ui/tabs';
import {useDebounce} from '@/hooks/useDebounce';
import {usePatientSearch} from '../hooks/usePatientSearch';
import {useRecentPatients} from '../hooks/useRecentPatients';
import {PatientSearchResults} from './PatientSearchResults';
import {SearchFilters} from './SearchFilters';
import {RecentPatientsPanel} from './RecentPatientsPanel';
import {EmptySearchState} from './EmptySearchState';
import {patientSearchStore} from '../store/patient-search.store';
import type {PatientSummary, SearchFilters as SearchFiltersType} from '../types/patient.types';

interface PatientSearchInterfaceProps {
    onPatientSelect: (patient: PatientSummary) => void;
    initialQuery?: string;
    showRecentPatients?: boolean;
    placeholder?: string;
}

export const PatientSearchInterface: React.FC<PatientSearchInterfaceProps> = ({
                                                                                  onPatientSelect,
                                                                                  initialQuery = '',
                                                                                  showRecentPatients = true,
                                                                                  placeholder
                                                                              }) => {
    const {t} = useTranslation('patients');
    const [searchQuery, setSearchQuery] = useState(initialQuery);
    const [showFilters, setShowFilters] = useState(false);
    const [activeTab, setActiveTab] = useState<'search' | 'recent'>('search');

    // Search state management
    const {
        searchQuery: storeSearchQuery,
        filters,
        setSearchQuery: setStoreSearchQuery,
        setFilters,
        clearSearch
    } = patientSearchStore();

    // Debounced search query for API calls
    const debouncedQuery = useDebounce(searchQuery, 300);

    // API hooks
    const {
        data: searchResults,
        isLoading: isSearching,
        error: searchError,
        hasNextPage,
        fetchNextPage,
        isFetchingNextPage
    } = usePatientSearch({
        search: debouncedQuery,
        filters,
        enabled: debouncedQuery.length >= 2
    });

    const {
        data: recentPatients,
        isLoading: isLoadingRecent
    } = useRecentPatients({
        enabled: showRecentPatients && activeTab === 'recent'
    });

    // Sync local state with store
    useEffect(() => {
        if (debouncedQuery !== storeSearchQuery) {
            setStoreSearchQuery(debouncedQuery);
        }
    }, [debouncedQuery, storeSearchQuery, setStoreSearchQuery]);

    const handleSearchChange = useCallback((value: string) => {
        setSearchQuery(value);
        if (value.length >= 2) {
            setActiveTab('search');
        }
    }, []);

    const handleFilterChange = useCallback((newFilters: SearchFiltersType) => {
        setFilters(newFilters);
    }, [setFilters]);

    const handleClearSearch = useCallback(() => {
        setSearchQuery('');
        clearSearch();
        setShowFilters(false);
    }, [clearSearch]);

    const handlePatientSelect = useCallback((patient: PatientSummary) => {
        onPatientSelect(patient);
    }, [onPatientSelect]);

    const hasSearchQuery = searchQuery.length >= 2;
    const hasSearchResults = searchResults?.pages?.[0]?.data?.length > 0;
    const totalResults = searchResults?.pages?.[0]?.pagination?.total || 0;

    return (
        <div className = "space-y-4" >
        <Card>
            <CardHeader className = "pb-4" >
        <CardTitle className = "text-lg flex items-center gap-2" >
        <Search className = "h-5 w-5" / >
            {t('search.title')
}
    </CardTitle>
    < /CardHeader>
    < CardContent
    className = "space-y-4" >
        {/* Search Input */}
        < div
    className = "relative" >
    <Search className = "absolute left-3 top-1/2 transform -translate-y-1/2 h-4 w-4 text-muted-foreground" / >
    <Input
        value = {searchQuery}
    onChange = {(e)
=>
    handleSearchChange(e.target.value)
}
    placeholder = {placeholder || t('search.placeholder')
}
    className = "pl-9 pr-20"
    autoComplete = "off"
        / >
        {searchQuery && (
            <div className = "absolute right-2 top-1/2 transform -translate-y-1/2 flex gap-1" >
            <Button
                type = "button"
    variant = "ghost"
    size = "sm"
    onClick = {()
=>
    setShowFilters(!showFilters)
}
    className = "h-7 px-2"
    >
    <Filter className = "h-4 w-4" / >
        </Button>
        < Button
    type = "button"
    variant = "ghost"
    size = "sm"
    onClick = {handleClearSearch}
    className = "h-7 px-2"
        >
                  ×
                </Button>
                < /div>
)
}
    </div>

    {/* Advanced Filters */
    }
    {
        showFilters && (
            <SearchFilters
                filters = {filters}
        onFilterChange = {handleFilterChange}
        onClose = {()
    =>
        setShowFilters(false)
    }
        />
    )
    }

    {/* Search Results Count */
    }
    {
        hasSearchQuery && (
            <div className = "text-sm text-muted-foreground" >
                {
                    isSearching ? (
                        t('search.searching')
                    ) : hasSearchResults ? (
                        t('search.results_count', {count: totalResults})
                    ) : (
                        t('search.no_results')
                    )
                }
                < /div>
        )
    }
    </CardContent>
    < /Card>

    {/* Results Tabs */
    }
    <Tabs value = {activeTab}
    onValueChange = {(value)
=>
    setActiveTab(value as 'search' | 'recent')
}>
    <TabsList className = "grid w-full grid-cols-2" >
    <TabsTrigger value = "search"
    className = "flex items-center gap-2" >
    <Search className = "h-4 w-4" / >
        {t('search.search_results'
)
}
    </TabsTrigger>
    {
        showRecentPatients && (
            <TabsTrigger value = "recent"
        className = "flex items-center gap-2" >
        <Clock className = "h-4 w-4" / >
            {t('search.recent_patients'
    )
    }
        </TabsTrigger>
    )
    }
    </TabsList>

    {/* Search Results Tab */
    }
    <TabsContent value = "search"
    className = "mt-4" >
        {!
    hasSearchQuery ? (
        <EmptySearchState
            title = {t('search.empty_state.title')
}
    description = {t('search.empty_state.description'
)
}
    icon = {Search}
    />
) :
    isSearching ? (
        <PatientSearchSkeleton / >
    ) : hasSearchResults ? (
            <PatientSearchResults
                results = {searchResults}
        onPatientSelect = {handlePatientSelect}
    hasNextPage = {hasNextPage}
    onLoadMore = {fetchNextPage}
    isLoadingMore = {isFetchingNextPage}
    />
) :
    (
        <EmptySearchState
            title = {t('search.no_results_state.title')
}
    description = {t('search.no_results_state.description',
    {
        query: searchQuery
    }
)
}
    icon = {Users}
    action = {
    {
        label: t('search.no_results_state.create_patient'),
            onClick
    :
        () => {
            // Navigate to patient registration with pre-filled search term
        }
    }
}
    />
)
}

    {
        searchError && (
            <div className = "p-4 text-center text-red-600" >
                {t('search.error.general')
    }
        </div>
    )
    }
    </TabsContent>

    {/* Recent Patients Tab */
    }
    {
        showRecentPatients && (
            <TabsContent value = "recent"
        className = "mt-4" >
        <RecentPatientsPanel
            recentPatients = {recentPatients}
        isLoading = {isLoadingRecent}
        onPatientSelect = {handlePatientSelect}
        />
        < /TabsContent>
    )
    }
    </Tabs>
    < /div>
)
    ;
};

// src/features/patients/components/PatientSearchResults.tsx
import React from 'react';
import {useTranslation} from 'react-i18next';
import {InfiniteData} from '@tanstack/react-query';
import {Card, CardContent} from '@/components/ui/card';
import {Button} from '@/components/ui/button';
import {Avatar, AvatarFallback, AvatarImage} from '@/components/ui/avatar';
import {Badge} from '@/components/ui/badge';
import {formatDistanceToNow} from 'date-fns';
import {PatientSummary, PaginatedResult} from '../types/patient.types';

interface PatientSearchResultsProps {
    results: InfiniteData<PaginatedResult<PatientSummary>> | undefined;
    onPatientSelect: (patient: PatientSummary) => void;
    hasNextPage?: boolean;
    onLoadMore?: () => void;
    isLoadingMore?: boolean;
}

export const PatientSearchResults: React.FC<PatientSearchResultsProps> = ({
                                                                              results,
                                                                              onPatientSelect,
                                                                              hasNextPage,
                                                                              onLoadMore,
                                                                              isLoadingMore
                                                                          }) => {
    const {t} = useTranslation('patients');

    const allPatients = results?.pages?.flatMap(page => page.data) || [];

    if (allPatients.length === 0) {
        return null;
    }

    return (
        <div className = "space-y-3" >
            {
                allPatients.map((patient) => (
                    <PatientSearchCard
                        key = {patient.id}
                patient = {patient}
                onClick = {()
=>
    onPatientSelect(patient)
}
    />
))
}

    {
        hasNextPage && (
            <div className = "flex justify-center pt-4" >
            <Button
                variant = "outline"
        onClick = {onLoadMore}
        disabled = {isLoadingMore}
        loading = {isLoadingMore}
        >
        {isLoadingMore
            ? t('search.loading_more')
            : t('search.load_more')
    }
        </Button>
        < /div>
    )
    }
    </div>
)
    ;
};

// src/features/patients/components/PatientSearchCard.tsx
import React from 'react';
import {useTranslation} from 'react-i18next';
import {Card, CardContent} from '@/components/ui/card';
import {Avatar, AvatarFallback, AvatarImage} from '@/components/ui/avatar';
import {Badge} from '@/components/ui/badge';
import {Phone, Mail, Calendar} from 'lucide-react';
import {formatDistanceToNow} from 'date-fns';
import {PatientSummary} from '../types/patient.types';
import {getPatientStatusColor, calculateAge} from '../utils/patient.utils';

interface PatientSearchCardProps {
    patient: PatientSummary;
    onClick: () => void;
    showLastVisit?: boolean;
}

export const PatientSearchCard: React.FC<PatientSearchCardProps> = ({
                                                                        patient,
                                                                        onClick,
                                                                        showLastVisit = true
                                                                    }) => {
    const {t} = useTranslation('patients');

    const patientAge = calculateAge(patient.dateOfBirth);
    const statusColor = getPatientStatusColor(patient.status);

    return (
        <Card
            className = "cursor-pointer hover:shadow-md transition-shadow"
    onClick = {onClick}
    >
    <CardContent className = "p-4" >
    <div className = "flex items-start gap-4" >
        {/* Patient Avatar */}
        < Avatar
    className = "h-12 w-12" >
    <AvatarImage src = {patient.photoUrl}
    alt = {patient.firstName}
    />
    < AvatarFallback >
    {patient.firstName.charAt(0)}
    {
        patient.lastName.charAt(0)
    }
    </AvatarFallback>
    < /Avatar>

    {/* Patient Information */
    }
    <div className = "flex-1 min-w-0" >
    <div className = "flex items-start justify-between" >
    <div>
        <h3 className = "font-medium text-base" >
        {patient.firstName}
    {
        patient.lastName
    }
    </h3>
    < p
    className = "text-sm text-muted-foreground" >
        {t('patient_card.age_gender',
    {
        age: patientAge,
            gender
    :
        t(`gender.${patient.gender}`)
    }
)
}
    </p>
    < /div>
    < Badge
    variant = {statusColor} >
        {t(`patient_status.${patient.status}`
)
}
    </Badge>
    < /div>

    {/* Contact Information */
    }
    <div className = "flex flex-wrap gap-4 mt-2 text-sm text-muted-foreground" >
        {
            patient.phone && (
                <div className = "flex items-center gap-1" >
                <Phone className = "h-3 w-3" / >
                    <span>{patient.phone} < /span>
                    < /div>
            )
        }
    {
        patient.email && (
            <div className = "flex items-center gap-1" >
            <Mail className = "h-3 w-3" / >
            <span className = "truncate max-w-[200px]" > {patient.email} < /span>
                < /div>
        )
    }
    {
        showLastVisit && patient.lastVisit && (
            <div className = "flex items-center gap-1" >
            <Calendar className = "h-3 w-3" / >
                <span>
                    {t('patient_card.last_visit')
    }:
        {
            formatDistanceToNow(new Date(patient.lastVisit), {addSuffix: true})
        }
        </span>
        < /div>
    )
    }
    </div>
    < /div>
    < /div>
    < /CardContent>
    < /Card>
)
    ;
};

// src/features/patients/hooks/usePatientSearch.ts
import {useInfiniteQuery} from '@tanstack/react-query';
import {patientApi} from '../api/patients.api';
import {SearchPatientsParams} from '../types/patient.types';

interface UsePatientSearchOptions extends SearchPatientsParams {
    enabled?: boolean;
}

export const usePatientSearch = (options: UsePatientSearchOptions) => {
    const {enabled = true, ...searchParams} = options;

    return useInfiniteQuery({
        queryKey: ['patients', 'search', searchParams],
        queryFn: ({pageParam = 1}) =>
            patientApi.search({
                ...searchParams,
                page: pageParam
            }),
        getNextPageParam: (lastPage) => {
            const {page, totalPages} = lastPage.pagination;
            return page < totalPages ? page + 1 : undefined;
        },
        enabled: enabled && (searchParams.search?.length >= 2 || Object.keys(searchParams.filters || {}).length > 0),
        staleTime: 30000, // 30 seconds
        cacheTime: 300000, // 5 minutes
    });
};

// src/features/patients/hooks/useRecentPatients.ts
import {useQuery} from '@tanstack/react-query';
import {patientApi} from '../api/patients.api';

interface UseRecentPatientsOptions {
    enabled?: boolean;
}

export const useRecentPatients = (options: UseRecentPatientsOptions = {}) => {
    const {enabled = true} = options;

    return useQuery({
        queryKey: ['patients', 'recent'],
        queryFn: () => patientApi.getRecent(),
        enabled,
        staleTime: 60000, // 1 minute
        cacheTime: 300000, // 5 minutes
    });
};

// src/features/patients/store/patient-search.store.ts
import {create} from 'zustand';
import {persist} from 'zustand/middleware';
import {SearchFilters} from '../types/patient.types';

interface PatientSearchStore {
    searchQuery: string;
    filters: SearchFilters;
    sortBy: string;
    sortOrder: 'asc' | 'desc';

    setSearchQuery: (query: string) => void;
    setFilters: (filters: SearchFilters) => void;
    setSorting: (sortBy: string, sortOrder: 'asc' | 'desc') => void;
    clearSearch: () => void;
    clearFilters: () => void;
}

export const patientSearchStore = create<PatientSearchStore>()(
    persist(
        (set) => ({
            searchQuery: '',
            filters: {},
            sortBy: 'relevance',
            sortOrder: 'desc',

            setSearchQuery: (searchQuery) => set({searchQuery}),

            setFilters: (filters) => set({filters}),

            setSorting: (sortBy, sortOrder) => set({sortBy, sortOrder}),

            clearSearch: () => set({
                searchQuery: '',
                filters: {},
                sortBy: 'relevance',
                sortOrder: 'desc'
            }),

            clearFilters: () => set({filters: {}})
        }),
        {
            name: 'patient-search-state',
            partialize: (state) => ({
                filters: state.filters,
                sortBy: state.sortBy,
                sortOrder: state.sortOrder
            })
        }
    )
);
```

## Testing Requirements

### Test Coverage

```yaml
unit_tests:
  - focus: "Search component logic and state management"
  - coverage_target: "85%"
  - key_scenarios: [
    "Search input debouncing and API calls",
    "Filter application and persistence",
    "Recent patients display and selection",
    "Search results rendering and pagination",
    "Error handling and empty states"
  ]

integration_tests:
  - focus: "Complete search workflow and API integration"
  - test_environment: "Testing environment with mock patient data"
  - key_flows: [
    "End-to-end patient search workflow",
    "Filter combination and result refinement",
    "Recent patients functionality",
    "Search performance and caching"
  ]
```

### Test Scenarios

```yaml
happy_path:
  - scenario: "Search for patient by name"
    steps: [
      "Enter patient name in search input",
      "Wait for debounced API call",
      "Verify search results display",
      "Click on patient to select"
    ]
    expected_result: "Patient selected and callback triggered"

error_cases:
  - scenario: "Search with no network connection"
    trigger: "Attempt search while offline"
    expected_behavior: "Error message displayed, retry option available"

  - scenario: "Search with invalid characters"
    trigger: "Enter special characters in search input"
    expected_behavior: "Input sanitized, appropriate error message if needed"

edge_cases:
  - scenario: "Search with Arabic patient names"
    conditions: "Patient database contains Arabic names"
    expected_behavior: "Search works correctly with RTL text and Arabic characters"
```

## Context References

### From Project

```yaml
tech_stack_references:
  - "React with TypeScript for search components"
  - "TanStack Query for data fetching and caching"
  - "Zustand for search state management"
  - "shadcn/ui for consistent UI components"

architecture_patterns:
  - "Custom hooks for API integration"
  - "Store pattern for search state persistence"
  - "Component composition for modular search interface"

code_standards:
  - "Debounced input handling for performance"
  - "Accessible search interface with ARIA attributes"
  - "Responsive design for various screen sizes"
```

### From Epic

```yaml
business_rules:
  - "BR-P017: Multi-criteria search support"
  - "BR-P018: Relevance-based result sorting"
  - "BR-P019: Case-insensitive search"
  - "BR-P020: Recent patients tracking"
  - "BR-P021: Permission-based result filtering"

domain_model:
  - "PatientSummary for search result display"
  - "SearchFilters for advanced filtering"
  - "PaginatedResult for search response handling"

integration_points:
  - "Consumes Patient Search API endpoints"
  - "Provides patient selection for other workflows"
  - "Integrates with recent patients tracking"
```

## Acceptance Criteria

### Functional Acceptance

- [ ] Real-time search with debounced input works correctly
- [ ] Advanced filtering options refine search results properly
- [ ] Recent patients panel provides quick access to last 10 patients
- [ ] Search results display relevant patient information clearly
- [ ] Pagination or infinite scroll handles large result sets
- [ ] Search performance meets responsiveness requirements

### Technical Acceptance

- [ ] Components follow React and TypeScript best practices
- [ ] Unit tests written and passing (>= 85% coverage)
- [ ] Integration tests cover search workflows
- [ ] Search state persistence works across sessions
- [ ] Error handling provides appropriate user feedback
- [ ] Responsive design works on all target devices

### Quality Acceptance

- [ ] Code review completed and approved
- [ ] Accessibility testing passes WCAG 2.1 AA standards
- [ ] Performance testing confirms search response times
- [ ] User experience testing validates intuitive search flow
- [ ] Multi-language testing confirms proper text display

---
**Last Updated:** 2025-01-16