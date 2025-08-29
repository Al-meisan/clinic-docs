# EPIC: EPIC-001 - Core Patient Management

## Epic Overview

### Goals & Value

**Business Goal:** Establish the foundational patient management system that digitalizes patient records and streamlines patient data workflows for Algerian healthcare providers, starting with single-doctor practices.

**User Value:** 
- Healthcare providers can efficiently register, search, and manage patient information
- Administrative staff can quickly access and update patient profiles
- Patients benefit from accurate, centralized medical records and faster check-in processes
- Clinic operations become more organized with digital patient database

**Success Criteria:** 
- 90% reduction in patient registration time compared to paper-based systems
- Patient searches return results promptly
- Zero data loss during patient profile updates
- 95% user satisfaction with patient search and registration workflows
- Support for Arabic and French patient data entry and display

### Scope

**In Scope:**
- Complete patient registration workflow with demographics, contact info, and emergency contacts
- Comprehensive patient search with multiple criteria (name, phone, DOB, patient ID)
- Patient profile management with edit capabilities and audit trail
- billing
- Patient status management (active, inactive, deceased, merged)
- Real-time patient data validation and duplicate detection
- Multi-language support (Arabic/French) for patient information
- Single Doctor Mode optimized interface with simplified workflows

**Out of Scope:**
- Medical history management (covered in EPIC-003)
- Appointment scheduling integration (covered in EPIC-002)
- Billing and payment processing (covered in EPIC-004)
- Multi-provider clinic features (covered in EPIC-006)
- Patient portal or self-service features
- Integration with external medical systems

## User Stories & Requirements

### Primary User Stories

```yaml
story_1:
  as_a: "Administrative Staff"
  i_want: "to register new patients quickly with all required information"
  so_that: "patients can be scheduled for appointments and receive care without delays"
  acceptance_criteria:
    - "Complete patient registration quickly"
    - "Automatic generation of unique patient ID"
    - "Validation prevents duplicate patient creation"
    - "Support for Arabic and French data entry"
    - "Photo capture for patient identification (optional)"

story_2:
  as_a: "Healthcare Provider"
  i_want: "to quickly search and find patient records during consultations"
  so_that: "I can access patient information efficiently and provide appropriate care"
  acceptance_criteria:
    - "Search by name, phone, DOB, or patient ID"
    - "Results displayed promptly"
    - "Recent patients list for quick access"
    - "Filter options for active/inactive patients"
    - "Clear patient identification with photos"

story_3:
  as_a: "Clinical Staff"
  i_want: "to update patient contact information when changes occur"
  so_that: "clinic can maintain accurate communication with patients"
  acceptance_criteria:
    - "Edit all contact fields including address, phone, email"
    - "Update emergency contact information"
    - "Changes are immediately reflected across the system"
    - "Audit trail shows who made changes and when"
    - "Validation prevents invalid contact information"

```

### Detailed Requirements

1. **Patient Registration System**
   - Description: Complete digital patient onboarding with all required demographic and contact information
   - Priority: HIGH
   - Dependencies: User authentication system, database infrastructure

2. **Advanced Patient Search**
   - Description: Multi-criteria search system with fuzzy matching and real-time results
   - Priority: HIGH
   - Dependencies: Database indexing, search optimization

3. **Patient Profile Management**
   - Description: Comprehensive patient information editing with validation and audit trail
   - Priority: HIGH
   - Dependencies: Patient registration system

4. **Patient Status Workflow**
   - Description: Manage patient lifecycle status with proper transitions and business rules
   - Priority: MEDIUM
   - Dependencies: Patient profile system, audit logging

5. **Multi-language Patient Data**
   - Description: Support Arabic and French for patient information display and data entry
   - Priority: HIGH
   - Dependencies: Internationalization framework

6. **Duplicate Detection System**
   - Description: Prevent duplicate patient creation using intelligent matching algorithms
   - Priority: MEDIUM
   - Dependencies: Patient search system

## Domain Model (Epic-Specific)

### Entities

```yaml
Patient:
  attributes:
    - id: UUID (auto-generated)
    - firstName: string
    - lastName: string
    - middleName: string (optional)
    - dateOfBirth: Date
    - gender: Gender enum
    - status: PatientStatus enum
    - preferredLanguage: string (ar/fr)
    - preferredContactMethod: ContactMethod enum
    - notes: string (optional)
    - clinicId: UUID
    - createdAt: DateTime
    - updatedAt: DateTime
    - createdBy: UUID
    - updatedBy: UUID
  relationships:
    - relates_to: "ContactInfo"
      type: "one-to-one"
    - relates_to: "EmergencyContact"
      type: "one-to-many"
  business_rules:
    - "Patient ID must be unique across the entire system"
    - "Date of birth cannot be in the future"
    - "First name, last name, date of birth, and phone are mandatory"
    - "Email must be unique if provided"
    - "Status transitions must follow defined workflow"

ContactInfo:
  attributes:
    - phone: string (required)
    - email: string (optional, unique)
    - address: Address
    - preferredContactMethod: ContactMethod enum
  relationships:
    - relates_to: "Patient"
      type: "one-to-one"
    - relates_to: "Address"
      type: "one-to-one"
  business_rules:
    - "Phone number must follow Algerian format validation"
    - "Email must be valid format if provided"
    - "Address must include at least street and city"

Address:
  attributes:
    - street: string
    - city: string
    - state: string
    - zipCode: string
    - country: string (default: DZ)
    - coordinates: GeoCoordinates (optional)
  business_rules:
    - "Street and city are mandatory fields"
    - "Country defaults to DZ (Algeria)"
    - "Coordinates are optional for mapping features"

EmergencyContact:
  attributes:
    - id: UUID
    - name: string
    - relationship: RelationshipType enum
    - phone: string
    - email: string (optional)
    - address: Address (optional)
    - isPrimary: boolean
  relationships:
    - relates_to: "Patient"
      type: "many-to-one"
  business_rules:
    - "Each patient must have at least one emergency contact"
    - "Primary emergency contact must be designated"
    - "Emergency contact phone is mandatory"

```

### Business Rules

```yaml
rules:
  - rule_id: "BR-P001"
    rule: "Every patient must have a unique patient ID generated automatically upon registration"
    applies_to: "Patient creation process"
    validation: "To ensure that the id is unique don't let the database auto generate it, generate a new UUID before user registration"
    
  - rule_id: "BR-P002"
    rule: "Patient first name, last name, date of birth, and phone number are mandatory fields"
    applies_to: "Patient registration and updates"
    validation: "Required field validation on form submission"
    
  - rule_id: "BR-P003"
    rule: "Patient email address must be unique across the system if provided"
    applies_to: "Patient registration and contact updates"
    validation: "Database uniqueness constraint on email field"
    
  - rule_id: "BR-P004"
    rule: "Patient date of birth cannot be in the future"
    applies_to: "Patient registration and update"
    validation: "Check if this date is in future on form submission"
    
  - rule_id: "BR-P006":
    rule: "New patients must be assigned 'active' status by default"
    applies_to: "Patient registration and update"
    validation: "In the patient registration thie value must be set implicitly, in the database this must be configured as the default value"
    
  - rule_id: "BR-P008"
    rule: "Patient status must follow valid transitions: active ↔ inactive, active → deceased, active → merged"
    applies_to: "Patient status management"
    validation: "State machine validation for status changes"
    
  - rule_id: "BR-P016"
    rule: "System must prevent duplicate patient creation based on name, DOB, and phone combination"
    applies_to: "Patient registration process"
    validation: "Fuzzy matching algorithm during registration"
    
  - rule_id: "BR-P017"
    rule: "Patient search must support name, phone, DOB, and patient ID criteria"
    applies_to: "Patient search functionality"
    validation: "Multi-field search implementation with proper indexing"
```

## Integration Points

### Dependencies on Other Epics

```yaml
dependencies:
  - epic: "Infrastructure Setup"
    dependency_type: "HARD"
    what_we_need: "AWS infrastructure, database, authentication system, i18n framework"
    
  - epic: "EPIC-002 (Appointment Scheduling)"
    dependency_type: "SOFT"
    what_we_need: "Patient context for appointment booking workflows"
    
  - epic: "EPIC-003 (Clinical Documentation)"
    dependency_type: "SOFT"
    what_we_need: "Patient profile integration for medical history access"
```

### APIs This Epic Provides

```yaml
provided_apis:
  - endpoint: "GET /api/patients"
    purpose: "Search and list patients with filtering and pagination"
    consumers: "Appointment scheduling, clinical documentation, billing systems"
    
  - endpoint: "POST /api/patients"
    purpose: "Create new patient records"
    consumers: "Patient registration interface, data import systems"
    
  - endpoint: "GET /api/patients/{patientId}"
    purpose: "Retrieve comprehensive patient information"
    consumers: "All other epics requiring patient data"
    
  - endpoint: "PUT /api/patients/{patientId}"
    purpose: "Update patient demographic and contact information"
    consumers: "Patient profile management, check-in processes"
```

### APIs This Epic Consumes

```yaml
consumed_apis:
  - source: "AWS Cognito"
    endpoints: "User authentication and authorization endpoints"
    purpose: "User authentication and scope-based access control"
```

## Technical Approach

### Architecture Decisions

```yaml
decisions:
  - decision: "Use feature-based modular architecture for patient management"
    rationale: "Enables clear separation of concerns and easier maintenance"
    impact: "Better code organization, easier testing, scalable development"
    
  - decision: "Implement fuzzy search with PostgreSQL full-text search and similarity functions"
    rationale: "Provides fast, accurate patient search with typo tolerance"
    impact: "Better search performance, reduced duplicate registrations"
    
  - decision: "Use optimistic UI updates with TanStack Query for patient profile edits"
    rationale: "Improves user experience with immediate feedback"
    impact: "Better UX, automatic error handling, consistent state management"
    
  - decision: "Implement automatic patient ID generation using UUID v4"
    rationale: "Ensures global uniqueness without collision risk"
    impact: "Eliminates ID conflicts, supports distributed systems"
```

### Performance Requirements

```yaml
performance:
  - metric: "Patient search response time"
    target: "Fast search query responses"
    measurement: "API response time monitoring"
    
  - metric: "Patient registration completion time"
    target: "Efficient registration process"
    measurement: "User interaction tracking from start to finish"
    
  - metric: "Database query performance"
    target: "Optimized patient lookup queries"
    measurement: "Database query monitoring and slow query logging"
    
  - metric: "Concurrent user support"
    target: "Support multiple concurrent users effectively"
    measurement: "Load testing and performance monitoring"
```

### Security Considerations

```yaml
security:
  - consideration: "Patient data security and protection"
    implementation: "AWS RDS security, HTTPS for all API calls"
    validation: "Security audit and penetration testing"
    
  - consideration: "Role-based access control for patient information"
    implementation: "OAuth2 scopes with patient:read/write permissions"
    validation: "Access control testing for different user roles"
    
  - consideration: "Audit trail for all patient data modifications"
    implementation: "Database triggers and application-level audit logging"
    validation: "Audit log completeness and integrity verification"
    
  - consideration: "Input validation and sanitization"
    implementation: "class-validator for DTOs, database parameter binding"
    validation: "Security scanning and input fuzzing tests"
```

## Current Tasks

- [ ] **[TASK-001]:** Patient Registration Backend API - Create NestJS module with DTOs, services, and controllers for patient registration
- [ ] **[TASK-002]:** Patient Search Implementation - Implement advanced search with PostgreSQL full-text search and fuzzy matching
- [ ] **[TASK-003]:** Patient Profile Management API - Create endpoints for patient profile updates with audit trail
- [ ] **[TASK-004]:** Patient Registration Frontend Interface - Build React components for patient registration form with validation
- [ ] **[TASK-005]:** Patient Search Frontend Interface - Create patient search components with real-time results and filtering
- [ ] **[TASK-006]:** Patient Profile Frontend Interface - Build patient profile view and edit components
- [ ] **[TASK-007]:** Duplicate Detection System - Implement intelligent duplicate prevention during registration
- [ ] **[TASK-008]:** Multi-language Support Integration - Implement Arabic/French support for patient data
- [ ] **[TASK-009]:** Patient Status Management - Create status workflow management with proper transitions
- [ ] **[TASK-010]:** Database Optimization - Implement proper indexing and query optimization for patient searches
- [ ] **[TASK-011]:** Integration Testing - Comprehensive testing of all patient management workflows
- [ ] **[TASK-012]:** Performance Optimization - Optimize search performance and database queries
- [ ] **[TASK-013]:** Security Implementation - Implement role-based access control and audit logging
