# EasyExpressPro Development Guide

## Overview

This guide covers setting up the development environment for EasyExpressPro, a full-stack express shipping management system.

> **User Guide**: For end-user documentation (how to use the system by role), see [USER_GUIDE.md](./USER_GUIDE.md).

## Prerequisites

- Node.js >= 18.0.0
- pnpm >= 8.0.0
- MongoDB (local or MongoDB Atlas)
- Git

## Project Structure

```
EasyExpressPro/
├── apps/
│   ├── backend/          # NestJS API server
│   │   ├── src/
│   │   │   ├── auth/     # Authentication & Clerk integration
│   │   │   ├── users/    # User management
│   │   │   ├── branches/ # Branch management
│   │   │   ├── shipments/ # Core business logic
│   │   │   ├── payments/ # Payment processing
│   │   │   ├── notifications/ # Email & push notifications
│   │   │   ├── webhooks/ # Webhook handlers
│   │   │   ├── storage/  # File storage
│   │   │   ├── websocket/ # Real-time updates
│   │   │   ├── cache/    # Caching layer
│   │   │   ├── monitoring/ # Error tracking & logging
│   │   │   └── security/ # Rate limiting & security
│   │   └── test/         # Tests
│   └── frontend/         # Next.js application
│       ├── src/
│       │   ├── app/      # App Router pages
│       │   ├── components/ # React components
│       │   ├── services/ # API services
│       │   └── utils/   # Utility functions
│       └── public/       # Static assets
├── shared/
│   └── types/           # Shared TypeScript types
├── docs/                 # Documentation
└── package.json          # Root workspace configuration
```

## Setup Instructions

### 1. Clone Repository

```bash
git clone https://github.com/your-username/easy-express-pro.git
cd easy-express-pro
```

### 2. Install Dependencies

```bash
pnpm install
```

### 3. Environment Setup

#### Backend Environment

Create `apps/backend/.env`:

```bash
# Application
NODE_ENV=development
PORT=3001
FRONTEND_URL=http://localhost:3000

# Database
MONGODB_URI=mongodb://localhost:27017/easy-express-pro

# Clerk Authentication (get from clerk.com)
CLERK_SECRET_KEY=sk_test_your_secret_key
CLERK_PUBLISHABLE_KEY=pk_test_your_publishable_key
CLERK_WEBHOOK_SECRET=whsec_your_webhook_secret

# Stripe Payments (get from stripe.com)
STRIPE_SECRET_KEY=sk_test_your_stripe_secret_key
STRIPE_WEBHOOK_SECRET=whsec_your_stripe_webhook_secret

# AWS S3 (optional for development)
AWS_ACCESS_KEY_ID=your_aws_access_key
AWS_SECRET_ACCESS_KEY=your_aws_secret_key
AWS_REGION=us-east-1
AWS_S3_BUCKET_NAME=your-bucket-name

# SendGrid (optional for development)
SENDGRID_API_KEY=your_sendgrid_api_key

# Sentry (optional for development)
SENTRY_DSN=https://your-sentry-dsn@sentry.io/project-id

# Redis (optional for development)
REDIS_URL=redis://localhost:6379
```

#### Frontend Environment

Create `apps/frontend/.env.local`:

```bash
# Clerk Authentication
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_your_publishable_key
CLERK_SECRET_KEY=sk_test_your_secret_key

# Backend URL
NEXT_PUBLIC_BACKEND_URL=http://localhost:3001

# Site URL
NEXT_PUBLIC_SITE_URL=http://localhost:3000
```

### 4. Database Setup

#### Local MongoDB

```bash
# Install MongoDB (macOS)
brew install mongodb-community

# Start MongoDB
brew services start mongodb-community

# Or using Docker
docker run -d -p 27017:27017 --name mongodb mongo:7
```

#### MongoDB Atlas (Cloud)

1. Create account at [MongoDB Atlas](https://www.mongodb.com/cloud/atlas)
2. Create a new cluster
3. Create a database user
4. Whitelist your IP address
5. Get the connection string

### 5. Start Development Servers

```bash
# Start both frontend and backend
pnpm dev

# Or start individually
pnpm --filter @easy-express-pro/backend dev
pnpm --filter @easy-express-pro/frontend dev
```

The application will be available at:
- Frontend: http://localhost:3000
- Backend: http://localhost:3001

## Development Workflow

### Code Organization

#### Backend Structure

- **Modules**: Each feature has its own module (users, shipments, etc.)
- **Services**: Business logic implementation
- **Controllers**: HTTP request/response handling
- **Schemas**: Database models (Mongoose)
- **DTOs**: Data transfer objects for validation
- **Guards**: Authentication and authorization
- **Interceptors**: Cross-cutting concerns (logging, caching)
- **Filters**: Error handling

#### Frontend Structure

- **Pages**: Next.js App Router pages
- **Components**: Reusable UI components (atomic design)
- **Services**: API communication
- **Hooks**: Custom React hooks
- **Utils**: Helper functions
- **Types**: TypeScript type definitions

### Adding New Features

#### Backend Feature

1. Create module directory: `apps/backend/src/feature-name/`
2. Create service: `feature-name.service.ts`
3. Create controller: `feature-name.controller.ts`
4. Create module: `feature-name.module.ts`
5. Create schema (if needed): `schemas/feature-name.schema.ts`
6. Add to app module: `app.module.ts`
7. Write tests: `feature-name.service.spec.ts`

#### Frontend Feature

1. Create page: `apps/frontend/src/app/feature-name/page.tsx`
2. Create components: `apps/frontend/src/components/feature-name/`
3. Create API route: `apps/frontend/src/app/api/feature-name/route.ts`
4. Add types: `shared/types/src/index.ts`
5. Write tests: `feature-name.test.tsx`

### Testing

#### Backend Tests

```bash
# Run all tests
pnpm test

# Run unit tests
pnpm test:unit

# Run integration tests
pnpm test:integration

# Run tests with coverage
pnpm test:cov

# Run tests in watch mode
pnpm test:watch
```

#### Frontend Tests

```bash
# Run all tests
pnpm test

# Run tests with coverage
pnpm test:cov

# Run tests in watch mode
pnpm test:watch
```

### Code Quality

#### Linting

```bash
# Lint all code
pnpm lint

# Fix linting issues
pnpm lint:fix
```

#### Type Checking

```bash
# Check types
pnpm type-check
```

#### Formatting

```bash
# Format code
pnpm format

# Check formatting
pnpm format:check
```

## Key Features Implementation

### Authentication (Clerk)

The application uses Clerk for authentication:

```typescript
// Backend: JWT verification
@Injectable()
export class ClerkStrategy extends PassportStrategy(Strategy, 'clerk') {
  async validate(payload: any): Promise<ClerkJWT> {
    const verifiedPayload = await verifyToken(payload, {
      secretKey: this.configService.get<string>('CLERK_SECRET_KEY'),
    });
    return verifiedPayload;
  }
}

// Frontend: User context
const { user, isLoaded } = useUser();
const { getToken } = useAuth();
```

### Real-time Updates (WebSocket)

WebSocket integration for real-time notifications:

```typescript
// Backend: WebSocket gateway
@WebSocketGateway({ namespace: '/shipments' })
export class WebsocketGateway {
  @SubscribeMessage('join_shipment')
  async handleJoinShipment(data: { shipmentId: string }) {
    // Join shipment room
  }
}

// Frontend: WebSocket service
const { connect, on, off } = useWebSocket();
```

### File Storage

Multi-environment file storage:

```typescript
// Backend: Storage service
@Injectable()
export class StorageService {
  async uploadFile(file: Express.Multer.File, folder: string) {
    if (this.isProduction) {
      return this.uploadToS3(file, key);
    } else {
      return this.uploadLocally(file, key);
    }
  }
}
```

### Caching

Redis-based caching with fallback:

```typescript
// Backend: Cache service
@Injectable()
export class CacheService {
  async getOrSet<T>(key: string, factory: () => Promise<T>) {
    const cached = await this.get<T>(key);
    if (cached !== undefined) return cached;
    
    const value = await factory();
    await this.set(key, value);
    return value;
  }
}
```

### Error Monitoring

Comprehensive error tracking:

```typescript
// Backend: Error reporting
@Injectable()
export class MonitoringService {
  reportError(report: ErrorReport) {
    this.logger.error(report.error.message, report.error.stack);
    if (this.sentryEnabled) {
      Sentry.captureException(report.error);
    }
  }
}
```

## API Development

### Creating Endpoints

```typescript
// Controller
@Controller('shipments')
export class ShipmentsController {
  @Post()
  @UseGuards(JwtAuthGuard)
  async create(@Body() createShipmentDto: CreateShipmentDto) {
    return this.shipmentsService.create(createShipmentDto);
  }
}

// Service
@Injectable()
export class ShipmentsService {
  async create(createShipmentDto: CreateShipmentDto) {
    // Business logic
  }
}
```

### Validation

```typescript
// DTO with validation
export class CreateShipmentDto {
  @IsString()
  @IsNotEmpty()
  branchId: string;

  @IsString()
  @IsNotEmpty()
  pickupAddress: string;

  @IsNumber()
  @Min(0)
  price: number;
}
```

### Error Handling

```typescript
// Custom exceptions
export class ShipmentNotFoundException extends NotFoundException {
  constructor(shipmentId: string) {
    super(`Shipment with ID ${shipmentId} not found`);
  }
}
```

## Frontend Development

### Component Development

```typescript
// Atomic design pattern
interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'outline' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
  loading?: boolean;
  disabled?: boolean;
  children: React.ReactNode;
}

export function Button({ variant = 'primary', size = 'md', ...props }: ButtonProps) {
  return (
    <button className={cn(buttonVariants({ variant, size }))} {...props}>
      {props.loading ? 'Loading...' : props.children}
    </button>
  );
}
```

### API Integration

```typescript
// API service
export class ApiService {
  async createShipment(data: CreateShipmentDto) {
    const token = await getToken();
    const response = await fetch('/api/shipments', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(data),
    });
    return response.json();
  }
}
```

### State Management

```typescript
// Custom hooks
export function useShipments() {
  const [shipments, setShipments] = useState<Shipment[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchShipments().then(setShipments).finally(() => setLoading(false));
  }, []);

  return { shipments, loading, refetch: () => fetchShipments() };
}
```

## Database Development

### Schema Design

```typescript
// Mongoose schema
@Schema({ timestamps: true })
export class Shipment {
  @Prop({ type: String, ref: 'User' })
  customerId?: string;

  @Prop({ type: String, ref: 'Branch', required: true })
  branchId: string;

  @Prop({ required: true })
  pickupAddress: string;

  @Prop({ required: true })
  deliveryAddress: string;

  @Prop({ required: true })
  price: number;

  @Prop({ 
    required: true, 
    enum: ['pending', 'confirmed', 'picked_up', 'in_transit', 'delivered', 'cancelled', 'failed'],
    default: 'pending'
  })
  status: ShipmentStatus;
}
```

### Indexing

```typescript
// Add indexes for performance
ShipmentSchema.index({ customerId: 1 });
ShipmentSchema.index({ branchId: 1 });
ShipmentSchema.index({ status: 1 });
ShipmentSchema.index({ branchId: 1, status: 1 });
```

## Debugging

### Backend Debugging

```bash
# Debug mode
pnpm start:debug

# View logs
tail -f logs/combined.log

# Database queries
mongosh easy-express-pro
```

### Frontend Debugging

```bash
# Development mode with debugging
pnpm dev

# Browser DevTools
# - Network tab for API calls
# - Console for errors
# - React DevTools for component state
```

### Common Issues

1. **Port conflicts**: Change ports in environment variables
2. **Database connection**: Check MongoDB is running
3. **Authentication**: Verify Clerk keys are correct
4. **CORS issues**: Check frontend URL in backend config
5. **Type errors**: Run `pnpm type-check`

## Performance Optimization

### Backend Optimization

- Use database indexes
- Implement caching
- Optimize queries
- Use connection pooling
- Monitor performance metrics

### Frontend Optimization

- Code splitting
- Image optimization
- Bundle analysis
- Lazy loading
- Service worker caching

## Security Considerations

- Input validation
- SQL injection prevention
- XSS protection
- CSRF protection
- Rate limiting
- Authentication checks
- Authorization rules
- Secure headers

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Write tests
5. Run linting and tests
6. Submit a pull request

## Resources

- [NestJS Documentation](https://docs.nestjs.com/)
- [Next.js Documentation](https://nextjs.org/docs)
- [Clerk Documentation](https://clerk.com/docs)
- [MongoDB Documentation](https://docs.mongodb.com/)
- [TypeScript Documentation](https://www.typescriptlang.org/docs/)