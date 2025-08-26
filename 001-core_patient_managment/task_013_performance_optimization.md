# TASK: TASK-013 - Performance Optimization

## Task Overview

### Goal
**What to Build:** Optimize patient search performance and database queries to meet specified performance targets, ensuring the system can handle concurrent users efficiently while maintaining data integrity.

**Why:** Patient search is a critical workflow that must be fast and responsive. Poor performance directly impacts user experience and clinic operational efficiency, especially during peak hours when multiple staff members are accessing patient records simultaneously.

**Definition of Done:** 
- Patient search responds within 2 seconds for any search query
- Database queries execute under 100ms for patient lookup operations  
- System supports 50 concurrent users without performance degradation
- Search performance is maintained with database growth (tested up to 10,000 patient records)
- Slow query monitoring and alerting is implemented

### Scope
```yaml
task_type: "BACKEND"
complexity: "HIGH"
```

## Requirements

### Functional Requirements
1. **Database Query Optimization**
   - Description: Optimize all patient-related database queries for maximum performance
   - Acceptance Criteria: 
     - All patient lookup queries execute under 100ms
     - Complex search queries (multi-field, fuzzy matching) execute under 500ms
     - Pagination queries are optimized for large datasets
   - Priority: HIGH

2. **Search Index Implementation**
   - Description: Implement comprehensive database indexes for patient search operations
   - Acceptance Criteria:
     - Full-text search index on patient names (first, middle, last)
     - Composite indexes on frequently queried field combinations
     - Phone number and email indexes with proper formatting
     - Patient ID index for direct lookups
   - Priority: HIGH

3. **Query Result Caching**
   - Description: Implement intelligent caching for frequently accessed patient data
   - Acceptance Criteria:
     - Patient profile data cached for 5 minutes after access
     - Search results cached for 2 minutes with proper invalidation
     - Cache hit rate above 70% for patient lookups
     - Cache invalidation on data updates
   - Priority: MEDIUM

4. **Concurrent User Support**
   - Description: Ensure system performance under concurrent load
   - Acceptance Criteria:
     - Support 50 concurrent users performing patient searches
     - No performance degradation under concurrent load
     - Database connection pooling properly configured
     - Resource contention monitoring implemented
   - Priority: HIGH

### Technical Requirements
```yaml
performance:
  - requirement: "Patient search response time"
    target: "< 2 seconds for any search query"
    
  - requirement: "Database query execution time"
    target: "< 100ms for patient lookup operations"
    
  - requirement: "Concurrent user support"
    target: "50 concurrent users without degradation"

database:
  - requirement: "Query optimization"
    implementation: "Analyze and optimize slow queries, implement proper indexing"
    
  - requirement: "Connection pooling"
    implementation: "Configure TypeORM connection pooling for optimal performance"

monitoring:
  - requirement: "Performance monitoring"
    implementation: "Implement query performance logging and alerting"
```

## Implementation Details

### Technical Approach
```yaml
# Database Optimization
database_optimization:
  indexes:
    - index_type: "GIN (Generalized Inverted Index)"
      purpose: "Full-text search on patient names"
      fields: "first_name, middle_name, last_name"
      
    - index_type: "B-tree composite index"
      purpose: "Multi-field patient search"
      fields: "last_name, first_name, date_of_birth"
      
    - index_type: "B-tree index"
      purpose: "Phone number search"
      fields: "phone (with normalization function)"
      
    - index_type: "Unique B-tree index"
      purpose: "Email lookup"
      fields: "email (lowercase)"
      
    - index_type: "B-tree index"
      purpose: "Date-based queries"
      fields: "created_at, updated_at"

query_optimization:
  - optimization: "Use EXPLAIN ANALYZE to identify slow queries"
    implementation: "Analyze all patient-related queries and optimize execution plans"
    
  - optimization: "Implement query result pagination"
    implementation: "Use offset/limit with proper indexes for large result sets"
    
  - optimization: "Optimize JOIN operations"
    implementation: "Ensure foreign key indexes exist and JOIN conditions are optimal"

# Caching Strategy
caching_implementation:
  cache_store: "Redis with ElastiCache"
  cache_patterns:
    - pattern: "Cache-aside"
      usage: "Patient profile data"
      ttl: "300 seconds (5 minutes)"
      
    - pattern: "Write-through"
      usage: "Frequently accessed patient metadata"
      ttl: "600 seconds (10 minutes)"
      
    - pattern: "Refresh-ahead"
      usage: "Patient search results"
      ttl: "120 seconds (2 minutes)"

# Connection Pooling
connection_pooling:
  - setting: "maxConnections"
    value: "20"
    rationale: "Support concurrent users with safety margin"
    
  - setting: "minConnections"
    value: "5"
    rationale: "Maintain baseline connections for immediate availability"
    
  - setting: "acquireTimeoutMillis"
    value: "30000"
    rationale: "30-second timeout for connection acquisition"
    
  - setting: "idleTimeoutMillis"
    value: "600000"
    rationale: "10-minute idle timeout to free unused connections"
```

### Integration Points
```yaml
dependencies:
  - dependency: "Patient Management API endpoints"
    type: "API"
    status: "AVAILABLE"
    
  - dependency: "Database schema and patient entities"
    type: "DATABASE"
    status: "AVAILABLE"
    
  - dependency: "Redis caching infrastructure"
    type: "SERVICE"
    status: "PLANNED"

provides:
  - deliverable: "Optimized patient search performance"
    interface: "Existing patient API endpoints with improved performance"
    consumers: "Frontend patient search components, appointment booking system"
    
  - deliverable: "Performance monitoring and alerting"
    interface: "CloudWatch metrics and alerts"
    consumers: "DevOps team, system administrators"
```

### Code Patterns to Follow
```typescript
// src/features/patients/repositories/patients.repository.ts
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { Patient } from '../entities/patient.entity';
import { SearchPatientDto } from '../dto/search-patient.dto';

@Injectable()
export class PatientsRepository {
  constructor(
    @InjectRepository(Patient)
    private readonly patientRepository: Repository<Patient>,
  ) {}

  /**
   * Optimized patient search with full-text search and proper indexing
   */
  async searchPatients(searchDto: SearchPatientDto): Promise<[Patient[], number]> {
    const query = this.patientRepository
      .createQueryBuilder('patient')
      .leftJoinAndSelect('patient.contactInfo', 'contactInfo')
      .where('patient.clinicId = :clinicId', { clinicId: searchDto.clinicId });

    // Use full-text search for name queries
    if (searchDto.search) {
      query.andWhere(
        `to_tsvector('simple', 
          COALESCE(patient.firstName, '') || ' ' || 
          COALESCE(patient.middleName, '') || ' ' || 
          COALESCE(patient.lastName, '')
        ) @@ plainto_tsquery('simple', :search)`,
        { search: searchDto.search }
      );
    }

    // Use indexed fields for specific searches
    if (searchDto.phone) {
      query.andWhere('contactInfo.phone = :phone', { 
        phone: this.normalizePhone(searchDto.phone) 
      });
    }

    if (searchDto.email) {
      query.andWhere('LOWER(contactInfo.email) = LOWER(:email)', { 
        email: searchDto.email 
      });
    }

    if (searchDto.dateOfBirth) {
      query.andWhere('patient.dateOfBirth = :dateOfBirth', { 
        dateOfBirth: searchDto.dateOfBirth 
      });
    }

    // Apply status filter with index
    if (searchDto.status) {
      query.andWhere('patient.status = :status', { status: searchDto.status });
    }

    // Optimized pagination
    return query
      .orderBy('patient.lastName', 'ASC')
      .addOrderBy('patient.firstName', 'ASC')
      .skip(searchDto.skip)
      .take(searchDto.limit)
      .getManyAndCount();
  }

  /**
   * Get patient by ID with caching
   */
  async findById(id: string, useCache = true): Promise<Patient | null> {
    if (useCache) {
      const cacheKey = `patient:${id}`;
      const cached = await this.cacheService.get(cacheKey);
      if (cached) {
        return cached;
      }
    }

    const patient = await this.patientRepository
      .createQueryBuilder('patient')
      .leftJoinAndSelect('patient.contactInfo', 'contactInfo')
      .leftJoinAndSelect('patient.emergencyContacts', 'emergencyContacts')
      .leftJoinAndSelect('patient.insurance', 'insurance')
      .where('patient.id = :id', { id })
      .getOne();

    if (patient && useCache) {
      await this.cacheService.set(`patient:${id}`, patient, 300); // 5 minutes
    }

    return patient;
  }

  private normalizePhone(phone: string): string {
    // Remove all non-numeric characters for consistent indexing
    return phone.replace(/\D/g, '');
  }
}

// Database migration for indexes
// src/infrastructure/database/migrations/add-patient-search-indexes.ts
import { MigrationInterface, QueryRunner } from 'typeorm';

export class AddPatientSearchIndexes implements MigrationInterface {
  name = 'AddPatientSearchIndexes';

  public async up(queryRunner: QueryRunner): Promise<void> {
    // Full-text search index for patient names
    await queryRunner.query(`
      CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_patient_fulltext_search 
      ON patients USING gin(
        to_tsvector('simple', 
          COALESCE(first_name, '') || ' ' || 
          COALESCE(middle_name, '') || ' ' || 
          COALESCE(last_name, '')
        )
      )
    `);

    // Composite index for name-based searches
    await queryRunner.query(`
      CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_patient_name_composite 
      ON patients (last_name, first_name, date_of_birth)
    `);

    // Phone number index with normalization
    await queryRunner.query(`
      CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_contact_info_phone 
      ON contact_info (regexp_replace(phone, '[^0-9]', '', 'g'))
    `);

    // Email index with case-insensitive search
    await queryRunner.query(`
      CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_contact_info_email 
      ON contact_info (LOWER(email))
    `);

    // Patient status index
    await queryRunner.query(`
      CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_patient_status 
      ON patients (status)
    `);

    // Clinic and status composite index
    await queryRunner.query(`
      CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_patient_clinic_status 
      ON patients (clinic_id, status)
    `);

    // Created/updated timestamp indexes for audit queries
    await queryRunner.query(`
      CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_patient_created_at 
      ON patients (created_at)
    `);

    await queryRunner.query(`
      CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_patient_updated_at 
      ON patients (updated_at)
    `);
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`DROP INDEX IF EXISTS idx_patient_fulltext_search`);
    await queryRunner.query(`DROP INDEX IF EXISTS idx_patient_name_composite`);
    await queryRunner.query(`DROP INDEX IF EXISTS idx_contact_info_phone`);
    await queryRunner.query(`DROP INDEX IF EXISTS idx_contact_info_email`);
    await queryRunner.query(`DROP INDEX IF EXISTS idx_patient_status`);
    await queryRunner.query(`DROP INDEX IF EXISTS idx_patient_clinic_status`);
    await queryRunner.query(`DROP INDEX IF EXISTS idx_patient_created_at`);
    await queryRunner.query(`DROP INDEX IF EXISTS idx_patient_updated_at`);
  }
}

// Performance monitoring service
// src/shared/monitoring/performance.service.ts
import { Injectable, Logger } from '@nestjs/common';

@Injectable()
export class PerformanceService {
  private readonly logger = new Logger(PerformanceService.name);

  async measureQueryPerformance<T>(
    operation: string,
    queryFn: () => Promise<T>
  ): Promise<T> {
    const startTime = Date.now();
    
    try {
      const result = await queryFn();
      const duration = Date.now() - startTime;
      
      // Log slow queries
      if (duration > 100) {
        this.logger.warn(`Slow query detected: ${operation} took ${duration}ms`);
      }
      
      // Send metrics to CloudWatch
      await this.sendMetric('QueryDuration', duration, 'Milliseconds', {
        Operation: operation
      });
      
      return result;
    } catch (error) {
      const duration = Date.now() - startTime;
      this.logger.error(`Query failed: ${operation} failed after ${duration}ms`, error);
      throw error;
    }
  }

  private async sendMetric(
    name: string, 
    value: number, 
    unit: string, 
    dimensions: Record<string, string>
  ): Promise<void> {
    // Implementation for CloudWatch metrics
    // This would use AWS SDK to send custom metrics
  }
}
```

## Testing Requirements

### Test Coverage
```yaml
unit_tests:
  - focus: "Database query optimization and indexing"
  - coverage_target: "90%"
  - key_scenarios: 
    - "Patient search with various criteria combinations"
    - "Large dataset pagination performance"
    - "Concurrent query execution"
    - "Cache hit/miss scenarios"
  
performance_tests:
  - focus: "Search performance under load"
  - test_environment: "Staging with production-like data volume"
  - key_metrics:
    - "Search response time under concurrent load"
    - "Database connection pool efficiency"
    - "Cache performance metrics"
    - "Memory usage during peak load"
  
integration_tests:
  - focus: "End-to-end search performance"
  - test_environment: "Staging environment"
  - key_flows:
    - "Patient search from frontend to database"
    - "Cache invalidation on patient updates"
    - "Performance monitoring alert triggers"
```

### Test Scenarios
```yaml
happy_path:
  - scenario: "Fast patient search with common criteria"
    steps: 
      - "Search for patient by last name"
      - "Search for patient by phone number"
      - "Search for patient by email"
    expected_result: "All searches return results within 2 seconds"
    
load_testing:
  - scenario: "Concurrent user search performance"
    trigger: "50 users performing simultaneous patient searches"
    expected_behavior: "System maintains response time targets under concurrent load"
    
stress_testing:
  - scenario: "Large dataset search performance"
    conditions: "Database with 10,000+ patient records"
    expected_behavior: "Search performance remains within targets regardless of dataset size"
```

## Context References

### From Project
```yaml
tech_stack_references:
  - "PostgreSQL 15+ with TypeORM for database operations"
  - "Redis with ElastiCache for caching implementation"
  - "AWS CloudWatch for performance monitoring"
  
architecture_patterns:
  - "Repository pattern for database access abstraction"
  - "Service layer for business logic and caching"
  - "Performance monitoring interceptors"
  
code_standards:
  - "Database query logging and monitoring"
  - "Proper error handling for performance failures"
  - "TypeScript strict typing for performance metrics"
```

### From Epic
```yaml
business_rules:
  - "Patient search must support name, phone, DOB, and patient ID criteria (BR-P017)"
  - "Search results must be sorted by relevance with exact matches first (BR-P018)"
  - "Search must be case-insensitive and support partial matching (BR-P019)"
  
domain_model:
  - "Patient entity with proper indexing requirements"
  - "ContactInfo relationship optimization"
  - "Search performance requirements integration"
  
integration_points:
  - "Patient search API optimization affects appointment booking workflows"
  - "Performance improvements benefit clinical documentation access"
```

## Acceptance Criteria

### Functional Acceptance
- [ ] All patient search queries respond within 2 seconds
- [ ] Database queries execute under 100ms for lookups
- [ ] System supports 50 concurrent users without degradation
- [ ] Full-text search properly implemented and indexed
- [ ] Caching system implemented with proper invalidation

### Technical Acceptance  
- [ ] Database indexes created and optimized
- [ ] Connection pooling properly configured
- [ ] Performance monitoring and alerting implemented
- [ ] Cache hit rate above 70% for patient lookups
- [ ] Slow query detection and logging functional

### Quality Acceptance
- [ ] Performance tests pass under concurrent load
- [ ] No memory leaks during extended usage
- [ ] Cache invalidation works correctly on updates
- [ ] Performance metrics tracked in CloudWatch
- [ ] Code review completed with performance focus

---
**Last Updated:** December 2024
