# Allersense - Requirements Document

## Project Overview

**Project Name:** Allersense  
**Version:** 1.0  
**Date:** February 2026  
**Status:** Hackathon Prototype → MVP Development

Allersense is an AI-powered allergen detection platform that prevents allergic reactions by scanning medications, topical products, and cosmetics for hidden allergens. The system uses AWS AI services to intelligently match product ingredients against user allergy profiles, detecting even obscure botanical names and cross-reactive ingredients.

---

## 1. Functional Requirements

### 1.1 User Authentication & Profile Management

#### FR-1.1.1: User Registration
- **Description:** Users shall be able to create accounts using email, phone number, or social login (Google, Apple)
- **Priority:** MUST HAVE
- **Acceptance Criteria:**
  - Email verification required within 24 hours
  - Strong password requirements (min 8 chars, uppercase, lowercase, number, special char)
  - Multi-factor authentication available
  - HIPAA-compliant consent forms displayed and accepted
  - User data encrypted at rest and in transit

#### FR-1.1.2: Allergy Profile Creation
- **Description:** Users shall be able to create and maintain comprehensive allergy profiles
- **Priority:** MUST HAVE
- **Acceptance Criteria:**
  - Support for food, medication, environmental, and contact allergies
  - Severity levels: mild, moderate, severe, life-threatening
  - Reaction history tracking (symptoms, dates, treatments)
  - Medical documentation upload (PDF, images)
  - Healthcare provider verification option
  - Multiple profiles per account (for family management)

#### FR-1.1.3: Profile Sharing
- **Description:** Users shall be able to securely share allergy profiles with healthcare providers
- **Priority:** SHOULD HAVE
- **Acceptance Criteria:**
  - Time-limited access codes (1 hour, 1 day, 1 week)
  - Revocable access at any time
  - Audit log of all profile access
  - QR code generation for emergency situations
  - FHIR-compliant data export

### 1.2 Product Scanning & Analysis

#### FR-1.2.1: Barcode Scanning
- **Description:** Users shall be able to scan product barcodes to retrieve ingredient information
- **Priority:** MUST HAVE
- **Acceptance Criteria:**
  - Support for UPC, EAN, Code 128, QR codes
  - Scan success rate > 90% in various lighting conditions
  - Offline cache of previously scanned products
  - Fallback to manual barcode entry
  - Support for damaged/partial barcodes

#### FR-1.2.2: Ingredient Label OCR
- **Description:** System shall extract ingredient lists from product label images using OCR
- **Priority:** MUST HAVE
- **Acceptance Criteria:**
  - OCR accuracy > 90% for clear images
  - Support for multiple languages (English, Hindi, Spanish, French)
  - Text extraction from various label formats (tables, columns, curved text)
  - Image quality validation and user feedback
  - Multi-page label support
  - Handwriting recognition for prescription labels

#### FR-1.2.3: Manual Ingredient Entry
- **Description:** Users shall be able to manually enter ingredients for products not in database
- **Priority:** MUST HAVE
- **Acceptance Criteria:**
  - Auto-complete suggestions from ingredient database
  - Ingredient name validation
  - Community contribution moderation
  - User-submitted data flagged until verified

#### FR-1.2.4: Allergen Detection
- **Description:** System shall analyze ingredients against user allergy profile using AI
- **Priority:** MUST HAVE
- **Acceptance Criteria:**
  - Detection accuracy > 95% for direct allergen matches
  - Synonym recognition (e.g., "Prunus amygdalus" = almond)
  - Botanical name matching
  - Chemical name cross-referencing (CAS numbers)
  - Multi-language ingredient name support
  - Response time < 3 seconds for scan analysis

#### FR-1.2.5: Cross-Reactivity Detection
- **Description:** System shall identify cross-reactive allergens based on medical research
- **Priority:** SHOULD HAVE
- **Acceptance Criteria:**
  - Latex-fruit syndrome detection
  - Oral allergy syndrome patterns
  - Related protein cross-reactivity
  - Confidence scores for cross-reactive matches
  - User education on cross-reactivity risks

### 1.3 Alert & Notification System

#### FR-1.3.1: Real-Time Allergen Alerts
- **Description:** System shall immediately alert users when allergens are detected
- **Priority:** MUST HAVE
- **Acceptance Criteria:**
  - Push notifications within 1 second of detection
  - Visual alerts (red warning screen)
  - Audio alerts (optional, configurable)
  - Haptic feedback on mobile devices
  - Alert severity levels based on allergy severity
  - Clear, non-technical language in alerts

#### FR-1.3.2: Safe Product Confirmation
- **Description:** System shall clearly indicate when products are safe to use
- **Priority:** MUST HAVE
- **Acceptance Criteria:**
  - Green checkmark visual confirmation
  - "Safe to use" message display
  - List of analyzed ingredients shown
  - Option to save safe products to favorites
  - Confidence score displayed

#### FR-1.3.3: Email/SMS Notifications
- **Description:** Users shall receive email and SMS alerts for critical allergen detections
- **Priority:** SHOULD HAVE
- **Acceptance Criteria:**
  - Configurable notification preferences
  - Emergency contact notification option
  - Daily/weekly scan summary emails
  - Product recall notifications
  - Opt-out capability for non-critical alerts

#### FR-1.3.4: Healthcare Provider Alerts
- **Description:** Integrated providers shall be notified when patients scan allergen-containing products
- **Priority:** COULD HAVE
- **Acceptance Criteria:**
  - Provider dashboard alerts
  - Patient risk summary
  - Intervention workflow
  - Alert aggregation to prevent notification fatigue

### 1.4 Healthcare Provider Portal

#### FR-1.4.1: Provider Authentication
- **Description:** Healthcare providers shall have secure access to provider portal
- **Priority:** MUST HAVE
- **Acceptance Criteria:**
  - NPI (National Provider Identifier) verification
  - Hospital/clinic credential verification
  - Role-based access control (doctor, nurse, pharmacist)
  - Audit logging of all access
  - Automatic session timeout (15 minutes)

#### FR-1.4.2: Patient Allergy Profile Access
- **Description:** Providers shall view patient allergy profiles with patient consent
- **Priority:** MUST HAVE
- **Acceptance Criteria:**
  - Patient consent required before access
  - Read-only access to patient data
  - Comprehensive allergy history view
  - Reaction timeline visualization
  - Export to EHR capability

#### FR-1.4.3: Pre-Prescription Product Check
- **Description:** Providers shall check products against patient allergies before prescribing
- **Priority:** MUST HAVE
- **Acceptance Criteria:**
  - Product search by name or barcode
  - Instant allergen match results
  - Alternative product suggestions
  - Integration with e-prescribing workflow
  - Documentation in patient record

#### FR-1.4.4: Provider Dashboard
- **Description:** Providers shall have analytics dashboard for patient allergy management
- **Priority:** SHOULD HAVE
- **Acceptance Criteria:**
  - Patient risk stratification
  - Common allergen patterns in patient population
  - Recent scan activity
  - Intervention success metrics
  - Exportable reports

### 1.5 EHR Integration

#### FR-1.5.1: FHIR API Integration
- **Description:** System shall integrate with EHR systems using FHIR standard
- **Priority:** MUST HAVE
- **Acceptance Criteria:**
  - FHIR R4 compliance
  - Allergy Intolerance resource support
  - Patient resource synchronization
  - Medication Statement integration
  - OAuth 2.0 authentication
  - Bi-directional data sync

#### FR-1.5.2: HL7 Messaging
- **Description:** System shall support HL7 v2.x messaging for legacy EHR systems
- **Priority:** SHOULD HAVE
- **Acceptance Criteria:**
  - ADT (Admission/Discharge/Transfer) message support
  - ORM (Order) message support
  - ORU (Results) message support
  - Message queue reliability
  - Error handling and retry logic

#### FR-1.5.3: Allergy Data Synchronization
- **Description:** Patient allergy data shall sync between Allersense and EHR systems
- **Priority:** MUST HAVE
- **Acceptance Criteria:**
  - Real-time or near-real-time sync (< 5 minutes)
  - Conflict resolution strategy
  - Audit trail of all changes
  - Data integrity validation
  - Rollback capability

### 1.6 Product Database & Management

#### FR-1.6.1: Product Database
- **Description:** System shall maintain comprehensive database of products and ingredients
- **Priority:** MUST HAVE
- **Acceptance Criteria:**
  - 100,000+ products at launch
  - Medication database (FDA)
  - Cosmetics database
  - OTC products database
  - Food products database
  - Weekly database updates

#### FR-1.6.2: Ingredient Synonym Database
- **Description:** System shall maintain database of ingredient synonyms and translations
- **Priority:** MUST HAVE
- **Acceptance Criteria:**
  - 50,000+ ingredient variations
  - Botanical names
  - Chemical names (IUPAC, CAS)
  - Brand names
  - Multi-language translations
  - Regular updates from medical literature

#### FR-1.6.3: Product Information Updates
- **Description:** System shall automatically update product information from external sources
- **Priority:** SHOULD HAVE
- **Acceptance Criteria:**
  - FDA API integration
  - OpenFoodFacts API integration
  - Manufacturer API integrations
  - Daily automated updates
  - Change notification to affected users

#### FR-1.6.4: User Contribution System
- **Description:** Users shall be able to contribute missing product information
- **Priority:** SHOULD HAVE
- **Acceptance Criteria:**
  - Photo upload of product label
  - Manual ingredient entry
  - Admin moderation queue
  - Contribution credit system
  - Quality validation before publication

### 1.7 Scan History & Reporting

#### FR-1.7.1: Scan History Tracking
- **Description:** System shall maintain complete history of all product scans
- **Priority:** MUST HAVE
- **Acceptance Criteria:**
  - Unlimited history retention
  - Search and filter capability
  - Sort by date, product name, result
  - Export to PDF/CSV
  - Delete history option (GDPR compliance)

#### FR-1.7.2: Analytics Dashboard
- **Description:** Users shall view analytics of their scanning activity
- **Priority:** SHOULD HAVE
- **Acceptance Criteria:**
  - Scans per week/month visualization
  - Allergen detection frequency
  - Most scanned product categories
  - Risk trends over time
  - Shareable reports

#### FR-1.7.3: Family Account Management
- **Description:** Users shall manage multiple family member profiles
- **Priority:** SHOULD HAVE
- **Acceptance Criteria:**
  - Add up to 5 family members
  - Separate allergy profiles per member
  - Scan history per member
  - Parental controls for children
  - Age-appropriate interfaces

### 1.8 Educational Content

#### FR-1.8.1: Allergen Information Library
- **Description:** System shall provide educational content about allergens
- **Priority:** SHOULD HAVE
- **Acceptance Criteria:**
  - Detailed allergen profiles
  - Symptoms and reactions
  - Emergency response procedures
  - Cross-reactivity information
  - Medical research references
  - Multi-language support

#### FR-1.8.2: Safe Product Recommendations
- **Description:** System shall suggest safe alternative products when allergens detected
- **Priority:** SHOULD HAVE
- **Acceptance Criteria:**
  - AI-powered recommendations
  - Similar efficacy products
  - Availability information
  - Price comparison
  - User ratings and reviews

---

## 2. Non-Functional Requirements

### 2.1 Performance Requirements

#### NFR-2.1.1: Response Time
- **Requirement:** Product scan analysis shall complete in < 3 seconds for 95% of requests
- **Priority:** MUST HAVE
- **Measurement:** Average response time monitored via CloudWatch

#### NFR-2.1.2: API Latency
- **Requirement:** API endpoints shall respond in < 200ms for 99th percentile
- **Priority:** MUST HAVE
- **Measurement:** P99 latency < 200ms

#### NFR-2.1.3: OCR Processing Time
- **Requirement:** Image OCR shall complete in < 5 seconds
- **Priority:** MUST HAVE
- **Measurement:** Average OCR processing time

#### NFR-2.1.4: Database Query Performance
- **Requirement:** Database queries shall execute in < 100ms for 95% of queries
- **Priority:** MUST HAVE
- **Measurement:** Database query metrics

### 2.2 Scalability Requirements

#### NFR-2.2.1: Concurrent Users
- **Requirement:** System shall support 100,000 concurrent users
- **Priority:** MUST HAVE
- **Measurement:** Load testing results

#### NFR-2.2.2: Scan Volume
- **Requirement:** System shall process 1,000,000 scans per day
- **Priority:** MUST HAVE
- **Measurement:** Daily scan metrics

#### NFR-2.2.3: Auto-Scaling
- **Requirement:** System shall automatically scale based on load
- **Priority:** MUST HAVE
- **Measurement:** Auto-scaling events and resource utilization

#### NFR-2.2.4: Database Scalability
- **Requirement:** Database shall support 10 million user profiles
- **Priority:** SHOULD HAVE
- **Measurement:** Database capacity and performance under load

### 2.3 Availability Requirements

#### NFR-2.3.1: System Uptime
- **Requirement:** System shall maintain 99.9% uptime (8.76 hours downtime/year)
- **Priority:** MUST HAVE
- **Measurement:** Uptime monitoring and SLA tracking

#### NFR-2.3.2: Disaster Recovery
- **Requirement:** System shall recover from failures within 4 hours (RTO)
- **Priority:** MUST HAVE
- **Measurement:** Disaster recovery drills

#### NFR-2.3.3: Data Recovery
- **Requirement:** System shall recover data with max 1 hour data loss (RPO)
- **Priority:** MUST HAVE
- **Measurement:** Backup and recovery testing

#### NFR-2.3.4: Geographic Redundancy
- **Requirement:** System shall operate in multiple AWS regions for redundancy
- **Priority:** SHOULD HAVE
- **Measurement:** Multi-region deployment verification

### 2.4 Security Requirements

#### NFR-2.4.1: HIPAA Compliance
- **Requirement:** System shall be fully HIPAA compliant for handling PHI
- **Priority:** MUST HAVE
- **Measurement:** HIPAA compliance audit certification

#### NFR-2.4.2: Data Encryption
- **Requirement:** All data shall be encrypted at rest (AES-256) and in transit (TLS 1.3)
- **Priority:** MUST HAVE
- **Measurement:** Encryption verification audit

#### NFR-2.4.3: Authentication Security
- **Requirement:** System shall enforce strong authentication with MFA option
- **Priority:** MUST HAVE
- **Measurement:** Authentication security audit

#### NFR-2.4.4: Authorization
- **Requirement:** System shall implement role-based access control (RBAC)
- **Priority:** MUST HAVE
- **Measurement:** Access control testing

#### NFR-2.4.5: Audit Logging
- **Requirement:** All access to PHI shall be logged and retained for 7 years
- **Priority:** MUST HAVE
- **Measurement:** Audit log completeness verification

#### NFR-2.4.6: Penetration Testing
- **Requirement:** System shall undergo quarterly penetration testing
- **Priority:** MUST HAVE
- **Measurement:** Penetration test reports and remediation

#### NFR-2.4.7: Vulnerability Management
- **Requirement:** Critical vulnerabilities shall be patched within 24 hours
- **Priority:** MUST HAVE
- **Measurement:** Vulnerability scan results and patch times

### 2.5 Accuracy Requirements

#### NFR-2.5.1: Allergen Detection Accuracy
- **Requirement:** Allergen detection shall achieve > 95% accuracy
- **Priority:** MUST HAVE
- **Measurement:** False positive/negative rate < 5%

#### NFR-2.5.2: OCR Accuracy
- **Requirement:** OCR text extraction shall achieve > 90% accuracy
- **Priority:** MUST HAVE
- **Measurement:** Character-level accuracy testing

#### NFR-2.5.3: Barcode Scanning Accuracy
- **Requirement:** Barcode scanning shall achieve > 95% first-scan success rate
- **Priority:** MUST HAVE
- **Measurement:** Scan success rate metrics

#### NFR-2.5.4: Ingredient Matching Accuracy
- **Requirement:** Ingredient synonym matching shall achieve > 98% accuracy
- **Priority:** MUST HAVE
- **Measurement:** Synonym match validation testing

### 2.6 Usability Requirements

#### NFR-2.6.1: Accessibility
- **Requirement:** System shall meet WCAG 2.1 AA accessibility standards
- **Priority:** MUST HAVE
- **Measurement:** Accessibility audit compliance

#### NFR-2.6.2: Mobile Responsiveness
- **Requirement:** Mobile app shall work on devices iOS 13+ and Android 8+
- **Priority:** MUST HAVE
- **Measurement:** Device compatibility testing

#### NFR-2.6.3: Screen Reader Support
- **Requirement:** All features shall be accessible via screen readers
- **Priority:** MUST HAVE
- **Measurement:** Screen reader testing (JAWS, NVDA, VoiceOver)

#### NFR-2.6.4: Language Support
- **Requirement:** System shall support English, Hindi, Spanish, and French
- **Priority:** SHOULD HAVE
- **Measurement:** Translation completeness and accuracy

#### NFR-2.6.5: User Onboarding
- **Requirement:** 90% of users shall complete profile setup within 5 minutes
- **Priority:** SHOULD HAVE
- **Measurement:** Onboarding completion analytics

#### NFR-2.6.6: Error Messaging
- **Requirement:** All error messages shall be clear, actionable, and non-technical
- **Priority:** MUST HAVE
- **Measurement:** User testing feedback

### 2.7 Reliability Requirements

#### NFR-2.7.1: Error Rate
- **Requirement:** API error rate shall be < 0.1%
- **Priority:** MUST HAVE
- **Measurement:** Error rate monitoring

#### NFR-2.7.2: Data Integrity
- **Requirement:** Zero data loss or corruption
- **Priority:** MUST HAVE
- **Measurement:** Data integrity validation checks

#### NFR-2.7.3: Offline Capability
- **Requirement:** App shall cache last 100 scanned products for offline access
- **Priority:** SHOULD HAVE
- **Measurement:** Offline functionality testing

### 2.8 Maintainability Requirements

#### NFR-2.8.1: Code Quality
- **Requirement:** Code shall maintain > 80% test coverage
- **Priority:** MUST HAVE
- **Measurement:** Code coverage reports

#### NFR-2.8.2: Documentation
- **Requirement:** All APIs shall have complete OpenAPI/Swagger documentation
- **Priority:** MUST HAVE
- **Measurement:** Documentation completeness review

#### NFR-2.8.3: Deployment Frequency
- **Requirement:** System shall support daily deployments with zero downtime
- **Priority:** SHOULD HAVE
- **Measurement:** Deployment frequency and downtime metrics

### 2.9 Compliance Requirements

#### NFR-2.9.1: GDPR Compliance
- **Requirement:** System shall comply with GDPR for EU users
- **Priority:** MUST HAVE
- **Measurement:** GDPR compliance audit

#### NFR-2.9.2: Data Retention
- **Requirement:** User data shall be deleted within 30 days of account deletion request
- **Priority:** MUST HAVE
- **Measurement:** Data deletion verification

#### NFR-2.9.3: Terms of Service
- **Requirement:** Users shall accept ToS and Privacy Policy before use
- **Priority:** MUST HAVE
- **Measurement:** User consent tracking

---

## 3. User Stories

### 3.1 Patient User Stories

#### US-3.1.1: Product Scanning
**As a** patient with severe nut allergies  
**I want to** scan a lotion before purchasing  
**So that** I can avoid products containing tree nuts or nut oils

**Acceptance Criteria:**
- Scan completes in < 3 seconds
- Clear alert if allergens detected
- List of all detected allergens shown
- Safe alternatives suggested

#### US-3.1.2: Emergency Profile Access
**As a** patient with multiple allergies  
**I want to** generate a QR code of my allergy profile  
**So that** emergency responders can quickly access my allergen list

**Acceptance Criteria:**
- QR code generated instantly
- Contains critical allergy information
- Works offline
- Updates automatically when profile changes

#### US-3.1.3: Family Management
**As a** parent of children with allergies  
**I want to** manage allergy profiles for all my children  
**So that** I can quickly check products for any family member

**Acceptance Criteria:**
- Switch between profiles easily
- Scan once, check all profiles
- Separate history per child
- Age-appropriate interface for kids

#### US-3.1.4: Prescription Checking
**As a** patient with medication allergies  
**I want to** scan my new prescription before filling it  
**So that** I can catch allergens my doctor might have missed

**Acceptance Criteria:**
- OCR reads prescription label
- Matches against my medication allergies
- Alerts before I pick up from pharmacy
- Provider notification option

### 3.2 Healthcare Provider User Stories

#### US-3.2.1: Pre-Prescription Check
**As a** physician  
**I want to** check if a medication contains patient allergens before prescribing  
**So that** I can avoid adverse reactions

**Acceptance Criteria:**
- Search medication by name
- Instant allergen match results
- Alternative medication suggestions
- Integration with e-prescribing system

#### US-3.2.2: Patient Allergy Review
**As a** emergency room doctor  
**I want to** quickly access a patient's comprehensive allergy profile  
**So that** I can make safe treatment decisions under time pressure

**Acceptance Criteria:**
- Access via patient consent/emergency override
- View complete allergy history
- Severity indicators clear
- Reaction history visible

#### US-3.2.3: Population Health Monitoring
**As a** clinic administrator  
**I want to** view allergen trends in my patient population  
**So that** I can stock appropriate alternatives and educate staff

**Acceptance Criteria:**
- Dashboard with top allergens
- Trend analysis over time
- Export reports
- Patient risk stratification

### 3.3 Pharmacist User Stories

#### US-3.3.1: Dispensing Safety Check
**As a** pharmacist  
**I want to** verify patient allergies before dispensing medication  
**So that** I can catch any prescribing errors

**Acceptance Criteria:**
- Integration with pharmacy management system
- Real-time allergy alerts
- Generic/brand name cross-reference
- Interaction checker

#### US-3.3.2: Patient Counseling
**As a** pharmacist  
**I want to** educate patients about allergen cross-reactivity  
**So that** they understand risks beyond direct allergens

**Acceptance Criteria:**
- Educational content library
- Printable patient handouts
- Visual cross-reactivity charts
- Multi-language support

### 3.4 Caregiver User Stories

#### US-3.4.1: Elderly Care
**As a** caregiver for elderly patients  
**I want to** scan all medications and topicals  
**So that** I can ensure safe products for patients with multiple allergies

**Acceptance Criteria:**
- Large text mode
- Voice-guided scanning
- Simplified interface
- Alert history for healthcare team

---

## 4. Technical Requirements

### 4.1 AWS Infrastructure

#### TR-4.1.1: AWS Bedrock
- **Requirement:** Use Claude 4.5 Sonnet for ingredient analysis
- **Configuration:** 
  - Model: claude-sonnet-4-5-20250929
  - Max tokens: 4000
  - Temperature: 0.1 (low for accuracy)
- **Usage:** Natural language ingredient matching, synonym detection, cross-reactivity analysis

#### TR-4.1.2: Amazon Textract
- **Requirement:** OCR service for ingredient label extraction
- **Configuration:**
  - DetectDocumentText API for simple labels
  - AnalyzeDocument API for complex layouts
  - Table extraction enabled
- **Usage:** Extract ingredient lists from product photos

#### TR-4.1.3: Amazon Comprehend Medical
- **Requirement:** Medical entity extraction from prescriptions
- **Configuration:**
  - DetectEntitiesV2 API
  - Entity types: MEDICATION, DOSAGE, ROUTE_OR_MODE
- **Usage:** Parse prescription information, medical terminology normalization

#### TR-4.1.4: AWS Lambda
- **Requirement:** Serverless compute for event-driven processing
- **Configuration:**
  - Runtime: Python 3.11, Node.js 18
  - Memory: 512MB - 3GB based on function
  - Timeout: 30 seconds max
  - Concurrent executions: 1000
- **Usage:** Barcode processing, image preprocessing, API orchestration

#### TR-4.1.5: Amazon DynamoDB
- **Requirement:** NoSQL database for user profiles and scan history
- **Configuration:**
  - On-demand billing mode
  - Point-in-time recovery enabled
  - Global tables for multi-region
  - Encryption at rest with KMS
- **Tables:**
  - Users
  - AllergyProfiles
  - ScanHistory
  - Sessions

#### TR-4.1.6: Amazon RDS (PostgreSQL)
- **Requirement:** Relational database for product catalog
- **Configuration:**
  - Instance: db.r6g.xlarge
  - Storage: 500GB SSD
  - Multi-AZ deployment
  - Automated backups (7-day retention)
  - Read replicas in 2 regions
- **Database:** Product ingredients, allergen synonyms, cross-reactivity data

#### TR-4.1.7: Amazon S3
- **Requirement:** Object storage for images and media
- **Configuration:**
  - Standard storage class
  - Lifecycle policies (transition to Glacier after 90 days)
  - Versioning enabled
  - Cross-region replication
- **Buckets:**
  - product-images
  - user-uploads
  - ml-models

#### TR-4.1.8: AWS API Gateway
- **Requirement:** RESTful API management
- **Configuration:**
  - Regional endpoint
  - Request throttling: 10,000 req/sec
  - API key authentication
  - CORS enabled
  - CloudWatch logging
- **APIs:** Mobile app API, Provider portal API, Integration API

#### TR-4.1.9: AWS Cognito
- **Requirement:** User authentication and authorization
- **Configuration:**
  - User pools for patients and providers
  - MFA enabled (optional for users, required for providers)
  - Password policy enforcement
  - OAuth 2.0 support
  - Social login (Google, Apple)

#### TR-4.1.10: AWS SNS/SES
- **Requirement:** Notification service
- **Configuration:**
  - SNS topics: allergen-alerts, system-notifications
  - SES verified domains
  - Email templates
  - SMS delivery via SNS
- **Usage:** Push notifications, email alerts, SMS alerts

#### TR-4.1.11: AWS CloudWatch
- **Requirement:** Monitoring and logging
- **Configuration:**
  - Log retention: 90 days
  - Custom metrics
  - Alarms for critical thresholds
  - Dashboards for KPIs
- **Metrics:** API latency, error rates, scan volume, accuracy

#### TR-4.1.12: AWS KMS
- **Requirement:** Encryption key management
- **Configuration:**
  - Customer-managed keys
  - Automatic key rotation
  - Key policies for least privilege
- **Usage:** Encrypt DynamoDB tables, RDS, S3 buckets

#### TR-4.1.13: AWS WAF
- **Requirement:** Web application firewall
- **Configuration:**
  - Rate limiting rules
  - IP blacklist/whitelist
  - SQL injection protection
  - XSS protection
- **Usage:** Protect API Gateway and CloudFront

#### TR-4.1.14: AWS CloudTrail
- **Requirement:** Audit logging
- **Configuration:**
  - All API calls logged
  - Log file integrity validation
  - S3 bucket with versioning
  - 7-year retention
- **Usage:** HIPAA compliance, security audits

### 4.2 Mobile Application

#### TR-4.2.1: Platform Support
- **iOS:** 13.0 and above
- **Android:** API 26 (Android 8.0) and above
- **Framework:** React Native 0.73+
- **Languages:** TypeScript

#### TR-4.2.2: Key Libraries
- react-native-camera: Product scanning
- react-native-vision-camera: Enhanced camera features
- react-native-permissions: Permission management
- react-native-push-notification: Local/remote notifications
- react-navigation: Navigation
- redux-toolkit: State management
- axios: HTTP client
- react-native-keychain: Secure storage

#### TR-4.2.3: Offline Capability
- AsyncStorage for local data
- Redux Persist for state persistence
- Local SQLite database for scan cache
- Queue system for offline scans

### 4.3 Web Portal

#### TR-4.3.1: Platform
- **Framework:** React 18+
- **Language:** TypeScript
- **Styling:** Tailwind CSS
- **State:** Redux Toolkit
- **Routing:** React Router v6
- **Forms:** React Hook Form
- **API:** Axios

#### TR-4.3.2: Browser Support
- Chrome 90+
- Firefox 88+
- Safari 14+
- Edge 90+

### 4.4 API Specifications

#### TR-4.4.1: RESTful Design
- Resource-based URLs
- HTTP methods: GET, POST, PUT, DELETE
- JSON request/response
- Versioning in URL (/api/v1/)
- HATEOAS principles

#### TR-4.4.2: Authentication
- JWT tokens (15-minute expiry)
- Refresh tokens (30-day expiry)
- API keys for provider integrations
- OAuth 2.0 for third-party integrations

#### TR-4.4.3: Rate Limiting
- 1000 requests/hour per user (mobile)
- 10,000 requests/hour per provider
- 429 status code for exceeded limits
- Retry-After header

### 4.5 Integration Requirements

#### TR-4.5.1: EHR Integration
- **Standard:** FHIR R4
- **Resources:** AllergyIntolerance, Patient, Medication, Observation
- **Authentication:** OAuth 2.0 with SMART on FHIR
- **Fallback:** HL7 v2.5.1 for legacy systems

#### TR-4.5.2: External APIs
- FDA Drug Database API
- OpenFoodFacts API
- NIH PubChem API
- USDA FoodData Central API

#### TR-4.5.3: Webhook Support
- Outbound webhooks for events
- Signature verification
- Retry logic (exponential backoff)
- Event types: allergen-detected, scan-completed, profile-updated

---

## 5. Data Requirements

### 5.1 Data Models

#### DM-5.1.1: User Profile
```json
{
  "userId": "uuid",
  "email": "string",
  "phoneNumber": "string",
  "firstName": "string",
  "lastName": "string",
  "dateOfBirth": "date",
  "emergencyContacts": [
    {
      "name": "string",
      "phone": "string",
      "relationship": "string"
    }
  ],
  "preferences": {
    "language": "string",
    "notificationEnabled": "boolean",
    "theme": "string"
  },
  "createdAt": "timestamp",
  "updatedAt": "timestamp"
}
```

#### DM-5.1.2: Allergy Profile
```json
{
  "allergyId": "uuid",
  "userId": "uuid",
  "allergen": "string",
  "allergenType": "food|medication|environmental|contact",
  "severity": "mild|moderate|severe|life-threatening",
  "reactions": ["string"],
  "onsetDate": "date",
  "verifiedBy": "self|doctor|lab-test",
  "verificationDate": "date",
  "notes": "string",
  "crossReactivities": ["string"],
  "createdAt": "timestamp",
  "updatedAt": "timestamp"
}
```

#### DM-5.1.3: Product
```json
{
  "productId": "uuid",
  "barcode": "string",
  "name": "string",
  "manufacturer": "string",
  "category": "medication|cosmetic|food|supplement",
  "subcategory": "string",
  "ingredients": [
    {
      "name": "string",
      "quantity": "string",
      "purpose": "string"
    }
  ],
  "images": ["url"],
  "fdaApprovalNumber": "string",
  "lastUpdated": "timestamp"
}
```

#### DM-5.1.4: Scan Record
```json
{
  "scanId": "uuid",
  "userId": "uuid",
  "productId": "uuid",
  "scanMethod": "barcode|ocr|manual",
  "timestamp": "timestamp",
  "result": "safe|allergen-detected|unknown",
  "allergensDetected": [
    {
      "allergen": "string",
      "confidence": "float",
      "severity": "string"
    }
  ],
  "imageUrl": "string",
  "location": {
    "latitude": "float",
    "longitude": "float"
  }
}
```

### 5.2 Data Privacy

#### DP-5.2.1: PHI Protection
- All PHI encrypted at rest and in transit
- Access logging for all PHI access
- De-identification for analytics
- Minimum necessary principle

#### DP-5.2.2: User Rights
- Right to access data
- Right to export data (FHIR format)
- Right to delete data
- Right to data portability

---

## 6. Quality Attributes

### 6.1 Accuracy Metrics

| Metric | Target | Measurement Method |
|--------|--------|-------------------|
| Allergen Detection Accuracy | > 95% | False positive/negative rate |
| OCR Accuracy | > 90% | Character-level accuracy |
| Barcode Scan Success | > 95% | First-scan success rate |
| Ingredient Match Accuracy | > 98% | Synonym validation |

### 6.2 Performance Metrics

| Metric | Target | Measurement Method |
|--------|--------|-------------------|
| Scan Analysis Time | < 3 sec | P95 latency |
| API Response Time | < 200ms | P99 latency |
| OCR Processing Time | < 5 sec | Average processing time |
| App Launch Time | < 2 sec | Time to interactive |

### 6.3 Reliability Metrics

| Metric | Target | Measurement Method |
|--------|--------|-------------------|
| System Uptime | 99.9% | Availability monitoring |
| API Error Rate | < 0.1% | Error rate percentage |
| Crash-free Sessions | > 99.5% | Mobile crash analytics |

---

## 7. Constraints

### 7.1 Technical Constraints
- Must use AWS cloud infrastructure
- Must support offline mode for critical features
- OCR limited by image quality
- AI accuracy depends on training data quality

### 7.2 Regulatory Constraints
- HIPAA compliance mandatory for US operations
- GDPR compliance for EU users
- FDA regulations for medical device classification (if applicable)
- State-specific medical practice laws

### 7.3 Business Constraints
- MVP budget: ₹90,00,000 (~$108,000)
- Launch timeline: 6 months
- Initial market: India, US
- Free tier limitations to encourage premium upgrades

### 7.4 Resource Constraints
- Development team: 8-10 people
- Medical advisory board required
- Legal/compliance team required
- Customer support infrastructure

---

## 8. Assumptions

1. Users have smartphones with cameras
2. Products have readable ingredient labels
3. Healthcare providers will adopt the platform
4. EHR systems support FHIR or HL7
5. Internet connectivity available for initial setup
6. Users provide accurate allergy information
7. Product databases remain accessible and up-to-date
8. AWS services maintain current pricing and availability
9. Medical advisory board available for validation
10. Regulatory approval obtained before launch

---

## 9. Dependencies

### 9.1 External Dependencies
- AWS service availability
- FDA database API uptime
- Third-party ingredient databases
- EHR vendor cooperation
- App store approval (Apple, Google)

### 9.2 Internal Dependencies
- Medical advisory board for allergen validation
- Legal team for compliance review
- Marketing for user acquisition
- Customer support for user assistance
- DevOps for infrastructure management

---

## 10. Success Metrics

### 10.1 Adoption Metrics
- 10,000 active users in first 3 months
- 100 healthcare providers onboarded in first 6 months
- 50,000 scans per month by month 6
- 4.5+ star rating in app stores

### 10.2 Impact Metrics
- 90% of users report increased confidence in product safety
- Zero severe allergic reactions from app-approved products
- 50% reduction in allergen-related ER visits for active users
- 80% user retention after 6 months

### 10.3 Business Metrics
- 10% conversion to premium tier
- 20 enterprise contracts in first year
- $500K ARR by end of year 1
- Break-even by month 18

### 10.4 Technical Metrics
- 99.9% uptime achieved
- < 0.1% error rate maintained
- 95% allergen detection accuracy validated
- < 3 second scan time for 95% of scans

---

## 11. Risks & Mitigation

### 11.1 Technical Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| AI false negatives (missed allergens) | Critical | Medium | Extensive testing, medical validation, conservative thresholds |
| OCR failure on poor images | High | High | Image quality validation, user feedback, manual entry fallback |
| AWS service outage | High | Low | Multi-region deployment, failover mechanisms |
| Database performance degradation | Medium | Medium | Query optimization, caching, read replicas |

### 11.2 Regulatory Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| HIPAA non-compliance | Critical | Low | Regular audits, compliance automation |
| FDA classification as medical device | High | Medium | Legal consultation, compliance strategy |
| State medical practice violations | High | Low | Legal review, provider credential verification |

### 11.3 Business Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Low user adoption | Critical | Medium | Marketing campaign, provider partnerships |
| Liability from incorrect alerts | Critical | Medium | Insurance, ToS disclaimers, accuracy validation |
| Competition from established players | High | High | Unique AI features, superior UX, provider relationships |

---

## 12. Glossary

- **PHI:** Protected Health Information
- **FHIR:** Fast Healthcare Interoperability Resources
- **HL7:** Health Level 7 (healthcare data standard)
- **OCR:** Optical Character Recognition
- **NPI:** National Provider Identifier
- **HIPAA:** Health Insurance Portability and Accountability Act
- **GDPR:** General Data Protection Regulation
- **RTO:** Recovery Time Objective
- **RPO:** Recovery Point Objective
- **WCAG:** Web Content Accessibility Guidelines
- **MFA:** Multi-Factor Authentication
- **RBAC:** Role-Based Access Control
- **JWT:** JSON Web Token
- **CAS Number:** Chemical Abstracts Service Registry Number

---

## Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | Feb 2026 | Product Team | Initial requirements document |

---

**Document Status:** Draft for Review  
**Next Review Date:** March 2026  
**Approval Required From:** Product Owner, Technical Lead, Medical Advisory Board, Legal Team
