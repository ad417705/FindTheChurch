Design Document:
# Find the Church - Design Document

## Document Version Control

| Version | Date | Updated By | Changes |
|---------|------|------------|---------|
| 1.0 | 2024-11-28 | Andrew Dana | Initial design document created |
| 1.1 | 2024-11-28 | Andrew Dana | Updated mission statement, added AWS/Docker infrastructure |

---

## 1. Project Overview

**Mission**: Remove every barrier between people and their church community. When someone says "I can't find a church near me," we make that excuse impossible. We connect seekers to local congregations, making church discovery effortless and accessible.

**Core Value Proposition**: A clean, unbiased church discovery platform that focuses on factual information and location-based search rather than subjective ratings. Whether you're new to faith, relocating, or simply exploring, finding your spiritual home should be simple.

**MVP Approach**: Start with **no user authentication required** - users can browse and search churches without creating accounts. Authentication will be added in Phase 2 for features like favorites and check-ins.

## 2. Technical Stack

### Frontend
- **Framework**: Next.js 14+ (App Router)
- **Styling**: Tailwind CSS
- **Maps**: Mapbox GL JS or Google Maps API
- **State Management**: React Context/Zustand (for simple state)

### Backend & Infrastructure
- **Framework**: Next.js API Routes (serverless functions)
- **Database**: PostgreSQL via AWS RDS
- **Authentication**: AWS Cognito (or Supabase Auth initially)
- **File Storage**: AWS S3 (for church images)
- **Containerization**: Docker
- **Container Orchestration**: AWS ECS (Elastic Container Service) or EKS (Kubernetes)
- **API Gateway**: AWS API Gateway (optional for advanced routing)
- **CDN**: AWS CloudFront
- **Monitoring**: AWS CloudWatch

### DevOps & Deployment
- **CI/CD**: GitHub Actions → AWS ECR (Elastic Container Registry) → ECS
- **Infrastructure as Code**: AWS CDK or Terraform (optional)
- **Environment Management**: Docker Compose (local), AWS ECS (production)

### Why This Stack?
- Next.js provides SSR/SSG for SEO (important for church discovery)
- Docker ensures consistent environments across development and production
- AWS offers enterprise-grade scalability and a complete ecosystem
- Learning AWS, Docker, and containers provides industry-standard DevOps skills
- PostgreSQL on RDS with PostGIS extension for geospatial queries
- All components integrate well and provide production-ready infrastructure

## 3. Core Features (MVP)

### Phase 1: Basic Discovery (No Authentication Required)
1. **Browse churches without login** - Public access to all church listings
2. **Location-based search** - Find churches near user's current location
3. **State/City search** - Explore churches in different regions nationwide
4. **Church detail pages** - View comprehensive church information
5. **Simple filtering** - By denomination, service times, size
6. **Map integration** - Interactive map showing church locations

### Phase 2: User Engagement (Future - Requires Authentication)
1. **User accounts** - Optional registration to unlock additional features
2. **Save favorite churches** - Bookmark churches for later reference
3. **Check-ins** - Users can mark attendance (factual, not ratings)
4. **Personal history** - Track churches visited

### Phase 3: Church Management (Future)
1. **Church self-registration** - Allow churches to add themselves to the platform
2. **Church admin claims** - Existing churches can claim and manage their listings
3. **Verified data** - Badge system for churches that verify their information
4. **Updated service times** - Churches can update their own schedules

## 4. Database Schema

### Tables

#### `churches`
```sql
id                UUID PRIMARY KEY
name              VARCHAR(255) NOT NULL
denomination      VARCHAR(100)
address           TEXT NOT NULL
city              VARCHAR(100) NOT NULL
state             VARCHAR(2) NOT NULL
zip_code          VARCHAR(10)
country           VARCHAR(50) DEFAULT 'USA'
latitude          DECIMAL(10, 8) NOT NULL
longitude         DECIMAL(11, 8) NOT NULL
phone             VARCHAR(20)
email             VARCHAR(255)
website           VARCHAR(255)
description       TEXT
founded_year      INTEGER
avg_attendance    INTEGER
service_times     JSONB -- {sunday: ["9am", "11am"], wednesday: ["7pm"]}
languages         TEXT[] -- ["English", "Spanish"]
image_url         TEXT
verified          BOOLEAN DEFAULT FALSE
created_at        TIMESTAMP DEFAULT NOW()
updated_at        TIMESTAMP DEFAULT NOW()
```

#### `users`
```sql
id                UUID PRIMARY KEY (from Supabase Auth)
email             VARCHAR(255) UNIQUE NOT NULL
full_name         VARCHAR(255)
location_city     VARCHAR(100)
location_state    VARCHAR(2)
created_at        TIMESTAMP DEFAULT NOW()
updated_at        TIMESTAMP DEFAULT NOW()
```

#### `favorites` (Future Phase)
```sql
id                UUID PRIMARY KEY
user_id           UUID REFERENCES users(id)
church_id         UUID REFERENCES churches(id)
created_at        TIMESTAMP DEFAULT NOW()
UNIQUE(user_id, church_id)
```

#### `check_ins` (Future Phase)
```sql
id                UUID PRIMARY KEY
user_id           UUID REFERENCES users(id)
church_id         UUID REFERENCES churches(id)
check_in_date     DATE NOT NULL
created_at        TIMESTAMP DEFAULT NOW()
```

### Indexes
```sql
CREATE INDEX idx_churches_location ON churches USING GIST (
  geography(ST_MakePoint(longitude, latitude))
);
CREATE INDEX idx_churches_state_city ON churches(state, city);
CREATE INDEX idx_churches_denomination ON churches(denomination);
```

## 5. Key API Endpoints

### Church Discovery
- `GET /api/churches` - List churches with pagination
  - Query params: `lat`, `lng`, `radius`, `state`, `city`, `denomination`, `page`
- `GET /api/churches/[id]` - Get single church details
- `GET /api/churches/nearby` - Find churches near coordinates
- `GET /api/churches/search` - Text search by name or location

### User Management (Future)
- `POST /api/auth/signup` - Create account
- `POST /api/auth/login` - Authenticate
- `GET /api/users/favorites` - Get user's saved churches
- `POST /api/users/favorites` - Save a church

### Admin (Future)
- `POST /api/churches` - Add new church (admin or claimed)
- `PUT /api/churches/[id]` - Update church info
- `POST /api/churches/[id]/claim` - Church admin claims listing

## 6. User Flows

### Primary Flow: Finding a Church
1. User lands on homepage
2. System detects location OR user enters city/state
3. Map displays churches in area with list view
4. User filters by denomination/service time
5. User clicks church to view details
6. User gets directions or visits website

### Secondary Flow: Exploring New Areas
1. User searches for "Dallas, TX" or "California"
2. Map updates to show churches in that region
3. User browses and explores different churches
4. User can save churches for later (with account)

## 7. Avoiding Bias: Design Decisions

Instead of ratings/reviews, focus on **factual, objective information**:

✅ **Include**:
- Check-in counts (shows activity level)
- Verified attendance numbers (from church admins)
- Service times and languages offered
- Denomination and doctrinal statements
- Photos and basic descriptions
- Links to church's own website/social media

❌ **Exclude**:
- Star ratings
- Written reviews
- Comparison rankings
- "Best church" algorithms

**Alternative**: Consider a "Verified Visitor" badge for users who've checked in 3+ times, adding credibility without subjective opinions.

## 8. Development Roadmap

### Week 1-2: Local Development Setup
- Initialize Next.js 14+ project with App Router
- Create Dockerfile for Next.js application
- Set up Docker Compose for local development (Next.js + PostgreSQL)
- Configure PostgreSQL database with PostGIS extension
- Set up Google Places API credentials
- Configure environment variables
- Test containerized local environment

### Week 3-4: AWS Infrastructure Setup
- Create AWS account and configure IAM roles
- Set up AWS RDS PostgreSQL instance
- Configure AWS S3 bucket for image storage
- Set up AWS ECR (Elastic Container Registry) for Docker images
- Create initial ECS cluster and task definitions
- Configure AWS CloudWatch for logging
- Set up staging and production environments

### Week 5-6: Data Layer & API Integration
- Build API route to fetch churches from Google Places API
- Create caching strategy to store results in RDS
- Implement geolocation functionality
- Build church search endpoints (by location, city, state)
- Add pagination and filtering logic
- Containerize API services

### Week 7-8: Frontend & Map Integration
- Build homepage with map component (Mapbox or Google Maps)
- Create church list view with cards
- Implement church detail pages
- Add search and filter UI
- Make design responsive for mobile
- Test in Docker container locally

### Week 9-10: CI/CD & Deployment
- Set up GitHub Actions for automated builds
- Configure pipeline: Build → Push to ECR → Deploy to ECS
- Set up AWS CloudFront for CDN
- Configure custom domain and SSL certificates
- Add loading states and error handling
- Optimize for SEO (metadata, sitemap)
- Deploy MVP to production

### Week 11-12: Monitoring & Polish
- Set up AWS CloudWatch dashboards
- Configure alerts for errors and performance issues
- Load testing with Docker containers
- Final bug fixes and optimizations
- Documentation for deployment process

### Post-MVP: Phase 2 Features
- Add AWS Cognito for user authentication
- Build favorites and check-in features
- Create user profile pages
- Implement church claim system (Phase 3)
- Consider AWS Lambda for background jobs

## 9. Next.js Development Focus Areas

Key Next.js concepts essential for this project:

1. **App Router basics** - File-based routing in `app/` directory
2. **Server vs Client Components** - "use client" directive
3. **Data fetching** - Server-side and client-side patterns
4. **API Routes** - Building endpoints in `app/api/`
5. **Metadata & SEO** - Dynamic metadata for church pages

## 10. AWS & Docker Learning Focus Areas

Key AWS and containerization concepts for this project:

### Docker Fundamentals
1. **Dockerfile creation** - Building optimized images for Next.js
2. **Docker Compose** - Managing multi-container local development
3. **Container networking** - Connecting services together
4. **Volume management** - Persisting data and managing file uploads
5. **Multi-stage builds** - Optimizing image size and security

### AWS Core Services
1. **ECS (Elastic Container Service)** - Running and managing Docker containers
2. **ECR (Elastic Container Registry)** - Storing Docker images
3. **RDS (Relational Database Service)** - Managed PostgreSQL database
4. **S3 (Simple Storage Service)** - Object storage for images and static assets
5. **CloudFront** - CDN for fast global content delivery
6. **CloudWatch** - Logging, monitoring, and alerts
7. **IAM (Identity and Access Management)** - Security and permissions
8. **VPC (Virtual Private Cloud)** - Network isolation and security

### CI/CD Pipeline
1. **GitHub Actions** - Automated testing and builds
2. **AWS ECR push** - Automated image deployment
3. **ECS deployment** - Zero-downtime rolling updates
4. **Environment management** - Staging vs production configurations

## 11. Open Questions to Decide

### Church Data Strategy

**Option 1: Google Places API (Recommended for MVP)**
- **Pros**: 
  - Access to existing church data immediately
  - No manual data entry needed
  - Includes addresses, phone numbers, websites, photos
  - 10,000 free API calls per month
  - Users can discover churches right away
- **Cons**: 
  - Costs money after free tier ($17 per 1,000 requests)
  - Limited control over data accuracy
  - May have incomplete information for some churches
- **Best for**: Getting started quickly with real data

**Option 2: Manual Curation**
- **Pros**: 
  - Complete control over data quality
  - Can ensure accurate, detailed information
  - No ongoing API costs
- **Cons**: 
  - Time-intensive to build initial database
  - Slow to scale
  - Requires significant upfront work
- **Best for**: Long-term goal after proving concept

**Option 3: Hybrid Approach (Recommended Long-Term)**
- Start with Google Places API to populate initial database
- Cache results in your database to reduce API costs
- Allow churches to claim and enhance their listings (Phase 3)
- Gradually build proprietary database with richer information
- Eventually reduce dependence on Google Places

### Other Decisions
1. What geographic region to launch in first? (Start local, then expand?)
2. Mobile app in the future, or web-only initially?
3. Revenue model: Free forever, premium features for churches, or minimal ads?
4. How will you handle churches that close or merge?
5. Should users be able to suggest edits to church information?

---
