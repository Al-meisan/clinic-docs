# TASK: TASK-008 - Duplicate Detection System

## Task Overview

### Goal
**What to Build:** Intelligent duplicate patient detection system using fuzzy matching algorithms to prevent duplicate patient registrations while handling variations in names, phone numbers, and addresses common in Algerian healthcare settings.

**Why:** Prevent duplicate patient records that can lead to fragmented medical histories, billing errors, and compromised patient care quality by detecting potential duplicates during registration.

**Definition of Done:** 
- Fuzzy matching algorithm detects potential duplicates based on multiple criteria
- Real-time duplicate checking during patient registration
- Manual duplicate resolution workflow for ambiguous cases
- Patient record merging capabilities for confirmed duplicates
- Configurable matching thresholds and rules

### Scope
```yaml
task_type: "BACKEND"
complexity: "HIGH"
```

## Requirements

### Functional Requirements
1. **Real-Time Duplicate Detection**
   - Description: Check for potential duplicates during patient registration process
   - Acceptance Criteria: 
     - Fuzzy matching on name, phone, and date of birth combination
     - Real-time checking with sub-500ms response time
     - Configurable similarity thresholds (default 0.8 for exact matches, 0.6 for potential matches)
     - Support for Arabic and French name variations
   - Priority: HIGH

2. **Multi-Criteria Matching Algorithm**
   - Description: Comprehensive matching using multiple patient attributes
   - Acceptance Criteria:
     - Name similarity using Levenshtein distance and phonetic matching
     - Phone number normalization and partial matching
     - Date of birth exact match and near-date detection
     - Address similarity for same-household detection
   - Priority: HIGH

3. **Duplicate Resolution Workflow**
   - Description: Manual review and resolution process for potential duplicates
   - Acceptance Criteria:
     - Potential duplicates flagged for manual review
     - Side-by-side comparison interface for ambiguous cases
     - Merge/keep separate decision workflow
     - Audit trail for all duplicate resolution decisions
   - Priority: MEDIUM

4. **Patient Record Merging**
   - Description: Consolidate duplicate patient records while preserving data integrity
   - Acceptance Criteria:
     - Merge patient demographics with conflict resolution
     - Consolidate appointment and medical history
     - Update references across all related entities
     - Preserve audit trail of merge operations
   - Priority: MEDIUM

5. **Batch Duplicate Detection**
   - Description: Identify existing duplicates in patient database
   - Acceptance Criteria:
     - Scan existing patient records for potential duplicates
     - Generate duplicate candidate reports
     - Prioritize review based on confidence scores
     - Progress tracking for large dataset processing
   - Priority: LOW

### Technical Requirements
```yaml
performance:
  - requirement: "Real-time duplicate checking"
    target: "< 500ms response time for duplicate detection"
    
  - requirement: "Batch processing performance"
    target: "Process 1000+ patient records per minute for batch detection"
    
security:
  - requirement: "Merge operation integrity"
    implementation: "Transactional merges with rollback capability"
    
  - requirement: "Audit trail for all duplicate operations"
    implementation: "Complete logging of detection and resolution activities"
    
compatibility:
  - requirement: "Multi-language name matching"
    scope: "Support Arabic and French name variations and transliterations"
```

## Implementation Details

### Technical Approach
```yaml
# Duplicate Detection Strategy
algorithm_implementation:
  - algorithm: "Composite Similarity Score"
    components: "Name similarity (40%), Phone similarity (30%), DOB match (30%)"
    thresholds: "High confidence (>0.8), Medium confidence (0.6-0.8), Low confidence (<0.6)"
    
  - algorithm: "Name Matching"
    techniques: "Levenshtein distance, Double Metaphone phonetic algorithm"
    normalization: "Remove diacritics, standardize spacing, handle Arabic transliteration"
    
  - algorithm: "Phone Number Matching"
    normalization: "Remove formatting, handle country codes, partial number matching"
    fuzzy_matching: "Allow for common data entry errors"

backend_services:
  - service: "DuplicateDetectionService"
    purpose: "Core duplicate detection logic and scoring"
    methods: "detectDuplicates, calculateSimilarity, fuzzyMatch"
    
  - service: "PatientMergeService"
    purpose: "Patient record consolidation and merging"
    methods: "mergePatients, resolveConflicts, updateReferences"
    
database_optimization:
  - optimization: "Similarity search indexes"
    details: "GiST indexes for trigram similarity on names and addresses"
    migration_required: true
    
  - optimization: "Duplicate candidates table"
    details: "Store potential duplicates with confidence scores for review"
    migration_required: true
```

### Integration Points
```yaml
dependencies:
  - dependency: "Patient Registration API (TASK-001)"
    type: "API"
    status: "IN_PROGRESS"
    
  - dependency: "PostgreSQL with pg_trgm extension"
    type: "DATABASE"
    status: "AVAILABLE"
    
provides:
  - deliverable: "Duplicate detection APIs and services"
    interface: "RESTful API for duplicate checking and resolution"
    consumers: "Patient registration workflows, administrative tools"
```

### Code Patterns to Follow
```typescript
// src/features/duplicate-detection/entities/duplicate-candidate.entity.ts
@Entity('duplicate_candidates')
export class DuplicateCandidate {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column('uuid')
  patientId1: string;

  @Column('uuid')
  patientId2: string;

  @Column('decimal', { precision: 5, scale: 3 })
  similarityScore: number;

  @Column('json')
  matchDetails: {
    nameScore: number;
    phoneScore: number;
    dobMatch: boolean;
    addressScore?: number;
    matchedFields: string[];
  };

  @Column({ 
    type: 'enum', 
    enum: ['pending', 'confirmed_duplicate', 'confirmed_different', 'merged'],
    default: 'pending'
  })
  status: DuplicateStatus;

  @Column('uuid', { nullable: true })
  reviewedBy: string;

  @Column({ type: 'timestamp', nullable: true })
  reviewedAt: Date;

  @Column({ type: 'timestamp', default: () => 'CURRENT_TIMESTAMP' })
  createdAt: Date;

  @ManyToOne(() => Patient)
  @JoinColumn({ name: 'patientId1' })
  patient1: Patient;

  @ManyToOne(() => Patient)
  @JoinColumn({ name: 'patientId2' })
  patient2: Patient;

  @ManyToOne(() => User)
  @JoinColumn({ name: 'reviewedBy' })
  reviewer: User;
}

// src/features/duplicate-detection/services/duplicate-detection.service.ts
@Injectable()
export class DuplicateDetectionService {
  constructor(
    private readonly patientRepository: PatientRepository,
    private readonly duplicateCandidateRepository: DuplicateCandidateRepository,
    private readonly nameSimilarityService: NameSimilarityService,
    private readonly phoneNormalizationService: PhoneNormalizationService,
    private readonly logger: Logger
  ) {}

  async detectDuplicatesForPatient(
    patientData: CreatePatientDto,
    clinicId: string
  ): Promise<DuplicateDetectionResult> {
    try {
      const startTime = Date.now();
      
      // Get potential candidates using database similarity search
      const candidates = await this.findPotentialCandidates(patientData, clinicId);
      
      // Calculate similarity scores for each candidate
      const scoredCandidates = await Promise.all(
        candidates.map(candidate => this.calculateSimilarityScore(patientData, candidate))
      );
      
      // Filter and sort by confidence
      const duplicates = scoredCandidates
        .filter(result => result.similarityScore >= this.getMinimumThreshold())
        .sort((a, b) => b.similarityScore - a.similarityScore);

      const executionTime = Date.now() - startTime;
      
      this.logger.log(`Duplicate detection completed in ${executionTime}ms`, {
        candidatesFound: candidates.length,
        duplicatesDetected: duplicates.length,
        patientData: { 
          firstName: patientData.firstName, 
          lastName: patientData.lastName 
        }
      });

      return {
        hasDuplicates: duplicates.length > 0,
        duplicates,
        executionTime
      };

    } catch (error) {
      this.logger.error('Duplicate detection failed', error);
      throw new Error('Failed to detect duplicates');
    }
  }

  private async findPotentialCandidates(
    patientData: CreatePatientDto,
    clinicId: string
  ): Promise<Patient[]> {
    const nameQuery = `${patientData.firstName} ${patientData.lastName}`.toLowerCase();
    const normalizedPhone = this.phoneNormalizationService.normalize(patientData.phone);

    // Use PostgreSQL similarity search with multiple strategies
    const query = this.patientRepository
      .createQueryBuilder('patient')
      .leftJoinAndSelect('patient.contactInfo', 'contact')
      .where('patient.clinicId = :clinicId', { clinicId })
      .andWhere('patient.status != :mergedStatus', { mergedStatus: PatientStatus.MERGED })
      .andWhere(
        new Brackets(qb => {
          // Name similarity using trigrams
          qb.where(
            `similarity(LOWER(patient.firstName || ' ' || patient.lastName), :nameQuery) > :nameThreshold`,
            { nameQuery, nameThreshold: 0.3 }
          )
          // Phone number similarity
          .orWhere(
            `similarity(REGEXP_REPLACE(contact.phone, '[^0-9]', '', 'g'), :normalizedPhone) > :phoneThreshold`,
            { normalizedPhone, phoneThreshold: 0.7 }
          )
          // Exact date of birth match
          .orWhere('patient.dateOfBirth = :dateOfBirth', { 
            dateOfBirth: patientData.dateOfBirth 
          });
        })
      )
      .limit(50); // Limit to prevent excessive processing

    return await query.getMany();
  }

  private async calculateSimilarityScore(
    newPatient: CreatePatientDto,
    existingPatient: Patient
  ): Promise<DuplicateMatchResult> {
    const nameScore = await this.calculateNameSimilarity(
      newPatient.firstName,
      newPatient.lastName,
      existingPatient.firstName,
      existingPatient.lastName
    );

    const phoneScore = this.calculatePhoneSimilarity(
      newPatient.phone,
      existingPatient.contactInfo?.phone
    );

    const dobMatch = this.calculateDateOfBirthMatch(
      newPatient.dateOfBirth,
      existingPatient.dateOfBirth
    );

    const addressScore = await this.calculateAddressSimilarity(
      newPatient.address,
      existingPatient.contactInfo?.address
    );

    // Weighted composite score
    const similarityScore = this.calculateCompositeScore({
      nameScore,
      phoneScore,
      dobMatch: dobMatch ? 1.0 : 0.0,
      addressScore
    });

    return {
      patient: existingPatient,
      similarityScore,
      matchDetails: {
        nameScore,
        phoneScore,
        dobMatch,
        addressScore,
        matchedFields: this.getMatchedFields({
          nameScore,
          phoneScore,
          dobMatch,
          addressScore
        })
      }
    };
  }

  private async calculateNameSimilarity(
    firstName1: string,
    lastName1: string,
    firstName2: string,
    lastName2: string
  ): Promise<number> {
    // Normalize names for comparison
    const normalizedName1 = this.nameSimilarityService.normalize(`${firstName1} ${lastName1}`);
    const normalizedName2 = this.nameSimilarityService.normalize(`${firstName2} ${lastName2}`);

    // Calculate multiple similarity metrics
    const levenshteinScore = this.nameSimilarityService.levenshteinSimilarity(
      normalizedName1,
      normalizedName2
    );

    const phoneticScore = this.nameSimilarityService.phoneticSimilarity(
      normalizedName1,
      normalizedName2
    );

    const trigramScore = this.nameSimilarityService.trigramSimilarity(
      normalizedName1,
      normalizedName2
    );

    // Return weighted average of similarity metrics
    return (levenshteinScore * 0.4 + phoneticScore * 0.4 + trigramScore * 0.2);
  }

  private calculatePhoneSimilarity(phone1: string, phone2: string): number {
    if (!phone1 || !phone2) return 0;

    const normalized1 = this.phoneNormalizationService.normalize(phone1);
    const normalized2 = this.phoneNormalizationService.normalize(phone2);

    if (normalized1 === normalized2) return 1.0;

    // Check for partial matches (last 8 digits)
    const suffix1 = normalized1.slice(-8);
    const suffix2 = normalized2.slice(-8);
    
    if (suffix1 === suffix2 && suffix1.length >= 6) return 0.8;

    // Calculate edit distance for phone numbers
    const editDistance = this.calculateEditDistance(normalized1, normalized2);
    const maxLength = Math.max(normalized1.length, normalized2.length);
    
    return Math.max(0, 1 - (editDistance / maxLength));
  }

  private calculateDateOfBirthMatch(date1: Date | string, date2: Date): boolean {
    const d1 = new Date(date1);
    const d2 = new Date(date2);
    
    // Exact match
    if (d1.getTime() === d2.getTime()) return true;
    
    // Allow for common data entry errors (day/month swap, year off by 1)
    const dayMonthSwapped = new Date(d1.getFullYear(), d1.getDate() - 1, d1.getMonth());
    if (dayMonthSwapped.getTime() === d2.getTime()) return true;
    
    const yearOffByOne = new Date(d1.getFullYear() + 1, d1.getMonth(), d1.getDate());
    const yearOffByOneNeg = new Date(d1.getFullYear() - 1, d1.getMonth(), d1.getDate());
    
    return yearOffByOne.getTime() === d2.getTime() || yearOffByOneNeg.getTime() === d2.getTime();
  }

  private async calculateAddressSimilarity(
    address1: any,
    address2: any
  ): Promise<number> {
    if (!address1 || !address2) return 0;

    const street1 = address1.street?.toLowerCase() || '';
    const street2 = address2.street?.toLowerCase() || '';
    const city1 = address1.city?.toLowerCase() || '';
    const city2 = address2.city?.toLowerCase() || '';

    const streetScore = this.nameSimilarityService.trigramSimilarity(street1, street2);
    const cityScore = this.nameSimilarityService.trigramSimilarity(city1, city2);

    return (streetScore * 0.7 + cityScore * 0.3);
  }

  private calculateCompositeScore(scores: {
    nameScore: number;
    phoneScore: number;
    dobMatch: number;
    addressScore: number;
  }): number {
    // Weighted scoring algorithm
    const weights = {
      name: 0.4,
      phone: 0.3,
      dob: 0.2,
      address: 0.1
    };

    return (
      scores.nameScore * weights.name +
      scores.phoneScore * weights.phone +
      scores.dobMatch * weights.dob +
      scores.addressScore * weights.address
    );
  }

  private getMatchedFields(scores: any): string[] {
    const matchedFields: string[] = [];
    
    if (scores.nameScore > 0.7) matchedFields.push('name');
    if (scores.phoneScore > 0.8) matchedFields.push('phone');
    if (scores.dobMatch) matchedFields.push('dateOfBirth');
    if (scores.addressScore > 0.6) matchedFields.push('address');
    
    return matchedFields;
  }

  private getMinimumThreshold(): number {
    return 0.6; // Configurable threshold for potential duplicates
  }

  private calculateEditDistance(str1: string, str2: string): number {
    const matrix = [];
    const len1 = str1.length;
    const len2 = str2.length;

    for (let i = 0; i <= len2; i++) {
      matrix[i] = [i];
    }

    for (let j = 0; j <= len1; j++) {
      matrix[0][j] = j;
    }

    for (let i = 1; i <= len2; i++) {
      for (let j = 1; j <= len1; j++) {
        if (str2.charAt(i - 1) === str1.charAt(j - 1)) {
          matrix[i][j] = matrix[i - 1][j - 1];
        } else {
          matrix[i][j] = Math.min(
            matrix[i - 1][j - 1] + 1,
            matrix[i][j - 1] + 1,
            matrix[i - 1][j] + 1
          );
        }
      }
    }

    return matrix[len2][len1];
  }
}

// src/features/duplicate-detection/services/name-similarity.service.ts
@Injectable()
export class NameSimilarityService {
  normalize(name: string): string {
    return name
      .toLowerCase()
      .normalize('NFD') // Decompose accented characters
      .replace(/[\u0300-\u036f]/g, '') // Remove diacritics
      .replace(/[^\w\s]/g, '') // Remove special characters
      .replace(/\s+/g, ' ') // Normalize whitespace
      .trim();
  }

  levenshteinSimilarity(str1: string, str2: string): number {
    const distance = this.calculateLevenshteinDistance(str1, str2);
    const maxLength = Math.max(str1.length, str2.length);
    return maxLength === 0 ? 1 : 1 - (distance / maxLength);
  }

  phoneticSimilarity(str1: string, str2: string): number {
    const phonetic1 = this.generatePhoneticCode(str1);
    const phonetic2 = this.generatePhoneticCode(str2);
    return phonetic1 === phonetic2 ? 1.0 : 0.0;
  }

  trigramSimilarity(str1: string, str2: string): number {
    const trigrams1 = this.generateTrigrams(str1);
    const trigrams2 = this.generateTrigrams(str2);
    
    const intersection = trigrams1.filter(t => trigrams2.includes(t));
    const union = [...new Set([...trigrams1, ...trigrams2])];
    
    return union.length === 0 ? 1 : intersection.length / union.length;
  }

  private calculateLevenshteinDistance(str1: string, str2: string): number {
    // Implementation same as in DuplicateDetectionService
    // ... (previous implementation)
  }

  private generatePhoneticCode(str: string): string {
    // Simplified Double Metaphone implementation
    // For production, use a proper phonetic library
    return str.replace(/[aeiou]/gi, '').toUpperCase();
  }

  private generateTrigrams(str: string): string[] {
    const padded = `  ${str}  `;
    const trigrams: string[] = [];
    
    for (let i = 0; i < padded.length - 2; i++) {
      trigrams.push(padded.substr(i, 3));
    }
    
    return trigrams;
  }
}

// src/features/duplicate-detection/services/patient-merge.service.ts
@Injectable()
export class PatientMergeService {
  constructor(
    private readonly patientRepository: PatientRepository,
    private readonly appointmentRepository: AppointmentRepository,
    private readonly clinicalRepository: ClinicalRepository,
    private readonly billingRepository: BillingRepository,
    private readonly auditService: AuditService,
    private readonly logger: Logger
  ) {}

  async mergePatients(
    primaryPatientId: string,
    duplicatePatientId: string,
    mergeDecisions: MergeDecisions,
    userId: string
  ): Promise<Result<Patient, ValidationError>> {
    const queryRunner = this.patientRepository.dataSource.createQueryRunner();
    await queryRunner.connect();
    await queryRunner.startTransaction();

    try {
      // Get both patient records
      const primaryPatient = await queryRunner.manager.findOne(Patient, {
        where: { id: primaryPatientId },
        relations: ['contactInfo', 'emergencyContacts', 'insurance']
      });

      const duplicatePatient = await queryRunner.manager.findOne(Patient, {
        where: { id: duplicatePatientId },
        relations: ['contactInfo', 'emergencyContacts', 'insurance']
      });

      if (!primaryPatient || !duplicatePatient) {
        throw new Error('One or both patients not found');
      }

      // Apply merge decisions to create merged patient data
      const mergedPatientData = this.applyMergeDecisions(
        primaryPatient,
        duplicatePatient,
        mergeDecisions
      );

      // Update primary patient with merged data
      await queryRunner.manager.update(Patient, primaryPatientId, mergedPatientData);

      // Transfer related records to primary patient
      await this.transferRelatedRecords(
        queryRunner,
        duplicatePatientId,
        primaryPatientId
      );

      // Mark duplicate patient as merged
      await queryRunner.manager.update(Patient, duplicatePatientId, {
        status: PatientStatus.MERGED,
        mergedIntoPatientId: primaryPatientId,
        updatedAt: new Date()
      });

      // Create audit trail
      await this.auditService.logPatientMerge(
        primaryPatientId,
        duplicatePatientId,
        userId,
        mergeDecisions
      );

      await queryRunner.commitTransaction();

      // Get the merged patient record
      const mergedPatient = await this.patientRepository.findById(primaryPatientId);
      
      this.logger.log(`Patients merged successfully: ${duplicatePatientId} -> ${primaryPatientId}`);
      return Result.success(mergedPatient);

    } catch (error) {
      await queryRunner.rollbackTransaction();
      this.logger.error('Patient merge failed', error);
      return Result.error(new ValidationError('Failed to merge patients'));
    } finally {
      await queryRunner.release();
    }
  }

  private applyMergeDecisions(
    primaryPatient: Patient,
    duplicatePatient: Patient,
    decisions: MergeDecisions
  ): Partial<Patient> {
    const mergedData: Partial<Patient> = {};

    // Apply field-level merge decisions
    Object.keys(decisions.fieldDecisions).forEach(field => {
      const decision = decisions.fieldDecisions[field];
      if (decision === 'use_duplicate') {
        mergedData[field] = duplicatePatient[field];
      } else if (decision === 'use_primary') {
        mergedData[field] = primaryPatient[field];
      }
      // 'keep_both' would require special handling for array fields
    });

    mergedData.updatedAt = new Date();
    return mergedData;
  }

  private async transferRelatedRecords(
    queryRunner: any,
    fromPatientId: string,
    toPatientId: string
  ): Promise<void> {
    // Transfer appointments
    await queryRunner.manager.update(
      'appointments',
      { patientId: fromPatientId },
      { patientId: toPatientId }
    );

    // Transfer clinical documentation
    await queryRunner.manager.update(
      'clinical_documentation',
      { patientId: fromPatientId },
      { patientId: toPatientId }
    );

    // Transfer prescriptions
    await queryRunner.manager.update(
      'prescriptions',
      { patientId: fromPatientId },
      { patientId: toPatientId }
    );

    // Transfer billing records
    await queryRunner.manager.update(
      'bills',
      { patientId: fromPatientId },
      { patientId: toPatientId }
    );

    // Transfer insurance information (may require special handling for duplicates)
    await queryRunner.manager.update(
      'insurance_info',
      { patientId: fromPatientId },
      { patientId: toPatientId }
    );
  }
}

// src/features/duplicate-detection/controllers/duplicate-detection.controller.ts
@Controller('duplicate-detection')
@UseGuards(AuthGuard, RolesGuard)
export class DuplicateDetectionController {
  constructor(
    private readonly duplicateDetectionService: DuplicateDetectionService,
    private readonly patientMergeService: PatientMergeService
  ) {}

  @Post('check')
  @Roles('patient:write')
  @ApiOperation({ summary: 'Check for duplicate patients' })
  @ApiResponse({ status: 200, description: 'Duplicate detection completed' })
  async checkForDuplicates(
    @Body() patientData: CreatePatientDto,
    @CurrentUser() user: User
  ): Promise<DuplicateDetectionResult> {
    const result = await this.duplicateDetectionService.detectDuplicatesForPatient(
      patientData,
      user.clinicId
    );
    
    return result;
  }

  @Post('merge')
  @Roles('patient:write')
  @ApiOperation({ summary: 'Merge duplicate patients' })
  @ApiResponse({ status: 200, description: 'Patients merged successfully' })
  async mergePatients(
    @Body() mergeRequest: MergePatientsDto,
    @CurrentUser() user: User
  ): Promise<Patient> {
    const result = await this.patientMergeService.mergePatients(
      mergeRequest.primaryPatientId,
      mergeRequest.duplicatePatientId,
      mergeRequest.mergeDecisions,
      user.id
    );

    if (result.isError) {
      throw new BadRequestException(result.error.message);
    }

    return result.data;
  }

  @Get('candidates')
  @Roles('patient:read')
  @ApiOperation({ summary: 'Get pending duplicate candidates for review' })
  async getDuplicateCandidates(
    @Query() query: GetDuplicateCandidatesDto,
    @CurrentUser() user: User
  ): Promise<DuplicateCandidate[]> {
    return await this.duplicateDetectionService.getDuplicateCandidates(
      user.clinicId,
      query
    );
  }
}
```

## Testing Requirements

### Test Coverage
```yaml
unit_tests:
  - focus: "Duplicate detection algorithms and similarity calculations"
  - coverage_target: "90%"
  - key_scenarios: [
    "Name similarity calculation with various inputs",
    "Phone number normalization and matching",
    "Composite scoring algorithm validation",
    "Edge cases for Arabic and French names",
    "Patient merge logic and data integrity"
  ]
  
integration_tests:
  - focus: "Complete duplicate detection and resolution workflow"
  - test_environment: "Test database with diverse patient sample data"
  - key_flows: [
    "Real-time duplicate detection during registration",
    "Batch duplicate detection processing",
    "Patient merge with related record transfer",
    "Performance testing with large datasets"
  ]
```

### Test Scenarios
```yaml
happy_path:
  - scenario: "Detect exact duplicate during registration"
    steps: [
      "Submit patient registration with exact name match",
      "System detects high confidence duplicate",
      "Duplicate warning presented to user",
      "User confirms or rejects duplicate status"
    ]
    expected_result: "Duplicate detected with >0.9 similarity score"
    
error_cases:
  - scenario: "Merge operation failure with rollback"
    trigger: "Database error during patient merge process"
    expected_behavior: "Transaction rolled back, no data corruption, error logged"
    
  - scenario: "Performance degradation with large dataset"
    trigger: "Run duplicate detection on database with 10000+ patients"
    expected_behavior: "Detection completes within performance requirements"
    
edge_cases:
  - scenario: "Arabic name variations and transliterations"
    conditions: "Patient names in Arabic with multiple transliteration variants"
    expected_behavior: "System detects duplicates despite transliteration differences"
```

## Context References

### From Project
```yaml
tech_stack_references:
  - "PostgreSQL with pg_trgm extension for similarity search"
  - "NestJS with TypeORM for duplicate detection services"
  - "Advanced string similarity algorithms (Levenshtein, phonetic)"
  
architecture_patterns:
  - "Service-oriented pattern for modular duplicate detection"
  - "Transaction management for safe patient merging"
  - "Audit trail pattern for compliance and traceability"
  
code_standards:
  - "Comprehensive logging for duplicate detection performance"
  - "Configurable thresholds for different matching scenarios"
  - "Robust error handling and transaction safety"
```

### From Epic
```yaml
business_rules:
  - "BR-P016: System must prevent duplicate patient creation"
  - "Fuzzy matching must handle Arabic and French name variations"
  - "Patient merge operations must preserve all historical data"
  - "All duplicate resolution decisions must be audited"
  
domain_model:
  - "DuplicateCandidate entity for tracking potential duplicates"
  - "Patient merge audit trail for compliance"
  - "Configurable similarity thresholds per clinic"
  
integration_points:
  - "Integrates with patient registration workflow"
  - "Provides duplicate prevention for data quality"
  - "Supports administrative duplicate resolution processes"
```

## Acceptance Criteria

### Functional Acceptance
- [ ] Real-time duplicate detection responds within 500ms during registration
- [ ] Fuzzy matching algorithms handle Arabic and French name variations
- [ ] Duplicate detection accuracy meets minimum 85% precision for high-confidence matches
- [ ] Patient merge functionality preserves all related data with transaction safety
- [ ] Batch duplicate detection processes large datasets efficiently
- [ ] Manual duplicate resolution workflow supports administrative review

### Technical Acceptance  
- [ ] Duplicate detection services follow NestJS architectural patterns
- [ ] Unit tests written and passing (>= 90% coverage)
- [ ] Integration tests cover complete duplicate detection workflows
- [ ] Database optimization supports efficient similarity searching
- [ ] Performance requirements met for real-time and batch operations
- [ ] Transaction safety ensures data integrity during merge operations

### Quality Acceptance
- [ ] Code review completed and approved
- [ ] Performance testing validates requirements under load
- [ ] Accuracy testing with diverse name variations and edge cases
- [ ] Security validation for merge operations and data access
- [ ] User experience testing for duplicate resolution workflows

---
**Last Updated:** 2025-01-16