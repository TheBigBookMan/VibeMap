# VibeMap

## Project Overview
VibeMap is an application which brings people together by allowing users to create plans and other users can join that plan. Meeting up in real life based on messaging through the app for a specific activity.

## Tech Stack
### Product
| Technology | Purpose |
|------------|---------|
| Linear     | Product management |

### Frontend

| Technology | Purpose |
|------------|---------|
| Vite | Build tool |
| React 18 | UI framework |
| TypeScript | Type safety |
| Tailwind CSS | Styling |
| TanStack Query | Server state management |
| Zustand | client state management |
| TanStack Router | Routing |
| React Hook Form | Form handling |
| Zod | Validation (shared with backend) |
| Socket.IO Client | Real-time communication |
| Leaflet / Mapbox | Map display |
| shadcn/ui | Component library |
| axios | HTTP request wrapper |
| react-hot-toast | Toasts |
| Framer Motion | animation |
| React Icons | icons |
| date-fns | Date manipulation |

### Backend (Node.js)

| Technology | Purpose |
|------------|---------|
| Node.js 20 LTS | Runtime |
| TypeScript | Type safety |
| Express.js | HTTP framework |
| Helmet | Express Header Security |
| Socket.IO | WebSocket server |
| Prisma | ORM |
| Zod | Validation |
| express-async-errors | Express async errors |
| dotenv | environment variable handler |
| pino | Logging |
| rate-limiter-flexible | Rate limiting |
| jsonwebtoken | JWT auth |
| argon2 | Password hashing |
| @aws-sdk/* | AWS services |
| stripe | Payments |

### Backend (Python Worker)

| Technology | Purpose |
|------------|---------|
| Python 3.12 | Runtime |
| Flask | HTTP framework (for callbacks) |
| boto3 | AWS SQS |
| scikit-learn / transformers | ML models |
| pydantic | Data validation |
| pytest | Testing |
| requests | HTTP client |
| redis-py | redis client |

### Backend (Go Notification Dispatcher)

| Technology | Purpose |
|------------|---------|
| Go 1.22+ | Runtime |
| Gin | HTTP framework |
| Goroutines + Channels | Concurrency |
| Context | Cancellation/timeouts |
| Worker Pools | Bounded concurrency |
| go-redis/redis | Redis client |
| jackc/pgx | PostgreSQL driver |
| aws-sdk-go-v2 | AWS SQS consumer |
| zerolog | Structured logging |
| validator/v10 | Validation |
| resty | HTTP client |
| testify | Testing assertions |
| gomock | Mocking |
| golang.org/x/time/rate | Rate limiting |
| sony/gobreaker | Circuit breaker |
| cenkalti/backoff | Retry logic |
| prometheus/client_golang | Metrics |
| viper | Configuration |

### Data Layer

| Technology | Purpose |
|------------|---------|
| PostgreSQL 16 | Primary database |
| PostGIS | Geospatial queries |
| Redis 7 | Caching, rate limiting, Socket.IO adapter |
| ioredis | Better redis client |

### Infrastructure

| Technology | Purpose |
|------------|---------|
| Docker | Containerization |
| Docker Compose | Local development |
| AWS EC2 | Compute |
| AWS RDS | Managed PostgreSQL |
| AWS ElastiCache | Managed Redis |
| AWS S3 | File storage |
| AWS SQS | Message queue |
| AWS CloudWatch | Logging & monitoring |
| GitHub Actions | CI/CD |
| Stripe | Payments |

### Testing

| Technology | Purpose |
|------------|---------|
| Vitest | Frontend unit & integration tests |
| @testing-library/react | React component testing |
| @testing-library/user-event | User interaction simulation |
| Jest | Backend unit & integration tests |
| supertest | HTTP endpoint testing |
| Playwright | E2E browser testing |
| Storybook | testing frontend |
| pytest | Python worker unit tests |
| msw | API mocking frontend tests for integration tests |
| faker | test data generation |
| testcontainers | spin up real postgres/redis for integration |

## Setup Project