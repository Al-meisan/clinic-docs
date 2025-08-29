# TASK: TASK-009 - Multi-language Support Integration

## Task Overview

### Goal

**What to Build:** Comprehensive multi-language support system with Arabic and French localization, RTL layout support,
and proper character encoding for patient data entry and display throughout the patient management system.

**Why:** Support Algerian healthcare providers by providing native language interfaces in Arabic and French, ensuring
cultural appropriateness and improved usability for healthcare staff and patients.

**Definition of Done:**

- Complete Arabic and French translation coverage for all patient management interfaces
- RTL (Right-to-Left) layout support for Arabic with proper text direction
- Proper character encoding and validation for Arabic and French patient data
- Dynamic language switching with persistent user preferences
- Localized date, number, and address formatting

### Scope

```yaml
task_type: "FRONTEND"
complexity: "MEDIUM"
```

## Requirements

### Functional Requirements

1. **Translation System Implementation**
    - Description: Complete translation coverage for all patient management interfaces
    - Acceptance Criteria:
        - Arabic and French translations for all UI text and labels
        - Proper pluralization handling for both languages
        - Context-aware translations for medical terminology
        - Translation validation and completeness checking
    - Priority: HIGH

2. **RTL Layout Support**
    - Description: Right-to-left layout support for Arabic language interface
    - Acceptance Criteria:
        - Automatic layout direction switching based on selected language
        - Proper RTL alignment for forms, tables, and navigation elements
        - Mirrored icons and UI elements where culturally appropriate
        - Consistent spacing and alignment in RTL mode
    - Priority: HIGH

3. **Character Encoding and Validation**
    - Description: Proper handling of Arabic and French characters in patient data
    - Acceptance Criteria:
        - UTF-8 character encoding support throughout the system
        - Validation rules that support Arabic and French character sets
        - Proper text input handling for diacritics and special characters
        - Search functionality that works with Arabic and French text
    - Priority: HIGH

4. **Dynamic Language Switching**
    - Description: Runtime language switching with user preference persistence
    - Acceptance Criteria:
        - Language selector in application header
        - Immediate UI language switching without page reload
        - User language preference stored in profile and localStorage
        - Language setting synchronized across user sessions
    - Priority: MEDIUM

5. **Localized Formatting**
    - Description: Culture-specific formatting for dates, numbers, and addresses
    - Acceptance Criteria:
        - Arabic and French date formatting using appropriate calendars
        - Number formatting with correct decimal separators and digit grouping
        - Address format localization for Algerian address standards
        - Phone number formatting for Algerian number patterns
    - Priority: MEDIUM

### Technical Requirements

```yaml
security:
  - requirement: "Character encoding security"
    implementation: "Prevent XSS through proper character encoding and validation"

  - requirement: "Input validation for multi-language text"
    implementation: "Validate Arabic and French text input properly"

compatibility:
  - requirement: "Font support for Arabic script"
    scope: "Ensure proper Arabic font rendering across browsers"
```

## Implementation Details

### Technical Approach

```yaml
# Multi-language Implementation Strategy
frontend_implementation:
  - library: "react-i18next with i18next"
    features: "Translation management, namespace organization, lazy loading"

  - component: "LanguageProvider"
    purpose: "Global language state management and context"
    features: "Language switching, persistence, direction detection"

  - component: "RTLProvider"
    purpose: "Layout direction management for Arabic"
    features: "Automatic RTL/LTR switching, CSS direction handling"

translation_structure:
  - organization: "Feature-based namespaces (patients, common, forms)"
    format: "Nested JSON with interpolation support"
    validation: "Translation completeness checking and missing key detection"

  - arabic_support: "Proper Arabic text rendering and shaping"
    features: "Diacritics support, contextual character forms"

  - french_support: "French accent and special character handling"
    features: "Proper accent rendering, French-specific formatting"

styling_approach:
  - css_strategy: "CSS logical properties for directional layouts"
    implementation: "margin-inline-start/end instead of left/right"

  - rtl_handling: "Automatic layout mirroring and text direction"
    tools: "CSS dir() pseudo-class and [dir] attribute selectors"
```

### Integration Points

```yaml
dependencies:
  - dependency: "Patient Registration Frontend (TASK-004)"
    type: "COMPONENT"
    status: "IN_PROGRESS"

  - dependency: "Patient Search Frontend (TASK-005)"
    type: "COMPONENT"
    status: "IN_PROGRESS"

  - dependency: "Patient Profile Frontend (TASK-006)"
    type: "COMPONENT"
    status: "IN_PROGRESS"

provides:
  - deliverable: "Multi-language infrastructure and translation system"
    interface: "React context providers and translation hooks"
    consumers: "All frontend patient management components"
```

### Code Patterns to Follow

```typescript
// src/lib/i18n/config.ts
import i18n from 'i18next';
import {initReactI18next} from 'react-i18next';
import LanguageDetector from 'i18next-browser-languagedetector';
import Backend from 'i18next-http-backend';

// Language resources
const resources = {
    ar: {
        common: () => import('../../assets/locales/ar/common.json'),
        patients: () => import('../../assets/locales/ar/patients.json'),
        forms: () => import('../../assets/locales/ar/forms.json'),
        errors: () => import('../../assets/locales/ar/errors.json')
    },
    fr: {
        common: () => import('../../assets/locales/fr/common.json'),
        patients: () => import('../../assets/locales/fr/patients.json'),
        forms: () => import('../../assets/locales/fr/forms.json'),
        errors: () => import('../../assets/locales/fr/errors.json')
    }
};

i18n
    .use(Backend)
    .use(LanguageDetector)
    .use(initReactI18next)
    .init({
        fallbackLng: 'fr',
        supportedLngs: ['ar', 'fr'],

        // Language detection configuration
        detection: {
            order: ['localStorage', 'navigator', 'htmlTag'],
            caches: ['localStorage'],
            lookupLocalStorage: 'i18nextLng'
        },

        // Backend configuration for lazy loading
        backend: {
            loadPath: '/locales/{{lng}}/{{ns}}.json',
            addPath: '/locales/add/{{lng}}/{{ns}}'
        },

        // React i18next configuration
        react: {
            useSuspense: true,
            transSupportBasicHtmlNodes: true,
            transKeepBasicHtmlNodesFor: ['br', 'strong', 'i', 'em']
        },

        // Interpolation configuration
        interpolation: {
            escapeValue: false, // React already does escaping
            formatSeparator: ',',
            format: (value, format, lng) => {
                if (format === 'uppercase') return value.toUpperCase();
                if (format === 'currency') return formatCurrency(value, lng);
                if (format === 'date') return formatDate(value, lng);
                return value;
            }
        },

        // Debugging in development
        debug: process.env.NODE_ENV === 'development'
    });

export default i18n;

// src/providers/LanguageProvider.tsx
import React, {createContext, useContext, useEffect, useState} from 'react';
import {useTranslation} from 'react-i18next';

interface LanguageContextType {
    language: string;
    direction: 'ltr' | 'rtl';
    isRTL: boolean;
    changeLanguage: (lng: string) => void;
    isLoading: boolean;
}

const LanguageContext = createContext<LanguageContextType | undefined>(undefined);

interface LanguageProviderProps {
    children: React.ReactNode;
}

export const LanguageProvider: React.FC<LanguageProviderProps> = ({children}) => {
    const {i18n} = useTranslation();
    const [isLoading, setIsLoading] = useState(false);

    const language = i18n.language;
    const direction = language === 'ar' ? 'rtl' : 'ltr';
    const isRTL = direction === 'rtl';

    // Update document direction and lang attribute
    useEffect(() => {
        document.documentElement.dir = direction;
        document.documentElement.lang = language;

        // Update body class for styling
        document.body.classList.toggle('rtl', isRTL);
        document.body.classList.toggle('ltr', !isRTL);
    }, [direction, language, isRTL]);

    const changeLanguage = async (lng: string) => {
        setIsLoading(true);
        try {
            await i18n.changeLanguage(lng);

            // Update user preference if authenticated
            const user = getCurrentUser();
            if (user) {
                await updateUserLanguagePreference(lng);
            }
        } catch (error) {
            console.error('Failed to change language:', error);
        } finally {
            setIsLoading(false);
        }
    };

    const value: LanguageContextType = {
        language,
        direction,
        isRTL,
        changeLanguage,
        isLoading
    };

    return (
        <LanguageContext.Provider value = {value} >
            {children}
            < /LanguageContext.Provider>
    );
};

export const useLanguage = (): LanguageContextType => {
    const context = useContext(LanguageContext);
    if (context === undefined) {
        throw new Error('useLanguage must be used within a LanguageProvider');
    }
    return context;
};

// src/components/LanguageSelector.tsx
import React from 'react';
import {useTranslation} from 'react-i18next';
import {Button} from '@/components/ui/button';
import {
    DropdownMenu,
    DropdownMenuContent,
    DropdownMenuItem,
    DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu';
import {Globe, Check} from 'lucide-react';
import {useLanguage} from '@/providers/LanguageProvider';

const SUPPORTED_LANGUAGES = [
    {code: 'ar', name: 'العربية', flag: '🇩🇿'},
    {code: 'fr', name: 'Français', flag: '🇫🇷'}
];

export const LanguageSelector: React.FC = () => {
    const {t} = useTranslation('common');
    const {language, changeLanguage, isLoading} = useLanguage();

    const currentLanguage = SUPPORTED_LANGUAGES.find(lang => lang.code === language);

    return (
        <DropdownMenu>
            <DropdownMenuTrigger asChild >
        <Button
            variant = "ghost"
    size = "sm"
    className = "gap-2"
    disabled = {isLoading}
    >
    <Globe className = "h-4 w-4" / >
    <span className = "hidden sm:inline" >
        {currentLanguage?.flag
}
    {
        currentLanguage?.name
    }
    </span>
    < /Button>
    < /DropdownMenuTrigger>
    < DropdownMenuContent
    align = "end"
    className = "w-48" >
        {
            SUPPORTED_LANGUAGES.map((lang) => (
                <DropdownMenuItem
                    key = {lang.code}
            onClick = {()
=>
    changeLanguage(lang.code)
}
    className = "flex items-center gap-3"
    >
    <span className = "text-lg" > {lang.flag} < /span>
        < span
    className = "flex-1" > {lang.name} < /span>
    {
        language === lang.code && (
            <Check className = "h-4 w-4 text-primary" / >
        )
    }
    </DropdownMenuItem>
))
}
    </DropdownMenuContent>
    < /DropdownMenu>
)
    ;
};

// src/hooks/useLocalization.ts
import {useTranslation} from 'react-i18next';
import {useLanguage} from '@/providers/LanguageProvider';
import {format as dateFnsFormat} from 'date-fns';
import {ar, fr} from 'date-fns/locale';

export const useLocalization = () => {
    const {t, i18n} = useTranslation();
    const {language, isRTL} = useLanguage();

    const formatDate = (date: Date | string, formatStr: string = 'PPP'): string => {
        const dateObj = typeof date === 'string' ? new Date(date) : date;
        const locale = language === 'ar' ? ar : fr;
        return dateFnsFormat(dateObj, formatStr, {locale});
    };

    const formatCurrency = (amount: number, currency: string = 'DZD'): string => {
        return new Intl.NumberFormat(language === 'ar' ? 'ar-DZ' : 'fr-DZ', {
            style: 'currency',
            currency: currency,
            minimumFractionDigits: 2
        }).format(amount);
    };

    const formatNumber = (number: number): string => {
        return new Intl.NumberFormat(language === 'ar' ? 'ar-DZ' : 'fr-DZ').format(number);
    };

    const formatPhoneNumber = (phone: string): string => {
        // Algerian phone number formatting
        const cleaned = phone.replace(/\D/g, '');
        if (cleaned.startsWith('213')) {
            // International format
            const national = cleaned.slice(3);
            return `+213 ${national.slice(0, 1)} ${national.slice(1, 3)} ${national.slice(3, 5)} ${national.slice(5)}`;
        } else if (cleaned.startsWith('0')) {
            // National format
            return `${cleaned.slice(0, 2)} ${cleaned.slice(2, 4)} ${cleaned.slice(4, 6)} ${cleaned.slice(6)}`;
        }
        return phone;
    };

    const getTextDirection = (text: string): 'ltr' | 'rtl' => {
        // Simple Arabic text detection
        const arabicRegex = /[\u0600-\u06FF\u0750-\u077F]/;
        return arabicRegex.test(text) ? 'rtl' : 'ltr';
    };

    const validateName = (name: string): boolean => {
        // Allow Arabic, French, and basic Latin characters
        const validNameRegex = /^[\u0600-\u06FF\u0750-\u077F\u00C0-\u017F\u0020\u002D\u002E\u0027a-zA-Z\s]+$/;
        return validNameRegex.test(name.trim());
    };

    return {
        t,
        language,
        isRTL,
        formatDate,
        formatCurrency,
        formatNumber,
        formatPhoneNumber,
        getTextDirection,
        validateName
    };
};

// src/components/ui/MultiLanguageInput.tsx
import React, {forwardRef} from 'react';
import {Input, InputProps} from '@/components/ui/input';
import {useLanguage} from '@/providers/LanguageProvider';
import {cn} from '@/lib/utils';

interface MultiLanguageInputProps extends InputProps {
    supportedLanguages?: string[];
}

export const MultiLanguageInput = forwardRef<HTMLInputElement, MultiLanguageInputProps>(
    ({className, supportedLanguages = ['ar', 'fr'], ...props}, ref) => {
        const {language, isRTL} = useLanguage();

        return (
            <Input
                ref = {ref}
        className = {
            cn(
                className,
                isRTL && 'text-right',
        language === 'ar' && 'font-arabic'
    )
    }
        dir = {isRTL ? 'rtl' : 'ltr'}
        {...
            props
        }
        />
    )
        ;
    }
);

MultiLanguageInput.displayName = 'MultiLanguageInput';

// src/assets/locales/ar/patients.json
{
    "title"
:
    "إدارة المرضى",
        "search"
:
    {
        "title"
    :
        "البحث عن المرضى",
            "placeholder"
    :
        "ابحث بالاسم أو رقم الهاتف أو تاريخ الميلاد",
            "no_results"
    :
        "لم يتم العثور على نتائج",
            "searching"
    :
        "جاري البحث...",
            "results_count"
    :
        "تم العثور على {{count}} مريض",
            "results_count_plural"
    :
        "تم العثور على {{count}} مرضى"
    }
,
    "registration"
:
    {
        "title"
    :
        "تسجيل مريض جديد",
            "steps"
    :
        {
            "basic_info"
        :
            "المعلومات الأساسية",
                "contact_info"
        :
            "معلومات التواصل",
                "emergency_contacts"
        :
            "جهات الاتصال الطارئة",
                "photo"
        :
            "الصورة الشخصية"
        }
    ,
        "success"
    :
        {
            "patient_created"
        :
            "تم تسجيل المريض بنجاح"
        }
    ,
        "error"
    :
        {
            "creation_failed"
        :
            "فشل في تسجيل المريض"
        }
    }
,
    "fields"
:
    {
        "first_name"
    :
        "الاسم الأول",
            "last_name"
    :
        "اسم العائلة",
            "middle_name"
    :
        "الاسم الأوسط",
            "date_of_birth"
    :
        "تاريخ الميلاد",
            "gender"
    :
        "الجنس",
            "phone"
    :
        "رقم الهاتف",
            "email"
    :
        "البريد الإلكتروني",
            "address"
    :
        "العنوان",
            "emergency_contact"
    :
        "جهة الاتصال الطارئة"
    }
,
    "gender"
:
    {
        "male"
    :
        "ذكر",
            "female"
    :
        "أنثى",
            "other"
    :
        "آخر"
    }
,
    "placeholders"
:
    {
        "first_name"
    :
        "أدخل الاسم الأول",
            "last_name"
    :
        "أدخل اسم العائلة",
            "phone"
    :
        "أدخل رقم الهاتف",
            "email"
    :
        "أدخل البريد الإلكتروني",
            "search_patients"
    :
        "ابحث عن المرضى..."
    }
,
    "patient_status"
:
    {
        "active"
    :
        "نشط",
            "inactive"
    :
        "غير نشط",
            "deceased"
    :
        "متوفى",
            "merged"
    :
        "مدمج"
    }
,
    "profile"
:
    {
        "age_gender"
    :
        "العمر {{age}} سنة، {{gender}}",
            "tabs"
    :
        {
            "overview"
        :
            "نظرة عامة",
                "contact"
        :
            "معلومات التواصل",
                "history"
        :
            "السجل"
        }
    ,
        "sections"
    :
        {
            "basic_info"
        :
            "المعلومات الأساسية",
                "contact_info"
        :
            "معلومات التواصل",
                "emergency_contacts"
        :
            "جهات الاتصال الطارئة"
        }
    }
}

// src/assets/locales/fr/patients.json
{
    "title"
:
    "Gestion des Patients",
        "search"
:
    {
        "title"
    :
        "Recherche de Patients",
            "placeholder"
    :
        "Rechercher par nom, téléphone ou date de naissance",
            "no_results"
    :
        "Aucun résultat trouvé",
            "searching"
    :
        "Recherche en cours...",
            "results_count"
    :
        "{{count}} patient trouvé",
            "results_count_plural"
    :
        "{{count}} patients trouvés"
    }
,
    "registration"
:
    {
        "title"
    :
        "Enregistrement d'un Nouveau Patient",
            "steps"
    :
        {
            "basic_info"
        :
            "Informations de Base",
                "contact_info"
        :
            "Informations de Contact",
                "emergency_contacts"
        :
            "Contacts d'Urgence",
                "photo"
        :
            "Photo"
        }
    ,
        "success"
    :
        {
            "patient_created"
        :
            "Patient enregistré avec succès"
        }
    ,
        "error"
    :
        {
            "creation_failed"
        :
            "Échec de l'enregistrement du patient"
        }
    }
,
    "fields"
:
    {
        "first_name"
    :
        "Prénom",
            "last_name"
    :
        "Nom de Famille",
            "middle_name"
    :
        "Deuxième Prénom",
            "date_of_birth"
    :
        "Date de Naissance",
            "gender"
    :
        "Sexe",
            "phone"
    :
        "Téléphone",
            "email"
    :
        "Email",
            "address"
    :
        "Adresse",
            "emergency_contact"
    :
        "Contact d'Urgence"
    }
,
    "gender"
:
    {
        "male"
    :
        "Masculin",
            "female"
    :
        "Féminin",
            "other"
    :
        "Autre"
    }
,
    "placeholders"
:
    {
        "first_name"
    :
        "Entrez le prénom",
            "last_name"
    :
        "Entrez le nom de famille",
            "phone"
    :
        "Entrez le numéro de téléphone",
            "email"
    :
        "Entrez l'adresse email",
            "search_patients"
    :
        "Rechercher des patients..."
    }
,
    "patient_status"
:
    {
        "active"
    :
        "Actif",
            "inactive"
    :
        "Inactif",
            "deceased"
    :
        "Décédé",
            "merged"
    :
        "Fusionné"
    }
,
    "profile"
:
    {
        "age_gender"
    :
        "{{age}} ans, {{gender}}",
            "tabs"
    :
        {
            "overview"
        :
            "Aperçu",
                "contact"
        :
            "Contact",
                "history"
        :
            "Historique"
        }
    ,
        "sections"
    :
        {
            "basic_info"
        :
            "Informations de Base",
                "contact_info"
        :
            "Informations de Contact",
                "emergency_contacts"
        :
            "Contacts d'Urgence"
        }
    }
}

// src/styles/rtl.css
/* RTL-specific styles */
.
rtl
{
    direction: rtl;
    text - align
:
    right;
}

.
ltr
{
    direction: ltr;
    text - align
:
    left;
}

/* CSS Logical Properties for better RTL support */
.
patient - card
{
    margin - inline - start
:
    1
    rem;
    margin - inline - end
:
    0.5
    rem;
    padding - inline
:
    1
    rem;
    border - inline - start
:
    2
    px
    solid
    var (
    --primary
)
    ;
}

/* RTL-specific component adjustments */
.
rtl.patient - search - icon
{
    margin - inline - start
:
    0.5
    rem;
    margin - inline - end
:
    0;
}

.
rtl.breadcrumb - separator
{
    transform: scaleX(-1);
}

/* Arabic font family */
.
font - arabic
{
    font - family
:
    'Noto Sans Arabic', 'Arial Unicode MS', sans - serif;
}

/* Form layout adjustments for RTL */
.
rtl.form - grid
{
    grid - template - columns
:
    repeat(auto - fit, minmax(250
    px, 1
    fr
))
    ;
}

.
rtl.form - label
{
    text - align
:
    right;
}

/* Navigation adjustments */
.
rtl.nav - menu
{
    padding - inline - start
:
    0;
    padding - inline - end
:
    1
    rem;
}

.
rtl.dropdown - menu
{
    left: auto;
    right: 0;
}

/* Table RTL support */
.
rtl
table
{
    direction: rtl;
}

.
rtl
th,
.
rtl
td
{
    text - align
:
    right;
}

.
rtl
th:first - child,
.
rtl
td:first - child
{
    border - radius
:
    0
    0.375
    rem
    0.375
    rem
    0;
}

.
rtl
th:last - child,
.
rtl
td:last - child
{
    border - radius
:
    0.375
    rem
    0
    0
    0.375
    rem;
}
```

## Testing Requirements

### Test Coverage

```yaml
unit_tests:
  - focus: "Translation functionality and language switching"
  - coverage_target: "80%"
  - key_scenarios: [
    "Language switching and preference persistence",
    "RTL layout rendering and direction changes",
    "Text validation for Arabic and French characters",
    "Date and number formatting in different locales",
    "Translation key completeness and fallbacks"
  ]

integration_tests:
  - focus: "Complete multi-language workflow testing"
  - test_environment: "Testing environment with Arabic and French content"
  - key_flows: [
    "Patient registration in Arabic and French",
    "Search functionality with multi-language input",
    "Profile management with RTL layout",
    "Language switching across all components"
  ]
```

### Test Scenarios

```yaml
happy_path:
  - scenario: "Switch language from French to Arabic"
    steps: [
      "Start with French interface",
      "Click language selector",
      "Select Arabic language",
      "Verify UI switches to Arabic with RTL layout"
    ]
    expected_result: "Complete interface translation with proper RTL layout"

error_cases:
  - scenario: "Invalid Arabic characters in input field"
    trigger: "Enter invalid characters in Arabic name field"
    expected_behavior: "Validation error with appropriate Arabic error message"

  - scenario: "Missing translation key"
    trigger: "Render component with missing translation"
    expected_behavior: "Fallback to key name or default language, error logged"

edge_cases:
  - scenario: "Mixed Arabic and French text in patient names"
    conditions: "Patient name contains both Arabic and French characters"
    expected_behavior: "Proper rendering with appropriate text direction detection"
```

## Context References

### From Project

```yaml
tech_stack_references:
  - "react-i18next for React translation management"
  - "i18next with lazy loading for performance"
  - "CSS logical properties for RTL layout support"
  - "Intl API for locale-specific formatting"

architecture_patterns:
  - "Context provider pattern for global language state"
  - "Namespace-based translation organization"
  - "Lazy loading pattern for translation resources"

code_standards:
  - "Proper accessibility with language attributes"
  - "Consistent translation key naming conventions"
  - "Performance optimization for language switching"
```

### From Epic

```yaml
business_rules:
  - "Support Arabic and French as primary languages"
  - "RTL layout required for Arabic interface"
  - "User language preference persistence"
  - "Cultural appropriateness for Algerian healthcare context"

domain_model:
  - "User language preference in profile"
  - "Multi-language validation for patient data"
  - "Localized formatting for dates and numbers"

integration_points:
  - "Integrates with all patient management components"
  - "Supports backend API with proper character encoding"
  - "Enables culturally appropriate user interfaces"
```

## Acceptance Criteria

### Functional Acceptance

- [ ] Complete Arabic and French translation coverage for patient management
- [ ] RTL layout support with proper text direction and element alignment
- [ ] Dynamic language switching with immediate UI updates
- [ ] User language preference persistence across sessions
- [ ] Proper character encoding and validation for Arabic and French text
- [ ] Localized date, number, and phone formatting

### Technical Acceptance

- [ ] Translation system follows React i18next best practices
- [ ] Unit tests written and passing (>= 80% coverage)
- [ ] Integration tests cover multi-language workflows
- [ ] CSS logical properties used for RTL layout support
- [ ] Accessibility compliance with language attributes

### Quality Acceptance

- [ ] Code review completed and approved
- [ ] Translation quality review by native speakers
- [ ] RTL layout testing across different screen sizes
- [ ] Cross-browser testing for Arabic font rendering
- [ ] User experience testing with Arabic and French interfaces

---
**Last Updated:** 2025-01-16