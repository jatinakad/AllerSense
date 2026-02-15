# Allersense - System Design Document

## Document Information

**Project Name:** Allersense  
**Version:** 1.0  
**Date:** February 2026  
**Status:** Design Specification  
**Classification:** Internal

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [System Architecture](#2-system-architecture)
3. [Component Design](#3-component-design)
4. [Data Architecture](#4-data-architecture)
5. [API Design](#5-api-design)
6. [User Interface Design](#6-user-interface-design)
7. [AI/ML Design](#7-aiml-design)
8. [Security Design](#8-security-design)
9. [Integration Design](#9-integration-design)
10. [Deployment Architecture](#10-deployment-architecture)
11. [Design Decisions](#11-design-decisions)

---

## 1. Executive Summary

### 1.1 Purpose
This document describes the system design for Allersense, an AI-powered allergen detection platform that prevents allergic reactions by scanning medications, topical products, and cosmetics for hidden allergens.

### 1.2 Scope
The design covers the complete system architecture including mobile applications, web portal, backend services, AI/ML components, data storage, integrations, and security mechanisms.

### 1.3 Target Audience
- Software Engineers
- System Architects
- DevOps Engineers
- QA Engineers
- Product Managers
- Medical Advisory Board

### 1.4 Design Principles

1. **Safety First:** Zero tolerance for false negatives in allergen detection
2. **User Privacy:** HIPAA-compliant data handling with encryption
3. **Performance:** Sub-3-second scan results for 95% of requests
4. **Scalability:** Support 100,000+ concurrent users
5. **Reliability:** 99.9% uptime with multi-region redundancy
6. **Accessibility:** WCAG 2.1 AA compliance
7. **Maintainability:** Microservices architecture with clear separation of concerns

---

## 2. System Architecture

### 2.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                         CLIENT LAYER                                 │
├────────────────────────────────┬────────────────────────────────────┤
│   Mobile App                   │   Web Portal                       │
│   (React Native)               │   (React.js)                       │
│   • iOS / Android              │   • Provider Dashboard             │
│   • Offline-first              │   • Patient Management             │
│   • Camera Integration         │   • Analytics & Reports            │
└───────────────┬────────────────┴───────────┬────────────────────────┘
                │                            │
                └────────────┬───────────────┘
                             │
                    ┌────────▼────────┐
                    │  Amazon         │
                    │  CloudFront     │
                    │  (CDN)          │
                    └────────┬────────┘
                             │
┌────────────────────────────▼────────────────────────────────────────┐
│                      API GATEWAY LAYER                               │
├──────────────────────────────────────────────────────────────────────┤
│   AWS API Gateway                                                    │
│   • REST API Endpoints                                               │
│   • WebSocket for Real-time                                          │
│   • Request Validation                                               │
│   • Rate Limiting                                                    │
│   • CORS Configuration                                               │
└────────────────────┬─────────────────────────────────────────────────┘
                     │
    ┌────────────────┼────────────────┬────────────────┐
    │                │                │                │
    ▼                ▼                ▼                ▼
┌─────────┐    ┌──────────┐    ┌─────────┐    ┌──────────┐
│ Scanning│    │ Allergen │    │ Profile │    │Notification│
│ Service │    │ Service  │    │ Service │    │  Service   │
└────┬────┘    └────┬─────┘    └────┬────┘    └─────┬──────┘
     │              │               │               │
     │         ┌────▼───────────────▼───────┐       │
     │         │   AI/ML Layer              │       │
     │         │   • AWS Bedrock (Claude)   │       │
     │         │   • Textract (OCR)         │       │
     │         │   • Comprehend Medical     │       │
     │         └────────────────────────────┘       │
     │                                              │
     └──────────────────┬───────────────────────────┘
                        │
┌───────────────────────▼──────────────────────────────────────────────┐
│                        DATA LAYER                                     │
├───────────────┬──────────────────┬────────────────┬──────────────────┤
│  DynamoDB     │  RDS PostgreSQL  │  Amazon S3     │  ElastiCache     │
│  • Users      │  • Products      │  • Images      │  • Sessions      │
│  • Allergies  │  • Ingredients   │  • Uploads     │  • API Cache     │
│  • Scans      │  • Synonyms      │  • Backups     │  • Query Cache   │
└───────────────┴──────────────────┴────────────────┴──────────────────┘
                        │
┌───────────────────────▼──────────────────────────────────────────────┐
│                   INTEGRATION LAYER                                   │
├───────────────────────────────────────────────────────────────────────┤
│  • EHR Systems (FHIR/HL7)                                             │
│  • FDA Database API                                                   │
│  • OpenFoodFacts API                                                  │
│  • Third-party Product Databases                                      │
└───────────────────────────────────────────────────────────────────────┘
```

### 2.2 Architecture Patterns

#### 2.2.1 Microservices Architecture
- **Services:** Scanning, Allergen Detection, Profile Management, Notification
- **Communication:** Synchronous (REST) and Asynchronous (SNS/SQS)
- **Benefits:** Independent scaling, fault isolation, technology flexibility

#### 2.2.2 Event-Driven Architecture
- **Events:** ScanCompleted, AllergenDetected, ProfileUpdated
- **Message Bus:** Amazon SNS + SQS
- **Benefits:** Loose coupling, scalability, real-time processing

#### 2.2.3 CQRS Pattern (Command Query Responsibility Segregation)
- **Write Model:** DynamoDB for fast writes (scans, profiles)
- **Read Model:** ElastiCache for fast reads (product lookups)
- **Sync:** Event-driven synchronization between models

#### 2.2.4 API Gateway Pattern
- **Single Entry Point:** AWS API Gateway
- **Benefits:** Unified authentication, rate limiting, monitoring
- **Features:** Request transformation, response caching, throttling

---

## 3. Component Design

### 3.1 Mobile Application

#### 3.1.1 Architecture Pattern
- **Pattern:** Clean Architecture with Redux
- **Layers:** Presentation → Domain → Data
- **State Management:** Redux Toolkit with Redux Persist

#### 3.1.2 Component Structure
```
src/
├── app/
│   ├── store/              # Redux store configuration
│   └── navigation/         # React Navigation setup
├── features/
│   ├── scanning/
│   │   ├── components/     # Scan UI components
│   │   ├── screens/        # Scan screen
│   │   ├── hooks/          # Custom hooks
│   │   └── slice.ts        # Redux slice
│   ├── profile/
│   ├── history/
│   └── alerts/
├── shared/
│   ├── components/         # Reusable UI components
│   ├── hooks/              # Common hooks
│   └── utils/              # Utilities
├── services/
│   ├── api/                # API client
│   ├── camera/             # Camera service
│   ├── storage/            # Local storage
│   └── notifications/      # Push notifications
└── types/                  # TypeScript types
```

#### 3.1.3 Key Components

**ScanningModule**
```typescript
interface ScanningModule {
  camera: CameraService;
  barcodeScanner: BarcodeScanner;
  imageProcessor: ImageProcessor;
  apiClient: ApiClient;
  
  scanBarcode(): Promise<ScanResult>;
  captureLabel(): Promise<Image>;
  analyzeScan(scan: ScanData): Promise<AnalysisResult>;
}
```

**ProfileModule**
```typescript
interface ProfileModule {
  allergyManager: AllergyManager;
  profileStorage: SecureStorage;
  syncService: SyncService;
  
  addAllergy(allergy: Allergy): Promise<void>;
  updateProfile(profile: UserProfile): Promise<void>;
  syncWithCloud(): Promise<void>;
}
```

**NotificationModule**
```typescript
interface NotificationModule {
  pushService: PushNotificationService;
  localNotifier: LocalNotifier;
  alertManager: AlertManager;
  
  showAllergenAlert(allergens: Allergen[]): void;
  scheduleScanReminder(time: Date): void;
  handleNotification(notification: Notification): void;
}
```

#### 3.1.4 Offline Strategy
- **Priority 1:** Cache last 100 scanned products
- **Priority 2:** Queue scans for when online
- **Priority 3:** Local allergy profile always available
- **Sync:** Background sync when connectivity restored

### 3.2 Web Portal

#### 3.2.1 Architecture
- **Framework:** React 18 with TypeScript
- **State:** Redux Toolkit + RTK Query
- **Routing:** React Router v6
- **Styling:** Tailwind CSS + HeadlessUI

#### 3.2.2 Module Structure
```
src/
├── app/
│   ├── store/
│   ├── router/
│   └── App.tsx
├── features/
│   ├── dashboard/
│   ├── patients/
│   ├── products/
│   └── analytics/
├── components/
│   ├── layout/
│   ├── forms/
│   └── charts/
├── services/
│   ├── api/
│   └── auth/
└── types/
```

#### 3.2.3 Key Pages

**Provider Dashboard**
- Real-time patient activity feed
- Recent allergen alerts
- Patient risk summary
- Quick product check

**Patient Management**
- Patient search and filter
- Allergy profile viewer
- Scan history
- Intervention tracking

**Product Checker**
- Product search interface
- Ingredient analysis
- Alternative suggestions
- Prescription integration

**Analytics Dashboard**
- Population health metrics
- Common allergens chart
- Scan volume trends
- Intervention success rate

### 3.3 Backend Services

#### 3.3.1 Scanning Service

```python
# Lambda Function: scan-processor
# Runtime: Python 3.11
# Memory: 1024 MB
# Timeout: 30 seconds

import boto3
import json
from typing import Dict, Any

class ScanProcessor:
    def __init__(self):
        self.textract = boto3.client('textract')
        self.s3 = boto3.client('s3')
        self.dynamodb = boto3.resource('dynamodb')
        
    def process_barcode(self, barcode: str) -> Dict[str, Any]:
        """Look up product by barcode"""
        # Query RDS for product details
        product = self.get_product_by_barcode(barcode)
        return {
            'productId': product.id,
            'name': product.name,
            'ingredients': product.ingredients
        }
    
    def process_image(self, image_key: str) -> Dict[str, Any]:
        """Extract text from image using Textract"""
        response = self.textract.detect_document_text(
            Document={'S3Object': {
                'Bucket': 'Allersense-uploads',
                'Name': image_key
            }}
        )
        
        # Extract ingredient text
        text = self.extract_ingredients(response)
        
        return {
            'rawText': text,
            'confidence': self.calculate_confidence(response)
        }
    
    def extract_ingredients(self, textract_response: Dict) -> str:
        """Parse Textract response to extract ingredient list"""
        blocks = textract_response['Blocks']
        lines = [b['Text'] for b in blocks if b['BlockType'] == 'LINE']
        
        # Find "Ingredients:" section
        ingredients_start = next(
            (i for i, line in enumerate(lines) 
             if 'ingredient' in line.lower()), 
            None
        )
        
        if ingredients_start is None:
            return ' '.join(lines)
        
        return ' '.join(lines[ingredients_start+1:])

def lambda_handler(event, context):
    processor = ScanProcessor()
    
    scan_type = event['scanType']  # 'barcode' or 'image'
    
    if scan_type == 'barcode':
        result = processor.process_barcode(event['barcode'])
    else:
        result = processor.process_image(event['imageKey'])
    
    return {
        'statusCode': 200,
        'body': json.dumps(result)
    }
```

#### 3.3.2 Allergen Detection Service

```python
# Lambda Function: allergen-detector
# Runtime: Python 3.11
# Memory: 2048 MB
# Timeout: 60 seconds

import boto3
import json
from typing import List, Dict, Any

class AllergenDetector:
    def __init__(self):
        self.bedrock = boto3.client('bedrock-runtime')
        self.dynamodb = boto3.resource('dynamodb')
        self.rds = self.get_rds_connection()
        
    def detect_allergens(
        self, 
        ingredients: List[str], 
        user_allergies: List[Dict]
    ) -> Dict[str, Any]:
        """
        Use Claude to detect allergens in ingredient list
        """
        
        # Build prompt for Claude
        prompt = self.build_detection_prompt(ingredients, user_allergies)
        
        # Call Bedrock
        response = self.bedrock.invoke_model(
            modelId='anthropic.claude-sonnet-4-5-20250929',
            contentType='application/json',
            accept='application/json',
            body=json.dumps({
                'anthropic_version': 'bedrock-2023-05-31',
                'max_tokens': 4000,
                'temperature': 0.1,
                'messages': [{
                    'role': 'user',
                    'content': prompt
                }]
            })
        )
        
        # Parse response
        result = json.loads(response['body'].read())
        analysis = json.loads(result['content'][0]['text'])
        
        # Validate against synonym database
        validated = self.validate_with_database(analysis)
        
        # Check cross-reactivity
        cross_reactive = self.check_cross_reactivity(
            validated['detected_allergens'],
            user_allergies
        )
        
        return {
            'allergens_detected': validated['detected_allergens'],
            'cross_reactive_allergens': cross_reactive,
            'safe_to_use': len(validated['detected_allergens']) == 0,
            'confidence': validated['confidence'],
            'analysis_details': analysis
        }
    
    def build_detection_prompt(
        self, 
        ingredients: List[str], 
        allergies: List[Dict]
    ) -> str:
        """Build Claude prompt for allergen detection"""
        
        allergy_list = '\n'.join([
            f"- {a['allergen']} (severity: {a['severity']})" 
            for a in allergies
        ])
        
        ingredient_list = '\n'.join([f"- {ing}" for ing in ingredients])
        
        return f"""You are an expert allergen detection system. Analyze the following product ingredients against the user's known allergies.

USER ALLERGIES:
{allergy_list}

PRODUCT INGREDIENTS:
{ingredient_list}

INSTRUCTIONS:
1. Check each ingredient against the allergy list
2. Consider botanical names (e.g., "Prunus amygdalus" = almond)
3. Consider chemical names and CAS numbers
4. Consider ingredient synonyms across languages
5. Look for hidden allergen sources

Respond ONLY with a JSON object in this exact format:
{{
  "detected_allergens": [
    {{
      "ingredient_name": "exact ingredient name from list",
      "matches_allergy": "allergy name",
      "match_type": "direct|botanical|chemical|synonym",
      "confidence": 0.0-1.0,
      "explanation": "brief explanation"
    }}
  ],
  "overall_confidence": 0.0-1.0,
  "requires_human_review": true|false
}}

Be extremely conservative - if there's any doubt, flag it.
"""
    
    def validate_with_database(self, ai_analysis: Dict) -> Dict:
        """Cross-reference AI results with synonym database"""
        
        validated_allergens = []
        
        for allergen in ai_analysis['detected_allergens']:
            # Query RDS for synonyms
            synonyms = self.get_allergen_synonyms(
                allergen['matches_allergy']
            )
            
            # Check if ingredient matches any known synonym
            if self.ingredient_in_synonyms(
                allergen['ingredient_name'], 
                synonyms
            ):
                validated_allergens.append({
                    **allergen,
                    'database_validated': True
                })
            else:
                # Lower confidence if not in database
                validated_allergens.append({
                    **allergen,
                    'database_validated': False,
                    'confidence': allergen['confidence'] * 0.8
                })
        
        return {
            'detected_allergens': validated_allergens,
            'confidence': ai_analysis['overall_confidence']
        }
    
    def check_cross_reactivity(
        self, 
        detected: List[Dict], 
        allergies: List[Dict]
    ) -> List[Dict]:
        """Check for cross-reactive allergens"""
        
        cross_reactive = []
        
        for allergy in allergies:
            # Query cross-reactivity table
            related = self.get_cross_reactive_allergens(
                allergy['allergen']
            )
            
            # Check if any related allergens are in detected list
            for allergen in detected:
                if allergen['matches_allergy'] in related:
                    cross_reactive.append({
                        'primary_allergy': allergy['allergen'],
                        'cross_reactive_ingredient': allergen['ingredient_name'],
                        'severity': related[allergen['matches_allergy']]['severity']
                    })
        
        return cross_reactive

def lambda_handler(event, context):
    detector = AllergenDetector()
    
    ingredients = event['ingredients']
    user_id = event['userId']
    
    # Get user allergies from DynamoDB
    user_allergies = detector.get_user_allergies(user_id)
    
    # Detect allergens
    result = detector.detect_allergens(ingredients, user_allergies)
    
    # Store scan result
    detector.store_scan_result(user_id, event['scanId'], result)
    
    # Send alert if allergens detected
    if not result['safe_to_use']:
        detector.send_allergen_alert(user_id, result)
    
    return {
        'statusCode': 200,
        'body': json.dumps(result)
    }
```

#### 3.3.3 Profile Service

```typescript
// Lambda Function: profile-manager
// Runtime: Node.js 18
// Memory: 512 MB

import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { DynamoDBDocumentClient, PutCommand, GetCommand } from "@aws-sdk/lib-dynamodb";

const client = new DynamoDBClient({});
const docClient = DynamoDBDocumentClient.from(client);

interface AllergyProfile {
  userId: string;
  allergyId: string;
  allergen: string;
  severity: 'mild' | 'moderate' | 'severe' | 'life-threatening';
  reactions: string[];
  verifiedBy: 'self' | 'doctor' | 'lab-test';
  createdAt: string;
  updatedAt: string;
}

export const handler = async (event: any) => {
  const action = event.action;
  
  switch (action) {
    case 'createProfile':
      return await createProfile(event.profile);
    case 'addAllergy':
      return await addAllergy(event.userId, event.allergy);
    case 'getProfile':
      return await getProfile(event.userId);
    default:
      return { statusCode: 400, body: 'Invalid action' };
  }
};

async function createProfile(profile: any) {
  const command = new PutCommand({
    TableName: 'Users',
    Item: {
      userId: profile.userId,
      email: profile.email,
      firstName: profile.firstName,
      lastName: profile.lastName,
      createdAt: new Date().toISOString(),
      updatedAt: new Date().toISOString()
    }
  });
  
  await docClient.send(command);
  
  return {
    statusCode: 200,
    body: JSON.stringify({ userId: profile.userId })
  };
}

async function addAllergy(userId: string, allergy: Partial<AllergyProfile>) {
  const allergyId = generateId();
  
  const command = new PutCommand({
    TableName: 'AllergyProfiles',
    Item: {
      userId,
      allergyId,
      ...allergy,
      createdAt: new Date().toISOString(),
      updatedAt: new Date().toISOString()
    }
  });
  
  await docClient.send(command);
  
  // Trigger sync with EHR if integrated
  await syncWithEHR(userId, allergyId);
  
  return {
    statusCode: 200,
    body: JSON.stringify({ allergyId })
  };
}

async function getProfile(userId: string) {
  const command = new GetCommand({
    TableName: 'Users',
    Key: { userId }
  });
  
  const result = await docClient.send(command);
  
  return {
    statusCode: 200,
    body: JSON.stringify(result.Item)
  };
}

function generateId(): string {
  return `${Date.now()}-${Math.random().toString(36).substring(7)}`;
}

async function syncWithEHR(userId: string, allergyId: string) {
  // Implementation for EHR sync
  // Uses FHIR API to push allergy data
}
```

#### 3.3.4 Notification Service

```python
# Lambda Function: notification-sender
# Runtime: Python 3.11
# Memory: 512 MB

import boto3
import json
from typing import Dict, List

class NotificationService:
    def __init__(self):
        self.sns = boto3.client('sns')
        self.ses = boto3.client('ses')
        self.dynamodb = boto3.resource('dynamodb')
        
    def send_allergen_alert(
        self, 
        user_id: str, 
        allergens: List[Dict],
        product_name: str
    ):
        """Send multi-channel allergen alert"""
        
        user = self.get_user(user_id)
        
        # Send push notification
        if user.get('pushToken'):
            self.send_push_notification(
                user['pushToken'],
                title="⚠️ ALLERGEN DETECTED",
                body=f"Product '{product_name}' contains allergens!",
                data={'allergens': allergens, 'productName': product_name}
            )
        
        # Send email
        if user.get('email') and user.get('emailNotifications', True):
            self.send_email_alert(
                user['email'],
                allergens,
                product_name
            )
        
        # Send SMS if severe allergen
        severe_allergens = [
            a for a in allergens 
            if a.get('severity') in ['severe', 'life-threatening']
        ]
        
        if severe_allergens and user.get('phone'):
            self.send_sms_alert(
                user['phone'],
                severe_allergens,
                product_name
            )
        
        # Notify emergency contacts if life-threatening
        life_threatening = [
            a for a in allergens 
            if a.get('severity') == 'life-threatening'
        ]
        
        if life_threatening and user.get('emergencyContacts'):
            self.notify_emergency_contacts(
                user['emergencyContacts'],
                life_threatening,
                product_name
            )
    
    def send_push_notification(
        self, 
        token: str, 
        title: str, 
        body: str, 
        data: Dict
    ):
        """Send FCM/APNS push notification via SNS"""
        
        message = {
            'default': body,
            'GCM': json.dumps({
                'notification': {
                    'title': title,
                    'body': body,
                    'sound': 'default',
                    'priority': 'high'
                },
                'data': data
            }),
            'APNS': json.dumps({
                'aps': {
                    'alert': {
                        'title': title,
                        'body': body
                    },
                    'sound': 'default',
                    'badge': 1
                },
                'data': data
            })
        }
        
        self.sns.publish(
            TargetArn=token,
            Message=json.dumps(message),
            MessageStructure='json'
        )
    
    def send_email_alert(
        self, 
        email: str, 
        allergens: List[Dict],
        product_name: str
    ):
        """Send formatted email alert"""
        
        allergen_list = '\n'.join([
            f"• {a['allergen']} (Severity: {a['severity']})"
            for a in allergens
        ])
        
        html_body = f"""
        <html>
          <head></head>
          <body>
            <h2 style="color: #DC2626;">⚠️ Allergen Alert</h2>
            <p>Allersense has detected allergens in the product you scanned:</p>
            <p><strong>Product:</strong> {product_name}</p>
            <p><strong>Detected Allergens:</strong></p>
            <ul style="color: #DC2626;">
              {allergen_list}
            </ul>
            <p><strong>Action Required:</strong> Do NOT use this product.</p>
            <p>Stay safe,<br>The Allersense Team</p>
          </body>
        </html>
        """
        
        self.ses.send_email(
            Source='alerts@Allersense.com',
            Destination={'ToAddresses': [email]},
            Message={
                'Subject': {'Data': '⚠️ ALLERGEN ALERT - Do Not Use Product'},
                'Body': {'Html': {'Data': html_body}}
            }
        )
    
    def send_sms_alert(
        self, 
        phone: str, 
        allergens: List[Dict],
        product_name: str
    ):
        """Send SMS for severe allergens"""
        
        message = (
            f"⚠️ ALLERGEN ALERT from Allersense\n"
            f"Product: {product_name}\n"
            f"DO NOT USE - Contains: {', '.join([a['allergen'] for a in allergens])}"
        )
        
        self.sns.publish(
            PhoneNumber=phone,
            Message=message
        )

def lambda_handler(event, context):
    service = NotificationService()
    
    notification_type = event['type']
    
    if notification_type == 'allergen_alert':
        service.send_allergen_alert(
            event['userId'],
            event['allergens'],
            event['productName']
        )
    
    return {'statusCode': 200}
```

---

## 4. Data Architecture

### 4.1 Database Design

#### 4.1.1 DynamoDB Tables

**Users Table**
```
Partition Key: userId (String)

Attributes:
- email (String)
- phoneNumber (String)
- firstName (String)
- lastName (String)
- dateOfBirth (String - ISO 8601)
- emergencyContacts (List)
- preferences (Map)
- createdAt (String - ISO 8601)
- updatedAt (String - ISO 8601)

GSI-1:
- Partition Key: email
- Purpose: Login by email

GSI-2:
- Partition Key: phoneNumber
- Purpose: Login by phone
```

**AllergyProfiles Table**
```
Partition Key: userId (String)
Sort Key: allergyId (String)

Attributes:
- allergen (String)
- allergenType (String) # food, medication, environmental, contact
- severity (String) # mild, moderate, severe, life-threatening
- reactions (List<String>)
- onsetDate (String)
- verifiedBy (String) # self, doctor, lab-test
- verificationDate (String)
- notes (String)
- createdAt (String)
- updatedAt (String)

LSI-1:
- Sort Key: severity
- Purpose: Get high-severity allergies first
```

**ScanHistory Table**
```
Partition Key: userId (String)
Sort Key: timestamp (String - ISO 8601)

Attributes:
- scanId (String)
- productId (String)
- productName (String)
- scanMethod (String) # barcode, ocr, manual
- result (String) # safe, allergen-detected, unknown
- allergensDetected (List)
- confidence (Number)
- imageUrl (String)
- location (Map) # {latitude, longitude}

GSI-1:
- Partition Key: scanId
- Purpose: Look up specific scan

TTL Attribute: expiresAt (Number)
- Automatically delete scans older than 2 years
```

**Sessions Table**
```
Partition Key: sessionId (String)

Attributes:
- userId (String)
- deviceId (String)
- deviceType (String) # ios, android, web
- ipAddress (String)
- userAgent (String)
- createdAt (String)
- lastActivity (String)
- expiresAt (Number) # TTL

TTL Attribute: expiresAt
- Automatically delete expired sessions
```

#### 4.1.2 RDS PostgreSQL Schema

```sql
-- Products Table
CREATE TABLE products (
    product_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    barcode VARCHAR(20) UNIQUE,
    name VARCHAR(255) NOT NULL,
    manufacturer VARCHAR(255),
    category VARCHAR(100) NOT NULL, -- medication, cosmetic, food, supplement
    subcategory VARCHAR(100),
    fda_approval_number VARCHAR(50),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Indexes
    INDEX idx_barcode (barcode),
    INDEX idx_category (category),
    INDEX idx_manufacturer (manufacturer)
);

-- Ingredients Table
CREATE TABLE ingredients (
    ingredient_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL UNIQUE,
    botanical_name VARCHAR(255),
    chemical_name VARCHAR(255),
    cas_number VARCHAR(20),
    allergen_category VARCHAR(100), -- nut, dairy, gluten, etc.
    is_common_allergen BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Indexes
    INDEX idx_name (name),
    INDEX idx_botanical (botanical_name),
    INDEX idx_cas (cas_number),
    INDEX idx_allergen_category (allergen_category)
);

-- Product Ingredients Junction Table
CREATE TABLE product_ingredients (
    product_id UUID REFERENCES products(product_id) ON DELETE CASCADE,
    ingredient_id UUID REFERENCES ingredients(ingredient_id),
    quantity VARCHAR(50),
    purpose VARCHAR(100), -- active, inactive, preservative, fragrance
    position INT, -- order in ingredient list
    
    PRIMARY KEY (product_id, ingredient_id),
    
    -- Indexes
    INDEX idx_product (product_id),
    INDEX idx_ingredient (ingredient_id)
);

-- Ingredient Synonyms Table
CREATE TABLE ingredient_synonyms (
    synonym_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    ingredient_id UUID REFERENCES ingredients(ingredient_id),
    synonym VARCHAR(255) NOT NULL,
    language VARCHAR(10) DEFAULT 'en',
    synonym_type VARCHAR(50), -- botanical, chemical, brand, translation
    
    -- Indexes
    INDEX idx_synonym (synonym),
    INDEX idx_ingredient (ingredient_id),
    INDEX idx_language (language)
);

-- Cross-Reactivity Table
CREATE TABLE cross_reactivity (
    cross_reactivity_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    primary_allergen VARCHAR(255) NOT NULL,
    cross_reactive_allergen VARCHAR(255) NOT NULL,
    severity VARCHAR(50) NOT NULL, -- low, medium, high
    scientific_basis TEXT,
    reference_url VARCHAR(500),
    
    -- Indexes
    INDEX idx_primary (primary_allergen),
    INDEX idx_cross_reactive (cross_reactive_allergen),
    
    -- Constraint
    UNIQUE (primary_allergen, cross_reactive_allergen)
);

-- Allergen Categories Table
CREATE TABLE allergen_categories (
    category_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL UNIQUE,
    description TEXT,
    common_reactions TEXT[],
    emergency_instructions TEXT
);

-- Product Images Table
CREATE TABLE product_images (
    image_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    product_id UUID REFERENCES products(product_id) ON DELETE CASCADE,
    s3_key VARCHAR(500) NOT NULL,
    image_type VARCHAR(50), -- front, back, ingredients, nutrition
    uploaded_by VARCHAR(50), -- manufacturer, user, admin
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Indexes
    INDEX idx_product (product_id),
    INDEX idx_type (image_type)
);

-- User Contributions Table (for community-submitted data)
CREATE TABLE user_contributions (
    contribution_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id VARCHAR(100) NOT NULL,
    contribution_type VARCHAR(50), -- product, ingredient, image
    product_id UUID REFERENCES products(product_id),
    status VARCHAR(50) DEFAULT 'pending', -- pending, approved, rejected
    data JSONB,
    reviewed_by VARCHAR(100),
    reviewed_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Indexes
    INDEX idx_user (user_id),
    INDEX idx_status (status),
    INDEX idx_type (contribution_type)
);

-- Create full-text search indexes
CREATE INDEX idx_product_name_fts ON products USING gin(to_tsvector('english', name));
CREATE INDEX idx_ingredient_name_fts ON ingredients USING gin(to_tsvector('english', name));

-- Create triggers for updated_at
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ language 'plpgsql';

CREATE TRIGGER update_products_updated_at BEFORE UPDATE ON products
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

### 4.2 Data Flow Diagrams

#### 4.2.1 Product Scan Flow
```
┌──────────┐
│  User    │
│  Scans   │
│  Product │
└────┬─────┘
     │
     ▼
┌─────────────────┐
│ 1. Upload Image │──────┐
│    to S3        │      │
└─────────────────┘      │
                         │
                    ┌────▼────┐
                    │   SNS   │ Trigger Event
                    └────┬────┘
                         │
     ┌───────────────────┼───────────────────┐
     │                   │                   │
     ▼                   ▼                   ▼
┌─────────┐      ┌──────────────┐    ┌────────────┐
│Textract │      │  Check if    │    │  Store     │
│Extract  │      │  Barcode in  │    │  Scan      │
│  Text   │      │  Database    │    │  Metadata  │
└────┬────┘      └──────┬───────┘    │(DynamoDB)  │
     │                  │             └────────────┘
     │                  │
     └──────────┬───────┘
                │
                ▼
        ┌───────────────┐
        │ Get User      │
        │ Allergy       │
        │ Profile       │
        │ (DynamoDB)    │
        └───────┬───────┘
                │
                ▼
        ┌───────────────┐
        │ Bedrock       │
        │ (Claude)      │
        │ Analyze       │
        │ Ingredients   │
        └───────┬───────┘
                │
                ▼
        ┌───────────────┐
        │ Validate      │
        │ Against       │
        │ Synonym DB    │
        │ (RDS)         │
        └───────┬───────┘
                │
          ┌─────┴─────┐
          │           │
          ▼           ▼
    ┌─────────┐  ┌─────────┐
    │ Safe    │  │ Allergen│
    │ Product │  │ Detected│
    └────┬────┘  └────┬────┘
         │            │
         │            ▼
         │      ┌──────────┐
         │      │Send Alert│
         │      │(SNS/SES) │
         │      └──────────┘
         │
         └──────────┬─────────┘
                    │
                    ▼
            ┌───────────────┐
            │ Return Result │
            │ to User       │
            └───────────────┘
```

#### 4.2.2 EHR Sync Flow
```
┌──────────────┐                    ┌──────────┐
│ Allersense │◄───────────────────┤   EHR    │
│   Creates    │   1. Request       │  System  │
│   Allergy    │      Patient       │          │
│   Record     │      Allergy       │          │
└──────┬───────┘      Data          └──────────┘
       │
       │ 2. Format as
       │    FHIR Resource
       │
       ▼
┌────────────────────┐
│ FHIR AllergyInto   │
│ Resource Created   │
│ {                  │
│   "resourceType":  │
│   "AllergyInto",   │
│   "patient": {...},│
│   "substance": {...}│
│ }                  │
└──────┬─────────────┘
       │
       │ 3. POST to
       │    EHR FHIR
       │    Endpoint
       │
       ▼
┌──────────────┐
│   EHR FHIR   │───────┐
│   Server     │       │ 4. Validate
└──────┬───────┘       │    & Store
       │               │
       │ 5. Return     │
       │    Resource   │
       │    Location   │
       │               │
       ▼               │
┌──────────────┐       │
│ Update       │◄──────┘
│ Allersense │
│ with EHR     │
│ Resource ID  │
└──────────────┘
```

### 4.3 Caching Strategy

#### 4.3.1 ElastiCache Redis Configuration
```
Cluster Configuration:
- Node Type: cache.r6g.large
- Nodes: 3 (1 primary, 2 replicas)
- Multi-AZ: Enabled
- Automatic Failover: Enabled
```

**Cache Keys:**
```
# Product cache (24 hours)
product:{barcode} → Product JSON

# User allergy profile (1 hour)
allergies:{userId} → List of allergies

# Scan results (15 minutes)
scan:{scanId} → Scan result JSON

# API responses (5 minutes)
api:product-search:{query} → Search results

# Ingredient synonyms (7 days)
synonym:{ingredientName} → List of synonyms

# Cross-reactivity (7 days)
cross:{allergen} → Related allergens
```

**Cache Invalidation:**
- User updates profile → Invalidate `allergies:{userId}`
- Product updated → Invalidate `product:{barcode}`
- Scan completed → Set scan cache with 15-min TTL
- Database update → Pub/Sub to invalidate related caches

---

## 5. API Design

### 5.1 REST API Specification

**Base URL:** `https://api.Allersense.com/v1`

#### 5.1.1 Authentication Endpoints

```
POST /auth/register
Request:
{
  "email": "user@example.com",
  "password": "SecurePass123!",
  "firstName": "John",
  "lastName": "Doe"
}

Response: 201 Created
{
  "userId": "uuid",
  "accessToken": "jwt-token",
  "refreshToken": "refresh-token",
  "expiresIn": 900
}
```

```
POST /auth/login
Request:
{
  "email": "user@example.com",
  "password": "SecurePass123!"
}

Response: 200 OK
{
  "accessToken": "jwt-token",
  "refreshToken": "refresh-token",
  "expiresIn": 900,
  "user": {
    "userId": "uuid",
    "email": "user@example.com",
    "firstName": "John"
  }
}
```

#### 5.1.2 Scanning Endpoints

```
POST /scans
Request:
{
  "scanMethod": "barcode",
  "barcode": "012345678901"
}
OR
{
  "scanMethod": "image",
  "imageData": "base64-encoded-image"
}

Response: 202 Accepted
{
  "scanId": "uuid",
  "status": "processing",
  "estimatedTime": 3
}
```

```
GET /scans/{scanId}
Response: 200 OK
{
  "scanId": "uuid",
  "status": "completed",
  "product": {
    "name": "Example Lotion",
    "manufacturer": "Company Inc."
  },
  "result": "allergen-detected",
  "allergensDetected": [
    {
      "allergen": "Almond",
      "ingredientName": "Prunus amygdalus oil",
      "severity": "severe",
      "confidence": 0.98
    }
  ],
  "safeToUse": false,
  "timestamp": "2026-02-15T10:30:00Z"
}
```

```
GET /scans/history
Query Parameters:
- limit (default: 20, max: 100)
- offset (default: 0)
- result (filter: safe|allergen-detected|unknown)

Response: 200 OK
{
  "scans": [...],
  "total": 45,
  "limit": 20,
  "offset": 0
}
```

#### 5.1.3 Profile Endpoints

```
GET /profile
Response: 200 OK
{
  "userId": "uuid",
  "email": "user@example.com",
  "firstName": "John",
  "allergies": [...]
}
```

```
POST /profile/allergies
Request:
{
  "allergen": "Peanuts",
  "severity": "life-threatening",
  "reactions": ["Anaphylaxis", "Hives"],
  "verifiedBy": "doctor"
}

Response: 201 Created
{
  "allergyId": "uuid",
  "allergen": "Peanuts",
  "severity": "life-threatening"
}
```

```
DELETE /profile/allergies/{allergyId}
Response: 204 No Content
```

#### 5.1.4 Product Endpoints

```
GET /products/search
Query Parameters:
- q (search query)
- category (filter)
- limit (default: 10)

Response: 200 OK
{
  "products": [
    {
      "productId": "uuid",
      "name": "Product Name",
      "manufacturer": "Company",
      "category": "cosmetic"
    }
  ]
}
```

```
GET /products/{productId}
Response: 200 OK
{
  "productId": "uuid",
  "barcode": "012345678901",
  "name": "Product Name",
  "manufacturer": "Company",
  "category": "cosmetic",
  "ingredients": [
    {
      "name": "Water",
      "purpose": "solvent"
    },
    {
      "name": "Glycerin",
      "purpose": "humectant"
    }
  ]
}
```

```
POST /products/check
Request:
{
  "productId": "uuid",
  "userId": "uuid"
}

Response: 200 OK
{
  "safeToUse": false,
  "allergensDetected": [...],
  "alternatives": [
    {
      "productId": "uuid",
      "name": "Safe Alternative",
      "similarity": 0.85
    }
  ]
}
```

#### 5.1.5 Provider Endpoints

```
GET /provider/patients
Query Parameters:
- search (name or ID)
- limit
- offset

Response: 200 OK
{
  "patients": [
    {
      "userId": "uuid",
      "name": "John Doe",
      "lastScan": "2026-02-14T15:30:00Z",
      "allergyCount": 3,
      "riskLevel": "high"
    }
  ]
}
```

```
GET /provider/patients/{userId}/allergies
Response: 200 OK
{
  "patient": {
    "userId": "uuid",
    "name": "John Doe"
  },
  "allergies": [
    {
      "allergen": "Peanuts",
      "severity": "life-threatening",
      "lastReaction": "2025-06-10"
    }
  ]
}
```

```
POST /provider/check-product
Request:
{
  "userId": "uuid",
  "productName": "Aspirin 500mg"
}

Response: 200 OK
{
  "safeToUse": true,
  "recommendations": [
    "No allergens detected",
    "Safe to prescribe"
  ]
}
```

### 5.2 WebSocket API

**Connection:** `wss://ws.Allersense.com`

#### 5.2.1 Real-time Scan Updates

```javascript
// Client connects
const ws = new WebSocket('wss://ws.Allersense.com');

// Authenticate
ws.send(JSON.stringify({
  action: 'authenticate',
  token: 'jwt-token'
}));

// Subscribe to scan updates
ws.send(JSON.stringify({
  action: 'subscribe',
  channel: 'scan-updates',
  scanId: 'uuid'
}));

// Receive updates
ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  
  if (data.type === 'scan-progress') {
    console.log(`Scan ${data.progress}% complete`);
  }
  
  if (data.type === 'scan-complete') {
    console.log('Result:', data.result);
  }
};
```

#### 5.2.2 Provider Alerts

```javascript
// Provider subscribes to patient alerts
ws.send(JSON.stringify({
  action: 'subscribe',
  channel: 'provider-alerts'
}));

// Receive alert when patient scans allergen
ws.onmessage = (event) => {
  const alert = JSON.parse(event.data);
  
  if (alert.type === 'patient-allergen-detected') {
    showAlert({
      patient: alert.patientName,
      product: alert.productName,
      allergens: alert.allergens
    });
  }
};
```

### 5.3 API Rate Limiting

```
Tier: Free
- 100 requests/hour
- 10 scans/day

Tier: Premium  
- 1,000 requests/hour
- Unlimited scans

Tier: Provider
- 10,000 requests/hour
- Unlimited scans
- Real-time alerts

Tier: Enterprise
- 100,000 requests/hour
- Custom limits
- Dedicated support
```

**Rate Limit Headers:**
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 987
X-RateLimit-Reset: 1709638800
```

---

## 6. User Interface Design

### 6.1 Mobile App UI/UX

#### 6.1.1 Design System

**Colors:**
```
Primary: #10B981 (Green - Safety)
Danger: #DC2626 (Red - Alert)
Warning: #F59E0B (Amber)
Info: #3B82F6 (Blue)
Background: #FFFFFF
Surface: #F9FAFB
Text Primary: #111827
Text Secondary: #6B7280
```

**Typography:**
```
Heading 1: 28px, Bold
Heading 2: 24px, Semi-bold
Heading 3: 20px, Semi-bold
Body: 16px, Regular
Caption: 14px, Regular
Small: 12px, Regular
```

**Spacing:**
```
xs: 4px
sm: 8px
md: 16px
lg: 24px
xl: 32px
```

#### 6.1.2 Key Screens

**Home Screen**
```
┌─────────────────────────┐
│  Allersense      ☰   │
├─────────────────────────┤
│                         │
│   Welcome, John!        │
│                         │
│  ┌───────────────────┐  │
│  │                   │  │
│  │   [SCAN BUTTON]   │  │
│  │   Camera Icon     │  │
│  │                   │  │
│  └───────────────────┘  │
│                         │
│  Recent Scans           │
│  ┌──────────────────┐   │
│  │ Product A  ✓Safe │   │
│  └──────────────────┘   │
│  ┌──────────────────┐   │
│  │ Product B  ⚠️Alert│   │
│  └──────────────────┘   │
│                         │
│  [View All History]     │
│                         │
└─────────────────────────┘
```

**Scan Screen**
```
┌─────────────────────────┐
│  ←  Scanning            │
├─────────────────────────┤
│                         │
│  ┌───────────────────┐  │
│  │                   │  │
│  │   Camera View     │  │
│  │                   │  │
│  │   [Viewfinder]    │  │
│  │                   │  │
│  │                   │  │
│  └───────────────────┘  │
│                         │
│  [📷 Capture] [🔦Flash] │
│                         │
│  Or enter barcode:      │
│  [___________________]  │
│                         │
└─────────────────────────┘
```

**Alert Screen** (Allergen Detected)
```
┌─────────────────────────┐
│  ⚠️  ALLERGEN DETECTED   │
├─────────────────────────┤
│                         │
│  Product Name           │
│  Manufacturer           │
│                         │
│  ┌───────────────────┐  │
│  │ DO NOT USE        │  │
│  │                   │  │
│  │ Contains:         │  │
│  │ • Almond Oil      │  │
│  │   (Severe)        │  │
│  │                   │  │
│  │ Matches your      │  │
│  │ allergy to:       │  │
│  │ Tree Nuts         │  │
│  └───────────────────┘  │
│                         │
│  [View Alternatives]    │
│  [Contact Doctor]       │
│                         │
└─────────────────────────┘
```

**Safe Product Screen**
```
┌─────────────────────────┐
│  ✓  SAFE TO USE         │
├─────────────────────────┤
│                         │
│  Product Name           │
│  Manufacturer           │
│                         │
│  ┌───────────────────┐  │
│  │  ✓                │  │
│  │                   │  │
│  │  No allergens     │  │
│  │  detected         │  │
│  │                   │  │
│  │  Analyzed:        │  │
│  │  • Water          │  │
│  │  • Glycerin       │  │
│  │  • Vitamin E      │  │
│  └───────────────────┘  │
│                         │
│  [Save to Favorites]    │
│  [View Ingredients]     │
│                         │
└─────────────────────────┘
```

**Allergy Profile Screen**
```
┌─────────────────────────┐
│  ←  My Allergies        │
├─────────────────────────┤
│                         │
│  Active Allergies:      │
│                         │
│  ┌──────────────────┐   │
│  │ 🥜 Peanuts       │   │
│  │ Life-threatening │   │
│  │ [Edit] [Delete]  │   │
│  └──────────────────┘   │
│                         │
│  ┌──────────────────┐   │
│  │ 🌳 Tree Nuts     │   │
│  │ Severe           │   │
│  │ [Edit] [Delete]  │   │
│  └──────────────────┘   │
│                         │
│  [+ Add New Allergy]    │
│                         │
│  [Generate QR Code]     │
│  [Share Profile]        │
│                         │
└─────────────────────────┘
```

### 6.2 Web Portal UI/UX

#### 6.2.1 Provider Dashboard
```
┌─────────────────────────────────────────────────────────┐
│  Allersense Provider Portal           [User Menu ▼]  │
├─────────────────────────────────────────────────────────┤
│  Dashboard  |  Patients  |  Products  |  Analytics      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Recent Alerts                                          │
│  ┌────────────────────────────────────────────────┐    │
│  │ ⚠️ John Doe scanned allergen-containing product│    │
│  │ 2 minutes ago                         [View]   │    │
│  └────────────────────────────────────────────────┘    │
│                                                         │
│  ┌──────────────────┐  ┌──────────────────┐            │
│  │  Active Patients │  │  Scans Today     │            │
│  │      127         │  │      245         │            │
│  └──────────────────┘  └──────────────────┘            │
│                                                         │
│  ┌──────────────────┐  ┌──────────────────┐            │
│  │  High-Risk       │  │  Interventions   │            │
│  │  Patients: 12    │  │  This Week: 8    │            │
│  └──────────────────┘  └──────────────────┘            │
│                                                         │
│  Common Allergens in Population                         │
│  ┌────────────────────────────────────────────────┐    │
│  │ Peanuts     ████████████████░░░░░░  75%        │    │
│  │ Tree Nuts   ██████████████░░░░░░░░  65%        │    │
│  │ Shellfish   ████████░░░░░░░░░░░░░░  40%        │    │
│  └────────────────────────────────────────────────┘    │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

#### 6.2.2 Product Check Interface
```
┌─────────────────────────────────────────────────────────┐
│  Product Safety Check                                   │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Patient: [Search patients...        ▼]                │
│                                                         │
│  Product: [Enter product name or scan barcode]         │
│           [Scan Barcode]                                │
│                                                         │
│  ┌────────────────────────────────────────────────┐    │
│  │  Selected: John Doe                            │    │
│  │  Known Allergies: Peanuts, Tree Nuts, Latex    │    │
│  └────────────────────────────────────────────────┘    │
│                                                         │
│  [Check Product Safety]                                 │
│                                                         │
│  Result:                                                │
│  ┌────────────────────────────────────────────────┐    │
│  │  ⚠️ ALLERGEN DETECTED                           │    │
│  │                                                 │    │
│  │  Product: Generic Hand Lotion                  │    │
│  │  Manufacturer: Example Corp                     │    │
│  │                                                 │    │
│  │  Contains: Almond Oil                          │    │
│  │  Matches: Tree Nuts (Severe)                   │    │
│  │                                                 │    │
│  │  Recommendation: DO NOT PRESCRIBE              │    │
│  │                                                 │    │
│  │  Safe Alternatives:                            │    │
│  │  • Product A (95% similar, no allergens)       │    │
│  │  • Product B (90% similar, no allergens)       │    │
│  └────────────────────────────────────────────────┘    │
│                                                         │
│  [Add to Patient Record]  [Print Report]                │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 7. AI/ML Design

### 7.1 AWS Bedrock Integration

#### 7.1.1 Claude Prompt Engineering

**System Prompt:**
```
You are an expert allergen detection system integrated into Allersense, a healthcare application. Your role is to analyze product ingredients against user allergy profiles with extreme accuracy and caution.

CRITICAL REQUIREMENTS:
1. Never miss a potential allergen - false negatives are unacceptable
2. Be conservative - flag anything questionable
3. Recognize botanical names, chemical names, and synonyms
4. Consider cross-reactive allergens
5. Provide confidence scores for all matches
6. Explain your reasoning clearly

RESPONSE FORMAT:
Always respond with valid JSON matching this schema:
{
  "detected_allergens": [...],
  "cross_reactive_warnings": [...],
  "overall_safety": "safe|warning|danger",
  "confidence": 0.0-1.0,
  "requires_human_review": boolean,
  "explanation": "string"
}

EXAMPLES OF MATCHES TO CATCH:
- "Prunus amygdalus" = Almond
- "Arachis hypogaea" = Peanut
- "Triticum vulgare" = Wheat/Gluten
- "Bos taurus" = Dairy/Milk
- Latex + Banana = Cross-reactivity
```

**User Prompt Template:**
```python
def build_allergen_detection_prompt(
    ingredients: List[str],
    allergies: List[Dict],
    product_context: Dict
) -> str:
    
    allergy_details = '\n'.join([
        f"- {a['allergen']} ({a['severity']})\n"
        f"  Previous reactions: {', '.join(a['reactions'])}\n"
        f"  Verified by: {a['verifiedBy']}"
        for a in allergies
    ])
    
    ingredient_list = '\n'.join([
        f"{i+1}. {ing}" 
        for i, ing in enumerate(ingredients)
    ])
    
    return f"""
TASK: Analyze ingredients for allergens

PATIENT ALLERGY PROFILE:
{allergy_details}

PRODUCT INFORMATION:
Name: {product_context['name']}
Category: {product_context['category']}
Manufacturer: {product_context['manufacturer']}

INGREDIENTS TO ANALYZE:
{ingredient_list}

INSTRUCTIONS:
1. Check each ingredient against the allergy list
2. Look for direct matches AND synonyms (botanical, chemical, common names)
3. Consider product category context
4. Flag potential cross-reactive allergens
5. Be EXTREMELY conservative - when in doubt, flag it

Respond ONLY with JSON. Do not include any markdown formatting.
"""
```

#### 7.1.2 Response Parsing & Validation

```python
import json
import re
from typing import Dict, List, Any

class ClaudeResponseParser:
    def __init__(self):
        self.required_fields = [
            'detected_allergens',
            'overall_safety',
            'confidence'
        ]
    
    def parse_response(self, claude_response: str) -> Dict[str, Any]:
        """
        Parse and validate Claude's JSON response
        """
        try:
            # Remove markdown formatting if present
            cleaned = re.sub(r'```json\s*|\s*```', '', claude_response)
            cleaned = cleaned.strip()
            
            # Parse JSON
            data = json.loads(cleaned)
            
            # Validate required fields
            self.validate_schema(data)
            
            # Sanitize and normalize
            normalized = self.normalize_response(data)
            
            return normalized
            
        except json.JSONDecodeError as e:
            raise ValueError(f"Invalid JSON from Claude: {e}")
    
    def validate_schema(self, data: Dict) -> None:
        """Ensure response has required fields"""
        for field in self.required_fields:
            if field not in data:
                raise ValueError(f"Missing required field: {field}")
        
        # Validate safety level
        valid_safety = ['safe', 'warning', 'danger']
        if data['overall_safety'] not in valid_safety:
            raise ValueError(f"Invalid safety level: {data['overall_safety']}")
        
        # Validate confidence
        if not 0 <= data['confidence'] <= 1:
            raise ValueError(f"Invalid confidence: {data['confidence']}")
    
    def normalize_response(self, data: Dict) -> Dict:
        """Normalize and enrich response"""
        
        # Add severity to each detected allergen based on user profile
        for allergen in data.get('detected_allergens', []):
            if 'severity' not in allergen:
                allergen['severity'] = 'unknown'
        
        # Add timestamp
        data['analyzed_at'] = datetime.utcnow().isoformat()
        
        # Add model version
        data['model_version'] = 'claude-sonnet-4-5-20250929'
        
        return data
```

### 7.2 Amazon Textract Integration

#### 7.2.1 OCR Configuration

```python
import boto3
from typing import Dict, List

class TextractProcessor:
    def __init__(self):
        self.textract = boto3.client('textract')
    
    def extract_text(self, s3_bucket: str, s3_key: str) -> Dict:
        """
        Extract text from product label image
        """
        
        # Use DetectDocumentText for simple labels
        response = self.textract.detect_document_text(
            Document={
                'S3Object': {
                    'Bucket': s3_bucket,
                    'Name': s3_key
                }
            }
        )
        
        return self.process_textract_response(response)
    
    def extract_tables(self, s3_bucket: str, s3_key: str) -> Dict:
        """
        Extract structured data (like nutrition facts tables)
        """
        
        # Use AnalyzeDocument for complex layouts
        response = self.textract.analyze_document(
            Document={
                'S3Object': {
                    'Bucket': s3_bucket,
                    'Name': s3_key
                }
            },
            FeatureTypes=['TABLES', 'FORMS']
        )
        
        return self.process_analyze_response(response)
    
    def process_textract_response(self, response: Dict) -> Dict:
        """
        Process Textract DetectDocumentText response
        """
        
        blocks = response['Blocks']
        
        # Extract lines in reading order
        lines = []
        for block in blocks:
            if block['BlockType'] == 'LINE':
                lines.append({
                    'text': block['Text'],
                    'confidence': block['Confidence'],
                    'geometry': block['Geometry']
                })
        
        # Find ingredients section
        ingredients_section = self.find_ingredients_section(lines)
        
        return {
            'full_text': ' '.join([l['text'] for l in lines]),
            'lines': lines,
            'ingredients': ingredients_section,
            'average_confidence': sum([l['confidence'] for l in lines]) / len(lines)
        }
    
    def find_ingredients_section(self, lines: List[Dict]) -> str:
        """
        Identify and extract ingredients list from text
        """
        
        ingredients_keywords = [
            'ingredients',
            'contains',
            'composition'
        ]
        
        # Find start of ingredients section
        start_idx = None
        for i, line in enumerate(lines):
            text_lower = line['text'].lower()
            if any(keyword in text_lower for keyword in ingredients_keywords):
                start_idx = i
                break
        
        if start_idx is None:
            # No explicit ingredients section found
            # Return all text for Claude to parse
            return ' '.join([l['text'] for l in lines])
        
        # Extract from ingredients section to end or next section
        end_idx = len(lines)
        section_keywords = ['directions', 'warnings', 'storage']
        
        for i in range(start_idx + 1, len(lines)):
            if any(kw in lines[i]['text'].lower() for kw in section_keywords):
                end_idx = i
                break
        
        ingredients_lines = lines[start_idx:end_idx]
        return ' '.join([l['text'] for l in ingredients_lines])
```

#### 7.2.2 Image Quality Validation

```python
import boto3
from PIL import Image
import io

class ImageQualityChecker:
    def __init__(self):
        self.rekognition = boto3.client('rekognition')
        self.min_width = 400
        self.min_height = 400
        self.max_size_mb = 10
    
    def validate_image(self, image_bytes: bytes) -> Dict:
        """
        Validate image quality before OCR
        """
        
        # Check file size
        size_mb = len(image_bytes) / (1024 * 1024)
        if size_mb > self.max_size_mb:
            return {
                'valid': False,
                'reason': f'Image too large: {size_mb:.1f}MB (max 10MB)'
            }
        
        # Check dimensions
        img = Image.open(io.BytesIO(image_bytes))
        width, height = img.size
        
        if width < self.min_width or height < self.min_height:
            return {
                'valid': False,
                'reason': f'Image too small: {width}x{height} (min 400x400)'
            }
        
        # Check for blur using Rekognition
        response = self.rekognition.detect_labels(
            Image={'Bytes': image_bytes},
            MaxLabels=1,
            Features=['GENERAL_LABELS', 'IMAGE_PROPERTIES']
        )
        
        # Get quality score
        quality = response.get('ImageProperties', {}).get('Quality', {})
        sharpness = quality.get('Sharpness', 0)
        brightness = quality.get('Brightness', 0)
        
        if sharpness < 50:
            return {
                'valid': False,
                'reason': 'Image too blurry. Please retake photo.'
            }
        
        if brightness < 30 or brightness > 90:
            return {
                'valid': False,
                'reason': 'Poor lighting. Please adjust and retake.'
            }
        
        return {
            'valid': True,
            'quality_score': {
                'sharpness': sharpness,
                'brightness': brightness
            }
        }
```

### 7.3 Amazon Comprehend Medical

#### 7.3.1 Prescription Processing

```python
import boto3
from typing import Dict, List

class PrescriptionProcessor:
    def __init__(self):
        self.comprehend = boto3.client('comprehendmedical')
    
    def extract_medication_info(self, prescription_text: str) -> Dict:
        """
        Extract medication entities from prescription
        """
        
        response = self.comprehend.detect_entities_v2(
            Text=prescription_text
        )
        
        medications = []
        dosages = []
        routes = []
        
        for entity in response['Entities']:
            if entity['Category'] == 'MEDICATION':
                medications.append({
                    'name': entity['Text'],
                    'confidence': entity['Score'],
                    'generic_name': self.get_generic_name(entity)
                })
            
            elif entity['Category'] == 'DOSAGE':
                dosages.append({
                    'amount': entity['Text'],
                    'confidence': entity['Score']
                })
            
            elif entity['Category'] == 'ROUTE_OR_MODE':
                routes.append({
                    'route': entity['Text'],
                    'confidence': entity['Score']
                })
        
        return {
            'medications': medications,
            'dosages': dosages,
            'routes': routes,
            'full_entities': response['Entities']
        }
    
    def get_generic_name(self, entity: Dict) -> str:
        """
        Extract generic medication name from attributes
        """
        attributes = entity.get('Attributes', [])
        
        for attr in attributes:
            if attr['Type'] == 'GENERIC_NAME':
                return attr['Text']
        
        return entity['Text']  # Fallback to entity text
```

---

## 8. Security Design

### 8.1 Authentication & Authorization

#### 8.1.1 AWS Cognito Configuration

```yaml
UserPool:
  PoolName: Allersense-users
  Policies:
    PasswordPolicy:
      MinimumLength: 8
      RequireUppercase: true
      RequireLowercase: true
      RequireNumbers: true
      RequireSymbols: true
      TemporaryPasswordValidityDays: 7
  
  MfaConfiguration: OPTIONAL
  EnabledMfas:
    - SOFTWARE_TOKEN_MFA
    - SMS_MFA
  
  AccountRecoverySetting:
    RecoveryMechanisms:
      - Name: verified_email
        Priority: 1
      - Name: verified_phone_number
        Priority: 2
  
  Schema:
    - Name: email
      Required: true
      Mutable: false
    - Name: phone_number
      Required: false
      Mutable: true
    - Name: custom:user_type
      AttributeDataType: String
      Mutable: true
  
  UserPoolTags:
    Environment: production
    Application: Allersense
```

#### 8.1.2 JWT Token Design

```javascript
// Access Token (15 minutes)
{
  "sub": "user-uuid",
  "iss": "https://cognito-idp.us-east-1.amazonaws.com/...",
  "aud": "client-id",
  "exp": 1677777777,
  "iat": 1677776877,
  "token_use": "access",
  "scope": "openid profile email",
  "custom:user_type": "patient",
  "cognito:groups": ["premium-users"]
}

// Refresh Token (30 days)
{
  "sub": "user-uuid",
  "exp": 1680369777,
  "iat": 1677776877,
  "token_use": "refresh"
}
```

#### 8.1.3 Role-Based Access Control

```python
from enum import Enum
from typing import List

class UserRole(Enum):
    PATIENT = "patient"
    PROVIDER = "provider"
    PHARMACIST = "pharmacist"
    ADMIN = "admin"

class Permission(Enum):
    # Scan permissions
    SCAN_PRODUCT = "scan:product"
    VIEW_SCAN_HISTORY = "scan:view_history"
    DELETE_SCAN = "scan:delete"
    
    # Profile permissions
    MANAGE_OWN_PROFILE = "profile:manage_own"
    VIEW_OTHER_PROFILES = "profile:view_others"
    EDIT_OTHER_PROFILES = "profile:edit_others"
    
    # Product permissions
    CHECK_PRODUCT = "product:check"
    ADD_PRODUCT = "product:add"
    EDIT_PRODUCT = "product:edit"
    
    # Provider permissions
    VIEW_PATIENT_DATA = "provider:view_patients"
    PRESCRIBE = "provider:prescribe"

# Role-Permission Mapping
ROLE_PERMISSIONS = {
    UserRole.PATIENT: [
        Permission.SCAN_PRODUCT,
        Permission.VIEW_SCAN_HISTORY,
        Permission.DELETE_SCAN,
        Permission.MANAGE_OWN_PROFILE,
        Permission.CHECK_PRODUCT
    ],
    
    UserRole.PROVIDER: [
        Permission.VIEW_PATIENT_DATA,
        Permission.PRESCRIBE,
        Permission.CHECK_PRODUCT,
        Permission.VIEW_OTHER_PROFILES
    ],
    
    UserRole.PHARMACIST: [
        Permission.CHECK_PRODUCT,
        Permission.VIEW_PATIENT_DATA
    ],
    
    UserRole.ADMIN: [
        # All permissions
        perm for perm in Permission
    ]
}

def check_permission(user_role: UserRole, permission: Permission) -> bool:
    """Check if user role has permission"""
    return permission in ROLE_PERMISSIONS.get(user_role, [])
```

### 8.2 Data Encryption

#### 8.2.1 Encryption at Rest

```python
# KMS Key Configuration
{
    "KeyId": "arn:aws:kms:us-east-1:123456789012:key/...",
    "Description": "Allersense master key",
    "KeyUsage": "ENCRYPT_DECRYPT",
    "KeyState": "Enabled",
    "KeyPolicy": {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "Enable IAM policies",
                "Effect": "Allow",
                "Principal": {
                    "AWS": "arn:aws:iam::123456789012:root"
                },
                "Action": "kms:*",
                "Resource": "*"
            },
            {
                "Sid": "Allow services to use key",
                "Effect": "Allow",
                "Principal": {
                    "Service": [
                        "dynamodb.amazonaws.com",
                        "rds.amazonaws.com",
                        "s3.amazonaws.com"
                    ]
                },
                "Action": [
                    "kms:Decrypt",
                    "kms:DescribeKey",
                    "kms:CreateGrant"
                ],
                "Resource": "*"
            }
        ]
    },
    "Tags": [
        {
            "Key": "Environment",
            "Value": "production"
        }
    ]
}

# DynamoDB Encryption
Table(
    encryption_specification=EncryptionSpecification(
        encryption_type="KMS",
        kms_master_key_id="alias/Allersense-key"
    )
)

# RDS Encryption
DBInstance(
    storage_encrypted=True,
    kms_key_id="alias/Allersense-key"
)

# S3 Encryption
Bucket(
    encryption_configuration=ServerSideEncryptionConfiguration(
        rules=[
            ServerSideEncryptionRule(
                apply_server_side_encryption_by_default=ServerSideEncryptionByDefault(
                    sse_algorithm="aws:kms",
                    kms_master_key_id="alias/Allersense-key"
                )
            )
        ]
    )
)
```

#### 8.2.2 Field-Level Encryption

```python
import boto3
from cryptography.fernet import Fernet
import base64
import os

class FieldEncryption:
    def __init__(self):
        self.kms = boto3.client('kms')
        self.key_id = os.environ['KMS_KEY_ID']
        
    def encrypt_field(self, plaintext: str) -> str:
        """
        Encrypt sensitive field using KMS data key
        """
        
        # Generate data key
        response = self.kms.generate_data_key(
            KeyId=self.key_id,
            KeySpec='AES_256'
        )
        
        plaintext_key = response['Plaintext']
        encrypted_key = response['CiphertextBlob']
        
        # Encrypt data
        f = Fernet(base64.urlsafe_b64encode(plaintext_key))
        encrypted_data = f.encrypt(plaintext.encode())
        
        # Return encrypted key + encrypted data
        return base64.b64encode(
            encrypted_key + b'::' + encrypted_data
        ).decode()
    
    def decrypt_field(self, encrypted_value: str) -> str:
        """
        Decrypt sensitive field
        """
        
        # Decode
        decoded = base64.b64decode(encrypted_value)
        encrypted_key, encrypted_data = decoded.split(b'::', 1)
        
        # Decrypt data key
        response = self.kms.decrypt(
            CiphertextBlob=encrypted_key
        )
        plaintext_key = response['Plaintext']
        
        # Decrypt data
        f = Fernet(base64.urlsafe_b64encode(plaintext_key))
        plaintext = f.decrypt(encrypted_data)
        
        return plaintext.decode()

# Usage in DynamoDB
encryption = FieldEncryption()

# Store
item = {
    'userId': 'uuid',
    'email_encrypted': encryption.encrypt_field('user@example.com'),
    'phone_encrypted': encryption.encrypt_field('+1234567890')
}

# Retrieve
email = encryption.decrypt_field(item['email_encrypted'])
```

### 8.3 HIPAA Compliance

#### 8.3.1 Audit Logging

```python
import boto3
import json
from datetime import datetime
from typing import Dict, Any

class AuditLogger:
    def __init__(self):
        self.cloudwatch = boto3.client('logs')
        self.log_group = '/aws/Allersense/audit'
        self.log_stream = f'audit-{datetime.now().strftime("%Y%m%d")}'
        
    def log_phi_access(
        self,
        user_id: str,
        accessed_user_id: str,
        action: str,
        resource_type: str,
        success: bool,
        metadata: Dict = None
    ):
        """
        Log all PHI access for HIPAA compliance
        """
        
        log_event = {
            'timestamp': datetime.utcnow().isoformat(),
            'event_type': 'phi_access',
            'user_id': user_id,
            'accessed_user_id': accessed_user_id,
            'action': action,
            'resource_type': resource_type,
            'success': success,
            'ip_address': self.get_caller_ip(),
            'user_agent': self.get_user_agent(),
            'metadata': metadata or {}
        }
        
        self.cloudwatch.put_log_events(
            logGroupName=self.log_group,
            logStreamName=self.log_stream,
            logEvents=[
                {
                    'timestamp': int(datetime.utcnow().timestamp() * 1000),
                    'message': json.dumps(log_event)
                }
            ]
        )
    
    def log_data_modification(
        self,
        user_id: str,
        table_name: str,
        record_id: str,
        action: str,  # create, update, delete
        old_value: Any = None,
        new_value: Any = None
    ):
        """
        Log all data modifications for audit trail
        """
        
        log_event = {
            'timestamp': datetime.utcnow().isoformat(),
            'event_type': 'data_modification',
            'user_id': user_id,
            'table_name': table_name,
            'record_id': record_id,
            'action': action,
            'old_value': self.sanitize_for_logging(old_value),
            'new_value': self.sanitize_for_logging(new_value)
        }
        
        self.cloudwatch.put_log_events(
            logGroupName=self.log_group,
            logStreamName=self.log_stream,
            logEvents=[
                {
                    'timestamp': int(datetime.utcnow().timestamp() * 1000),
                    'message': json.dumps(log_event)
                }
            ]
        )
    
    def sanitize_for_logging(self, value: Any) -> Any:
        """
        Remove sensitive fields from logged values
        """
        if isinstance(value, dict):
            return {
                k: '***REDACTED***' if k in ['password', 'token', 'ssn'] else v
                for k, v in value.items()
            }
        return value
```

---

## 9. Integration Design

### 9.1 EHR Integration

#### 9.1.1 FHIR Implementation

```python
from fhirclient import client
from fhirclient.models.allergyintolerance import AllergyIntolerance
from fhirclient.models.patient import Patient
from fhirclient.models.codeableconcept import CodeableConcept
from fhirclient.models.coding import Coding
import requests

class FHIRIntegration:
    def __init__(self, fhir_base_url: str, client_id: str, client_secret: str):
        self.settings = {
            'app_id': 'Allersense',
            'api_base': fhir_base_url
        }
        self.client = client.FHIRClient(settings=self.settings)
        self.client_id = client_id
        self.client_secret = client_secret
        
    def create_allergy_resource(
        self,
        patient_fhir_id: str,
        allergen: str,
        severity: str,
        reactions: list
    ) -> AllergyIntolerance:
        """
        Create FHIR AllergyIntolerance resource
        """
        
        allergy = AllergyIntolerance()
        
        # Patient reference
        allergy.patient = {
            'reference': f'Patient/{patient_fhir_id}'
        }
        
        # Code (allergen)
        allergy.code = CodeableConcept()
        allergy.code.coding = [
            Coding({
                'system': 'http://snomed.info/sct',
                'code': self.get_snomed_code(allergen),
                'display': allergen
            })
        ]
        allergy.code.text = allergen
        
        # Clinical status
        allergy.clinicalStatus = CodeableConcept()
        allergy.clinicalStatus.coding = [
            Coding({
                'system': 'http://terminology.hl7.org/CodeSystem/allergyintolerance-clinical',
                'code': 'active',
                'display': 'Active'
            })
        ]
        
        # Verification status
        allergy.verificationStatus = CodeableConcept()
        allergy.verificationStatus.coding = [
            Coding({
                'system': 'http://terminology.hl7.org/CodeSystem/allergyintolerance-verification',
                'code': 'confirmed',
                'display': 'Confirmed'
            })
        ]
        
        # Type
        allergy.type = 'allergy'
        
        # Category
        allergy.category = [self.determine_category(allergen)]
        
        # Criticality
        allergy.criticality = self.map_severity_to_criticality(severity)
        
        # Reactions
        if reactions:
            allergy.reaction = []
            for reaction in reactions:
                allergy.reaction.append({
                    'manifestation': [
                        CodeableConcept({
                            'text': reaction
                        })
                    ]
                })
        
        return allergy
    
    def push_to_ehr(self, allergy: AllergyIntolerance) -> str:
        """
        Push allergy resource to EHR system
        """
        
        # Convert to JSON
        allergy_json = allergy.as_json()
        
        # POST to FHIR server
        response = requests.post(
            f'{self.settings["api_base"]}/AllergyIntolerance',
            json=allergy_json,
            headers={
                'Content-Type': 'application/fhir+json',
                'Authorization': f'Bearer {self.get_access_token()}'
            }
        )
        
        if response.status_code == 201:
            # Extract resource ID from Location header
            location = response.headers.get('Location')
            resource_id = location.split('/')[-1]
            return resource_id
        else:
            raise Exception(f'Failed to create resource: {response.text}')
    
    def sync_from_ehr(self, patient_fhir_id: str) -> list:
        """
        Pull allergy data from EHR system
        """
        
        # Search for patient allergies
        response = requests.get(
            f'{self.settings["api_base"]}/AllergyIntolerance',
            params={'patient': patient_fhir_id},
            headers={'Authorization': f'Bearer {self.get_access_token()}'}
        )
        
        if response.status_code == 200:
            bundle = response.json()
            allergies = []
            
            for entry in bundle.get('entry', []):
                resource = entry['resource']
                allergies.append({
                    'fhir_id': resource['id'],
                    'allergen': resource['code']['text'],
                    'severity': self.map_criticality_to_severity(
                        resource.get('criticality')
                    ),
                    'reactions': [
                        r['manifestation'][0]['text']
                        for r in resource.get('reaction', [])
                    ]
                })
            
            return allergies
        else:
            raise Exception(f'Failed to retrieve allergies: {response.text}')
    
    def get_access_token(self) -> str:
        """
        Get OAuth access token for EHR system
        """
        # Implement OAuth 2.0 flow
        # This is simplified - actual implementation depends on EHR vendor
        pass
    
    def get_snomed_code(self, allergen: str) -> str:
        """
        Map allergen name to SNOMED CT code
        """
        # Simplified - would use SNOMED CT terminology service
        snomed_map = {
            'Peanut': '256349002',
            'Tree nut': '762952008',
            'Milk': '425525006',
            'Egg': '102263004',
            # ... more mappings
        }
        return snomed_map.get(allergen, '0')
    
    def determine_category(self, allergen: str) -> str:
        """
        Determine FHIR category from allergen
        """
        food_allergens = ['Peanut', 'Tree nut', 'Milk', 'Egg', 'Wheat']
        if allergen in food_allergens:
            return 'food'
        
        medication_allergens = ['Penicillin', 'Aspirin']
        if allergen in medication_allergens:
            return 'medication'
        
        return 'environment'
    
    def map_severity_to_criticality(self, severity: str) -> str:
        """
        Map Allersense severity to FHIR criticality
        """
        mapping = {
            'mild': 'low',
            'moderate': 'high',
            'severe': 'high',
            'life-threatening': 'unable-to-assess'  # Requires immediate action
        }
        return mapping.get(severity, 'high')
    
    def map_criticality_to_severity(self, criticality: str) -> str:
        """
        Map FHIR criticality to Allersense severity
        """
        mapping = {
            'low': 'mild',
            'high': 'severe',
            'unable-to-assess': 'life-threatening'
        }
        return mapping.get(criticality, 'moderate')
```

### 9.2 External API Integrations

#### 9.2.1 FDA Drug Database

```python
import requests
from typing import Dict, List, Optional

class FDADrugAPI:
    def __init__(self):
        self.base_url = 'https://api.fda.gov/drug'
    
    def search_by_name(self, drug_name: str) -> List[Dict]:
        """
        Search FDA database for drug information
        """
        
        response = requests.get(
            f'{self.base_url}/label.json',
            params={
                'search': f'openfda.brand_name:"{drug_name}"',
                'limit': 10
            }
        )
        
        if response.status_code == 200:
            data = response.json()
            return self.parse_drug_labels(data['results'])
        
        return []
    
    def get_ingredients(self, ndc_code: str) -> Optional[Dict]:
        """
        Get drug ingredients by NDC code
        """
        
        response = requests.get(
            f'{self.base_url}/ndc.json',
            params={'search': f'product_ndc:"{ndc_code}"'}
        )
        
        if response.status_code == 200:
            data = response.json()
            if data['results']:
                return self.parse_ingredients(data['results'][0])
        
        return None
    
    def parse_drug_labels(self, results: List[Dict]) -> List[Dict]:
        """
        Parse FDA drug label data
        """
        
        parsed = []
        for result in results:
            openfda = result.get('openfda', {})
            
            parsed.append({
                'brand_name': openfda.get('brand_name', [''])[0],
                'generic_name': openfda.get('generic_name', [''])[0],
                'manufacturer_name': openfda.get('manufacturer_name', [''])[0],
                'active_ingredients': self.extract_active_ingredients(result),
                'inactive_ingredients': self.extract_inactive_ingredients(result),
                'warnings': result.get('warnings', [''])
            })
        
        return parsed
    
    def extract_active_ingredients(self, label: Dict) -> List[str]:
        """
        Extract active ingredients from label
        """
        active = label.get('active_ingredient', [])
        if active:
            return [ing.strip() for ing in active[0].split(',')]
        return []
    
    def extract_inactive_ingredients(self, label: Dict) -> List[str]:
        """
        Extract inactive ingredients from label
        """
        inactive = label.get('inactive_ingredient', [])
        if inactive:
            # Parse ingredient list
            ingredients = []
            for section in inactive:
                ingredients.extend([
                    ing.strip() 
                    for ing in section.split(',')
                ])
            return ingredients
        return []
```

#### 9.2.2 OpenFoodFacts Integration

```python
import requests
from typing import Dict, Optional

class OpenFoodFactsAPI:
    def __init__(self):
        self.base_url = 'https://world.openfoodfacts.org/api/v0'
        
    def get_product_by_barcode(self, barcode: str) -> Optional[Dict]:
        """
        Get product information by barcode
        """
        
        response = requests.get(
            f'{self.base_url}/product/{barcode}.json'
        )
        
        if response.status_code == 200:
            data = response.json()
            if data['status'] == 1:
                return self.parse_product(data['product'])
        
        return None
    
    def parse_product(self, product: Dict) -> Dict:
        """
        Parse OpenFoodFacts product data
        """
        
        return {
            'product_name': product.get('product_name'),
            'brands': product.get('brands'),
            'categories': product.get('categories'),
            'ingredients_text': product.get('ingredients_text'),
            'allergens': product.get('allergens'),
            'traces': product.get('traces'),
            'image_url': product.get('image_url'),
            'nutrition_grades': product.get('nutrition_grades')
        }
```

---

## 10. Deployment Architecture

### 10.1 Multi-Region Deployment

```
Primary Region: us-east-1 (N. Virginia)
Secondary Region: us-west-2 (Oregon)

┌─────────────────────────────────────────────────────────────┐
│                      Route 53 (Global)                       │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Health Checks + Latency-based Routing                 │  │
│  └───────────────────┬────────────────────────────────────┘  │
└────────────────────┼─────────────────────────────────────────┘
                     │
       ┌─────────────┴─────────────┐
       │                           │
       ▼                           ▼
┌──────────────┐          ┌──────────────┐
│  us-east-1   │          │  us-west-2   │
│  (Primary)   │◄────────►│  (Secondary) │
└──────┬───────┘          └──────┬───────┘
       │                         │
       │  DynamoDB Global Tables │
       │  RDS Cross-region Repl. │
       │  S3 Cross-region Repl.  │
       └─────────────────────────┘
```

### 10.2 Infrastructure as Code

```yaml
# CloudFormation Template (simplified)
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Allersense Infrastructure'

Parameters:
  Environment:
    Type: String
    AllowedValues: [dev, staging, production]
    Default: production

Resources:
  # VPC
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsHostnames: true
      EnableDnsSupport: true
      Tags:
        - Key: Name
          Value: !Sub '${Environment}-Allersense-vpc'

  # API Gateway
  ApiGateway:
    Type: AWS::ApiGatewayV2::Api
    Properties:
      Name: !Sub '${Environment}-Allersense-api'
      ProtocolType: HTTP
      CorsConfiguration:
        AllowOrigins:
          - '*'
        AllowMethods:
          - GET
          - POST
          - PUT
          - DELETE
        AllowHeaders:
          - '*'

  # Lambda Functions
  ScanProcessorFunction:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: !Sub '${Environment}-scan-processor'
      Runtime: python3.11
      Handler: index.lambda_handler
      Code:
        S3Bucket: !Ref LambdaCodeBucket
        S3Key: scan-processor.zip
      Role: !GetAtt LambdaExecutionRole.Arn
      Environment:
        Variables:
          ENVIRONMENT: !Ref Environment
          DYNAMODB_TABLE: !Ref ScanHistoryTable
      MemorySize: 1024
      Timeout: 30

  # DynamoDB Tables
  UsersTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: !Sub '${Environment}-Users'
      BillingMode: PAY_PER_REQUEST
      AttributeDefinitions:
        - AttributeName: userId
          AttributeType: S
        - AttributeName: email
          AttributeType: S
      KeySchema:
        - AttributeName: userId
          KeyType: HASH
      GlobalSecondaryIndexes:
        - IndexName: email-index
          KeySchema:
            - AttributeName: email
              KeyType: HASH
          Projection:
            ProjectionType: ALL
      SSESpecification:
        SSEEnabled: true
        SSEType: KMS
        KMSMasterKeyId: !Ref KMSKey

  # RDS Database
  RDSInstance:
    Type: AWS::RDS::DBInstance
    Properties:
      DBInstanceIdentifier: !Sub '${Environment}-Allersense-db'
      Engine: postgres
      EngineVersion: '15.4'
      DBInstanceClass: db.r6g.xlarge
      AllocatedStorage: 500
      StorageType: gp3
      StorageEncrypted: true
      KmsKeyId: !Ref KMSKey
      MasterUsername: !Ref DBMasterUsername
      MasterUserPassword: !Ref DBMasterPassword
      MultiAZ: true
      BackupRetentionPeriod: 7
      PreferredBackupWindow: '03:00-04:00'
      PreferredMaintenanceWindow: 'sun:04:00-sun:05:00'

  # S3 Buckets
  ProductImagesBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub '${Environment}-Allersense-product-images'
      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: 'aws:kms'
              KMSMasterKeyID: !Ref KMSKey
      VersioningConfiguration:
        Status: Enabled
      LifecycleConfiguration:
        Rules:
          - Id: TransitionToGlacier
            Status: Enabled
            Transitions:
              - TransitionInDays: 90
                StorageClass: GLACIER
      ReplicationConfiguration:
        Role: !GetAtt S3ReplicationRole.Arn
        Rules:
          - Id: ReplicateToSecondaryRegion
            Status: Enabled
            Destination:
              Bucket: !Sub 'arn:aws:s3:::${Environment}-Allersense-product-images-replica'
              StorageClass: STANDARD

  # KMS Key
  KMSKey:
    Type: AWS::KMS::Key
    Properties:
      Description: 'Allersense encryption key'
      KeyPolicy:
        Version: '2012-10-17'
        Statement:
          - Sid: Enable IAM policies
            Effect: Allow
            Principal:
              AWS: !Sub 'arn:aws:iam::${AWS::AccountId}:root'
            Action: 'kms:*'
            Resource: '*'

Outputs:
  ApiGatewayUrl:
    Description: 'API Gateway URL'
    Value: !GetAtt ApiGateway.ApiEndpoint
    Export:
      Name: !Sub '${Environment}-api-url'
```

### 10.3 CI/CD Pipeline

```yaml
# GitHub Actions Workflow
name: Deploy to Production

on:
  push:
    branches:
      - main

env:
  AWS_REGION: us-east-1
  ECR_REPOSITORY: Allersense

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
          pip install pytest coverage
      
      - name: Run tests
        run: |
          pytest tests/ -v --cov=src --cov-report=xml
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage.xml

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}
      
      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1
      
      - name: Build and push Docker image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
      
      - name: Deploy Lambda functions
        run: |
          aws lambda update-function-code \
            --function-name scan-processor \
            --image-uri ${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY:${{ github.sha }}
      
      - name: Deploy CloudFormation stack
        run: |
          aws cloudformation deploy \
            --template-file infrastructure/cloudformation.yaml \
            --stack-name Allersense-production \
            --parameter-overrides Environment=production \
            --capabilities CAPABILITY_IAM
```

---

## 11. Design Decisions

### 11.1 Technology Choices

#### 11.1.1 Why AWS Bedrock (Claude)?
**Decision:** Use Claude 4.5 Sonnet for allergen detection

**Reasoning:**
- Natural language understanding superior for complex ingredient matching
- Can handle botanical names, synonyms, and multi-language inputs
- Context-aware reasoning for cross-reactivity detection
- Lower hallucination rate than alternatives (critical for medical use)
- Fast inference (< 2 seconds for typical requests)

**Alternatives Considered:**
- OpenAI GPT-4: Higher cost, slower inference
- Custom ML model: Insufficient training data, high development cost
- Rule-based system: Cannot handle synonym variations and new ingredients

#### 11.1.2 Why DynamoDB + RDS?
**Decision:** Hybrid database approach

**Reasoning:**
- DynamoDB for user data: Fast reads/writes, auto-scaling, low latency
- RDS for product catalog: Complex queries, relationships, ACID compliance
- Separation of concerns: User activity vs. product information

**Trade-offs:**
- Increased complexity vs. better performance for each use case
- Need for data synchronization vs. optimized query patterns

#### 11.1.3 Why React Native?
**Decision:** Single codebase for iOS and Android

**Reasoning:**
- 90% code reuse across platforms
- Faster development and feature parity
- Large ecosystem and community support
- Native performance with react-native-vision-camera

**Alternatives Considered:**
- Native (Swift/Kotlin): Higher quality but 2x development time
- Flutter: Less mature ecosystem for medical apps
- PWA: Insufficient camera and offline capabilities

### 11.2 Architecture Patterns

#### 11.2.1 Microservices vs. Monolith
**Decision:** Microservices architecture with separate Lambda functions

**Reasoning:**
- Independent scaling of scan processing (high volume) vs. profile management (low volume)
- Fault isolation: Scan failures don't affect profile access
- Technology flexibility: Python for ML, Node.js for API logic
- Easier deployment: Update services independently

**Trade-offs:**
- Increased operational complexity
- Distributed system challenges (logging, debugging)
- Higher cost at low scale

#### 11.2.2 Synchronous vs. Asynchronous Processing
**Decision:** Asynchronous scan processing with WebSocket updates

**Reasoning:**
- Scan analysis takes 2-5 seconds (too long for synchronous HTTP)
- Better user experience with progress updates
- Allows for retry logic and error handling
- Reduces API Gateway timeout issues

**Implementation:**
1. User initiates scan → Immediate response with scanId
2. Backend processes asynchronously
3. WebSocket sends real-time progress updates
4. Final result pushed to client

#### 11.2.3 CQRS for Read/Write Separation
**Decision:** Separate read and write paths for product data

**Reasoning:**
- Read-heavy workload (10:1 read:write ratio)
- ElastiCache for frequently accessed products
- DynamoDB for fast writes (scan history)
- RDS for complex product queries

**Data Flow:**
- Writes → DynamoDB (scan results) + RDS (new products)
- Reads → ElastiCache → RDS (on cache miss)
- Cache invalidation via SNS pub/sub

### 11.3 Security Design Decisions

#### 11.3.1 Why Field-Level Encryption?
**Decision:** Encrypt sensitive PHI fields separately from table-level encryption

**Reasoning:**
- HIPAA requires protection beyond database encryption
- Limits exposure if database backup is compromised
- Allows for fine-grained access control
- Supports encryption key rotation per field

**Trade-offs:**
- Performance overhead (encrypt/decrypt on each access)
- Increased code complexity
- Higher KMS API costs

#### 11.3.2 OAuth 2.0 for EHR Integration
**Decision:** Use OAuth 2.0 with SMART on FHIR for EHR access

**Reasoning:**
- Industry standard for health data exchange
- Patient-controlled authorization
- Granular scope permissions
- Support from major EHR vendors (Epic, Cerner)

**Implementation:**
- Authorization Code Flow for provider access
- Scopes: patient/*.read, patient/AllergyIntolerance.*
- Token refresh for long-lived sessions

---

## Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | Feb 2026 | Architecture Team | Initial design document |

---

**Document Status:** Final  
**Review Date:** March 2026  
**Approvers:** CTO, VP Engineering, CISO, Medical Director
