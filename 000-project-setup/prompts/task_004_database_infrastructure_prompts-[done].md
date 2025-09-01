# Database Infrastructure Task Breakdown (TASK-004)

## ~~Prompt 1: PostgreSQL Database Setup and Configuration~~

### Problem Statement

Set up PostgreSQL database instance with proper configuration, extensions, and user management for the MedFlow
healthcare application.

### Context

- **Tech Stack**: PostgreSQL 15+, NestJS backend with TypeORM
- **Architecture**: Healthcare data requiring secure storage, audit trails, and multi-tenant isolation
- **Environment**: Development and production database configuration needed
- **Security Requirements**: Encrypted connections, role-based access, audit logging

### Detailed Requirements

1. **PostgreSQL Instance Configuration**
    - Install and configure PostgreSQL 15+ with UTF8 encoding
    - Set timezone to UTC for all database operations
    - Configure shared_buffers: 256MB, effective_cache_size: 1GB
    - Set max_connections: 100, work_mem: 4MB, maintenance_work_mem: 64MB

2. **Database Extensions Setup**
    - Enable `uuid-ossp` extension for UUID generation
    - Enable `pg_trgm` extension for trigram text search
    - Enable `unaccent` extension for text normalization

3. **User and Security Configuration**
    - Create dedicated database user `medflow` with appropriate permissions
    - Configure secure connections
    - Set up role-based access control for different application components

### Success Criteria

- [ ] PostgreSQL instance accepts connections on configured port
- [ ] All required extensions are installed and functional
- [ ] Database user can perform CRUD operations with proper permissions
- [ ] SSL connections are enforced and working
- [ ] Performance configuration parameters are applied and measurable

### Test Cases

```sql
-- Test 1: Extension verification
SELECT *
FROM pg_extension
WHERE extname IN ('uuid-ossp', 'pg_trgm', 'unaccent');

-- Test 2: UUID generation
SELECT uuid_generate_v4();

-- Test 3: Connection with SSL
\conninfo -- Should show SSL connection

-- Test 4: Performance settings verification
SHOW shared_buffers;
SHOW
effective_cache_size;
SHOW
max_connections;
```

### Deliverables

- PostgreSQL configuration files (postgresql.conf, pg_hba.conf)
- Database setup script with user creation
- Environment-specific configuration documentation
- Connection verification script

---

## ~~Prompt 2: TypeORM Configuration and Migration Framework~~

### Problem Statement

Configure TypeORM with NestJS and implement a complete migration system supporting schema versioning, rollbacks, and
environment-specific configurations.

### Context

- **Framework**: NestJS with TypeORM module
- **Database**: PostgreSQL with snake_case naming convention
- **Requirements**: Bidirectional migrations, development data seeding, production safety
- **Architecture**: Repository pattern with base entities for audit trails

### Detailed Requirements

1. **TypeORM Module Configuration**
    - Configure TypeORM with PostgreSQL connection
    - Implement snake_case naming strategy for database columns
    - Set up connection pooling (min: 2, max: 10 connections)
    - Configure query logging for development, error logging for production

2. **Migration System Setup**
    - Create migration directory structure: `src/infrastructure/database/migrations/`
    - Configure automatic migration detection and execution
    - Implement CLI commands for migration generation and execution
    - Set up rollback capabilities with transaction safety

3. **Configuration Management**
    - Environment-based configuration (development, staging, production)
    - Secure credential management using environment variables
    - Connection timeout and retry configuration

### Success Criteria

- [ ] TypeORM connects successfully to PostgreSQL
- [ ] Migration system generates new migrations from entity changes
- [ ] Forward and backward migration execution works without data loss
- [ ] Connection pooling maintains stable connection count under load
- [ ] Environment-specific configurations load correctly

### Test Cases

```typescript
// Test 1: Database connection verification
describe('Database Connection', () => {
    it('should connect to PostgreSQL with proper configuration', async () => {
        const connection = await dataSource.initialize();
        expect(connection.isInitialized).toBe(true);
        expect(connection.options.type).toBe('postgres');
    });
});

// Test 2: Migration execution
describe('Migrations', () => {
    it('should run all migrations without errors', async () => {
        const migrations = await dataSource.runMigrations();
        expect(migrations.length).toBeGreaterThan(0);
    });

    it('should rollback migrations successfully', async () => {
        await expect(dataSource.undoLastMigration()).resolves.not.toThrow();
    });
});

// Test 3: Connection pool behavior
describe('Connection Pool', () => {
    it('should maintain connection limits under concurrent load', async () => {
        // Simulate 15 concurrent queries (exceeding pool size of 10)
        const queries = Array(15).fill(0).map(() =>
            dataSource.query('SELECT 1 as test')
        );
        const results = await Promise.all(queries);
        expect(results).toHaveLength(15);
    });
});
```

### Deliverables

- TypeORM configuration module (`database.config.ts`)
- Migration CLI commands and documentation
- Base migration template with examples
- Environment configuration files
- Connection pool monitoring setup

---

## ~~Prompt 3: Base Entity Pattern Implementation~~

### Problem Statement

update the standardized base entity class to provides common fields, audit trails, and soft delete functionality for all
healthcare data entities.

### Context

- **Framework**: TypeORM with PostgreSQL
- **Requirements**: UUID primary keys, audit trails, soft deletes, multi-tenant support
- **Compliance**: Healthcare data requires comprehensive audit logging
- **Pattern**: All entities must inherit from base entity for consistency

### Detailed Requirements

1. **Base Entity Definition**
    - UUID primary key using `uuid_generate_v4()`
    - Automatic timestamp fields: `createdAt`, `updatedAt`, `deletedAt`
    - Audit fields: `createdBy`, `updatedBy` (UUID references to user entity)
    - Soft delete implementation using `deletedAt` column

2. **Audit Trail Integration**
    - Automatic tracking of entity changes
    - User context capture for all modifications
    - Timestamp precision to milliseconds
    - Immutable creation timestamp

3. **TypeORM Decorators and Validation**
    - Proper column types and constraints
    - Database-level default values
    - Index optimization for common queries
    - Validation rules for required fields

### Success Criteria

- [ ] Base entity compiles and generates correct database schema
- [ ] All timestamp fields auto-populate on entity operations
- [ ] Soft delete functionality works without affecting queries
- [ ] Audit fields capture user context correctly
- [ ] Index performance meets query time requirements (<10ms for primary key lookups)

### Test Cases

```typescript
// Test 1: Base entity creation
describe('BaseEntity', () => {
    it('should generate UUID primary key automatically', async () => {
        const entity = new TestEntity();
        await repository.save(entity);
        expect(entity.id).toMatch(/^[0-9a-f]{8}-[0-9a-f]{4}-4[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$/);
    });

    it('should set timestamps automatically', async () => {
        const entity = new TestEntity();
        const beforeSave = new Date();
        await repository.save(entity);
        const afterSave = new Date();

        expect(entity.createdAt.getTime()).toBeGreaterThanOrEqual(beforeSave.getTime());
        expect(entity.createdAt.getTime()).toBeLessThanOrEqual(afterSave.getTime());
        expect(entity.updatedAt).toEqual(entity.createdAt);
    });
});

// Test 2: Soft delete functionality
describe('Soft Delete', () => {
    it('should soft delete entity without removing from database', async () => {
        const entity = await repository.save(new TestEntity());
        await repository.softDelete(entity.id);

        const deletedEntity = await repository.findOne({
            where: {id: entity.id},
            withDeleted: true
        });
        const activeEntity = await repository.findOne({where: {id: entity.id}});

        expect(deletedEntity.deletedAt).toBeDefined();
        expect(activeEntity).toBeNull();
    });
});

// Test 3: Audit trail capture
describe('Audit Trail', () => {
    it('should capture user context in audit fields', async () => {
        const userId = 'test-user-uuid';
        const entity = new TestEntity();
        entity.createdBy = userId;

        await repository.save(entity);
        expect(entity.createdBy).toBe(userId);
    });
});
```

### Deliverables

- `BaseEntity` abstract class with all required fields
- TypeORM decorators and column configurations
- Soft delete implementation methods
- Unit tests covering all functionality
- Documentation for entity inheritance pattern

---

## ~~Prompt 4: Initial Healthcare Schema Creation~~

### Problem Statement

Create initial database schema with core entities (User, Clinic, AuditLog) including proper relationships, constraints,
and indexes optimized for healthcare data operations.

### Context

- **Domain**: Healthcare clinic management system
- **Entities**: Users (staff, providers), Clinics (multi-tenant), AuditLog (compliance)
- **Requirements**: Multi-tenant data isolation, role-based access, comprehensive audit trails
- **Standards**: Healthcare privacy compliance considerations, referential integrity

### Detailed Requirements

1. **User Entity Schema**
    - Fields: id (UUID), email (unique), firstName, lastName, role, clinicId (tenant isolation)
    - Constraints: email uniqueness, non-null required fields
    - Indexes: email lookup, clinic-based queries, role filtering
    - Relationships: Many-to-one with Clinic entity

2. **Clinic Entity Schema**
    - Fields: id (UUID), name, address, phone, email, settings (JSONB)
    - Constraints: unique clinic names within system
    - Indexes: name search, location-based queries
    - Relationships: One-to-many with Users

3. **AuditLog Entity Schema**
    - Fields: id (UUID), entityType, entityId, action, oldValues (JSONB), newValues (JSONB), userId, timestamp
    - Constraints: non-null entity tracking fields
    - Indexes: entity lookups, user activity, timestamp ranges
    - Relationships: References to User for audit trail

### Success Criteria

- [ ] All three entities create tables with correct column types and constraints
- [ ] Foreign key relationships maintain referential integrity
- [ ] Indexes provide sub-100ms query performance for common operations
- [ ] Multi-tenant queries properly filter by clinicId
- [ ] Audit log captures all entity changes with complete data

### Test Cases

```typescript
// Test 1: Entity creation and relationships
describe('Healthcare Schema', () => {
    it('should create user with clinic relationship', async () => {
        const clinic = await clinicRepo.save(new Clinic({name: 'Test Clinic'}));
        const user = await userRepo.save(new User({
            email: 'test@clinic.com',
            firstName: 'John',
            lastName: 'Doe',
            clinicId: clinic.id
        }));

        const userWithClinic = await userRepo.findOne({
            where: {id: user.id},
            relations: ['clinic']
        });
        expect(userWithClinic.clinic.name).toBe('Test Clinic');
    });
});

// Test 2: Multi-tenant data isolation
describe('Multi-tenant Isolation', () => {
    it('should only return users from same clinic', async () => {
        const clinic1 = await clinicRepo.save(new Clinic({name: 'Clinic 1'}));
        const clinic2 = await clinicRepo.save(new Clinic({name: 'Clinic 2'}));

        await userRepo.save(new User({email: 'user1@clinic1.com', clinicId: clinic1.id}));
        await userRepo.save(new User({email: 'user2@clinic2.com', clinicId: clinic2.id}));

        const clinic1Users = await userRepo.find({where: {clinicId: clinic1.id}});
        expect(clinic1Users).toHaveLength(1);
        expect(clinic1Users[0].email).toBe('user1@clinic1.com');
    });
});

// Test 3: Audit log functionality
describe('Audit Logging', () => {
    it('should create audit log entry for entity changes', async () => {
        const user = await userRepo.save(new User({email: 'audit@test.com'}));
        user.firstName = 'Updated';
        await userRepo.save(user);

        const auditEntries = await auditRepo.find({
            where: {entityId: user.id, entityType: 'User'}
        });
        expect(auditEntries.length).toBeGreaterThan(0);
        expect(auditEntries[0].action).toBe('UPDATE');
    });
});

// Test 4: Index performance
describe('Query Performance', () => {
    it('should perform email lookup under 10ms', async () => {
        const startTime = Date.now();
        await userRepo.findOne({where: {email: 'test@clinic.com'}});
        const endTime = Date.now();

        expect(endTime - startTime).toBeLessThan(10);
    });
});
```

### Deliverables

- User entity with proper TypeORM decorators
- Clinic entity with JSONB settings field
- AuditLog entity with change tracking
- Migration scripts for initial schema
- Index definitions for performance optimization
- Relationship mapping and validation

---

## ~~Prompt 5: Database Connection Management and Performance Configuration~~

### Problem Statement

For this prompt some functionalities are already implemented, before any change analyze the current existing code only
implement modification for missing criteria, execute this prompt with the minimum amount of change possible.

Implement robust database connection management with connection pooling, query optimization monitoring, and performance
tuning for concurrent healthcare application usage.

### Context

- **Usage Pattern**: Multiple concurrent users, high read/write operations
- **Requirements**: Sub-100ms response times, connection stability, resource efficiency
- **Monitoring**: Query performance tracking, connection pool metrics, slow query logging
- **Environment**: Development and production configurations

### Detailed Requirements

1. **Connection Pool Configuration**
    - Pool size: min 2, max 10 connections
    - Timeout settings: acquire 30s, create 30s, idle 30s
    - Retry logic: 200ms intervals for failed connections
    - Health checks: connection validation on acquire

2. **Query Performance Monitoring**
    - Log queries exceeding 100ms execution time
    - Track connection pool utilization metrics
    - Monitor database lock conflicts and deadlocks
    - Alert on connection pool exhaustion

3. **Performance Optimization**
    - Query result caching configuration
    - Connection reuse strategies
    - Bulk operation optimization
    - Transaction management for data consistency

### Success Criteria

- [ ] Connection pool maintains stable performance under 50 concurrent operations
- [ ] Query execution times remain under 100ms for 95th percentile
- [ ] Connection pool utilization stays below 80% during normal operations
- [ ] Slow query logging captures and reports performance issues
- [ ] Failed connection attempts recover automatically within retry intervals

### Test Cases

```typescript
// Test 1: Connection pool stress test
describe('Connection Pool Management', () => {
    it('should handle concurrent connections without exhaustion', async () => {
        const concurrentOps = Array(50).fill(0).map(async (_, i) => {
            return await dataSource.query('SELECT $1 as test_id', [i]);
        });

        const results = await Promise.all(concurrentOps);
        expect(results).toHaveLength(50);
        results.forEach((result, i) => {
            expect(result[0].test_id).toBe(i);
        });
    });

    it('should recover from connection failures', async () => {
        // Simulate connection failure
        await dataSource.destroy();

        // Attempt reconnection
        const reconnected = await dataSource.initialize();
        expect(reconnected.isInitialized).toBe(true);

        // Verify functionality
        const result = await dataSource.query('SELECT 1 as health_check');
        expect(result[0].health_check).toBe(1);
    });
});

// Test 2: Query performance monitoring
describe('Query Performance', () => {
    it('should log slow queries exceeding threshold', async () => {
        const logSpy = jest.spyOn(console, 'warn');

        // Execute intentionally slow query
        await dataSource.query('SELECT pg_sleep(0.15)'); // 150ms delay

        expect(logSpy).toHaveBeenCalledWith(
            expect.stringContaining('Query execution time exceeded threshold')
        );
    });

    it('should maintain fast query performance for standard operations', async () => {
        const startTime = Date.now();
        await userRepo.findOne({where: {email: 'performance@test.com'}});
        const endTime = Date.now();

        expect(endTime - startTime).toBeLessThan(100);
    });
});

// Test 3: Transaction management
describe('Transaction Management', () => {
    it('should handle transaction rollback on error', async () => {
        const queryRunner = dataSource.createQueryRunner();
        await queryRunner.connect();
        await queryRunner.startTransaction();

        try {
            await queryRunner.manager.save(new User({email: 'tx-test@clinic.com'}));
            // Force error to trigger rollback
            await queryRunner.manager.query('SELECT invalid_column FROM users');
        } catch (error) {
            await queryRunner.rollbackTransaction();
        } finally {
            await queryRunner.release();
        }

        const user = await userRepo.findOne({where: {email: 'tx-test@clinic.com'}});
        expect(user).toBeNull(); // Should be rolled back
    });
});
```

### Deliverables

- Connection pool configuration with monitoring
- Query performance logging implementation
- Health check endpoints for database connectivity
- Performance benchmarking scripts
- Connection failure recovery mechanisms
- Monitoring dashboard configuration

---

## ~~Prompt 6: Development Data Seeding System~~

### Problem Statement
For this prompt some functionalities are already implemented, before any change analyze the current existing code only
implement modification for missing criteria, execute this prompt with the minimum amount of change possible.

Create a comprehensive data seeding system for development and testing environments with realistic healthcare data that
maintains referential integrity and provides consistent test scenarios.

### Context

- **Purpose**: Development environment setup, integration testing, demonstration data
- **Data Types**: Users, clinics, sample patient records (anonymized), appointments
- **Requirements**: Referentially consistent, easily resetable, environment-specific
- **Compliance**: No real patient data, healthcare privacy-compliant synthetic data

### Detailed Requirements

1. **Seed Data Structure**
    - Create 3 sample clinics with different configurations
    - Generate 15-20 users across different roles (admin, provider, staff)
    - Produce 50+ synthetic patient records with realistic but fake data
    - Generate appointment history spanning 6 months

2. **Data Generation Logic**
    - Use faker.js for realistic but synthetic data generation
    - Maintain referential integrity across all related entities
    - Include edge cases: inactive users, cancelled appointments, etc.
    - Support incremental seeding and full reset capabilities

3. **Seeding Commands and Scripts**
    - CLI command for full database reset and reseed
    - Selective seeding for specific entity types
    - Environment detection to prevent accidental production seeding
    - Rollback capabilities for seed data removal

### Success Criteria

- [ ] Seed data creates without constraint violations or foreign key errors
- [ ] Generated data passes all entity validation rules
- [ ] Seeding process completes in under 60 seconds for full dataset
- [ ] Data provides comprehensive test coverage for all major user workflows
- [ ] Seeding works consistently across development team environments

### Test Cases

```typescript
// Test 1: Complete seeding process
describe('Data Seeding', () => {
    it('should seed all entities without constraint violations', async () => {
        await seedingService.runFullSeed();

        const clinicCount = await clinicRepo.count();
        const userCount = await userRepo.count();
        const patientCount = await patientRepo.count();

        expect(clinicCount).toBeGreaterThanOrEqual(3);
        expect(userCount).toBeGreaterThanOrEqual(15);
        expect(patientCount).toBeGreaterThanOrEqual(50);
    });

    it('should maintain referential integrity', async () => {
        await seedingService.runFullSeed();

        // Verify all users belong to valid clinics
        const usersWithInvalidClinic = await userRepo
            .createQueryBuilder('user')
            .leftJoin('user.clinic', 'clinic')
            .where('clinic.id IS NULL')
            .getCount();

        expect(usersWithInvalidClinic).toBe(0);
    });
});

// Test 2: Selective seeding
describe('Selective Seeding', () => {
    it('should seed only specified entity types', async () => {
        const initialUserCount = await userRepo.count();
        await seedingService.seedUsers();
        const finalUserCount = await userRepo.count();

        expect(finalUserCount).toBeGreaterThan(initialUserCount);
    });
});

// Test 3: Data reset functionality
describe('Data Reset', () => {
    it('should clear all seed data and reseed', async () => {
        await seedingService.runFullSeed();
        const initialCount = await userRepo.count();

        await seedingService.resetAndReseed();
        const finalCount = await userRepo.count();

        expect(finalCount).toBeGreaterThanOrEqual(initialCount);
    });
});

// Test 4: Environment safety
describe('Environment Safety', () => {
    it('should prevent seeding in production environment', async () => {
        process.env.NODE_ENV = 'production';

        await expect(seedingService.runFullSeed())
            .rejects
            .toThrow('Seeding is not allowed in production environment');

        process.env.NODE_ENV = 'development';
    });
});
```

### Deliverables

- Seeding service with faker.js integration
- CLI commands for various seeding operations
- Entity factory classes for consistent data generation
- Seed data configuration files
- Environment safety checks
- Documentation for adding new seed data types

---

## Prompt 7: Database Testing and Validation

### Problem Statement
For this prompt some functionalities are already implemented, before any change analyze the current existing code only
implement modification for missing criteria, execute this prompt with the minimum amount of change possible.

Implement comprehensive testing suite for database operations including unit tests and integration tests to ensure reliability.

### Context

- **Testing Scope**: Entity operations, migrations, concurrent access
- **Test Environment**: Isolated test database, automated test data management
- **Validation**: Healthcare data integrity, audit trail accuracy, multi-tenant isolation

### Detailed Requirements

1. **Unit Testing Framework**
    - Test all entity CRUD operations
    - Validate constraint enforcement and data validation
    - Test soft delete and audit logging functionality
    - Verify multi-tenant data isolation

2. **Integration Testing**
    - Test complete database workflows
    - Validate migration forward and backward compatibility
    - Test connection pool behavior under load
    - Verify backup and recovery procedures


### Success Criteria

- [ ] All entity operations pass unit tests with 100% success rate
- [ ] Migration tests verify bidirectional compatibility without data loss
- [ ] Load tests demonstrate stable behavior with 50+ concurrent operations
- [ ] Test coverage exceeds 80% for all database-related code

### Test Cases

```typescript
// Test 1: Entity CRUD operations
describe('Entity CRUD Operations', () => {
    it('should create, read, update, and delete users correctly', async () => {
        // Create
        const user = await userRepo.save(new User({
            email: 'crud-test@clinic.com',
            firstName: 'Test',
            lastName: 'User'
        }));
        expect(user.id).toBeDefined();

        // Read
        const foundUser = await userRepo.findOne({where: {id: user.id}});
        expect(foundUser.email).toBe('crud-test@clinic.com');

        // Update
        foundUser.firstName = 'Updated';
        const updatedUser = await userRepo.save(foundUser);
        expect(updatedUser.firstName).toBe('Updated');
        expect(updatedUser.updatedAt).not.toEqual(updatedUser.createdAt);

        // Soft Delete
        await userRepo.softDelete(user.id);
        const deletedUser = await userRepo.findOne({where: {id: user.id}});
        expect(deletedUser).toBeNull();

        // Verify soft delete (with deleted entities)
        const softDeletedUser = await userRepo.findOne({
            where: {id: user.id},
            withDeleted: true
        });
        expect(softDeletedUser.deletedAt).toBeDefined();
    });
});

// Test 2: Migration testing
describe('Migration Testing', () => {
    it('should run all migrations forward without errors', async () => {
        await dataSource.runMigrations();
        const appliedMigrations = await dataSource.query(
            'SELECT * FROM migrations ORDER BY timestamp'
        );
        expect(appliedMigrations.length).toBeGreaterThan(0);
    });

    it('should rollback migrations without data loss', async () => {
        // Seed some test data
        const user = await userRepo.save(new User({email: 'migration-test@clinic.com'}));

        // Rollback last migration
        await dataSource.undoLastMigration();

        // Rerun migrations
        await dataSource.runMigrations();

        // Verify data integrity (if schema allows)
        const preservedUser = await userRepo.findOne({
            where: {email: 'migration-test@clinic.com'}
        });
        // Data should be preserved if migration is backward compatible
    });
});


// Test 4: Multi-tenant isolation testing
describe('Multi-tenant Data Isolation', () => {
    it('should prevent cross-tenant data access', async () => {
        const clinic1 = await clinicRepo.save(new Clinic({name: 'Clinic 1'}));
        const clinic2 = await clinicRepo.save(new Clinic({name: 'Clinic 2'}));

        const user1 = await userRepo.save(new User({
            email: 'tenant1@clinic.com',
            clinicId: clinic1.id
        }));

        const user2 = await userRepo.save(new User({
            email: 'tenant2@clinic.com',
            clinicId: clinic2.id
        }));

        // Query with tenant isolation
        const clinic1Users = await userRepo.find({where: {clinicId: clinic1.id}});
        const clinic2Users = await userRepo.find({where: {clinicId: clinic2.id}});

        expect(clinic1Users).toHaveLength(1);
        expect(clinic2Users).toHaveLength(1);
        expect(clinic1Users[0].id).not.toBe(clinic2Users[0].id);
    });
});
```

### Deliverables

- Complete test suite for all database operations
- Migration testing framework
- Load testing configuration
- Test data management utilities
- Database testing documentation and best practices

---

## Implementation Guidelines

### Branch and Commit Strategy

- **Branch naming**: `database/setup-postgresql`, `database/typeorm-config`, `database/base-entities`, etc.
- **Commit format**:
  ```
  feat(database): add PostgreSQL configuration with SSL support
  
  - Configure connection pooling with proper timeouts
  - Enable required PostgreSQL extensions
  - Set up secure database connections
  - Add environment-specific database configurations
  ```

### Documentation Requirements

- Each prompt should result in clear README documentation
- Include setup instructions for new developers
- Document configuration options and environment variables
- Provide troubleshooting guides for common database issues

### Dependencies and Integration

- Coordinate with NestJS application setup (TASK-003)
- Ensure compatibility with authentication system requirements
- Plan for frontend integration points
- Consider automated deployment scripts for migration automation

### Quality Assurance

- All database changes must include appropriate tests
- Performance benchmarks must be documented and repeatable
- Security configurations must be verified and documented
- Multi-tenant data isolation must be thoroughly tested