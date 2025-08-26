# PROMPTS: TASK-008 - Frontend UI Framework Integration

# PROMPT 1: Tailwind CSS Setup and Configuration

Set up Tailwind CSS in the React application with healthcare-optimized configuration, custom themes, and responsive design utilities. Configure Tailwind to work with Vite build system and establish the foundation for the design system.

## PROJECT CONTEXT

**Project:** MedFlow - Healthcare Practice Management System for Algerian clinics
**Tech Stack:** React + TypeScript + Vite for frontend, NestJS for backend
**Architecture:** Domain-driven design with feature-based folder structure
**Frontend Structure:**
```
src/
├── components/           # Shared UI components
├── features/            # Feature-based modules
├── lib/                 # Core utilities and services
├── hooks/               # Shared custom hooks
├── assets/              # Static assets
└── App.tsx              # Root component
```

**Current State:** React application is set up with TypeScript and Vite (TASK-007 completed)

## EPIC CONTEXT

**Epic:** EPIC-000 - Project Setup and Infrastructure
**Goal:** Establish foundational technical infrastructure for rapid healthcare feature development
**Dependencies:** Frontend React Application Setup (TASK-007) must be completed
**Provides:** UI framework foundation for all feature development and layout system

## TASK CONTEXT

**Task:** TASK-008 - Frontend UI Framework Integration
**Goal:** Integrate Tailwind CSS and shadcn/ui component library with healthcare-optimized design system
**Priority:** HIGH - Required foundation for all UI development
**Complexity:** MEDIUM

## BUSINESS DOMAIN CONTEXT

**Healthcare Interface Requirements:**
- Professional appearance appropriate for medical environments
- Support for both single-doctor and multi-provider clinic modes
- Responsive design for desktop and tablet use in clinical settings
- Accessibility compliance for healthcare professionals
- Foundation for Arabic/French language support (RTL/LTR)

## FUNCTIONAL REQUIREMENTS

1. **Tailwind CSS Installation and Configuration**
   - Install Tailwind CSS, PostCSS, and Autoprefixer
   - Configure Vite to process Tailwind CSS
   - Set up content paths for proper CSS purging
   - Create base CSS file with Tailwind directives

2. **Healthcare-Optimized Theme Configuration**
   - Define custom color palette suitable for medical interfaces
   - Configure typography scales appropriate for healthcare data
   - Set up spacing system optimized for clinical workflows
   - Configure responsive breakpoints for healthcare environments

3. **CSS Architecture Setup**
   - Establish CSS organization structure
   - Configure CSS custom properties for theming
   - Set up utility classes for common healthcare UI patterns
   - Create foundation for RTL support

## TECHNICAL REQUIREMENTS

1. **Performance Optimization**
   - CSS bundle size under 50KB for initial load
   - Proper CSS purging configuration
   - Optimize build process for development and production

2. **Development Experience**
   - IntelliSense support for Tailwind classes
   - Proper integration with TypeScript
   - Hot reload functionality for CSS changes

3. **Accessibility Foundation**
   - Color contrast ratios meeting WCAG 2.1 AA standards
   - Focus indicators and screen reader compatibility
   - Semantic color naming system

## TESTING REQUIREMENTS

**Manual Tests:**
- [ ] Verify Tailwind classes apply correctly in components
- [ ] Test responsive breakpoints work as expected
- [ ] Validate custom theme colors display properly
- [ ] Check CSS build process works in development and production
- [ ] Test hot reload functionality for CSS changes

**Integration Tests:**
- [ ] Verify Vite build process includes Tailwind CSS
- [ ] Test CSS purging removes unused styles
- [ ] Validate custom theme variables are accessible

## DOCUMENTATION REQUIREMENTS

- Update project README with Tailwind CSS setup instructions
- Document custom theme configuration and usage
- Create style guide for healthcare-specific color usage
- Document responsive breakpoint usage guidelines

## VALIDATION CRITERIA

- [ ] Tailwind CSS properly installed and configured with Vite
- [ ] Custom healthcare theme configuration working correctly
- [ ] Responsive breakpoints configured for clinical environments
- [ ] CSS bundle size optimized and within performance targets
- [ ] Custom color palette applied and documented
- [ ] Development tools (IntelliSense) working with Tailwind
- [ ] Hot reload functionality working for CSS changes
- [ ] Foundation for RTL support configured
- [ ] Documentation updated with setup and usage instructions

# PROMPT 2: shadcn/ui Component Library Installation and Setup

Install and configure shadcn/ui component library with healthcare-specific customizations and accessibility features. Set up the component architecture and establish integration patterns with the existing React application.

## PROJECT CONTEXT

**Project:** MedFlow - Healthcare Practice Management System
**Frontend Stack:** React + TypeScript + Vite + Tailwind CSS (now configured)
**Component Architecture:** Feature-based structure with shared components
**Build System:** Vite with TypeScript and Tailwind CSS integration

**Current State:** Tailwind CSS is configured and working (Prompt 1 completed)

## EPIC CONTEXT

**Epic:** EPIC-000 - Project Setup and Infrastructure
**Integration Point:** Provides reusable component library for all feature development
**Downstream Dependencies:** Layout System (TASK-012), Authentication UI (TASK-010)

## TASK CONTEXT

**Task:** TASK-008 - Frontend UI Framework Integration
**Focus:** shadcn/ui component library setup and healthcare customization
**Priority:** HIGH - Core UI components needed for all features

## BUSINESS DOMAIN CONTEXT

**Healthcare Component Requirements:**
- Medical-grade accessibility (WCAG 2.1 AA compliance)
- Professional styling appropriate for clinical environments
- Form controls suitable for patient data entry
- Status indicators for appointment and patient states
- Data display components for medical information

## FUNCTIONAL REQUIREMENTS

1. **shadcn/ui Installation and Configuration**
   - Install shadcn/ui CLI and core dependencies
   - Initialize shadcn/ui configuration
   - Set up component installation directory structure
   - Configure integration with existing Tailwind theme

2. **Core Component Installation**
   - Button component with healthcare variants
   - Input components (text, email, phone, search)
   - Card component for patient/appointment displays
   - Dialog component for modals and confirmations
   - Form components with validation support

3. **Healthcare-Specific Customizations**
   - Custom variants for medical status indicators
   - Healthcare-appropriate color schemes
   - Accessibility enhancements for clinical use
   - Touch-friendly sizing for tablet interfaces

## TECHNICAL REQUIREMENTS

1. **Component Architecture**
   - Proper TypeScript integration with component props
   - Consistent prop patterns across components
   - Integration with React Hook Form for form components
   - Proper tree-shaking and bundle optimization

2. **Accessibility Standards**
   - ARIA attributes on all interactive components
   - Keyboard navigation support
   - Screen reader compatibility
   - Focus management for modal components

3. **Performance Optimization**
   - Lazy loading for large component sets
   - Optimal bundle splitting for components
   - Runtime performance for form interactions

## TESTING REQUIREMENTS

**Component Tests:**
- [ ] Test each installed component renders correctly
- [ ] Validate component props and variants work
- [ ] Test accessibility features (keyboard navigation, ARIA)
- [ ] Verify form components integrate with validation

**Integration Tests:**
- [ ] Test components work with existing Tailwind theme
- [ ] Validate component styling consistency
- [ ] Test responsive behavior across breakpoints

**Manual Tests:**
- [ ] Test touch interactions on tablet devices
- [ ] Validate color contrast meets WCAG standards
- [ ] Test screen reader compatibility

## DOCUMENTATION REQUIREMENTS

- Document installed components and their usage patterns
- Create component documentation with healthcare examples
- Document accessibility features and compliance
- Create component variant guide for healthcare contexts

## VALIDATION CRITERIA

- [ ] shadcn/ui CLI installed and configured correctly
- [ ] Core components installed and working with TypeScript
- [ ] Components integrate properly with Tailwind theme
- [ ] Healthcare variants and customizations applied
- [ ] All components meet WCAG 2.1 AA accessibility standards
- [ ] Form components integrate with validation libraries
- [ ] Components are responsive and touch-friendly
- [ ] Bundle size impact is acceptable (< 200KB additional)
- [ ] Component documentation created with examples
- [ ] Integration with existing React app architecture verified

# PROMPT 3: Healthcare Design System Implementation

Create a comprehensive healthcare-specific design system with color palette, typography, spacing, and component patterns optimized for medical interfaces and clinical workflows.

## PROJECT CONTEXT

**Project:** MedFlow - Healthcare Practice Management System
**Frontend Foundation:** React + TypeScript + Vite + Tailwind CSS + shadcn/ui
**Target Users:** Healthcare providers, clinical staff, administrative personnel
**Environment:** Desktop and tablet interfaces in clinical settings

**Current State:** Tailwind CSS and shadcn/ui components are configured (Prompts 1-2 completed)

## EPIC CONTEXT

**Epic:** EPIC-000 - Project Setup and Infrastructure
**Business Goal:** Professional healthcare interface that enables efficient clinical workflows
**Integration:** Foundation for all feature UI development and layout systems

## TASK CONTEXT

**Task:** TASK-008 - Frontend UI Framework Integration
**Focus:** Healthcare-specific design system and visual identity
**Priority:** MEDIUM - Establishes consistent visual foundation

## BUSINESS DOMAIN CONTEXT

**Healthcare Design Requirements:**
- Professional medical environment aesthetics
- Clear status communication (patient status, appointment states)
- Efficient information density for clinical data
- Trust-building visual design for patient interactions
- Compliance with medical software design standards

**Workflow Considerations:**
- Quick visual scanning of patient information
- Clear differentiation between critical and routine information
- Consistent status indicators across different workflows
- Reduced cognitive load during busy clinical periods

## FUNCTIONAL REQUIREMENTS

1. **Healthcare Color Palette System**
   - Primary colors suitable for medical environments
   - Status colors for appointments (scheduled, confirmed, in-progress, completed)
   - Patient status indicators (active, inactive, urgent)
   - Alert and notification color hierarchy
   - Accessible color combinations meeting WCAG 2.1 AA standards

2. **Medical Interface Typography**
   - Readable font hierarchy for clinical data
   - Typography scales for different information types
   - Clear font weights for data emphasis
   - Optimal line spacing for dense medical information
   - Support for Arabic and Latin text (RTL/LTR foundation)

3. **Healthcare-Optimized Spacing System**
   - Compact layouts for information-dense interfaces
   - Appropriate spacing for touch targets on tablets
   - Consistent spacing patterns for clinical workflows
   - Grid system for structured data display

4. **Component Pattern Library**
   - Status badge variants for healthcare contexts
   - Card patterns for patient and appointment information
   - Form patterns for medical data entry
   - Navigation patterns for clinical workflows

## TECHNICAL REQUIREMENTS

1. **Design Token System**
   - CSS custom properties for consistent theming
   - Semantic naming system for healthcare contexts
   - Easy customization for different clinic types
   - Support for single-doctor vs multi-provider modes

2. **Accessibility Standards**
   - Color contrast ratios >= 4.5:1 for normal text
   - Color contrast ratios >= 3:1 for large text
   - Non-color-dependent status indicators
   - Focus indicators for all interactive elements

3. **Responsive Design Foundation**
   - Mobile-first approach with desktop optimization
   - Tablet-specific optimizations for clinical use
   - Flexible layouts for varying content lengths
   - Touch-friendly interaction patterns

## TESTING REQUIREMENTS

**Visual Regression Tests:**
- [ ] Test color palette consistency across components
- [ ] Validate typography scales render correctly
- [ ] Test responsive behavior of design system elements

**Accessibility Tests:**
- [ ] Automated color contrast testing
- [ ] Screen reader compatibility testing
- [ ] Keyboard navigation testing
- [ ] Focus indicator visibility testing

**Manual Tests:**
- [ ] Test design system in realistic healthcare scenarios
- [ ] Validate visual hierarchy with clinical data
- [ ] Test touch interactions on tablet devices
- [ ] Verify status indicators are clearly distinguishable

## DOCUMENTATION REQUIREMENTS

- Create comprehensive design system documentation
- Document color usage guidelines for healthcare contexts
- Create component pattern library with examples
- Document accessibility compliance and testing procedures
- Create usage guidelines for single-doctor vs multi-provider modes

## VALIDATION CRITERIA

- [ ] Healthcare color palette implemented with semantic naming
- [ ] Typography system optimized for clinical data display
- [ ] Spacing system supports both compact and touch-friendly layouts
- [ ] Status indicator system covers all healthcare workflow states
- [ ] All colors meet WCAG 2.1 AA contrast requirements
- [ ] Design tokens implemented with CSS custom properties
- [ ] Responsive design principles applied consistently
- [ ] Component patterns documented with healthcare examples
- [ ] Visual hierarchy tested with realistic medical content
- [ ] Foundation for RTL support properly configured
- [ ] Single-doctor mode styling variants created
- [ ] Design system documentation comprehensive and accessible

# PROMPT 4: Responsive Layout Foundation and Component Integration

Implement responsive layout utilities and demonstrate the complete UI framework integration with sample healthcare components that showcase the design system, accessibility features, and responsive behavior.

## PROJECT CONTEXT

**Project:** MedFlow - Healthcare Practice Management System
**UI Foundation:** React + TypeScript + Vite + Tailwind CSS + shadcn/ui + Healthcare Design System
**Target Devices:** Desktop (primary), tablet (clinical use), mobile (limited support)
**User Contexts:** Clinical environments with varying screen sizes and usage patterns

**Current State:** Design system implemented with healthcare theming (Prompts 1-3 completed)

## EPIC CONTEXT

**Epic:** EPIC-000 - Project Setup and Infrastructure
**Integration Point:** Complete UI framework ready for feature development
**Provides:** Responsive foundation for layout system (TASK-012) and all feature UIs

## TASK CONTEXT

**Task:** TASK-008 - Frontend UI Framework Integration
**Focus:** Responsive utilities and complete framework integration validation
**Priority:** HIGH - Final validation of complete UI framework setup

## BUSINESS DOMAIN CONTEXT

**Healthcare Responsive Requirements:**
- Desktop: Primary interface for detailed clinical work
- Tablet: Bedside consultations and mobile clinical work
- Mobile: Limited emergency access and basic information viewing
- Varying screen orientations in clinical environments

**Clinical Workflow Considerations:**
- Information density optimization across screen sizes
- Touch-friendly interfaces for tablet use
- Quick access patterns for urgent information
- Consistent experience across device types

## FUNCTIONAL REQUIREMENTS

1. **Responsive Layout Utilities**
   - Mobile-first responsive design implementation
   - Healthcare-optimized breakpoint system
   - Flexible grid system for clinical data
   - Container queries for component-level responsiveness

2. **Healthcare-Specific Responsive Patterns**
   - Patient card layouts that adapt to screen size
   - Appointment schedule views for different devices
   - Form layouts optimized for data entry
   - Navigation patterns that work across devices

3. **Sample Component Implementation**
   - Patient information card with responsive behavior
   - Appointment status dashboard component
   - Clinical data entry form with validation
   - Healthcare navigation component

4. **Framework Integration Validation**
   - All UI components working together seamlessly
   - Design system applied consistently
   - Accessibility features functioning properly
   - Performance optimization verified

## TECHNICAL REQUIREMENTS

1. **Responsive Design Standards**
   - Mobile-first CSS approach using Tailwind
   - Breakpoints: mobile (320px+), tablet (768px+), desktop (1024px+)
   - Flexible typography scaling
   - Adaptive spacing and layout patterns

2. **Component Architecture**
   - Reusable responsive component patterns
   - Prop-based responsive behavior control
   - Consistent API patterns across components
   - TypeScript interfaces for responsive props

3. **Performance Optimization**
   - Efficient CSS for responsive utilities
   - Optimized image handling for different densities
   - Lazy loading for non-critical responsive elements
   - Bundle size optimization for mobile devices

## TESTING REQUIREMENTS

**Responsive Testing:**
- [ ] Test all sample components across breakpoints
- [ ] Validate touch interactions on tablet devices
- [ ] Test responsive typography scaling
- [ ] Verify layout stability during orientation changes

**Integration Testing:**
- [ ] Test complete UI framework stack working together
- [ ] Validate design system consistency across components
- [ ] Test accessibility features in responsive contexts
- [ ] Verify performance targets across device types

**Manual Testing:**
- [ ] Test real-world healthcare scenarios on different devices
- [ ] Validate clinical workflow efficiency across screen sizes
- [ ] Test with realistic medical content and data volumes
- [ ] Verify user experience consistency

## DOCUMENTATION REQUIREMENTS

- Complete UI framework integration guide
- Responsive design patterns documentation
- Sample component usage examples
- Healthcare-specific responsive guidelines
- Performance optimization best practices
- Complete setup and development workflow documentation

## VALIDATION CRITERIA

- [ ] Responsive layout utilities implemented and working
- [ ] Healthcare-optimized breakpoint system configured
- [ ] Sample components demonstrate complete framework integration
- [ ] All components responsive and accessible across device types
- [ ] Performance targets met (CSS bundle < 100KB, load time < 3s)
- [ ] Design system applied consistently across all samples
- [ ] Accessibility compliance verified (WCAG 2.1 AA)
- [ ] Touch interactions optimized for tablet use
- [ ] Healthcare workflow patterns implemented effectively
- [ ] TypeScript integration working properly with all components
- [ ] Hot reload and development experience optimized
- [ ] Complete documentation created and tested
- [ ] Integration with existing React app architecture verified
- [ ] Foundation ready for layout system development (TASK-012)
- [ ] All UI framework requirements from TASK-008 satisfied