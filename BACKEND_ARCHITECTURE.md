# KoloShop - Backend Architecture & Implementation

## Table des matières

1. [Structure Complète du Projet Backend](#structure-complète)
2. [Modules Détaillés](#modules-détaillés)
3. [Services Métier](#services-métier)
4. [Validation & Schemas](#validation--schemas)
5. [Error Handling](#error-handling)
6. [Middleware Stack](#middleware-stack)
7. [Database Repositories](#database-repositories)
8. [Queue System](#queue-system)
9. [Configuration](#configuration)
10. [Development Workflow](#development-workflow)

---

## Structure Complète

```
backend/
│
├── src/
│   ├── main.ts                      # App entry point
│   │
│   ├── config/
│   │   ├── env.ts                   # Environment validation + loading
│   │   ├── database.ts              # Database configuration
│   │   ├── redis.ts                 # Redis client config
│   │   ├── auth.ts                  # JWT + auth config
│   │   ├── payment-providers.ts     # Payment API keys
│   │   ├── aws.ts                   # S3 configuration
│   │   ├── email.ts                 # SendGrid config
│   │   └── constants.ts             # Application constants
│   │
│   ├── infrastructure/
│   │   ├── database.ts              # Prisma + connection pooling
│   │   ├── redis.ts                 # Redis client + utilities
│   │   ├── http-client.ts           # Axios instance with interceptors
│   │   ├── storage.ts               # S3 client for file uploads
│   │   ├── email.ts                 # SendGrid integration
│   │   ├── sms.ts                   # Twilio SMS client
│   │   └── vault.ts                 # HashiCorp Vault integration (prod)
│   │
│   ├── plugins/
│   │   ├── database.plugin.ts       # Prisma + pgBouncer
│   │   ├── redis.plugin.ts          # Redis connection pool
│   │   ├── request-id.ts            # Correlation IDs
│   │   ├── helmet.ts                # Security headers
│   │   ├── rate-limit.ts            # Rate limiting
│   │   ├── cors.ts                  # CORS configuration
│   │   ├── swagger.ts               # OpenAPI documentation
│   │   └── health-check.ts          # Liveness + readiness probes
│   │
│   ├── middleware/
│   │   ├── error-handler.ts         # Global error handling
│   │   ├── request-logger.ts        # Structured logging
│   │   ├── metrics.ts               # Prometheus metrics
│   │   ├── authentication.ts        # JWT validation
│   │   ├── authorization.ts         # RBAC + permissions
│   │   ├── validation.ts            # Request body validation
│   │   ├── request-context.ts       # Store userId, etc.
│   │   ├── rate-limiter.ts          # Per-endpoint rate limits
│   │   ├── input-sanitizer.ts       # XSS prevention
│   │   └── activity-logger.ts       # Audit trail
│   │
│   ├── domain/                      # Business logic layer (DDD)
│   │   ├── user/
│   │   │   ├── user.entity.ts
│   │   │   ├── user.value-object.ts
│   │   │   ├── user.service.ts      # Business logic
│   │   │   ├── user.repository.ts   # Data access
│   │   │   ├── user.errors.ts
│   │   │   └── user.types.ts
│   │   │
│   │   ├── seller/
│   │   │   ├── seller.entity.ts
│   │   │   ├── seller.service.ts
│   │   │   ├── seller.repository.ts
│   │   │   ├── seller.types.ts
│   │   │   └── seller.errors.ts
│   │   │
│   │   ├── product/
│   │   │   ├── product.entity.ts
│   │   │   ├── product.service.ts
│   │   │   ├── product.repository.ts
│   │   │   ├── search.service.ts
│   │   │   ├── product.types.ts
│   │   │   └── product.errors.ts
│   │   │
│   │   ├── order/
│   │   │   ├── order.entity.ts
│   │   │   ├── order.state-machine.ts    # Order status flow
│   │   │   ├── order.service.ts
│   │   │   ├── order.repository.ts
│   │   │   ├── order.types.ts
│   │   │   ├── order.errors.ts
│   │   │   └── order.events.ts          # Domain events
│   │   │
│   │   ├── payment/
│   │   │   ├── payment.entity.ts
│   │   │   ├── payment.state-machine.ts
│   │   │   ├── payment.service.ts
│   │   │   ├── payment.repository.ts
│   │   │   ├── payment.types.ts
│   │   │   ├── payment.errors.ts
│   │   │   └── payment.events.ts
│   │   │
│   │   ├── delivery/
│   │   │   ├── delivery.entity.ts
│   │   │   ├── delivery.service.ts
│   │   │   ├── delivery.repository.ts
│   │   │   ├── delivery.types.ts
│   │   │   └── delivery.errors.ts
│   │   │
│   │   └── wallet/
│   │       ├── wallet.entity.ts
│   │       ├── wallet.service.ts
│   │       ├── wallet.repository.ts
│   │       └── wallet.types.ts
│   │
│   ├── api/                         # HTTP Controllers & Routes
│   │   ├── auth/
│   │   │   ├── auth.controller.ts
│   │   │   ├── auth.routes.ts
│   │   │   ├── auth.schemas.ts
│   │   │   └── auth.tests.ts
│   │   │
│   │   ├── users/
│   │   │   ├── users.controller.ts
│   │   │   ├── users.routes.ts
│   │   │   ├── users.schemas.ts
│   │   │   └── users.tests.ts
│   │   │
│   │   ├── products/
│   │   │   ├── products.controller.ts
│   │   │   ├── products.routes.ts
│   │   │   ├── products.schemas.ts
│   │   │   ├── search.routes.ts
│   │   │   └── products.tests.ts
│   │   │
│   │   ├── orders/
│   │   │   ├── orders.controller.ts
│   │   │   ├── orders.routes.ts
│   │   │   ├── orders.schemas.ts
│   │   │   └── orders.tests.ts
│   │   │
│   │   ├── payments/
│   │   │   ├── payments.controller.ts
│   │   │   ├── payments.routes.ts
│   │   │   ├── payments.schemas.ts
│   │   │   └── payments.tests.ts
│   │   │
│   │   ├── delivery/
│   │   │   ├── delivery.controller.ts
│   │   │   ├── delivery.routes.ts
│   │   │   ├── delivery.schemas.ts
│   │   │   └── delivery.tests.ts
│   │   │
│   │   ├── sellers/
│   │   │   ├── sellers.controller.ts
│   │   │   ├── sellers.routes.ts
│   │   │   ├── sellers.schemas.ts
│   │   │   └── sellers.tests.ts
│   │   │
│   │   ├── admin/
│   │   │   ├── admin.controller.ts
│   │   │   ├── admin.routes.ts
│   │   │   ├── admin.schemas.ts
│   │   │   └── users-list.routes.ts
│   │   │
│   │   └── health.routes.ts         # /health, /ready endpoints
│   │
│   ├── integration/
│   │   ├── payment-providers/
│   │   │   ├── payment-provider.interface.ts
│   │   │   ├── moov.provider.ts
│   │   │   ├── orange-money.provider.ts
│   │   │   ├── stripe.provider.ts
│   │   │   └── paystack.provider.ts
│   │   │
│   │   ├── webhooks/
│   │   │   ├── webhook.handler.ts
│   │   │   ├── moov-webhook.ts
│   │   │   ├── orange-webhook.ts
│   │   │   ├── stripe-webhook.ts
│   │   │   └── webhook.types.ts
│   │   │
│   │   ├── external-services/
│   │   │   ├── sms.service.ts       # Twilio
│   │   │   ├── email.service.ts     # SendGrid
│   │   │   ├── storage.service.ts   # S3
│   │   │   └── maps.service.ts      # Google Maps / OpenStreetMap
│   │   │
│   │   └── third-party/
│   │       ├── kyc.ts               # KYC provider integration
│   │       ├── fraud-detection.ts   # Fraud ML service
│   │       └── analytics.ts         # Mixpanel / Amplitude
│   │
│   ├── services/
│   │   ├── cache.service.ts         # Cache wrapper (Redis)
│   │   ├── encryption.service.ts    # Data encryption (NaCl)
│   │   ├── hash.service.ts          # Argon2 hashing
│   │   ├── token.service.ts         # JWT generation
│   │   ├── audit.service.ts         # Audit logging
│   │   ├── notification.service.ts  # Push / SMS / Email
│   │   ├── storage.service.ts       # File uploads
│   │   ├── metrics.service.ts       # Prometheus metrics
│   │   ├── logger.service.ts        # Structured logging
│   │   └── geo.service.ts           # Geolocation / distance calc
│   │
│   ├── workers/
│   │   ├── worker.ts                # Bull queue setup
│   │   ├── queues/
│   │   │   ├── email.queue.ts
│   │   │   ├── notification.queue.ts
│   │   │   ├── payment-reconciliation.queue.ts
│   │   │   ├── delivery-sync.queue.ts
│   │   │   ├── image-processing.queue.ts
│   │   │   ├── seller-payout.queue.ts
│   │   │   └── report-generation.queue.ts
│   │   │
│   │   ├── jobs/
│   │   │   ├── send-email.job.ts
│   │   │   ├── send-push.job.ts
│   │   │   ├── reconcile-payments.job.ts
│   │   │   ├── sync-delivery-status.job.ts
│   │   │   ├── process-image.job.ts
│   │   │   ├── generate-seller-report.job.ts
│   │   │   ├── process-payout.job.ts
│   │   │   └── notify-merchants.job.ts
│   │   │
│   │   └── processors/
│   │       ├── email.processor.ts
│   │       ├── notification.processor.ts
│   │       ├── payment.processor.ts
│   │       └── delivery.processor.ts
│   │
│   ├── scheduled-tasks/
│   │   ├── scheduler.ts             # node-cron setup
│   │   ├── payment-reconciliation.cron.ts  # Daily reconciliation
│   │   ├── seller-payout.cron.ts     # Weekly payout processing
│   │   ├── order-timeout.cron.ts     # Cancel old pending orders
│   │   ├── seller-ranking.cron.ts    # Update seller ratings
│   │   ├── report-generation.cron.ts # Daily/weekly reports
│   │   └── cleanup.cron.ts           # Archive old logs
│   │
│   ├── utils/
│   │   ├── errors.ts                # Custom error classes
│   │   ├── validators.ts            # Utility validators
│   │   ├── formatters.ts            # Number, date, currency formatting
│   │   ├── crypto.ts                # Encryption/decryption utils
│   │   ├── pagination.ts            # Pagination helper
│   │   ├── cache-keys.ts            # Redis key generation
│   │   ├── constants.ts             # App constants
│   │   ├── date-utils.ts            # Date calculations
│   │   └── geo-utils.ts             # Distance, coordinates
│   │
│   ├── types/
│   │   ├── common.ts                # Common types and interfaces
│   │   ├── auth.ts                  # Auth types
│   │   ├── api-response.ts          # API response shape
│   │   ├── database.ts              # DB-related types
│   │   ├── error.ts                 # Error types
│   │   └── express.d.ts             # Fastify request augmentation
│   │
│   ├── fixtures/                    # Test data
│   │   ├── users.fixture.ts
│   │   ├── products.fixture.ts
│   │   ├── orders.fixture.ts
│   │   └── factories/
│   │       ├── user.factory.ts
│   │       ├── product.factory.ts
│   │       └── order.factory.ts
│   │
│   └── app.ts                       # Fastify app factory
│
├── tests/
│   ├── unit/
│   │   ├── services/
│   │   │   ├── user.service.test.ts
│   │   │   ├── order.service.test.ts
│   │   │   ├── payment.service.test.ts
│   │   │   └── wallet.service.test.ts
│   │   │
│   │   ├── utils/
│   │   │   ├── validators.test.ts
│   │   │   └── formatters.test.ts
│   │   │
│   │   └── domain/
│   │       └── payment.state-machine.test.ts
│   │
│   ├── integration/
│   │   ├── auth.test.ts
│   │   ├── products.test.ts
│   │   ├── orders.test.ts
│   │   ├── payments.test.ts
│   │   └── delivery.test.ts
│   │
│   ├── e2e/
│   │   ├── user-journey.test.ts     # Complete user flow
│   │   ├── order-flow.test.ts       # Order to delivery
│   │   └── payment-flow.test.ts     # Payment processing
│   │
│   └── setup.ts                     # Test environment setup
│
├── migrations/
│   ├── 001_init_schema.sql
│   ├── 002_add_payment_indexes.sql
│   ├── 003_create_partitions.sql
│   └── README.md              # Migration guide
│
├── prisma/
│   └── schema.prisma          # Prisma schema definition
│
├── docker/
│   ├── Dockerfile             # Production Docker image
│   ├── Dockerfile.worker      # Background worker image
│   ├── Dockerfile.webhook     # Webhook handler image
│   └── .dockerignore
│
├── k8s/                       # Kubernetes manifests (future)
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   └── ingress.yaml
│
├── .env.example               # Environment template
├── .env.local                 # Local development (git-ignored)
├── .env.test                  # Test environment (git-ignored)
├── .gitignore
├── package.json
├── pnpm-lock.yaml
├── tsconfig.json
├── tsconfig.build.json
├── vitest.config.ts
├── eslint.config.mjs
├── prettier.config.js
├── Dockerfile
├── docker-compose.yml
└── README.md
```

---

## Modules Détaillés

### 1. Module Authentification (Auth)

```typescript
// src/api/auth/auth.schemas.ts
import { z } from 'zod';

export const LoginSchema = z.object({
  email: z.string().email("Invalid email"),
  password: z.string().min(8, "Password must be at least 8 characters"),
  deviceFingerprint: z.string().optional()
});

export const RegisterSchema = z.object({
  email: z.string().email(),
  phone: z.string().regex(/^\+?[0-9]{10,14}$/),
  password: z.string().min(8),
  firstName: z.string().min(2),
  lastName: z.string().min(2),
  userType: z.enum(["customer", "seller", "rider"]),
  acceptTerms: z.boolean().refine(val => val === true, {
    message: "Must accept terms"
  })
});

export const RefreshTokenSchema = z.object({
  refreshToken: z.string().min(10)
});

export const VerifyOTPSchema = z.object({
  email: z.string().email(),
  otp: z.string().length(6),
  type: z.enum(["email_verification", "password_reset"])
});

// src/api/auth/auth.controller.ts
export class AuthController {
  constructor(
    private authService: AuthService,
    private userService: UserService,
    private tokenService: TokenService
  ) {}

  async login(req: FastifyRequest, reply: FastifyReply) {
    const { email, password, deviceFingerprint } = req.body;

    try {
      const user = await this.authService.validateCredentials(email, password);
      
      // Check fraud
      await this.authService.checkFraud(user.id, req.ip, deviceFingerprint);

      // Generate tokens
      const { accessToken, refreshToken } = await this.tokenService.generateTokenPair(
        user.id,
        user.userType
      );

      // Store refresh token in DB with family tracking
      await this.authService.storeRefreshToken({
        userId: user.id,
        refreshToken,
        ipAddress: req.ip,
        userAgent: req.headers["user-agent"],
        deviceFingerprint
      });

      // Set HTTP-only cookie
      reply.setCookie("refreshToken", refreshToken, {
        httpOnly: true,
        secure: process.env.NODE_ENV === "production",
        sameSite: "strict",
        maxAge: 30 * 24 * 60 * 60 * 1000  // 30 days
      });

      // Log activity
      await this.authService.logActivity({
        userId: user.id,
        action: "user.login",
        ipAddress: req.ip
      });

      reply.send({
        accessToken,
        user: user.toDTO()
      });
    } catch (error) {
      if (error instanceof InvalidCredentialsError) {
        // Rate limit brute force
        await this.authService.recordFailedLogin(email, req.ip);
        return reply.code(401).send({ error: "Invalid credentials" });
      }
      throw error;
    }
  }

  async register(req: FastifyRequest, reply: FastifyReply) {
    const data = req.body;

    // Check if user exists
    const existing = await this.userService.findByEmail(data.email);
    if (existing) {
      return reply.code(409).send({ error: "User already exists" });
    }

    // Create user
    const user = await this.userService.create({
      email: data.email,
      phone: data.phone,
      firstName: data.firstName,
      lastName: data.lastName,
      userType: data.userType,
      password: data.password  // Will be hashed by service
    });

    // Send verification email
    await this.authService.sendVerificationEmail(user.id, user.email);

    reply.code(201).send({
      message: "Registration successful. Check email for verification link.",
      userId: user.id
    });
  }

  async refreshToken(req: FastifyRequest, reply: FastifyReply) {
    const { refreshToken } = req.cookies;

    if (!refreshToken) {
      return reply.code(401).send({ error: "No refresh token" });
    }

    try {
      const { accessToken, newRefreshToken } =
        await this.tokenService.rotateRefreshToken(refreshToken, req.ip);

      // Update cookie
      reply.setCookie("refreshToken", newRefreshToken, {
        httpOnly: true,
        secure: true,
        sameSite: "strict",
        maxAge: 30 * 24 * 60 * 60 * 1000
      });

      reply.send({ accessToken });
    } catch (error) {
      reply.code(401).send({ error: "Invalid refresh token" });
    }
  }

  async logout(req: FastifyRequest, reply: FastifyReply) {
    const refreshToken = req.cookies.refreshToken;

    // Revoke token
    if (refreshToken) {
      await this.authService.revokeRefreshToken(refreshToken);
    }

    // Clear cookie
    reply.clearCookie("refreshToken");

    reply.send({ message: "Logged out successfully" });
  }
}

// src/api/auth/auth.routes.ts
export async function authRoutes(fastify: FastifyInstance) {
  const controller = new AuthController(
    fastify.authService,
    fastify.userService,
    fastify.tokenService
  );

  fastify.post(
    "/auth/login",
    { schema: { body: LoginSchema } },
    (req, reply) => controller.login(req, reply)
  );

  fastify.post(
    "/auth/register",
    { schema: { body: RegisterSchema } },
    (req, reply) => controller.register(req, reply)
  );

  fastify.post(
    "/auth/refresh",
    (req, reply) => controller.refreshToken(req, reply)
  );

  fastify.post(
    "/auth/logout",
    { onRequest: [fastify.authenticate] },
    (req, reply) => controller.logout(req, reply)
  );
}
```

### 2. Module Commandes (Orders)

```typescript
// src/domain/order/order.state-machine.ts
export enum OrderStatus {
  PENDING = "pending",
  CONFIRMED = "confirmed",
  PROCESSING = "processing",
  SHIPPED = "shipped",
  DELIVERED = "delivered",
  CANCELLED = "cancelled",
  REFUNDED = "refunded"
}

export class OrderStateMachine {
  private currentState: OrderStatus;

  constructor(initialState: OrderStatus) {
    this.currentState = initialState;
  }

  transition(action: string): OrderStatus {
    const transitions: Record<OrderStatus, Record<string, OrderStatus>> = {
      [OrderStatus.PENDING]: {
        confirm: OrderStatus.CONFIRMED,
        cancel: OrderStatus.CANCELLED
      },
      [OrderStatus.CONFIRMED]: {
        process: OrderStatus.PROCESSING,
        cancel: OrderStatus.CANCELLED
      },
      [OrderStatus.PROCESSING]: {
        ship: OrderStatus.SHIPPED,
        cancel: OrderStatus.CANCELLED
      },
      [OrderStatus.SHIPPED]: {
        deliver: OrderStatus.DELIVERED,
        cancel: OrderStatus.CANCELLED
      },
      [OrderStatus.DELIVERED]: {
        refund: OrderStatus.REFUNDED
      },
      [OrderStatus.CANCELLED]: {},
      [OrderStatus.REFUNDED]: {}
    };

    const nextState = transitions[this.currentState]?.[action];
    if (!nextState) {
      throw new InvalidStateTransitionError(
        `Cannot ${action} from ${this.currentState}`
      );
    }

    this.currentState = nextState;
    return nextState;
  }

  canTransition(action: string): boolean {
    const validActions = Object.keys(
      transitions[this.currentState] || {}
    );
    return validActions.includes(action);
  }
}

// src/domain/order/order.service.ts
export class OrderService {
  constructor(
    private orderRepository: OrderRepository,
    private paymentService: PaymentService,
    private deliveryService: DeliveryService,
    private walletService: WalletService,
    private queue: Queue,
    private eventBus: EventBus
  ) {}

  async createOrder(customerId: bigint, input: CreateOrderInput): Promise<Order> {
    // 1. Validate
    const customer = await this.userService.getById(customerId);
    if (!customer) throw new NotFoundError("Customer not found");

    // 2. Calculate totals
    const orderItems = await Promise.all(
      input.items.map(item => this.validateAndPriceItem(item))
    );

    const subtotal = orderItems.reduce((sum, item) => sum + item.totalPrice, 0);
    const deliveryFee = await this.deliveryService.calculateFee(
      input.deliveryAddressId
    );
    const tax = this.calculateTax(subtotal);
    const total = subtotal + deliveryFee + tax;

    // 3. Create order in transaction
    const order = await this.orderRepository.create({
      orderNumber: this.generateOrderNumber(),
      customerId,
      sellerId: input.sellerId,  // For single-seller order
      status: OrderStatus.PENDING,
      itemsCount: orderItems.length,
      subtotal,
      deliveryFee,
      taxAmount: tax,
      total,
      customerNotes: input.notes,
      deliveryAddressId: input.deliveryAddressId,
      items: orderItems
    });

    // 4. Reserve inventory
    for (const item of orderItems) {
      await this.productService.reserveStock(item.productId, item.quantity);
    }

    // 5. Reserve wallet balance (for wallet-based payment)
    if (input.paymentMethod === "wallet") {
      await this.walletService.reserve(customerId, total);
    }

    // 6. Emit events
    this.eventBus.emit("order.created", {
      orderId: order.id,
      customerId,
      total: order.total
    });

    // 7. Queue notifications
    await this.queue.add("send-notification", {
      userId: customerId,
      type: "order_confirmation",
      orderId: order.id
    });

    return order;
  }

  async confirmPayment(orderId: bigint, paymentId: bigint): Promise<void> {
    const order = await this.orderRepository.getById(orderId);
    const payment = await this.paymentService.getById(paymentId);

    if (payment.status !== "completed") {
      throw new ValidationError("Payment not completed");
    }

    // Update order status
    const stateMachine = new OrderStateMachine(order.status as OrderStatus);
    const newStatus = stateMachine.transition("confirm");

    await this.orderRepository.update(orderId, {
      status: newStatus,
      statusHistory: [
        ...(order.statusHistory || []),
        { status: newStatus, timestamp: new Date() }
      ]
    });

    // Credit seller wallet
    await this.walletService.credit(order.sellerId, order.total);

    // Create delivery
    await this.deliveryService.createDelivery(orderId);

    // Emit event
    this.eventBus.emit("order.confirmed", { orderId });
  }

  async cancelOrder(orderId: bigint, reason: string): Promise<void> {
    const order = await this.orderRepository.getById(orderId);

    // Only pending/confirmed orders can be cancelled
    if (![OrderStatus.PENDING, OrderStatus.CONFIRMED].includes(order.status)) {
      throw new ValidationError(`Cannot cancel ${order.status} order`);
    }

    // Release inventory
    for (const item of order.items) {
      await this.productService.releaseStock(item.productId, item.quantity);
    }

    // Refund wallet
    if (order.paymentMethod === "wallet") {
      await this.walletService.release(order.customerId, order.total);
    }

    // Update status
    await this.orderRepository.update(orderId, {
      status: OrderStatus.CANCELLED,
      cancelledAt: new Date()
    });

    // Emit event
    this.eventBus.emit("order.cancelled", { orderId, reason });
  }
}
```

---

## Validation & Schemas

```typescript
// src/utils/validators.ts
import { z } from "zod";

export const StringValidators = {
  email: z.string().email("Invalid email format"),
  phone: z.string().regex(/^\+?[0-9]{10,14}$/, "Invalid phone number"),
  url: z.string().url("Invalid URL"),
  uuid: z.string().uuid("Invalid UUID"),
  slug: z.string().regex(/^[a-z0-9]+(?:-[a-z0-9]+)*$/, "Invalid slug format"),
  xafAmount: z.number().positive().multipleOf(0.01, "Amount must have max 2 decimals"),
  percentage: z.number().min(0).max(100)
};

export const createPasswordSchema = () =>
  z.string()
    .min(8, "Password must be at least 8 characters")
    .regex(/[A-Z]/, "Must contain uppercase letter")
    .regex(/[0-9]/, "Must contain number")
    .regex(/[!@#$%^&*]/, "Must contain special character");

// Reusable API response schema
export const PaginatedResponse = <T extends z.ZodTypeAny>(itemSchema: T) =>
  z.object({
    items: z.array(itemSchema),
    pagination: z.object({
      page: z.number().int().positive(),
      limit: z.number().int().positive(),
      total: z.number().int(),
      hasMore: z.boolean()
    })
  });

export const ApiResponse = <T extends z.ZodTypeAny>(dataSchema: T) =>
  z.object({
    data: dataSchema,
    meta: z.object({
      requestId: z.string().uuid(),
      timestamp: z.string().datetime()
    }).optional()
  });
```

---

## Error Handling

```typescript
// src/utils/errors.ts
export class AppError extends Error {
  constructor(
    public code: string,
    public message: string,
    public statusCode: number = 400,
    public details?: Record<string, any>
  ) {
    super(message);
    Object.setPrototypeOf(this, AppError.prototype);
  }
}

export class ValidationError extends AppError {
  constructor(message: string, details?: Record<string, any>) {
    super("VALIDATION_ERROR", message, 400, details);
  }
}

export class UnauthorizedError extends AppError {
  constructor(message = "Unauthorized") {
    super("UNAUTHORIZED", message, 401);
  }
}

export class ForbiddenError extends AppError {
  constructor(message = "Forbidden") {
    super("FORBIDDEN", message, 403);
  }
}

export class NotFoundError extends AppError {
  constructor(resource: string) {
    super("NOT_FOUND", `${resource} not found`, 404);
  }
}

export class ConflictError extends AppError {
  constructor(message: string) {
    super("CONFLICT", message, 409);
  }
}

export class RateLimitError extends AppError {
  constructor(retryAfter: number) {
    super("RATE_LIMITED", "Too many requests", 429, { retryAfter });
  }
}

// Global error handler middleware
export async function errorHandler(err: Error, request: FastifyRequest, reply: FastifyReply) {
  const requestId = request.id;

  if (err instanceof AppError) {
    logger.warn({
      requestId,
      error: err.code,
      message: err.message,
      details: err.details
    });

    return reply.code(err.statusCode).send({
      error: {
        code: err.code,
        message: err.message,
        details: err.details,
        requestId
      }
    });
  }

  // Unknown error
  logger.error({
    requestId,
    error: err.message,
    stack: err.stack
  });

  reply.code(500).send({
    error: {
      code: "INTERNAL_ERROR",
      message: "Internal server error",
      requestId
    }
  });
}
```

---

## Middleware Stack

```typescript
// src/middleware/index.ts
export async function installMiddleware(fastify: FastifyInstance) {
  // 1. Security
  await fastify.register(import("@fastify/helmet"));
  await fastify.register(import("@fastify/cors"), {
    origin: process.env.CORS_ORIGINS?.split(","),
    credentials: true
  });

  // 2. Request context
  fastify.addHook("preHandler", (req, reply, done) => {
    req.id = req.headers["x-request-id"] || crypto.randomUUID();
    req.startTime = Date.now();
    done();
  });

  // 3. Request logging
  fastify.addHook("onResponse", (req, reply, done) => {
    const duration = Date.now() - req.startTime;
    logger.info({
      requestId: req.id,
      method: req.method,
      url: req.url,
      statusCode: reply.statusCode,
      duration
    });
    done();
  });

  // 4. Global error handler
  fastify.setErrorHandler(errorHandler);

  // 5. Rate limiting
  await fastify.register(import("@fastify/rate-limit"), {
    max: 100,
    timeWindow: "1 minute",
    cache: 10000,
    allowList: ["127.0.0.1"],
    redis: redisClient
  });
}
```

---

## Development Workflow

```bash
# Install dependencies
pnpm install

# Setup environment
cp .env.example .env.local

# Start local services (PostgreSQL, Redis)
docker-compose up -d

# Run migrations
pnpm run migrate

# Seed test data
pnpm run seed

# Start dev server
pnpm run dev

# Run tests
pnpm run test

# Run tests with coverage
pnpm run test:coverage

# Lint
pnpm run lint

# Build for production
pnpm run build

# Start production server
pnpm run start
```

---

**Backend Architecture Document**  
Updated: May 27, 2026
