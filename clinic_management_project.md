# PROJECT: MedFlow

## Project Overview

### What We're Building
**Vision:** A streamlined, user-friendly healthcare management platform designed primarily for single-doctor practices in Algeria, with scalability for mid-size clinics.

**Problem:** Algerian healthcare providers, especially solo practitioners, struggle with manual record-keeping, inefficient appointment scheduling, and fragmented patient management, leading to administrative overhead and reduced patient care quality.

**Solution:** MedFlow offers a simplified yet comprehensive clinic management system with two deployment modes - an intuitive interface optimized for solo practitioners and an advanced mode for multi-provider clinics - delivered as a SaaS solution tailored for the Algerian healthcare market.

**Users:** Healthcare providers (doctors, general practitioners, specialists), clinical staff (nurses, medical assistants), administrative staff (front desk, billing), clinic managers, and system administrators.

### Business Goals
- **Primary Goal:** Digitize and streamline healthcare delivery for Algerian single-doctor practices while reducing administrative burden and operational costs
- **Success Metrics:** 
  - 50% reduction in patient check-in time
  - 30% increase in daily appointment capacity
  - 85% user satisfaction rate (considering local digital literacy)
  - 99% system uptime
  - Local healthcare regulation compliance
  - 100+ single-doctor practices onboarded in first year
- **Timeline:** 
  - MVP (Single Doctor Mode - Arabic/French UI): 8 months
  - Multi-provider Mode: 14 months
  - Advanced Analytics & Mobile App: 20 months

### Business Model & Market Focus
- **Target Market:** Algerian healthcare sector with focus on:
  - Primary: Single-doctor practices (general practitioners, specialists)
  - Secondary: Small to mid-size clinics (2-10 providers)
  - Geographic: Algeria
  
- **Revenue Model:** SaaS subscription with tiered pricing:
  - Starter Plan: Single doctor practices (monthly/annual)
  - Professional Plan: Multi-provider clinics
  - Enterprise Plan: Large clinics with advanced features
  
- **Market Considerations:**
  - French and Arabic language support
  - Local healthcare regulations compliance
  - Algerian payment methods integration
  - Cultural healthcare practices accommodation
  - Gradual digital transformation approach

## Tech Stack

### Core Technologies
```yaml
frontend: "React with TypeScript"
backend: "TypeScript with NestJS"
ui_components: "shadcn/ui"
infrastructure: "AWS (cost-effective approach) with Terraform for IaC"
database: "PostgreSQL 15+ with TypeORM migrations"
architecture: "Domain-driven design with feature modules"
```

### Key Libraries & Tools
```yaml
state_management: "Zustand + TanStack Query (React-Query)"
styling: "Tailwind CSS + shadcn/ui components"
testing: "Jest + React Testing Library + Cypress"
build_tools: "Vite + ESBuild"
user_management: "AWS Cognito + database user profiles"
authentication: "AWS Cognito with OAuth2 scope-based authorization + JWT"
api_documentation: "OpenAPI 3.0 + Swagger UI"
infrastructure: "Terraform for Infrastructure as Code"
monitoring: "AWS CloudWatch + custom metrics"
database_orm: "TypeORM with migration scripts"
validation: "class-validator + class-transformer"
internationalization: "react-i18next + i18next"
deployment: "AWS ECS + RDS + ElastiCache + S3 + Cognito"
email_service: "AWS SES"
file_storage: "AWS S3 with CloudFront CDN"
```

## Architecture Patterns

### Code Organization

#### Backend Structure
```yaml
backend_directory_structure: |
  src/
  ├── main.ts                    # Application entry point
  ├── app.module.ts              # Root module
  ├── common/                    # Reusable code across app
  │   ├── decorators/            # Custom decorators (@Roles, @CurrentUser)
  │   ├── exceptions/            # Custom exception classes
  │   │   ├── validation.exception.ts
  │   │   ├── business.exception.ts
  │   │   └── index.ts
  │   ├── guards/                # Auth/role guards
  │   │   ├── auth.guard.ts
  │   │   ├── roles.guard.ts
  │   │   └── index.ts
  │   ├── interceptors/          # Response shaping, logging
  │   │   ├── response.interceptor.ts
  │   │   ├── logging.interceptor.ts
  │   │   └── index.ts
  │   ├── utils/                 # Utility functions
  │   │   ├── date.utils.ts
  │   │   ├── validation.utils.ts
  │   │   └── index.ts
  │   └── interfaces/            # Shared interfaces
  │       ├── auth.interface.ts
  │       ├── response.interface.ts
  │       └── index.ts
  ├── config/                    # Configuration files
  │   ├── app.config.ts          # App-level config
  │   ├── auth.config.ts         # AWS Cognito configuration
  │   ├── database.config.ts     # DB connection options
  │   └── index.ts
  ├── features/                  # Feature modules
  │   ├── patients/              # Patient management
  │   │   ├── controllers/
  │   │   │   └── patients.controller.ts
  │   │   ├── services/
  │   │   │   └── patients.service.ts
  │   │   ├── repositories/
  │   │   │   └── patients.repository.ts
  │   │   ├── entities/
  │   │   │   └── patient.entity.ts
  │   │   ├── dto/
  │   │   │   ├── create-patient.dto.ts
  │   │   │   ├── update-patient.dto.ts
  │   │   │   └── index.ts
  │   │   ├── patients.module.ts
  │   │   └── index.ts
  │   ├── appointments/          # Appointment scheduling
  │   │   └── [similar structure]
  │   ├── providers/             # Healthcare provider management
  │   │   └── [similar structure]
  │   └── clinical/              # Clinical documentation
  │       └── [similar structure]
  ├── shared/                    # Shared modules/services
  │   ├── base.repository.ts     # Base repository class
  │   ├── base.service.ts        # Shared service logic
  │   ├── logger/                # Custom logging service
  │   │   ├── logger.service.ts
  │   │   ├── logger.module.ts
  │   │   └── index.ts
  │   └── index.ts
  └── infrastructure/            # External systems
      ├── database/              # DB connection, migrations
      │   ├── database.module.ts
      │   ├── migrations/
      │   └── seeds/
      ├── aws/                   # AWS service integrations
      │   ├── cognito/
      │   │   ├── cognito.service.ts
      │   │   └── cognito.module.ts
      │   ├── s3/
      │   │   ├── s3.service.ts
      │   │   └── s3.module.ts
      └── external-apis/         # Third-party integrations
          ├── payment-gateways/
          └── sms-service/
```

#### Frontend Structure
```yaml
frontend_directory_structure: |
  src/
  ├── assets/                    # Static assets
  │   ├── fonts/                 # Font files (Arabic, Latin)
  │   ├── images/                # Image files
  │   ├── icons/                 # Icon files
  │   └── locales/               # Translation files
  │       ├── ar/                # Arabic translations
  │       │   ├── common.json
  │       │   ├── patients.json
  │       │   └── appointments.json
  │       └── fr/                # French translations
  │           ├── common.json
  │           ├── patients.json
  │           └── appointments.json
  ├── components/                # Shared UI components
  │   ├── ui/                    # shadcn/ui components
  │   │   ├── button.tsx
  │   │   ├── input.tsx
  │   │   ├── card.tsx
  │   │   └── index.ts
  │   ├── layouts/               # Layout components
  │   │   ├── AuthLayout.tsx     # Auth pages layout
  │   │   ├── DashboardLayout.tsx # Dashboard layout
  │   │   ├── SingleDoctorLayout.tsx # Simplified layout
  │   │   └── index.ts
  │   ├── forms/                 # Reusable form components
  │   │   ├── FormField.tsx
  │   │   ├── FormErrors.tsx
  │   │   └── index.ts
  │   └── common/                # Common components
  │       ├── LoadingSpinner.tsx
  │       ├── ErrorBoundary.tsx
  │       ├── Breadcrumbs.tsx
  │       └── index.ts
  ├── config/                    # Configuration files
  │   ├── app.config.ts          # Application settings
  │   ├── routes.config.ts       # Route definitions
  │   ├── api.config.ts          # API configuration
  │   └── cognito.config.ts      # AWS Cognito client config
  ├── hooks/                     # Shared custom hooks
  │   ├── useClickOutside.ts
  │   ├── useLocalStorage.ts
  │   ├── useMediaQuery.ts
  │   ├── useAuth.ts             # Cognito authentication hook
  │   └── index.ts
  ├── lib/                       # Core utilities and services
  │   ├── api/                   # API client setup
  │   │   ├── client.ts          # Base API client with Cognito integration
  │   │   ├── interceptors.ts    # Request/response interceptors
  │   │   ├── error-handling.ts  # Error handling utilities
  │   │   └── index.ts
  │   ├── auth/                  # Authentication utilities
  │   │   ├── cognito.ts         # AWS Cognito service
  │   │   ├── tokens.ts          # JWT token handling
  │   │   └── index.ts
  │   ├── store.ts               # Global Zustand store
  │   ├── query.ts               # TanStack Query setup
  │   └── utils/                 # Utility functions
  │       ├── date.utils.ts
  │       ├── validation.utils.ts
  │       ├── format.utils.ts
  │       └── index.ts
  ├── providers/                 # Context providers
  │   ├── AppProvider.tsx        # Root provider
  │   ├── AuthProvider.tsx       # Cognito auth provider
  │   ├── ThemeProvider.tsx      # Theme context provider
  │   ├── QueryProvider.tsx      # TanStack Query provider
  │   └── index.ts
  ├── routes/                    # Routing setup
  │   ├── AppRoutes.tsx          # Main routing configuration
  │   ├── ProtectedRoute.tsx     # Route guard component
  │   ├── PublicRoute.tsx        # Public route component
  │   └── index.ts
  ├── features/                  # Feature modules
  │   ├── patients/              # Patient management feature
  │   │   ├── api/               # API integration
  │   │   │   ├── patients.api.ts
  │   │   │   ├── patients.types.ts
  │   │   │   └── index.ts
  │   │   ├── components/        # Feature components
  │   │   │   ├── PatientList.tsx
  │   │   │   ├── PatientCard.tsx
  │   │   │   ├── PatientForm.tsx
  │   │   │   ├── PatientFilter.tsx
  │   │   │   └── index.ts
  │   │   ├── hooks/             # Feature hooks
  │   │   │   ├── usePatients.ts
  │   │   │   ├── usePatient.ts
  │   │   │   ├── usePatientMutations.ts
  │   │   │   └── index.ts
  │   │   ├── store/             # Feature state
  │   │   │   ├── patients.store.ts
  │   │   │   └── index.ts
  │   │   ├── utils/             # Feature utilities
  │   │   │   ├── patient.utils.ts
  │   │   │   └── index.ts
  │   │   ├── pages/             # Page components
  │   │   │   ├── PatientListPage.tsx
  │   │   │   ├── PatientDetailPage.tsx
  │   │   │   ├── PatientEditPage.tsx
  │   │   │   ├── PatientCreatePage.tsx
  │   │   │   └── index.ts
  │   │   └── index.ts           # Public API
  │   ├── appointments/          # Appointment scheduling feature
  │   │   └── [similar structure]
  │   ├── auth/                  # Authentication feature
  │   │   └── [similar structure]
  │   └── dashboard/             # Dashboard feature
  │       └── [similar structure]
  ├── types/                     # Global TypeScript types
  │   ├── api.types.ts
  │   ├── auth.types.ts
  │   ├── common.types.ts
  │   └── index.ts
  ├── App.tsx                    # Root application component
  ├── main.tsx                   # Application entry point
  └── index.css                  # Entry CSS file

infrastructure_structure: |
  infra/
  ├── environments/
  │   ├── dev/
  │   │   ├── main.tf
  │   │   ├── variables.tf
  │   │   └── terraform.tfvars
  │   ├── staging/
  │   │   ├── main.tf
  │   │   ├── variables.tf
  │   │   └── terraform.tfvars
  │   └── prod/
  │       ├── main.tf
  │       ├── variables.tf
  │       └── terraform.tfvars
  ├── modules/
  │   ├── vpc/
  │   │   ├── main.tf
  │   │   ├── variables.tf
  │   │   └── outputs.tf
  │   ├── rds/
  │   │   ├── main.tf
  │   │   ├── variables.tf
  │   │   └── outputs.tf
  │   ├── ecs/
  │   │   ├── main.tf
  │   │   ├── variables.tf
  │   │   └── outputs.tf
  │   ├── cognito/
  │   │   ├── main.tf
  │   │   ├── variables.tf
  │   │   └── outputs.tf
  │   ├── s3/
  │   │   ├── main.tf
  │   │   ├── variables.tf
  │   │   └── outputs.tf
  │   ├── cloudwatch/
  │   │   ├── main.tf
  │   │   ├── variables.tf
  │   │   └── outputs.tf
  │   └── elasticache/
  │       ├── main.tf
  │       ├── variables.tf
  │       └── outputs.tf
  ├── scripts/
  │   ├── deploy.sh
  │   ├── destroy.sh
  │   └── backup.sh
  └── README.md
```

### Architectural Patterns Implementation

#### Backend Patterns
```yaml
module_organization:
  - "One module per business domain (patients, appointments, providers)"
  - "Feature modules are self-contained with controllers, services, repositories"
  - "Shared modules for cross-cutting concerns (logging, auth, database)"
  - "Infrastructure modules for external system integrations"

controller_pattern:
  - "Thin controllers handling only routing and request/response"
  - "Use DTOs with class-validator for input validation"
  - "Leverage NestJS guards for authentication and authorization"
  - "Support pagination, filtering, and sorting via query parameters"
  - "RESTful endpoint design following URL standardization guide"

service_pattern:
  - "All business logic encapsulated in services"
  - "Return Result<T, E> objects for error handling"
  - "Use dependency injection for repository and external service access"
  - "Implement transactions for multi-step operations"
  - "Log critical business events for audit trail"

repository_pattern:
  - "One repository per domain entity"
  - "Abstract TypeORM implementation details"
  - "Support complex queries with query builder"
  - "Implement soft deletes for data integrity"
  - "Handle pagination and sorting at repository level"

error_handling:
  - "Global exception filter for consistent error responses"
  - "Custom business exception classes"
  - "Result pattern to avoid throwing exceptions in business logic"
  - "Structured error responses with internationalization support"
```

#### Frontend Patterns
```yaml
state_management_strategy:
  local_state:
    - "React useState/useReducer for UI-only state"
    - "Form inputs, modal visibility, component-specific toggles"
    - "Temporary state that doesn't need persistence"
  
  feature_stores:
    - "Zustand stores for feature-specific shared state"
    - "Filters, pagination, cached manipulated data"
    - "State shared between multiple components in same feature"
  
  global_store:
    - "User authentication state and profile"
    - "Application settings (theme, language, clinic mode)"
    - "Global notifications and alerts"
    - "Current clinic/doctor context"
  
  server_state:
    - "TanStack Query for all API data fetching"
    - "Automatic background updates and caching"
    - "Loading and error state management"
    - "Optimistic updates for better UX"

component_architecture:
  - "Feature-based organization with clear boundaries"
  - "Shared components in global components folder"
  - "Page components act as containers coordinating features"
  - "Layouts handle common UI structure and navigation"
  - "Each feature exposes public API through index.ts"

performance_optimization:
  - "Route-based code splitting with React.lazy"
  - "Component memoization with React.memo for expensive renders"
  - "Virtualization for large lists (patient lists, appointment schedules)"
  - "Image optimization and lazy loading"
  - "Bundle size monitoring and optimization"
```

### Integration Architecture

#### AWS Cognito Integration
```yaml
authentication_flow:
  - "AWS Cognito User Pools for user management"
  - "Custom attributes for medical licenses and specialties"
  - "User groups for role-based access (doctor, admin, staff)"
  - "JWT tokens with custom claims for scopes"
  - "Automatic token refresh handling"

authorization_strategy:
  - "Scope-based authorization using Cognito custom attributes"
  - "Backend guards validate scopes for each endpoint"
  - "Frontend route guards based on user permissions"
  - "Role inheritance following standardized hierarchy"
```

## Code Standards

### Naming Conventions
```yaml
# Backend
backend_folders_files: "lowercase with dashes (e.g., user-profile, auth-token)"
backend_classes: "PascalCase (e.g., UserService, PatientsController)"
backend_variables_functions: "camelCase"
backend_constants: "UPPER_SNAKE_CASE"
database_tables: "snake_case"
api_endpoints: "kebab-case with RESTful conventions"

# Frontend  
frontend_components: "PascalCase (e.g., PatientProfile.tsx)"
frontend_variables_functions: "camelCase"
frontend_files: "PascalCase (React components), kebab-case (utilities)"
frontend_constants: "UPPER_SNAKE_CASE"

# Shared
git_branches: "<type>:<branch-name> (e.g., feature:patient-registration)"
git_commits: "<type>(<scope>): <short summary>"
```

### Quality Standards
- **Testing:** 
  - Minimum 80% code coverage
  - Unit tests for services, repositories, and utilities
  - Integration tests for API endpoints and database operations
  - E2E tests for critical workflows (patient registration, appointment booking)
  - Performance testing with React DevTools Profiler

- **Performance:** 
  - Optimal page loading
  - Fast API responses
  - Efficient database queries
  - High system availability
  - Lazy loading for non-critical features

- **Accessibility:** 
  - WCAG 2.1 AA compliance
  - Semantic HTML elements
  - Proper ARIA attributes
  - Keyboard navigation support
  - Screen reader compatibility
  - RTL layout support for Arabic

- **Security:** 
  - GDPR compliance for data protection
  - Secure data handling and protection
  - OAuth2 scope-based authorization
  - Input validation and sanitization
  - Regular security audits and dependency updates

- **Internationalization:**
  - Arabic and French language support
  - RTL layout support with CSS logical properties
  - Cultural localization for Algerian healthcare practices
  - Number and date formatting with Intl API
  - Lazy loading of translation files

### Code Style Examples
```typescript
// Backend Service Pattern
@Injectable()
export class PatientService {
  constructor(
    private readonly patientRepository: PatientRepository,
    private readonly logger: Logger
  ) {}

  /**
   * Creates a new patient record
   * @param data Patient creation data
   * @returns Promise resolving to created patient or validation error
   */
  async createPatient(data: CreatePatientDto): Promise<Result<Patient, ValidationError>> {
    try {
      // Validation
      const validation = await this.validatePatientData(data);
      if (validation.isError) {
        return Result.error(validation.error);
      }
      
      // Business logic
      const patient = await this.patientRepository.create(data);
      this.logger.log(`Patient created: ${patient.id}`);
      
      return Result.success(patient);
    } catch (error) {
      this.logger.error('Patient creation failed', error);
      return Result.error(new ValidationError('Patient creation failed'));
    }
  }
}

// Frontend Component Pattern
interface PatientProfileProps {
  patientId: string;
  onUpdate?: (patient: Patient) => void;
}

export const PatientProfile: React.FC<PatientProfileProps> = ({ 
  patientId, 
  onUpdate 
}) => {
  const { data: patient, isLoading, error } = useQuery({
    queryKey: ['patient', patientId],
    queryFn: () => patientApi.getById(patientId)
  });

  const { t } = useTranslation('patients');

  if (isLoading) return <PatientProfileSkeleton />;
  if (error) return <ErrorMessage error={error} />;
  if (!patient) return <NotFound />;

  return (
    <Card>
      <CardHeader>
        <CardTitle>{patient.firstName} {patient.lastName}</CardTitle>
      </CardHeader>
      <CardContent>
        {/* Component content */}
      </CardContent>
    </Card>
  );
};

// API Client Pattern
class PatientApiClient {
  private readonly baseUrl = '/api/patients';

  async getById(id: string): Promise<Patient> {
    const response = await apiClient.get(`${this.baseUrl}/${id}`);
    return response.data;
  }

  async create(data: CreatePatientRequest): Promise<Patient> {
    const response = await apiClient.post(this.baseUrl, data);
    return response.data;
  }
}

// Error Handling Pattern
const result = await patientService.createPatient(patientData);
if (result.isError) {
  // Handle specific error types
  if (result.error instanceof ValidationError) {
    return handleValidationError(result.error);
  }
  return handleGenericError(result.error);
}
// Use success result
const patient = result.data;
```

## Domain Model

### Core Entities
```yaml
entities:
  # Core Business Entities
  Patient:
    - id: UUID
    - firstName: string
    - lastName: string
    - middleName: string (optional)
    - dateOfBirth: Date
    - gender: Gender
    - status: PatientStatus
    - contactInfo: ContactInfo
    - emergencyContacts: EmergencyContact[]
    - medicalHistory: MedicalHistory
    - preferredLanguage: string (ar/fr)
    - preferredContactMethod: ContactMethod
    - notes: string (optional)
    - clinicId: UUID
    - createdAt: DateTime
    - updatedAt: DateTime
    - createdBy: UUID
    - updatedBy: UUID
    
  Provider:
    - id: UUID
    - firstName: string
    - lastName: string
    - middleName: string (optional)
    - title: string (Dr., Prof., etc.)
    - credentials: ProviderCredentials
    - specialties: string[]
    - status: ProviderStatus
    - schedule: ProviderSchedule
    - contactInfo: ContactInfo
    - emergencyContact: EmergencyContact
    - hireDate: Date
    - notes: string (optional)
    - clinicId: UUID
    - createdAt: DateTime
    - updatedAt: DateTime
    
  Appointment:
    - id: UUID
    - patientId: UUID
    - providerId: UUID
    - clinicId: UUID
    - dateTime: DateTime
    - duration: number (minutes)
    - type: AppointmentType
    - status: AppointmentStatus
    - reason: string
    - notes: string (optional)
    - checkedInAt: DateTime (optional)
    - completedAt: DateTime (optional)
    - cancelledAt: DateTime (optional)
    - cancellationReason: string (optional)
    - waitTime: number (minutes, optional)
    - createdAt: DateTime
    - updatedAt: DateTime
    - createdBy: UUID
    - updatedBy: UUID
    
  Prescription:
    - id: UUID
    - patientId: UUID
    - providerId: UUID
    - appointmentId: UUID (optional)
    - clinicId: UUID
    - medication: PrescriptionMedication
    - status: PrescriptionStatus
    - prescribedDate: DateTime
    - expirationDate: DateTime
    - pharmacy: Pharmacy (optional)
    - interactions: DrugInteraction[]
    - allergiesChecked: boolean
    - notes: string (optional)
    - createdAt: DateTime
    - updatedAt: DateTime
    
  User:
    - id: UUID
    - cognitoId: string (AWS Cognito user ID)
    - email: string
    - firstName: string
    - lastName: string
    - role: UserRole
    - scopes: Scope[]
    - isActive: boolean
    - clinicId: UUID
    - providerId: UUID (optional, if user is a provider)
    - lastLoginAt: DateTime (optional)
    - preferences: UserPreferences
    - createdAt: DateTime
    - updatedAt: DateTime
    
  Clinic:
    - id: UUID
    - name: string
    - mode: ClinicMode
    - address: Address
    - contactInfo: ContactInfo
    - license: string
    - taxId: string
    - settings: ClinicSettings
    - subscriptionPlan: SubscriptionPlan
    - isActive: boolean
    - createdAt: DateTime
    - updatedAt: DateTime

  # Contact & Address Entities
  ContactInfo:
    - phone: string
    - email: string (optional)
    - address: Address
    - preferredContactMethod: ContactMethod
    
  Address:
    - street: string
    - city: string
    - state: string
    - zipCode: string
    - country: string (default: DZ)
    - coordinates: GeoCoordinates (optional)
    
  EmergencyContact:
    - name: string
    - relationship: RelationshipType
    - phone: string
    - email: string (optional)
    - address: Address (optional)
    
  # Medical Entities
  MedicalHistory:
    - patientId: UUID
    - pastMedicalHistory: MedicalCondition[]
    - currentMedications: Medication[]
    - allergies: Allergy[]
    - familyHistory: FamilyMedicalHistory[]
    - socialHistory: SocialHistory
    - immunizations: Immunization[]
    - updatedAt: DateTime
    - updatedBy: UUID
    
  MedicalCondition:
    - id: UUID
    - condition: string
    - icdCode: string (optional)
    - diagnosedDate: Date (optional)
    - status: ConditionStatus
    - severity: Severity (optional)
    - notes: string (optional)
    - diagnosedBy: string (optional)
    
  Medication:
    - id: UUID
    - name: string
    - genericName: string (optional)
    - dosage: string
    - frequency: string
    - route: string
    - startDate: Date
    - endDate: Date (optional)
    - prescribedBy: string
    - status: MedicationStatus
    - notes: string (optional)
    
  Allergy:
    - id: UUID
    - allergen: string
    - allergenType: AllergenType
    - severity: AllergySeverity
    - reaction: string
    - diagnosedDate: Date (optional)
    - notes: string (optional)
    
  FamilyMedicalHistory:
    - id: UUID
    - relationship: RelationshipType
    - condition: string
    - ageAtDiagnosis: number (optional)
    - ageAtDeath: number (optional)
    - causeOfDeath: string (optional)
    - notes: string (optional)
    
  SocialHistory:
    - smokingStatus: SmokingStatus
    - alcoholUse: AlcoholUse
    - exerciseFrequency: string (optional)
    - occupation: string (optional)
    - maritalStatus: MaritalStatus
    - notes: string (optional)
    
  Immunization:
    - id: UUID
    - vaccine: string
    - dateAdministered: Date
    - nextDueDate: Date (optional)
    - provider: string (optional)
    - lotNumber: string (optional)
    - siteAdministered: string (optional)
    - notes: string (optional)

  
  # Provider-Related Entities
  ProviderCredentials:
    - medicalLicense: string
    - licenseState: string
    - licenseExpiration: Date
    - specialtyLicenses: string[]
    
  ProviderSchedule:
    - providerId: UUID
    - weeklySchedule: DaySchedule[]
    - appointmentTypes: AppointmentTypeConfig[]
    - breakTimes: BreakTime[]
    - timeOff: TimeOff[]
    - workingHours: WorkingHours
    - bufferTime: number (minutes between appointments)
    - updatedAt: DateTime
    
  DaySchedule:
    - dayOfWeek: DayOfWeek
    - isAvailable: boolean
    - startTime: string (HH:mm format)
    - endTime: string (HH:mm format)
    - breaks: BreakTime[]
    
  AppointmentTypeConfig:
    - id: UUID
    - name: string
    - duration: number (minutes)
    - color: string (hex color)
    - description: string (optional)
    - isActive: boolean
    - requiresPreparation: boolean
    - preparationInstructions: string (optional)
    
  BreakTime:
    - id: UUID
    - dayOfWeek: DayOfWeek
    - startTime: string (HH:mm format)
    - endTime: string (HH:mm format)
    - type: BreakType
    - description: string (optional)
    
  TimeOff:
    - id: UUID
    - providerId: UUID
    - startDate: Date
    - endDate: Date
    - type: TimeOffType
    - reason: string (optional)
    - isApproved: boolean
    - approvedBy: UUID (optional)
    - createdAt: DateTime

  # Prescription-Related Entities
  PrescriptionMedication:
    - name: string
    - genericName: string (optional)
    - strength: string
    - form: MedicationForm
    - dosage: string
    - frequency: string
    - quantity: number
    - refills: number
    - instructions: string
    - ndcCode: string (optional)
    - rxNorm: string (optional)
    
  Pharmacy:
    - id: UUID
    - name: string
    - phone: string
    - address: Address
    - ncpdpId: string (optional)
    - isPreferred: boolean
    
  DrugInteraction:
    - id: UUID
    - interactingDrug: string
    - severity: InteractionSeverity
    - description: string
    - clinicalEffect: string
    - recommendation: string
    - reference: string (optional)
    
  AllergyAlert:
    - id: UUID
    - prescriptionId: UUID
    - allergen: string
    - severity: AllergySeverity
    - reaction: string
    - recommendation: string
    - overridden: boolean
    - overrideReason: string (optional)

  # Clinical Documentation
  ClinicalDocumentation:
    - id: UUID
    - patientId: UUID
    - providerId: UUID
    - appointmentId: UUID
    - type: DocumentationType
    - chiefComplaint: string
    - historyOfPresentIllness: string (optional)
    - reviewOfSystems: string (optional)
    - physicalExamination: PhysicalExamination (optional)
    - assessment: Assessment
    - plan: TreatmentPlan
    - notes: string (optional)
    - isSigned: boolean
    - signedAt: DateTime (optional)
    - signedBy: UUID (optional)
    - createdAt: DateTime
    - updatedAt: DateTime
    
  VitalSigns:
    - id: UUID
    - patientId: UUID
    - appointmentId: UUID (optional)
    - bloodPressureSystolic: number (optional)
    - bloodPressureDiastolic: number (optional)
    - heartRate: number (optional)
    - temperature: number (optional)
    - respiratoryRate: number (optional)
    - oxygenSaturation: number (optional)
    - weight: number (optional)
    - height: number (optional)
    - bmi: number (calculated)
    - painLevel: number (0-10 scale, optional)
    - measuredBy: UUID
    - measuredAt: DateTime
    
  PhysicalExamination:
    - general: string (optional)
    - cardiovascular: string (optional)
    - respiratory: string (optional)
    - gastrointestinal: string (optional)
    - neurological: string (optional)
    - musculoskeletal: string (optional)
    - skin: string (optional)
    - other: string (optional)
    
  Assessment:
    - primaryDiagnosis: Diagnosis
    - secondaryDiagnoses: Diagnosis[]
    - differentialDiagnoses: string[]
    - clinicalImpression: string (optional)
    
  Diagnosis:
    - code: string (ICD-10)
    - description: string
    - type: DiagnosisType
    - confidence: ConfidenceLevel
    - notes: string (optional)
    
  TreatmentPlan:
    - medications: PrescriptionMedication[]
    - procedures: string[]
    - followUpInstructions: string (optional)
    - nextAppointment: string (optional)
    - referrals: string[]
    - lifestyle: string (optional)
    - patientEducation: string (optional)

  # Billing & Payment
  Bill:
    - id: UUID
    - patientId: UUID
    - appointmentId: UUID (optional)
    - clinicId: UUID
    - amount: number
    - currency: string (DZD)
    - services: BillItem[]
    - status: BillStatus
    - dueDate: Date
    - paidDate: Date (optional)
    - notes: string (optional)
    - createdAt: DateTime
    - updatedAt: DateTime
    
  BillItem:
    - id: UUID
    - description: string
    - quantity: number
    - unitPrice: number
    - totalPrice: number
    - serviceCode: string (optional)
    
  Payment:
    - id: UUID
    - billId: UUID
    - patientId: UUID
    - amount: number
    - currency: string (DZD)
    - method: PaymentMethod
    - status: PaymentStatus
    - transactionId: string (optional)
    - processedAt: DateTime (optional)
    - notes: string (optional)
    - createdAt: DateTime

  # Waiting List & Scheduling
  WaitingListEntry:
    - id: UUID
    - patientId: UUID
    - providerId: UUID (optional)
    - clinicId: UUID
    - preferredDates: Date[]
    - preferredTimes: string[]
    - priority: Priority
    - reason: string
    - appointmentType: AppointmentType
    - duration: number (minutes)
    - contactPreference: ContactMethod
    - waitingSince: DateTime
    - lastContacted: DateTime (optional)
    - status: WaitingListStatus
    - notes: string (optional)

  # System & Configuration
  UserPreferences:
    - language: string (ar/fr)
    - theme: string (light/dark)
    - timezone: string
    - dateFormat: string
    - timeFormat: string (12h/24h)
    - notifications: NotificationPreferences
    
  NotificationPreferences:
    - email: boolean
    - sms: boolean
    - inApp: boolean
    - appointmentReminders: boolean
    - prescriptionAlerts: boolean
    - labResults: boolean
    
  ClinicSettings:
    - operatingHours: WorkingHours
    - timeZone: string
    - defaultAppointmentDuration: number (minutes)
    - bufferTime: number (minutes)
    - maxAdvanceBooking: number (days)
    - allowWalkIns: boolean
    - reminderSettings: ReminderSettings
    - billingSettings: BillingSettings
    
  ReminderSettings:
    - emailReminders: boolean
    - smsReminders: boolean
    - reminderTiming: number[] (hours before appointment)
    - templates: NotificationTemplate[]
    
  BillingSettings:
    - currency: string (DZD)
    - taxRate: number (percentage)
    - paymentMethods: PaymentMethod[]
    - invoiceTemplate: string
    - autoGenerateInvoices: boolean
    
  # Audit & Logging
  AuditLog:
    - id: UUID
    - entityType: string
    - entityId: UUID
    - action: AuditAction
    - oldValues: JSON (optional)
    - newValues: JSON (optional)
    - userId: UUID
    - clinicId: UUID
    - ipAddress: string (optional)
    - userAgent: string (optional)
    - timestamp: DateTime
    
  # Notifications
  Notification:
    - id: UUID
    - userId: UUID (optional)
    - patientId: UUID (optional)
    - type: NotificationType
    - title: string
    - message: string
    - data: JSON (optional)
    - status: NotificationStatus
    - method: ContactMethod
    - sentAt: DateTime (optional)
    - readAt: DateTime (optional)
    - createdAt: DateTime
```

### Enums
```yaml
enums:
  # Core Status Enums
  PatientStatus:
    - active
    - inactive  
    - deceased
    - merged
    
  AppointmentStatus:
    - scheduled
    - confirmed
    - checked_in
    - in_progress
    - completed
    - billed
    - cancelled
    - no_show
    - rescheduled
    
  ProviderStatus:
    - active
    - inactive
    - on_leave
    - terminated
    
  PaymentStatus:
    - pending
    - partial
    - paid
    - overdue
    - written_off
    - refunded
    
  BillStatus:
    - draft
    - pending
    - sent
    - partial_payment
    - paid
    - overdue
    - cancelled
    
  PrescriptionStatus:
    - active
    - expired
    - cancelled
    - renewed
    - discontinued
    
  # User & Role Enums
  UserRole:
    - system_administrator
    - clinic_manager
    - healthcare_provider
    - clinical_staff
    - administrative_staff
    
  Scope:
    - admin_full
    - clinic_manage
    - patient_read
    - patient_write
    - appointment_read
    - appointment_write
    - clinical_read
    - clinical_write
    - prescription_read
    - prescription_write
    - billing_read
    - billing_write
    
  # Demographics & Contact Enums
  Gender:
    - male
    - female
    - other
    
  ContactMethod:
    - phone
    - email
    - sms
    - whatsapp
    
  RelationshipType:
    - spouse
    - child
    - parent
    - sibling
    - guardian
    - grandparent
    - grandchild
    - other
    
  MaritalStatus:
    - single
    - married
    - divorced
    - widowed
    - separated
    
  # Medical Enums
  AllergySeverity:
    - mild
    - moderate
    - severe
    - life_threatening
    
  AllergenType:
    - medication
    - food
    - environmental
    - contact
    - unknown
    
  MedicationStatus:
    - active
    - discontinued
    - completed
    - on_hold
    
  ConditionStatus:
    - active
    - resolved
    - chronic
    - in_remission
    
  SmokingStatus:
    - never
    - former
    - current
    - unknown
    
  AlcoholUse:
    - none
    - occasional
    - moderate
    - heavy
    - unknown
    
  MedicationForm:
    - tablet
    - capsule
    - liquid
    - injection
    - topical
    - inhaler
    - suppository
    - patch
    - other
    
  # Appointment & Scheduling Enums
  AppointmentType:
    - consultation
    - follow_up
    - urgent_care
    - physical_exam
    - vaccination
    - procedure
    - telemedicine
    - other
    
  Priority:
    - low
    - medium
    - high
    - urgent
    
  DayOfWeek:
    - sunday
    - monday
    - tuesday
    - wednesday
    - thursday
    - friday
    - saturday
    
  BreakType:
    - lunch
    - break
    - meeting
    - emergency
    - other
    
  TimeOffType:
    - vacation
    - sick_leave
    - personal
    - training
    - conference
    - emergency
    
  WaitingListStatus:
    - active
    - contacted
    - scheduled
    - expired
    - cancelled
    
  # Clinical Documentation Enums
  DocumentationType:
    - consultation_note
    - progress_note
    - discharge_summary
    - procedure_note
    - referral_letter
    - lab_order
    - prescription
    
  DiagnosisType:
    - primary
    - secondary
    - differential
    - provisional
    
  ConfidenceLevel:
    - definitive
    - probable
    - possible
    - rule_out
    
  Severity:
    - mild
    - moderate
    - severe
    - critical
    
  # Drug Interaction Enums
  InteractionSeverity:
    - minor
    - moderate
    - major
    - contraindicated
    
  # Payment & Billing Enums
  PaymentMethod:
    - cash
    - card
    - bank_transfer
    - check
    - installment
    
  # Clinic & System Enums
  ClinicMode:
    - single_doctor
    - multi_provider
    
  SubscriptionPlan:
    - starter
    - professional
    - enterprise
    - trial
    
  NotificationType:
    - appointment_reminder
    - appointment_confirmation
    - prescription_ready
    - lab_results
    - payment_due
    - system_alert
    - promotional
    
  NotificationStatus:
    - pending
    - sent
    - delivered
    - failed
    - read
    
  AuditAction:
    - create
    - update
    - delete
    - view
    - login
    - logout
    - export
    - import
```

## Current Epics

### Planned Epics (Priority Order)
- **[EPIC-001]:** Core Patient Management - Registration, profiles, search optimized for solo practitioners
- **[EPIC-002]:** Simplified Appointment Scheduling - Streamlined booking and management for single-doctor workflow
- **[EPIC-003]:** Essential Clinical Documentation - Basic medical records and consultation notes
- **[EPIC-004]:** Payment Integration - Algerian payment methods and basic billing
- **[EPIC-005]:** Mobile-Responsive Interface - Tablet/mobile optimization for on-the-go access
- **[EPIC-006]:** Multi-Provider Mode - Scaling features for small clinics
- **[EPIC-007]:** Advanced Clinical Features - Prescription management, medical history
- **[EPIC-008]:** Analytics & Reporting - Practice performance and patient insights
- **[EPIC-009]:** Integration Hub - Lab systems and government health platform connections

## Integration Points

### External APIs
```yaml
external_services:
  - service: "Local Payment Gateways (CIB, Satim)"
    purpose: "Algerian banking system integration for payments"
    auth: "Bank-specific API credentials"
    docs: "Local banking API documentation"
    
  - service: "Twilio Communications"
    purpose: "SMS notifications and appointment reminders"
    auth: "Account SID and Auth Token"
    docs: "https://www.twilio.com/docs"
    
  - service: "AWS S3"
    purpose: "Document storage and patient file management"
    auth: "IAM roles and policies"
    docs: "https://docs.aws.amazon.com/s3/"
    
```

### Internal Services
```yaml
internal_apis:
  - service: "User Management Service"
    responsibility: "User profile management, role assignment, Cognito integration"
    endpoints: "/users, /users/{id}/profile, /users/{id}/roles"
    
  - service: "Patient Service"
    responsibility: "Patient CRUD operations, medical history"
    endpoints: "/patients, /patients/{id}, /patients/{id}/medical-history"
    
  - service: "Appointment Service"
    responsibility: "Scheduling, status management, provider availability"
    endpoints: "/appointments, /appointments/daily, /appointments/waiting-list"
    
  - service: "Clinical Service"
    responsibility: "Medical documentation, prescriptions, clinical workflows"
    endpoints: "/clinical/documentation, /prescriptions, /prescriptions/renewals"
    
  - service: "Billing Service"
    responsibility: "Payment processing, billing management, local payment integration"
    endpoints: "/billing, /payments, /payments/check-in"
    
  - service: "Notification Service"
    responsibility: "SMS, email, and in-app notifications"
    endpoints: "/notifications, /notifications/send, /notifications/templates"
```
