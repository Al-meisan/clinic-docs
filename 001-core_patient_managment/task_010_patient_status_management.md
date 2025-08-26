# TASK: TASK-010 - Patient Status Management

## Task Overview

### Goal
**What to Build:** Comprehensive patient status management system with workflow validation, automated status transitions, and audit trail to ensure proper patient lifecycle management according to business rules.

**Why:** Maintain accurate patient status information for billing, scheduling, and clinical operations while ensuring compliance with healthcare regulations and clinic operational requirements.

**Definition of Done:** 
- Patient status workflow with validated transitions (active ↔ inactive, active → deceased, active → merged)
- Automated status management with business rule enforcement
- Status change audit trail with user attribution and reasons
- Status-based access control and filtering throughout the system
- Bulk status management for administrative operations

### Scope
```yaml
task_type: "FULLSTACK"
complexity: "MEDIUM"
```

## Requirements

### Functional Requirements
1. **Status Transition Workflow**
   - Description: Implement valid patient status transitions with business rule enforcement
   - Acceptance Criteria: 
     - Valid transitions: active ↔ inactive, active → deceased, active → merged
     - Invalid transitions blocked with clear error messages
     - Reason required for all status changes
     - Authorization checks for sensitive status changes (deceased, merged)
   - Priority: HIGH

2. **Automated Status Management**
   - Description: Automatic status updates based on patient activity and system events
   - Acceptance Criteria:
     - Auto-activate inactive patients when scheduled for appointments
     - Inactive status for patients with no activity for configurable period
     - Status notifications for administrative review
     - Batch status processing for large datasets
   - Priority: MEDIUM

3. **Status Change Audit Trail**
   - Description: Complete tracking of all patient status modifications
   - Acceptance Criteria:
     - Record all status changes with timestamp and user attribution
     - Track reason for status change and supporting documentation
     - Status change history accessible from patient profile
     - Audit reports for compliance and administrative review
   - Priority: HIGH

4. **Status-Based System Behavior**
   - Description: System behavior adaptation based on patient status
   - Acceptance Criteria:
     - Deceased patients cannot be scheduled for new appointments
     - Merged patients redirect to primary patient record
     - Inactive patients show warnings during scheduling
     - Status-based filtering in patient search and reports
   - Priority: HIGH

5. **Bulk Status Management**
   - Description: Administrative tools for managing multiple patient statuses
   - Acceptance Criteria:
     - Bulk status update interface with validation
     - CSV import/export for status management
     - Status change preview and confirmation
     - Progress tracking for large batch operations
   - Priority: LOW

### Technical Requirements
```yaml
performance:
  - requirement: "Status change processing time"
    target: "< 200ms for individual status updates"
    
  - requirement: "Bulk status operation performance"
    target: "Process 1000 status changes per minute"
    
security:
  - requirement: "Status change authorization"
    implementation: "Role-based permissions for different status transitions"
    
  - requirement: "Audit trail integrity"
    implementation: "Immutable status change records with cryptographic verification"
    
compatibility:
  - requirement: "Status consistency across system"
    scope: "Ensure status changes are reflected in all dependent systems"
```

## Implementation Details

### Technical Approach
```yaml
# Status Management Strategy
backend_implementation:
  - service: "PatientStatusService"
    purpose: "Core status management logic and validation"
    methods: "changeStatus, validateTransition, getStatusHistory"
    
  - service: "StatusWorkflowService"
    purpose: "Workflow automation and business rule enforcement"
    methods: "automateStatusChanges, scheduleStatusReviews"
    
  - service: "StatusAuditService"
    purpose: "Audit trail management and compliance reporting"
    methods: "logStatusChange, getAuditTrail, generateComplianceReport"

frontend_implementation:
  - component: "PatientStatusManager"
    purpose: "Status change interface with validation"
    features: "Status selection, reason entry, confirmation dialogs"
    
  - component: "StatusHistoryPanel"
    purpose: "Display status change history and audit trail"
    features: "Timeline view, filtering, detailed change information"
    
  - component: "BulkStatusManager"
    purpose: "Administrative bulk status management"
    features: "Batch selection, preview, progress tracking"

database_schema:
  - table: "patient_status_changes"
    purpose: "Audit trail for all status modifications"
    fields: "patient_id, old_status, new_status, reason, user_id, timestamp"
    
  - table: "status_workflow_rules"
    purpose: "Configurable status transition rules"
    fields: "from_status, to_status, required_permissions, automation_rules"
```

### Integration Points
```yaml
dependencies:
  - dependency: "Patient Profile Management API (TASK-003)"
    type: "API"
    status: "IN_PROGRESS"
    
  - dependency: "User authentication and authorization system"
    type: "SERVICE"
    status: "AVAILABLE"
    
provides:
  - deliverable: "Patient status management APIs and UI components"
    interface: "RESTful API and React components for status management"
    consumers: "Patient profile, search, scheduling, billing systems"
```

### Code Patterns to Follow
```typescript
// Backend Implementation

// src/features/patient-status/entities/patient-status-change.entity.ts
@Entity('patient_status_changes')
export class PatientStatusChange {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column('uuid')
  patientId: string;

  @Column({ 
    type: 'enum', 
    enum: PatientStatus 
  })
  oldStatus: PatientStatus;

  @Column({ 
    type: 'enum', 
    enum: PatientStatus 
  })
  newStatus: PatientStatus;

  @Column()
  reason: string;

  @Column({ nullable: true })
  notes: string;

  @Column('uuid')
  changedBy: string;

  @Column({ type: 'timestamp', default: () => 'CURRENT_TIMESTAMP' })
  changedAt: Date;

  @Column({ nullable: true })
  automatedChange: boolean;

  @Column('json', { nullable: true })
  metadata: {
    triggerEvent?: string;
    relatedEntityId?: string;
    systemNote?: string;
  };

  @ManyToOne(() => Patient)
  @JoinColumn({ name: 'patientId' })
  patient: Patient;

  @ManyToOne(() => User)
  @JoinColumn({ name: 'changedBy' })
  user: User;
}

// src/features/patient-status/dto/change-status.dto.ts
export class ChangePatientStatusDto {
  @IsEnum(PatientStatus)
  newStatus: PatientStatus;

  @IsString()
  @MinLength(5)
  @MaxLength(500)
  reason: string;

  @IsOptional()
  @IsString()
  @MaxLength(1000)
  notes?: string;

  @IsOptional()
  @IsUUID()
  version?: string; // For optimistic locking
}

export class BulkStatusChangeDto {
  @IsArray()
  @ArrayMinSize(1)
  @ArrayMaxSize(100)
  @IsUUID('4', { each: true })
  patientIds: string[];

  @IsEnum(PatientStatus)
  newStatus: PatientStatus;

  @IsString()
  @MinLength(5)
  @MaxLength(500)
  reason: string;

  @IsOptional()
  @IsString()
  @MaxLength(1000)
  notes?: string;
}

// src/features/patient-status/services/patient-status.service.ts
@Injectable()
export class PatientStatusService {
  constructor(
    private readonly patientRepository: PatientRepository,
    private readonly statusChangeRepository: StatusChangeRepository,
    private readonly statusWorkflowService: StatusWorkflowService,
    private readonly auditService: AuditService,
    private readonly logger: Logger
  ) {}

  async changePatientStatus(
    patientId: string,
    changeDto: ChangePatientStatusDto,
    userId: string
  ): Promise<Result<Patient, ValidationError>> {
    try {
      // Get current patient with version for optimistic locking
      const patient = await this.patientRepository.findByIdWithVersion(patientId);
      if (!patient) {
        return Result.error(new ValidationError('Patient not found'));
      }

      // Validate status transition
      const transitionValidation = await this.statusWorkflowService.validateTransition(
        patient.status,
        changeDto.newStatus,
        userId
      );

      if (transitionValidation.isError) {
        return Result.error(transitionValidation.error);
      }

      // Check for version conflicts (optimistic locking)
      if (changeDto.version && changeDto.version !== patient.version) {
        return Result.error(new ValidationError(
          'Patient was modified by another user. Please refresh and try again.'
        ));
      }

      // Perform status change in transaction
      const result = await this.performStatusChange(
        patient,
        changeDto,
        userId
      );

      if (result.isError) {
        return result;
      }

      // Trigger post-status-change workflows
      await this.statusWorkflowService.handleStatusChangeEffects(
        patientId,
        patient.status,
        changeDto.newStatus
      );

      this.logger.log(`Patient status changed: ${patientId} ${patient.status} -> ${changeDto.newStatus}`);
      return result;

    } catch (error) {
      this.logger.error('Status change failed', error);
      return Result.error(new ValidationError('Failed to change patient status'));
    }
  }

  async bulkChangeStatus(
    bulkChangeDto: BulkStatusChangeDto,
    userId: string
  ): Promise<Result<BulkStatusChangeResult, ValidationError>> {
    try {
      const results: BulkStatusChangeResult = {
        successful: [],
        failed: [],
        totalProcessed: 0
      };

      // Process in batches to avoid overwhelming the database
      const batchSize = 10;
      for (let i = 0; i < bulkChangeDto.patientIds.length; i += batchSize) {
        const batch = bulkChangeDto.patientIds.slice(i, i + batchSize);
        
        const batchPromises = batch.map(async (patientId) => {
          const changeResult = await this.changePatientStatus(
            patientId,
            {
              newStatus: bulkChangeDto.newStatus,
              reason: bulkChangeDto.reason,
              notes: bulkChangeDto.notes
            },
            userId
          );

          if (changeResult.isError) {
            results.failed.push({
              patientId,
              error: changeResult.error.message
            });
          } else {
            results.successful.push(patientId);
          }

          results.totalProcessed++;
        });

        await Promise.all(batchPromises);
      }

      // Log bulk operation
      await this.auditService.logBulkStatusChange(
        bulkChangeDto,
        results,
        userId
      );

      return Result.success(results);

    } catch (error) {
      this.logger.error('Bulk status change failed', error);
      return Result.error(new ValidationError('Bulk status change failed'));
    }
  }

  async getPatientStatusHistory(
    patientId: string,
    options?: { limit?: number; fromDate?: Date; toDate?: Date }
  ): Promise<PatientStatusChange[]> {
    const queryBuilder = this.statusChangeRepository
      .createQueryBuilder('change')
      .leftJoinAndSelect('change.user', 'user')
      .where('change.patientId = :patientId', { patientId })
      .orderBy('change.changedAt', 'DESC');

    if (options?.limit) {
      queryBuilder.limit(options.limit);
    }

    if (options?.fromDate) {
      queryBuilder.andWhere('change.changedAt >= :fromDate', { 
        fromDate: options.fromDate 
      });
    }

    if (options?.toDate) {
      queryBuilder.andWhere('change.changedAt <= :toDate', { 
        toDate: options.toDate 
      });
    }

    return await queryBuilder.getMany();
  }

  private async performStatusChange(
    patient: Patient,
    changeDto: ChangePatientStatusDto,
    userId: string
  ): Promise<Result<Patient, ValidationError>> {
    const queryRunner = this.patientRepository.dataSource.createQueryRunner();
    await queryRunner.connect();
    await queryRunner.startTransaction();

    try {
      const oldStatus = patient.status;
      const newVersion = uuidv4();

      // Update patient status
      await queryRunner.manager.update(Patient, patient.id, {
        status: changeDto.newStatus,
        version: newVersion,
        updatedAt: new Date()
      });

      // Create status change record
      const statusChange = queryRunner.manager.create(PatientStatusChange, {
        patientId: patient.id,
        oldStatus,
        newStatus: changeDto.newStatus,
        reason: changeDto.reason,
        notes: changeDto.notes,
        changedBy: userId,
        automatedChange: false
      });

      await queryRunner.manager.save(statusChange);

      await queryRunner.commitTransaction();

      // Return updated patient
      const updatedPatient = { ...patient, status: changeDto.newStatus, version: newVersion };
      return Result.success(updatedPatient);

    } catch (error) {
      await queryRunner.rollbackTransaction();
      throw error;
    } finally {
      await queryRunner.release();
    }
  }
}

// src/features/patient-status/services/status-workflow.service.ts
@Injectable()
export class StatusWorkflowService {
  constructor(
    private readonly authService: AuthService,
    private readonly appointmentService: AppointmentService,
    private readonly logger: Logger
  ) {}

  async validateTransition(
    fromStatus: PatientStatus,
    toStatus: PatientStatus,
    userId: string
  ): Promise<Result<void, ValidationError>> {
    // Check if transition is valid according to business rules
    const allowedTransitions = this.getAllowedTransitions(fromStatus);
    
    if (!allowedTransitions.includes(toStatus)) {
      return Result.error(new ValidationError(
        `Invalid status transition from ${fromStatus} to ${toStatus}`
      ));
    }

    // Check user permissions for sensitive transitions
    if (this.isSensitiveTransition(fromStatus, toStatus)) {
      const hasPermission = await this.authService.hasPermission(
        userId,
        'patient:status:sensitive'
      );

      if (!hasPermission) {
        return Result.error(new ValidationError(
          'Insufficient permissions for this status change'
        ));
      }
    }

    // Additional validation for specific transitions
    if (toStatus === PatientStatus.DECEASED) {
      return await this.validateDeceasedTransition(fromStatus);
    }

    if (toStatus === PatientStatus.MERGED) {
      return await this.validateMergedTransition(fromStatus);
    }

    return Result.success();
  }

  async handleStatusChangeEffects(
    patientId: string,
    oldStatus: PatientStatus,
    newStatus: PatientStatus
  ): Promise<void> {
    try {
      // Handle deceased status effects
      if (newStatus === PatientStatus.DECEASED) {
        await this.handleDeceasedStatusEffects(patientId);
      }

      // Handle merged status effects
      if (newStatus === PatientStatus.MERGED) {
        await this.handleMergedStatusEffects(patientId);
      }

      // Handle reactivation effects
      if (oldStatus === PatientStatus.INACTIVE && newStatus === PatientStatus.ACTIVE) {
        await this.handleReactivationEffects(patientId);
      }

    } catch (error) {
      this.logger.error('Status change effects processing failed', error, {
        patientId,
        oldStatus,
        newStatus
      });
      // Don't throw error to avoid rolling back the main status change
    }
  }

  private getAllowedTransitions(fromStatus: PatientStatus): PatientStatus[] {
    const transitions = {
      [PatientStatus.ACTIVE]: [
        PatientStatus.INACTIVE,
        PatientStatus.DECEASED,
        PatientStatus.MERGED
      ],
      [PatientStatus.INACTIVE]: [
        PatientStatus.ACTIVE
      ],
      [PatientStatus.DECEASED]: [], // No transitions allowed from deceased
      [PatientStatus.MERGED]: []    // No transitions allowed from merged
    };

    return transitions[fromStatus] || [];
  }

  private isSensitiveTransition(fromStatus: PatientStatus, toStatus: PatientStatus): boolean {
    const sensitiveTransitions = [
      { from: PatientStatus.ACTIVE, to: PatientStatus.DECEASED },
      { from: PatientStatus.INACTIVE, to: PatientStatus.DECEASED },
      { from: PatientStatus.ACTIVE, to: PatientStatus.MERGED },
      { from: PatientStatus.INACTIVE, to: PatientStatus.MERGED }
    ];

    return sensitiveTransitions.some(
      transition => transition.from === fromStatus && transition.to === toStatus
    );
  }

  private async validateDeceasedTransition(fromStatus: PatientStatus): Promise<Result<void, ValidationError>> {
    // Additional validation for deceased status
    // Could include checks for death certificate, authorization, etc.
    return Result.success();
  }

  private async validateMergedTransition(fromStatus: PatientStatus): Promise<Result<void, ValidationError>> {
    // Additional validation for merged status
    // Could include checks for merge documentation, duplicate resolution, etc.
    return Result.success();
  }

  private async handleDeceasedStatusEffects(patientId: string): Promise<void> {
    // Cancel future appointments
    await this.appointmentService.cancelFutureAppointments(
      patientId,
      'Patient deceased'
    );

    // Other deceased status effects
    this.logger.log(`Processed deceased status effects for patient: ${patientId}`);
  }

  private async handleMergedStatusEffects(patientId: string): Promise<void> {
    // Cancel future appointments and redirect to primary patient
    await this.appointmentService.cancelFutureAppointments(
      patientId,
      'Patient record merged'
    );

    this.logger.log(`Processed merged status effects for patient: ${patientId}`);
  }

  private async handleReactivationEffects(patientId: string): Promise<void> {
    // Log reactivation for audit purposes
    this.logger.log(`Patient reactivated: ${patientId}`);
  }
}

// Frontend Implementation

// src/features/patients/components/PatientStatusManager.tsx
import React, { useState } from 'react';
import { useTranslation } from 'react-i18next';
import { AlertTriangle, Clock, Check, X } from 'lucide-react';
import { Button } from '@/components/ui/button';
import { Badge } from '@/components/ui/badge';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogTrigger } from '@/components/ui/dialog';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { Textarea } from '@/components/ui/textarea';
import { Label } from '@/components/ui/label';
import { toast } from 'sonner';
import { useChangePatientStatus } from '../hooks/useChangePatientStatus';
import { getPatientStatusColor, getValidStatusTransitions } from '../utils/patient-status.utils';
import type { Patient, PatientStatus } from '../types/patient.types';

interface PatientStatusManagerProps {
  patient: Patient;
  disabled?: boolean;
  onStatusChanged?: (newStatus: PatientStatus) => void;
}

export const PatientStatusManager: React.FC<PatientStatusManagerProps> = ({
  patient,
  disabled = false,
  onStatusChanged
}) => {
  const { t } = useTranslation('patients');
  const [showDialog, setShowDialog] = useState(false);
  const [selectedStatus, setSelectedStatus] = useState<PatientStatus | null>(null);
  const [reason, setReason] = useState('');
  const [notes, setNotes] = useState('');

  const changeStatusMutation = useChangePatientStatus({
    onSuccess: (updatedPatient) => {
      toast.success(t('status.change_success'));
      onStatusChanged?.(updatedPatient.status);
      setShowDialog(false);
      resetForm();
    },
    onError: (error) => {
      toast.error(error.message || t('status.change_error'));
    }
  });

  const validTransitions = getValidStatusTransitions(patient.status);
  const currentStatusColor = getPatientStatusColor(patient.status);

  const resetForm = () => {
    setSelectedStatus(null);
    setReason('');
    setNotes('');
  };

  const handleStatusChange = async () => {
    if (!selectedStatus || !reason.trim()) {
      toast.error(t('status.validation.required_fields'));
      return;
    }

    await changeStatusMutation.mutateAsync({
      patientId: patient.id,
      newStatus: selectedStatus,
      reason: reason.trim(),
      notes: notes.trim() || undefined,
      version: patient.version
    });
  };

  const getStatusIcon = (status: PatientStatus) => {
    switch (status) {
      case 'active': return <Check className="h-4 w-4" />;
      case 'inactive': return <Clock className="h-4 w-4" />;
      case 'deceased': return <X className="h-4 w-4" />;
      case 'merged': return <AlertTriangle className="h-4 w-4" />;
      default: return null;
    }
  };

  return (
    <Card>
      <CardHeader className="pb-4">
        <CardTitle className="text-base">{t('status.current_status')}</CardTitle>
      </CardHeader>
      <CardContent className="space-y-4">
        <div className="flex items-center justify-between">
          <div className="flex items-center gap-2">
            <Badge variant={currentStatusColor} className="flex items-center gap-1">
              {getStatusIcon(patient.status)}
              {t(`patient_status.${patient.status}`)}
            </Badge>
          </div>
          
          {!disabled && validTransitions.length > 0 && (
            <Dialog open={showDialog} onOpenChange={setShowDialog}>
              <DialogTrigger asChild>
                <Button size="sm" variant="outline">
                  {t('status.change_status')}
                </Button>
              </DialogTrigger>
              <DialogContent className="max-w-md">
                <DialogHeader>
                  <DialogTitle>{t('status.change_patient_status')}</DialogTitle>
                </DialogHeader>
                
                <div className="space-y-4">
                  <div className="space-y-2">
                    <Label>{t('status.current_status')}</Label>
                    <Badge variant={currentStatusColor}>
                      {t(`patient_status.${patient.status}`)}
                    </Badge>
                  </div>

                  <div className="space-y-2">
                    <Label>{t('status.new_status')}</Label>
                    <Select
                      value={selectedStatus || ''}
                      onValueChange={(value) => setSelectedStatus(value as PatientStatus)}
                    >
                      <SelectTrigger>
                        <SelectValue placeholder={t('status.select_status')} />
                      </SelectTrigger>
                      <SelectContent>
                        {validTransitions.map((status) => (
                          <SelectItem key={status} value={status}>
                            <div className="flex items-center gap-2">
                              {getStatusIcon(status)}
                              {t(`patient_status.${status}`)}
                            </div>
                          </SelectItem>
                        ))}
                      </SelectContent>
                    </Select>
                  </div>

                  <div className="space-y-2">
                    <Label className="required">{t('status.reason')}</Label>
                    <Textarea
                      value={reason}
                      onChange={(e) => setReason(e.target.value)}
                      placeholder={t('status.reason_placeholder')}
                      rows={3}
                      required
                    />
                  </div>

                  <div className="space-y-2">
                    <Label>{t('status.notes')}</Label>
                    <Textarea
                      value={notes}
                      onChange={(e) => setNotes(e.target.value)}
                      placeholder={t('status.notes_placeholder')}
                      rows={2}
                    />
                  </div>

                  {selectedStatus && (
                    <div className="p-3 bg-muted rounded-lg">
                      <p className="text-sm text-muted-foreground">
                        {t(`status.warnings.${selectedStatus}`, { 
                          returnObjects: false,
                          defaultValue: ''
                        })}
                      </p>
                    </div>
                  )}

                  <div className="flex justify-end gap-2 pt-4">
                    <Button
                      variant="outline"
                      onClick={() => {
                        setShowDialog(false);
                        resetForm();
                      }}
                      disabled={changeStatusMutation.isPending}
                    >
                      {t('common.cancel')}
                    </Button>
                    <Button
                      onClick={handleStatusChange}
                      disabled={!selectedStatus || !reason.trim() || changeStatusMutation.isPending}
                      loading={changeStatusMutation.isPending}
                    >
                      {t('status.confirm_change')}
                    </Button>
                  </div>
                </div>
              </DialogContent>
            </Dialog>
          )}
        </div>

        {validTransitions.length === 0 && !disabled && (
          <p className="text-sm text-muted-foreground">
            {t('status.no_transitions_available')}
          </p>
        )}
      </CardContent>
    </Card>
  );
};

// src/features/patients/utils/patient-status.utils.ts
import { PatientStatus } from '../types/patient.types';

export function getPatientStatusColor(status: PatientStatus): string {
  switch (status) {
    case 'active': return 'default';
    case 'inactive': return 'secondary';
    case 'deceased': return 'destructive';
    case 'merged': return 'outline';
    default: return 'secondary';
  }
}

export function getValidStatusTransitions(currentStatus: PatientStatus): PatientStatus[] {
  const transitions = {
    'active': ['inactive', 'deceased', 'merged'],
    'inactive': ['active'],
    'deceased': [],
    'merged': []
  };

  return transitions[currentStatus] || [];
}

export function isStatusTransitionValid(
  fromStatus: PatientStatus, 
  toStatus: PatientStatus
): boolean {
  const validTransitions = getValidStatusTransitions(fromStatus);
  return validTransitions.includes(toStatus);
}

export function getStatusTransitionWarning(
  fromStatus: PatientStatus,
  toStatus: PatientStatus
): string | null {
  if (toStatus === 'deceased') {
    return 'This will mark the patient as deceased and cancel all future appointments.';
  }
  
  if (toStatus === 'merged') {
    return 'This will mark the patient as merged and redirect all references to the primary patient.';
  }
  
  if (fromStatus === 'inactive' && toStatus === 'active') {
    return 'This will reactivate the patient and make them available for scheduling.';
  }
  
  return null;
}
```

## Testing Requirements

### Test Coverage
```yaml
unit_tests:
  - focus: "Status transition validation and workflow logic"
  - coverage_target: "90%"
  - key_scenarios: [
    "Valid status transition validation",
    "Invalid status transition blocking",
    "Permission-based status change authorization",
    "Status change audit trail creation",
    "Bulk status change processing"
  ]
  
integration_tests:
  - focus: "Complete status management workflow"
  - test_environment: "Test database with patient fixtures and status scenarios"
  - key_flows: [
    "End-to-end status change workflow",
    "Status-dependent system behavior",
    "Status change effects on appointments and billing",
    "Bulk status management operations"
  ]
```

### Test Scenarios
```yaml
happy_path:
  - scenario: "Change active patient to inactive status"
    steps: [
      "Select patient with active status",
      "Choose inactive status with valid reason",
      "Confirm status change",
      "Verify status updated and audit trail created"
    ]
    expected_result: "Status changed successfully with audit record"
    
error_cases:
  - scenario: "Invalid status transition attempt"
    trigger: "Attempt to change deceased patient to active status"
    expected_behavior: "Transition blocked with clear error message"
    
  - scenario: "Insufficient permissions for sensitive transition"
    trigger: "User without sensitive permissions tries to mark patient as deceased"
    expected_behavior: "Permission error with appropriate message"
    
edge_cases:
  - scenario: "Concurrent status changes on same patient"
    conditions: "Two users attempt status changes simultaneously"
    expected_behavior: "Optimistic locking prevents conflicts, second change fails gracefully"
```

## Context References

### From Project
```yaml
tech_stack_references:
  - "NestJS with TypeORM for status management services"
  - "React with TypeScript for status management UI"
  - "Role-based authorization for status change permissions"
  
architecture_patterns:
  - "Service-oriented pattern for status workflow management"
  - "Audit trail pattern for compliance and tracking"
  - "State machine pattern for status transitions"
  
code_standards:
  - "Comprehensive validation for status changes"
  - "Transaction safety for status updates"
  - "Proper error handling and user feedback"
```

### From Epic
```yaml
business_rules:
  - "BR-P008: Patient status must follow valid transitions"
  - "Status changes require proper authorization and audit trail"
  - "Deceased and merged patients have restricted system access"
  - "Status changes affect appointment scheduling and billing"
  
domain_model:
  - "PatientStatus enum with defined transitions"
  - "PatientStatusChange entity for audit trail"
  - "Status-dependent business logic throughout system"
  
integration_points:
  - "Integrates with patient profile management"
  - "Affects appointment scheduling workflows"
  - "Impacts billing and payment processing"
```

## Acceptance Criteria

### Functional Acceptance
- [ ] Patient status transitions follow defined business rules correctly
- [ ] Invalid status transitions are blocked with clear error messages
- [ ] Status changes require appropriate authorization and reasons
- [ ] Complete audit trail tracks all status modifications
- [ ] Status-dependent system behavior works correctly across all modules
- [ ] Bulk status management supports administrative operations

### Technical Acceptance  
- [ ] Status management services follow NestJS architectural patterns
- [ ] Unit tests written and passing (>= 90% coverage)
- [ ] Integration tests cover status workflow scenarios
- [ ] Transaction safety ensures data integrity during status changes
- [ ] Performance requirements met for status operations
- [ ] Audit trail provides complete compliance tracking

### Quality Acceptance
- [ ] Code review completed and approved
- [ ] Security validation for status change authorization
- [ ] User experience testing confirms intuitive status management
- [ ] Error handling provides clear feedback for all scenarios
- [ ] Compliance testing validates audit trail completeness

---
**Last Updated:** 2025-01-16