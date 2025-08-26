# TASK-006: Backend Authentication System

## Task Overview

### Goal
**What to Build:** Implement comprehensive JWT token handling, authentication guards, role-based access control decorators, and user context management system integrated with AWS Cognito for the MedFlow backend application.

**Why:** Provides secure, scalable authentication infrastructure that enforces healthcare-specific access controls, manages user sessions, and ensures proper authorization for all API endpoints based on medical roles and permissions.

**Definition of Done:** Complete authentication middleware system operational with JWT validation, scope-based authorization, user context injection, and automatic token refresh handling across all API endpoints.

### Scope
```yaml
task_type: "BACKEND"
complexity: "MEDIUM"
```

## Requirements

### Functional Requirements
1. **JWT Token Management System**
   - Description: Complete JWT token validation, parsing, and refresh mechanism with proper error handling
   - Acceptance Criteria: Tokens validated on every request, automatic refresh before expiration, and secure token storage patterns
   - Priority: HIGH

2. **Authentication Guards Implementation**
   - Description: NestJS guards for authentication verification and user context injection
   - Acceptance Criteria: All protected endpoints require valid authentication, user context available in request objects
   - Priority: HIGH

3. **Role-Based Authorization System**
   - Description: Scope-based authorization guards and decorators for healthcare workflow permissions
   - Acceptance Criteria: API endpoints enforce proper scopes, role hierarchy respected, and permission violations handled gracefully
   - Priority: HIGH

4. **User Context and Session Management**
   - Description: User profile injection, clinic association, and session state management
   - Acceptance Criteria: Current user information available throughout request lifecycle, clinic boundaries enforced
   - Priority: MEDIUM

### Technical Requirements
```yaml
performance:
  - requirement: "Authentication middleware overhead"
    target: "< 50ms additional latency for authentication checks"
    
security:
  - requirement: "Token security and validation"
    implementation: "Secure JWT validation with signature verification and expiration checking"
    
compatibility:
  - requirement: "Multi-tenant authentication support"
    scope: "Clinic isolation enforced at authentication layer"
```

## Implementation Details

### Technical Approach
```yaml
authentication_architecture:
  guards:
    - JwtAuthGuard (token validation and user extraction)
    - ScopesGuard (permission checking based on required scopes)
    - ClinicGuard (multi-tenant data isolation)
    - OptionalAuthGuard (for endpoints with optional authentication)
    
  decorators:
    - @RequireScopes() (specify required permissions)
    - @CurrentUser() (inject user context)
    - @UserClinic() (inject user's clinic context)
    - @Public() (mark endpoints as publicly accessible)
    
  middleware:
    - TokenRefreshMiddleware (automatic token refresh)
    - AuditLoggerMiddleware (log authentication events)
    - RateLimitingMiddleware (prevent authentication abuse)

jwt_implementation:
  token_structure:
    - header: "JWT algorithm and type"
    - payload: "user_id, clinic_id, scopes, role, exp, iat"
    - signature: "AWS Cognito signature verification"
    
  validation_process:
    - signature_verification: "Verify against Cognito public keys"
    - expiration_check: "Ensure token not expired"
    - scope_extraction: "Parse user permissions from token"
    - user_context: "Build user object for request context"
```

### Integration Points
```yaml
dependencies:
  - dependency: "AWS Cognito integration service"
    type: "SERVICE"
    status: "IN_PROGRESS"
    
  - dependency: "NestJS application framework"
    type: "API"
    status: "AVAILABLE"
    
  - dependency: "Database user and clinic entities"
    type: "DATABASE"
    status: "PLANNED"
    
provides:
  - deliverable: "Authentication middleware system"
    interface: "Guards, decorators, and middleware for all endpoints"
    consumers: "All API controllers requiring authentication"
    
  - deliverable: "User context injection"
    interface: "CurrentUser and UserClinic decorators"
    consumers: "Business logic services needing user information"
    
  - deliverable: "Authorization enforcement"
    interface: "Scope-based access control for API endpoints"
    consumers: "All protected API operations"
```

### Code Patterns to Follow

**Main Authentication Guard:**
```typescript
// src/common/guards/jwt-auth.guard.ts
@Injectable()
export class JwtAuthGuard implements CanActivate {
  constructor(
    private readonly cognitoService: CognitoService,
    private readonly userService: UserService,
    private readonly logger: Logger,
  ) {}
  
  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const isPublic = this.reflector.getAllAndOverride<boolean>('isPublic', [
      context.getHandler(),
      context.getClass(),
    ]);
    
    if (isPublic) {
      return true;
    }
    
    try {
      const token = this.extractTokenFromHeader(request);
      if (!token) {
        throw new UnauthorizedException('Authentication token required');
      }
      
      const cognitoUser = await this.cognitoService.validateToken(token);
      const userContext = await this.buildUserContext(cognitoUser);
      
      request.user = userContext;
      return true;
    } catch (error) {
      this.logger.warn('Authentication failed', { error: error.message });
      throw new UnauthorizedException('Invalid or expired authentication token');
    }
  }
  
  private async buildUserContext(cognitoUser: CognitoUserPayload): Promise<UserContext> {
    const user = await this.userService.findByCognitoId(cognitoUser.sub);
    if (!user) {
      throw new UnauthorizedException('User not found in system');
    }
    
    return {
      id: user.id,
      cognitoId: cognitoUser.sub,
      email: cognitoUser.email,
      firstName: user.firstName,
      lastName: user.lastName,
      role: user.role,
      scopes: this.extractScopes(cognitoUser),
      clinicId: user.clinicId,
      isActive: user.isActive,
    };
  }
}
```

**Scope-Based Authorization Guard:**
```typescript
// src/common/guards/scopes.guard.ts
@Injectable()
export class ScopesGuard implements CanActivate {
  constructor(private readonly reflector: Reflector) {}
  
  canActivate(context: ExecutionContext): boolean {
    const requiredScopes = this.reflector.getAllAndOverride<string[]>('scopes', [
      context.getHandler(),
      context.getClass(),
    ]);
    
    if (!requiredScopes || requiredScopes.length === 0) {
      return true;
    }
    
    const request = context.switchToHttp().getRequest();
    const user = request.user as UserContext;
    
    if (!user) {
      throw new ForbiddenException('User context not available');
    }
    
    const hasRequiredScopes = requiredScopes.every(scope => 
      user.scopes.includes(scope) || this.checkScopeHierarchy(user.scopes, scope)
    );
    
    if (!hasRequiredScopes) {
      throw new ForbiddenException(`Insufficient permissions. Required scopes: ${requiredScopes.join(', ')}`);
    }
    
    return true;
  }
  
  private checkScopeHierarchy(userScopes: string[], requiredScope: string): boolean {
    // admin:full provides all permissions
    if (userScopes.includes('admin:full')) {
      return true;
    }
    
    // clinic:manage provides operational permissions
    if (userScopes.includes('clinic:manage') && 
        ['patient:read', 'appointment:read', 'clinical:read'].includes(requiredScope)) {
      return true;
    }
    
    return false;
  }
}
```

**Clinic Isolation Guard:**
```typescript
// src/common/guards/clinic.guard.ts
@Injectable()
export class ClinicGuard implements CanActivate {
  constructor() {}
  
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    const user = request.user as UserContext;
    const clinicId = request.params.clinicId || request.body?.clinicId || request.query.clinicId;
    
    // System administrators can access all clinics
    if (user.scopes.includes('admin:full')) {
      return true;
    }
    
    // Users can only access their own clinic data
    if (clinicId && clinicId !== user.clinicId) {
      throw new ForbiddenException('Access denied to resources outside your clinic');
    }
    
    return true;
  }
}
```

**Custom Authentication Decorators:**
```typescript
// src/common/decorators/auth.decorators.ts
export const RequireScopes = (...scopes: string[]) => 
  SetMetadata('scopes', scopes);

export const Public = () => 
  SetMetadata('isPublic', true);

export const CurrentUser = createParamDecorator(
  (data: keyof UserContext | undefined, ctx: ExecutionContext): UserContext | any => {
    const request = ctx.switchToHttp().getRequest();
    const user = request.user;
    
    return data ? user?.[data] : user;
  },
);

export const UserClinic = createParamDecorator(
  (data: unknown, ctx: ExecutionContext): string => {
    const request = ctx.switchToHttp().getRequest();
    return request.user?.clinicId;
  },
);
```

**Controller Usage Example:**
```typescript
// src/features/patients/controllers/patients.controller.ts
@Controller('patients')
@UseGuards(JwtAuthGuard, ScopesGuard, ClinicGuard)
@ApiTags('patients')
export class PatientsController {
  constructor(private readonly patientsService: PatientsService) {}
  
  @Get()
  @RequireScopes('patient:read')
  @ApiOperation({ summary: 'List patients' })
  async findAll(
    @CurrentUser() user: UserContext,
    @UserClinic() clinicId: string,
    @Query() query: ListPatientsDto,
  ): Promise<PaginatedResponse<PatientSummary>> {
    return this.patientsService.findAll(clinicId, query);
  }
  
  @Post()
  @RequireScopes('patient:write')
  @ApiOperation({ summary: 'Create new patient' })
  async create(
    @CurrentUser() user: UserContext,
    @Body() createPatientDto: CreatePatientDto,
  ): Promise<Patient> {
    return this.patientsService.create(user.clinicId, createPatientDto, user.id);
  }
  
  @Get('public/search')
  @Public()
  @ApiOperation({ summary: 'Public patient search (limited)' })
  async publicSearch(@Query() searchDto: PublicSearchDto): Promise<any> {
    return this.patientsService.publicSearch(searchDto);
  }
}
```

## Testing Requirements

### Test Coverage
```yaml
unit_tests:
  - focus: "Guards, decorators, and authentication logic"
  - coverage_target: "95%"
  - key_scenarios: ["token_validation", "scope_checking", "user_context_injection"]
  
integration_tests:
  - focus: "Authentication flow across different endpoints"
  - test_environment: "Test database with mock users"
  - key_flows: ["protected_endpoint_access", "scope_enforcement", "clinic_isolation"]
  
e2e_tests:
  - focus: "Complete authentication workflows with real tokens"
  - user_scenarios: ["login_and_api_access", "insufficient_permissions", "cross_clinic_access_attempt"]
```

### Test Scenarios
```yaml
happy_path:
  - scenario: "Authenticated user accessing permitted endpoint"
    steps: ["Login to get token", "Make API request with token", "Verify user context injection"]
    expected_result: "Request processed with proper user context available"
    
error_cases:
  - scenario: "Invalid or expired token"
    trigger: "Malformed JWT or expired timestamp"
    expected_behavior: "401 Unauthorized with clear error message"
    
  - scenario: "Insufficient permissions"
    trigger: "User accessing endpoint requiring higher-level scopes"
    expected_behavior: "403 Forbidden with required scopes information"
    
edge_cases:
  - scenario: "Cross-clinic data access attempt"
    conditions: "User from Clinic A trying to access Clinic B data"
    expected_behavior: "403 Forbidden with clinic isolation message"
```

## Context References

### From Project
```yaml
tech_stack_references:
  - "NestJS guards and decorators for access control"
  - "JWT token validation with AWS Cognito"
  - "OAuth2 scope-based authorization system"
  - "Multi-tenant architecture with clinic boundaries"
  
architecture_patterns:
  - "Guard pattern for authentication and authorization"
  - "Decorator pattern for metadata and user context injection"
  - "Middleware pattern for request processing"
  - "Exception filter pattern for consistent error handling"
  
code_standards:
  - "Consistent error messages and HTTP status codes"
  - "Proper logging for security events"
  - "Clean separation between authentication and authorization"
  - "Reusable decorators and guards across controllers"
```

### From Epic
```yaml
business_rules:
  - "OAuth2 scope-based authorization system"
  - "Cross-clinic data access is prohibited unless explicitly authorized"
  - "Healthcare provider roles require appropriate scopes for clinical operations"
  - "All authentication and authorization events must be audited"
  
integration_points:
  - "Required by all business feature modules for access control"
  - "Integrates with AWS Cognito for user management"
  - "Provides foundation for frontend protected routes"
```

## Acceptance Criteria

### Functional Acceptance
- [ ] JWT authentication guard validates tokens and injects user context
- [ ] Scope-based authorization enforces permissions on all protected endpoints
- [ ] Clinic isolation prevents cross-tenant data access
- [ ] Public endpoints accessible without authentication
- [ ] User context available throughout request lifecycle

### Technical Acceptance  
- [ ] Authentication middleware adds minimal performance overhead
- [ ] Error handling provides appropriate HTTP status codes and messages
- [ ] Token refresh mechanism works automatically
- [ ] Audit logging captures all authentication events
- [ ] Guards and decorators are reusable across all controllers

### Quality Acceptance
- [ ] Comprehensive unit tests for all guards and decorators
- [ ] Integration tests verify end-to-end authentication flows
- [ ] Security testing confirms proper access control enforcement
- [ ] Performance testing shows acceptable authentication overhead
- [ ] Documentation includes usage examples and security guidelines
