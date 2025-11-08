# MatchMyShade - System Architecture

## Table of Contents
1. [Overview](#overview)
2. [Technology Stack](#technology-stack)
3. [System Components](#system-components)
4. [Architecture Diagram](#architecture-diagram)
5. [Data Flow](#data-flow)
6. [AI/ML Pipeline](#aiml-pipeline)
7. [Database Schema](#database-schema)
8. [API Endpoints](#api-endpoints)
9. [Security & Privacy](#security--privacy)
10. [Scalability Considerations](#scalability-considerations)

---

## Overview

MatchMyShade is a mobile-first application built on a three-tier architecture: presentation layer (mobile app), application layer (backend services), and data layer (databases and file storage). The system leverages AI/ML for real-time skin tone analysis and matching algorithms.

### Core Technical Goals
- **Accuracy:** 95%+ match accuracy for skin tone detection
- **Performance:** <3 second scan-to-result time
- **Scalability:** Support 100K+ concurrent users
- **Reliability:** 99.9% uptime SLA
- **Privacy:** End-to-end encryption for user images

---

## Technology Stack

### Frontend (Mobile)
- **Framework:** React Native 0.73+
  - Cross-platform development (iOS & Android)
  - Shared codebase (~90% code reuse)
  - Native performance with JavaScript flexibility
  
- **State Management:** Redux Toolkit
  - Centralized state management
  - Predictable state updates
  - DevTools for debugging

- **UI Components:** React Native Paper + Custom components
  - Material Design principles
  - Accessibility-first design
  - Dark mode support

- **Camera Integration:** react-native-vision-camera
  - High-performance camera access
  - Frame processing capabilities
  - ML Kit integration support

### Backend
- **Runtime:** Node.js 20 LTS
  - Non-blocking I/O for high concurrency
  - Rich ecosystem of packages
  - JavaScript/TypeScript consistency with frontend

- **Framework:** Express.js 4.18+
  - Lightweight and flexible
  - Robust routing
  - Extensive middleware support

- **API Style:** RESTful APIs + GraphQL (future)
  - RESTful for MVP simplicity
  - GraphQL for complex queries in future iterations

- **Authentication:** JWT + OAuth 2.0
  - Stateless authentication
  - Social login support (Google, Apple)
  - Refresh token rotation

### AI/ML Stack
- **On-Device ML:** TensorFlow Lite 2.14+
  - Runs on-device for privacy
  - Low latency inference
  - Offline capability

- **Model Training:** TensorFlow/Keras (Cloud)
  - Custom CNN for skin tone classification
  - Transfer learning from pretrained models
  - Regular retraining with user feedback

- **Image Processing:** OpenCV 4.8+
  - Face detection and alignment
  - Color space conversions
  - Lighting normalization

- **Computer Vision:** MediaPipe
  - Face mesh detection (468 landmarks)
  - Real-time performance
  - Cross-platform support

### Databases

#### Primary Database: PostgreSQL 15+
**Use Case:** User accounts, matches, preferences, transactions

**Why PostgreSQL:**
- ACID compliance for transactional data
- Rich indexing for fast queries
- JSON support for flexible schemas
- Proven reliability at scale

**Key Tables:**
- Users
- UserProfiles
- ScanHistory
- SavedMatches
- Subscriptions

#### Secondary Database: MongoDB 6.0+
**Use Case:** Product catalog, brand information, dynamic content

**Why MongoDB:**
- Flexible schema for diverse product data
- High read performance
- Easy horizontal scaling
- Rich query language

**Key Collections:**
- Brands
- Products
- FoundationShades
- Reviews

#### Cache Layer: Redis 7.2+
**Use Case:** Session management, API rate limiting, frequently accessed data

**Why Redis:**
- In-memory speed
- Built-in data structures
- Pub/sub for real-time features
- Automatic expiration

### Cloud Infrastructure
- **Provider:** AWS
- **Compute:** ECS (Fargate) for containerized backend
- **Storage:** S3 for user images (encrypted at rest)
- **CDN:** CloudFront for global content delivery
- **Functions:** Lambda for background jobs
- **Queue:** SQS for asynchronous processing
- **Monitoring:** CloudWatch + DataDog

---

## System Components

### 1. Mobile Application
**Responsibilities:**
- User interface and experience
- Camera capture and preview
- On-device ML inference
- Local caching
- Push notifications

**Key Modules:**
- Authentication Module
- Camera & Scan Module
- Match Results Module
- Brand Browser Module
- User Profile Module
- Feedback Module

### 2. API Gateway
**Responsibilities:**
- Request routing
- Rate limiting
- Authentication verification
- Request/response transformation
- API versioning

**Technology:** AWS API Gateway or custom Express middleware

### 3. Backend Services

#### Authentication Service
- User registration and login
- JWT token generation and validation
- Password reset flows
- OAuth integration

#### Scan Service
- Receive and validate scan images
- Trigger ML processing
- Store scan results
- Return matched shades

#### Product Service
- CRUD operations for products
- Brand management
- Search and filtering
- Recommendation engine

#### User Service
- Profile management
- Preferences and settings
- Saved matches
- Scan history

#### Payment Service
- Subscription management
- Payment processing (Stripe integration)
- Invoice generation
- Billing cycles

### 4. ML Service
**Responsibilities:**
- Model serving
- Batch predictions
- Model versioning
- A/B testing for model improvements

**Components:**
- Model Registry (MLflow)
- Inference Engine (TensorFlow Serving)
- Training Pipeline (Airflow)

### 5. Admin Dashboard
**Technology:** React.js with Next.js

**Features:**
- User management
- Product catalog management
- Analytics and metrics
- Model performance monitoring
- Content moderation

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                         MOBILE APP (React Native)                │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────────────┐    │
│  │  Camera  │  │   Scan   │  │  Browse  │  │   Profile   │    │
│  │  Module  │  │  Module  │  │  Module  │  │   Module    │    │
│  └──────────┘  └──────────┘  └──────────┘  └─────────────┘    │
│         │             │              │              │            │
│         └─────────────┴──────────────┴──────────────┘            │
│                           │                                       │
│                  ┌────────▼────────┐                            │
│                  │  TensorFlow Lite │ (On-device ML)            │
│                  └─────────────────┘                            │
└──────────────────────────┬──────────────────────────────────────┘
                           │ HTTPS/REST APIs
                           │
┌──────────────────────────▼──────────────────────────────────────┐
│                      AWS API GATEWAY                             │
│              (Rate Limiting, Auth, Routing)                      │
└──────────────────────────┬──────────────────────────────────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
┌───────▼────────┐  ┌──────▼──────┐  ┌──────▼──────┐
│  Auth Service  │  │ Scan Service│  │User Service │
│   (Node.js)    │  │  (Node.js)  │  │  (Node.js)  │
└───────┬────────┘  └──────┬──────┘  └──────┬──────┘
        │                  │                  │
        └──────────────────┼──────────────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
┌───────▼────────┐  ┌──────▼──────┐  ┌──────▼──────┐
│   PostgreSQL   │  │   MongoDB   │  │    Redis    │
│  (User Data)   │  │  (Products) │  │   (Cache)   │
└────────────────┘  └─────────────┘  └─────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                    SUPPORTING SERVICES                           │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────────────┐    │
│  │AWS S3    │  │CloudFront│  │AWS Lambda│  │  DataDog    │    │
│  │(Storage) │  │  (CDN)   │  │ (Jobs)   │  │(Monitoring) │    │
│  └──────────┘  └──────────┘  └──────────┘  └─────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

---

## Data Flow

### User Scan Flow

1. **User Opens Camera**
   - App requests camera permissions
   - Initializes react-native-vision-camera
   - Displays real-time preview with face detection overlay

2. **User Captures Photo**
   - Image captured in high resolution
   - Compressed for processing (JPEG, ~500KB)
   - Face detection validates presence of face

3. **On-Device Pre-Processing**
   - Face alignment and cropping
   - Lighting normalization
   - Color space conversion (RGB to LAB)

4. **On-Device ML Inference**
   - TensorFlow Lite model runs on device
   - Extracts skin tone features
   - Classifies undertone
   - Generates tone vector (128 dimensions)

5. **API Request to Backend**
   ```
   POST /api/v1/scans
   Headers: Authorization: Bearer <jwt_token>
   Body: {
     "toneVector": [0.234, 0.456, ...],
     "undertone": "warm",
     "metadata": {
       "lighting": "natural",
       "confidence": 0.94
     }
   }
   ```

6. **Backend Processing**
   - Validates request and authentication
   - Queries MongoDB for matching shades
   - Applies matching algorithm (cosine similarity)
   - Ranks results by compatibility score
   - Stores scan in PostgreSQL

7. **Response to App**
   ```json
   {
     "scanId": "scan_abc123",
     "matches": [
       {
         "brand": "Fenty Beauty",
         "product": "Pro Filt'r Foundation",
         "shade": "390",
         "shadeName": "Deep with warm undertones",
         "matchScore": 0.97,
         "imageUrl": "https://cdn.../fenty-390.jpg"
       }
     ],
     "totalMatches": 15
   }
   ```

8. **Display Results**
   - App renders matched shades
   - Virtual try-on available
   - Option to save favorites
   - Link to purchase

---

## AI/ML Pipeline

### Training Pipeline

1. **Data Collection**
   - User-uploaded images (with consent)
   - User feedback on match accuracy
   - Manually labeled dataset (diverse skin tones)
   - Synthetic data augmentation

2. **Data Preprocessing**
   - Face detection and alignment
   - Lighting normalization
   - Data augmentation (rotation, brightness)
   - Train/validation/test split (70/15/15)

3. **Model Architecture**
   ```
   Custom CNN Architecture:
   - Input: 224x224x3 RGB image
   - Base: MobileNetV3 (pretrained on ImageNet)
   - Custom Layers:
     * Global Average Pooling
     * Dense(256, activation='relu')
     * Dropout(0.3)
     * Dense(128, activation='relu') → Tone Vector
     * Dense(7, activation='softmax') → Tone Classification
     * Dense(3, activation='softmax') → Undertone Classification
   ```

4. **Training Process**
   - Framework: TensorFlow 2.14
   - Optimizer: Adam (lr=0.001)
   - Loss: Categorical Crossentropy + Custom Similarity Loss
   - Epochs: 50 with early stopping
   - Batch size: 32
   - Hardware: AWS p3.2xlarge (V100 GPU)

5. **Model Evaluation**
   - Accuracy metrics per skin tone category
   - Confusion matrix analysis
   - Bias testing across demographics
   - A/B testing against previous version

6. **Model Deployment**
   - Convert to TensorFlow Lite (quantized)
   - Size optimization (<10MB)
   - Deploy to mobile apps via OTA update
   - Gradual rollout (10% → 50% → 100%)

### Matching Algorithm

```python
def match_foundation_shades(tone_vector, undertone, top_k=15):
    """
    Matches user skin tone to foundation shades
    
    Args:
        tone_vector: 128-dim feature vector from ML model
        undertone: 'warm', 'cool', or 'neutral'
        top_k: number of top matches to return
    
    Returns:
        List of matched shades with scores
    """
    # 1. Filter shades by undertone compatibility
    compatible_shades = filter_by_undertone(undertone)
    
    # 2. Calculate cosine similarity
    similarities = cosine_similarity(tone_vector, compatible_shades.vectors)
    
    # 3. Apply undertone boost
    scores = similarities * undertone_boost_factor(undertone, compatible_shades)
    
    # 4. Rank and return top K
    top_matches = argsort(scores)[:top_k]
    
    return compatible_shades[top_matches]
```

---

## Database Schema

### PostgreSQL Schema

```sql
-- Users table
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    full_name VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_verified BOOLEAN DEFAULT FALSE,
    subscription_tier VARCHAR(50) DEFAULT 'free'
);

-- User profiles
CREATE TABLE user_profiles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    skin_tone_category VARCHAR(50),
    undertone VARCHAR(20),
    skin_type VARCHAR(50),
    preferences JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Scan history
CREATE TABLE scans (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    tone_vector FLOAT[128],
    undertone VARCHAR(20),
    confidence_score FLOAT,
    image_url VARCHAR(500),
    metadata JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Saved matches
CREATE TABLE saved_matches (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    scan_id UUID REFERENCES scans(id) ON DELETE SET NULL,
    brand_id VARCHAR(100),
    product_id VARCHAR(100),
    shade_code VARCHAR(50),
    match_score FLOAT,
    notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Feedback
CREATE TABLE feedback (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    scan_id UUID REFERENCES scans(id) ON DELETE CASCADE,
    rating INTEGER CHECK (rating BETWEEN 1 AND 5),
    was_accurate BOOLEAN,
    comments TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Subscriptions
CREATE TABLE subscriptions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    tier VARCHAR(50) NOT NULL,
    status VARCHAR(50) DEFAULT 'active',
    stripe_subscription_id VARCHAR(255),
    current_period_start TIMESTAMP,
    current_period_end TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### MongoDB Schema

```javascript
// Brands collection
{
  "_id": "brand_fenty_beauty",
  "name": "Fenty Beauty",
  "slug": "fenty-beauty",
  "description": "Beauty for all",
  "logo_url": "https://...",
  "website": "https://fentybeauty.com",
  "is_inclusive": true,
  "shade_range_count": 50,
  "created_at": ISODate("2025-01-01T00:00:00Z")
}

// Products collection
{
  "_id": "prod_fenty_profilt",
  "brand_id": "brand_fenty_beauty",
  "name": "Pro Filt'r Soft Matte Foundation",
  "category": "foundation",
  "type": "liquid",
  "finish": "matte",
  "coverage": "full",
  "price": {
    "amount": 40.00,
    "currency": "USD"
  },
  "shades": [
    {
      "code": "390",
      "name": "390 Deep with warm undertones",
      "undertone": "warm",
      "tone_vector": [0.234, 0.456, ...], // 128 dimensions
      "hex_color": "#8B5A3C",
      "images": ["https://..."],
      "in_stock": true
    }
  ],
  "rating": 4.7,
  "review_count": 12453,
  "created_at": ISODate("2025-01-01T00:00:00Z")
}

// Reviews collection
{
  "_id": ObjectId("..."),
  "product_id": "prod_fenty_profilt",
  "shade_code": "390",
  "user_id": "user_uuid",
  "rating": 5,
  "review_text": "Perfect match for my skin!",
  "skin_tone": "deep",
  "undertone": "warm",
  "helpful_count": 23,
  "created_at": ISODate("2025-01-15T10:30:00Z")
}
```

---

## API Endpoints

### Authentication APIs

```
POST   /api/v1/auth/register
POST   /api/v1/auth/login
POST   /api/v1/auth/logout
POST   /api/v1/auth/refresh
POST   /api/v1/auth/forgot-password
POST   /api/v1/auth/reset-password
GET    /api/v1/auth/verify-email/:token
```

### Scan APIs

```
POST   /api/v1/scans              # Create new scan
GET    /api/v1/scans/:id          # Get scan details
GET    /api/v1/scans              # Get user's scan history
DELETE /api/v1/scans/:id          # Delete scan
POST   /api/v1/scans/:id/feedback # Submit feedback
```

### Product APIs

```
GET    /api/v1/brands             # List all brands
GET    /api/v1/brands/:id         # Get brand details
GET    /api/v1/products           # Search/filter products
GET    /api/v1/products/:id       # Get product details
GET    /api/v1/products/:id/shades # Get product shades
```

### User APIs

```
GET    /api/v1/users/me           # Get current user
PATCH  /api/v1/users/me           # Update user profile
DELETE /api/v1/users/me           # Delete account
GET    /api/v1/users/me/saved-matches # Get saved matches
POST   /api/v1/users/me/saved-matches # Save a match
DELETE /api/v1/users/me/saved-matches/:id # Remove saved match
```

### Subscription APIs

```
POST   /api/v1/subscriptions      # Create subscription
GET    /api/v1/subscriptions      # Get subscription status
POST   /api/v1/subscriptions/cancel # Cancel subscription
POST   /api/v1/subscriptions/resume # Resume subscription
```

---

## Security & Privacy

### Authentication & Authorization
- JWT tokens with 15-minute expiry
- Refresh tokens with 7-day expiry
- Secure password hashing (bcrypt, 12 rounds)
- OAuth 2.0 for social login
- Rate limiting on auth endpoints

### Data Protection
- **Images:** Encrypted at rest (AES-256) in S3
- **Transmission:** TLS 1.3 for all API calls
- **Database:** Encrypted at rest (AWS RDS encryption)
- **PII:** Minimal collection, encrypted in database
- **Image Retention:** 90-day auto-deletion policy

### Privacy Measures
- **On-Device ML:** Inference happens on user's device
- **Opt-in Data:** Users control if images are used for training
- **Anonymization:** All training data is anonymized
- **GDPR Compliance:** Right to deletion, data export
- **No Third-Party Sharing:** User images never shared

### Security Best Practices
- Input validation and sanitization
- SQL injection prevention (parameterized queries)
- XSS protection (Content Security Policy)
- CORS configuration
- API versioning for backward compatibility
- Regular security audits
- Dependency vulnerability scanning

---

## Scalability Considerations

### Current MVP Scale
- **Users:** Up to 10K concurrent users
- **Requests:** 100 req/sec
- **Database:** Single PostgreSQL instance (db.t3.medium)
- **Backend:** 2-4 ECS tasks (2 vCPU, 4GB RAM each)

### Growth Strategy

#### Phase 1: 10K → 100K users
- Enable PostgreSQL read replicas
- Implement Redis caching layer
- Scale ECS tasks horizontally (auto-scaling)
- CDN for static assets

#### Phase 2: 100K → 1M users
- Database sharding (by user_id)
- MongoDB Atlas with auto-scaling
- ElastiCache Redis clusters
- Multi-region deployment
- DynamoDB for high-velocity data

#### Phase 3: 1M+ users
- Microservices architecture
- Event-driven architecture (Kafka)
- GraphQL Federation
- Global CDN with edge computing
- Dedicated ML infrastructure

### Performance Optimization
- Database indexing strategy
- Query optimization (EXPLAIN ANALYZE)
- Connection pooling (pg-pool, mongoose)
- Response compression (gzip)
- Image lazy loading
- Pagination for large datasets
- Background job processing (Bull queue)

---

## Technical Feasibility

### Why This Architecture Works

1. **Proven Technologies**
   - React Native: Used by Instagram, Discord, Shopify
   - Node.js: Handles 10K+ req/sec with proper scaling
   - PostgreSQL: Battle-tested for ACID transactions
   - TensorFlow Lite: Industry standard for mobile ML

2. **Cost-Effective MVP**
   - AWS Free Tier covers initial development
   - Estimated monthly cost: $200-500 for MVP
   - Pay-as-you-grow model

3. **Fast Development**
   - React Native = single codebase for iOS + Android
   - Rich ecosystem of libraries
   - JavaScript throughout = easier hiring

4. **ML Feasibility**
   - Skin tone classification is a solved problem
   - Similar apps (Google Pixel camera) prove viability
   - On-device ML is mature technology

5. **Scalability Path**
   - Clear migration path from monolith to microservices
   - Cloud-native from day one
   - Horizontal scaling built-in

---

## Next Steps

1. **Development Setup**
   - Initialize React Native project
   - Set up backend boilerplate
   - Configure AWS services
   - Establish CI/CD pipeline

2. **MVP Timeline** (12 weeks)
   - Weeks 1-2: Setup and infrastructure
   - Weeks 3-5: Backend APIs
   - Weeks 6-9: Mobile app development
   - Weeks 10-11: ML model integration
   - Week 12: Testing and deployment

3. **Success Criteria**
   - 95% scan success rate
   - <3s scan-to-result time
   - 500+ products in database
   - 10+ inclusive brands

---
 4. **Documentation**
     - Maintain comprehensive technical documentation
     - Keep architecture diagrams updated
     
**Document Version:** 1.0  
**Last Updated:** November 8 2025  
**Author:** MatchMyShade Technical Team
