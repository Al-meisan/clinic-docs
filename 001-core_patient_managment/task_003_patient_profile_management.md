# TASK: TASK-003 - Patient Profile Management API

## Task Overview

### Goal

**What to Build:** Complete patient profile management API with update capabilities, comprehensive audit trail, data
validation, and support for partial updates while maintaining data integrity.

**Why:** Enable clinic staff to maintain accurate, up-to-date patient information essential for effective healthcare
delivery and communication.

**Definition of Done:**

- Patient profile update API endpoints are fully functional
- Audit trail tracks all changes with detailed change history
- Partial updates support selective field modifications
- Data validation ensures data integrity and business rule compliance
- Emergency contact and address management integrated

### Scope

```yaml
task_type: "BACKEND"
complexity: "MEDIUM"
```

## Requirements

### Functional Requirements

1. **Patient Profile Update Endpoint**
    - Description: Update patient demographic and contact information with validation
    - Acceptance Criteria:
        - Supports partial updates (only modified fields)
        - Validates all input data according to business rules
        - Maintains data integrity during concurrent updates
        - Returns updated patient profile or validation errors
    - Priority: HIGH

2. **Emergency Contact Management**
    - Description: Add, update, and remove emergency contacts for patients
    - Acceptance Criteria:
        - Support multiple emergency contacts per patient
        - Designate primary emergency contact
        - Validate emergency contact information
        - Maintain relationship information and contact details
    - Priority: HIGH

3. **Address Information Management**
    - Description: Update patient address information with validation
    - Acceptance Criteria:
        - Support Algerian address format validation
        - Handle address changes with historical tracking
        - Validate required address fields
        - Support optional geocoding for mapping
    - Priority: MEDIUM

4. **Change History and Audit Trail**
    - Description: Comprehensive tracking of all patient profile modifications
    - Acceptance Criteria:
        - Record all field changes with before/after values
        - Track user ID and timestamp for each change
        - Provide change history retrieval endpoint
        - Support audit reporting and compliance needs
    - Priority: HIGH

5. **Patient Status Management**
    - Description: Handle patient status transitions with proper workflow
    - Acceptance Criteria:
        - Enforce valid status transitions (BR-P008)
        - Require reason and authorization for status changes
        - Prevent invalid status changes
        - Maintain status change history
    - Priority: MEDIUM

### Technical Requirements

```yaml
security:
  - requirement: "Data validation and sanitization"
    implementation: "Comprehensive input validation with business rule enforcement"

  - requirement: "Change authorization"
    implementation: "Verify user permissions for all profile modifications"

compatibility:
  - requirement: "Optimistic locking"
    scope: "Prevent data loss during concurrent updates"
```

## Implementation Details

### Technical Approach

```yaml
# Profile Management Strategy
api_endpoints:
  - endpoint: "PUT /api/patients/{patientId}"
    purpose: "Update patient profile information"
    request_format: "UpdatePatientDto with partial update support"
    response_format: "Updated patient profile or validation errors"

  - endpoint: "GET /api/patients/{patientId}/history"
    purpose: "Get patient change history"
    request_format: "Patient ID with optional date filtering"
    response_format: "Array of change history records"

  - endpoint: "PUT /api/patients/{patientId}/status"
    purpose: "Update patient status with workflow validation"
    request_format: "Status change request with reason"
    response_format: "Updated patient with new status"

data_operations:
  - operation: "UPDATE patient"
    entity: "Patient"
    business_logic: "Validate changes, check permissions, create audit record, update fields"

  - operation: "TRACK changes"
    entity: "PatientAuditLog"
    business_logic: "Record before/after values, timestamp, user ID"

database_optimization:
  - optimization_type: "ADD_COLUMN"
    details: "version column for optimistic locking"
    migration_required: true

  - optimization_type: "CREATE_INDEX"
    details: "Index on audit log patient_id and timestamp for history queries"
    migration_required: true
```

### Integration Points

```yaml
dependencies:
  - dependency: "Patient Registration API (TASK-001)"
    type: "API"
    status: "IN_PROGRESS"

  - dependency: "Patient Search Implementation (TASK-002)"
    type: "API"
    status: "IN_PROGRESS"

provides:
  - deliverable: "Patient update and management APIs"
    interface: "RESTful update endpoints with audit capabilities"
    consumers: "Frontend patient forms, check-in processes, clinical workflows"
```

### Code Patterns to Follow

```typescript
// src/features/patients/dto/update-patient.dto.ts
import {IsOptional, IsString, IsEmail, IsEnum, MinLength, MaxLength, ValidateNested, IsUUID} from 'class-validator';
import {Type} from 'class-transformer';
import {Gender, ContactMethod, PatientStatus} from '../enums/patient.enums';

export class UpdatePatientDto {
    @IsOptional()
    @IsString()
    @MinLength(1)
    @MaxLength(50)
    firstName?: string;

    @IsOptional()
    @IsString()
    @MinLength(1)
    @MaxLength(50)
    lastName?: string;

    @IsOptional()
    @IsString()
    @MaxLength(50)
    middleName?: string;

    @IsOptional()
    @IsString()
    phone?: string;

    @IsOptional()
    @IsEmail()
    email?: string;

    @IsOptional()
    @IsEnum(ContactMethod)
    preferredContactMethod?: ContactMethod;

    @IsOptional()
    @IsString()
    notes?: string;

    @IsOptional()
    @Type(() => UpdateAddressDto)
    @ValidateNested()
    address?: UpdateAddressDto;

    @IsOptional()
    @Type(() => UpdateEmergencyContactDto)
    @ValidateNested({each: true})
    emergencyContacts?: UpdateEmergencyContactDto[];

    // Version for optimistic locking
    @IsOptional()
    @IsUUID()
    version?: string;
}

export class UpdatePatientStatusDto {
    @IsEnum(PatientStatus)
    status: PatientStatus;

    @IsString()
    @MinLength(1)
    reason: string;

    @IsOptional()
    @IsString()
    notes?: string;

    @IsUUID()
    version: string;
}

// src/features/patients/entities/patient-audit-log.entity.ts
@Entity('patient_audit_logs')
export class PatientAuditLog {
    @PrimaryGeneratedColumn('uuid')
    id: string;

    @Column('uuid')
    patientId: string;

    @Column('uuid')
    userId: string;

    @Column()
    action: string; // 'UPDATE', 'CREATE', 'STATUS_CHANGE'

    @Column('json', {nullable: true})
    oldValues: Record<string, any>;

    @Column('json', {nullable: true})
    newValues: Record<string, any>;

    @Column('json', {nullable: true})
    changedFields: string[];

    @Column({nullable: true})
    reason: string;

    @Column({type: 'timestamp', default: () => 'CURRENT_TIMESTAMP'})
    createdAt: Date;

    @ManyToOne(() => Patient)
    @JoinColumn({name: 'patientId'})
    patient: Patient;

    @ManyToOne(() => User)
    @JoinColumn({name: 'userId'})
    user: User;
}

// src/features/patients/services/patient-profile.service.ts
@Injectable()
export class PatientProfileService {
    constructor(
        private readonly patientRepository: PatientRepository,
        private readonly auditService: PatientAuditService,
        private readonly logger: Logger
    ) {
    }

    async updatePatient(
        patientId: string,
        updateDto: UpdatePatientDto,
        userId: string
    ): Promise<Result<Patient, ValidationError | ConflictError>> {
        try {
            // Get current patient for optimistic locking and audit trail
            const currentPatient = await this.patientRepository.findByIdWithVersion(patientId);
            if (!currentPatient) {
                return Result.error(new NotFoundError('Patient not found'));
            }

            // Verify clinic access
            await this.verifyClinicAccess(currentPatient.clinicId, userId);

            // Optimistic locking check
            if (updateDto.version && updateDto.version !== currentPatient.version) {
                return Result.error(new ConflictError('Patient was modified by another user. Please refresh and try again.'));
            }

            // Validate business rules
            const validation = await this.validateUpdateData(updateDto, currentPatient);
            if (validation.isError) {
                return Result.error(validation.error);
            }

            // Prepare update data
            const updateData = this.prepareUpdateData(updateDto, currentPatient);
            const changedFields = this.getChangedFields(updateData, currentPatient);

            if (changedFields.length === 0) {
                return Result.success(currentPatient); // No changes to apply
            }

            // Perform update with new version
            const newVersion = uuidv4();
            const updatedPatient = await this.patientRepository.updateWithVersion(
                patientId,
                updateData,
                currentPatient.version,
                newVersion
            );

            // Create audit trail
            await this.auditService.logPatientUpdate(
                patientId,
                userId,
                currentPatient,
                updatedPatient,
                changedFields,
                'Profile update'
            );

            this.logger.log(`Patient profile updated: ${patientId}`, {
                changedFields,
                userId
            });

            return Result.success(updatedPatient);

        } catch (error) {
            this.logger.error('Patient update failed', error);
            return Result.error(new ValidationError('Failed to update patient profile'));
        }
    }

    async updatePatientStatus(
        patientId: string,
        statusDto: UpdatePatientStatusDto,
        userId: string
    ): Promise<Result<Patient, ValidationError | ConflictError>> {
        try {
            const currentPatient = await this.patientRepository.findByIdWithVersion(patientId);
            if (!currentPatient) {
                return Result.error(new NotFoundError('Patient not found'));
            }

            // Verify status transition is valid (BR-P008)
            const isValidTransition = this.isValidStatusTransition(
                currentPatient.status,
                statusDto.status
            );

            if (!isValidTransition) {
                return Result.error(new ValidationError(
                    `Invalid status transition from ${currentPatient.status} to ${statusDto.status}`
                ));
            }

            // Optimistic locking check
            if (statusDto.version !== currentPatient.version) {
                return Result.error(new ConflictError('Patient was modified by another user'));
            }

            // Update status with new version
            const newVersion = uuidv4();
            const updatedPatient = await this.patientRepository.updateWithVersion(
                patientId,
                {status: statusDto.status},
                currentPatient.version,
                newVersion
            );

            // Create audit trail for status change
            await this.auditService.logStatusChange(
                patientId,
                userId,
                currentPatient.status,
                statusDto.status,
                statusDto.reason,
                statusDto.notes
            );

            return Result.success(updatedPatient);

        } catch (error) {
            this.logger.error('Patient status update failed', error);
            return Result.error(new ValidationError('Failed to update patient status'));
        }
    }

    async getPatientHistory(
        patientId: string,
        userId: string,
        options?: { fromDate?: Date; toDate?: Date; limit?: number }
    ): Promise<Result<PatientAuditLog[], ValidationError>> {
        try {
            // Verify access to patient
            const patient = await this.patientRepository.findById(patientId);
            if (!patient) {
                return Result.error(new NotFoundError('Patient not found'));
            }

            await this.verifyClinicAccess(patient.clinicId, userId);

            const history = await this.auditService.getPatientHistory(patientId, options);
            return Result.success(history);

        } catch (error) {
            this.logger.error('Failed to get patient history', error);
            return Result.error(new ValidationError('Failed to retrieve patient history'));
        }
    }

    private async validateUpdateData(
        updateDto: UpdatePatientDto,
        currentPatient: Patient
    ): Promise<Result<void, ValidationError>> {
        // Email uniqueness check (BR-P003)
        if (updateDto.email && updateDto.email !== currentPatient.contactInfo?.email) {
            const existingPatient = await this.patientRepository.findByEmail(updateDto.email);
            if (existingPatient && existingPatient.id !== currentPatient.id) {
                return Result.error(new ValidationError('Email address already exists'));
            }
        }

        // Additional validation logic...
        return Result.success();
    }

    private isValidStatusTransition(currentStatus: PatientStatus, newStatus: PatientStatus): boolean {
        const validTransitions = {
            [PatientStatus.ACTIVE]: [PatientStatus.INACTIVE, PatientStatus.DECEASED, PatientStatus.MERGED],
            [PatientStatus.INACTIVE]: [PatientStatus.ACTIVE],
            [PatientStatus.DECEASED]: [], // No transitions allowed from deceased
            [PatientStatus.MERGED]: [] // No transitions allowed from merged
        };

        return validTransitions[currentStatus]?.includes(newStatus) || false;
    }

    private getChangedFields(updateData: any, currentPatient: Patient): string[] {
        const changedFields: string[] = [];

        Object.keys(updateData).forEach(key => {
            if (updateData[key] !== currentPatient[key]) {
                changedFields.push(key);
            }
        });

        return changedFields;
    }

    private prepareUpdateData(updateDto: UpdatePatientDto, currentPatient: Patient): Partial<Patient> {
        const updateData: Partial<Patient> = {};

        // Only include fields that are actually being updated
        Object.keys(updateDto).forEach(key => {
            if (key !== 'version' && updateDto[key] !== undefined) {
                updateData[key] = updateDto[key];
            }
        });

        updateData.updatedAt = new Date();
        return updateData;
    }
}

// src/features/patients/services/patient-audit.service.ts
@Injectable()
export class PatientAuditService {
    constructor(
        @InjectRepository(PatientAuditLog)
        private readonly auditRepository: Repository<PatientAuditLog>
    ) {
    }

    async logPatientUpdate(
        patientId: string,
        userId: string,
        oldPatient: Patient,
        newPatient: Patient,
        changedFields: string[],
        reason?: string
    ): Promise<PatientAuditLog> {
        const auditLog = this.auditRepository.create({
            patientId,
            userId,
            action: 'UPDATE',
            oldValues: this.extractFields(oldPatient, changedFields),
            newValues: this.extractFields(newPatient, changedFields),
            changedFields,
            reason
        });

        return await this.auditRepository.save(auditLog);
    }

    async logStatusChange(
        patientId: string,
        userId: string,
        oldStatus: PatientStatus,
        newStatus: PatientStatus,
        reason: string,
        notes?: string
    ): Promise<PatientAuditLog> {
        const auditLog = this.auditRepository.create({
            patientId,
            userId,
            action: 'STATUS_CHANGE',
            oldValues: {status: oldStatus},
            newValues: {status: newStatus},
            changedFields: ['status'],
            reason,
            notes
        });

        return await this.auditRepository.save(auditLog);
    }

    async getPatientHistory(
        patientId: string,
        options?: { fromDate?: Date; toDate?: Date; limit?: number }
    ): Promise<PatientAuditLog[]> {
        const queryBuilder = this.auditRepository
            .createQueryBuilder('audit')
            .leftJoinAndSelect('audit.user', 'user')
            .where('audit.patientId = :patientId', {patientId})
            .orderBy('audit.createdAt', 'DESC');

        if (options?.fromDate) {
            queryBuilder.andWhere('audit.createdAt >= :fromDate', {fromDate: options.fromDate});
        }

        if (options?.toDate) {
            queryBuilder.andWhere('audit.createdAt <= :toDate', {toDate: options.toDate});
        }

        if (options?.limit) {
            queryBuilder.limit(options.limit);
        }

        return await queryBuilder.getMany();
    }

    private extractFields(object: any, fields: string[]): Record<string, any> {
        const extracted = {};
        fields.forEach(field => {
            if (object[field] !== undefined) {
                extracted[field] = object[field];
            }
        });
        return extracted;
    }
}

// src/features/patients/controllers/patients.controller.ts (update endpoints)
@Controller('patients')
@UseGuards(AuthGuard, RolesGuard)
export class PatientsController {
    constructor(
        private readonly patientProfileService: PatientProfileService
    ) {
    }

    @Put(':patientId')
    @Roles('patient:write')
    @ApiOperation({summary: 'Update patient profile'})
    @ApiResponse({status: 200, description: 'Patient updated successfully', type: Patient})
    @ApiResponse({status: 400, description: 'Validation error'})
    @ApiResponse({status: 409, description: 'Conflict - patient was modified by another user'})
    async updatePatient(
        @Param('patientId', ParseUUIDPipe) patientId: string,
        @Body() updatePatientDto: UpdatePatientDto,
        @CurrentUser() user: User
    ): Promise<Patient> {
        const result = await this.patientProfileService.updatePatient(
            patientId,
            updatePatientDto,
            user.id
        );

        if (result.isError) {
            if (result.error instanceof ConflictError) {
                throw new ConflictException(result.error.message);
            }
            throw new BadRequestException(result.error.message);
        }

        return result.data;
    }

    @Put(':patientId/status')
    @Roles('patient:write')
    @ApiOperation({summary: 'Update patient status'})
    @ApiResponse({status: 200, description: 'Patient status updated successfully'})
    @ApiResponse({status: 400, description: 'Invalid status transition'})
    async updatePatientStatus(
        @Param('patientId', ParseUUIDPipe) patientId: string,
        @Body() statusDto: UpdatePatientStatusDto,
        @CurrentUser() user: User
    ): Promise<Patient> {
        const result = await this.patientProfileService.updatePatientStatus(
            patientId,
            statusDto,
            user.id
        );

        if (result.isError) {
            if (result.error instanceof ConflictError) {
                throw new ConflictException(result.error.message);
            }
            throw new BadRequestException(result.error.message);
        }

        return result.data;
    }

    @Get(':patientId/history')
    @Roles('patient:read')
    @ApiOperation({summary: 'Get patient change history'})
    @ApiResponse({status: 200, description: 'Patient history retrieved', type: [PatientAuditLog]})
    async getPatientHistory(
        @Param('patientId', ParseUUIDPipe) patientId: string,
        @Query('fromDate') fromDate?: string,
        @Query('toDate') toDate?: string,
        @Query('limit', new DefaultValuePipe(50), ParseIntPipe) limit?: number,
        @CurrentUser() user: User
    ): Promise<PatientAuditLog[]> {
        const options = {
            fromDate: fromDate ? new Date(fromDate) : undefined,
            toDate: toDate ? new Date(toDate) : undefined,
            limit
        };

        const result = await this.patientProfileService.getPatientHistory(
            patientId,
            user.id,
            options
        );

        if (result.isError) {
            throw new BadRequestException(result.error.message);
        }

        return result.data;
    }
}
```

## Testing Requirements

### Test Coverage

```yaml
unit_tests:
  - focus: "Update validation logic and audit trail creation"
  - coverage_target: "90%"
  - key_scenarios: [
    "Successful profile updates with partial data",
    "Optimistic locking conflict handling",
    "Status transition validation",
    "Audit trail creation for all changes",
    "Business rule validation"
  ]

integration_tests:
  - focus: "API endpoints and database operations"
  - test_environment: "Test database with patient fixtures"
  - key_flows: [
    "End-to-end profile update workflow",
    "Concurrent update handling",
    "Status change workflows",
    "Audit history retrieval",
    "Error handling for various scenarios"
  ]
```

### Test Scenarios

```yaml
happy_path:
  - scenario: "Update patient profile with partial data"
    steps: [
      "Submit PUT /api/patients/{id} with partial UpdatePatientDto",
      "Verify only specified fields are updated",
      "Verify audit trail records changes",
      "Verify version number is updated"
    ]
    expected_result: "200 OK with updated patient profile"

error_cases:
  - scenario: "Update patient with stale version"
    trigger: "Submit update with outdated version number"
    expected_behavior: "409 Conflict with concurrency error message"

  - scenario: "Invalid status transition"
    trigger: "Attempt to change deceased patient status"
    expected_behavior: "400 Bad Request with transition error"

edge_cases:
  - scenario: "Concurrent updates to same patient"
    conditions: "Multiple users updating same patient simultaneously"
    expected_behavior: "First update succeeds, subsequent updates get conflict errors"
```

## Context References

### From Project

```yaml
tech_stack_references:
  - "TypeORM with optimistic locking for concurrent updates"
  - "class-validator for comprehensive input validation"
  - "NestJS guards for authorization enforcement"

architecture_patterns:
  - "Service pattern for business logic encapsulation"
  - "Repository pattern for data access"
  - "Audit service pattern for change tracking"

code_standards:
  - "Result pattern for consistent error handling"
  - "Comprehensive audit logging for compliance"
  - "Version-based optimistic locking for data integrity"
```

### From Epic

```yaml
business_rules:
  - "BR-P007: Audit trail for all modifications"
  - "BR-P008: Valid status transitions only"
  - "BR-P003: Email uniqueness enforcement"
  - "BR-P015: Contact information validation"

domain_model:
  - "UpdatePatientDto for partial updates"
  - "PatientAuditLog for change tracking"
  - "Patient entity with version control"

integration_points:
  - "Provides profile management for frontend forms"
  - "Integrates with audit service for compliance"
  - "Supports check-in process updates"
```

## Acceptance Criteria

### Functional Acceptance

- [ ] Patient profile update API handles partial updates correctly
- [ ] Optimistic locking prevents data corruption during concurrent updates
- [ ] All business rules are enforced during updates
- [ ] Complete audit trail tracks all profile changes
- [ ] Patient status management follows valid transition rules
- [ ] Emergency contact and address management works properly

### Technical Acceptance

- [ ] Code follows NestJS and project architectural patterns
- [ ] Unit tests written and passing (>= 90% coverage)
- [ ] Integration tests cover all update scenarios
- [ ] Performance requirements met (fast API response time)
- [ ] Concurrent update handling tested and working
- [ ] Database migrations create audit table and indexes

### Quality Acceptance

- [ ] Code review completed and approved
- [ ] Security validation prevents unauthorized updates
- [ ] Error handling provides clear feedback for all scenarios
- [ ] Audit trail provides complete change history
- [ ] API documentation includes update examples

---
**Last Updated:** 2025-01-16