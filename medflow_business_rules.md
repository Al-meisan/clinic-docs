# MedFlow Business Rules

## Patient Management Rules

### Patient Registration (BR-P001 to BR-P007)
- **BR-P001**: Every patient must have a unique patient ID generated automatically upon registration
- **BR-P002**: Patient first name, last name, date of birth, and phone number are mandatory fields
- **BR-P003**: Patient email address must be unique across the system if provided
- **BR-P004**: Patient date of birth cannot be in the future
- **BR-P005**: Patient age must be calculated accurately for minors and adults
- **BR-P006**: New patients must be assigned 'active' status by default
- **BR-P007**: Patient records must maintain audit trail of all modifications with user ID and timestamp

### Patient Status Management (BR-P008 to BR-P016)
- **BR-P008**: Patient status must follow valid transitions: active ↔ inactive, active → deceased, active → merged
- **BR-P009**: Only authorized users with `patient:write` scope can modify patient status
- **BR-P010**: Deceased patients cannot be changed to any other status
- **BR-P011**: Inactive patients can still have appointments but should show warnings
- **BR-P012**: Patient status changes must be logged with reason and timestamp
- **BR-P013**: Patients cannot be permanently deleted, only status changed
- **BR-P014**: Active patients must have valid contact information
- **BR-P015**: Patient reactivation from inactive status requires verification of contact information
- **BR-P016**: System must prevent duplicate patient creation based on name, DOB, and phone combination

### Patient Search and Retrieval (BR-P017 to BR-P021)
- **BR-P017**: Patient search must support name, phone, DOB, and patient ID criteria
- **BR-P018**: Search results must be sorted by relevance with exact matches first
- **BR-P019**: Search must be case-insensitive and support partial matching
- **BR-P020**: Recent patients list must show last 10 accessed patients per user
- **BR-P021**: Search results must respect user permissions and clinic boundaries

## Provider Management Rules (BR-PR001 to BR-PR015)

### Provider Registration (BR-PR001 to BR-PR008)
- **BR-PR001**: Provider medical license number must be unique and valid
- **BR-PR002**: Provider NPI number must be unique and follow valid format
- **BR-PR003**: Provider must have at least one medical specialty assigned
- **BR-PR004**: Provider credentials must include license expiration date
- **BR-PR005**: Only users with `clinic:manage` scope can create provider records
- **BR-PR006**: Provider email must be unique across the system
- **BR-PR007**: DEA number is optional but must be valid format if provided
- **BR-PR008**: Provider status defaults to 'active' upon creation

### Provider Schedule Management (BR-PR009 to BR-PR015)
- **BR-PR009**: Provider schedule must have at least one available time slot per week
- **BR-PR010**: Appointment types must have valid duration (15-480 minutes)
- **BR-PR011**: Break times cannot overlap with each other on the same day
- **BR-PR012**: Working hours must be logical (start time before end time)
- **BR-PR013**: Time off requests must not conflict with existing appointments
- **BR-PR014**: Provider availability determines appointment booking slots
- **BR-PR015**: Schedule changes affecting existing appointments require confirmation

## Appointment Management Rules (BR-A001 to BR-A030)

### Appointment Scheduling (BR-A001 to BR-A015)
- **BR-A001**: Appointments cannot be double-booked for the same provider and time slot
- **BR-A002**: Appointment duration must match available provider schedule slots
- **BR-A003**: Appointment start time must be in the future (cannot schedule in the past)
- **BR-A004**: Maximum advance booking period is configurable per clinic (default 90 days)
- **BR-A005**: Patient must exist in system before scheduling appointment
- **BR-A006**: Provider must be active and available during requested time slot
- **BR-A007**: Appointment type must be valid for the selected provider
- **BR-A008**: Walk-in appointments can be scheduled if clinic allows and slots are available
- **BR-A009**: Recurring appointments follow the same booking rules for each instance
- **BR-A010**: Appointment conflicts must be resolved before confirmation
- **BR-A011**: Minimum notice period for cancellation is configurable (default 2 hours)
- **BR-A012**: Same-day appointments require provider approval if outside normal booking rules
- **BR-A013**: Appointment buffer time must be respected between consecutive appointments
- **BR-A014**: Emergency appointments can override normal scheduling restrictions with proper authorization
- **BR-A015**: Appointment creation requires `appointment:write` scope

### Appointment Status Transitions (BR-A016 to BR-A025)
- **BR-A016**: Appointment status must follow valid progression: scheduled → confirmed → checked_in → in_progress → completed → billed
- **BR-A017**: Alternative status transitions allowed: scheduled/confirmed → cancelled, scheduled/confirmed → rescheduled, checked_in → no_show
- **BR-A018**: Status changes must be logged with timestamp, user ID, and reason
- **BR-A019**: Completed appointments cannot be changed to earlier statuses without special authorization
- **BR-A020**: No_show status can only be set after appointment time has passed
- **BR-A021**: Cancelled appointments free up the time slot for rebooking
- **BR-A022**: Rescheduled appointments must reference the new appointment ID
- **BR-A023**: In_progress status can only be set by the assigned provider or authorized clinical staff
- **BR-A024**: Status transitions must respect user permissions and scopes

### Appointment Modification (BR-A026 to BR-A030)
- **BR-A025**: Appointment reschedule must follow the same availability rules as new appointments
- **BR-A026**: Cancellation requires reason and notification preferences
- **BR-A027**: Modifications to completed appointments require special authorization
- **BR-A028**: Provider changes require availability verification and patient notification
- **BR-A029**: Appointment modifications must update related waiting list entries

## Check-in Process Rules (BR-C001 to BR-C010)
- **BR-C001**: Patient can only check in within configured time window (default 30 minutes early to 15 minutes late)
- **BR-C002**: Check-in requires valid appointment in 'scheduled' or 'confirmed' status
- **BR-C003**: Contact information updates during check-in must be validated
- **BR-C004**: Check-in updates appointment status to 'checked_in' automatically
- **BR-C005**: Multiple check-ins for same appointment are not allowed
- **BR-C006**: Check-in time must be recorded for wait time calculation
- **BR-C007**: Failed check-in attempts must be logged for troubleshooting

## Waiting List Management Rules (BR-W001 to BR-W010)
- **BR-W001**: Patients can be added to waiting list when preferred time slots are unavailable
- **BR-W002**: Waiting list entries must specify preferred dates, times, and appointment type
- **BR-W003**: Priority levels (low, medium, high, urgent) determine notification order
- **BR-W004**: Contact preferences must be specified for notification methods
- **BR-W005**: Waiting list entries expire after configurable period (default 30 days)
- **BR-W006**: Automatic matching triggers when appointments are cancelled or slots become available
- **BR-W007**: Notification attempts must be logged with response tracking
- **BR-W008**: Successfully booked patients are automatically removed from waiting list
- **BR-W009**: Patients can be on waiting list for multiple appointment types simultaneously
- **BR-W010**: Waiting list priority can be adjusted by authorized staff based on medical urgency

## Clinical Documentation Rules (BR-CD001 to BR-CD020)

### Medical Records Documentation (BR-CD001 to BR-CD010)
- **BR-CD001**: All clinical documentation must be associated with a valid appointment
- **BR-CD002**: Chief complaint is mandatory for all clinical encounters
- **BR-CD003**: Assessment and plan sections are required for completed documentation
- **BR-CD004**: Clinical documentation must be signed by the attending provider
- **BR-CD005**: Unsigned documentation cannot be finalized or used for billing
- **BR-CD006**: Documentation templates can be used but must be customized for each patient
- **BR-CD007**: Previous visit notes must be accessible during current documentation
- **BR-CD008**: Critical findings must be flagged and followed up appropriately
- **BR-CD009**: Documentation modifications after signing require special authorization
- **BR-CD010**: All clinical documentation must be timestamped with creation and modification dates

### Medical History Management (BR-CD011 to BR-CD020)
- **BR-CD011**: Medical history updates require `clinical:write` permissions
- **BR-CD012**: Allergy information must be prominently displayed and easily accessible
- **BR-CD013**: Current medications list must be reconciled at each visit
- **BR-CD014**: Family history must include relationship and relevant conditions
- **BR-CD015**: Social history updates should be performed annually or when clinically relevant
- **BR-CD016**: Immunization records must include vaccine name, date, and provider
- **BR-CD017**: Previous medical conditions require status (active, resolved, chronic)
- **BR-CD018**: Drug allergies must include reaction type and severity
- **BR-CD019**: Medical history changes must maintain audit trail
- **BR-CD020**: Historical data cannot be deleted, only marked as entered in error with corrections

## Prescription Management Rules (BR-PM001 to BR-PM020)

### Electronic Prescribing (BR-PM001 to BR-PM012)
- **BR-PM001**: Only licensed providers with `prescription:write` scope can prescribe medications
- **BR-PM002**: All prescriptions must be associated with a patient encounter or appointment
- **BR-PM003**: Drug interaction checking is mandatory before prescription submission
- **BR-PM004**: Allergy checking against patient's known allergies is required
- **BR-PM005**: Prescription must include medication name, strength, form, dosage, frequency, quantity, and refills
- **BR-PM006**: DEA-controlled substances require valid DEA number and additional verification
- **BR-PM007**: Prescription quantity must be appropriate for duration and frequency specified
- **BR-PM008**: Special instructions and patient education must be included when applicable
- **BR-PM009**: Pharmacy information must be verified before electronic transmission
- **BR-PM010**: Prescription errors must be corrected through proper amendment process, not deletion
- **BR-PM011**: All prescriptions must be electronically signed .
- **BR-PM012**: Prescription history must be maintained for audit and continuity purposes

### Prescription Renewal Rules (BR-PM013 to BR-PM020)
- **BR-PM013**: Prescription renewals require clinical review.
- **BR-PM014**: Renewal requests must be processed within configurable timeframe (default 48 hours)
- **BR-PM015**: Number of renewals must not exceed the original prescription authorization
- **BR-PM016**: Controlled substances have stricter renewal requirements and may require new appointments
- **BR-PM017**: Renewal decisions (approve, deny, require appointment) must be documented with reasons
- **BR-PM018**: Patient medication adherence should be considered in renewal decisions
- **BR-PM019**: Renewal notifications must be sent to patient as appropriate
- **BR-PM020**: Expired prescriptions cannot be renewed; new prescriptions must be written

## Authorization and Security Rules (BR-S001 to BR-S020)

### Role-Based Access Control (BR-S001 to BR-S010)
- **BR-S001**: User access is controlled through OAuth2 scopes based on roles and permissions.
- **BR-S002**: System Administrator has `admin:full` scope with complete system access
- **BR-S003**: Clinic Manager has `clinic:manage` plus operational scopes
- **BR-S004**: Healthcare Provider has clinical and prescription scopes
- **BR-S005**: Clinical Staff has clinical scopes with limited patient write access
- **BR-S006**: Administrative Staff has patient and appointment scopes with limited clinical read access
- **BR-S007**: `prescription:write` scope is restricted to licensed prescribers only
- **BR-S008**: `clinical:write` scope requires clinical training/certification
- **BR-S009**: Cross-clinic data access is prohibited.
- **BR-S010**: User sessions must expire after configurable idle time (default 30 minutes)

### Data Security and Privacy (BR-S011 to BR-S020)
- **BR-S011**: All patient data must be encrypted at rest and in transit
- **BR-S012**: User access must be logged for audit purposes
- **BR-S013**: Patient data export requires special authorization and audit logging
- **BR-S014**: User password must meet complexity requirements and expire periodically
- **BR-S015**: Failed login attempts must be tracked and account locked after threshold
- **BR-S016**: Data backup and recovery procedures must be tested regularly
- **BR-S017**: Patient consent is required for data sharing with external entities
- **BR-S018**: User access must be reviewed and updated when roles change
- **BR-S019**: System access logs must be retained for regulatory compliance period

## Billing and Payment Rules (BR-B001 to BR-B015)

### Billing Process (BR-B001 to BR-B008)
- **BR-B001**: Bills can only be generated for completed appointments
- **BR-B002**: Service charges must be based on established fee schedule
- **BR-B003**: Bills must include itemized services with appropriate codes
- **BR-B004**: Tax calculations must follow local Algerian tax regulations
- **BR-B005**: Bill modifications require authorization and audit trail
- **BR-B006**: Billing requires `billing:write` scope permissions

### Payment Processing (BR-B009 to BR-B015)
- **BR-B007**: Payments can be collected via cash, card, bank transfer, or installment plans
- **BR-B008**: Payment amounts cannot exceed outstanding bill balance
- **BR-B009**: Partial payments must be properly allocated across bill items
- **BR-B010**: Payment reversals require special authorization and documentation
- **BR-B011**: Outstanding balances must be tracked and aged appropriately
- **BR-B012**: Payment plans must have defined terms and automatic monitoring
- **BR-B013**: All payment transactions must be recorded with audit trail

## Data Integrity and System Rules (BR-DI001 to BR-DI015)

### Data Validation (BR-DI001 to BR-DI008)
- **BR-DI001**: All required fields must be validated before record creation
- **BR-DI002**: Date fields must be in valid format and logical ranges
- **BR-DI003**: Phone numbers must follow valid format for region
- **BR-DI004**: Email addresses must be in valid format
- **BR-DI005**: Numeric fields must be within acceptable ranges
- **BR-DI006**: Foreign key relationships must be validated before creation
- **BR-DI007**: Duplicate detection must prevent creation of redundant records
- **BR-DI008**: Data import must validate all fields before bulk creation

### System Configuration (BR-DI009 to BR-DI015)
- **BR-DI009**: Clinic operating hours must be configured for appointment scheduling
- **BR-DI010**: System timezone must be properly configured for accurate timestamps
- **BR-DI011**: Default appointment duration can be configured per clinic
- **BR-DI012**: Reminder timing can be customized (default 24 hours and 2 hours before)
- **BR-DI013**: Maximum advance booking period is configurable per clinic
- **BR-DI014**: User interface language must be selectable (Arabic/French)
- **BR-DI015**: System backup must be performed daily with retention policy

## Notification and Communication Rules (BR-N001 to BR-N010)
- **BR-N001**: Appointment reminders must be sent based on patient preferences
- **BR-N002**: Notification methods include SMS, email, and phone calls
- **BR-N003**: Critical alerts (prescription interactions, urgent results) must be immediately delivered
- **BR-N004**: Notification delivery status must be tracked and logged
- **BR-N005**: Failed notification attempts must be retried with escalation procedures
- **BR-N006**: Patients can opt-out of non-critical notifications
- **BR-N007**: Provider notifications for schedule changes must be immediate
- **BR-N008**: Notification templates must support multiple languages
- **BR-N009**: Communication logs must be maintained for audit purposes

## Multi-Tenancy and Clinic Mode Rules (BR-MT001 to BR-MT010)
- **BR-MT001**: Single-doctor practices operate in simplified mode with reduced UI complexity
- **BR-MT002**: Multi-provider clinics have full feature access including provider management
- **BR-MT003**: Data isolation must be maintained between different clinics
- **BR-MT004**: Clinic mode can be upgraded but not downgraded without data review
- **BR-MT005**: Provider scheduling in single-doctor mode assumes single provider
- **BR-MT006**: Multi-provider mode requires provider selection for all appointments
- **BR-MT007**: Clinic settings inheritance follows mode-specific defaults
- **BR-MT008**: Feature availability is controlled by clinic mode and subscription plan
- **BR-MT009**: Data sharing between clinics requires explicit authorization
- **BR-MT010**: Clinic mode changes require administrative approval and system reconfiguration
