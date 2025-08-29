# TASK: TASK-014 - Security Implementation

## Task Overview

### Goal

**What to Build:** Implement comprehensive role-based access control (RBAC) and audit logging system for patient
management, ensuring secure access to patient data while maintaining complete accountability for all system
interactions.

**Why:** Patient data security is critical for healthcare compliance and patient privacy. Proper access control prevents
unauthorized data access, while audit logging ensures accountability and supports regulatory compliance requirements for
healthcare data management.

**Definition of Done:**

- OAuth2 scope-based authorization fully implemented for all patient endpoints
- Role-based access control prevents unauthorized patient data access
- Complete audit trail captures all patient data modifications with user identification
- Security compliance meets healthcare data protection standards
- Failed authentication attempts are tracked and secured
- Data protection implemented for sensitive patient information

### Scope

```yaml
task_type: "FULLSTACK"
complexity: "HIGH"
```

## Requirements

### Functional Requirements

1. **OAuth2 Scope-Based Authorization**
    - Description: Implement comprehensive scope-based access control for all patient management operations
    - Acceptance Criteria:
        - All patient API endpoints require appropriate scopes (`patient:read`, `patient:write`)
        - Unauthorized access attempts return 403 Forbidden with clear error messages
        - Scope validation works correctly for all user roles
        - Cross-clinic data access is prevented
    - Priority: HIGH

2. **Role-Based Access Control Implementation**
    - Description: Implement role hierarchy and permissions based on standardized role definitions
    - Acceptance Criteria:
        - System Administrator has `admin:full` scope with complete patient access
        - Healthcare Provider has clinical and patient scopes
        - Clinical Staff has limited patient write access with clinical read
        - Administrative Staff has patient and appointment scopes with limited clinical read
        - Role inheritance works correctly (higher roles include lower-level scopes)
    - Priority: HIGH

3. **Comprehensive Audit Logging**
    - Description: Implement complete audit trail for all patient data operations
    - Acceptance Criteria:
        - All patient data modifications logged with user ID, timestamp, and changes
        - Read operations on sensitive data are audited
        - Audit logs include old and new values for data changes
        - Failed access attempts are logged with user and resource information
        - Audit logs are immutable and tamper-resistant
    - Priority: HIGH

4. **Data Protection and Security**
    - Description: Implement secure handling for sensitive patient data storage and transmission
    - Acceptance Criteria:
        - All patient data securely stored in database
        - HTTPS enforced for all API communications
        - Sensitive medical data has additional protection
        - Data security keys managed securely with AWS
    - Priority: MEDIUM

### Technical Requirements

```yaml
security:
  - requirement: "Role-based access control"
    implementation: "OAuth2 scopes with NestJS guards and decorators"

  - requirement: "Audit logging"
    implementation: "Database audit tables with immutable logging"

  - requirement: "Data protection"
    implementation: "AWS RDS security and application-level protection for sensitive fields"

authentication:
  - requirement: "JWT token validation"
    implementation: "AWS Cognito JWT validation with scope checking"

  - requirement: "Session management"
    implementation: "Secure session handling with proper token refresh"

compliance:
  - requirement: "Healthcare data protection"
    implementation: "GDPR and healthcare regulation compliance measures"
```

## Implementation Details

### Technical Approach

```yaml
# Backend Security Implementation
authentication_guards:
  - guard_type: "JWT Authentication Guard"
    purpose: "Validate AWS Cognito JWT tokens"
    implementation: "Custom NestJS guard with Cognito integration"

  - guard_type: "Scopes Authorization Guard"
    purpose: "Check user scopes against required permissions"
    implementation: "Decorator-based scope validation"

  - guard_type: "Clinic Isolation Guard"
    purpose: "Ensure cross-clinic data access prevention"
    implementation: "Tenant-aware data filtering"

authorization_decorators:
  - decorator: "@RequireScopes(['patient:read'])"
    usage: "Controller methods requiring patient read access"

  - decorator: "@RequireScopes(['patient:write'])"
    usage: "Controller methods requiring patient write access"

  - decorator: "@RequireScopes(['admin:full'])"
    usage: "Administrative operations"

# Audit Logging System
audit_implementation:
  - component: "Audit Entity"
    purpose: "Store all audit log entries"
    fields: "entityType, entityId, action, oldValues, newValues, userId, clinicId, timestamp"

  - component: "Audit Interceptor"
    purpose: "Automatically capture API operations"
    triggers: "All patient CRUD operations"

  - component: "Database Triggers"
    purpose: "Capture direct database modifications"
    implementation: "PostgreSQL triggers for data change audit"

# Data Protection Strategy
protection_layers:
  - layer: "Database Security"
    implementation: "AWS RDS secure storage"
    coverage: "All database data"

  - layer: "Application-Level Protection"
    implementation: "Field-level security for sensitive data"
    coverage: "Medical record numbers, sensitive clinical data"

  - layer: "Transit Security"
    implementation: "HTTPS for all API communications"
    coverage: "All client-server communications"
```

### Integration Points

```yaml
dependencies:
  - dependency: "AWS Cognito User Pool configuration"
    type: "SERVICE"
    status: "AVAILABLE"

  - dependency: "Patient Management API endpoints"
    type: "API"
    status: "AVAILABLE"

  - dependency: "Database schema and entities"
    type: "DATABASE"
    status: "AVAILABLE"

  - dependency: "AWS security services for data protection"
    type: "SERVICE"
    status: "PLANNED"

provides:
  - deliverable: "Secure patient data access controls"
    interface: "Protected API endpoints with scope validation"
    consumers: "Frontend applications, external integrations"

  - deliverable: "Comprehensive audit logging"
    interface: "Audit log queries and reporting endpoints"
    consumers: "Compliance reporting, security monitoring"

  - deliverable: "Role-based permission system"
    interface: "User role management and scope assignment"
    consumers: "User management system, clinic administration"
```

### Code Patterns to Follow

```typescript
// src/common/guards/auth.guard.ts
import {Injectable, CanActivate, ExecutionContext, UnauthorizedException} from '@nestjs/common';
import {JwtService} from '@nestjs/jwt';
import {Request} from 'express';

@Injectable()
export class AuthGuard implements CanActivate {
    constructor(private readonly jwtService: JwtService) {
    }

    async canActivate(context: ExecutionContext): Promise<boolean> {
        const request = context.switchToHttp().getRequest<Request>();
        const token = this.extractTokenFromHeader(request);

        if (!token) {
            throw new UnauthorizedException('Authentication token required');
        }

        try {
            // Validate JWT token with AWS Cognito
            const payload = await this.jwtService.verifyAsync(token);

            // Attach user info to request
            request['user'] = {
                id: payload.sub,
                email: payload.email,
                scopes: payload['custom:scopes']?.split(',') || [],
                clinicId: payload['custom:clinicId'],
                role: payload['custom:role']
            };

            return true;
        } catch (error) {
            throw new UnauthorizedException('Invalid authentication token');
        }
    }

    private extractTokenFromHeader(request: Request): string | undefined {
        const [type, token] = request.headers.authorization?.split(' ') ?? [];
        return type === 'Bearer' ? token : undefined;
    }
}

// src/common/guards/scopes.guard.ts
import {Injectable, CanActivate, ExecutionContext, ForbiddenException} from '@nestjs/common';
import {Reflector} from '@nestjs/core';
import {SCOPES_KEY} from '../decorators/require-scopes.decorator';

@Injectable()
export class ScopesGuard implements CanActivate {
    constructor(private reflector: Reflector) {
    }

    canActivate(context: ExecutionContext): boolean {
        const requiredScopes = this.reflector.getAllAndOverride<string[]>(SCOPES_KEY, [
            context.getHandler(),
            context.getClass(),
        ]);

        if (!requiredScopes) {
            return true;
        }

        const request = context.switchToHttp().getRequest();
        const user = request.user;

        if (!user) {
            throw new ForbiddenException('User information not available');
        }

        const hasRequiredScope = requiredScopes.some(scope =>
            user.scopes?.includes(scope) || user.scopes?.includes('admin:full')
        );

        if (!hasRequiredScope) {
            throw new ForbiddenException(
                `Access denied. Required scopes: ${requiredScopes.join(', ')}`
            );
        }

        return true;
    }
}

// src/common/decorators/require-scopes.decorator.ts
import {SetMetadata} from '@nestjs/common';

export const SCOPES_KEY = 'scopes';
export const RequireScopes = (...scopes: string[]) => SetMetadata(SCOPES_KEY, scopes);

// src/common/guards/clinic-isolation.guard.ts
import {Injectable, CanActivate, ExecutionContext, ForbiddenException} from '@nestjs/common';

@Injectable()
export class ClinicIsolationGuard implements CanActivate {
    canActivate(context: ExecutionContext): boolean {
        const request = context.switchToHttp().getRequest();
        const user = request.user;
        const clinicId = request.params.clinicId || request.body.clinicId;

        // System administrators can access any clinic
        if (user.scopes?.includes('admin:full')) {
            return true;
        }

        // Ensure user can only access their own clinic's data
        if (clinicId && clinicId !== user.clinicId) {
            throw new ForbiddenException('Access denied to clinic data');
        }

        return true;
    }
}

// src/features/patients/controllers/patients.controller.ts
import {Controller, Get, Post, Put, Param, Body, UseGuards, Query} from '@nestjs/common';
import {AuthGuard} from '../../../common/guards/auth.guard';
import {ScopesGuard} from '../../../common/guards/scopes.guard';
import {ClinicIsolationGuard} from '../../../common/guards/clinic-isolation.guard';
import {RequireScopes} from '../../../common/decorators/require-scopes.decorator';
import {AuditLog} from '../../../common/decorators/audit-log.decorator';
import {PatientsService} from '../services/patients.service';

@Controller('patients')
@UseGuards(AuthGuard, ScopesGuard, ClinicIsolationGuard)
export class PatientsController {
    constructor(private readonly patientsService: PatientsService) {
    }

    @Get()
    @RequireScopes('patient:read')
    async searchPatients(@Query() searchDto: SearchPatientDto) {
        return this.patientsService.searchPatients(searchDto);
    }

    @Get(':patientId')
    @RequireScopes('patient:read')
    @AuditLog('PATIENT_VIEW')
    async getPatient(@Param('patientId') patientId: string) {
        return this.patientsService.findById(patientId);
    }

    @Post()
    @RequireScopes('patient:write')
    @AuditLog('PATIENT_CREATE')
    async createPatient(@Body() createPatientDto: CreatePatientDto) {
        return this.patientsService.createPatient(createPatientDto);
    }

    @Put(':patientId')
    @RequireScopes('patient:write')
    @AuditLog('PATIENT_UPDATE')
    async updatePatient(
        @Param('patientId') patientId: string,
        @Body() updatePatientDto: UpdatePatientDto
    ) {
        return this.patientsService.updatePatient(patientId, updatePatientDto);
    }
}

// src/shared/audit/entities/audit-log.entity.ts
import {Entity, PrimaryGeneratedColumn, Column, CreateDateColumn} from 'typeorm';

@Entity('audit_logs')
export class AuditLog {
    @PrimaryGeneratedColumn('uuid')
    id: string;

    @Column()
    entityType: string;

    @Column('uuid')
    entityId: string;

    @Column()
    action: string;

    @Column('jsonb', {nullable: true})
    oldValues: any;

    @Column('jsonb', {nullable: true})
    newValues: any;

    @Column('uuid')
    userId: string;

    @Column('uuid')
    clinicId: string;

    @Column({nullable: true})
    ipAddress: string;

    @Column({nullable: true})
    userAgent: string;

    @CreateDateColumn()
    timestamp: Date;
}

// src/common/decorators/audit-log.decorator.ts
import {SetMetadata} from '@nestjs/common';

export const AUDIT_ACTION_KEY = 'auditAction';
export const AuditLog = (action: string) => SetMetadata(AUDIT_ACTION_KEY, action);

// src/common/interceptors/audit.interceptor.ts
import {Injectable, NestInterceptor, ExecutionContext, CallHandler} from '@nestjs/common';
import {Reflector} from '@nestjs/core';
import {Observable, tap} from 'rxjs';
import {AuditService} from '../../shared/audit/audit.service';
import {AUDIT_ACTION_KEY} from '../decorators/audit-log.decorator';

@Injectable()
export class AuditInterceptor implements NestInterceptor {
    constructor(
        private readonly auditService: AuditService,
        private readonly reflector: Reflector,
    ) {
    }

    intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
        const auditAction = this.reflector.get<string>(AUDIT_ACTION_KEY, context.getHandler());

        if (!auditAction) {
            return next.handle();
        }

        const request = context.switchToHttp().getRequest();
        const user = request.user;
        const entityId = request.params.patientId || request.params.id;

        return next.handle().pipe(
            tap(async (response) => {
                await this.auditService.logAction({
                    entityType: 'PATIENT',
                    entityId,
                    action: auditAction,
                    userId: user.id,
                    clinicId: user.clinicId,
                    ipAddress: request.ip,
                    userAgent: request.headers['user-agent'],
                    newValues: response,
                });
            }),
        );
    }
}

// src/shared/audit/audit.service.ts
import {Injectable} from '@nestjs/common';
import {InjectRepository} from '@nestjs/typeorm';
import {Repository} from 'typeorm';
import {AuditLog} from './entities/audit-log.entity';

interface AuditLogData {
    entityType: string;
    entityId: string;
    action: string;
    userId: string;
    clinicId: string;
    oldValues?: any;
    newValues?: any;
    ipAddress?: string;
    userAgent?: string;
}

@Injectable()
export class AuditService {
    constructor(
        @InjectRepository(AuditLog)
        private readonly auditRepository: Repository<AuditLog>,
    ) {
    }

    async logAction(data: AuditLogData): Promise<void> {
        const auditLog = this.auditRepository.create({
            entityType: data.entityType,
            entityId: data.entityId,
            action: data.action,
            userId: data.userId,
            clinicId: data.clinicId,
            oldValues: data.oldValues,
            newValues: data.newValues,
            ipAddress: data.ipAddress,
            userAgent: data.userAgent,
        });

        await this.auditRepository.save(auditLog);
    }

    async getAuditTrail(entityType: string, entityId: string): Promise<AuditLog[]> {
        return this.auditRepository.find({
            where: {entityType, entityId},
            order: {timestamp: 'DESC'},
        });
    }
}

// Frontend security implementation
// src/lib/api/client.ts
import axios, {AxiosInstance, AxiosError} from 'axios';
import {useAuthStore} from '../store/auth.store';

class ApiClient {
    private client: AxiosInstance;

    constructor() {
        this.client = axios.create({
            baseURL: import.meta.env.VITE_API_BASE_URL,
            timeout: 10000,
        });

        this.setupInterceptors();
    }

    private setupInterceptors() {
        // Request interceptor to add auth token
        this.client.interceptors.request.use(
            (config) => {
                const {accessToken} = useAuthStore.getState();
                if (accessToken) {
                    config.headers.Authorization = `Bearer ${accessToken}`;
                }
                return config;
            },
            (error) => Promise.reject(error)
        );

        // Response interceptor for error handling
        this.client.interceptors.response.use(
            (response) => response,
            async (error: AxiosError) => {
                if (error.response?.status === 401) {
                    // Token expired, attempt refresh
                    const {refreshToken} = useAuthStore.getState();
                    if (refreshToken) {
                        try {
                            await this.refreshTokens();
                            // Retry original request
                            return this.client.request(error.config!);
                        } catch (refreshError) {
                            // Refresh failed, redirect to login
                            useAuthStore.getState().logout();
                            window.location.href = '/login';
                        }
                    }
                }

                if (error.response?.status === 403) {
                    // Insufficient permissions
                    console.error('Access denied:', error.response.data);
                }

                return Promise.reject(error);
            }
        );
    }

    private async refreshTokens(): Promise<void> {
        const {refreshToken, setTokens} = useAuthStore.getState();

        const response = await this.client.post('/auth/refresh', {
            refreshToken,
        });

        setTokens(response.data.accessToken, response.data.refreshToken);
    }

    get instance(): AxiosInstance {
        return this.client;
    }
}

export const apiClient = new ApiClient().instance;

// src/hooks/usePermissions.ts
import {useAuthStore} from '../lib/store/auth.store';

export const usePermissions = () => {
    const {user} = useAuthStore();

    const hasScope = (scope: string): boolean => {
        if (!user?.scopes) return false;
        return user.scopes.includes(scope) || user.scopes.includes('admin:full');
    };

    const hasAnyScope = (scopes: string[]): boolean => {
        return scopes.some(scope => hasScope(scope));
    };

    const canRead = (resource: string): boolean => {
        return hasScope(`${resource}:read`);
    };

    const canWrite = (resource: string): boolean => {
        return hasScope(`${resource}:write`);
    };

    return {
        hasScope,
        hasAnyScope,
        canRead,
        canWrite,
        canReadPatients: () => canRead('patient'),
        canWritePatients: () => canWrite('patient'),
        canManageClinic: () => hasScope('clinic:manage'),
        isAdmin: () => hasScope('admin:full'),
    };
};

// src/components/ProtectedComponent.tsx
import React from 'react';
import {usePermissions} from '../hooks/usePermissions';

interface ProtectedComponentProps {
    requiredScopes: string[];
    children: React.ReactNode;
    fallback?: React.ReactNode;
}

export const ProtectedComponent: React.FC<ProtectedComponentProps> = ({
                                                                          requiredScopes,
                                                                          children,
                                                                          fallback = null,
                                                                      }) => {
    const {hasAnyScope} = usePermissions();

    if (!hasAnyScope(requiredScopes)) {
        return <>{fallback} < />;
    }

    return <>{children} < />;
};
```

## Testing Requirements

### Test Coverage

```yaml
unit_tests:
  - focus: "Authentication and authorization logic"
  - coverage_target: "95%"
  - key_scenarios:
      - "JWT token validation with various token states"
      - "Scope-based access control for different user roles"
      - "Audit logging for all CRUD operations"
      - "secure storage/decryption of sensitive data"

integration_tests:
  - focus: "End-to-end security workflows"
  - test_environment: "Staging with realistic user roles and data"
  - key_flows:
      - "Complete user authentication and authorization flow"
      - "Cross-role access attempt prevention"
      - "Audit trail generation and verification"
      - "Error handling for unauthorized access attempts"

security_tests:
  - focus: "Penetration testing and vulnerability assessment"
  - test_environment: "Dedicated security testing environment"
  - key_scenarios:
      - "Attempted privilege escalation"
      - "Cross-clinic data access attempts"
      - "Token manipulation and replay attacks"
      - "SQL injection and data exposure attempts"
```

### Test Scenarios

```yaml
authentication_tests:
  - scenario: "Valid JWT token authentication"
    steps:
      - "Present valid AWS Cognito JWT token"
      - "Access protected patient endpoint"
    expected_result: "Access granted with proper user context"

  - scenario: "Invalid or expired token handling"
    trigger: "Access attempt with expired/invalid token"
    expected_behavior: "401 Unauthorized response with clear error message"

authorization_tests:
  - scenario: "Scope-based access control validation"
    conditions: "User with patient:read scope accessing patient:write endpoint"
    expected_behavior: "403 Forbidden response with scope requirement message"

audit_logging_tests:
  - scenario: "Complete audit trail for patient data modifications"
    trigger: "Patient profile update by clinical staff"
    expected_behavior: "Audit log entry with user ID, timestamp, old/new values"
```

## Context References

### From Project

```yaml
tech_stack_references:
  - "AWS Cognito for user management and JWT tokens"
  - "NestJS guards and decorators for authorization"
  - "PostgreSQL for secure audit log storage"

architecture_patterns:
  - "OAuth2 scope-based authorization pattern"
  - "Audit logging interceptor pattern"
  - "Role-based access control with inheritance"

code_standards:
  - "Security-first approach to patient data handling"
  - "Comprehensive error handling for security failures"
  - "Immutable audit logging for compliance"
```

### From Epic

```yaml
business_rules:
  - "Only authorized users with patient:read/write scopes can access patient data"
  - "All patient data modifications must be logged with user identification"
  - "Cross-clinic data access must be prevented for non-admin users"

domain_model:
  - "User entity with role and scope management"
  - "AuditLog entity for comprehensive change tracking"
  - "Patient data security and access control requirements"

integration_points:
  - "Security implementation affects all patient management workflows"
  - "Audit logging supports compliance reporting requirements"
```

## Acceptance Criteria

### Functional Acceptance

- [ ] OAuth2 scope-based authorization implemented for all patient endpoints
- [ ] Role-based access control prevents unauthorized access based on user roles
- [ ] Comprehensive audit logging captures all patient data operations
- [ ] Data protection implemented for sensitive patient information
- [ ] Cross-clinic data access prevention working correctly

### Technical Acceptance

- [ ] JWT token validation integrated with AWS Cognito
- [ ] NestJS guards and decorators properly implemented
- [ ] Audit interceptor captures all required operations
- [ ] Database security enabled for patient data
- [ ] Security testing passes with no critical vulnerabilities

### Quality Acceptance

- [ ] Security code review completed and approved
- [ ] Penetration testing performed with acceptable results
- [ ] Audit log integrity verified and tamper-resistant
- [ ] Performance impact of security measures within acceptable limits
- [ ] Documentation updated with security implementation details

---
**Last Updated:** December 2024