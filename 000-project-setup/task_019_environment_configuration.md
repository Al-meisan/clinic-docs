# TASK-019: Environment Configuration Management

## Task Overview

### Goal
**What to Build:** Set up comprehensive environment-specific configuration management for development, staging, and production environments with secure secret management, feature flags, and environment-specific settings.

**Why:** Essential for maintaining different configurations across environments, securing sensitive data, enabling feature toggles, and ensuring smooth deployment workflows without configuration conflicts.

**Definition of Done:** Complete configuration management system operational across all environments with secure secret handling, automated configuration deployment, and environment-specific feature controls.

### Scope
```yaml
task_type: "FULLSTACK"
complexity: "HIGH"
```

## Requirements

### Functional Requirements
1. **Environment-Specific Configuration Files**
   - Description: Separate configuration files for dev, staging, and production environments
   - Acceptance Criteria: Each environment has isolated configuration with appropriate defaults
   - Priority: HIGH

2. **Secret Management System**
   - Description: Secure handling of sensitive configuration data and API keys
   - Acceptance Criteria: Secrets securely stored and in transit, with role-based access
   - Priority: HIGH

3. **Feature Flag System**
   - Description: Runtime feature toggles for gradual rollouts and A/B testing
   - Acceptance Criteria: Features can be enabled/disabled per environment without deployment
   - Priority: MEDIUM

4. **Configuration Validation**
   - Description: Automated validation of configuration completeness and correctness
   - Acceptance Criteria: Application startup fails with clear errors for invalid configuration
   - Priority: HIGH

5. **Dynamic Configuration Updates**
   - Description: Runtime configuration updates without application restart
   - Acceptance Criteria: Non-sensitive configuration can be updated dynamically
   - Priority: MEDIUM

### Technical Requirements
```yaml
performance:
  - requirement: "Configuration loading time"
    target: "Fast application startup with configuration initialization"
    
  - requirement: "Feature flag evaluation"
    target: "Minimal performance impact from feature toggles"
    
security:
  - requirement: "Secret secure storage at rest and in transit"
    implementation: "AWS KMS secure storage for sensitive configuration data"
    
  - requirement: "Role-based configuration access"
    implementation: "IAM-based access control for configuration management"
    
  - requirement: "Configuration audit trail"
    implementation: "All configuration changes logged and tracked"
    
compatibility:
  - requirement: "Multi-environment deployment support"
    scope: "Consistent configuration management across all environments"
    
  - requirement: "Local development environment configuration"
    scope: "Easy setup for developers with minimal configuration"
```

## Implementation Details

### Technical Approach
```yaml
# Configuration Architecture
configuration_structure:
  - layer: "Base Configuration"
    purpose: "Common settings shared across all environments"
    format: "TypeScript configuration objects with type safety"
    
  - layer: "Environment-Specific Overrides"
    purpose: "Environment-specific settings and feature flags"
    format: "JSON/YAML files per environment"
    
  - layer: "Secret Management"
    purpose: "Encrypted secrets and sensitive configuration"
    implementation: "AWS Secrets Manager with automatic rotation"
    
  - layer: "Runtime Configuration"
    purpose: "Dynamic settings that can change without restart"
    backend: "Redis-based configuration cache"

# Environment Management
environment_definitions:
  - environment: "development"
    purpose: "Local development with debug features enabled"
    characteristics: "Relaxed security, verbose logging, mock services"
    
  - environment: "staging"
    purpose: "Production-like environment for testing"
    characteristics: "Production security, limited data, full integration"
    
  - environment: "production"
    purpose: "Live production environment"
    characteristics: "Maximum security, performance optimization, monitoring"

# Configuration Categories
configuration_categories:
  - category: "Database Configuration"
    settings: "Connection strings, pool sizes, timeout values"
    security: "Passwords and connection secrets encrypted"
    
  - category: "API Configuration"
    settings: "External API endpoints, timeout values, retry policies"
    environment_specific: "Different API endpoints per environment"
    
  - category: "Feature Flags"
    settings: "Feature enablement per environment and user groups"
    dynamic: "Runtime toggles without deployment"
    
  - category: "Security Configuration"
    settings: "JWT secrets, secure storage keys, CORS settings"
    sensitive: "All security settings encrypted and access-controlled"
```

### Integration Points
```yaml
dependencies:
  - dependency: "AWS Infrastructure (TASK-002)"
    type: "INFRASTRUCTURE"
    status: "AVAILABLE"
    
  - dependency: "Backend Application (TASK-003)"
    type: "APPLICATION"
    status: "AVAILABLE"
    
  - dependency: "Frontend Application (TASK-007)"
    type: "APPLICATION"
    status: "AVAILABLE"
    
provides:
  - deliverable: "Configuration Management Service"
    interface: "Centralized configuration API and SDK"
    consumers: "All application services and components"
    
  - deliverable: "Environment Setup Scripts"
    interface: "Automated environment provisioning and configuration"
    consumers: "DevOps, CI/CD pipeline, developers"
    
  - deliverable: "Feature Flag System"
    interface: "Feature toggle API and management dashboard"
    consumers: "Development team, product managers, QA team"
```

### Code Patterns to Follow
```typescript
// Configuration Service (NestJS)
import { Injectable, Logger } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import { SecretsManagerClient, GetSecretValueCommand } from '@aws-sdk/client-secrets-manager';

interface DatabaseConfig {
  host: string;
  port: number;
  username: string;
  password: string;
  database: string;
  ssl: boolean;
  pool: {
    min: number;
    max: number;
    acquireTimeoutMillis: number;
  };
}

interface ApiConfig {
  baseUrl: string;
  timeout: number;
  retryAttempts: number;
  rateLimitPerSecond: number;
}

interface FeatureFlags {
  patientPortal: boolean;
  advancedSearch: boolean;
  multiLanguage: boolean;
  auditLogging: boolean;
}

@Injectable()
export class AppConfigService {
  private readonly logger = new Logger(AppConfigService.name);
  private readonly secretsClient: SecretsManagerClient;
  private configCache = new Map<string, any>();

  constructor(private configService: ConfigService) {
    this.secretsClient = new SecretsManagerClient({
      region: this.configService.get('AWS_REGION'),
    });
  }

  async getDatabaseConfig(): Promise<DatabaseConfig> {
    const baseConfig = {
      host: this.configService.get('DB_HOST'),
      port: this.configService.get<number>('DB_PORT', 5432),
      database: this.configService.get('DB_NAME'),
      ssl: this.configService.get<boolean>('DB_SSL', true),
      pool: {
        min: this.configService.get<number>('DB_POOL_MIN', 2),
        max: this.configService.get<number>('DB_POOL_MAX', 10),
        acquireTimeoutMillis: this.configService.get<number>('DB_TIMEOUT', 30000),
      },
    };

    // Get sensitive configuration from AWS Secrets Manager
    const credentials = await this.getSecret('database-credentials');
    
    return {
      ...baseConfig,
      username: credentials.username,
      password: credentials.password,
    };
  }

  getApiConfig(): ApiConfig {
    return {
      baseUrl: this.configService.get('API_BASE_URL'),
      timeout: this.configService.get<number>('API_TIMEOUT', 30000),
      retryAttempts: this.configService.get<number>('API_RETRY_ATTEMPTS', 3),
      rateLimitPerSecond: this.configService.get<number>('API_RATE_LIMIT', 100),
    };
  }

  async getFeatureFlags(): Promise<FeatureFlags> {
    const cacheKey = 'feature-flags';
    
    if (this.configCache.has(cacheKey)) {
      return this.configCache.get(cacheKey);
    }

    const flags = {
      patientPortal: this.configService.get<boolean>('FEATURE_PATIENT_PORTAL', false),
      advancedSearch: this.configService.get<boolean>('FEATURE_ADVANCED_SEARCH', true),
      multiLanguage: this.configService.get<boolean>('FEATURE_MULTI_LANGUAGE', true),
      auditLogging: this.configService.get<boolean>('FEATURE_AUDIT_LOGGING', true),
    };

    // Cache for 5 minutes
    this.configCache.set(cacheKey, flags);
    setTimeout(() => this.configCache.delete(cacheKey), 5 * 60 * 1000);

    return flags;
  }

  private async getSecret(secretName: string): Promise<any> {
    try {
      const command = new GetSecretValueCommand({
        SecretId: `${this.configService.get('APP_NAME')}/${this.configService.get('NODE_ENV')}/${secretName}`,
      });

      const response = await this.secretsClient.send(command);
      return JSON.parse(response.SecretString || '{}');
    } catch (error) {
      this.logger.error(`Failed to retrieve secret ${secretName}:`, error);
      throw new Error(`Configuration error: Unable to retrieve ${secretName}`);
    }
  }

  async refreshConfiguration(): Promise<void> {
    this.logger.log('Refreshing configuration cache');
    this.configCache.clear();
    
    // Preload critical configuration
    await this.getFeatureFlags();
    await this.getDatabaseConfig();
    
    this.logger.log('Configuration cache refreshed');
  }
}

// Configuration Validation Schema
import Joi from 'joi';

export const configValidationSchema = Joi.object({
  NODE_ENV: Joi.string()
    .valid('development', 'staging', 'production')
    .required(),
  
  PORT: Joi.number()
    .port()
    .default(3000),
    
  DB_HOST: Joi.string()
    .required(),
    
  DB_PORT: Joi.number()
    .port()
    .default(5432),
    
  DB_NAME: Joi.string()
    .required(),
    
  JWT_SECRET: Joi.string()
    .min(32)
    .required(),
    
  AWS_REGION: Joi.string()
    .required(),
    
  CORS_ORIGINS: Joi.string()
    .required()
    .custom((value, helpers) => {
      try {
        JSON.parse(value);
        return value;
      } catch {
        return helpers.error('any.invalid');
      }
    }),
    
  FEATURE_PATIENT_PORTAL: Joi.boolean()
    .default(false),
    
  FEATURE_ADVANCED_SEARCH: Joi.boolean()
    .default(true),
});

// Environment-Specific Configuration Files
// config/development.ts
export const developmentConfig = {
  database: {
    logging: true,
    synchronize: true,
    dropSchema: false,
  },
  logging: {
    level: 'debug',
    prettyPrint: true,
  },
  security: {
    corsOrigins: ['http://localhost:3000', 'http://localhost:5173'],
    rateLimiting: {
      enabled: false,
    },
  },
  features: {
    patientPortal: true,
    advancedSearch: true,
    multiLanguage: false,
    auditLogging: false,
  },
  external: {
    mockServices: true,
    apiTimeout: 10000,
  },
};

// config/production.ts
export const productionConfig = {
  database: {
    logging: false,
    synchronize: false,
    ssl: {
      rejectUnauthorized: true,
    },
  },
  logging: {
    level: 'info',
    prettyPrint: false,
  },
  security: {
    corsOrigins: ['https://clinic.example.com'],
    rateLimiting: {
      enabled: true,
      windowMs: 15 * 60 * 1000, // 15 minutes
      max: 1000, // limit each IP to 1000 requests per windowMs
    },
  },
  features: {
    patientPortal: true,
    advancedSearch: true,
    multiLanguage: true,
    auditLogging: true,
  },
  external: {
    mockServices: false,
    apiTimeout: 30000,
  },
};

// Feature Flag Hook (React)
import { useState, useEffect, useContext, createContext } from 'react';

interface FeatureFlagContextType {
  flags: FeatureFlags;
  isLoading: boolean;
  refreshFlags: () => Promise<void>;
}

const FeatureFlagContext = createContext<FeatureFlagContextType | null>(null);

export const FeatureFlagProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [flags, setFlags] = useState<FeatureFlags>({} as FeatureFlags);
  const [isLoading, setIsLoading] = useState(true);

  const fetchFlags = async () => {
    try {
      const response = await fetch('/api/config/feature-flags');
      const flagData = await response.json();
      setFlags(flagData);
    } catch (error) {
      console.error('Failed to fetch feature flags:', error);
      // Use default flags on error
      setFlags({
        patientPortal: false,
        advancedSearch: true,
        multiLanguage: false,
        auditLogging: true,
      });
    } finally {
      setIsLoading(false);
    }
  };

  const refreshFlags = async () => {
    await fetchFlags();
  };

  useEffect(() => {
    fetchFlags();
    
    // Refresh flags every 5 minutes
    const interval = setInterval(fetchFlags, 5 * 60 * 1000);
    return () => clearInterval(interval);
  }, []);

  return (
    <FeatureFlagContext.Provider value={{ flags, isLoading, refreshFlags }}>
      {children}
    </FeatureFlagContext.Provider>
  );
};

export const useFeatureFlags = () => {
  const context = useContext(FeatureFlagContext);
  if (!context) {
    throw new Error('useFeatureFlags must be used within FeatureFlagProvider');
  }
  return context;
};

// Usage in components
export const PatientPortalButton: React.FC = () => {
  const { flags } = useFeatureFlags();

  if (!flags.patientPortal) {
    return null; // Feature disabled
  }

  return (
    <Button onClick={() => navigate('/patient-portal')}>
      Access Patient Portal
    </Button>
  );
};

// Environment Configuration Loader
import { DynamicModule } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';

export class ConfigurationModule {
  static forRoot(): DynamicModule {
    const envConfig = this.loadEnvironmentConfig();
    
    return {
      module: ConfigurationModule,
      imports: [
        ConfigModule.forRoot({
          isGlobal: true,
          load: [() => envConfig],
          validationSchema: configValidationSchema,
          validationOptions: {
            abortEarly: true,
            allowUnknown: false,
          },
        }),
      ],
      providers: [AppConfigService],
      exports: [AppConfigService],
    };
  }

  private static loadEnvironmentConfig() {
    const env = process.env.NODE_ENV || 'development';
    
    switch (env) {
      case 'development':
        return { ...developmentConfig };
      case 'staging':
        return { ...stagingConfig };
      case 'production':
        return { ...productionConfig };
      default:
        throw new Error(`Unknown environment: ${env}`);
    }
  }
}

// Configuration Management CLI Tool
#!/usr/bin/env node

import { Command } from 'commander';
import { SecretsManagerClient, PutSecretValueCommand } from '@aws-sdk/client-secrets-manager';

const program = new Command();

program
  .name('config-manager')
  .description('CLI tool for managing application configuration')
  .version('1.0.0');

program
  .command('set-secret')
  .description('Set a secret in AWS Secrets Manager')
  .requiredOption('-n, --name <name>', 'Secret name')
  .requiredOption('-v, --value <value>', 'Secret value')
  .requiredOption('-e, --environment <env>', 'Environment (dev/staging/prod)')
  .action(async (options) => {
    const client = new SecretsManagerClient({ region: process.env.AWS_REGION });
    
    try {
      await client.send(new PutSecretValueCommand({
        SecretId: `clinic-app/${options.environment}/${options.name}`,
        SecretString: options.value,
      }));
      
      console.log(`Secret ${options.name} set successfully for ${options.environment}`);
    } catch (error) {
      console.error('Failed to set secret:', error);
      process.exit(1);
    }
  });

program
  .command('validate-config')
  .description('Validate configuration for environment')
  .requiredOption('-e, --environment <env>', 'Environment to validate')
  .action(async (options) => {
    try {
      // Load and validate configuration
      process.env.NODE_ENV = options.environment;
      const config = loadEnvironmentConfig();
      
      const { error } = configValidationSchema.validate(process.env);
      if (error) {
        throw new Error(`Configuration validation failed: ${error.message}`);
      }
      
      console.log(`Configuration for ${options.environment} is valid`);
    } catch (error) {
      console.error('Configuration validation failed:', error.message);
      process.exit(1);
    }
  });

program.parse();
```

## Testing Requirements

### Test Coverage
```yaml
unit_tests:
  - focus: "Configuration loading, validation, and secret management"
  - coverage_target: "90%"
  - key_scenarios: ["config loading", "validation", "feature flag evaluation"]
  
integration_tests:
  - focus: "Environment-specific configuration integration"
  - test_environment: "All three environments (dev/staging/prod)"
  - key_flows: ["secret retrieval", "feature flag updates", "configuration refresh"]
  
e2e_tests:
  - focus: "Complete configuration management workflows"
  - user_scenarios: ["environment deployment", "feature flag rollout", "secret rotation"]
```

### Test Scenarios
```yaml
happy_path:
  - scenario: "Application starts with valid configuration"
    steps: ["Load config", "Validate settings", "Start application", "Verify functionality"]
    expected_result: "Application starts successfully with all features configured"
    
error_cases:
  - scenario: "Invalid configuration provided"
    trigger: "Missing required environment variable or invalid secret"
    expected_behavior: "Application fails to start with clear error message"
    
  - scenario: "AWS Secrets Manager unavailable"
    trigger: "Network connectivity issue or service outage"
    expected_behavior: "Graceful fallback with cached configuration or default values"
    
edge_cases:
  - scenario: "Configuration hot reload during runtime"
    conditions: "Feature flags updated while application is running"
    expected_behavior: "New configuration applied without service restart"
```

## Context References

### From Project
```yaml
tech_stack_references:
  - "NestJS ConfigModule with validation and type safety"
  - "AWS Secrets Manager for secure secret management"
  - "React Context for frontend configuration management"
  
architecture_patterns:
  - "Twelve-factor app configuration methodology"
  - "Environment-specific deployment patterns"
  - "Secure secret management and rotation"
  
code_standards:
  - "Type-safe configuration with validation schemas"
  - "Environment-specific configuration files"
  - "Centralized configuration service pattern"
```

### From Epic
```yaml
business_rules:
  - "Healthcare compliance requires secure configuration management"
  - "Different environments need appropriate security levels"
  - "Feature rollouts must be controlled and reversible"
  
domain_model:
  - "Clinic settings configurable per environment"
  - "Provider features enabled based on deployment environment"
  - "Patient data handling varies by environment requirements"
  
integration_points:
  - "AWS services configuration per environment"
  - "Database connection settings environment-specific"
  - "Third-party integrations with different endpoints per environment"
```

## Acceptance Criteria

### Functional Acceptance
- [ ] Environment-specific configuration files operational for dev/staging/production
- [ ] Secure secret management with AWS Secrets Manager integration
- [ ] Feature flag system with runtime toggles and management interface
- [ ] Configuration validation preventing startup with invalid settings
- [ ] Dynamic configuration updates for non-sensitive settings

### Technical Acceptance  
- [ ] Code follows configuration management best practices
- [ ] Unit tests written and passing (>= 90% coverage)
- [ ] Integration tests validating environment-specific configurations
- [ ] Configuration deployment automation via CI/CD pipeline
- [ ] Security audit of secret management and access controls

### Quality Acceptance
- [ ] Code review completed and approved by technical leads
- [ ] Configuration documentation updated and accessible
- [ ] Environment setup procedures tested and documented
- [ ] Configuration management CLI tools functional and documented
- [ ] Ready for multi-environment deployment with secure configuration

---
**Last Updated:** August 23, 2025