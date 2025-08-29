# TASK: TASK-002 - Patient Search Implementation

## Task Overview

### Goal

**What to Build:** Advanced patient search system with multi-criteria search, fuzzy matching, real-time results, and
PostgreSQL full-text search optimization for fast patient retrieval.

**Why:** Enable healthcare providers to quickly locate patient records during consultations and administrative tasks,
supporting efficient clinic operations.

**Definition of Done:**

- Multi-criteria patient search
- Fuzzy matching handles typos and partial matches
- Search supports name, phone, DOB, and patient ID criteria
- Recent patients functionality for quick access
- Proper indexing and query optimization implemented

### Scope

```yaml
task_type: "BACKEND"
complexity: "MEDIUM"
```

## Requirements

### Functional Requirements

1. **Multi-Criteria Search Endpoint**
    - Description: Search patients using multiple criteria with flexible matching
    - Acceptance Criteria:
        - Supports search by name (first/last), phone, DOB, patient ID
        - Handles partial matches and case-insensitive search
        - Returns paginated results with relevance sorting
        - Filters by patient status and clinic boundaries
    - Priority: HIGH

2. **Fuzzy Matching Implementation**
    - Description: Intelligent matching that handles typos and similar names
    - Acceptance Criteria:
        - Uses PostgreSQL similarity functions for name matching
        - Prioritizes exact matches over fuzzy matches
        - Handles Arabic and French character variations
    - Priority: HIGH

3. **Recent Patients Tracking**
    - Description: Track and retrieve recently accessed patients per user
    - Acceptance Criteria:
        - Maintains list of last 10 accessed patients per user
        - Updates access time on patient retrieval
        - Provides quick access endpoint for recent patients
        - Respects user permissions and clinic boundaries
    - Priority: MEDIUM

4. **Advanced Filtering Options**
    - Description: Additional search filters for refined results
    - Acceptance Criteria:
        - Filter by patient status (active, inactive, etc.)
        - Filter by age ranges and gender
        - Date range filtering for registration dates
        - Provider-specific patient filtering
    - Priority: MEDIUM

### Technical Requirements

```yaml
security:
  - requirement: "Access control enforcement"
    implementation: "Respect clinic boundaries and user permissions"

  - requirement: "Search input sanitization"
    implementation: "Prevent SQL injection and validate all search inputs"

compatibility:
  - requirement: "Multi-language search support"
    scope: "Handle Arabic and French character searching properly"
```

## Implementation Details

### Technical Approach

```yaml
# Search Implementation Strategy
api_endpoints:
  - endpoint: "GET /api/patients"
    purpose: "Search and list patients with filtering and pagination"
    request_format: "Query parameters for search criteria, filters, pagination"
    response_format: "Paginated patient results with metadata"

  - endpoint: "GET /api/patients/recent"
    purpose: "Get recently accessed patients for current user"
    request_format: "User authentication token"
    response_format: "Array of recent patient summaries"

data_operations:
  - operation: "SEARCH patients"
    entity: "Patient"
    business_logic: "Full-text search, fuzzy matching, relevance scoring, permission filtering"

  - operation: "TRACK patient access"
    entity: "PatientAccess"
    business_logic: "Update access timestamp, maintain recent list per user"

database_optimization:
  - optimization_type: "CREATE_INDEX"
    details: "GIN index on patient search vectors for full-text search"
    migration_required: true

  - optimization_type: "CREATE_INDEX"
    details: "Composite indexes on commonly searched fields (name, phone, DOB)"
    migration_required: true
```

### Integration Points

```yaml
dependencies:
  - dependency: "Patient Registration API (TASK-001)"
    type: "API"
    status: "IN_PROGRESS"

  - dependency: "PostgreSQL full-text search extensions"
    type: "DATABASE"
    status: "AVAILABLE"

provides:
  - deliverable: "Patient search API endpoints"
    interface: "RESTful search API with query parameters"
    consumers: "Frontend patient search components, appointment scheduling"
```

### Code Patterns to Follow

```typescript
// src/features/patients/dto/search-patients.dto.ts
import {IsOptional, IsString, IsEnum, IsDateString, IsInt, Min, Max} from 'class-validator';
import {Transform, Type} from 'class-transformer';
import {PatientStatus, Gender} from '../enums/patient.enums';

export class SearchPatientsDto {
    @IsOptional()
    @IsString()
    search?: string; // General search term

    @IsOptional()
    @IsString()
    firstName?: string;

    @IsOptional()
    @IsString()
    lastName?: string;

    @IsOptional()
    @IsString()
    phone?: string;

    @IsOptional()
    @IsDateString()
    dateOfBirth?: string;

    @IsOptional()
    @IsEnum(PatientStatus)
    status?: PatientStatus;

    @IsOptional()
    @IsEnum(Gender)
    gender?: Gender;

    @IsOptional()
    @Type(() => Number)
    @IsInt()
    @Min(0)
    ageFrom?: number;

    @IsOptional()
    @Type(() => Number)
    @IsInt()
    @Max(150)
    ageTo?: number;

    @IsOptional()
    @Type(() => Number)
    @IsInt()
    @Min(1)
    page?: number = 1;

    @IsOptional()
    @Type(() => Number)
    @IsInt()
    @Min(1)
    @Max(100)
    limit?: number = 20;

    @IsOptional()
    @IsString()
    sortBy?: string = 'relevance';

    @IsOptional()
    @IsEnum(['asc', 'desc'])
    sortOrder?: 'asc' | 'desc' = 'desc';
}

// src/features/patients/repositories/patients.repository.ts
@Injectable()
export class PatientRepository extends BaseRepository<Patient> {
    constructor(
        @InjectRepository(Patient)
        private readonly patientRepository: Repository<Patient>,
        private readonly logger: Logger
    ) {
        super(patientRepository);
    }

    async searchPatients(
        searchDto: SearchPatientsDto,
        clinicId: string
    ): Promise<PaginatedResult<PatientSummary>> {
        const queryBuilder = this.patientRepository
            .createQueryBuilder('patient')
            .leftJoinAndSelect('patient.contactInfo', 'contact')
            .where('patient.clinicId = :clinicId', {clinicId});

        // Apply search criteria
        if (searchDto.search) {
            queryBuilder.andWhere(
                `(
          to_tsvector('simple', patient.firstName || ' ' || patient.lastName) @@ plainto_tsquery('simple', :search)
          OR similarity(patient.firstName || ' ' || patient.lastName, :search) > 0.3
          OR patient.firstName ILIKE :searchLike
          OR patient.lastName ILIKE :searchLike
          OR contact.phone LIKE :phoneSearch
        )`,
                {
                    search: searchDto.search,
                    searchLike: `%${searchDto.search}%`,
                    phoneSearch: `%${searchDto.search.replace(/\D/g, '')}%`
                }
            );
        }

        // Apply specific field searches
        if (searchDto.firstName) {
            queryBuilder.andWhere(
                'similarity(patient.firstName, :firstName) > 0.3 OR patient.firstName ILIKE :firstNameLike',
                {
                    firstName: searchDto.firstName,
                    firstNameLike: `%${searchDto.firstName}%`
                }
            );
        }

        if (searchDto.lastName) {
            queryBuilder.andWhere(
                'similarity(patient.lastName, :lastName) > 0.3 OR patient.lastName ILIKE :lastNameLike',
                {
                    lastName: searchDto.lastName,
                    lastNameLike: `%${searchDto.lastName}%`
                }
            );
        }

        if (searchDto.phone) {
            const cleanPhone = searchDto.phone.replace(/\D/g, '');
            queryBuilder.andWhere('contact.phone LIKE :phone', {phone: `%${cleanPhone}%`});
        }

        if (searchDto.dateOfBirth) {
            queryBuilder.andWhere('patient.dateOfBirth = :dateOfBirth', {
                dateOfBirth: searchDto.dateOfBirth
            });
        }

        if (searchDto.status) {
            queryBuilder.andWhere('patient.status = :status', {status: searchDto.status});
        }

        if (searchDto.gender) {
            queryBuilder.andWhere('patient.gender = :gender', {gender: searchDto.gender});
        }

        // Age filtering
        if (searchDto.ageFrom || searchDto.ageTo) {
            const currentDate = new Date();
            if (searchDto.ageFrom) {
                const maxBirthDate = new Date(currentDate.getFullYear() - searchDto.ageFrom, currentDate.getMonth(), currentDate.getDate());
                queryBuilder.andWhere('patient.dateOfBirth <= :maxBirthDate', {maxBirthDate});
            }
            if (searchDto.ageTo) {
                const minBirthDate = new Date(currentDate.getFullYear() - searchDto.ageTo - 1, currentDate.getMonth(), currentDate.getDate());
                queryBuilder.andWhere('patient.dateOfBirth > :minBirthDate', {minBirthDate});
            }
        }

        // Sorting
        if (searchDto.sortBy === 'relevance' && searchDto.search) {
            queryBuilder.addSelect(
                `(
          CASE 
            WHEN patient.firstName || ' ' || patient.lastName = :exactSearch THEN 100
            WHEN patient.firstName ILIKE :exactSearchLike OR patient.lastName ILIKE :exactSearchLike THEN 90
            WHEN contact.phone LIKE :exactPhoneSearch THEN 85
            ELSE similarity(patient.firstName || ' ' || patient.lastName, :exactSearch) * 100
          END
        )`,
                'relevance_score'
            )
                .setParameter('exactSearch', searchDto.search)
                .setParameter('exactSearchLike', searchDto.search)
                .setParameter('exactPhoneSearch', searchDto.search.replace(/\D/g, ''))
                .orderBy('relevance_score', 'DESC');
        } else {
            const sortField = this.getSortField(searchDto.sortBy);
            queryBuilder.orderBy(sortField, searchDto.sortOrder?.toUpperCase() as 'ASC' | 'DESC');
        }

        // Pagination
        const offset = (searchDto.page - 1) * searchDto.limit;
        queryBuilder.skip(offset).take(searchDto.limit);

        const [patients, total] = await queryBuilder.getManyAndCount();

        return {
            data: patients.map(patient => this.mapToPatientSummary(patient)),
            pagination: {
                page: searchDto.page,
                limit: searchDto.limit,
                total,
                totalPages: Math.ceil(total / searchDto.limit)
            }
        };
    }

    async getRecentPatients(userId: string, clinicId: string): Promise<PatientSummary[]> {
        const recentAccess = await this.patientRepository
            .createQueryBuilder('patient')
            .leftJoinAndSelect('patient.contactInfo', 'contact')
            .innerJoin('patient_access_log', 'access', 'access.patientId = patient.id')
            .where('access.userId = :userId', {userId})
            .andWhere('patient.clinicId = :clinicId', {clinicId})
            .orderBy('access.accessedAt', 'DESC')
            .limit(10)
            .getMany();

        return recentAccess.map(patient => this.mapToPatientSummary(patient));
    }

    private getSortField(sortBy: string): string {
        const sortFields = {
            'name': 'patient.lastName',
            'firstName': 'patient.firstName',
            'lastName': 'patient.lastName',
            'dateOfBirth': 'patient.dateOfBirth',
            'createdAt': 'patient.createdAt',
            'updatedAt': 'patient.updatedAt'
        };
        return sortFields[sortBy] || 'patient.updatedAt';
    }

    private mapToPatientSummary(patient: Patient): PatientSummary {
        return {
            id: patient.id,
            firstName: patient.firstName,
            lastName: patient.lastName,
            dateOfBirth: patient.dateOfBirth,
            gender: patient.gender,
            phone: patient.contactInfo?.phone,
            email: patient.contactInfo?.email,
            status: patient.status,
            lastVisit: patient.lastVisit // This would come from appointments
        };
    }
}

// src/features/patients/services/patient-search.service.ts
@Injectable()
export class PatientSearchService {
    constructor(
        private readonly patientRepository: PatientRepository,
        private readonly patientAccessService: PatientAccessService,
        private readonly logger: Logger
    ) {
    }

    async searchPatients(
        searchDto: SearchPatientsDto,
        userId: string
    ): Promise<Result<PaginatedResult<PatientSummary>, ValidationError>> {
        try {
            const clinicId = await this.getClinicIdForUser(userId);

            const startTime = Date.now();
            const result = await this.patientRepository.searchPatients(searchDto, clinicId);
            const searchTime = Date.now() - startTime;

            // Log slow searches for optimization
            if (searchTime > 1000) {
                this.logger.warn(`Slow patient search: ${searchTime}ms`, {
                    searchCriteria: searchDto,
                    resultCount: result.data.length
                });
            }

            return Result.success(result);
        } catch (error) {
            this.logger.error('Patient search failed', error);
            return Result.error(new ValidationError('Search failed'));
        }
    }

    async getRecentPatients(userId: string): Promise<Result<PatientSummary[], ValidationError>> {
        try {
            const clinicId = await this.getClinicIdForUser(userId);
            const recentPatients = await this.patientRepository.getRecentPatients(userId, clinicId);

            return Result.success(recentPatients);
        } catch (error) {
            this.logger.error('Failed to get recent patients', error);
            return Result.error(new ValidationError('Failed to get recent patients'));
        }
    }

    async trackPatientAccess(patientId: string, userId: string): Promise<void> {
        try {
            await this.patientAccessService.recordAccess(patientId, userId);
        } catch (error) {
            // Log but don't fail the main operation
            this.logger.error('Failed to track patient access', error);
        }
    }
}

// src/features/patients/controllers/patients.controller.ts (search endpoints)
@Controller('patients')
@UseGuards(AuthGuard, RolesGuard)
export class PatientsController {
    constructor(
        private readonly patientSearchService: PatientSearchService
    ) {
    }

    @Get()
    @Roles('patient:read')
    @ApiOperation({summary: 'Search and list patients'})
    @ApiResponse({status: 200, description: 'Patients found', type: PaginatedPatientResponse})
    async searchPatients(
        @Query() searchDto: SearchPatientsDto,
        @CurrentUser() user: User
    ): Promise<PaginatedResult<PatientSummary>> {
        const result = await this.patientSearchService.searchPatients(searchDto, user.id);

        if (result.isError) {
            throw new BadRequestException(result.error.message);
        }

        return result.data;
    }

    @Get('recent')
    @Roles('patient:read')
    @ApiOperation({summary: 'Get recently accessed patients'})
    @ApiResponse({status: 200, description: 'Recent patients retrieved', type: [PatientSummary]})
    async getRecentPatients(
        @CurrentUser() user: User
    ): Promise<PatientSummary[]> {
        const result = await this.patientSearchService.getRecentPatients(user.id);

        if (result.isError) {
            throw new BadRequestException(result.error.message);
        }

        return result.data;
    }

    @Get(':patientId')
    @Roles('patient:read')
    async getPatient(
        @Param('patientId', ParseUUIDPipe) patientId: string,
        @CurrentUser() user: User
    ): Promise<Patient> {
        // Track access for recent patients list
        await this.patientSearchService.trackPatientAccess(patientId, user.id);

        // ... existing patient retrieval logic
    }
}
```

## Testing Requirements

### Test Coverage

```yaml
unit_tests:
  - focus: "Search algorithm logic and query building"
  - coverage_target: "85%"
  - key_scenarios: [
    "Multi-criteria search combinations",
    "Fuzzy matching with various similarity scores",
    "Pagination and sorting logic",
    "Recent patients tracking",
    "Search performance optimization"
  ]

integration_tests:
  - focus: "Database search queries and API endpoints"
  - test_environment: "Test database with diverse patient sample data"
  - key_flows: [
    "Full-text search with PostgreSQL",
    "Search API endpoint with various parameters",
    "Recent patients API functionality",
    "Search performance under load"
  ]
```

### Test Scenarios

```yaml
happy_path:
  - scenario: "Search patients by partial name"
    steps: [
      "Submit GET /api/patients?search=john",
      "Verify results include patients with 'John' in first/last name",
      "Verify results are sorted by relevance",
      "Verify response time is acceptable"
    ]
    expected_result: "200 OK with relevant patient results"

error_cases:
  - scenario: "Search with invalid parameters"
    trigger: "Submit search with invalid age range (ageFrom > ageTo)"
    expected_behavior: "400 Bad Request with validation error"

  - scenario: "Search with no results"
    trigger: "Submit search criteria that matches no patients"
    expected_behavior: "200 OK with empty results array"

edge_cases:
  - scenario: "Search with Arabic patient names"
    conditions: "Patient database contains Arabic names"
    expected_behavior: "Search returns correct results with proper character handling"
```

## Context References

### From Project

```yaml
tech_stack_references:
  - "PostgreSQL full-text search with GIN indexes"
  - "TypeORM query builder for complex search queries"
  - "class-validator for search parameter validation"

architecture_patterns:
  - "Repository pattern for search query encapsulation"
  - "Service pattern for search business logic"
  - "Result pattern for error handling"

code_standards:
  - "Comprehensive logging for search performance monitoring"
  - "Input sanitization and SQL injection prevention"
  - "Consistent API response formatting"
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
  - "PatientSummary entity for search results"
  - "SearchPatientsDto for search parameters"
  - "PaginatedResult for search response format"

integration_points:
  - "Consumed by frontend patient search components"
  - "Used by appointment scheduling for patient selection"
  - "Integrates with patient access tracking"
```

## Acceptance Criteria

### Functional Acceptance

- [ ] Multi-criteria search works with all specified parameters
- [ ] Fuzzy matching handles typos and partial matches correctly
- [ ] Recent patients tracking works for last 10 accessed patients
- [ ] Search respects clinic boundaries and user permissions
- [ ] Arabic and French character search works properly

### Technical Acceptance

- [ ] Database indexes optimize search query performance
- [ ] Unit tests written and passing (>= 85% coverage)
- [ ] Integration tests cover all search scenarios
- [ ] API documentation complete with search examples
- [ ] Performance monitoring and slow query logging implemented

### Quality Acceptance

- [ ] Code review completed and approved
- [ ] Search performance meets requirements under load testing
- [ ] Security validation prevents SQL injection attacks
- [ ] Error handling provides helpful search feedback
- [ ] Search relevance ranking produces intuitive results

---
**Last Updated:** 2025-01-16