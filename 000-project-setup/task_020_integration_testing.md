# TASK-020: Integration Testing Framework

## Task Overview

### Goal
**What to Build:** Set up comprehensive end-to-end testing framework with proper test data management, environment isolation, and automated testing capabilities for the clinic management system.

**Why:** Critical for ensuring system reliability, preventing regressions, validating complex user workflows, and maintaining quality standards in a healthcare environment where system failures impact patient care.

**Definition of Done:** Complete testing framework operational with automated test suites, test data management, CI/CD integration, and comprehensive coverage of critical user workflows.

### Scope
```yaml
task_type: "FULLSTACK"
complexity: "HIGH"
```

## Requirements

### Functional Requirements
1. **End-to-End Test Framework**
   - Description: Automated testing framework covering complete user workflows
   - Acceptance Criteria: Tests covering patient registration, appointments, clinical documentation, and billing
   - Priority: HIGH

2. **Test Data Management System**
   - Description: Automated test data generation and cleanup for consistent testing
   - Acceptance Criteria: Reproducible test scenarios with isolated test data per test run
   - Priority: HIGH

3. **Multi-Environment Testing**
   - Description: Testing framework supporting development, staging, and production-like environments
   - Acceptance Criteria: Tests run consistently across different environments with environment-specific configurations
   - Priority: HIGH

4. **Performance and Load Testing**
   - Description: Automated performance testing for critical system operations
   - Acceptance Criteria: Load tests validating system performance under expected user loads
   - Priority: MEDIUM

5. **API Integration Testing**
   - Description: Comprehensive API testing covering all endpoints and integration points
   - Acceptance Criteria: All API endpoints tested with various scenarios including error conditions
   - Priority: HIGH

### Technical Requirements
```yaml
performance:
  - requirement: "Test suite execution time"
    target: "Reasonable CI/CD pipeline execution time"
    
  - requirement: "Test environment setup time"
    target: "Quick test environment provisioning"
    
security:
  - requirement: "Test data anonymization and privacy protection"
    implementation: "No real patient data in test environments"
    
  - requirement: "Secure test environment access"
    implementation: "Access controls and authentication for test environments"
    
compatibility:
  - requirement: "Cross-browser testing support"
    scope: "Tests run on Chrome, Firefox, Safari, and Edge"
    
  - requirement: "Mobile device testing"
    scope: "Responsive design testing on various screen sizes"
```

## Implementation Details

### Technical Approach
```yaml
# Testing Framework Architecture
testing_stack:
  - tool: "Playwright"
    purpose: "End-to-end browser automation and testing"
    features: "Cross-browser, mobile testing, API testing, visual comparisons"
    
  - tool: "Jest"
    purpose: "Test runner and assertion framework"
    features: "Unit tests, integration tests, mocking, coverage reporting"
    
  - tool: "Supertest"
    purpose: "API testing and HTTP assertions"
    features: "Express/NestJS integration, request/response testing"
    
  - tool: "Docker Compose"
    purpose: "Test environment containerization"
    features: "Isolated test environments, database containers, service orchestration"

# Test Data Management
test_data_strategy:
  - approach: "Factory Pattern"
    purpose: "Programmatic test data generation"
    implementation: "TypeScript factories for all domain entities"
    
  - approach: "Database Seeding"
    purpose: "Consistent test database state"
    implementation: "Automated database setup and teardown per test suite"
    
  - approach: "Mock Services"
    purpose: "External API mocking for consistent testing"
    implementation: "MSW (Mock Service Worker) for API mocking"
    
  - approach: "Test Data Cleanup"
    purpose: "Clean test environment after each test run"
    implementation: "Automated cleanup scripts and database rollback"

# Test Categories and Coverage
test_coverage:
  - category: "Unit Tests"
    scope: "Individual functions, components, and services"
    target: "80% code coverage minimum"
    
  - category: "Integration Tests"
    scope: "API endpoints, database operations, service interactions"
    target: "100% API endpoint coverage"
    
  - category: "End-to-End Tests"
    scope: "Complete user workflows and business processes"
    target: "Critical user journeys covered"
    
  - category: "Performance Tests"
    scope: "Load testing, stress testing, database performance"
    target: "Response time requirements validated"
```

### Integration Points
```yaml
dependencies:
  - dependency: "Backend API Implementation (TASK-003)"
    type: "API"
    status: "AVAILABLE"
    
  - dependency: "Frontend Application (TASK-007)"
    type: "APPLICATION"
    status: "AVAILABLE"
    
  - dependency: "Database Infrastructure (TASK-004)"
    type: "DATABASE"
    status: "AVAILABLE"
    
  - dependency: "CI/CD Pipeline (TASK-015)"
    type: "INFRASTRUCTURE"
    status: "AVAILABLE"
    
provides:
  - deliverable: "Automated Testing Framework"
    interface: "Test suites executable via CLI and CI/CD"
    consumers: "Development team, QA engineers, CI/CD pipeline"
    
  - deliverable: "Test Data Management Tools"
    interface: "Scripts and utilities for test data generation"
    consumers: "QA team, developers, automated testing pipeline"
    
  - deliverable: "Performance Testing Suite"
    interface: "Load testing scripts and performance benchmarks"
    consumers: "Performance engineers, DevOps team, stakeholders"
```

### Code Patterns to Follow
```typescript
// Test Data Factory Pattern
import { faker } from '@faker-js/faker';
import { Patient, Provider, Appointment } from '../src/entities';

export class PatientFactory {
  static create(overrides: Partial<Patient> = {}): Patient {
    const patient = new Patient();
    patient.id = faker.string.uuid();
    patient.firstName = faker.person.firstName();
    patient.lastName = faker.person.lastName();
    patient.email = faker.internet.email();
    patient.phoneNumber = faker.phone.number();
    patient.dateOfBirth = faker.date.past({ years: 80, refDate: new Date(2005, 0, 1) });
    patient.gender = faker.helpers.arrayElement(['male', 'female', 'other']);
    patient.address = {
      street: faker.location.streetAddress(),
      city: faker.location.city(),
      state: faker.location.state(),
      zipCode: faker.location.zipCode(),
      country: 'US',
    };
    patient.emergencyContact = {
      name: faker.person.fullName(),
      relationship: faker.helpers.arrayElement(['spouse', 'parent', 'sibling', 'child', 'friend']),
      phoneNumber: faker.phone.number(),
    };
    patient.status = 'active';
    patient.createdAt = faker.date.recent();
    patient.updatedAt = faker.date.recent();

    return Object.assign(patient, overrides);
  }

  static createMany(count: number, overrides: Partial<Patient> = {}): Patient[] {
    return Array.from({ length: count }, () => this.create(overrides));
  }
}

export class AppointmentFactory {
  static create(
    patientId: string,
    providerId: string,
    overrides: Partial<Appointment> = {}
  ): Appointment {
    const appointment = new Appointment();
    appointment.id = faker.string.uuid();
    appointment.patientId = patientId;
    appointment.providerId = providerId;
    appointment.dateTime = faker.date.future();
    appointment.duration = faker.helpers.arrayElement([15, 30, 45, 60]);
    appointment.type = faker.helpers.arrayElement(['routine', 'follow-up', 'consultation', 'emergency']);
    appointment.status = 'scheduled';
    appointment.notes = faker.lorem.paragraph();
    appointment.createdAt = faker.date.recent();
    appointment.updatedAt = faker.date.recent();

    return Object.assign(appointment, overrides);
  }
}

// E2E Test Base Class
import { test, expect, Page, BrowserContext } from '@playwright/test';

export class BasePage {
  constructor(protected page: Page) {}

  async navigateTo(path: string) {
    await this.page.goto(`${process.env.BASE_URL}${path}`);
  }

  async waitForLoadingToFinish() {
    await this.page.waitForSelector('[data-testid="loading-spinner"]', { state: 'detached' });
  }

  async login(email: string, password: string) {
    await this.page.fill('[data-testid="email-input"]', email);
    await this.page.fill('[data-testid="password-input"]', password);
    await this.page.click('[data-testid="login-button"]');
    await this.page.waitForURL('**/dashboard');
  }

  async logout() {
    await this.page.click('[data-testid="user-menu"]');
    await this.page.click('[data-testid="logout-button"]');
    await this.page.waitForURL('**/login');
  }
}

export class PatientManagementPage extends BasePage {
  async navigateToPatientRegistration() {
    await this.page.click('[data-testid="register-patient-button"]');
    await this.page.waitForURL('**/patients/new');
  }

  async fillPatientForm(patient: Partial<Patient>) {
    await this.page.fill('[data-testid="first-name-input"]', patient.firstName || '');
    await this.page.fill('[data-testid="last-name-input"]', patient.lastName || '');
    await this.page.fill('[data-testid="email-input"]', patient.email || '');
    await this.page.fill('[data-testid="phone-input"]', patient.phoneNumber || '');
    await this.page.fill('[data-testid="date-of-birth-input"]', 
      patient.dateOfBirth?.toISOString().split('T')[0] || '');
    
    if (patient.address) {
      await this.page.fill('[data-testid="street-input"]', patient.address.street);
      await this.page.fill('[data-testid="city-input"]', patient.address.city);
      await this.page.fill('[data-testid="state-input"]', patient.address.state);
      await this.page.fill('[data-testid="zip-code-input"]', patient.address.zipCode);
    }
  }

  async submitPatientForm() {
    await this.page.click('[data-testid="submit-patient-button"]');
    await this.waitForLoadingToFinish();
  }

  async searchPatient(searchTerm: string) {
    await this.page.fill('[data-testid="patient-search-input"]', searchTerm);
    await this.page.click('[data-testid="search-button"]');
    await this.waitForLoadingToFinish();
  }

  async verifyPatientInResults(patientName: string) {
    await expect(this.page.locator('[data-testid="patient-results"]')).toContainText(patientName);
  }
}

// E2E Test Suite Example
import { test, expect } from '@playwright/test';

test.describe('Patient Management Workflow', () => {
  let patientPage: PatientManagementPage;
  let testPatient: Patient;

  test.beforeEach(async ({ page }) => {
    patientPage = new PatientManagementPage(page);
    testPatient = PatientFactory.create();
    
    // Setup test environment
    await setupTestDatabase();
    
    // Login as healthcare provider
    await patientPage.navigateTo('/login');
    await patientPage.login('provider@test.com', 'password123');
  });

  test.afterEach(async () => {
    // Cleanup test data
    await cleanupTestDatabase();
  });

  test('should register a new patient successfully', async () => {
    // Navigate to patient registration
    await patientPage.navigateTo('/patients');
    await patientPage.navigateToPatientRegistration();

    // Fill and submit patient form
    await patientPage.fillPatientForm(testPatient);
    await patientPage.submitPatientForm();

    // Verify success message and redirect
    await expect(patientPage.page.locator('[data-testid="success-message"]'))
      .toContainText('Patient registered successfully');
    
    await expect(patientPage.page).toHaveURL(/.*\/patients\/\d+/);

    // Verify patient data is displayed correctly
    await expect(patientPage.page.locator('[data-testid="patient-name"]'))
      .toContainText(`${testPatient.firstName} ${testPatient.lastName}`);
  });

  test('should search and find registered patient', async () => {
    // Pre-register a patient
    await createTestPatient(testPatient);

    // Navigate to patient list and search
    await patientPage.navigateTo('/patients');
    await patientPage.searchPatient(testPatient.lastName);

    // Verify patient appears in search results
    await patientPage.verifyPatientInResults(`${testPatient.firstName} ${testPatient.lastName}`);
    
    // Verify search result contains correct information
    const patientRow = patientPage.page.locator(`[data-testid="patient-${testPatient.id}"]`);
    await expect(patientRow).toContainText(testPatient.email);
    await expect(patientRow).toContainText(testPatient.phoneNumber);
  });

  test('should handle patient registration validation errors', async () => {
    await patientPage.navigateTo('/patients/new');

    // Submit form without required fields
    await patientPage.submitPatientForm();

    // Verify validation errors are displayed
    await expect(patientPage.page.locator('[data-testid="first-name-error"]'))
      .toContainText('First name is required');
    await expect(patientPage.page.locator('[data-testid="last-name-error"]'))
      .toContainText('Last name is required');
    await expect(patientPage.page.locator('[data-testid="email-error"]'))
      .toContainText('Email is required');
  });
});

// API Integration Tests
import request from 'supertest';
import { Test, TestingModule } from '@nestjs/testing';
import { INestApplication } from '@nestjs/common';
import { AppModule } from '../src/app.module';

describe('Patient API Integration Tests', () => {
  let app: INestApplication;
  let authToken: string;

  beforeAll(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleFixture.createNestApplication();
    await app.init();

    // Get authentication token
    const loginResponse = await request(app.getHttpServer())
      .post('/auth/login')
      .send({
        email: 'test.provider@clinic.com',
        password: 'testpassword123',
      })
      .expect(200);

    authToken = loginResponse.body.access_token;
  });

  afterAll(async () => {
    await app.close();
  });

  beforeEach(async () => {
    await setupCleanTestDatabase();
  });

  afterEach(async () => {
    await cleanupTestDatabase();
  });

  describe('POST /patients', () => {
    it('should create a new patient successfully', async () => {
      const patientData = PatientFactory.create();

      const response = await request(app.getHttpServer())
        .post('/patients')
        .set('Authorization', `Bearer ${authToken}`)
        .send(patientData)
        .expect(201);

      expect(response.body).toMatchObject({
        id: expect.any(String),
        firstName: patientData.firstName,
        lastName: patientData.lastName,
        email: patientData.email,
        status: 'active',
      });

      expect(response.body.id).toBeDefined();
      expect(response.body.createdAt).toBeDefined();
    });

    it('should return validation error for invalid patient data', async () => {
      const invalidPatientData = {
        firstName: '', // Empty required field
        email: 'invalid-email', // Invalid email format
      };

      const response = await request(app.getHttpServer())
        .post('/patients')
        .set('Authorization', `Bearer ${authToken}`)
        .send(invalidPatientData)
        .expect(400);

      expect(response.body.message).toContain('firstName should not be empty');
      expect(response.body.message).toContain('email must be an email');
    });

    it('should return 409 for duplicate patient email', async () => {
      const patientData = PatientFactory.create();
      
      // Create first patient
      await request(app.getHttpServer())
        .post('/patients')
        .set('Authorization', `Bearer ${authToken}`)
        .send(patientData)
        .expect(201);

      // Try to create patient with same email
      await request(app.getHttpServer())
        .post('/patients')
        .set('Authorization', `Bearer ${authToken}`)
        .send({ ...patientData, firstName: 'Different', lastName: 'Name' })
        .expect(409);
    });
  });

  describe('GET /patients', () => {
    it('should return paginated list of patients', async () => {
      const patients = PatientFactory.createMany(15);
      await createTestPatients(patients);

      const response = await request(app.getHttpServer())
        .get('/patients?page=1&limit=10')
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);

      expect(response.body.data).toHaveLength(10);
      expect(response.body.meta).toMatchObject({
        page: 1,
        limit: 10,
        total: 15,
        totalPages: 2,
      });
    });

    it('should search patients by name', async () => {
      const testPatient = PatientFactory.create({ firstName: 'John', lastName: 'Doe' });
      await createTestPatient(testPatient);

      const response = await request(app.getHttpServer())
        .get('/patients?search=John Doe')
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);

      expect(response.body.data).toHaveLength(1);
      expect(response.body.data[0].firstName).toBe('John');
      expect(response.body.data[0].lastName).toBe('Doe');
    });
  });
});

// Performance Test Suite
import { test, expect } from '@playwright/test';

test.describe('Performance Tests', () => {
  test('patient search should respond promptly', async ({ page }) => {
    // Setup test data
    await setupLargePatientDataset(1000); // Create 1000 test patients

    await page.goto('/patients');
    await page.fill('[data-testid="patient-search-input"]', 'Smith');
    
    const startTime = Date.now();
    await page.click('[data-testid="search-button"]');
    await page.waitForSelector('[data-testid="search-results"]');
    const endTime = Date.now();

    const responseTime = endTime - startTime;
    const EXPECTED_RESPONSE_TIME = process.env.TEST_TIMEOUT || 5000;
    expect(responseTime).toBeLessThan(EXPECTED_RESPONSE_TIME); // Should respond promptly
  });

  test('patient registration form should load quickly', async ({ page }) => {
    const startTime = Date.now();
    await page.goto('/patients/new');
    await page.waitForSelector('[data-testid="patient-form"]');
    const endTime = Date.now();

    const loadTime = endTime - startTime;
    const EXPECTED_LOAD_TIME = process.env.LOAD_TIMEOUT || 3000;
    expect(loadTime).toBeLessThan(EXPECTED_LOAD_TIME); // Should load quickly
  });
});

// Test Utilities
export async function setupTestDatabase() {
  // Initialize test database with clean state
  await exec('npm run db:test:reset');
  await exec('npm run db:test:seed');
}

export async function cleanupTestDatabase() {
  // Clean up test data
  await exec('npm run db:test:cleanup');
}

export async function createTestPatient(patient: Patient): Promise<void> {
  // Helper to create patient in test database
  const patientRepository = getTestRepository(Patient);
  await patientRepository.save(patient);
}

export async function createTestPatients(patients: Patient[]): Promise<void> {
  const patientRepository = getTestRepository(Patient);
  await patientRepository.save(patients);
}
```

## Testing Requirements

### Test Coverage
```yaml
unit_tests:
  - focus: "Testing framework components and utilities"
  - coverage_target: "90%"
  - key_scenarios: ["test data factories", "page objects", "test utilities"]
  
integration_tests:
  - focus: "Complete testing framework functionality"
  - test_environment: "Isolated test environment with containerized services"
  - key_flows: ["test data setup/cleanup", "multi-environment testing", "CI/CD integration"]
  
e2e_tests:
  - focus: "Critical user workflows and business processes"
  - user_scenarios: ["patient registration", "appointment scheduling", "clinical documentation"]
```

### Test Scenarios
```yaml
happy_path:
  - scenario: "Complete patient management workflow test"
    steps: ["Register patient", "Schedule appointment", "Complete appointment", "Process billing"]
    expected_result: "All workflow steps complete successfully with proper data flow"
    
error_cases:
  - scenario: "Test framework handles application errors gracefully"
    trigger: "Application throws unexpected error during test execution"
    expected_behavior: "Test fails with clear error message and proper cleanup"
    
edge_cases:
  - scenario: "Concurrent test execution isolation"
    conditions: "Multiple test suites running simultaneously"
    expected_behavior: "Tests run independently without data conflicts or race conditions"
```

## Context References

### From Project
```yaml
tech_stack_references:
  - "NestJS backend testing with Jest and Supertest"
  - "React frontend testing with Playwright and Jest"
  - "PostgreSQL test database management and seeding"
  
architecture_patterns:
  - "Page Object Model for E2E test organization"
  - "Factory pattern for test data generation"
  - "Dependency injection for test utilities and services"
  
code_standards:
  - "Descriptive test naming and clear test structure"
  - "Test data isolation and cleanup between tests"
  - "Comprehensive error handling and assertion messages"
```

### From Epic
```yaml
business_rules:
  - "Patient data privacy and security maintained in test environments"
  - "Healthcare workflow testing covers compliance requirements"
  - "Critical patient care processes thoroughly tested"
  
domain_model:
  - "Patient registration and management workflows"
  - "Provider-patient relationship testing"
  - "Appointment scheduling and clinical documentation flows"
  
integration_points:
  - "AWS services integration testing with mock environments"
  - "External API integration testing with service mocking"
  - "Multi-service communication testing and validation"
```

## Acceptance Criteria

### Functional Acceptance
- [ ] E2E testing framework covering critical user workflows operational
- [ ] Test data management system with automated setup and cleanup
- [ ] Multi-environment testing capabilities with environment-specific configurations
- [ ] API integration tests covering all endpoints and error scenarios
- [ ] Performance testing suite validating system response times

### Technical Acceptance  
- [ ] Testing framework follows established patterns and best practices
- [ ] Test suites execute reliably with consistent results
- [ ] CI/CD integration with automated test execution and reporting
- [ ] Test coverage meets minimum requirements (80% unit, 100% API)
- [ ] Test execution time within acceptable limits

### Quality Acceptance
- [ ] Testing framework reviewed and approved by QA and development teams
- [ ] Test documentation comprehensive and accessible
- [ ] Test maintenance procedures documented and followed
- [ ] Test environment provisioning automated and reliable
- [ ] Ready for production deployment with comprehensive test coverage

---
**Last Updated:** August 23, 2025