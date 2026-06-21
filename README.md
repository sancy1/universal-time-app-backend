# 🌍 Universal Time API Backend

<div align="center">

### Production-Ready Timezone Management & Time Conversion API

A robust REST API for timezone management, time conversion, and current time retrieval built with **Express.js**, **TypeScript**, and **PostgreSQL**.

![TypeScript](https://img.shields.io/badge/TypeScript-5.8-blue)
![Node.js](https://img.shields.io/badge/Node.js-20-green)
![Express](https://img.shields.io/badge/Express-5.1-black)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-Database-blue)
![Docker](https://img.shields.io/badge/Docker-Ready-2496ED)
![License](https://img.shields.io/badge/License-ISC-green)

</div>

---

## 📖 Overview

Universal Time API Backend is a production-ready service for timezone discovery, current time retrieval, timezone conversion, API key management, and rate-limited access. This FULL_PRO edition preserves all original documentation content while enhancing GitHub presentation and readability.

---

## 📑 Table of Contents

- Features
- Prerequisites
- Installation
- Environment Variables
- Database Setup
- Running the Application
- API Documentation
- API Endpoints
- Authentication & Rate Limiting
- Architecture
- Technology Stack
- Design Patterns
- Deployment
- Testing
- Configuration
- Contributing
- License
- Support
- Related Projects
- Performance
- Security
- Monitoring
- Version History


# Universal Time API Backend

A robust REST API for timezone management, time conversion, and current time retrieval built with Express.js, TypeScript, and PostgreSQL.

## 🚀 Features

- **Timezone Management**: Get all timezones, popular timezones, and search timezones
- **Time Conversion**: Convert time between different timezones with accurate DST handling
- **Current Time**: Get current time for any timezone worldwide
- **API Key Authentication**: Secure API with rate limiting (100 req/min for API keys, 10 req/min for public)
- **Swagger Documentation**: Interactive API documentation available in development
- **PostgreSQL Integration**: Efficient database queries with connection pooling
- **Docker Support**: Containerized deployment with multi-stage builds
- **Rate Limiting**: IP-based rate limiting for public endpoints and API key-based limits

## 📋 Prerequisites

- **Node.js**: v20 or higher
- **pnpm**: v10.12.1 or higher (package manager)
- **PostgreSQL**: v12 or higher
- **Docker**: (optional, for containerized deployment)

## 🛠️ Installation

### 1. Clone the Repository

```bash
git clone <repository-url>
cd universal-time-app-backend
```

### 2. Install Dependencies

```bash
pnpm install
```

### 3. Set Up Environment Variables

Create a `.env` file in the root directory:

```env
DATABASE_URL=postgresql://username:password@localhost:5432/universal_time
PORT=3000
NODE_ENV=development
```

**Environment Variables:**

- `DATABASE_URL`: PostgreSQL connection string (required)
- `PORT`: Server port (default: 3000)
- `NODE_ENV`: Environment mode (development/production/test)
- `SSL_CERT`: SSL certificate for production PostgreSQL connections (optional)

### 4. Set Up Database

Create the database and required tables:

```sql
CREATE DATABASE universal_time;

-- Timezones table
CREATE TABLE timezones (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) UNIQUE NOT NULL,
    display_name VARCHAR(200) NOT NULL,
    utc_offset VARCHAR(10) NOT NULL,
    is_dst BOOLEAN DEFAULT false,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Popular timezones table
CREATE TABLE popular_timezones (
    id SERIAL PRIMARY KEY,
    timezone_id INTEGER REFERENCES timezones(id),
    display_order INTEGER DEFAULT 0
);

-- API keys table
CREATE TABLE api_keys (
    id SERIAL PRIMARY KEY,
    key VARCHAR(100) UNIQUE NOT NULL,
    is_active BOOLEAN DEFAULT true,
    request_count INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_used_at TIMESTAMP
);

-- Rate limits table
CREATE TABLE rate_limits (
    id SERIAL PRIMARY KEY,
    api_key_id INTEGER REFERENCES api_keys(id),
    endpoint VARCHAR(100) NOT NULL,
    count INTEGER DEFAULT 0,
    window_start TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(api_key_id, endpoint)
);

-- API request logs table
CREATE TABLE api_request_logs (
    id SERIAL PRIMARY KEY,
    api_key_id INTEGER REFERENCES api_keys(id),
    endpoint VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Populate the timezones table with IANA timezone data (you can use the provided `tables.txt` or import from a timezone database).

## 🏃 Running the Application

### Development Mode

```bash
pnpm dev
```

The server will start on `http://localhost:3000` with hot-reloading enabled.

### Production Mode

```bash
# Build the TypeScript code
pnpm build

# Start the production server
pnpm start
```

### Using Docker

```bash
# Build the Docker image
docker build -t universal-time-backend .

# Run the container
docker run -p 3000:3000 --env-file .env universal-time-backend
```

## 📚 API Documentation

Once the server is running, access the interactive Swagger documentation:

- **Development**: `http://localhost:3000/api-docs`
- **Production**: Disabled by default

### Base URL

- Development: `http://localhost:3000`
- Production: `https://api.universaltime.dev`

### API Endpoints

#### Health Check

```http
GET /health
```

Check server and database health status.

**Response:**
```json
{
  "status": "OK",
  "database": "Connected",
  "environment": "development",
  "timestamp": "2024-03-15T10:30:00.000Z"
}
```

#### Timezone Endpoints

##### Get All Timezones

```http
GET /api/timezones
```

Retrieve all available timezones.

**Query Parameters:**
- `api_key` (optional): API key for higher rate limits

**Response:**
```json
[
  {
    "id": 1,
    "name": "America/New_York",
    "display_name": "New York, USA (EST)",
    "utc_offset": "-05:00",
    "is_dst": true,
    "created_at": "2024-01-01T00:00:00.000Z",
    "updated_at": "2024-01-01T00:00:00.000Z"
  }
]
```

##### Get Popular Timezones

```http
GET /api/timezones/popular
```

Retrieve popular/most-used timezones.

**Response:** Same format as `/api/timezones`

##### Search Timezones

```http
GET /api/timezones/search?query=London
```

Search timezones by name or display name.

**Query Parameters:**
- `query` (required): Search term
- `api_key` (optional): API key for higher rate limits

**Response:** Same format as `/api/timezones`

#### Time Endpoints

##### Get Current Time

```http
GET /api/time?timezone=America/New_York
```

Get current time for a specific timezone.

**Query Parameters:**
- `timezone` (required): IANA timezone identifier (e.g., America/New_York)
- `api_key` (optional): API key for higher rate limits

**Response:**
```json
{
  "time": "2024-03-15T10:30:00.000Z",
  "timezone": "America/New_York",
  "display_name": "New York, USA (EST)",
  "utc_offset": "-05:00",
  "is_dst": true,
  "timestamp": 1710518400
}
```

##### Convert Time

```http
GET /api/convert?from=America/New_York&to=Europe/London&time=2024-03-15T14:30:00
```

Convert time from one timezone to another.

**Query Parameters:**
- `from` (required): Source timezone (e.g., America/New_York)
- `to` (required): Target timezone (e.g., Europe/London)
- `time` (required): Time to convert in ISO format (e.g., 2024-03-15T14:30:00)
- `api_key` (optional): API key for higher rate limits

**Response:**
```json
{
  "original_time": "2024-03-15T14:30:00.000Z",
  "original_timezone": "America/New_York",
  "original_display_name": "New York, USA (EST)",
  "converted_time": "2024-03-15T19:30:00.000Z",
  "converted_timezone": "Europe/London",
  "converted_display_name": "London, UK (GMT)"
}
```

#### API Key Endpoints

##### Generate API Key

```http
GET /api/generate-key
```

Generate a new API key for higher rate limits (100 req/min vs 10 req/min public).

**Response:**
```json
{
  "api_key": "ut_abc123xyz456"
}
```

##### Validate API Key

```http
GET /api/validate-key?key=ut_abc123xyz456
```

Validate an API key and check rate limit status.

**Query Parameters:**
- `key` (required): API key to validate

**Response:**
```json
{
  "valid": true,
  "limitExceeded": false
}
```

##### Get API Key Stats

```http
GET /api/key-stats?key=ut_abc123xyz456
```

Get usage statistics for an API key.

**Query Parameters:**
- `key` (required): API key to get stats for

**Response:**
```json
{
  "total_requests": 150,
  "requests_today": 45,
  "last_used": "2024-03-15T10:30:00.000Z"
}
```

## 🔐 Authentication & Rate Limiting

### Public Access
- **Rate Limit**: 10 requests per minute per IP
- **No API key required**
- **Usage**: Add `?api_key=YOUR_KEY` to any endpoint URL

### API Key Access
- **Rate Limit**: 100 requests per minute per key
- **Generate key**: `GET /api/generate-key`
- **Usage**: Add `?api_key=YOUR_KEY` to any endpoint URL

### Rate Limit Headers
When rate limits are exceeded, the API returns:
```json
{
  "error": "Rate limit exceeded",
  "message": "Get a free API key for higher limits (100 req/min)"
}
```

With HTTP status code `429 Too Many Requests`.

## 🏗️ Architecture

### Project Structure

```
universal-time-app-backend/
├── src/
│   ├── controllers/          # Request handlers
│   │   ├── TimeController.ts
│   │   └── ApiKeyController.ts
│   ├── services/             # Business logic
│   │   ├── TimeService.ts
│   │   ├── ApiKeyService.ts
│   │   └── RateLimiterService.ts
│   ├── repositories/         # Database access layer
│   │   ├── TimezoneRepository.ts
│   │   ├── ApiKeyRepository.ts
│   │   ├── RateLimitRepository.ts
│   │   └── UserPreferencesRepository.ts
│   ├── models/               # TypeScript interfaces
│   │   ├── Timezone.ts
│   │   ├── ApiKey.ts
│   │   ├── RateLimit.ts
│   │   └── UserPreferences.ts
│   ├── db/                   # Database configuration
│   │   └── connection.ts
│   ├── utils/                # Utility functions
│   │   └── swagger.ts
│   ├── app.ts                # Express app configuration
│   ├── config.ts             # Environment configuration
│   └── server.ts             # Server entry point
├── Dockerfile                # Docker configuration
├── render.yaml              # Render deployment config
├── package.json
├── tsconfig.json
└── .env                     # Environment variables
```

### Technology Stack

- **Runtime**: Node.js 20
- **Framework**: Express.js 5.1.0
- **Language**: TypeScript 5.8.3
- **Database**: PostgreSQL with `pg` library
- **Package Manager**: pnpm 10.12.1
- **Documentation**: Swagger/OpenAPI 3.0
- **Security**: Helmet, CORS
- **Logging**: Morgan
- **Validation**: Zod

### Design Patterns

- **Repository Pattern**: Separates database logic from business logic
- **Service Layer**: Encapsulates business rules
- **Controller Pattern**: Handles HTTP requests/responses
- **Dependency Injection**: Services receive dependencies via constructors

## 🚢 Deployment

### Render Deployment

The project includes a `render.yaml` configuration for easy deployment to Render:

1. Push your code to a Git repository
2. Create a new web service on Render
3. Connect your repository
4. Render will automatically use the `render.yaml` configuration
5. Set environment variables in the Render dashboard

**Required Environment Variables on Render:**
- `DATABASE_URL`: PostgreSQL connection string
- `PORT`: 3000
- `NODE_ENV`: production

### Docker Deployment

```bash
# Build the image
docker build -t universal-time-backend .

# Tag for registry
docker tag universal-time-backend your-registry/universal-time-backend

# Push to registry
docker push your-registry/universal-time-backend

# Run on production server
docker run -d \
  --name universal-time-api \
  -p 3000:3000 \
  --env-file .env \
  your-registry/universal-time-backend
```

## 🧪 Testing

```bash
# Run tests
pnpm test
```

## 📝 Scripts

- `pnpm dev` - Start development server with hot-reload
- `pnpm build` - Compile TypeScript to JavaScript
- `pnpm start` - Start production server
- `pnpm lint` - Run ESLint
- `pnpm test` - Run tests

## 🔧 Configuration

### Database Connection

The database connection is configured in `src/db/connection.ts`:

- **Connection Pooling**: Enabled with 10s idle timeout
- **SSL**: Required in production, optional in development
- **Error Handling**: Automatic reconnection on connection errors

### CORS Configuration

CORS is enabled for all origins in development. Restrict origins in production by modifying the CORS middleware in `src/app.ts`.

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the ISC License.

## 🆘 Support

For support, email support@universaltime.dev or open an issue in the repository.

## 🔗 Related Projects

- **Frontend**: [Universal Time Frontend](../universal-time-app/) - Next.js frontend application
- **API Documentation**: Available at `/api-docs` in development mode

## 📊 Performance

- **Response Time**: < 100ms for most endpoints
- **Database Queries**: Optimized with indexes
- **Connection Pooling**: Efficient database connection management
- **Rate Limiting**: Prevents abuse and ensures fair usage

## 🛡️ Security

- **Helmet**: Security headers for Express
- **CORS**: Configurable cross-origin resource sharing
- **Rate Limiting**: Prevents DDoS and abuse
- **API Key Authentication**: Optional authentication for higher limits
- **SQL Injection Prevention**: Parameterized queries
- **Environment Variables**: Sensitive data not hardcoded

## 📈 Monitoring

In production mode, the server logs:
- Request information (via Morgan)
- Database query performance
- Error details
- API usage statistics

## 🔄 Version History

- **1.0.0** - Initial release with timezone management and time conversion


---




# Universal Time

A full-stack timezone management application consisting of a REST API backend and a modern Next.js frontend. Track time across multiple timezones with beautiful analog and digital clock displays, convert time between zones, and access a comprehensive timezone API.

## 🌟 Overview

Universal Time is a complete solution for timezone management with two main components:

1. **Backend API** (`universal-time-app-backend/`): Express.js REST API with PostgreSQL
2. **Frontend App** (`universal-time-app/`): Next.js 15 application with React 19

## 🚀 Features

- **Real-time Clock Display**: Live analog and digital clocks for multiple timezones
- **Timezone Management**: Search, add, and track any timezone from the IANA database
- **Time Conversion**: Convert time between different timezones with accurate DST handling
- **API Access**: Public API with rate limiting and optional API key authentication
- **Persistent Storage**: Selected timezones saved in localStorage
- **Responsive Design**: Mobile-first design that works on all devices
- **Beautiful UI**: Dark purple theme with glass morphism effects

## 📁 Project Structure

```
universal-time/
├── universal-time-app-backend/    # Backend API
│   ├── src/
│   │   ├── controllers/          # Request handlers
│   │   ├── services/             # Business logic
│   │   ├── repositories/         # Database access
│   │   ├── models/               # TypeScript interfaces
│   │   ├── db/                   # Database configuration
│   │   ├── utils/                # Utilities
│   │   ├── app.ts                # Express app
│   │   ├── config.ts             # Configuration
│   │   └── server.ts             # Server entry point
│   ├── Dockerfile                # Docker configuration
│   ├── render.yaml              # Render deployment
│   └── package.json
│
└── universal-time-app/           # Frontend App
    ├── app/                      # Next.js App Router
    │   ├── page.tsx             # Home page
    │   ├── add-timezone/        # Add timezone page
    │   └── convert-time/        # Time conversion page
    ├── components/               # React components
    │   ├── ui/                 # shadcn/ui components
    │   ├── Header.tsx
    │   ├── ClockSection.tsx
    │   ├── AnalogClock.tsx
    │   ├── DigitalClock.tsx
    │   ├── TimezoneSelector.tsx
    │   ├── TimeConverterPanel.tsx
    │   └── ...
    ├── hooks/                   # Custom hooks
    ├── lib/                     # Utilities
    ├── components.json          # shadcn/ui config
    ├── tailwind.config.ts       # TailwindCSS config
    └── package.json
```

## 🛠️ Technology Stack

### Backend
- **Runtime**: Node.js 20
- **Framework**: Express.js 5.1.0
- **Language**: TypeScript 5.8.3
- **Database**: PostgreSQL
- **Package Manager**: pnpm 10.12.1
- **Documentation**: Swagger/OpenAPI 3.0
- **Security**: Helmet, CORS, Rate Limiting

### Frontend
- **Framework**: Next.js 15.2.4 (App Router)
- **UI Library**: React 19
- **Language**: TypeScript 5.x
- **Styling**: TailwindCSS 3.4.17
- **Components**: shadcn/ui (Radix UI)
- **Package Manager**: pnpm

## 🚀 Quick Start

### Prerequisites

- **Node.js**: v20 or higher
- **pnpm**: v10.12.1 or higher
- **PostgreSQL**: v12 or higher

### Installation

1. **Clone the repository**

```bash
git clone <repository-url>
cd universal-time
```

2. **Install backend dependencies**

```bash
cd universal-time-app-backend
pnpm install
```

3. **Set up backend environment**

Create `.env` file in `universal-time-app-backend/`:

```env
DATABASE_URL=postgresql://username:password@localhost:5432/universal_time
PORT=3000
NODE_ENV=development
```

4. **Set up database**

Create the PostgreSQL database and tables (see backend README for detailed SQL scripts).

5. **Start the backend**

```bash
cd universal-time-app-backend
pnpm dev
```

The backend will start on `http://localhost:3000`

6. **Install frontend dependencies**

```bash
cd ../universal-time-app
pnpm install
```

7. **Set up frontend environment**

Create `.env` file in `universal-time-app/`:

```env
NEXT_PUBLIC_API_BASE_URL=http://localhost:3000
```

8. **Start the frontend**

```bash
cd universal-time-app
pnpm dev
```

The frontend will start on `http://localhost:3000` (or `http://localhost:3001` if 3000 is taken)

## 🔗 How Frontend and Backend Connect

### API Communication

The frontend communicates with the backend through REST API calls:

```typescript
// Frontend component
const response = await fetch(
  `${process.env.NEXT_PUBLIC_API_BASE_URL}/api/time?timezone=America/New_York`
);
const data = await response.json();
```

### Data Flow

```
User Action (Frontend)
    ↓
Component Event Handler
    ↓
Custom Hook (useTimeData)
    ↓
API Call (fetch)
    ↓
Backend Controller
    ↓
Backend Service
    ↓
Backend Repository
    ↓
PostgreSQL Database
    ↓
Response (JSON)
    ↓
Frontend State Update
    ↓
UI Re-render
```

### API Endpoints Used by Frontend

1. **Get Current Time**
   - Endpoint: `GET /api/time?timezone={timezone}`
   - Used by: ClockSection, TimezoneClockDisplay
   - Returns: Current time, timezone info, UTC offset

2. **Get All Timezones**
   - Endpoint: `GET /api/timezones`
   - Used by: TimezoneSelectionPanel, TimeConverterPanel
   - Returns: List of all available timezones

3. **Convert Time**
   - Endpoint: `GET /api/convert?from={from}&to={to}&time={time}`
   - Used by: TimeConverterPanel
   - Returns: Converted time with original and target timezone info

### Environment Configuration

The connection between frontend and backend is configured via environment variables:

**Backend** (`universal-time-app-backend/.env`):
```env
DATABASE_URL=postgresql://...
PORT=3000
NODE_ENV=development
```

**Frontend** (`universal-time-app/.env`):
```env
NEXT_PUBLIC_API_BASE_URL=http://localhost:3000
```

The `NEXT_PUBLIC_` prefix makes the variable available in the browser, allowing the frontend to make API calls to the backend.

## 📚 Documentation

- **Backend README**: [universal-time-app-backend/README.md](./universal-time-app-backend/README.md)
  - API endpoints
  - Database schema
  - Deployment instructions
  - Architecture details

- **Frontend README**: [universal-time-app/README.md](./universal-time-app/README.md)
  - Component documentation
  - UI features
  - Development setup
  - Styling guide

## 🚢 Deployment

### Development Deployment

Both applications can run locally:

```bash
# Terminal 1 - Backend
cd universal-time-app-backend
pnpm dev

# Terminal 2 - Frontend
cd universal-time-app
pnpm dev
```

### Production Deployment

#### Backend Deployment (Render)

The backend includes `render.yaml` for easy deployment to Render:

1. Push backend code to Git repository
2. Create new web service on Render
3. Render automatically uses `render.yaml` configuration
4. Set environment variables in Render dashboard

#### Frontend Deployment (Vercel)

1. Push frontend code to Git repository
2. Import project in Vercel
3. Add environment variable: `NEXT_PUBLIC_API_BASE_URL`
4. Deploy

#### Docker Deployment

Both applications include Dockerfiles:

```bash
# Backend
cd universal-time-app-backend
docker build -t universal-time-backend .
docker run -p 3000:3000 --env-file .env universal-time-backend

# Frontend
cd universal-time-app
docker build -t universal-time-frontend .
docker run -p 3001:3000 --env-file .env universal-time-frontend
```

## 🔐 Authentication & Rate Limiting

### Public Access
- **Rate Limit**: 10 requests per minute per IP
- **No authentication required**

### API Key Access
- **Rate Limit**: 100 requests per minute per key
- **Generate key**: `GET /api/generate-key`
- **Usage**: Add `?api_key=YOUR_KEY` to API calls

The frontend can optionally use API keys by including them in fetch calls:

```typescript
const response = await fetch(
  `${process.env.NEXT_PUBLIC_API_BASE_URL}/api/time?timezone=${timezone}&api_key=${apiKey}`
);
```

## 🏗️ Architecture

### Backend Architecture

```
Express App
    ↓
Middleware (CORS, Helmet, Morgan, Rate Limiting)
    ↓
Controllers (TimeController, ApiKeyController)
    ↓
Services (TimeService, ApiKeyService, RateLimiterService)
    ↓
Repositories (TimezoneRepository, ApiKeyRepository, etc.)
    ↓
PostgreSQL Database
```

### Frontend Architecture

```
Next.js App Router
    ↓
Pages (Home, Add Timezone, Convert Time)
    ↓
Components (UI Components, Custom Components)
    ↓
Hooks (useTimeData, use-toast)
    ↓
API Calls (fetch to backend)
    ↓
State Management (React useState, useEffect)
    ↓
UI Updates
```

## 🧪 Testing

### Backend Testing

```bash
cd universal-time-app-backend
pnpm test
```

### Frontend Testing

```bash
cd universal-time-app
pnpm test
```

## 📝 Scripts

### Backend

- `pnpm dev` - Start development server with hot-reload
- `pnpm build` - Compile TypeScript to JavaScript
- `pnpm start` - Start production server
- `pnpm lint` - Run ESLint
- `pnpm test` - Run tests

### Frontend

- `pnpm dev` - Start development server
- `pnpm build` - Build for production
- `pnpm start` - Start production server
- `pnpm lint` - Run ESLint

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## 📄 License

This project is licensed under the ISC License.

## 🆘 Support

For support, email support@universaltime.dev or open an issue in the repository.

## 🔗 Links

- **Backend API Documentation**: Available at `/api-docs` when backend is running in development mode
- **Frontend Application**: `http://localhost:3000` (or configured port)
- **API Base URL**: Configured via `NEXT_PUBLIC_API_BASE_URL`

## 🎯 Use Cases

- **World Clock**: Track time across multiple timezones
- **Time Conversion**: Convert meeting times between timezones
- **API Integration**: Use the API in your own applications
- **Education**: Learn about timezone handling and DST
- **Business**: Coordinate with international teams

## 📊 Performance

- **Backend Response Time**: < 100ms for most endpoints
- **Frontend Load Time**: < 2s initial load
- **Database Queries**: Optimized with indexes
- **API Rate Limiting**: Prevents abuse and ensures fair usage

## 🛡️ Security

- **Backend**: Helmet, CORS, Rate Limiting, SQL Injection Prevention
- **Frontend**: XSS Protection (React), Environment Variables, Input Validation
- **API**: Optional API Key Authentication
- **Database**: Parameterized queries, connection pooling

## 🎨 Design Philosophy

- **User Experience First**: Intuitive interface with clear feedback
- **Performance Optimized**: Fast load times and smooth interactions
- **Accessible**: WCAG compliant with keyboard navigation
- **Responsive**: Works seamlessly on all devices
- **Beautiful**: Modern design with attention to detail

## 🔄 Version History

- **1.0.0** - Initial release with timezone tracking and time conversion
  - Backend API with PostgreSQL
  - Next.js frontend with React 19
  - Real-time clock displays
  - Time conversion functionality
  - API documentation

## 🚧 Roadmap

- [ ] User authentication and saved preferences
- [ ] Mobile app (React Native)
- [ ] Desktop widget
- [ ] Meeting scheduler
- [ ] Sunrise/sunset times
- [ ] Weather integration
- [ ] PWA support
- [ ] Dark/light theme toggle
- [ ] More clock styles
- [ ] API analytics dashboard
