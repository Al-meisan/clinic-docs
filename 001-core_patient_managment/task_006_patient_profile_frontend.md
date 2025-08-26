# TASK: TASK-006 - Patient Profile Frontend Interface

## Task Overview

### Goal
**What to Build:** Comprehensive patient profile interface with view and edit capabilities, change history display, status management, and seamless integration with the patient profile management API.

**Why:** Provide healthcare staff with a complete view of patient information that can be easily updated and maintained, supporting efficient patient care and administrative workflows.

**Definition of Done:** 
- Patient profile view displays all patient information in organized tabs
- Edit mode allows inline editing with validation and optimistic updates
- Change history shows complete audit trail of profile modifications
- Status management with proper workflow transitions
- Responsive design optimized for clinical workflows

### Scope
```yaml
task_type: "FRONTEND"
complexity: "HIGH"
```

## Requirements

### Functional Requirements
1. **Patient Profile Overview**
   - Description: Comprehensive patient information display with tabbed navigation
   - Acceptance Criteria: 
     - Patient demographics, contact info, and emergency contacts displayed
     - Photo display with update capability
     - Insurance information with coverage details
     - Quick action buttons for common tasks
   - Priority: HIGH

2. **Inline Edit Functionality**
   - Description: Edit patient information directly within the profile view
   - Acceptance Criteria:
     - Click-to-edit fields with inline validation
     - Optimistic updates for immediate feedback
     - Conflict resolution for concurrent edits
     - Save/cancel actions with proper state management
   - Priority: HIGH

3. **Change History Display**
   - Description: Complete audit trail of patient profile modifications
   - Acceptance Criteria:
     - Chronological list of all changes with timestamps
     - User attribution for each modification
     - Before/after value comparison
     - Filtering by date range and change type
   - Priority: MEDIUM

4. **Status Management Interface**
   - Description: Patient status updates with workflow validation
   - Acceptance Criteria:
     - Status change dropdown with valid transitions only
     - Reason requirement for status changes
     - Confirmation dialog for critical status changes
     - Visual indicators for current status
   - Priority: MEDIUM

5. **Emergency Contact Management**
   - Description: Add, edit, and manage patient emergency contacts
   - Acceptance Criteria:
     - Add/remove emergency contacts dynamically
     - Designate primary emergency contact
     - Contact validation and formatting
     - Relationship type management
   - Priority: HIGH

### Technical Requirements
```yaml
performance:
  - requirement: "Profile load time"
    target: "< 1 second for initial profile display"
    
  - requirement: "Edit mode responsiveness"
    target: "< 100ms for edit state transitions"
    
security:
  - requirement: "Optimistic locking support"
    implementation: "Handle concurrent edit conflicts gracefully"
    
  - requirement: "Field-level permissions"
    implementation: "Show/hide editable fields based on user permissions"
    
compatibility:
  - requirement: "Mobile responsiveness"
    scope: "Optimized layout for tablet and mobile use"
```

## Implementation Details

### Technical Approach
```yaml
# Patient Profile Implementation Strategy
ui_components:
  - component: "PatientProfilePage"
    purpose: "Main profile container with tabbed navigation"
    props: "patientId, mode, onModeChange"
    
  - component: "PatientOverviewTab"
    purpose: "Demographics and basic information display/edit"
    props: "patient, isEditing, onSave, onCancel"
    
  - component: "PatientContactTab"
    purpose: "Contact information and address management"
    props: "contactInfo, emergencyContacts, isEditing"
    
  - component: "PatientHistoryTab"
    purpose: "Change history and audit trail display"
    props: "patientId, history, filters"
    
  - component: "PatientStatusManager"
    purpose: "Status change interface with workflow validation"
    props: "currentStatus, allowedTransitions, onStatusChange"
    
state_management:
  - approach: "TanStack Query for data with optimistic updates"
  - data_flow: "Profile load -> edit mode -> optimistic update -> API sync"
  
integration_points:
  - api: "Patient Profile Management API (TASK-003)"
    endpoints: "GET/PUT /api/patients/{id}, GET /api/patients/{id}/history"
    error_handling: "Conflict resolution, validation errors, network failures"
```

### Integration Points
```yaml
dependencies:
  - dependency: "Patient Profile Management API (TASK-003)"
    type: "API"
    status: "IN_PROGRESS"
    
  - dependency: "Patient Search Frontend Interface (TASK-005)"
    type: "COMPONENT"
    status: "IN_PROGRESS"
    
provides:
  - deliverable: "Patient profile UI components"
    interface: "React components for patient information management"
    consumers: "Patient search results, appointment workflows, clinical documentation"
```

### Code Patterns to Follow
```typescript
// src/features/patients/pages/PatientProfilePage.tsx
import React, { useState } from 'react';
import { useParams, useNavigate } from 'react-router-dom';
import { useTranslation } from 'react-i18next';
import { ArrowLeft, Edit, Save, X, History, User, Contact, FileText } from 'lucide-react';
import { Button } from '@/components/ui/button';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs';
import { Badge } from '@/components/ui/badge';
import { Avatar, AvatarFallback, AvatarImage } from '@/components/ui/avatar';
import { toast } from 'sonner';
import { usePatient } from '../hooks/usePatient';
import { useUpdatePatient } from '../hooks/useUpdatePatient';
import { PatientOverviewTab } from '../components/PatientOverviewTab';
import { PatientContactTab } from '../components/PatientContactTab';
import { PatientHistoryTab } from '../components/PatientHistoryTab';
import { PatientStatusManager } from '../components/PatientStatusManager';
import { PatientNotFound } from '../components/PatientNotFound';
import { PatientProfileSkeleton } from '../components/PatientProfileSkeleton';
import { calculateAge, getPatientStatusColor } from '../utils/patient.utils';
import type { Patient, UpdatePatientRequest } from '../types/patient.types';

export const PatientProfilePage: React.FC = () => {
  const { patientId } = useParams<{ patientId: string }>();
  const navigate = useNavigate();
  const { t } = useTranslation('patients');
  const [isEditing, setIsEditing] = useState(false);
  const [editingData, setEditingData] = useState<Partial<UpdatePatientRequest> | null>(null);
  const [activeTab, setActiveTab] = useState('overview');

  // API hooks
  const {
    data: patient,
    isLoading,
    error,
    refetch
  } = usePatient(patientId!);

  const updatePatientMutation = useUpdatePatient({
    onSuccess: (updatedPatient) => {
      setIsEditing(false);
      setEditingData(null);
      toast.success(t('profile.update_success'));
    },
    onError: (error) => {
      if (error.status === 409) {
        toast.error(t('profile.conflict_error'));
        refetch(); // Refresh to get latest data
      } else {
        toast.error(t('profile.update_error'));
      }
    }
  });

  if (isLoading) {
    return <PatientProfileSkeleton />;
  }

  if (error || !patient) {
    return <PatientNotFound onGoBack={() => navigate('/patients')} />;
  }

  const handleEditToggle = () => {
    if (isEditing) {
      setIsEditing(false);
      setEditingData(null);
    } else {
      setIsEditing(true);
      setEditingData({
        firstName: patient.firstName,
        lastName: patient.lastName,
        middleName: patient.middleName,
        phone: patient.contactInfo?.phone,
        email: patient.contactInfo?.email,
        // ... other editable fields
        version: patient.version
      });
    }
  };

  const handleSave = async () => {
    if (!editingData || !patientId) return;

    try {
      await updatePatientMutation.mutateAsync({
        patientId,
        data: editingData
      });
    } catch (error) {
      // Error handling is done in the mutation
    }
  };

  const handleFieldChange = (field: keyof UpdatePatientRequest, value: any) => {
    setEditingData(prev => prev ? { ...prev, [field]: value } : null);
  };

  const patientAge = calculateAge(patient.dateOfBirth);
  const statusColor = getPatientStatusColor(patient.status);

  return (
    <div className="container mx-auto p-6 space-y-6">
      {/* Header */}
      <div className="flex items-start justify-between">
        <div className="flex items-start gap-4">
          <Button
            variant="ghost"
            size="sm"
            onClick={() => navigate('/patients')}
            className="mt-1"
          >
            <ArrowLeft className="h-4 w-4 mr-2" />
            {t('common.back')}
          </Button>
          
          <div className="flex items-start gap-4">
            <Avatar className="h-16 w-16">
              <AvatarImage src={patient.photoUrl} alt={patient.firstName} />
              <AvatarFallback className="text-lg">
                {patient.firstName.charAt(0)}{patient.lastName.charAt(0)}
              </AvatarFallback>
            </Avatar>
            
            <div>
              <h1 className="text-2xl font-bold">
                {patient.firstName} {patient.lastName}
              </h1>
              <p className="text-muted-foreground">
                {t('profile.age_gender', { 
                  age: patientAge, 
                  gender: t(`gender.${patient.gender}`) 
                })}
              </p>
              <div className="flex items-center gap-2 mt-1">
                <Badge variant={statusColor}>
                  {t(`patient_status.${patient.status}`)}
                </Badge>
                <span className="text-sm text-muted-foreground">
                  ID: {patient.id.slice(-8)}
                </span>
              </div>
            </div>
          </div>
        </div>

        {/* Actions */}
        <div className="flex items-center gap-2">
          {isEditing ? (
            <>
              <Button
                variant="outline"
                onClick={handleEditToggle}
                disabled={updatePatientMutation.isPending}
              >
                <X className="h-4 w-4 mr-2" />
                {t('common.cancel')}
              </Button>
              <Button
                onClick={handleSave}
                disabled={updatePatientMutation.isPending}
                loading={updatePatientMutation.isPending}
              >
                <Save className="h-4 w-4 mr-2" />
                {t('common.save')}
              </Button>
            </>
          ) : (
            <Button onClick={handleEditToggle}>
              <Edit className="h-4 w-4 mr-2" />
              {t('common.edit')}
            </Button>
          )}
        </div>
      </div>

      {/* Profile Tabs */}
      <Tabs value={activeTab} onValueChange={setActiveTab}>
        <TabsList>
          <TabsTrigger value="overview" className="flex items-center gap-2">
            <User className="h-4 w-4" />
            {t('profile.tabs.overview')}
          </TabsTrigger>
          <TabsTrigger value="contact" className="flex items-center gap-2">
            <Contact className="h-4 w-4" />
            {t('profile.tabs.contact')}
          </TabsTrigger>
          <TabsTrigger value="history" className="flex items-center gap-2">
            <History className="h-4 w-4" />
            {t('profile.tabs.history')}
          </TabsTrigger>
        </TabsList>

        <TabsContent value="overview" className="mt-6">
          <PatientOverviewTab
            patient={patient}
            isEditing={isEditing}
            editingData={editingData}
            onFieldChange={handleFieldChange}
          />
        </TabsContent>

        <TabsContent value="contact" className="mt-6">
          <PatientContactTab
            patient={patient}
            isEditing={isEditing}
            editingData={editingData}
            onFieldChange={handleFieldChange}
          />
        </TabsContent>

        <TabsContent value="history" className="mt-6">
          <PatientHistoryTab patientId={patientId!} />
        </TabsContent>
      </Tabs>
    </div>
  );
};

// src/features/patients/components/PatientOverviewTab.tsx
import React from 'react';
import { useTranslation } from 'react-i18next';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { Textarea } from '@/components/ui/textarea';
import { Calendar } from '@/components/ui/calendar';
import { Popover, PopoverContent, PopoverTrigger } from '@/components/ui/popover';
import { Button } from '@/components/ui/button';
import { CalendarIcon } from 'lucide-react';
import { format } from 'date-fns';
import { cn } from '@/lib/utils';
import { PatientStatusManager } from './PatientStatusManager';
import type { Patient, UpdatePatientRequest } from '../types/patient.types';

interface PatientOverviewTabProps {
  patient: Patient;
  isEditing: boolean;
  editingData: Partial<UpdatePatientRequest> | null;
  onFieldChange: (field: keyof UpdatePatientRequest, value: any) => void;
}

export const PatientOverviewTab: React.FC<PatientOverviewTabProps> = ({
  patient,
  isEditing,
  editingData,
  onFieldChange
}) => {
  const { t } = useTranslation('patients');

  const displayValue = (field: keyof UpdatePatientRequest, fallback: any) => {
    return isEditing ? editingData?.[field] ?? fallback : fallback;
  };

  return (
    <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
      {/* Basic Information */}
      <Card>
        <CardHeader>
          <CardTitle>{t('profile.sections.basic_info')}</CardTitle>
        </CardHeader>
        <CardContent className="space-y-4">
          <div className="grid grid-cols-2 gap-4">
            <div className="space-y-2">
              <Label>{t('fields.first_name')}</Label>
              {isEditing ? (
                <Input
                  value={displayValue('firstName', patient.firstName)}
                  onChange={(e) => onFieldChange('firstName', e.target.value)}
                />
              ) : (
                <p className="text-sm">{patient.firstName}</p>
              )}
            </div>
            
            <div className="space-y-2">
              <Label>{t('fields.last_name')}</Label>
              {isEditing ? (
                <Input
                  value={displayValue('lastName', patient.lastName)}
                  onChange={(e) => onFieldChange('lastName', e.target.value)}
                />
              ) : (
                <p className="text-sm">{patient.lastName}</p>
              )}
            </div>
          </div>

          <div className="space-y-2">
            <Label>{t('fields.middle_name')}</Label>
            {isEditing ? (
              <Input
                value={displayValue('middleName', patient.middleName) || ''}
                onChange={(e) => onFieldChange('middleName', e.target.value)}
              />
            ) : (
              <p className="text-sm">{patient.middleName || t('common.not_provided')}</p>
            )}
          </div>

          <div className="grid grid-cols-2 gap-4">
            <div className="space-y-2">
              <Label>{t('fields.date_of_birth')}</Label>
              {isEditing ? (
                <Popover>
                  <PopoverTrigger asChild>
                    <Button
                      variant="outline"
                      className={cn(
                        "w-full justify-start text-left font-normal",
                        !editingData?.dateOfBirth && "text-muted-foreground"
                      )}
                    >
                      <CalendarIcon className="mr-2 h-4 w-4" />
                      {editingData?.dateOfBirth
                        ? format(new Date(editingData.dateOfBirth), "PPP")
                        : format(new Date(patient.dateOfBirth), "PPP")
                      }
                    </Button>
                  </PopoverTrigger>
                  <PopoverContent className="w-auto p-0">
                    <Calendar
                      mode="single"
                      selected={editingData?.dateOfBirth 
                        ? new Date(editingData.dateOfBirth) 
                        : new Date(patient.dateOfBirth)
                      }
                      onSelect={(date) => onFieldChange('dateOfBirth', date?.toISOString())}
                      disabled={(date) => date > new Date()}
                      initialFocus
                    />
                  </PopoverContent>
                </Popover>
              ) : (
                <p className="text-sm">{format(new Date(patient.dateOfBirth), "PPP")}</p>
              )}
            </div>
            
            <div className="space-y-2">
              <Label>{t('fields.gender')}</Label>
              {isEditing ? (
                <Select
                  value={displayValue('gender', patient.gender)}
                  onValueChange={(value) => onFieldChange('gender', value)}
                >
                  <SelectTrigger>
                    <SelectValue />
                  </SelectTrigger>
                  <SelectContent>
                    <SelectItem value="male">{t('gender.male')}</SelectItem>
                    <SelectItem value="female">{t('gender.female')}</SelectItem>
                    <SelectItem value="other">{t('gender.other')}</SelectItem>
                  </SelectContent>
                </Select>
              ) : (
                <p className="text-sm">{t(`gender.${patient.gender}`)}</p>
              )}
            </div>
          </div>

          <div className="space-y-2">
            <Label>{t('fields.notes')}</Label>
            {isEditing ? (
              <Textarea
                value={displayValue('notes', patient.notes) || ''}
                onChange={(e) => onFieldChange('notes', e.target.value)}
                placeholder={t('placeholders.patient_notes')}
                rows={3}
              />
            ) : (
              <p className="text-sm whitespace-pre-wrap">
                {patient.notes || t('common.no_notes')}
              </p>
            )}
          </div>
        </CardContent>
      </Card>

      {/* Status Management */}
      <Card>
        <CardHeader>
          <CardTitle>{t('profile.sections.status_management')}</CardTitle>
        </CardHeader>
        <CardContent>
          <PatientStatusManager
            patient={patient}
            disabled={isEditing}
          />
        </CardContent>
      </Card>
    </div>
  );
};

// src/features/patients/components/PatientHistoryTab.tsx
import React, { useState } from 'react';
import { useTranslation } from 'react-i18next';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Input } from '@/components/ui/input';
import { Button } from '@/components/ui/button';
import { Calendar } from '@/components/ui/calendar';
import { Popover, PopoverContent, PopoverTrigger } from '@/components/ui/popover';
import { Badge } from '@/components/ui/badge';
import { CalendarIcon, Filter, User } from 'lucide-react';
import { format, formatDistanceToNow } from 'date-fns';
import { usePatientHistory } from '../hooks/usePatientHistory';
import { PatientHistorySkeleton } from './PatientHistorySkeleton';
import type { PatientAuditLog } from '../types/patient.types';

interface PatientHistoryTabProps {
  patientId: string;
}

export const PatientHistoryTab: React.FC<PatientHistoryTabProps> = ({
  patientId
}) => {
  const { t } = useTranslation('patients');
  const [dateFrom, setDateFrom] = useState<Date>();
  const [dateTo, setDateTo] = useState<Date>();
  const [showFilters, setShowFilters] = useState(false);

  const {
    data: history,
    isLoading,
    error
  } = usePatientHistory(patientId, {
    fromDate: dateFrom,
    toDate: dateTo
  });

  const clearFilters = () => {
    setDateFrom(undefined);
    setDateTo(undefined);
  };

  if (isLoading) {
    return <PatientHistorySkeleton />;
  }

  if (error) {
    return (
      <Card>
        <CardContent className="p-6 text-center text-red-600">
          {t('profile.history.error')}
        </CardContent>
      </Card>
    );
  }

  return (
    <div className="space-y-4">
      {/* Filters */}
      <Card>
        <CardHeader className="pb-4">
          <div className="flex items-center justify-between">
            <CardTitle className="text-lg">{t('profile.history.title')}</CardTitle>
            <Button
              variant="outline"
              size="sm"
              onClick={() => setShowFilters(!showFilters)}
            >
              <Filter className="h-4 w-4 mr-2" />
              {t('common.filters')}
            </Button>
          </div>
        </CardHeader>
        
        {showFilters && (
          <CardContent className="pt-0">
            <div className="flex gap-4 items-end">
              <div className="space-y-2">
                <label className="text-sm font-medium">{t('profile.history.from_date')}</label>
                <Popover>
                  <PopoverTrigger asChild>
                    <Button variant="outline" size="sm">
                      <CalendarIcon className="h-4 w-4 mr-2" />
                      {dateFrom ? format(dateFrom, "PPP") : t('common.select_date')}
                    </Button>
                  </PopoverTrigger>
                  <PopoverContent className="w-auto p-0">
                    <Calendar
                      mode="single"
                      selected={dateFrom}
                      onSelect={setDateFrom}
                      disabled={(date) => date > new Date()}
                    />
                  </PopoverContent>
                </Popover>
              </div>
              
              <div className="space-y-2">
                <label className="text-sm font-medium">{t('profile.history.to_date')}</label>
                <Popover>
                  <PopoverTrigger asChild>
                    <Button variant="outline" size="sm">
                      <CalendarIcon className="h-4 w-4 mr-2" />
                      {dateTo ? format(dateTo, "PPP") : t('common.select_date')}
                    </Button>
                  </PopoverTrigger>
                  <PopoverContent className="w-auto p-0">
                    <Calendar
                      mode="single"
                      selected={dateTo}
                      onSelect={setDateTo}
                      disabled={(date) => date > new Date() || (dateFrom && date < dateFrom)}
                    />
                  </PopoverContent>
                </Popover>
              </div>
              
              <Button variant="ghost" size="sm" onClick={clearFilters}>
                {t('common.clear')}
              </Button>
            </div>
          </CardContent>
        )}
      </Card>

      {/* History Timeline */}
      <div className="space-y-3">
        {history && history.length > 0 ? (
          history.map((entry) => (
            <HistoryEntryCard key={entry.id} entry={entry} />
          ))
        ) : (
          <Card>
            <CardContent className="p-6 text-center text-muted-foreground">
              {t('profile.history.no_changes')}
            </CardContent>
          </Card>
        )}
      </div>
    </div>
  );
};

// src/features/patients/components/HistoryEntryCard.tsx
import React, { useState } from 'react';
import { useTranslation } from 'react-i18next';
import { Card, CardContent } from '@/components/ui/card';
import { Badge } from '@/components/ui/badge';
import { Button } from '@/components/ui/button';
import { ChevronDown, ChevronRight, User } from 'lucide-react';
import { formatDistanceToNow } from 'date-fns';
import type { PatientAuditLog } from '../types/patient.types';

interface HistoryEntryCardProps {
  entry: PatientAuditLog;
}

export const HistoryEntryCard: React.FC<HistoryEntryCardProps> = ({ entry }) => {
  const { t } = useTranslation('patients');
  const [isExpanded, setIsExpanded] = useState(false);

  const getActionColor = (action: string) => {
    switch (action) {
      case 'CREATE': return 'default';
      case 'UPDATE': return 'secondary';
      case 'STATUS_CHANGE': return 'destructive';
      default: return 'outline';
    }
  };

  const getActionIcon = (action: string) => {
    switch (action) {
      case 'CREATE': return '➕';
      case 'UPDATE': return '✏️';
      case 'STATUS_CHANGE': return '🔄';
      default: return '📝';
    }
  };

  return (
    <Card className="relative">
      <CardContent className="p-4">
        <div className="flex items-start justify-between">
          <div className="flex items-start gap-3">
            <div className="text-xl">{getActionIcon(entry.action)}</div>
            <div className="flex-1">
              <div className="flex items-center gap-2 mb-1">
                <Badge variant={getActionColor(entry.action)}>
                  {t(`audit_actions.${entry.action.toLowerCase()}`)}
                </Badge>
                <span className="text-sm text-muted-foreground">
                  {formatDistanceToNow(new Date(entry.createdAt), { addSuffix: true })}
                </span>
              </div>
              
              <div className="flex items-center gap-2 text-sm text-muted-foreground">
                <User className="h-3 w-3" />
                <span>{entry.user?.firstName} {entry.user?.lastName}</span>
              </div>
              
              {entry.reason && (
                <p className="text-sm mt-1">{entry.reason}</p>
              )}
              
              {entry.changedFields && entry.changedFields.length > 0 && (
                <div className="mt-2">
                  <p className="text-sm text-muted-foreground">
                    {t('profile.history.changed_fields')}: {entry.changedFields.join(', ')}
                  </p>
                </div>
              )}
            </div>
          </div>
          
          {(entry.oldValues || entry.newValues) && (
            <Button
              variant="ghost"
              size="sm"
              onClick={() => setIsExpanded(!isExpanded)}
            >
              {isExpanded ? (
                <ChevronDown className="h-4 w-4" />
              ) : (
                <ChevronRight className="h-4 w-4" />
              )}
            </Button>
          )}
        </div>
        
        {isExpanded && (entry.oldValues || entry.newValues) && (
          <div className="mt-4 pt-4 border-t">
            <div className="grid grid-cols-1 md:grid-cols-2 gap-4 text-sm">
              {entry.oldValues && (
                <div>
                  <h4 className="font-medium text-muted-foreground mb-2">
                    {t('profile.history.old_values')}
                  </h4>
                  <pre className="bg-muted p-2 rounded text-xs overflow-auto">
                    {JSON.stringify(entry.oldValues, null, 2)}
                  </pre>
                </div>
              )}
              
              {entry.newValues && (
                <div>
                  <h4 className="font-medium text-muted-foreground mb-2">
                    {t('profile.history.new_values')}
                  </h4>
                  <pre className="bg-muted p-2 rounded text-xs overflow-auto">
                    {JSON.stringify(entry.newValues, null, 2)}
                  </pre>
                </div>
              )}
            </div>
          </div>
        )}
      </CardContent>
    </Card>
  );
};
```

## Testing Requirements

### Test Coverage
```yaml
unit_tests:
  - focus: "Profile display and edit functionality"
  - coverage_target: "85%"
  - key_scenarios: [
    "Profile data display and formatting",
    "Edit mode state management",
    "Optimistic updates and conflict resolution",
    "Status management workflow",
    "Change history display and filtering"
  ]
  
integration_tests:
  - focus: "Complete profile management workflow"
  - test_environment: "Testing environment with patient data fixtures"
  - key_flows: [
    "Profile view and navigation",
    "Profile editing and saving",
    "Concurrent edit conflict handling",
    "Status change workflow",
    "History viewing and filtering"
  ]
```

### Test Scenarios
```yaml
happy_path:
  - scenario: "View and edit patient profile"
    steps: [
      "Navigate to patient profile",
      "Click edit button",
      "Modify patient information",
      "Save changes successfully"
    ]
    expected_result: "Profile updated with optimistic UI feedback"
    
error_cases:
  - scenario: "Concurrent edit conflict"
    trigger: "Save changes while another user has modified the same patient"
    expected_behavior: "Conflict error displayed, option to refresh and retry"
    
  - scenario: "Invalid status transition"
    trigger: "Attempt to change patient status to invalid transition"
    expected_behavior: "Status change rejected with appropriate error message"
    
edge_cases:
  - scenario: "Profile with Arabic patient information"
    conditions: "Patient has Arabic name and address"
    expected_behavior: "Profile displays correctly with RTL text formatting"
```

## Context References

### From Project
```yaml
tech_stack_references:
  - "React with TypeScript for profile components"
  - "TanStack Query for data management with optimistic updates"
  - "React Hook Form for edit form management"
  - "shadcn/ui for consistent UI components"
  
architecture_patterns:
  - "Page-level components for route management"
  - "Tab-based navigation for profile sections"
  - "Optimistic updates for improved UX"
  
code_standards:
  - "Proper error handling and user feedback"
  - "Responsive design for various screen sizes"
  - "Accessibility compliance with proper ARIA attributes"
```

### From Epic
```yaml
business_rules:
  - "BR-P007: Audit trail for all modifications"
  - "BR-P008: Valid status transitions only"
  - "BR-P015: Contact information validation on updates"
  - "Optimistic locking to prevent data corruption"
  
domain_model:
  - "Patient entity with all profile information"
  - "PatientAuditLog for change history"
  - "UpdatePatientRequest for profile modifications"
  
integration_points:
  - "Integrates with Patient Profile Management API"
  - "Provides profile access for clinical workflows"
  - "Supports patient information updates from various entry points"
```

## Acceptance Criteria

### Functional Acceptance
- [ ] Patient profile displays all information in organized, readable format
- [ ] Edit mode allows inline editing with real-time validation
- [ ] Optimistic updates provide immediate feedback before API confirmation
- [ ] Change history shows complete audit trail with proper filtering
- [ ] Status management enforces valid transitions with proper workflow
- [ ] Emergency contact management works for multiple contacts

### Technical Acceptance  
- [ ] Components follow React and TypeScript best practices
- [ ] Unit tests written and passing (>= 85% coverage)
- [ ] Integration tests cover profile management workflows
- [ ] Optimistic locking handles concurrent edit conflicts properly
- [ ] Performance requirements met for profile load and edit operations
- [ ] Responsive design works across different screen sizes

### Quality Acceptance
- [ ] Code review completed and approved
- [ ] Accessibility testing passes WCAG 2.1 AA standards
- [ ] User experience testing confirms intuitive profile management
- [ ] Error handling provides clear feedback for all failure scenarios
- [ ] Multi-language testing validates proper text display and RTL support

---
**Last Updated:** 2025-01-16