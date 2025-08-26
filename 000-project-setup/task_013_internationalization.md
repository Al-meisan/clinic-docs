# TASK-013: Internationalization Foundation

## Task Overview

### Goal
**What to Build:** Implement comprehensive internationalization (i18n) system supporting Arabic and French languages with RTL layout support, healthcare-specific translations, and cultural localization for the Algerian healthcare market.

**Why:** Provides multilingual support essential for the Algerian healthcare market where Arabic and French are both official languages, ensuring accessibility for all healthcare providers and staff with proper cultural localization.

**Definition of Done:** Complete i18n system operational with Arabic and French translations, RTL layout support for Arabic, healthcare-specific translation namespaces, and dynamic language switching functionality.

### Scope
```yaml
task_type: "FRONTEND"
complexity: "MEDIUM"
```

## Requirements

### Functional Requirements
1. **Multi-Language Translation System**
   - Description: Complete i18next integration supporting Arabic and French with organized translation namespaces
   - Acceptance Criteria: All UI text translatable, proper language detection, and organized translation files by feature
   - Priority: HIGH

2. **RTL Layout Support**
   - Description: Right-to-left layout support for Arabic language with proper text direction and UI mirroring
   - Acceptance Criteria: Arabic interface properly displays RTL, UI elements mirror correctly, and mixed content handled properly
   - Priority: HIGH

3. **Healthcare-Specific Localization**
   - Description: Medical terminology, date/time formats, number formatting, and healthcare workflow translations
   - Acceptance Criteria: Medical terms properly translated, cultural healthcare practices respected, proper formatting
   - Priority: MEDIUM

4. **Dynamic Language Switching**
   - Description: Runtime language switching with preference persistence and immediate UI updates
   - Acceptance Criteria: Users can switch languages instantly, preferences saved, and all content updates properly
   - Priority: MEDIUM

### Technical Requirements
```yaml
performance:
  - requirement: "Language switching performance"
    target: "< 500ms for complete language switch with UI updates"
    
localization:
  - requirement: "Cultural accuracy for Algerian healthcare context"
    implementation: "Proper medical terminology and cultural healthcare practices"
    
compatibility:
  - requirement: "Cross-browser RTL support"
    scope: "Proper RTL rendering in all supported browsers"
```

## Implementation Details

### Technical Approach
```yaml
i18n_architecture:
  core_libraries:
    - react_i18next: "React integration for i18next"
    - i18next: "Core internationalization framework"
    - i18next_browser_languagedetector: "Browser language detection"
    - i18next_http_backend: "Translation file loading"
    
  translation_structure:
    - namespace_organization: "Feature-based translation namespaces"
    - fallback_system: "French as fallback for missing Arabic translations"
    - lazy_loading: "On-demand translation loading"
    - pluralization: "Proper plural forms for Arabic and French"
    
  rtl_implementation:
    - css_logical_properties: "Direction-independent styling"
    - layout_mirroring: "Automatic UI element mirroring"
    - text_direction: "Proper text direction handling"
    - mixed_content: "LTR content within RTL layouts"

healthcare_localization:
  medical_terminology:
    - anatomical_terms: "Proper Arabic medical terminology"
    - procedure_names: "Localized medical procedure names"
    - medication_categories: "Drug classification translations"
    - symptom_descriptions: "Patient symptom terminology"
    
  cultural_considerations:
    - date_formats: "DD/MM/YYYY for Algeria"
    - time_formats: "24-hour format preference"
    - number_formats: "Arabic-Indic numerals option"
    - address_formats: "Algerian address conventions"
```

### Integration Points
```yaml
dependencies:
  - dependency: "React application with component system"
    type: "FRONTEND"
    status: "AVAILABLE"
    
  - dependency: "Layout system for RTL adaptation"
    type: "FRONTEND"
    status: "IN_PROGRESS"
    
  - dependency: "Authentication system for language preferences"
    type: "FRONTEND"
    status: "IN_PROGRESS"
    
provides:
  - deliverable: "Translation system"
    interface: "React hooks and components for translations"
    consumers: "All frontend components requiring text content"
    
  - deliverable: "RTL layout support"
    interface: "CSS utilities and layout adaptations"
    consumers: "All UI components and layouts"
    
  - deliverable: "Localization utilities"
    interface: "Date, number, and currency formatting functions"
    consumers: "All components displaying formatted data"
```

### Code Patterns to Follow

**i18next Configuration:**
```typescript
// src/lib/i18n/index.ts
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import LanguageDetector from 'i18next-browser-languagedetector';
import Backend from 'i18next-http-backend';

// Translation namespaces
export const defaultNS = 'common';
export const namespaces = [
  'common',
  'auth',
  'navigation',
  'patients',
  'appointments',
  'clinical',
  'prescriptions',
  'billing',
  'errors',
  'validation',
] as const;

export type TranslationNamespace = typeof namespaces[number];

i18n
  .use(Backend)
  .use(LanguageDetector)
  .use(initReactI18next)
  .init({
    lng: 'fr', // Default to French for Algeria
    fallbackLng: 'fr',
    defaultNS,
    ns: namespaces,
    
    interpolation: {
      escapeValue: false, // React already escapes
    },
    
    detection: {
      order: [
        'localStorage',
        'navigator',
        'htmlTag',
        'path',
        'subdomain',
      ],
      lookupLocalStorage: 'medflow-language',
      caches: ['localStorage'],
    },
    
    backend: {
      loadPath: '/locales/{{lng}}/{{ns}}.json',
    },
    
    // Arabic-specific configuration
    lng: 'ar',
    pluralSeparator: '_',
    contextSeparator: '_',
    
    // Support for Arabic pluralization rules
    pluralRules: {
      ar: {
        plurals(n: number): number {
          if (n === 0) return 0; // zero
          if (n === 1) return 1; // one  
          if (n === 2) return 2; // two
          if (n % 100 >= 3 && n % 100 <= 10) return 3; // few
          if (n % 100 >= 11) return 4; // many
          return 5; // other
        },
      },
    },
    
    react: {
      bindI18n: 'languageChanged',
      bindI18nStore: '',
      transEmptyNodeValue: '',
      transSupportBasicHtmlNodes: true,
      transKeepBasicHtmlNodesFor: ['br', 'strong', 'i', 'em'],
    },
  });

export default i18n;
```

**Translation Files Structure:**
```json
// public/locales/ar/common.json
{
  "buttons": {
    "save": "حفظ",
    "cancel": "إلغاء",
    "delete": "حذف",
    "edit": "تعديل",
    "add": "إضافة",
    "search": "بحث",
    "submit": "إرسال",
    "back": "رجوع",
    "next": "التالي",
    "previous": "السابق"
  },
  "status": {
    "active": "نشط",
    "inactive": "غير نشط",
    "pending": "في الانتظار",
    "completed": "مكتمل",
    "cancelled": "ملغى"
  },
  "time": {
    "today": "اليوم",
    "yesterday": "أمس",
    "tomorrow": "غداً",
    "this_week": "هذا الأسبوع",
    "this_month": "هذا الشهر",
    "morning": "صباحاً",
    "afternoon": "بعد الظهر",
    "evening": "مساءً"
  },
  "validation": {
    "required": "هذا الحقل مطلوب",
    "invalid_email": "عنوان البريد الإلكتروني غير صحيح",
    "invalid_phone": "رقم الهاتف غير صحيح",
    "min_length": "يجب أن يحتوي على {{count}} أحرف على الأقل",
    "max_length": "يجب ألا يتجاوز {{count}} حرف"
  }
}

// public/locales/fr/common.json
{
  "buttons": {
    "save": "Enregistrer",
    "cancel": "Annuler",
    "delete": "Supprimer",
    "edit": "Modifier",
    "add": "Ajouter",
    "search": "Rechercher",
    "submit": "Soumettre",
    "back": "Retour",
    "next": "Suivant",
    "previous": "Précédent"
  },
  "status": {
    "active": "Actif",
    "inactive": "Inactif",
    "pending": "En attente",
    "completed": "Terminé",
    "cancelled": "Annulé"
  },
  "time": {
    "today": "Aujourd'hui",
    "yesterday": "Hier",
    "tomorrow": "Demain",
    "this_week": "Cette semaine",
    "this_month": "Ce mois-ci",
    "morning": "Matin",
    "afternoon": "Après-midi",
    "evening": "Soir"
  },
  "validation": {
    "required": "Ce champ est requis",
    "invalid_email": "Adresse email non valide",
    "invalid_phone": "Numéro de téléphone non valide",
    "min_length": "Doit contenir au moins {{count}} caractères",
    "max_length": "Ne doit pas dépasser {{count}} caractères"
  }
}
```

**Healthcare-Specific Translations:**
```json
// public/locales/ar/patients.json
{
  "title": "المرضى",
  "patient": "مريض",
  "patient_plural": "مرضى",
  "new_patient": "مريض جديد",
  "patient_profile": "ملف المريض",
  "personal_info": "المعلومات الشخصية",
  "contact_info": "معلومات الاتصال",
  "medical_history": "التاريخ الطبي",
  "fields": {
    "first_name": "الاسم الأول",
    "last_name": "اسم العائلة",
    "date_of_birth": "تاريخ الميلاد",
    "gender": "الجنس",
    "phone": "رقم الهاتف",
    "email": "البريد الإلكتروني",
    "address": "العنوان",
    "emergency_contact": "جهة الاتصال في حالات الطوارئ"
  },
  "gender": {
    "male": "ذكر",
    "female": "أنثى",
    "other": "آخر"
  },
  "status": {
    "active": "نشط",
    "inactive": "غير نشط",
    "deceased": "متوفى",
    "merged": "مدمج"
  }
}

// public/locales/ar/clinical.json
{
  "title": "السجلات السريرية",
  "chief_complaint": "الشكوى الرئيسية",
  "symptoms": "الأعراض",
  "diagnosis": "التشخيص",
  "treatment_plan": "خطة العلاج",
  "vital_signs": "العلامات الحيوية",
  "blood_pressure": "ضغط الدم",
  "heart_rate": "معدل نبضات القلب",
  "temperature": "درجة الحرارة",
  "weight": "الوزن",
  "height": "الطول",
  "allergies": "الحساسية",
  "medications": "الأدوية",
  "medical_conditions": {
    "diabetes": "داء السكري",
    "hypertension": "ارتفاع ضغط الدم",
    "asthma": "الربو",
    "heart_disease": "أمراض القلب",
    "arthritis": "التهاب المفاصل"
  }
}
```

**RTL CSS Support:**
```css
/* src/styles/rtl.css */

/* Base RTL support */
[dir="rtl"] {
  text-align: right;
  font-family: 'Cairo', 'Arial', sans-serif;
}

[dir="ltr"] {
  text-align: left;
}

/* Layout adaptations */
[dir="rtl"] .sidebar {
  left: auto;
  right: 0;
  border-left: 1px solid #e5e7eb;
  border-right: none;
}

[dir="rtl"] .main-content {
  margin-right: 16rem;
  margin-left: 0;
}

/* Navigation adaptations */
[dir="rtl"] .nav-item {
  padding-right: 1rem;
  padding-left: 0.5rem;
}

[dir="rtl"] .nav-item .icon {
  margin-left: 0.75rem;
  margin-right: 0;
}

[dir="rtl"] .breadcrumb {
  flex-direction: row-reverse;
}

[dir="rtl"] .breadcrumb-separator {
  transform: scaleX(-1);
}

/* Form adaptations */
[dir="rtl"] .form-label {
  text-align: right;
}

[dir="rtl"] .form-input {
  text-align: right;
  padding-right: 0.75rem;
  padding-left: 0.75rem;
}

[dir="rtl"] .form-input[type="tel"],
[dir="rtl"] .form-input[type="number"] {
  direction: ltr;
  text-align: left;
}

/* Table adaptations */
[dir="rtl"] .table {
  direction: rtl;
}

[dir="rtl"] .table th,
[dir="rtl"] .table td {
  text-align: right;
}

[dir="rtl"] .table th:first-child,
[dir="rtl"] .table td:first-child {
  border-left: none;
  border-right: 1px solid #e5e7eb;
}

/* Button adaptations */
[dir="rtl"] .btn .icon {
  margin-left: 0.5rem;
  margin-right: 0;
}

[dir="rtl"] .btn .icon-right {
  margin-left: 0;
  margin-right: 0.5rem;
}

/* Modal adaptations */
[dir="rtl"] .modal {
  text-align: right;
}

[dir="rtl"] .modal-header .close {
  left: 1rem;
  right: auto;
}

/* Responsive RTL */
@media (max-width: 768px) {
  [dir="rtl"] .mobile-menu {
    right: 0;
    left: auto;
    transform: translateX(100%);
  }
  
  [dir="rtl"] .mobile-menu.open {
    transform: translateX(0);
  }
}
```

**Language Switching Component:**
```typescript
// src/components/common/LanguageSwitcher.tsx
import React from 'react';
import { useTranslation } from 'react-i18next';
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu';
import { Button } from '@/components/ui/button';
import { Languages, Check } from 'lucide-react';
import { cn } from '@/lib/utils';

const languages = [
  {
    code: 'ar',
    name: 'العربية',
    flag: '🇩🇿',
    dir: 'rtl',
  },
  {
    code: 'fr',
    name: 'Français',
    flag: '🇫🇷',
    dir: 'ltr',
  },
] as const;

export const LanguageSwitcher: React.FC = () => {
  const { i18n } = useTranslation();

  const handleLanguageChange = async (langCode: string) => {
    const selectedLang = languages.find(lang => lang.code === langCode);
    if (!selectedLang) return;

    try {
      // Change language
      await i18n.changeLanguage(langCode);
      
      // Update document direction
      document.documentElement.dir = selectedLang.dir;
      document.documentElement.lang = langCode;
      
      // Update HTML class for styling
      document.documentElement.classList.toggle('rtl', selectedLang.dir === 'rtl');
      
      // Store preference
      localStorage.setItem('medflow-language', langCode);
      
      // Trigger layout recalculation for RTL
      window.dispatchEvent(new Event('languageChanged'));
      
    } catch (error) {
      console.error('Failed to change language:', error);
    }
  };

  const currentLanguage = languages.find(lang => lang.code === i18n.language) || languages[1];

  return (
    <DropdownMenu>
      <DropdownMenuTrigger asChild>
        <Button variant="ghost" size="sm" className="gap-2">
          <Languages className="h-4 w-4" />
          <span className="hidden sm:inline">
            {currentLanguage.flag} {currentLanguage.name}
          </span>
        </Button>
      </DropdownMenuTrigger>
      <DropdownMenuContent align="end" className="w-48">
        {languages.map((language) => (
          <DropdownMenuItem
            key={language.code}
            onClick={() => handleLanguageChange(language.code)}
            className="flex items-center justify-between cursor-pointer"
          >
            <div className="flex items-center gap-2">
              <span>{language.flag}</span>
              <span>{language.name}</span>
            </div>
            {i18n.language === language.code && (
              <Check className="h-4 w-4 text-primary" />
            )}
          </DropdownMenuItem>
        ))}
      </DropdownMenuContent>
    </DropdownMenu>
  );
};
```

**Translation Hooks:**
```typescript
// src/hooks/useTranslation.ts
import { useTranslation as useI18nTranslation } from 'react-i18next';
import { TranslationNamespace } from '@/lib/i18n';

// Enhanced useTranslation hook with TypeScript support
export const useTranslation = (namespace?: TranslationNamespace) => {
  const translation = useI18nTranslation(namespace);
  
  return {
    ...translation,
    // Add custom translation utilities
    formatDate: (date: Date | string) => {
      const dateObj = new Date(date);
      const locale = translation.i18n.language === 'ar' ? 'ar-DZ' : 'fr-DZ';
      
      return dateObj.toLocaleDateString(locale, {
        year: 'numeric',
        month: 'long',
        day: 'numeric',
      });
    },
    
    formatTime: (date: Date | string) => {
      const dateObj = new Date(date);
      const locale = translation.i18n.language === 'ar' ? 'ar-DZ' : 'fr-DZ';
      
      return dateObj.toLocaleTimeString(locale, {
        hour: '2-digit',
        minute: '2-digit',
      });
    },
    
    formatNumber: (number: number) => {
      const locale = translation.i18n.language === 'ar' ? 'ar-DZ' : 'fr-DZ';
      return number.toLocaleString(locale);
    },
    
    formatCurrency: (amount: number) => {
      const locale = translation.i18n.language === 'ar' ? 'ar-DZ' : 'fr-DZ';
      return new Intl.NumberFormat(locale, {
        style: 'currency',
        currency: 'DZD',
      }).format(amount);
    },
    
    isRTL: translation.i18n.language === 'ar',
    isArabic: translation.i18n.language === 'ar',
    isFrench: translation.i18n.language === 'fr',
  };
};

// Hook for accessing current language info
export const useLanguage = () => {
  const { i18n } = useI18nTranslation();
  
  const currentLanguage = {
    code: i18n.language,
    isRTL: i18n.language === 'ar',
    isArabic: i18n.language === 'ar',
    isFrench: i18n.language === 'fr',
    direction: i18n.language === 'ar' ? 'rtl' : 'ltr',
  };
  
  return {
    ...currentLanguage,
    changeLanguage: i18n.changeLanguage,
  };
};
```

**Localization Utilities:**
```typescript
// src/lib/localization.ts
import { format, parseISO } from 'date-fns';
import { ar, fr } from 'date-fns/locale';

export class LocalizationUtils {
  static getLocale(language: string) {
    return language === 'ar' ? ar : fr;
  }
  
  static formatDate(date: Date | string, language: string = 'fr'): string {
    const dateObj = typeof date === 'string' ? parseISO(date) : date;
    const locale = this.getLocale(language);
    
    return format(dateObj, 'dd MMMM yyyy', { locale });
  }
  
  static formatTime(date: Date | string, language: string = 'fr'): string {
    const dateObj = typeof date === 'string' ? parseISO(date) : date;
    const locale = this.getLocale(language);
    
    return format(dateObj, 'HH:mm', { locale });
  }
  
  static formatDateTime(date: Date | string, language: string = 'fr'): string {
    const dateObj = typeof date === 'string' ? parseISO(date) : date;
    const locale = this.getLocale(language);
    
    return format(dateObj, 'dd MMMM yyyy HH:mm', { locale });
  }
  
  static formatPhoneNumber(phone: string, language: string = 'fr'): string {
    // Algerian phone number formatting
    const cleaned = phone.replace(/\D/g, '');
    
    if (cleaned.startsWith('213')) {
      // International format
      const number = cleaned.slice(3);
      return `+213 ${number.slice(0, 1)} ${number.slice(1, 3)} ${number.slice(3, 5)} ${number.slice(5)}`;
    } else if (cleaned.startsWith('0')) {
      // National format
      const number = cleaned.slice(1);
      return `0${number.slice(0, 1)} ${number.slice(1, 3)} ${number.slice(3, 5)} ${number.slice(5)}`;
    }
    
    return phone;
  }
  
  static getPatientName(patient: { firstName: string; lastName: string }, language: string = 'fr'): string {
    if (language === 'ar') {
      // In Arabic, family name often comes first
      return `${patient.lastName} ${patient.firstName}`;
    }
    return `${patient.firstName} ${patient.lastName}`;
  }
  
  static getAddressFormat(address: any, language: string = 'fr'): string {
    if (language === 'ar') {
      return `${address.street}، ${address.city}، ${address.state} ${address.zipCode}`;
    }
    return `${address.street}, ${address.city}, ${address.state} ${address.zipCode}`;
  }
}
```

## Testing Requirements

### Test Coverage
```yaml
unit_tests:
  - focus: "Translation hooks and utilities"
  - coverage_target: "90%"
  - key_scenarios: ["language_switching", "translation_fallbacks", "rtl_utilities"]
  
integration_tests:
  - focus: "Complete i18n integration with components"
  - test_environment: "React Testing Library with i18n provider"
  - key_flows: ["language_change_flow", "rtl_layout_adaptation", "translation_loading"]
  
accessibility_tests:
  - focus: "RTL accessibility and screen reader compatibility"
  - test_environment: "Automated accessibility testing tools"
  - key_scenarios: ["rtl_navigation", "arabic_content_accessibility", "language_switching_accessibility"]
```

### Test Scenarios
```yaml
happy_path:
  - scenario: "Language switching with UI adaptation"
    steps: ["Switch to Arabic", "Verify RTL layout", "Switch to French", "Verify LTR layout"]
    expected_result: "Complete UI adaptation including text direction and layout mirroring"
    
error_cases:
  - scenario: "Missing translation keys"
    trigger: "Translation key not found in current language"
    expected_behavior: "Fallback to French translation or display key with warning"
    
edge_cases:
  - scenario: "Mixed content in RTL layout"
    conditions: "English text/numbers within Arabic interface"
    expected_behavior: "Proper text direction handling with bidirectional text support"
```

## Context References

### From Project
```yaml
tech_stack_references:
  - "React i18next for internationalization"
  - "CSS logical properties for RTL support"
  - "Healthcare-specific translation organization"
  - "Algerian cultural and language considerations"
  
architecture_patterns:
  - "Namespace-based translation organization"
  - "Lazy loading of translation files"
  - "Hook-based translation access"
  - "Utility-based localization functions"
  
code_standards:
  - "Consistent translation key naming patterns"
  - "Proper pluralization handling"
  - "Cultural sensitivity in translations"
  - "Performance-optimized translation loading"
```

### From Epic
```yaml
business_rules:
  - "Arabic and French language support for Algerian market"
  - "RTL layout support for Arabic interface"
  - "Healthcare terminology properly localized"
  - "Cultural healthcare practices respected in translations"
  
integration_points:
  - "Required by all user-facing components"
  - "Integrates with authentication for language preferences"
  - "Foundation for culturally appropriate healthcare interface"
```

## Acceptance Criteria

### Functional Acceptance
- [ ] Arabic and French translations implemented for all UI text
- [ ] RTL layout working properly for Arabic interface
- [ ] Language switching functional with immediate UI updates
- [ ] Healthcare-specific terminology properly translated
- [ ] Date, time, and number formatting culturally appropriate

### Technical Acceptance  
- [ ] Language switching meets performance requirements
- [ ] Translation files organized by feature namespaces
- [ ] RTL CSS properly implemented across all components
- [ ] Browser language detection and preference persistence working
- [ ] Fallback system prevents missing translations

### Quality Acceptance
- [ ] All translations reviewed by native speakers
- [ ] RTL layout tested across all supported browsers
- [ ] Accessibility testing passes for both languages
- [ ] Cultural sensitivity review completed
- [ ] Performance testing shows acceptable translation loading times

---

**Last Updated:** 2025-01-24