# TASK: TASK-001 - Patient Registration Backend API

## Task Overview

### Goal
**What to Build:** Complete NestJS backend API module for patient registration including DTOs, services, controllers, and repositories with comprehensive validation, audit trail, and duplicate detection.

**Why:** Establish the foundational patient management system that enables digital patient registration for Algerian healthcare providers, replacing manual paper-based processes.

**Definition of Done:** 
- Patient registration API endpoints are fully functional and tested
- All business rules from BR-P001 to BR-P016 are implemented
- Audit trail captures all patient creation and modification events
- Duplicate detection prevents duplicate patient creation
- API supports Arabic and French patient data

### Scope
```yaml
task_type: "BACKEND"
complexity: "HIGH"
```

## Requirements

### Functional Requirements
1. **Patient Registration Endpoint**
   - Description: Create new patient records with comprehensive demographic information
   - Acceptance Criteria: 
     - Accepts all required patient fields (firstName, lastName, dateOfBirth, phone)
     - Generates unique UUID for patient ID
     - Validates all input data according to business rules
     - Returns created patient with ID or validation errors
   - Priority: HIGH

2. **Patient Retrieval Endpoint**
   - Description: Get patient details by ID with full profile information
   - Acceptance Criteria:
     - Returns complete patient profile including contact info and emergency contacts
     - Handles non-existent patient IDs gracefully
     - Respects user permissions and clinic boundaries
   - Priority: HIGH

3. **Input Validation System**
   - Description: Comprehensive validation for all patient data fields
   - Acceptance Criteria:
     - Validates required fields (firstName, lastName, dateOfBirth, phone)
     - Ensures email uniqueness across system
     - Validates date of birth is not in future
     - Validates phone number format for Algeria
   - Priority: HIGH

4. **Audit Trail Implementation**
   - Description: Track all patient record creation and modification events
   - Acceptance Criteria:
     - Records user ID, timestamp, and changes for all operations
     - Maintains historical record of patient data changes
     - Audit logs are searchable and reportable
   - Priority: MEDIUM

### Technical Requirements
```yaml
performance:
  - requirement: "Patient registration API response time"
    target: "< 500ms for registration completion"
    
  - requirement: "Database query performance"
    target: "< 100ms for patient lookup queries"
    
security:
  - requirement: "Role-based access control"
    implementation: "OAuth2 scopes with patient:write/read permissions"
    
  - requirement: "Data encryption"
    implementation: "Encrypt sensitive patient data at rest and in transit"
    
compatibility:
  - requirement: "Multi-language data support"
    scope: "Support Arabic and French character sets and validation"
```

## Implementation Details

### Technical Approach
```yaml
# Backend Implementation Strategy
api_endpoints:
  - endpoint: "POST /api/patients"
    purpose: "Create new patient record"
    request_format: "CreatePatientDto with validation"
    response_format: "Created patient object or validation errors"
    
  - endpoint: "GET /api/patients/{patientId}"
    purpose: "Retrieve patient by ID"
    request_format: "UUID path parameter"
    response_format: "Complete patient profile object"
    
data_operations:
  - operation: "CREATE patient"
    entity: "Patient"
    business_logic: "Generate UUID, validate data, check duplicates, create audit record"
    
  - operation: "READ patient"
    entity: "Patient" 
    business_logic: "Verify permissions, load relationships, apply clinic boundary"

schema_changes:
  - change_type: "CREATE_TABLE"
    details: "patients table with all required fields and constraints"
    migration_required: true
    
  - change_type: "CREATE_TABLE" 
    details: "patient_audit_logs table for tracking changes"
    migration_required: true
```

### Integration Points
```yaml
dependencies:
  - dependency: "AWS Cognito User Management"
    type: "SERVICE"
    status: "AVAILABLE"
    
  - dependency: "PostgreSQL Database"
    type: "DATABASE"
    status: "AVAILABLE"
    
  - dependency: "TypeORM Configuration"
    type: "DATABASE"
    status: "AVAILABLE"
    
provides:
  - deliverable: "Patient CRUD API endpoints"
    interface: "RESTful HTTP API with standard response format"
    consumers: "Frontend patient registration, patient search, other patient workflows"
```

### Code Patterns to Follow
```typescript
// src/features/patients/dto/create-patient.dto.ts
import { IsString, IsEmail, IsOptional, IsNotEmpty, IsDateString, IsEnum, MinLength, MaxLength } from 'class-validator';
import { Transform } from 'class-transformer';
import { Gender, ContactMethod } from '../enums/patient.enums';

export class CreatePatientDto {
  @IsNotEmpty()
  @IsString()
  @MinLength(1)
  @MaxLength(50)
  firstName: string;

  @IsNotEmpty()
  @IsString()
  @MinLength(1) 
  @MaxLength(50)
  lastName: string;

  @IsOptional()
  @IsString()
  @MaxLength(50)
  middleName?: string;

  @IsNotEmpty()
  @IsDateString()
  @Transform(({ value }) => new Date(value))
  dateOfBirth: Date;

  @IsNotEmpty()
  @IsEnum(Gender)
  gender: Gender;

  @IsNotEmpty()
  @IsString()
  phone: string;

  @IsOptional()
  @IsEmail()
  email?: string;

  @IsOptional()
  @IsEnum(ContactMethod)
  preferredContactMethod?: ContactMethod;

  // Nested DTOs for address and emergency contacts
  @Type(() => CreateAddressDto)
  @ValidateNested()
  address: CreateAddressDto;

  @Type(() => CreateEmergencyContactDto)
  @ValidateNested()
  @IsOptional()
  emergencyContacts?: CreateEmergencyContactDto[];
}

// src/features/patients/services/patients.service.ts
@Injectable()
export class PatientsService {
  constructor(
    private readonly patientRepository: PatientRepository,
    private readonly auditService: AuditService,
    private readonly duplicateDetectionService: DuplicateDetectionService,
    private readonly logger: Logger
  ) {}

  async createPatient(
    createPatientDto: CreatePatientDto,
    userId: string
  ): Promise<Result<Patient, ValidationError>> {
    try {
      // Business rule validation
      const validation = await this.validatePatientData(createPatientDto);
      if (validation.isError) {
        return Result.error(validation.error);
      }

      // Duplicate detection (BR-P016)
      const duplicateCheck = await this.duplicateDetectionService.checkForDuplicates(
        createPatientDto.firstName,
        createPatientDto.lastName,
        createPatientDto.dateOfBirth,
        createPatientDto.phone
      );
      
      if (duplicateCheck.hasDuplicates) {
        return Result.error(new ValidationError('Potential duplicate patient found', duplicateCheck.matches));
      }

      // Generate unique patient ID (BR-P001)
      const patientId = uuidv4();
      
      // Create patient with default active status (BR-P006)
      const patientData = {
        ...createPatientDto,
        id: patientId,
        status: PatientStatus.ACTIVE,
        clinicId: await this.getCurrentClinicId(userId)
      };

      const patient = await this.patientRepository.create(patientData);
      
      // Create audit trail (BR-P007)
      await this.auditService.logPatientCreation(patient.id, userId, patientData);
      
      this.logger.log(`Patient created successfully: ${patient.id}`);
      return Result.success(patient);
      
    } catch (error) {
      this.logger.error('Patient creation failed', error);
      return Result.error(new ValidationError('Failed to create patient'));
    }
  }

  async getPatientById(patientId: string, userId: string): Promise<Result<Patient, NotFoundError>> {
    try {
      // Verify permissions and clinic boundaries (BR-P021)
      const hasAccess = await this.verifyPatientAccess(patientId, userId);
      if (!hasAccess) {
        return Result.error(new NotFoundError('Patient not found'));
      }

      const patient = await this.patientRepository.findByIdWithRelations(patientId);
      if (!patient) {
        return Result.error(new NotFoundError('Patient not found'));
      }

      return Result.success(patient);
    } catch (error) {
      this.logger.error(`Failed to retrieve patient: ${patientId}`, error);
      return Result.error(new NotFoundError('Patient not found'));
    }
  }

  private async validatePatientData(data: CreatePatientDto): Promise<Result<void, ValidationError>> {
    // BR-P004: Date of birth cannot be in future
    if (data.dateOfBirth > new Date()) {
      return Result.error(new ValidationError('Date of birth cannot be in the future'));
    }

    // BR-P003: Email uniqueness check if provided
    if (data.email) {
      const existingPatient = await this.patientRepository.findByEmail(data.email);
      if (existingPatient) {
        return Result.error(new ValidationError('Email address already exists'));
      }
    }

    return Result.success();
  }
}

// src/features/patients/controllers/patients.controller.ts
@Controller('patients')
@UseGuards(AuthGuard, RolesGuard)
export class PatientsController {
  constructor(private readonly patientsService: PatientsService) {}

  @Post()
  @Roles('patient:write')
  @ApiOperation({ summary: 'Create new patient' })
  @ApiResponse({ status: 201, description: 'Patient created successfully', type: Patient })
  @ApiResponse({ status: 400, description: 'Validation error' })
  async createPatient(
    @Body() createPatientDto: CreatePatientDto,
    @CurrentUser() user: User
  ): Promise<Patient> {
    const result = await this.patientsService.createPatient(createPatientDto, user.id);
    
    if (result.isError) {
      throw new BadRequestException(result.error.message);
    }
    
    return result.data;
  }

  @Get(':patientId')
  @Roles('patient:read')
  @ApiOperation({ summary: 'Get patient by ID' })
  @ApiResponse({ status: 200, description: 'Patient found', type: Patient })
  @ApiResponse({ status: 404, description: 'Patient not found' })
  async getPatient(
    @Param('patientId', ParseUUIDPipe) patientId: string,
    @CurrentUser() user: User
  ): Promise<Patient> {
    const result = await this.patientsService.getPatientById(patientId, user.id);
    
    if (result.isError) {
      throw new NotFoundException('Patient not found');
    }
    
    return result.data;
  }
}
```

## Testing Requirements

### Test Coverage
```yaml
unit_tests:
  - focus: "Service layer business logic and validation"
  - coverage_target: "90%"
  - key_scenarios: [
    "Patient creation with valid data",
    "Patient creation with invalid data", 
    "Duplicate detection scenarios",
    "Business rule validation",
    "Audit trail creation"
  ]
  
integration_tests:
  - focus: "API endpoints and database operations"
  - test_environment: "Test database with sample data"
  - key_flows: [
    "End-to-end patient registration flow",
    "Patient retrieval with permissions",
    "Validation error handling",
    "Database transaction rollback scenarios"
  ]
```

### Test Scenarios
```yaml
happy_path:
  - scenario: "Create patient with complete valid data"
    steps: [
      "Submit POST /api/patients with valid CreatePatientDto",
      "Verify patient is created with unique ID", 
      "Verify audit trail is created",
      "Verify response contains created patient data"
    ]
    expected_result: "201 Created with patient object"
    
error_cases:
  - scenario: "Create patient with duplicate information"
    trigger: "Submit patient data matching existing patient name, DOB, and phone"
    expected_behavior: "400 Bad Request with duplicate detection message"
    
  - scenario: "Create patient with future date of birth"
    trigger: "Submit patient with dateOfBirth in future"
    expected_behavior: "400 Bad Request with validation error"
    
edge_cases:
  - scenario: "Create patient with Arabic name characters"
    conditions: "Patient name contains Arabic characters"
    expected_behavior: "Patient created successfully with proper character encoding"
```

## Context References

### From Project
```yaml
tech_stack_references:
  - "TypeScript with NestJS for backend development"
  - "PostgreSQL with TypeORM for data persistence"
  - "class-validator for input validation"
  - "AWS Cognito for authentication and authorization"
  
architecture_patterns:
  - "Feature-based modular architecture"
  - "Repository pattern for data access"
  - "Result pattern for error handling"
  - "Domain-driven design principles"
  
code_standards:
  - "PascalCase for classes, camelCase for variables/functions"
  - "Comprehensive JSDoc documentation"
  - "Consistent error handling and logging"
```

### From Epic
```yaml
business_rules:
  - "BR-P001: Unique patient ID generation"
  - "BR-P002: Mandatory fields validation"
  - "BR-P003: Email uniqueness constraint"
  - "BR-P004: Date of birth validation"
  - "BR-P006: Default active status"
  - "BR-P007: Audit trail requirements"
  - "BR-P016: Duplicate prevention"
  
domain_model:
  - "Patient entity with all required attributes"
  - "ContactInfo and Address entities"
  - "EmergencyContact entity relationships"
  - "PatientStatus enum and transitions"
  
integration_points:
  - "Provides patient CRUD APIs for frontend consumption"
  - "Integrates with audit service for change tracking"
  - "Uses duplicate detection service for data quality"
```

## Acceptance Criteria

### Functional Acceptance
- [ ] POST /api/patients endpoint creates patients with all required validation
- [ ] GET /api/patients/{id} endpoint retrieves patient with proper access control
- [ ] All business rules BR-P001 through BR-P016 are implemented and enforced
- [ ] Duplicate detection prevents creation of duplicate patients
- [ ] Audit trail records all patient creation and modification events
- [ ] API supports Arabic and French character data properly

### Technical Acceptance  
- [ ] Code follows NestJS and project architectural patterns
- [ ] Unit tests written and passing (>= 90% coverage)
- [ ] Integration tests cover all API endpoints
- [ ] Database migrations create proper schema with constraints
- [ ] Error handling provides clear, actionable error messages
- [ ] API documentation is complete with Swagger/OpenAPI

### Quality Acceptance
- [ ] Code review completed and approved
- [ ] Security scan passes with no critical issues
- [ ] Performance requirements met (< 500ms response time)
- [ ] Database queries optimized (< 100ms execution time)
- [ ] Logging and monitoring integration working properly

---
**Last Updated:** 2025-01-16