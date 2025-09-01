# TASK-010: Frontend Authentication Implementation

## Task Overview

### Goal
**What to Build:** Create comprehensive frontend authentication system with login/logout components, JWT authentication integration, automatic token management, and user session handling for the MedFlow healthcare management application.

**Why:** Provides secure, user-friendly authentication experience that integrates seamlessly with backend authentication, handles healthcare-specific user roles, and maintains proper session state throughout the application.

**Definition of Done:** Complete authentication system operational with login/logout forms, automatic token refresh, user context management, and proper error handling for all authentication scenarios.

### Scope
```yaml
task_type: "FRONTEND"
complexity: "HIGH"
```

## Requirements

### Functional Requirements
1. **Login Interface Components**
   - Description: Professional login form with email/password authentication, loading states, and error handling
   - Acceptance Criteria: Login form validates inputs, handles errors gracefully, and provides clear feedback to users
   - Priority: HIGH

2. **Backend Authentication Integration**
   - Description: Complete integration with backend authentication service including user registration and password reset
   - Acceptance Criteria: Authentication flows work seamlessly with backend API, tokens properly managed, and user attributes retrieved
   - Priority: HIGH

3. **Automatic Token Management**
   - Description: JWT token storage, automatic refresh, and session state management
   - Acceptance Criteria: Tokens automatically refresh before expiration, secure storage implemented, and logout clears all session data
   - Priority: HIGH

4. **User Context Management**
   - Description: Global user state management with role-based access control and clinic association
   - Acceptance Criteria: User information available throughout app, role-based UI elements working, and clinic context maintained
   - Priority: MEDIUM

### Technical Requirements
```yaml
performance:
  - requirement: "Authentication response time"
    target: "< 2 seconds for login/logout operations"
    
security:
  - requirement: "Token security and storage"
    implementation: "Secure token storage with automatic cleanup on logout"
    
compatibility:
  - requirement: "Multi-language authentication interface"
    scope: "Login forms support Arabic and French languages"
```

## Implementation Details

### Technical Approach
```yaml
authentication_architecture:
  backend_integration:
    - api_client: "Axios or fetch for API communication"
    - jwt_decode: "Token parsing and validation"
    - secure_storage: "Token storage with secure storage"
    - auth_interceptors: "Request/response interceptors for token handling"
    
  state_management:
    - auth_context: "React Context for authentication state"
    - auth_hooks: "Custom hooks for authentication operations"
    - persistent_state: "Session persistence across browser sessions"
    - token_refresh: "Automatic token refresh mechanism"
    
  ui_components:
    - login_form: "Professional healthcare login interface"
    - loading_states: "Authentication operation feedback"
    - error_handling: "User-friendly error messages"
    - success_feedback: "Login success confirmation"

authentication_flow:
  login_process:
    - input_validation: "Email format and password requirements"
    - backend_authentication: "Backend login API call"
    - token_storage: "Secure JWT token storage"
    - user_profile_fetch: "Retrieve user profile and permissions"
    - redirect_handling: "Navigate to intended destination"
    
  token_management:
    - automatic_refresh: "Refresh tokens before expiration"
    - secure_storage: "Encrypted token storage mechanism"
    - cleanup_on_logout: "Complete session cleanup"
    - token_validation: "Verify token integrity and expiration"
```

### Integration Points
```yaml
dependencies:
  - dependency: "Backend authentication endpoints"
    type: "API"
    status: "IN_PROGRESS"
    
  - dependency: "React application with routing system"
    type: "FRONTEND"
    status: "AVAILABLE"
    
  - dependency: "UI framework components"
    type: "FRONTEND"
    status: "AVAILABLE"
    
provides:
  - deliverable: "Authentication context and hooks"
    interface: "React hooks and context for authentication state"
    consumers: "All frontend components requiring authentication"
    
  - deliverable: "Login/logout UI components"
    interface: "Reusable authentication forms and controls"
    consumers: "Authentication routes and navigation components"
    
  - deliverable: "Token management system"
    interface: "Automatic token handling and refresh"
    consumers: "API client and all authenticated requests"
```

### Code Patterns to Follow

**Authentication Context:**
```typescript
// src/providers/AuthProvider.tsx
import React, { createContext, useContext, useState, useEffect, useCallback } from 'react';
import { CognitoAuth } from '@/lib/auth/cognito';
import { User, AuthState, LoginCredentials } from '@/types/auth.types';

interface AuthContextType {
  user: User | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  login: (credentials: LoginCredentials) => Promise<void>;
  logout: () => Promise<void>;
  refreshToken: () => Promise<void>;
  checkAuthState: () => Promise<void>;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export const AuthProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [authState, setAuthState] = useState<AuthState>({
    user: null,
    isAuthenticated: false,
    isLoading: true,
  });

  const cognitoAuth = new CognitoAuth();

  const setUser = (user: User | null) => {
    setAuthState(prev => ({
      ...prev,
      user,
      isAuthenticated: !!user,
      isLoading: false,
    }));
  };

  const login = useCallback(async (credentials: LoginCredentials) => {
    try {
      setAuthState(prev => ({ ...prev, isLoading: true }));
      
      const authResult = await cognitoAuth.signIn(credentials);
      const user = await cognitoAuth.getCurrentUser();
      
      if (user) {
        setUser(user);
        // Store tokens securely
        await cognitoAuth.storeTokens(authResult);
      }
    } catch (error) {
      setAuthState(prev => ({ ...prev, isLoading: false }));
      throw error;
    }
  }, []);

  const logout = useCallback(async () => {
    try {
      setAuthState(prev => ({ ...prev, isLoading: true }));
      await cognitoAuth.signOut();
      setUser(null);
    } catch (error) {
      // Even if logout fails, clear local state
      setUser(null);
      throw error;
    }
  }, []);

  const refreshToken = useCallback(async () => {
    try {
      const newTokens = await cognitoAuth.refreshSession();
      if (newTokens) {
        await cognitoAuth.storeTokens(newTokens);
      }
    } catch (error) {
      // If refresh fails, user needs to re-authenticate
      setUser(null);
      throw error;
    }
  }, []);

  const checkAuthState = useCallback(async () => {
    try {
      setAuthState(prev => ({ ...prev, isLoading: true }));
      const user = await cognitoAuth.getCurrentUser();
      setUser(user);
    } catch (error) {
      setUser(null);
    }
  }, []);

  // Check authentication state on mount
  useEffect(() => {
    checkAuthState();
  }, [checkAuthState]);

  // Set up automatic token refresh
  useEffect(() => {
    if (authState.isAuthenticated) {
      const refreshInterval = setInterval(async () => {
        try {
          await refreshToken();
        } catch (error) {
          console.error('Token refresh failed:', error);
        }
      }, 15 * 60 * 1000); // Refresh every 15 minutes

      return () => clearInterval(refreshInterval);
    }
  }, [authState.isAuthenticated, refreshToken]);

  const value: AuthContextType = {
    user: authState.user,
    isAuthenticated: authState.isAuthenticated,
    isLoading: authState.isLoading,
    login,
    logout,
    refreshToken,
    checkAuthState,
  };

  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = (): AuthContextType => {
  const context = useContext(AuthContext);
  if (context === undefined) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
};
```

**Cognito Authentication Service:**
```typescript
// src/lib/auth/cognito.ts
import {
  CognitoUserPool,
  CognitoUser,
  AuthenticationDetails,
  CognitoUserSession,
} from 'amazon-cognito-identity-js';
import { jwtDecode } from 'jwt-decode';
import { User, LoginCredentials, AuthResult, CognitoUserPayload } from '@/types/auth.types';

export class CognitoAuth {
  private userPool: CognitoUserPool;

  constructor() {
    this.userPool = new CognitoUserPool({
      UserPoolId: import.meta.env.VITE_COGNITO_USER_POOL_ID,
      ClientId: import.meta.env.VITE_COGNITO_CLIENT_ID,
    });
  }

  async signIn(credentials: LoginCredentials): Promise<AuthResult> {
    return new Promise((resolve, reject) => {
      const authenticationDetails = new AuthenticationDetails({
        Username: credentials.email,
        Password: credentials.password,
      });

      const cognitoUser = new CognitoUser({
        Username: credentials.email,
        Pool: this.userPool,
      });

      cognitoUser.authenticateUser(authenticationDetails, {
        onSuccess: (session: CognitoUserSession) => {
          const authResult: AuthResult = {
            accessToken: session.getAccessToken().getJwtToken(),
            refreshToken: session.getRefreshToken().getToken(),
            idToken: session.getIdToken().getJwtToken(),
            expiresIn: session.getAccessToken().getExpiration(),
          };
          resolve(authResult);
        },
        onFailure: (error) => {
          reject(this.handleAuthError(error));
        },
        mfaRequired: (codeDeliveryDetails) => {
          // Handle MFA if enabled
          reject(new Error('MFA not implemented yet'));
        },
      });
    });
  }

  async signOut(): Promise<void> {
    const cognitoUser = this.userPool.getCurrentUser();
    if (cognitoUser) {
      return new Promise((resolve) => {
        cognitoUser.signOut(() => {
          this.clearStoredTokens();
          resolve();
        });
      });
    }
    this.clearStoredTokens();
  }

  async getCurrentUser(): Promise<User | null> {
    const cognitoUser = this.userPool.getCurrentUser();
    if (!cognitoUser) {
      return null;
    }

    return new Promise((resolve, reject) => {
      cognitoUser.getSession((error: any, session: CognitoUserSession) => {
        if (error) {
          reject(error);
          return;
        }

        if (!session.isValid()) {
          resolve(null);
          return;
        }

        // Get user attributes
        cognitoUser.getUserAttributes((attributeError, attributes) => {
          if (attributeError) {
            reject(attributeError);
            return;
          }

          try {
            const idToken = session.getIdToken().getJwtToken();
            const payload = jwtDecode<CognitoUserPayload>(idToken);
            
            const user: User = {
              id: payload.sub,
              cognitoId: payload.sub,
              email: payload.email,
              firstName: payload.given_name,
              lastName: payload.family_name,
              role: payload['custom:role'] || 'healthcare_provider',
              scopes: this.parseScopes(payload['custom:scopes'] || ''),
              clinicId: payload['custom:clinic_id'],
              isActive: true,
              preferences: {
                language: payload['custom:language'] || 'fr',
                theme: 'light',
                timezone: 'Africa/Algiers',
                dateFormat: 'DD/MM/YYYY',
                timeFormat: '24h',
              },
            };

            resolve(user);
          } catch (parseError) {
            reject(parseError);
          }
        });
      });
    });
  }

  async refreshSession(): Promise<AuthResult | null> {
    const cognitoUser = this.userPool.getCurrentUser();
    if (!cognitoUser) {
      return null;
    }

    return new Promise((resolve, reject) => {
      cognitoUser.getSession((error: any, session: CognitoUserSession) => {
        if (error) {
          reject(error);
          return;
        }

        if (session.isValid()) {
          const authResult: AuthResult = {
            accessToken: session.getAccessToken().getJwtToken(),
            refreshToken: session.getRefreshToken().getToken(),
            idToken: session.getIdToken().getJwtToken(),
            expiresIn: session.getAccessToken().getExpiration(),
          };
          resolve(authResult);
        } else {
          resolve(null);
        }
      });
    });
  }

  async storeTokens(authResult: AuthResult): Promise<void> {
    // Store tokens securely (consider secure storage for sensitive environments)
    localStorage.setItem('medflow_auth', JSON.stringify({
      accessToken: authResult.accessToken,
      refreshToken: authResult.refreshToken,
      idToken: authResult.idToken,
      expiresIn: authResult.expiresIn,
      timestamp: Date.now(),
    }));
  }

  private clearStoredTokens(): void {
    localStorage.removeItem('medflow_auth');
  }

  private parseScopes(scopesString: string): string[] {
    if (!scopesString) return [];
    return scopesString.split(',').map(scope => scope.trim()).filter(Boolean);
  }

  private handleAuthError(error: any): Error {
    const errorMessages: Record<string, string> = {
      'UserNotConfirmedException': 'Please check your email and confirm your account.',
      'PasswordResetRequiredException': 'Password reset is required.',
      'UserNotFoundException': 'Invalid email or password.',
      'NotAuthorizedException': 'Invalid email or password.',
      'TooManyRequestsException': 'Too many login attempts. Please try again later.',
    };

    const message = errorMessages[error.code] || error.message || 'Login failed. Please try again.';
    return new Error(message);
  }
}
```

**Login Component:**
```typescript
// src/features/auth/components/LoginForm.tsx
import React, { useState } from 'react';
import { useNavigate, useLocation } from 'react-router-dom';
import { useTranslation } from 'react-i18next';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card';
import { Alert, AlertDescription } from '@/components/ui/alert';
import { LoadingSpinner } from '@/components/common/LoadingSpinner';
import { useAuth } from '@/hooks/useAuth';
import { LoginCredentials } from '@/types/auth.types';

export const LoginForm: React.FC = () => {
  const [credentials, setCredentials] = useState<LoginCredentials>({
    email: '',
    password: '',
  });
  const [error, setError] = useState<string>('');
  const [isLoading, setIsLoading] = useState(false);

  const { login } = useAuth();
  const navigate = useNavigate();
  const location = useLocation();
  const { t } = useTranslation('auth');

  const from = location.state?.from?.pathname || '/dashboard';

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    if (!credentials.email || !credentials.password) {
      setError(t('login.error.required_fields'));
      return;
    }

    try {
      setIsLoading(true);
      setError('');
      
      await login(credentials);
      
      // Navigate to intended destination
      navigate(from, { replace: true });
    } catch (loginError: any) {
      setError(loginError.message || t('login.error.generic'));
    } finally {
      setIsLoading(false);
    }
  };

  const handleInputChange = (field: keyof LoginCredentials, value: string) => {
    setCredentials(prev => ({ ...prev, [field]: value }));
    // Clear error when user starts typing
    if (error) setError('');
  };

  return (
    <div className="flex min-h-screen items-center justify-center bg-gray-50 py-12 px-4 sm:px-6 lg:px-8">
      <div className="w-full max-w-md space-y-8">
        <div className="text-center">
          <img
            className="mx-auto h-12 w-auto"
            src="/logo.svg"
            alt="MedFlow"
          />
          <h2 className="mt-6 text-3xl font-bold tracking-tight text-gray-900">
            {t('login.title')}
          </h2>
          <p className="mt-2 text-sm text-gray-600">
            {t('login.subtitle')}
          </p>
        </div>

        <Card>
          <CardHeader>
            <CardTitle>{t('login.form.title')}</CardTitle>
            <CardDescription>
              {t('login.form.description')}
            </CardDescription>
          </CardHeader>
          <CardContent>
            <form onSubmit={handleSubmit} className="space-y-6">
              {error && (
                <Alert variant="destructive">
                  <AlertDescription>{error}</AlertDescription>
                </Alert>
              )}

              <div className="space-y-4">
                <div>
                  <label
                    htmlFor="email"
                    className="block text-sm font-medium text-gray-700"
                  >
                    {t('login.form.email.label')}
                  </label>
                  <Input
                    id="email"
                    name="email"
                    type="email"
                    autoComplete="email"
                    required
                    value={credentials.email}
                    onChange={(e) => handleInputChange('email', e.target.value)}
                    placeholder={t('login.form.email.placeholder')}
                    className="mt-1"
                    disabled={isLoading}
                  />
                </div>

                <div>
                  <label
                    htmlFor="password"
                    className="block text-sm font-medium text-gray-700"
                  >
                    {t('login.form.password.label')}
                  </label>
                  <Input
                    id="password"
                    name="password"
                    type="password"
                    autoComplete="current-password"
                    required
                    value={credentials.password}
                    onChange={(e) => handleInputChange('password', e.target.value)}
                    placeholder={t('login.form.password.placeholder')}
                    className="mt-1"
                    disabled={isLoading}
                  />
                </div>
              </div>

              <Button
                type="submit"
                className="w-full"
                size="lg"
                disabled={isLoading}
              >
                {isLoading ? (
                  <>
                    <LoadingSpinner className="mr-2 h-4 w-4" />
                    {t('login.form.button.loading')}
                  </>
                ) : (
                  t('login.form.button.login')
                )}
              </Button>
            </form>

            <div className="mt-6">
              <div className="text-center">
                <button
                  type="button"
                  className="text-sm text-primary hover:text-primary/80"
                  onClick={() => navigate('/forgot-password')}
                >
                  {t('login.links.forgot_password')}
                </button>
              </div>
            </div>
          </CardContent>
        </Card>
      </div>
    </div>
  );
};
```

**User Profile Component:**
```typescript
// src/components/common/UserProfile.tsx
import React from 'react';
import { useTranslation } from 'react-i18next';
import { LogOut, Settings, User } from 'lucide-react';
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuLabel,
  DropdownMenuSeparator,
  DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu';
import { Avatar, AvatarFallback } from '@/components/ui/avatar';
import { Button } from '@/components/ui/button';
import { useAuth } from '@/hooks/useAuth';
import { useNavigate } from 'react-router-dom';

export const UserProfile: React.FC = () => {
  const { user, logout } = useAuth();
  const { t } = useTranslation('common');
  const navigate = useNavigate();

  if (!user) {
    return null;
  }

  const handleLogout = async () => {
    try {
      await logout();
      navigate('/login');
    } catch (error) {
      console.error('Logout failed:', error);
    }
  };

  const initials = `${user.firstName.charAt(0)}${user.lastName.charAt(0)}`.toUpperCase();

  return (
    <DropdownMenu>
      <DropdownMenuTrigger asChild>
        <Button variant="ghost" className="relative h-8 w-8 rounded-full">
          <Avatar className="h-8 w-8">
            <AvatarFallback className="bg-primary text-primary-foreground">
              {initials}
            </AvatarFallback>
          </Avatar>
        </Button>
      </DropdownMenuTrigger>
      <DropdownMenuContent className="w-56" align="end" forceMount>
        <DropdownMenuLabel className="font-normal">
          <div className="flex flex-col space-y-1">
            <p className="text-sm font-medium leading-none">
              {user.firstName} {user.lastName}
            </p>
            <p className="text-xs leading-none text-muted-foreground">
              {user.email}
            </p>
            <p className="text-xs leading-none text-muted-foreground">
              {t(`roles.${user.role}`)}
            </p>
          </div>
        </DropdownMenuLabel>
        <DropdownMenuSeparator />
        <DropdownMenuItem onClick={() => navigate('/profile')}>
          <User className="mr-2 h-4 w-4" />
          <span>{t('user_menu.profile')}</span>
        </DropdownMenuItem>
        <DropdownMenuItem onClick={() => navigate('/settings')}>
          <Settings className="mr-2 h-4 w-4" />
          <span>{t('user_menu.settings')}</span>
        </DropdownMenuItem>
        <DropdownMenuSeparator />
        <DropdownMenuItem onClick={handleLogout}>
          <LogOut className="mr-2 h-4 w-4" />
          <span>{t('user_menu.logout')}</span>
        </DropdownMenuItem>
      </DropdownMenuContent>
    </DropdownMenu>
  );
};
```

**Authentication Types:**
```typescript
// src/types/auth.types.ts
export interface User {
  id: string;
  cognitoId: string;
  email: string;
  firstName: string;
  lastName: string;
  role: UserRole;
  scopes: string[];
  clinicId: string;
  isActive: boolean;
  preferences: UserPreferences;
}

export interface UserPreferences {
  language: 'ar' | 'fr';
  theme: 'light' | 'dark';
  timezone: string;
  dateFormat: string;
  timeFormat: '12h' | '24h';
}

export interface LoginCredentials {
  email: string;
  password: string;
}

export interface AuthResult {
  accessToken: string;
  refreshToken: string;
  idToken: string;
  expiresIn: number;
}

export interface AuthState {
  user: User | null;
  isAuthenticated: boolean;
  isLoading: boolean;
}

export interface CognitoUserPayload {
  sub: string;
  email: string;
  given_name: string;
  family_name: string;
  'custom:role': string;
  'custom:scopes': string;
  'custom:clinic_id': string;
  'custom:language': string;
  exp: number;
  iat: number;
}

export type UserRole = 
  | 'system_administrator'
  | 'clinic_manager'
  | 'healthcare_provider'
  | 'clinical_staff'
  | 'administrative_staff';
```

## Testing Requirements

### Test Coverage
```yaml
unit_tests:
  - focus: "Authentication context, hooks, and utilities"
  - coverage_target: "95%"
  - key_scenarios: ["login_flow", "token_management", "user_context"]
  
integration_tests:
  - focus: "Cognito integration and authentication flows"
  - test_environment: "Mock Cognito service or test environment"
  - key_flows: ["complete_login_process", "token_refresh", "logout_cleanup"]
  
e2e_tests:
  - focus: "Complete authentication user journeys"
  - user_scenarios: ["login_and_access_protected_route", "session_expiry", "logout_flow"]
```

### Test Scenarios
```yaml
happy_path:
  - scenario: "Successful login with valid credentials"
    steps: ["Enter email and password", "Submit form", "Receive tokens", "Navigate to dashboard"]
    expected_result: "User authenticated and redirected to intended destination"
    
error_cases:
  - scenario: "Login with invalid credentials"
    trigger: "Incorrect email or password"
    expected_behavior: "Clear error message displayed without exposing system details"
    
  - scenario: "Network error during authentication"
    trigger: "Network connectivity issues"
    expected_behavior: "Appropriate error message with retry option"
    
edge_cases:
  - scenario: "Token expiry during active session"
    conditions: "JWT token expires while user is using application"
    expected_behavior: "Automatic token refresh or graceful re-authentication"
```

## Context References

### From Project
```yaml
tech_stack_references:
  - "AWS Cognito for user authentication and management"
  - "React Context API for global authentication state"
  - "JWT tokens with automatic refresh mechanism"
  - "Secure token storage with secure storage considerations"
  
architecture_patterns:
  - "Provider pattern for authentication context"
  - "Hook pattern for authentication operations"
  - "Service layer for Cognito integration"
  - "Error boundary pattern for authentication failures"
  
code_standards:
  - "Consistent error handling with user-friendly messages"
  - "Security-first approach to token management"
  - "Accessibility compliance in authentication forms"
  - "Multi-language support for authentication interface"
```

### From Epic
```yaml
business_rules:
  - "Healthcare-specific user attributes in Cognito (medical license, specialties)"
  - "Role-based authentication with OAuth2 scopes"
  - "Clinic association for multi-tenant data isolation"
  - "Automatic logout for security in clinical environments"
  
integration_points:
  - "Required by routing system for protected routes"
  - "Foundation for all authenticated API calls"
  - "Enables role-based UI component visibility"
```

## Acceptance Criteria

### Functional Acceptance
- [ ] Login form validates inputs and handles authentication
- [ ] AWS Cognito integration working with proper token management
- [ ] User context available throughout application
- [ ] Automatic token refresh working before expiration
- [ ] Logout clears all session data and redirects properly

### Technical Acceptance  
- [ ] Authentication operations meet performance requirements
- [ ] Tokens stored securely with proper cleanup
- [ ] Error handling provides clear user feedback
- [ ] Multi-language support working in authentication interface
- [ ] Session state persists across browser refreshes

### Quality Acceptance
- [ ] Authentication flows tested with all user roles
- [ ] Security testing confirms proper token handling
- [ ] Accessibility testing passes for authentication forms
- [ ] Error scenarios handled gracefully without system exposure
- [ ] Integration testing confirms Cognito service connectivity

---

**Last Updated:** 2025-01-24