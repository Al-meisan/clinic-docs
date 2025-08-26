# TASK-004: Database Infrastructure

## Task Overview

### Goal
**What to Build:** Set up PostgreSQL database infrastructure with TypeORM configuration, migration system, initial schema, and connection management optimized for healthcare data requirements.

**Why:** Establishes the data persistence layer foundation that will store all patient records, appointments, clinical documentation, and business data for the MedFlow system with proper security and performance configurations.

**Definition of Done:** PostgreSQL database is operational with proper user permissions, TypeORM migrations working bidirectionally, connection pooling configured, and initial schema supporting core entities.

### Scope
```yaml
task_type: "DATABASE"
complexity: "HIGH"
```

## Requirements

### Functional Requirements
1. **PostgreSQL Database Setup**
   - Description: PostgreSQL instance configured with proper user accounts, database creation, and security settings
   - Acceptance Criteria: Database accepts connections, supports concurrent users, and has proper backup configuration
   - Priority: HIGH

2. **TypeORM Migration System**
   - Description: Complete migration framework with version control, rollback capabilities, and development data seeding
   - Acceptance Criteria: Migrations run successfully in both directions, schema changes are tracked, and development data can be seeded
   - Priority: HIGH

3. **Database Connection Management**
   - Description: Connection pooling, query optimization, and performance monitoring setup
   - Acceptance Criteria: Connection pool configured for load, slow queries logged, and connection limits respected
   - Priority: MEDIUM

4. **Initial Healthcare Schema**
   - Description: Base entities for users, clinics, and audit logging with proper relationships and constraints
   - Acceptance Criteria: Core tables created with proper foreign keys, indexes, and constraint validation
   - Priority: HIGH

### Technical Requirements
```yaml
performance:
  - requirement: "Database query response time"
    target: "< 100ms for standard CRUD operations"
    
security:
  - requirement: "Data encryption and access control"
    implementation: "Encrypted connections, role-based database users, audit logging"
    
compatibility:
  - requirement: "PostgreSQL version compatibility"
    scope: "PostgreSQL 15+ with support for JSON columns and advanced indexing"
```

## Implementation Details

### Technical Approach
```yaml
database_configuration:
  postgresql_setup:
    - version: "PostgreSQL 15+"
    - extensions: ["uuid-ossp", "pg_trgm", "unaccent"]
    - encoding: "UTF8"
    - locale: "en_US.UTF-8"
    - timezone: "UTC"
    
  connection_settings:
    - max_connections: 100
    - shared_buffers: "256MB"
    - effective_cache_size: "1GB"
    - work_mem: "4MB"
    - maintenance_work_mem: "64MB"

typeorm_configuration:
  connection_options:
    - type: "postgres"
    - poolSize: 10
    - maxQueryExecutionTime: 1000
    - logging: ["error", "warn", "migration"]
    - synchronize: false (production)
    - migrationsRun: true
    
  entity_configuration:
    - naming_strategy: "SnakeNamingStrategy"
    - timezone: "UTC"
    - cache: true (query result caching)

migration_structure:
  migrations/:
    - 001_initial_schema.ts
    - 002_create_users_table.ts
    - 003_create_clinics_table.ts
    - 004_create_audit_log_table.ts
    
  seeds/:
    - development-data.ts
    - test-data.ts
```

### Integration Points
```yaml
dependencies:
  - dependency: "AWS RDS or local PostgreSQL instance"
    type: "DATABASE"
    status: "PLANNED"
    
  - dependency: "NestJS application with TypeORM module"
    type: "API"
    status: "IN_PROGRESS"
    
provides:
  - deliverable: "Database schema and connection"
    interface: "TypeORM entities and repository pattern"
    consumers: "All backend services requiring data persistence"
    
  - deliverable: "Migration system"
    interface: "CLI commands for schema management"
    consumers: "Development team and CI/CD deployment pipeline"
    
  - deliverable: "Audit and logging infrastructure"
    interface: "Audit trail tables and triggers"
    consumers: "Compliance and security monitoring systems"
```

### Code Patterns to Follow

**Base Entity Pattern:**
```typescript
// src/shared/entities/base.entity.ts
@Entity()
export abstract class BaseEntity {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @CreateDateColumn({ type: 'timestamp', default: () => 'CURRENT_TIMESTAMP' })
  createdAt: Date;

  @UpdateDateColumn({ type: 'timestamp', default: () => 'CURRENT_TIMESTAMP' })
  updatedAt: Date;

  @Column({ type: 'uuid', nullable: true })
  createdBy: string;

  @Column({ type: 'uuid', nullable: true })
  updatedBy: string;

  @DeleteDateColumn({ type: 'timestamp', nullable: true })
  deletedAt: Date;
}
```

**Migration Pattern:**
```typescript
// src/infrastructure/database/migrations/001_initial_schema.ts
export class InitialSchema1234567890123 implements MigrationInterface {
  name = 'InitialSchema1234567890123';

  public async up(queryRunner: QueryRunner): Promise<void> {
    // Enable required extensions
    await queryRunner.query(`CREATE EXTENSION IF NOT EXISTS "uuid-ossp"`);
    await queryRunner.query(`CREATE EXTENSION IF NOT EXISTS "pg_trgm"`);
    
    // Create initial tables with proper constraints
    await queryRunner.query(`
      CREATE TABLE "users" (
        "id" uuid PRIMARY KEY DEFAULT uuid_generate_v4(),
        "email" varchar UNIQUE NOT NULL,
        "first_name" varchar NOT NULL,
        "last_name" varchar NOT NULL,
        "created_at" TIMESTAMP DEFAULT now(),
        "updated_at" TIMESTAMP DEFAULT now()
      )
    `);
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`DROP TABLE "users"`);
    await queryRunner.query(`DROP EXTENSION IF EXISTS "pg_trgm"`);
    await queryRunner.query(`DROP EXTENSION IF EXISTS "uuid-ossp"`);
  }
}
```

**Repository Pattern:**
```typescript
// src/shared/base.repository.ts
export abstract class BaseRepository<T extends BaseEntity> extends Repository<T> {
  protected constructor(
    private readonly entity: EntityTarget<T>,
    private readonly dataSource: DataSource,
  ) {
    super(entity, dataSource.createEntityManager());
  }

  async findByIdOrFail(id: string): Promise<T> {
    const entity = await this.findOne({ where: { id } as any });
    if (!entity) {
      throw new NotFoundException(`${this.entity.name} with ID ${id} not found`);
    }
    return entity;
  }

  async softDelete(id: string): Promise<void> {
    await this.update(id, { deletedAt: new Date() } as any);
  }
}
```

**Database Configuration:**
```typescript
// src/config/database.config.ts
export const getDatabaseConfig = (): TypeOrmModuleOptions => ({
  type: 'postgres',
  host: process.env.DB_HOST || 'localhost',
  port: parseInt(process.env.DB_PORT) || 5432,
  username: process.env.DB_USERNAME || 'medflow',
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME || 'medflow_dev',
  entities: [__dirname + '/../**/*.entity{.ts,.js}'],
  migrations: [__dirname + '/../infrastructure/database/migrations/*{.ts,.js}'],
  synchronize: process.env.NODE_ENV === 'development',
  logging: process.env.NODE_ENV === 'development' ? 'all' : ['error', 'warn'],
  pool: {
    min: 2,
    max: 10,
    acquireTimeoutMillis: 30000,
    createTimeoutMillis: 30000,
    destroyTimeoutMillis: 5000,
    idleTimeoutMillis: 30000,
    reapIntervalMillis: 1000,
    createRetryIntervalMillis: 200,
  },
});
```

## Testing Requirements

### Test Coverage
```yaml
database_tests:
  - focus: "Migration execution and rollback"
  - coverage_target: "100% of migration scripts"
  - key_scenarios: ["schema_creation", "data_migration", "rollback_safety"]
  
integration_tests:
  - focus: "Entity relationships and constraints"
  - test_environment: "Isolated test database"
  - key_flows: ["entity_creation", "relationship_integrity", "constraint_validation"]
  
performance_tests:
  - focus: "Query performance and connection pooling"
  - test_environment: "Load testing environment"
  - key_scenarios: ["concurrent_connections", "query_response_time", "connection_limits"]
```

### Test Scenarios
```yaml
happy_path:
  - scenario: "Database migration from empty to current schema"
    steps: ["Run all migrations", "Verify schema structure", "Seed development data"]
    expected_result: "Complete schema with proper relationships and sample data"
    
error_cases:
  - scenario: "Migration failure and rollback"
    trigger: "Invalid SQL in migration script"
    expected_behavior: "Transaction rollback with clear error message"
    
edge_cases:
  - scenario: "Connection pool exhaustion"
    conditions: "More concurrent connections than pool size"
    expected_behavior: "Proper queuing and timeout handling"
```

## Context References

### From Project
```yaml
tech_stack_references:
  - "PostgreSQL 15+ as primary database"
  - "TypeORM for database operations and migrations"
  - "AWS RDS for managed database hosting"
  - "Connection pooling for performance optimization"
  
architecture_patterns:
  - "Repository pattern for data access abstraction"
  - "Migration-based schema management"
  - "Soft delete pattern for data integrity"
  - "Audit trail for compliance requirements"
  
code_standards:
  - "Snake_case naming for database columns"
  - "UUID primary keys for all entities"
  - "Consistent timestamp handling in UTC"
  - "Foreign key constraints for referential integrity"
```

### From Epic
```yaml
business_rules:
  - "All patient data must be encrypted and auditable"
  - "Multi-tenant data isolation by clinic ID"
  - "Soft deletes to maintain data integrity"
  - "Audit trail for all data modifications"
  
integration_points:
  - "Foundation for patient, appointment, and clinical data"
  - "Authentication and user management storage"
  - "Billing and payment transaction storage"
```

## Acceptance Criteria

### Functional Acceptance
- [ ] PostgreSQL database operational with proper user accounts
- [ ] TypeORM migrations run successfully in both directions
- [ ] Connection pooling configured and tested under load
- [ ] Base entities (User, Clinic, AuditLog) created with proper relationships
- [ ] Development data seeding functional

### Technical Acceptance  
- [ ] Database schema follows healthcare data best practices
- [ ] Query performance meets response time requirements
- [ ] Connection management handles concurrent users properly
- [ ] Backup and recovery procedures documented and tested
- [ ] Security configurations implemented (encryption, access control)

### Quality Acceptance
- [ ] All migrations tested for forward and backward compatibility
- [ ] Database constraints prevent invalid data entry
- [ ] Audit logging captures all required data changes
- [ ] Performance benchmarks meet established targets
- [ ] Documentation includes schema diagrams and relationship mappings

---

**Last Updated:** 2025-01-24
