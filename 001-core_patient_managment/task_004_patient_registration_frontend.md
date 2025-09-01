# TASK: TASK-004 - Patient Registration Frontend Interface

## Task Overview

### Goal

**What to Build:** Complete React-based patient registration interface with form validation, multi-language support,
photo capture capabilities, and seamless integration with the backend patient registration API.

**Why:** Provide healthcare staff with an intuitive, efficient interface for registering new patients, replacing manual
paper-based processes and ensuring data accuracy.

**Definition of Done:**

- Patient registration form captures all required information with validation
- Real-time validation provides immediate feedback to users
- Arabic and French language support with proper RTL layout
- Photo capture functionality for patient identification
- Emergency contact management with multiple contact support
- Integration with backend API with proper error handling

### Scope

```yaml
task_type: "FRONTEND"
complexity: "HIGH"
```

## Requirements

### Functional Requirements

1. **Patient Registration Form**
    - Description: Multi-step form for capturing patient demographic and contact information
    - Acceptance Criteria:
        - Captures all required fields (firstName, lastName, dateOfBirth, phone)
        - Real-time validation with error messages
        - Progressive disclosure for optional information
        - Form state preservation during navigation
    - Priority: HIGH

2. **Emergency Contact Management**
    - Description: Dynamic interface for adding and managing emergency contacts
    - Acceptance Criteria:
        - Add/remove multiple emergency contacts
        - Designate primary emergency contact
        - Validate contact information in real-time
        - Support for relationship types and contact preferences
    - Priority: HIGH

3. **Address Input Component**
    - Description: Structured address input with Algerian address format support
    - Acceptance Criteria:
        - Street, city, state, zip code fields
        - Autocomplete suggestions for cities/states
        - Validation for Algerian postal codes
        - Optional geocoding integration
    - Priority: MEDIUM

4. **Photo Capture Feature**
    - Description: Patient photo capture using device camera or file upload
    - Acceptance Criteria:
        - Camera access for photo capture
        - File upload with image validation
        - Image preview and crop functionality
        - Compression and format optimization
    - Priority: MEDIUM

5. **Multi-language Support**
    - Description: Complete Arabic and French language support with RTL layout
    - Acceptance Criteria:
        - Dynamic language switching
        - RTL layout support for Arabic
        - Proper text direction and element alignment
        - Localized validation messages
    - Priority: HIGH

### Technical Requirements

```yaml

security:
  - requirement: "Input sanitization"
    implementation: "Client-side validation with server-side verification"

  - requirement: "Data privacy"
    implementation: "No sensitive data stored in localStorage or browser cache"

compatibility:
  - requirement: "Cross-browser support"
    scope: "Modern browsers with ES6+ support"
```

## Implementation Details

### Technical Approach

```yaml
# Frontend Implementation Strategy
ui_components:
  - component: "PatientRegistrationForm"
    purpose: "Main registration form container with multi-step navigation"
    props: "onSubmit, initialData, onCancel"

  - component: "PatientInfoStep"
    purpose: "Basic patient information capture"
    props: "formData, onUpdate, errors, validationRules"

  - component: "ContactInfoStep"
    purpose: "Contact and address information"
    props: "contactData, onUpdate, errors"

  - component: "EmergencyContactsStep"
    purpose: "Emergency contact management"
    props: "contacts, onAddContact, onRemoveContact, onUpdateContact"

state_management:
  - approach: "Local form state with React Hook Form"
  - data_flow: "Form data validation -> API submission -> Success/Error handling"

integration_points:
  - api: "Patient Registration API (TASK-001)"
    endpoints: "POST /api/patients"
    error_handling: "Display validation errors, handle network failures"
```

### Integration Points

```yaml
dependencies:
  - dependency: "Patient Registration Backend API (TASK-001)"
    type: "API"
    status: "IN_PROGRESS"

  - dependency: "Multi-language framework (react-i18next)"
    type: "COMPONENT"
    status: "AVAILABLE"

  - dependency: "Form validation library (React Hook Form + zod)"
    type: "COMPONENT"
    status: "AVAILABLE"

provides:
  - deliverable: "Patient registration UI components"
    interface: "React components with TypeScript props"
    consumers: "Main navigation, dashboard widgets, search flows"
```

### Code Patterns to Follow

```typescript
// src/features/patients/components/PatientRegistrationForm.tsx
import React, {useState} from 'react';
import {useForm, FormProvider} from 'react-hook-form';
import {zodResolver} from '@hookform/resolvers/zod';
import {useTranslation} from 'react-i18next';
import {useMutation, useQueryClient} from '@tanstack/react-query';
import {toast} from 'sonner';
import {Card, CardContent, CardHeader, CardTitle} from '@/components/ui/card';
import {Button} from '@/components/ui/button';
import {Progress} from '@/components/ui/progress';
import {PatientInfoStep} from './PatientInfoStep';
import {ContactInfoStep} from './ContactInfoStep';
import {EmergencyContactsStep} from './EmergencyContactsStep';
import {PhotoCaptureStep} from './PhotoCaptureStep';
import {patientApi} from '../api/patients.api';
import {createPatientSchema, CreatePatientFormData} from '../types/patient.types';

interface PatientRegistrationFormProps {
    onSuccess?: (patientId: string) => void;
    onCancel?: () => void;
    initialData?: Partial<CreatePatientFormData>;
}

const STEPS = [
    {key: 'info', titleKey: 'registration.steps.basic_info'},
    {key: 'contact', titleKey: 'registration.steps.contact_info'},
    {key: 'emergency', titleKey: 'registration.steps.emergency_contacts'},
    {key: 'photo', titleKey: 'registration.steps.photo', optional: true}
];

export const PatientRegistrationForm: React.FC<PatientRegistrationFormProps> = ({
                                                                                    onSuccess,
                                                                                    onCancel,
                                                                                    initialData
                                                                                }) => {
    const {t} = useTranslation('patients');
    const queryClient = useQueryClient();
    const [currentStep, setCurrentStep] = useState(0);
    const [isSubmitting, setIsSubmitting] = useState(false);

    const form = useForm<CreatePatientFormData>({
        resolver: zodResolver(createPatientSchema),
        defaultValues: {
            firstName: '',
            lastName: '',
            middleName: '',
            dateOfBirth: '',
            gender: undefined,
            phone: '',
            email: '',
            preferredContactMethod: 'phone',
            address: {
                street: '',
                city: '',
                state: '',
                zipCode: '',
                country: 'DZ'
            },
            emergencyContacts: [],
            ...initialData
        },
        mode: 'onChange'
    });

    const createPatientMutation = useMutation({
        mutationFn: patientApi.create,
        onSuccess: (patient) => {
            queryClient.invalidateQueries({queryKey: ['patients']});
            toast.success(t('registration.success.patient_created'));
            onSuccess?.(patient.id);
        },
        onError: (error) => {
            console.error('Patient registration failed:', error);
            toast.error(t('registration.error.creation_failed'));
        },
        onSettled: () => {
            setIsSubmitting(false);
        }
    });

    const handleNext = async () => {
        const currentStepKey = STEPS[currentStep].key;
        let fieldsToValidate: (keyof CreatePatientFormData)[] = [];

        // Define validation fields for each step
        switch (currentStepKey) {
            case 'info':
                fieldsToValidate = ['firstName', 'lastName', 'dateOfBirth', 'gender', 'phone'];
                break;
            case 'contact':
                fieldsToValidate = ['email', 'address'];
                break;
            case 'emergency':
                fieldsToValidate = ['emergencyContacts'];
                break;
        }

        const isValid = await form.trigger(fieldsToValidate);

        if (isValid) {
            if (currentStep < STEPS.length - 1) {
                setCurrentStep(currentStep + 1);
            } else {
                await handleSubmit();
            }
        }
    };

    const handlePrevious = () => {
        if (currentStep > 0) {
            setCurrentStep(currentStep - 1);
        }
    };

    const handleSubmit = async () => {
        const isValid = await form.trigger();
        if (!isValid) return;

        setIsSubmitting(true);
        const formData = form.getValues();

        try {
            // Transform form data to match API schema
            const patientData = {
                ...formData,
                dateOfBirth: new Date(formData.dateOfBirth),
                emergencyContacts: formData.emergencyContacts?.filter(contact => contact.name.trim())
            };

            await createPatientMutation.mutateAsync(patientData);
        } catch (error) {
            // Error handling is managed by the mutation
            console.error('Registration submission error:', error);
        }
    };

    const renderCurrentStep = () => {
        const stepKey = STEPS[currentStep].key;

        switch (stepKey) {
            case 'info':
                return <PatientInfoStep / >;
            case 'contact':
                return <ContactInfoStep / >;
            case 'emergency':
                return <EmergencyContactsStep / >;
            case 'photo':
                return <PhotoCaptureStep / >;
            default:
                return null;
        }
    };

    const progress = ((currentStep + 1) / STEPS.length) * 100;

    return (
        <div className = "max-w-4xl mx-auto p-6" >
        <Card>
            <CardHeader>
                <CardTitle className = "text-2xl" >
            {t('registration.title')
}
    </CardTitle>
    < div
    className = "space-y-2" >
    <Progress value = {progress}
    className = "w-full" / >
    <p className = "text-sm text-muted-foreground" >
        {t('registration.step_progress',
    {
        current: currentStep + 1,
            total
    :
        STEPS.length,
            step
    :
        t(STEPS[currentStep].titleKey)
    }
)
}
    </p>
    < /div>
    < /CardHeader>

    < CardContent
    className = "space-y-6" >
    <FormProvider {...form} >
    <form className = "space-y-6" >
        {renderCurrentStep()}

        < div
    className = "flex justify-between pt-6 border-t" >
    <div className = "flex gap-2" >
        {currentStep > 0 && (
            <Button
                type = "button"
    variant = "outline"
    onClick = {handlePrevious}
    disabled = {isSubmitting}
        >
        {t('common.previous'
)
}
    </Button>
)
}

    {
        onCancel && (
            <Button
                type = "button"
        variant = "ghost"
        onClick = {onCancel}
        disabled = {isSubmitting}
            >
            {t('common.cancel'
    )
    }
        </Button>
    )
    }
    </div>

    < Button
    type = "button"
    onClick = {handleNext}
    disabled = {isSubmitting}
    loading = {isSubmitting}
    >
    {currentStep === STEPS.length - 1
        ? t('registration.complete_registration')
        : t('common.next')
}
    </Button>
    < /div>
    < /form>
    < /FormProvider>
    < /CardContent>
    < /Card>
    < /div>
)
    ;
};

// src/features/patients/components/PatientInfoStep.tsx
import React from 'react';
import {useFormContext} from 'react-hook-form';
import {useTranslation} from 'react-i18next';
import {FormField, FormItem, FormLabel, FormControl, FormMessage} from '@/components/ui/form';
import {Input} from '@/components/ui/input';
import {Select, SelectContent, SelectItem, SelectTrigger, SelectValue} from '@/components/ui/select';
import {CalendarIcon} from 'lucide-react';
import {Calendar} from '@/components/ui/calendar';
import {Popover, PopoverContent, PopoverTrigger} from '@/components/ui/popover';
import {Button} from '@/components/ui/button';
import {cn} from '@/lib/utils';
import {format} from 'date-fns';
import {CreatePatientFormData} from '../types/patient.types';

export const PatientInfoStep: React.FC = () => {
    const {t} = useTranslation('patients');
    const form = useFormContext<CreatePatientFormData>();

    return (
        <div className = "space-y-6" >
        <div className = "grid grid-cols-1 md:grid-cols-2 gap-4" >
        <FormField
            control = {form.control}
    name = "firstName"
    render = {({field})
=>
    (
        <FormItem>
            <FormLabel className = "required" >
            {t('fields.first_name')
}
    </FormLabel>
    < FormControl >
    <Input
        {...field}
    placeholder = {t('placeholders.first_name'
)
}
    autoComplete = "given-name"
        / >
        </FormControl>
        < FormMessage / >
        </FormItem>
)
}
    />

    < FormField
    control = {form.control}
    name = "lastName"
    render = {({field})
=>
    (
        <FormItem>
            <FormLabel className = "required" >
            {t('fields.last_name')
}
    </FormLabel>
    < FormControl >
    <Input
        {...field}
    placeholder = {t('placeholders.last_name'
)
}
    autoComplete = "family-name"
        / >
        </FormControl>
        < FormMessage / >
        </FormItem>
)
}
    />
    < /div>

    < FormField
    control = {form.control}
    name = "middleName"
    render = {({field})
=>
    (
        <FormItem>
            <FormLabel>{t('fields.middle_name')
}
    </FormLabel>
    < FormControl >
    <Input
        {...field}
    placeholder = {t('placeholders.middle_name'
)
}
    autoComplete = "additional-name"
        / >
        </FormControl>
        < FormMessage / >
        </FormItem>
)
}
    />

    < div
    className = "grid grid-cols-1 md:grid-cols-2 gap-4" >
    <FormField
        control = {form.control}
    name = "dateOfBirth"
    render = {({field})
=>
    (
        <FormItem className = "flex flex-col" >
        <FormLabel className = "required" >
            {t('fields.date_of_birth')
}
    </FormLabel>
    < Popover >
    <PopoverTrigger asChild >
    <FormControl>
        <Button
            variant = "outline"
    className = {
        cn(
        "w-full pl-3 text-left font-normal",
    !field.value && "text-muted-foreground"
)
}
>
    {
        field.value ? (
            format(new Date(field.value), "PPP")
        ) : (
            <span>{t('placeholders.select_date')
    }
        </span>
    )
    }
    <CalendarIcon className = "ml-auto h-4 w-4 opacity-50" / >
        </Button>
        < /FormControl>
        < /PopoverTrigger>
        < PopoverContent
    className = "w-auto p-0"
    align = "start" >
    <Calendar
        mode = "single"
    selected = {field.value ? new Date(field.value) : undefined}
    onSelect = {(date)
=>
    field.onChange(date?.toISOString().split('T')[0])
}
    disabled = {(date)
=>
    date > new Date() || date < new Date("1900-01-01")
}
    initialFocus
    / >
    </PopoverContent>
    < /Popover>
    < FormMessage / >
    </FormItem>
)
}
    />

    < FormField
    control = {form.control}
    name = "gender"
    render = {({field})
=>
    (
        <FormItem>
            <FormLabel className = "required" >
            {t('fields.gender')
}
    </FormLabel>
    < Select
    onValueChange = {field.onChange}
    defaultValue = {field.value} >
    <FormControl>
        <SelectTrigger>
            <SelectValue placeholder = {t('placeholders.select_gender'
)
}
    />
    < /SelectTrigger>
    < /FormControl>
    < SelectContent >
    <SelectItem value = "male" > {t('gender.male'
)
}
    </SelectItem>
    < SelectItem
    value = "female" > {t('gender.female'
)
}
    </SelectItem>
    < SelectItem
    value = "other" > {t('gender.other'
)
}
    </SelectItem>
    < /SelectContent>
    < /Select>
    < FormMessage / >
    </FormItem>
)
}
    />
    < /div>

    < div
    className = "grid grid-cols-1 md:grid-cols-2 gap-4" >
    <FormField
        control = {form.control}
    name = "phone"
    render = {({field})
=>
    (
        <FormItem>
            <FormLabel className = "required" >
            {t('fields.phone')
}
    </FormLabel>
    < FormControl >
    <Input
        {...field}
    type = "tel"
    placeholder = {t('placeholders.phone'
)
}
    autoComplete = "tel"
        / >
        </FormControl>
        < FormMessage / >
        </FormItem>
)
}
    />

    < FormField
    control = {form.control}
    name = "email"
    render = {({field})
=>
    (
        <FormItem>
            <FormLabel>{t('fields.email')
}
    </FormLabel>
    < FormControl >
    <Input
        {...field}
    type = "email"
    placeholder = {t('placeholders.email'
)
}
    autoComplete = "email"
        / >
        </FormControl>
        < FormMessage / >
        </FormItem>
)
}
    />
    < /div>

    < FormField
    control = {form.control}
    name = "preferredContactMethod"
    render = {({field})
=>
    (
        <FormItem>
            <FormLabel>{t('fields.preferred_contact_method')
}
    </FormLabel>
    < Select
    onValueChange = {field.onChange}
    defaultValue = {field.value} >
    <FormControl>
        <SelectTrigger>
            <SelectValue / >
    </SelectTrigger>
    < /FormControl>
    < SelectContent >
    <SelectItem value = "phone" > {t('contact_method.phone'
)
}
    </SelectItem>
    < SelectItem
    value = "email" > {t('contact_method.email'
)
}
    </SelectItem>
    < SelectItem
    value = "sms" > {t('contact_method.sms'
)
}
    </SelectItem>
    < /SelectContent>
    < /Select>
    < FormMessage / >
    </FormItem>
)
}
    />
    < /div>
)
    ;
};

// src/features/patients/components/EmergencyContactsStep.tsx
import React from 'react';
import {useFieldArray, useFormContext} from 'react-hook-form';
import {useTranslation} from 'react-i18next';
import {Plus, Trash2} from 'lucide-react';
import {Button} from '@/components/ui/button';
import {Card, CardContent, CardHeader, CardTitle} from '@/components/ui/card';
import {FormField, FormItem, FormLabel, FormControl, FormMessage} from '@/components/ui/form';
import {Input} from '@/components/ui/input';
import {Select, SelectContent, SelectItem, SelectTrigger, SelectValue} from '@/components/ui/select';
import {Checkbox} from '@/components/ui/checkbox';
import {CreatePatientFormData} from '../types/patient.types';

export const EmergencyContactsStep: React.FC = () => {
    const {t} = useTranslation('patients');
    const form = useFormContext<CreatePatientFormData>();

    const {fields, append, remove} = useFieldArray({
        control: form.control,
        name: "emergencyContacts"
    });

    const addEmergencyContact = () => {
        append({
            name: '',
            relationship: '',
            phone: '',
            email: '',
            isPrimary: false
        });
    };

    const removeEmergencyContact = (index: number) => {
        remove(index);
    };

    return (
        <div className = "space-y-6" >
        <div className = "flex justify-between items-center" >
        <div>
            <h3 className = "text-lg font-medium" > {t('emergency_contacts.title')
}
    </h3>
    < p
    className = "text-sm text-muted-foreground" >
        {t('emergency_contacts.description'
)
}
    </p>
    < /div>
    < Button
    type = "button"
    variant = "outline"
    size = "sm"
    onClick = {addEmergencyContact}
    className = "flex items-center gap-2"
    >
    <Plus className = "h-4 w-4" / >
        {t('emergency_contacts.add_contact'
)
}
    </Button>
    < /div>

    {
        fields.length === 0 && (
            <Card className = "border-dashed" >
            <CardContent className = "flex flex-col items-center justify-center py-8" >
            <p className = "text-muted-foreground text-center" >
                {t('emergency_contacts.no_contacts')
    }
        </p>
        < Button
        type = "button"
        variant = "outline"
        onClick = {addEmergencyContact}
        className = "mt-4"
            >
            {t('emergency_contacts.add_first_contact'
    )
    }
        </Button>
        < /CardContent>
        < /Card>
    )
    }

    {
        fields.map((field, index) => (
            <Card key = {field.id} >
            <CardHeader className = "flex flex-row items-center justify-between space-y-0 pb-4" >
            <CardTitle className = "text-base" >
                {t('emergency_contacts.contact_number',
        {
            number: index + 1
        }
    )
    }
        </CardTitle>
        < Button
        type = "button"
        variant = "ghost"
        size = "sm"
        onClick = {()
    =>
        removeEmergencyContact(index)
    }
        className = "text-destructive hover:text-destructive"
        >
        <Trash2 className = "h-4 w-4" / >
            </Button>
            < /CardHeader>
            < CardContent
        className = "space-y-4" >
        <div className = "grid grid-cols-1 md:grid-cols-2 gap-4" >
        <FormField
            control = {form.control}
        name = {`emergencyContacts.${index}.name`
    }
        render = {({field})
    =>
        (
            <FormItem>
                <FormLabel className = "required" >
                {t('fields.full_name')
    }
        </FormLabel>
        < FormControl >
        <Input
            {...field}
        placeholder = {t('placeholders.emergency_contact_name'
    )
    }
        />
        < /FormControl>
        < FormMessage / >
        </FormItem>
    )
    }
        />

        < FormField
        control = {form.control}
        name = {`emergencyContacts.${index}.relationship`
    }
        render = {({field})
    =>
        (
            <FormItem>
                <FormLabel className = "required" >
                {t('fields.relationship')
    }
        </FormLabel>
        < Select
        onValueChange = {field.onChange}
        defaultValue = {field.value} >
        <FormControl>
            <SelectTrigger>
                <SelectValue placeholder = {t('placeholders.select_relationship'
    )
    }
        />
        < /SelectTrigger>
        < /FormControl>
        < SelectContent >
        <SelectItem value = "spouse" > {t('relationships.spouse'
    )
    }
        </SelectItem>
        < SelectItem
        value = "child" > {t('relationships.child'
    )
    }
        </SelectItem>
        < SelectItem
        value = "parent" > {t('relationships.parent'
    )
    }
        </SelectItem>
        < SelectItem
        value = "sibling" > {t('relationships.sibling'
    )
    }
        </SelectItem>
        < SelectItem
        value = "guardian" > {t('relationships.guardian'
    )
    }
        </SelectItem>
        < SelectItem
        value = "other" > {t('relationships.other'
    )
    }
        </SelectItem>
        < /SelectContent>
        < /Select>
        < FormMessage / >
        </FormItem>
    )
    }
        />
        < /div>

        < div
        className = "grid grid-cols-1 md:grid-cols-2 gap-4" >
        <FormField
            control = {form.control}
        name = {`emergencyContacts.${index}.phone`
    }
        render = {({field})
    =>
        (
            <FormItem>
                <FormLabel className = "required" >
                {t('fields.phone')
    }
        </FormLabel>
        < FormControl >
        <Input
            {...field}
        type = "tel"
        placeholder = {t('placeholders.emergency_contact_phone'
    )
    }
        />
        < /FormControl>
        < FormMessage / >
        </FormItem>
    )
    }
        />

        < FormField
        control = {form.control}
        name = {`emergencyContacts.${index}.email`
    }
        render = {({field})
    =>
        (
            <FormItem>
                <FormLabel>{t('fields.email')
    }
        </FormLabel>
        < FormControl >
        <Input
            {...field}
        type = "email"
        placeholder = {t('placeholders.emergency_contact_email'
    )
    }
        />
        < /FormControl>
        < FormMessage / >
        </FormItem>
    )
    }
        />
        < /div>

        < FormField
        control = {form.control}
        name = {`emergencyContacts.${index}.isPrimary`
    }
        render = {({field})
    =>
        (
            <FormItem className = "flex flex-row items-start space-x-3 space-y-0" >
            <FormControl>
                <Checkbox
                    checked = {field.value}
        onCheckedChange = {field.onChange}
        />
        < /FormControl>
        < div
        className = "space-y-1 leading-none" >
            <FormLabel>
                {t('emergency_contacts.primary_contact'
    )
    }
        </FormLabel>
        < p
        className = "text-xs text-muted-foreground" >
            {t('emergency_contacts.primary_contact_description'
    )
    }
        </p>
        < /div>
        < /FormItem>
    )
    }
        />
        < /CardContent>
        < /Card>
    ))
    }
    </div>
)
    ;
};

// src/features/patients/api/patients.api.ts
import {apiClient} from '@/lib/api/client';
import {CreatePatientRequest, Patient, PatientSummary, PaginatedResult} from '../types/patient.types';

export const patientApi = {
    async create(data: CreatePatientRequest): Promise<Patient> {
        const response = await apiClient.post('/patients', data);
        return response.data;
    },

    async getById(id: string): Promise<Patient> {
        const response = await apiClient.get(`/patients/${id}`);
        return response.data;
    },

    async search(params: SearchPatientsParams): Promise<PaginatedResult<PatientSummary>> {
        const response = await apiClient.get('/patients', {params});
        return response.data;
    },

    async update(id: string, data: UpdatePatientRequest): Promise<Patient> {
        const response = await apiClient.put(`/patients/${id}`, data);
        return response.data;
    }
};
```

## Testing Requirements

### Test Coverage

```yaml
unit_tests:
  - focus: "Component rendering and form validation logic"
  - coverage_target: "85%"
  - key_scenarios: [
    "Form validation for all steps",
    "Multi-step navigation",
    "Emergency contact management",
    "Language switching functionality",
    "Error handling and display"
  ]

integration_tests:
  - focus: "Complete registration workflow and API integration"
  - test_environment: "Testing environment with mock API"
  - key_flows: [
    "End-to-end patient registration",
    "Form validation with backend API",
    "Photo upload functionality",
    "Multi-language form submission"
  ]
```

### Test Scenarios

```yaml
happy_path:
  - scenario: "Complete patient registration with all information"
    steps: [
      "Fill out basic patient information",
      "Complete contact and address information",
      "Add emergency contacts",
      "Capture patient photo",
      "Submit form successfully"
    ]
    expected_result: "Patient created and success message displayed"

error_cases:
  - scenario: "Submit form with validation errors"
    trigger: "Attempt to proceed with incomplete required fields"
    expected_behavior: "Validation errors displayed, step navigation blocked"

  - scenario: "API error during registration"
    trigger: "Backend API returns error response"
    expected_behavior: "Error message displayed, form data preserved"

edge_cases:
  - scenario: "Registration with Arabic patient name"
    conditions: "Patient name contains Arabic characters"
    expected_behavior: "Form handles RTL text correctly, submission successful"
```

## Context References

### From Project

```yaml
tech_stack_references:
  - "React with TypeScript for component development"
  - "React Hook Form with zod for validation"
  - "shadcn/ui for consistent UI components"
  - "react-i18next for internationalization"

architecture_patterns:
  - "Feature-based component organization"
  - "Form state management with React Hook Form"
  - "API integration with TanStack Query"

code_standards:
  - "TypeScript interfaces for all props and data"
  - "Consistent error handling and user feedback"
  - "Accessibility compliance with ARIA attributes"
```

### From Epic

```yaml
business_rules:
  - "BR-P001: Auto-generated unique patient ID"
  - "BR-P002: Required fields validation"
  - "BR-P004: Date of birth validation"
  - "BR-P016: Duplicate detection integration"

domain_model:
  - "CreatePatientFormData interface"
  - "Emergency contact management"
  - "Address information structure"

integration_points:
  - "Integrates with Patient Registration API"
  - "Provides patient data for search and profile workflows"
  - "Supports appointment scheduling patient selection"
```

## Acceptance Criteria

### Functional Acceptance

- [ ] Multi-step patient registration form captures all required information
- [ ] Real-time validation provides immediate feedback for all fields
- [ ] Emergency contact management allows multiple contacts with primary designation
- [ ] Arabic and French language support with proper RTL layout
- [ ] Form state is preserved during step navigation

### Technical Acceptance

- [ ] Components follow React and TypeScript best practices
- [ ] Unit tests written and passing (>= 85% coverage)
- [ ] Integration tests cover complete registration workflow
- [ ] Form validation prevents invalid data submission
- [ ] Error handling provides clear user feedback
- [ ] Performance requirements met for form interactions

### Quality Acceptance

- [ ] Code review completed and approved
- [ ] Accessibility testing passes WCAG 2.1 AA standards
- [ ] Cross-browser testing completed
- [ ] Multi-language testing validates proper text display
- [ ] User experience testing confirms intuitive workflow

---
**Last Updated:** 2025-01-16