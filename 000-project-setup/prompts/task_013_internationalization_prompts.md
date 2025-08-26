# PROMPTS: TASK-013 - Internationalization Foundation

# PROMPT 1: Core i18n Infrastructure Setup

Implement the core internationalization infrastructure using react-i18next with proper configuration for Arabic and French language support, including browser language detection and preference persistence.

## PROJECT CONTEXT
**Technology Stack:**
- Frontend: React with TypeScript
- UI Components: shadcn/ui with Tailwind CSS
- State Management: Zustand stores
- Build Tool: Vite for development and production builds

**Current Application State:**
- React application with TypeScript established (TASK-007)
- Routing system with protected routes implemented (TASK-009)
- Layout system supporting responsive design (TASK-012)
- State management with Zustand configured (TASK-011)

**Folder Structure:**
```
src/
├── lib/              # Core utilities and services
├── providers/        # React context providers
├── config/          # Application configuration
└── hooks/           # Shared custom hooks
```

## EPIC CONTEXT
**Epic:** Project Setup Foundation
**Focus:** Establishing multilingual support for Algerian healthcare market where Arabic and French are both official languages, ensuring accessibility for all healthcare providers and staff.

## TASK CONTEXT
**Primary Goal:** Implement comprehensive i18n system supporting Arabic and French languages with RTL layout support and healthcare-specific translations.

**Core Requirements:**
- Multi-language translation system with Arabic and French
- Browser language detection and preference persistence
- Organized translation namespaces by feature
- Foundation for RTL layout support

## BUSINESS DOMAIN CONTEXT
**Healthcare Context:**
- Algerian healthcare market with Arabic and French as official languages
- Medical terminology requires proper localization
- Cultural sensitivity in healthcare communication
- Professional healthcare interface standards

**Language Requirements:**
- Arabic (ar): Primary language with RTL support
- French (fr): Secondary language, fallback for missing translations
- Browser detection: Automatic language selection based on user preferences
- Preference persistence: Language choice saved across sessions

## FUNCTIONAL REQUIREMENTS
1. **i18next Core Configuration**
   - Configure react-i18next with Arabic (ar) and French (fr) support
   - Implement browser language detection
   - Set up translation file loading from public/locales directory
   - Configure French as fallback language

2. **Translation Infrastructure**
   - Create translation hook for components (useTranslation)
   - Implement language context provider
   - Set up namespace-based translation organization
   - Create utility functions for language detection

3. **Storage and Persistence**
   - Store language preference in localStorage
   - Sync language preference with user profile (when authentication available)
   - Maintain language state across application restarts

## TECHNICAL REQUIREMENTS
```yaml
libraries:
  - react-i18next: "^13.5.0"
  - i18next: "^23.7.0"
  - i18next-browser-languagedetector: "^7.2.0"
  - i18next-http-backend: "^2.4.0"

performance:
  - initialization_time: "< 200ms for i18n setup"
  - translation_loading: "< 100ms for namespace loading"

file_structure:
  - config: "src/lib/i18n.ts"
  - provider: "src/providers/I18nProvider.tsx"
  - hooks: "src/hooks/useTranslation.ts"
  - translations: "public/locales/{lang}/{namespace}.json"
```

## TESTING REQUIREMENTS
**Unit Tests:**
- Test i18n configuration initialization
- Test language detection logic
- Test translation hook functionality
- Test language preference persistence

**Integration Tests:**
- Test i18n provider wrapping application
- Test translation loading from files
- Test fallback mechanism (Arabic → French)

**Manual Tests:**
- Verify browser language detection works
- Test language persistence across browser sessions
- Confirm translation files load correctly

## DOCUMENTATION REQUIREMENTS
**Code Documentation:**
- JSDoc comments for all i18n utilities and hooks
- TypeScript interfaces for translation configuration
- Usage examples for components

**Setup Documentation:**
- i18n configuration explanation
- Translation file organization guide
- Adding new languages process

## VALIDATION CRITERIA
- [ ] react-i18next successfully configured with Arabic and French support
- [ ] Browser language detection working correctly
- [ ] Language preference persisted in localStorage
- [ ] Translation files loading from public/locales directory structure
- [ ] useTranslation hook provides typed translation functions
- [ ] I18nProvider wraps application and provides language context
- [ ] French fallback working when Arabic translations missing
- [ ] All configuration properly typed with TypeScript
- [ ] Unit tests passing for core i18n functionality
- [ ] Integration tests confirming provider and hook integration

# PROMPT 2: Translation File Organization and Base Translations

Create the translation file structure and implement base translations for common UI elements, focusing on healthcare-appropriate terminology and proper Arabic translations with French fallbacks.

## PROJECT CONTEXT
**Current State After Prompt 1:**
- Core i18n infrastructure configured with react-i18next
- Language detection and persistence implemented
- Translation loading system established
- useTranslation hook available for components

**Required Integration:**
- Translation files must follow namespace organization
- Arabic translations require proper RTL considerations
- Healthcare terminology needs cultural sensitivity
- Common UI elements need consistent translation patterns

## EPIC CONTEXT
**Healthcare Localization Focus:**
- Professional medical terminology in Arabic and French
- Cultural appropriateness for Algerian healthcare context
- Consistent terminology across medical workflows
- Professional tone suitable for healthcare providers

## TASK CONTEXT
**Translation Organization Strategy:**
- Feature-based namespaces (auth, patients, appointments, clinical)
- Common namespace for shared UI elements
- Hierarchical key structure for related terms
- Consistent naming patterns across languages

**Key Translation Areas:**
- Authentication and access control
- Patient management terminology
- Appointment scheduling language
- Basic navigation and actions
- Form validation messages

## BUSINESS DOMAIN CONTEXT
**Healthcare Terminology Standards:**
- Medical terms: Proper Arabic medical vocabulary
- Professional titles: Healthcare provider designations
- Clinical workflow: Medical procedure terminology
- Patient interaction: Respectful, clear communication

**Cultural Considerations:**
- Arabic: Right-to-left reading direction
- Formal vs. informal address in healthcare contexts
- Gender-appropriate language in medical settings
- Algerian French vs. European French preferences

## FUNCTIONAL REQUIREMENTS
1. **Translation File Structure**
   - Create namespace-based translation files
   - Implement hierarchical key organization
   - Establish consistent key naming patterns
   - Set up proper file directory structure

2. **Core Translation Namespaces**
   - `common`: Shared UI elements, buttons, actions
   - `auth`: Authentication and login related terms
   - `patients`: Patient management terminology
   - `appointments`: Scheduling and appointment terms
   - `navigation`: Menu items and navigation elements

3. **Translation Content Quality**
   - Professional healthcare terminology
   - Culturally appropriate Arabic translations
   - Accurate French medical terms
   - Consistent tone across all translations

## TECHNICAL REQUIREMENTS
```yaml
file_structure:
  - "public/locales/ar/common.json"
  - "public/locales/ar/auth.json"
  - "public/locales/ar/patients.json"
  - "public/locales/ar/appointments.json"
  - "public/locales/ar/navigation.json"
  - "public/locales/fr/common.json"
  - "public/locales/fr/auth.json"
  - "public/locales/fr/patients.json"
  - "public/locales/fr/appointments.json"
  - "public/locales/fr/navigation.json"

key_structure:
  - "flat_structure": "feature.component.element.state"
  - "example": "patients.form.firstName.label"
  - "pluralization": Support i18next plural forms

validation:
  - "key_consistency": All keys exist in both languages
  - "structure_matching": JSON structure identical across languages
  - "placeholder_consistency": Variable placeholders match
```

## TESTING REQUIREMENTS
**Unit Tests:**
- Test translation key existence in both languages
- Test translation placeholder variable matching
- Test translation file JSON validity

**Integration Tests:**
- Test translation loading for all namespaces
- Test fallback from Arabic to French
- Test pluralization handling

**Manual Tests:**
- Review Arabic translations for accuracy
- Verify French translations for medical context
- Check translation consistency across features

## DOCUMENTATION REQUIREMENTS
**Translation Guidelines:**
- Key naming conventions
- Arabic translation standards
- French healthcare terminology guide
- Adding new translations process

## VALIDATION CRITERIA
- [ ] Translation files created for both Arabic and French languages
- [ ] All five core namespaces implemented (common, auth, patients, appointments, navigation)
- [ ] Consistent key structure across all translation files
- [ ] Healthcare-appropriate Arabic translations with proper medical terminology
- [ ] Professional French translations suitable for healthcare context
- [ ] JSON structure identical across language files
- [ ] Placeholder variables consistent between Arabic and French
- [ ] Pluralization forms properly implemented where needed
- [ ] Translation key naming follows documented conventions
- [ ] All translation files valid JSON format

# PROMPT 3: RTL Layout Infrastructure Implementation

Implement comprehensive RTL (Right-to-Left) layout support for Arabic language including CSS logical properties, direction-aware styling, and component adaptations for proper Arabic text display.

## PROJECT CONTEXT
**Current State After Prompt 2:**
- i18n infrastructure with Arabic and French support
- Translation files organized by namespaces
- Language detection and switching capability
- Base translations available for core features

**Integration Requirements:**
- RTL styling must work with existing Tailwind CSS classes
- Component layouts need direction-aware adaptations
- Text direction changes should not break existing designs
- RTL support should be toggleable based on language selection

## EPIC CONTEXT
**RTL Design Requirements:**
- Arabic text flows right-to-left naturally
- UI elements should mirror appropriately for RTL contexts
- Icons and directional indicators need RTL variations
- Layout spacing and alignment must adapt to text direction

## TASK CONTEXT
**RTL Implementation Scope:**
- CSS logical properties for direction-independent styling
- Document direction attribute management
- Tailwind CSS RTL plugin configuration
- Component-level RTL adaptations

**Key Areas Requiring RTL Support:**
- Navigation menus and sidebar layouts
- Form layouts with labels and inputs
- Table layouts with column alignment
- Card layouts with content flow
- Modal and dialog positioning

## BUSINESS DOMAIN CONTEXT
**Healthcare Interface Requirements:**
- Medical forms need proper RTL label alignment
- Patient information displays require RTL text flow
- Appointment schedules need RTL time formatting
- Clinical documentation needs RTL text entry support

**Arabic Language Considerations:**
- Arabic numerals vs. Hindi-Arabic numerals
- Mixed content handling (Arabic text with Latin numbers)
- Bidirectional text algorithm support
- Proper Arabic font rendering

## FUNCTIONAL REQUIREMENTS
1. **CSS RTL Foundation**
   - Implement CSS logical properties for direction-independent styling
   - Configure Tailwind CSS RTL plugin
   - Create RTL-specific utility classes
   - Set up document direction management

2. **Component RTL Adaptations**
   - Update layout components for RTL support
   - Modify form components for proper label/input alignment
   - Adapt navigation components for RTL flow
   - Update modal and dialog positioning

3. **Dynamic Direction Switching**
   - Implement language-based direction switching
   - Create direction-aware styling hooks
   - Set up proper component re-rendering for direction changes

## TECHNICAL REQUIREMENTS
```yaml
css_approach:
  - logical_properties: "margin-inline-start vs margin-left"
  - tailwind_rtl: "@tailwindcss/rtl plugin configuration"
  - direction_classes: ".rtl and .ltr utility classes"
  - bidirectional: "CSS dir() pseudo-class usage"

component_adaptations:
  - layout_components: "DashboardLayout, AuthLayout"
  - form_components: "Input, Select, TextArea labels"
  - navigation: "Sidebar, TopNav, Breadcrumbs"
  - content: "Cards, Tables, Lists"

performance:
  - rtl_switching: "< 200ms for complete RTL/LTR switch"
  - layout_stability: "No layout shifts during direction change"
  - font_loading: "Proper Arabic font loading and fallbacks"
```

## TESTING REQUIREMENTS
**Unit Tests:**
- Test direction utility functions
- Test RTL-specific styling utilities
- Test component direction prop handling

**Integration Tests:**
- Test complete RTL/LTR switching workflow
- Test component re-rendering on direction change
- Test Arabic font loading and display

**Visual Tests:**
- RTL layout visual regression testing
- Component positioning in RTL mode
- Text alignment and spacing verification

**Manual Tests:**
- Arabic interface review with native speaker
- RTL navigation flow testing
- Mixed content display verification

## DOCUMENTATION REQUIREMENTS
**RTL Development Guide:**
- CSS logical properties usage guidelines
- Component RTL adaptation patterns
- Common RTL layout pitfalls and solutions
- Arabic font and typography guidelines

## VALIDATION CRITERIA
- [ ] Tailwind CSS RTL plugin configured and working
- [ ] CSS logical properties implemented across component styles
- [ ] Document direction automatically switches with language
- [ ] Layout components properly mirror in RTL mode
- [ ] Form components show correct label/input alignment in Arabic
- [ ] Navigation components flow correctly right-to-left
- [ ] Arabic text displays with proper font rendering
- [ ] Mixed content (Arabic + numbers) handles bidirectional text correctly
- [ ] RTL/LTR switching happens smoothly without layout shifts
- [ ] All interactive elements accessible in RTL mode
- [ ] Direction-aware utility classes work correctly
- [ ] Component positioning and spacing appropriate for RTL

# PROMPT 4: Healthcare Localization and Cultural Formatting

Implement healthcare-specific localization utilities including medical terminology translation, culturally appropriate date/time formatting, Algerian phone number formatting, and healthcare workflow-specific localization features.

## PROJECT CONTEXT
**Current State After Prompt 3:**
- Complete i18n infrastructure with Arabic/French support
- Translation files organized with healthcare terminology
- RTL layout support fully implemented
- Dynamic language switching operational

**Integration Requirements:**
- Localization utilities must integrate with existing translation system
- Date/time formatting should work with appointment scheduling
- Phone number formatting needed for patient contact information
- Medical terminology must support clinical documentation features

## EPIC CONTEXT
**Algerian Healthcare Context:**
- Date formats: DD/MM/YYYY standard in Algeria
- Time formats: 24-hour format preferred in healthcare
- Phone numbers: Algerian national and international formats
- Currency: Algerian Dinar (DZD) for billing contexts
- Address formats: Algerian postal address conventions

## TASK CONTEXT
**Localization Utilities Scope:**
- Date and time formatting utilities
- Phone number and address formatting
- Currency and number formatting
- Medical terminology localization helpers
- Cultural name formatting conventions

**Healthcare-Specific Requirements:**
- Medical appointment time formatting
- Patient demographic information display
- Clinical measurement units (metric system)
- Prescription and medication formatting
- Healthcare provider name formatting

## BUSINESS DOMAIN CONTEXT
**Medical Terminology Standards:**
- Arabic medical terms: Proper anatomical and clinical vocabulary
- French medical terms: Healthcare terminology used in Algeria
- Measurement units: Metric system with proper Arabic numerals
- Drug classifications: International and local medication naming

**Cultural Formatting Rules:**
- Arabic names: Family name prominence in medical contexts
- French names: Professional title formatting
- Phone numbers: Mobile vs. landline format distinctions
- Addresses: Wilaya (province) and commune formatting

## FUNCTIONAL REQUIREMENTS
1. **Date and Time Localization**
   - Create date formatting utilities for Arabic and French locales
   - Implement time formatting with 24-hour healthcare standard
   - Support appointment scheduling time formats
   - Handle timezone considerations for Algeria

2. **Contact Information Formatting**
   - Algerian phone number formatting (national/international)
   - Address formatting following Algerian postal standards
   - Emergency contact information display
   - Healthcare provider contact formatting

3. **Medical Localization Utilities**
   - Patient name formatting (cultural name order preferences)
   - Medical measurement formatting (height, weight, vitals)
   - Medication and dosage formatting
   - Clinical documentation text formatting

4. **Number and Currency Formatting**
   - Algerian Dinar (DZD) currency formatting
   - Arabic-Indic numeral option for Arabic interface
   - Medical value formatting (blood pressure, temperature)
   - Billing amount formatting

## TECHNICAL REQUIREMENTS
```yaml
utilities_structure:
  - "src/lib/localization/index.ts"
  - "src/lib/localization/dateTime.ts"
  - "src/lib/localization/formatting.ts"
  - "src/lib/localization/healthcare.ts"
  - "src/lib/localization/constants.ts"

localization_libraries:
  - date_fns: "Date formatting with Arabic and French locales"
  - libphonenumber_js: "Phone number validation and formatting"
  - intl_api: "Browser Internationalization API usage"

cultural_constants:
  - algeria_locale: "ar-DZ and fr-DZ locale codes"
  - timezone: "Africa/Algiers timezone"
  - currency: "DZD Algerian Dinar"
  - phone_country_code: "+213"

performance:
  - formatting_speed: "< 5ms for any single formatting operation"
  - locale_loading: "Lazy load locale data when needed"
```

## TESTING REQUIREMENTS
**Unit Tests:**
- Test date formatting in both Arabic and French
- Test phone number formatting with various input formats
- Test currency formatting with different amounts
- Test name formatting with Arabic and French names
- Test medical value formatting (vitals, measurements)

**Integration Tests:**
- Test localization utilities with i18n language switching
- Test formatting consistency across application features
- Test edge cases with malformed input data

**Manual Tests:**
- Review formatted output with Arabic native speaker
- Verify healthcare terminology accuracy
- Test cultural appropriateness of formatting choices

## DOCUMENTATION REQUIREMENTS
**Localization Guide:**
- Healthcare formatting standards
- Cultural considerations for Algerian market
- Usage examples for all formatting utilities
- Common localization patterns and best practices

## VALIDATION CRITERIA
- [ ] Date formatting utilities working for Arabic and French locales
- [ ] Time formatting using 24-hour healthcare standard
- [ ] Algerian phone number formatting (national and international)
- [ ] Address formatting following Algerian postal conventions
- [ ] Currency formatting for Algerian Dinar (DZD)
- [ ] Patient name formatting respecting cultural preferences
- [ ] Medical measurement formatting (height, weight, vitals)
- [ ] Arabic-Indic numeral formatting option available
- [ ] Medication and dosage formatting utilities
- [ ] All formatting utilities properly typed with TypeScript
- [ ] Unit tests covering all formatting scenarios
- [ ] Performance requirements met (< 5ms per operation)
- [ ] Integration with existing translation system verified

# PROMPT 5: Dynamic Language Switching and User Preferences

Implement dynamic language switching functionality with user preference persistence, immediate UI updates, and integration with authentication system for storing language preferences in user profiles.

## PROJECT CONTEXT
**Current State After Prompt 4:**
- Complete i18n infrastructure with Arabic/French support
- RTL layout support with automatic direction switching
- Healthcare localization utilities implemented
- Translation system with namespace organization

**Integration Points:**
- Authentication system integration for user preferences
- State management with Zustand stores
- Layout system responsive to language changes
- Local storage for guest user preferences

## EPIC CONTEXT
**User Experience Requirements:**
- Immediate language switching without page reload
- Persistent language preferences across sessions
- Seamless RTL/LTR transition
- User profile integration for authenticated users
- Guest user preference storage

## TASK CONTEXT
**Language Switching Scope:**
- Runtime language switching component
- Language preference persistence logic
- User profile integration preparation
- Performance optimization for smooth transitions
- Language indicator and selection UI

**Key Features:**
- Language selector component (dropdown or toggle)
- Immediate UI translation updates
- Direction switching with layout adaptation
- Preference synchronization (local storage + user profile)
- Loading states during language switching

## BUSINESS DOMAIN CONTEXT
**Healthcare User Experience:**
- Medical staff need quick language switching during patient interactions
- Language preferences should persist between work shifts
- Emergency situations require immediate language accessibility
- Multi-language healthcare teams need flexible language options

**User Types and Language Needs:**
- Arabic-speaking healthcare providers: Arabic as primary language
- French-speaking healthcare staff: French as primary language
- Bilingual users: Quick switching capability needed
- Administrative staff: Language based on patient communication needs

## FUNCTIONAL REQUIREMENTS
1. **Language Selector Component**
   - Create accessible language selector dropdown/toggle
   - Display current language with clear visual indicators
   - Provide smooth transition animations
   - Support keyboard navigation

2. **Dynamic Switching Logic**
   - Implement immediate language switching without reload
   - Update all UI elements instantaneously
   - Handle RTL/LTR direction switching
   - Manage loading states during namespace loading

3. **Preference Persistence System**
   - Store language preference in localStorage for guests
   - Prepare user profile integration for authenticated users
   - Sync preferences across browser sessions
   - Handle preference conflicts (user profile vs. browser detection)

4. **Performance Optimization**
   - Preload critical translation namespaces
   - Optimize translation cache management
   - Minimize layout shifts during switching
   - Implement smooth transition animations

## TECHNICAL REQUIREMENTS
```yaml
components:
  - "src/components/ui/LanguageSelector.tsx"
  - "src/hooks/useLanguagePreferences.ts"
  - "src/lib/languagePreferences.ts"
  - "src/providers/LanguageProvider.tsx"

state_management:
  - language_store: "Zustand store for language state"
  - preference_sync: "localStorage and user profile sync"
  - transition_state: "Loading states during switching"

performance_targets:
  - switching_time: "< 300ms for complete language switch"
  - layout_stability: "No layout shifts during transition"
  - translation_cache: "Efficient namespace loading and caching"

user_experience:
  - accessibility: "ARIA labels and keyboard navigation"
  - visual_feedback: "Clear current language indication"
  - transition_animations: "Smooth language switching animations"
```

## TESTING REQUIREMENTS
**Unit Tests:**
- Test language selector component interaction
- Test preference persistence logic
- Test language switching state management
- Test preference sync between storage and user profile

**Integration Tests:**
- Test end-to-end language switching workflow
- Test authentication integration for preference storage
- Test preference conflict resolution
- Test performance during rapid language switching

**User Experience Tests:**
- Test accessibility of language selector
- Test keyboard navigation for language switching
- Test visual feedback during language transitions
- Test language switching with screen readers

**Performance Tests:**
- Measure language switching completion time
- Test translation cache efficiency
- Verify no layout shifts during transitions
- Test memory usage during repeated switching

## DOCUMENTATION REQUIREMENTS
**Language Switching Guide:**
- Component usage documentation
- Preference management system explanation
- Integration with authentication system
- Performance considerations and best practices

**User Guide:**
- How to change language preferences
- Language persistence behavior explanation
- Troubleshooting language switching issues

## VALIDATION CRITERIA
- [ ] Language selector component implemented with accessible design
- [ ] Immediate language switching working without page reload
- [ ] RTL/LTR direction switching happens automatically with language
- [ ] Language preference persisted in localStorage for guest users
- [ ] User profile integration prepared for authenticated language storage
- [ ] All UI elements update immediately during language switching
- [ ] Loading states shown during namespace loading if needed
- [ ] Language switching performance meets < 300ms target
- [ ] No layout shifts occur during language transitions
- [ ] Keyboard navigation works properly for language selector
- [ ] Screen reader accessibility verified for language switching
- [ ] Preference conflicts resolved correctly (profile vs. browser detection)
- [ ] Translation cache management optimized for performance
- [ ] Visual feedback clearly indicates current language selection

# PROMPT 6: Testing and Quality Assurance for i18n Implementation

Implement comprehensive testing strategy for the internationalization system including unit tests, integration tests, accessibility tests, and performance validation to ensure robust multilingual support.

## PROJECT CONTEXT
**Current State After Prompt 5:**
- Complete i18n infrastructure with Arabic/French support
- RTL layout implementation with direction switching
- Healthcare localization utilities
- Dynamic language switching with user preferences
- Translation files organized by namespaces

**Testing Requirements:**
- Verify all i18n components work correctly
- Ensure translation quality and completeness
- Validate RTL layout functionality
- Test performance of language switching
- Confirm accessibility compliance

## EPIC CONTEXT
**Quality Assurance Focus:**
- Healthcare application reliability requirements
- Multilingual interface must be error-free
- Cultural sensitivity in translations verified
- Performance standards for medical workflow efficiency
- Accessibility compliance for healthcare providers

## TASK CONTEXT
**Testing Scope:**
- i18n configuration and setup testing
- Translation completeness and accuracy validation
- RTL layout and direction switching tests
- Language switching functionality verification
- Performance and accessibility testing

**Test Categories:**
- Unit tests for individual i18n utilities
- Integration tests for complete workflows
- Visual tests for RTL layout verification
- Performance tests for switching speed
- Accessibility tests for multilingual support

## BUSINESS DOMAIN CONTEXT
**Healthcare Testing Priorities:**
- Medical terminology accuracy in translations
- Healthcare workflow language switching performance
- Clinical documentation multilingual support
- Patient-facing interface language quality
- Emergency situation language accessibility

**Quality Standards:**
- Zero tolerance for medical translation errors
- Professional healthcare terminology consistency
- Cultural appropriateness verification
- Performance standards for medical workflows

## FUNCTIONAL REQUIREMENTS
1. **Unit Testing Suite**
   - Test i18n configuration initialization
   - Test translation hook functionality
   - Test localization utility functions
   - Test language preference persistence

2. **Integration Testing**
   - Test complete language switching workflows
   - Test component integration with i18n system
   - Test RTL/LTR layout transitions
   - Test translation loading and caching

3. **Visual and Layout Testing**
   - RTL layout visual regression tests
   - Component positioning in both directions
   - Arabic text rendering verification
   - Mixed content display testing

4. **Performance and Accessibility Testing**
   - Language switching performance benchmarks
   - Translation loading speed tests
   - Screen reader compatibility verification
   - Keyboard navigation accessibility tests

## TECHNICAL REQUIREMENTS
```yaml
testing_framework:
  - jest: "Unit test framework"
  - react_testing_library: "Component testing utilities"
  - playwright: "E2E and visual testing"
  - axe_core: "Accessibility testing"

test_structure:
  - "src/__tests__/i18n/"
  - "src/__tests__/localization/"
  - "src/__tests__/components/language/"
  - "tests/e2e/i18n/"

performance_benchmarks:
  - language_switching: "< 300ms complete switch"
  - translation_loading: "< 100ms namespace loading"
  - rtl_transition: "< 200ms layout adaptation"

accessibility_standards:
  - wcag_2_1_aa: "WCAG 2.1 AA compliance"
  - screen_readers: "NVDA, JAWS, VoiceOver compatibility"
  - keyboard_navigation: "Full keyboard accessibility"
```

## TESTING REQUIREMENTS
**Unit Tests:**
```typescript
// Test i18n configuration
describe('i18n Configuration', () => {
  it('should initialize with correct languages')
  it('should detect browser language correctly')
  it('should fallback to French when Arabic missing')
  it('should persist language preference')
})

// Test localization utilities
describe('Localization Utilities', () => {
  it('should format dates correctly for Arabic locale')
  it('should format phone numbers for Algeria')
  it('should format patient names culturally appropriate')
  it('should handle currency formatting for DZD')
})

// Test RTL utilities
describe('RTL Support', () => {
  it('should detect RTL requirement for Arabic')
  it('should apply correct direction classes')
  it('should handle bidirectional text correctly')
})
```

**Integration Tests:**
```typescript
// Test complete language switching
describe('Language Switching Integration', () => {
  it('should switch language and update all UI elements')
  it('should change direction from LTR to RTL')
  it('should persist preference across page reloads')
  it('should sync with user profile when available')
})
```

**E2E Tests:**
```typescript
// Test user workflows in different languages
describe('Multilingual User Workflows', () => {
  it('should complete patient registration in Arabic')
  it('should schedule appointment in French')
  it('should switch languages mid-workflow')
  it('should maintain form data during language switch')
})
```

**Performance Tests:**
- Language switching speed benchmarks
- Translation loading performance
- RTL layout transition timing
- Memory usage during language operations

**Accessibility Tests:**
- Screen reader announcement testing
- Keyboard navigation verification
- Color contrast in both languages
- Focus management during language switching

## DOCUMENTATION REQUIREMENTS
**Testing Documentation:**
- i18n testing strategy and approach
- Test case documentation for translations
- Performance benchmark documentation
- Accessibility testing checklist

**Quality Assurance Guide:**
- Translation review process
- RTL layout verification checklist
- Healthcare terminology validation process
- Cultural sensitivity review guidelines

## VALIDATION CRITERIA
- [ ] Unit test suite covers all i18n utilities and hooks (>90% coverage)
- [ ] Integration tests verify complete language switching workflows
- [ ] Visual regression tests confirm RTL layout correctness
- [ ] Performance tests validate language switching speed (<300ms)
- [ ] Accessibility tests pass WCAG 2.1 AA standards
- [ ] Translation completeness verified (no missing keys)
- [ ] Medical terminology accuracy validated by healthcare professionals
- [ ] Screen reader compatibility confirmed for both languages
- [ ] Keyboard navigation works in RTL and LTR modes
- [ ] E2E tests cover critical user workflows in both languages
- [ ] Performance benchmarks meet healthcare workflow requirements
- [ ] Memory leaks prevented during repeated language switching
- [ ] Error handling tested for missing translations and malformed data
- [ ] Cultural sensitivity review completed for all translations
- [ ] Documentation updated with testing procedures and standards