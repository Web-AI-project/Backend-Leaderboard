# 🎮 Mobile Game Leaderboard API

A scalable, production-ready RESTful API for managing mobile game leaderboards with JWT authentication, role-based access control, rate limiting, and comprehensive logging.

## 📋 Table of Contents

- [Features](#-features)
- [Technology Stack](#️-technology-stack)
- [Architecture](#️-architecture)
- [Project Structure](#-project-structure)
- [Getting Started](#-getting-started)
- [API Documentation](#-api-documentation)
- [Flow Diagrams](#-flow-diagrams)
- [Environment Configuration](#️-environment-configuration)
- [Testing](#-testing)
- [CI/CD Pipeline](#-cicd-pipeline)
- [Deployment](#-deployment)
- [Security](#-security)
- [Performance Optimization](#-performance-optimization)
- [Contributing](#-contributing)
- [License](#-license)
- [Support](#-support)

## ✨ Features

### Core Features
- ✅ **Score Submission**: Authenticated users can submit their game scores
- ✅ **Leaderboard**: Public endpoint to view top 10 (or custom) high scores
- ✅ **JWT Authentication**: Secure token-based authentication
- ✅ **Role-Based Authorization**: Admin and User roles with different permissions
- ✅ **Rate Limiting**: Prevents API abuse (10 requests per 60 seconds on score submission)
- ✅ **Request Logging**: Comprehensive logging with IP, method, endpoint, and status
- ✅ **Input Validation**: Request validation using class-validator
- ✅ **Error Handling**: Centralized error handling with proper HTTP status codes

### Technical Features
- ✅ **TypeScript**: Full type safety throughout the application
- ✅ **NestJS Framework**: Modern, scalable Node.js framework
- ✅ **PostgreSQL Database**: Robust SQL database with TypeORM
- ✅ **Docker Support**: Containerized application with Docker Compose
- ✅ **Unit Tests**: Comprehensive test coverage for services
- ✅ **Multi-Environment**: Dev/Staging/Prod configuration profiles
- ✅ **CI/CD Pipeline**: Automated testing, building, and deployment

## 🛠️ Technology Stack

| Layer | Technology |
|-------|-----------|
| **Framework** | NestJS 10.x |
| **Language** | TypeScript 5.x |
| **Runtime** | Node.js 18.x |
| **Database** | PostgreSQL 15 |
| **ORM** | TypeORM 0.3.x |
| **Authentication** | JWT (Passport) |
| **Validation** | class-validator |
| **Logging** | Winston |
| **Rate Limiting** | @nestjs/throttler |
| **Testing** | Jest |
| **Containerization** | Docker & Docker Compose |
| **CI/CD** | GitHub Actions |

## 🏗️ Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         Mobile Game Client                       │
└────────────────────────────┬────────────────────────────────────┘
                             │ HTTP/HTTPS
                             │
┌────────────────────────────▼────────────────────────────────────┐
│                     API Gateway / Load Balancer                  │
└────────────────────────────┬────────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────────┐
│                       NestJS Application                         │
│  ┌────────────────────────────────────────────────────────┐    │
│  │  Global Middleware & Interceptors                       │    │
│  │  - Logging Interceptor                                  │    │
│  │  - Validation Pipe                                      │    │
│  │  - Rate Limiting Guard                                  │    │
│  └────────────────────────────────────────────────────────┘    │
│                             │                                    │
│  ┌─────────────┬────────────┴────────────┬──────────────┐      │
│  │             │                         │              │      │
│  ▼             ▼                         ▼              ▼      │
│ ┌────────┐  ┌────────┐               ┌────────┐    ┌──────┐  │
│ │  Auth  │  │ Scores │               │ JWT    │    │Roles │  │
│ │ Module │  │ Module │               │ Guard  │    │Guard │  │
│ └────────┘  └────────┘               └────────┘    └──────┘  │
│      │            │                       │             │      │
│      └────────────┴───────────────────────┴─────────────┘      │
│                             │                                    │
└────────────────────────────┬────────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────────┐
│                     PostgreSQL Database                          │
│  ┌──────────────────┐         ┌──────────────────┐             │
│  │   Users Table    │         │  Scores Table    │             │
│  │  - id (UUID)     │         │  - id (UUID)     │             │
│  │  - username      │◄────────┤  - userId (FK)   │             │
│  │  - password      │         │  - score         │             │
│  │  - role          │         │  - createdAt     │             │
│  │  - isActive      │         └──────────────────┘             │
│  │  - createdAt     │                                           │
│  │  - updatedAt     │                                           │
│  └──────────────────┘                                           │
└─────────────────────────────────────────────────────────────────┘
```

### Request Flow Architecture

```
┌──────────┐
│  Client  │
└────┬─────┘
     │
     │ 1. HTTP Request
     ▼
┌─────────────────────────────────────────────────────────┐
│                  NestJS Application                      │
│                                                          │
│  ┌────────────────────────────────────────────────┐    │
│  │  Step 1: Global Middleware & Interceptors      │    │
│  │  - LoggingInterceptor (logs request)           │    │
│  │  - ValidationPipe (validates DTO)              │    │
│  └───────────────────┬────────────────────────────┘    │
│                      │                                   │
│  ┌───────────────────▼────────────────────────────┐    │
│  │  Step 2: Rate Limiting                         │    │
│  │  - ThrottlerGuard checks rate limits           │    │
│  │  - Allows/Rejects based on quota               │    │
│  └───────────────────┬────────────────────────────┘    │
│                      │                                   │
│  ┌───────────────────▼────────────────────────────┐    │
│  │  Step 3: Authentication (if required)          │    │
│  │  - JwtAuthGuard validates JWT token            │    │
│  │  - Extracts user from token payload            │    │
│  └───────────────────┬────────────────────────────┘    │
│                      │                                   │
│  ┌───────────────────▼────────────────────────────┐    │
│  │  Step 4: Authorization (if required)           │    │
│  │  - RolesGuard checks user permissions          │    │
│  │  - ScoreOwnershipGuard validates ownership     │    │
│  └───────────────────┬────────────────────────────┘    │
│                      │                                   │
│  ┌───────────────────▼────────────────────────────┐    │
│  │  Step 5: Controller Handler                    │    │
│  │  - Routes request to appropriate method        │    │
│  └───────────────────┬────────────────────────────┘    │
│                      │                                   │
│  ┌───────────────────▼────────────────────────────┐    │
│  │  Step 6: Service Layer                         │    │
│  │  - Business logic execution                    │    │
│  │  - Database operations via TypeORM             │    │
│  └───────────────────┬────────────────────────────┘    │
│                      │                                   │
│  ┌───────────────────▼────────────────────────────┐    │
│  │  Step 7: Response                              │    │
│  │  - Transform data (remove sensitive fields)    │    │
│  │  - LoggingInterceptor logs response            │    │
│  └───────────────────┬────────────────────────────┘    │
└──────────────────────┼──────────────────────────────────┘
                       │
                       │ HTTP Response
                       ▼
                  ┌──────────┐
                  │  Client  │
                  └──────────┘
```

## 📁 Project Structure

```
leaderboard/
├── .github/
│   └── workflows/
│       └── ci-cd.yml              # CI/CD pipeline configuration
├── src/
│   ├── auth/                      # Authentication module
│   │   ├── decorators/
│   │   │   ├── current-user.decorator.ts
│   │   │   └── roles.decorator.ts
│   │   ├── dto/
│   │   │   ├── login.dto.ts
│   │   │   ├── register.dto.ts
│   │   │   └── index.ts
│   │   ├── guards/
│   │   │   ├── jwt-auth.guard.ts
│   │   │   ├── local-auth.guard.ts
│   │   │   └── roles.guard.ts
│   │   ├── strategies/
│   │   │   ├── jwt.strategy.ts
│   │   │   └── local.strategy.ts
│   │   ├── auth.controller.ts
│   │   ├── auth.module.ts
│   │   ├── auth.service.ts
│   │   └── auth.service.spec.ts
│   ├── scores/                    # Scores module
│   │   ├── dto/
│   │   │   ├── create-score.dto.ts
│   │   │   └── index.ts
│   │   ├── guards/
│   │   │   └── score-ownership.guard.ts
│   │   ├── scores.controller.ts
│   │   ├── scores.module.ts
│   │   ├── scores.service.ts
│   │   └── scores.service.spec.ts
│   ├── entities/                  # Database entities
│   │   ├── user.entity.ts
│   │   ├── score.entity.ts
│   │   └── index.ts
│   ├── config/                    # Configuration files
│   │   ├── database.config.ts
│   │   ├── jwt.config.ts
│   │   ├── throttle.config.ts
│   │   └── typeorm.config.ts
│   ├── common/                    # Shared utilities
│   │   ├── interceptors/
│   │   │   └── logging.interceptor.ts
│   │   ├── logger/
│   │   │   └── winston.logger.ts
│   │   └── index.ts
│   ├── app.module.ts              # Root module
│   └── main.ts                    # Application entry point
├── test/                          # E2E tests
├── logs/                          # Application logs
├── .env.development               # Development environment variables
├── .env.staging                   # Staging environment variables
├── .env.production                # Production environment variables
├── .env.example                   # Environment variables template
├── docker-compose.yml             # Docker Compose configuration
├── Dockerfile                     # Docker image configuration
├── init-db.sql                    # Database initialization script
├── package.json                   # Dependencies and scripts
├── package-lock.json              # Dependency lock file
├── tsconfig.json                  # TypeScript configuration
├── nest-cli.json                  # NestJS CLI configuration
├── setup.sh                       # Setup script for Unix/Linux
├── setup.bat                      # Setup script for Windows
├── postman_collection.json        # Postman API collection
├── README.md                      # Main documentation
├── API_EXAMPLES.md                # API usage examples
├── CODE_WALKTHROUGH.md            # Detailed code explanations
├── DEPLOYMENT.md                  # Deployment guide
├── PROJECT_EXPLANATION.md         # Project overview
├── PROJECT_SUMMARY.md             # Quick summary
└── QUICKSTART.md                  # Quick start guide
```

## 🚀 Getting Started

### Prerequisites

- Node.js 18.x or higher
- PostgreSQL 15.x (or use Docker)
- npm or yarn
- Docker & Docker Compose (optional but recommended)

### Installation

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd leaderboard
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Set up environment variables**
   ```bash
   cp .env.example .env.development
   # Edit .env.development with your configuration
   ```

4. **Start with Docker Compose (Recommended)**
   ```bash
   docker-compose up -d
   ```

   Or **Start locally**:
   ```bash
   # Start PostgreSQL separately
   # Then run the application
   npm run start:dev
   ```

5. **Verify installation**
   ```bash
   curl http://localhost:3000/api/leaderboard
   ```

### Quick Start Guide

1. **Register a new user**
   ```bash
   curl -X POST http://localhost:3000/api/auth/register \
     -H "Content-Type: application/json" \
     -d '{
       "username": "player1",
       "password": "password123"
     }'
   ```

2. **Login to get JWT token**
   ```bash
   curl -X POST http://localhost:3000/api/auth/login \
     -H "Content-Type: application/json" \
     -d '{
       "username": "player1",
       "password": "password123"
     }'
   ```

3. **Submit a score** (use token from login)
   ```bash
   curl -X POST http://localhost:3000/api/scores \
     -H "Content-Type: application/json" \
     -H "Authorization: Bearer YOUR_JWT_TOKEN" \
     -d '{
       "score": 1000
     }'
   ```

4. **View leaderboard**
   ```bash
   curl http://localhost:3000/api/leaderboard
   ```

## 📚 API Documentation

### Base URL
```
http://localhost:3000/api
```

### Authentication Endpoints

#### 1. Register User
**POST** `/auth/register`

Creates a new user account.

**Request Body:**
```json
{
  "username": "string",
  "password": "string",
  "role": "user" | "admin" (optional, defaults to "user")
}
```

**Response:**
```json
{
  "message": "User registered successfully",
  "user": {
    "id": "uuid",
    "username": "string",
    "role": "user"
  },
  "accessToken": "jwt-token"
}
```

**Status Codes:**
- `201 Created`: User registered successfully
- `409 Conflict`: Username already exists
- `400 Bad Request`: Invalid input data

---

#### 2. Login
**POST** `/auth/login`

Authenticates a user and returns a JWT token.

**Request Body:**
```json
{
  "username": "string",
  "password": "string"
}
```

**Response:**
```json
{
  "message": "Login successful",
  "user": {
    "id": "uuid",
    "username": "string",
    "role": "user"
  },
  "accessToken": "jwt-token"
}
```

**Status Codes:**
- `200 OK`: Login successful
- `401 Unauthorized`: Invalid credentials

---

### Score Endpoints

#### 3. Submit Score
**POST** `/scores`

Submits a new score for the authenticated user.

**Headers:**
```
Authorization: Bearer <jwt-token>
```

**Request Body:**
```json
{
  "score": number,
  "playerName": "string" (optional, admin only),
  "userId": "uuid" (optional, admin only)
}
```

**Response:**
```json
{
  "message": "Score submitted successfully",
  "score": {
    "id": "uuid",
    "score": number,
    "createdAt": "timestamp"
  }
}
```

**Status Codes:**
- `201 Created`: Score submitted successfully
- `401 Unauthorized`: Missing or invalid JWT token
- `403 Forbidden`: User trying to submit score for another user
- `429 Too Many Requests`: Rate limit exceeded

**Rate Limiting:** 10 requests per 60 seconds

**Authorization Rules:**
- Regular users can only submit scores for themselves
- Admins can submit scores for any user (using `userId` or `playerName`)

---

#### 4. Get Leaderboard
**GET** `/leaderboard`

Retrieves the top scores.

**Query Parameters:**
- `limit` (optional): Number of scores to return (default: 10)

**Response:**
```json
{
  "message": "Leaderboard retrieved successfully",
  "leaderboard": [
    {
      "rank": 1,
      "playerName": "string",
      "score": number,
      "achievedAt": "timestamp"
    }
  ]
}
```

**Status Codes:**
- `200 OK`: Leaderboard retrieved successfully

**Note:** This endpoint is public and doesn't require authentication.

---

#### 5. Get My Scores
**GET** `/scores/me`

Retrieves all scores for the authenticated user.

**Headers:**
```
Authorization: Bearer <jwt-token>
```

**Response:**
```json
{
  "message": "Your scores retrieved successfully",
  "scores": [
    {
      "id": "uuid",
      "score": number,
      "createdAt": "timestamp"
    }
  ]
}
```

**Status Codes:**
- `200 OK`: Scores retrieved successfully
- `401 Unauthorized`: Missing or invalid JWT token

---

#### 6. Get My Best Score
**GET** `/scores/me/best`

Retrieves the highest score for the authenticated user.

**Headers:**
```
Authorization: Bearer <jwt-token>
```

**Response:**
```json
{
  "message": "Your best score retrieved successfully",
  "score": {
    "id": "uuid",
    "score": number,
    "createdAt": "timestamp"
  }
}
```

**Status Codes:**
- `200 OK`: Best score retrieved successfully
- `401 Unauthorized`: Missing or invalid JWT token

---

## 🔄 Flow Diagrams

### Authentication Flow

```
┌────────┐                                      ┌────────────┐
│ Client │                                      │   Server   │
└───┬────┘                                      └─────┬──────┘
    │                                                 │
    │  POST /api/auth/register                       │
    │  { username, password }                        │
    ├────────────────────────────────────────────────►
    │                                                 │
    │                          1. Validate Input     │
    │                          2. Check if user exists│
    │                          3. Hash password      │
    │                          4. Save user to DB    │
    │                          5. Generate JWT token │
    │                                                 │
    │  { user, accessToken }                         │
    ◄────────────────────────────────────────────────┤
    │                                                 │
    │  POST /api/auth/login                          │
    │  { username, password }                        │
    ├────────────────────────────────────────────────►
    │                                                 │
    │                          1. Find user in DB    │
    │                          2. Verify password    │
    │                          3. Generate JWT token │
    │                                                 │
    │  { user, accessToken }                         │
    ◄────────────────────────────────────────────────┤
    │                                                 │
```

### Score Submission Flow (Authenticated User)

```
┌────────┐                                      ┌────────────┐
│ Client │                                      │   Server   │
└───┬────┘                                      └─────┬──────┘
    │                                                 │
    │  POST /api/scores                              │
    │  Authorization: Bearer <token>                 │
    │  { score: 1000 }                               │
    ├────────────────────────────────────────────────►
    │                                                 │
    │                          ┌──────────────────┐  │
    │                          │ Rate Limit Check │  │
    │                          │ (10 req/60s)     │  │
    │                          └────────┬─────────┘  │
    │                                   │             │
    │                          ┌────────▼─────────┐  │
    │                          │ JWT Verification │  │
    │                          │ Extract User     │  │
    │                          └────────┬─────────┘  │
    │                                   │             │
    │                          ┌────────▼─────────┐  │
    │                          │ Ownership Guard  │  │
    │                          │ Check if user=self│  │
    │                          └────────┬─────────┘  │
    │                                   │             │
    │                          ┌────────▼─────────┐  │
    │                          │ Validate Score   │  │
    │                          │ (must be positive)│  │
    │                          └────────┬─────────┘  │
    │                                   │             │
    │                          ┌────────▼─────────┐  │
    │                          │ Save to Database │  │
    │                          └────────┬─────────┘  │
    │                                   │             │
    │                          ┌────────▼─────────┐  │
    │                          │ Log Request      │  │
    │                          └──────────────────┘  │
    │                                                 │
    │  { message, score }                            │
    ◄────────────────────────────────────────────────┤
    │                                                 │
```

### Score Submission Flow (Admin Submitting for Another User)

```
┌────────┐                                      ┌────────────┐
│  Admin │                                      │   Server   │
└───┬────┘                                      └─────┬──────┘
    │                                                 │
    │  POST /api/scores                              │
    │  Authorization: Bearer <admin-token>           │
    │  { score: 1000, playerName: "player1" }       │
    ├────────────────────────────────────────────────►
    │                                                 │
    │                          ┌──────────────────┐  │
    │                          │ Rate Limit Check │  │
    │                          └────────┬─────────┘  │
    │                                   │             │
    │                          ┌────────▼─────────┐  │
    │                          │ JWT Verification │  │
    │                          │ Extract Admin    │  │
    │                          └────────┬─────────┘  │
    │                                   │             │
    │                          ┌────────▼─────────┐  │
    │                          │ Ownership Guard  │  │
    │                          │ Check role=admin │  │
    │                          │ ✓ Allow          │  │
    │                          └────────┬─────────┘  │
    │                                   │             │
    │                          ┌────────▼─────────┐  │
    │                          │ Find Target User │  │
    │                          │ by playerName    │  │
    │                          └────────┬─────────┘  │
    │                                   │             │
    │                          ┌────────▼─────────┐  │
    │                          │ Save Score with  │  │
    │                          │ Target User ID   │  │
    │                          └────────┬─────────┘  │
    │                                   │             │
    │  { message, score }                            │
    ◄────────────────────────────────────────────────┤
    │                                                 │
```

### Leaderboard Retrieval Flow

```
┌────────┐                                      ┌────────────┐
│ Client │                                      │   Server   │
└───┬────┘                                      └─────┬──────┘
    │                                                 │
    │  GET /api/leaderboard?limit=10                 │
    ├────────────────────────────────────────────────►
    │                                                 │
    │                          ┌──────────────────┐  │
    │                          │ No Auth Required │  │
    │                          └────────┬─────────┘  │
    │                                   │             │
    │                          ┌────────▼─────────┐  │
    │                          │ Query Database   │  │
    │                          │ - Group by user  │  │
    │                          │ - Get MAX score  │  │
    │                          │ - Order by score │  │
    │                          │ - Limit results  │  │
    │                          └────────┬─────────┘  │
    │                                   │             │
    │                          ┌────────▼─────────┐  │
    │                          │ Transform Data   │  │
    │                          │ Add rank numbers │  │
    │                          └────────┬─────────┘  │
    │                                   │             │
    │                          ┌────────▼─────────┐  │
    │                          │ Log Request      │  │
    │                          └──────────────────┘  │
    │                                                 │
    │  { message, leaderboard: [...] }               │
    ◄────────────────────────────────────────────────┤
    │                                                 │
```

### Database Schema Diagram

```
┌──────────────────────────────────────┐
│             users                     │
├──────────────────────────────────────┤
│ id                UUID    PK          │
│ username          VARCHAR UNIQUE      │
│ password          VARCHAR             │
│ role              ENUM (user, admin)  │
│ isActive          BOOLEAN             │
│ createdAt         TIMESTAMP           │
│ updatedAt         TIMESTAMP           │
└────────────┬─────────────────────────┘
             │
             │ 1:N
             │
             │
┌────────────▼─────────────────────────┐
│             scores                    │
├──────────────────────────────────────┤
│ id                UUID    PK          │
│ score             INTEGER             │
│ userId            UUID    FK          │
│ createdAt         TIMESTAMP           │
└──────────────────────────────────────┘

Indexes:
- scores.userId (for user score queries)
- scores.score (for leaderboard queries)
```

## ⚙️ Environment Configuration

The application supports multiple environments with different configuration profiles:

### Development Environment
- File: `.env.development`
- Features: Database synchronization enabled, verbose logging, debug mode
- Database: Local PostgreSQL or Docker

### Staging Environment
- File: `.env.staging`
- Features: Production-like settings, moderate logging
- Database: Staging database server

### Production Environment
- File: `.env.production`
- Features: Optimized for performance, minimal logging, security hardened
- Database: Production database server

### Environment Variables

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `NODE_ENV` | Environment (development/staging/production) | development | Yes |
| `PORT` | Application port | 3000 | Yes |
| `DB_HOST` | PostgreSQL host | localhost | Yes |
| `DB_PORT` | PostgreSQL port | 5432 | Yes |
| `DB_USERNAME` | Database username | postgres | Yes |
| `DB_PASSWORD` | Database password | postgres | Yes |
| `DB_NAME` | Database name | leaderboard | Yes |
| `DB_SYNCHRONIZE` | Auto-sync database schema | true (dev only) | Yes |
| `DB_LOGGING` | Enable database query logging | true (dev only) | Yes |
| `JWT_SECRET` | Secret key for JWT signing | - | Yes |
| `JWT_EXPIRATION` | JWT token expiration (seconds) | 3600 | Yes |
| `THROTTLE_TTL` | Rate limit time window (seconds) | 60 | Yes |
| `THROTTLE_LIMIT` | Max requests per time window | 10 | Yes |
| `LOG_LEVEL` | Logging level (debug/info/warn/error) | debug | No |

## 🧪 Testing

### Run Unit Tests
```bash
npm run test
```

### Run Tests with Coverage
```bash
npm run test:cov
```

### Run E2E Tests
```bash
npm run test:e2e
```

### Test Coverage
The project includes comprehensive unit tests for:
- Authentication Service (register, login, validate user)
- Scores Service (create score, get leaderboard, user scores)
- Guards and strategies
- Data validation

## 🔄 CI/CD Pipeline

The project includes a complete GitHub Actions workflow that automates:

### Pipeline Stages

1. **Test**
   - Runs linter
   - Executes unit tests with coverage
   - Uploads coverage reports to Codecov

2. **Build**
   - Installs dependencies
   - Builds the application
   - Stores build artifacts

3. **Docker**
   - Builds Docker image
   - Pushes to GitHub Container Registry
   - Tags with branch name and commit SHA

4. **Security**
   - Runs Trivy vulnerability scanner
   - Uploads results to GitHub Security

5. **Deploy**
   - **Development**: Auto-deploys from `develop` branch
   - **Staging**: Auto-deploys from `staging` branch
   - **Production**: Auto-deploys from `main` branch

### Triggering the Pipeline

```bash
# Push to trigger CI/CD
git push origin develop    # Triggers dev deployment
git push origin staging    # Triggers staging deployment
git push origin main       # Triggers production deployment
```

## 🚀 Deployment

### Docker Deployment (Recommended)

1. **Build and run with Docker Compose**
   ```bash
   docker-compose up -d
   ```

2. **Check logs**
   ```bash
   docker-compose logs -f app
   ```

3. **Stop services**
   ```bash
   docker-compose down
   ```

### Manual Deployment

1. **Build the application**
   ```bash
   npm run build
   ```

2. **Run migrations**
   ```bash
   npm run migration:run
   ```

3. **Start in production mode**
   ```bash
   npm run start:prod
   ```

### Environment-Specific Deployment

```bash
# Development
NODE_ENV=development npm run start:dev

# Staging
NODE_ENV=staging npm run start:prod

# Production
NODE_ENV=production npm run start:prod
```

## 🔒 Security

### Implemented Security Features

1. **Authentication & Authorization**
   - JWT-based authentication
   - Password hashing with bcrypt
   - Role-based access control (RBAC)
   - Ownership validation for score submissions

2. **Rate Limiting**
   - Global rate limiting on all endpoints
   - Stricter limits on score submission (10 req/60s)
   - Prevents API abuse and DDoS attacks

3. **Input Validation**
   - DTO validation using class-validator
   - Whitelist mode (removes unknown properties)
   - Type safety with TypeScript

4. **Security Headers**
   - CORS configuration
   - Request validation

5. **Logging & Monitoring**
   - Comprehensive request/response logging
   - IP address tracking
   - Error logging with stack traces

### Security Best Practices

- ✅ Never commit `.env` files to version control
- ✅ Use strong, unique JWT secrets in production
- ✅ Regularly update dependencies
- ✅ Use HTTPS in production
- ✅ Implement database connection pooling
- ✅ Regular security audits with `npm audit`
- ✅ Monitor logs for suspicious activity

## 📊 Performance Optimization

### Build Once, Modify Little, Scale Forever

This architecture is designed for scalability:

1. **Stateless Application**
   - No session storage on the server
   - JWT tokens for authentication
   - Easy horizontal scaling

2. **Database Optimization**
   - Indexed columns for fast queries
   - Connection pooling
   - Optimized leaderboard queries with aggregation

3. **Caching Strategy** (Future Enhancement)
   - Redis for leaderboard caching
   - Cache invalidation on new scores
   - Reduces database load

4. **Load Balancing** (Future Enhancement)
   - Multiple application instances
   - Nginx or cloud load balancer
   - Health check endpoints

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License.

## 📞 Support

For questions or issues, please open an issue on GitHub.

---

**Built with ❤️ using NestJS, TypeScript, and PostgreSQL**
