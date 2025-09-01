# TASK-003: Backend NestJS Application Setup

## Task Overview

### Goal
**What to Build:** Initialize a complete NestJS backend application with TypeScript, TypeORM database integration, cloud service connections, and foundational architecture following domain-driven design principles.

**Why:** Creates the core backend foundation that will handle all API endpoints, business logic, and data management for the MedFlow healthcare management system.

**Definition of Done:** NestJS application runs locally and in containerized environment, connects to PostgreSQL database, includes authentication setup, and provides a health check endpoint.

### Scope
```yaml
task_type: "BACKEND"
complexity: "HIGH"
```

## Requirements

### Functional Requirements
1. **NestJS Application Foundation**
   - Description: Complete NestJS application with TypeScript configuration, dependency injection, and modular architecture
   - Acceptance Criteria: Application starts successfully, serves API endpoints, and includes proper error handling
   - Priority: HIGH

2. **Database Integration with TypeORM**
   - Description: PostgreSQL database connection with TypeORM ORM, migration system, and base entity setup
   - Acceptance Criteria: Database connection established, migrations work, and basic CRUD operations functional
   - Priority: HIGH

3. **Cloud Service Integration Setup**
   - Description: Cloud SDK integration for file storage service with proper configuration management
   - Acceptance Criteria: Cloud services can be accessed with proper credentials and error handling
   - Priority: MEDIUM

4. **Authentication and Authorization Framework**
   - Description: JWT token handling, guards, and decorators for role-based access control
   - Acceptance Criteria: Authentication middleware working with proper scope validation
   - Priority: HIGH

### Technical Requirements
```yaml
performance:
  - requirement: "Application startup time"
    target: "< 5 seconds for development server startup"
    
security:
  - requirement: "Secure configuration management"
    implementation: "Environment variables for all secrets, no hardcoded values"
    
compatibility:
  - requirement: "Database migration support"
    scope: "Forward and backward migration capabilities"
```

## Implementation Details

### Technical Approach
```yaml
nestjs_architecture:
  core_modules:
    - AppModule (root module)
    - ConfigModule (environment configuration)
    - DatabaseModule (TypeORM connection)
    - AuthModule (authentication/authorization)
    - HealthModule (health check endpoints)
    
  shared_infrastructure:
    - guards/ (authentication and role guards)
    - decorators/ (custom decorators)
    - interceptors/ (response formatting, logging)
    - filters/ (exception handling)
    - pipes/ (validation pipes)

folder_structure:
  src/:
    - main.ts (application entry point)
    - app.module.ts (root module)
    - common/ (shared utilities and decorators)
    - config/ (configuration management)
    - features/ (domain modules - placeholder)
    - shared/ (shared services and utilities)
    - infrastructure/ (external system integrations)

database_setup:
  - entity: "Base entity with common fields"
  - migration: "Initial database schema setup"
  - connection: "TypeORM configuration with connection pooling"
  - seeding: "Development data seeding scripts"
```

### Integration Points
```yaml
dependencies:
  - dependency: "PostgreSQL database instance"
    type: "DATABASE"
    status: "PLANNED"
    
  - dependency: "Cloud service credentials"
    type: "SERVICE"
    status: "AVAILABLE"
    
  - dependency: "Deployment configuration"
    type: "INFRASTRUCTURE"
    status: "IN_PROGRESS"
    
provides:
  - deliverable: "REST API framework"
    interface: "HTTP endpoints with OpenAPI documentation"
    consumers: "Frontend application, mobile apps, third-party integrations"
    
  - deliverable: "Database abstraction layer"
    interface: "TypeORM entities and repositories"
    consumers: "All business logic modules and features"
    
  - deliverable: "Authentication service"
    interface: "JWT token validation and user context"
    consumers: "All protected API endpoints"
```

### Code Patterns to Follow

**Module Organization:**
```typescript
// Feature module structure
@Module({
  imports: [TypeOrmModule.forFeature([Entity])],
  controllers: [FeatureController],
  providers: [FeatureService, FeatureRepository],
  exports: [FeatureService],
})
export class FeatureModule {}
```

**Service Pattern:**
```typescript
@Injectable()
export class FeatureService {
  constructor(
    private readonly repository: FeatureRepository,
    private readonly logger: Logger
  ) {}
  
  async create(dto: CreateFeatureDto): Promise<Feature> {
    try {
      const entity = this.repository.create(dto);
      const result = await this.repository.save(entity);
      this.logger.log(`Feature created: ${result.id}`);
      return result;
    } catch (error) {
      this.logger.error('Feature creation failed', error);
      throw new BadRequestException('Failed to create feature');
    }
  }
}
```

**Controller Pattern:**
```typescript
@Controller('features')
@UseGuards(JwtAuthGuard, RolesGuard)
@ApiTags('features')
export class FeatureController {
  constructor(private readonly service: FeatureService) {}
  
  @Post()
  @RequireScopes('feature:write')
  @ApiOperation({ summary: 'Create new feature' })
  async create(@Body() dto: CreateFeatureDto): Promise<Feature> {
    return this.service.create(dto);
  }
}
```

**Error Handling:**
```typescript
@Catch()
export class GlobalExceptionFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();
    
    const status = exception instanceof HttpException 
      ? exception.getStatus() 
      : HttpStatus.INTERNAL_SERVER_ERROR;
      
    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      message: exception.message || 'Internal server error',
    });
  }
}
```

## Testing Requirements

### Test Coverage
```yaml
unit_tests:
  - focus: "Service layer business logic"
  - coverage_target: "80%"
  - key_scenarios: ["service_methods", "error_handling", "validation"]
  
integration_tests:
  - focus: "API endpoints and database operations"
  - test_environment: "Test database with Docker"
  - key_flows: ["api_requests", "database_transactions", "authentication"]
  
e2e_tests:
  - focus: "Complete request/response cycles"
  - user_scenarios: ["health_check", "authentication_flow", "basic_crud"]
```

### Test Scenarios
```yaml
happy_path:
  - scenario: "Application startup and health check"
    steps: ["Start application", "Call health endpoint", "Verify database connection"]
    expected_result: "HTTP 200 response with system status"
    
error_cases:
  - scenario: "Database connection failure"
    trigger: "Invalid database credentials or unavailable database"
    expected_behavior: "Application logs error and fails gracefully"
    
edge_cases:
  - scenario: "High concurrent request load"
    conditions: "Multiple simultaneous API requests"
    expected_behavior: "Connection pooling handles load without errors"
```

## Context References

### From Project
```yaml
tech_stack_references:
  - "NestJS with TypeScript for backend development"
  - "TypeORM for database operations"
  - "Cloud SDK for service integration"
  - "PostgreSQL as primary database"
  
architecture_patterns:
  - "Domain-driven design with feature modules"
  - "Dependency injection for loose coupling"
  - "Repository pattern for data access"
  - "Guard and decorator patterns for security"
  
code_standards:
  - "ESLint and Prettier for code quality"
  - "Class-validator for input validation"
  - "OpenAPI documentation for all endpoints"
  - "Consistent error handling and logging"
```

### From Epic
```yaml
business_rules:
  - "OAuth2 scope-based authorization system"
  - "Multi-tenant architecture with clinic boundaries"
  - "Audit trail for all data modifications"
  
integration_points:
  - "Foundation for patient management API"
  - "Authentication system for all features"
  - "Database layer for all business entities"
```

## Acceptance Criteria

### Functional Acceptance
- [ ] NestJS application starts and serves HTTP requests
- [ ] PostgreSQL database connection established with TypeORM
- [ ] Health check endpoint returns system status
- [ ] Basic authentication middleware functional
- [ ] Environment configuration working properly

### Technical Acceptance  
- [ ] Code follows NestJS and TypeScript best practices
- [ ] Database migrations can run forward and backward
- [ ] Cloud SDK integration configured (without credentials for security)
- [ ] OpenAPI documentation generated and accessible
- [ ] Error handling and logging properly implemented

### Quality Acceptance
- [ ] Unit tests written for core functionality
- [ ] Integration tests verify database operations
- [ ] Static analysis tools (ESLint) pass without errors
- [ ] Application can run in Docker container
- [ ] Performance meets startup time requirements

