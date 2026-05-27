# KoloShop - Audit Architectural & Transformation Production

**Document:** Architecture Production-Ready E-Commerce Afrique Centrale  
**Date:** Mai 2026  
**Niveau:** Enterprise Architecture  
**Scope:** MVP → Production à 100K utilisateurs  

---

## Table des matières

1. [Analyse Critique Existant](#1-analyse-critique-existant)
2. [Architecture Globale Recommandée](#2-architecture-globale-recommandée)
3. [Stack Technique Final](#3-stack-technique-final)
4. [Base de Données](#4-base-de-données)
5. [Authentification & Sécurité](#5-authentification--sécurité)
6. [Architecture Backend](#6-architecture-backend)
7. [Mobile Money & Paiements](#7-mobile-money--paiements)
8. [Livraison & Logistique](#8-livraison--logistique)
9. [Frontend](#9-frontend)
10. [DevOps & Infrastructure](#10-devops--infrastructure)
11. [Observabilité & Monitoring](#11-observabilité--monitoring)
12. [Scalabilité](#12-scalabilité)
13. [Business Logic](#13-business-logic)
14. [Production Readiness](#14-production-readiness)
15. [Risques Critiques](#15-risques-critiques)

---

## 1. Analyse Critique Existant

### État Actuel
- **Status:** MVP vierge (seulement README)
- **Avancement:** 0%
- **Infrastructure:** Aucune
- **Code:** Aucun

### Faiblesses Identifiées (si implémentation naïve)

| Catégorie | Problème | Impact | Priorité |
|-----------|---------|--------|----------|
| **Données** | Pas de schéma DB | Perte données, incohérence | CRITIQUE |
| **Sécurité** | Pas d'authentification | Accès non autorisé | CRITIQUE |
| **Paiements** | Absence architecture MM | Fraude, pertes $$ | CRITIQUE |
| **Livraison** | Pas de tracking | Insatisfaction clients | HAUTE |
| **Scalabilité** | Architecture monolithique | Limite croissance | HAUTE |
| **Monitoring** | Zéro observabilité | Blind in production | HAUTE |
| **DevOps** | Pas de CI/CD | Déploiements manuels | MOYENNE |

### Opportunités Critiques
- **Afrique Centrale:** Intégration Mobile Money (Moov, Orange, Wagni)
- **Offline-first:** Régions avec connectivité intermittente
- **Multi-devise:** Francs CFA, éventuellement autres monnaies
- **Vendeurs locaux:** Platform marketplace viable
- **Mobile-first:** 90% des accès via mobile

---

## 2. Architecture Globale Recommandée

### Vue d'ensemble

```
┌─────────────────────────────────────────────────────────────────┐
│                         CLIENTS                                  │
│  ┌──────────────────┐  ┌──────────────────┐  ┌────────────────┐ │
│  │  Mobile App      │  │  Web (Next.js)   │  │  Admin Portal  │ │
│  │  (React Native)  │  │                  │  │                │ │
│  └────────┬─────────┘  └─────────┬────────┘  └────────┬───────┘ │
└───────────┼────────────────────────┼───────────────────┼─────────┘
            │                        │                   │
            └────────────┬───────────┴───────────────────┘
                         │
         ┌───────────────▼───────────────┐
         │   API Gateway / LB            │
         │   • Auth middleware           │
         │   • Rate limiting             │
         │   • Request validation        │
         └───────────────┬───────────────┘
                         │
        ┌────────────────┼────────────────┐
        │                │                │
   ┌────▼─────┐    ┌────▼─────┐    ┌────▼─────┐
   │ Core API │    │  Worker  │    │  WebHook │
   │ (Fastify)│    │ Processor│    │ Handler  │
   │          │    │(Bull)    │    │(Fastify) │
   └────┬─────┘    └────┬─────┘    └────┬─────┘
        │                │               │
        └────────────────┼───────────────┘
                         │
        ┌────────────────┼────────────────┐
        │                │                │
   ┌────▼──────┐  ┌─────▼──────┐  ┌─────▼──────┐
   │PostgreSQL │  │Redis Cache │  │ S3 Storage │
   │           │  │            │  │ (Files)    │
   └───────────┘  └────────────┘  └────────────┘
        │                │               │
        └────────────────┼───────────────┘
                         │
        ┌────────────────┼────────────────┐
        │                │                │
   ┌────▼──────┐  ┌─────▼──────┐  ┌─────▼──────┐
   │ Logging   │  │ Metrics    │  │ Alerting   │
   │ (Loki)    │  │(Prometheus)│  │(AlertMgr)  │
   └───────────┘  └────────────┘  └────────────┘
```

### Composants Clés

#### Frontend Tier
- **Mobile:** React Native (iOS/Android) - offline-capable
- **Web:** Next.js 14 - SSR, SSG, API routes
- **Admin:** Next.js SPA - dashboard temps réel

#### API Tier
- **Gateway:** Nginx + Lua (VPS) ou AWS ALB
- **Services:** 3× instances Fastify (scalable horizontalement)
- **Communication:** gRPC interne, REST public

#### Data Tier
- **DB Principale:** PostgreSQL avec replicas
- **Cache:** Redis cluster
- **Files:** S3/MinIO
- **Queue:** Bull (RabbitMQ backend)

#### Background Tier
- **Workers:** Bull queue processors
- **Cron:** node-cron (migrations, reconciliation)
- **WebHooks:** Incoming handlers

#### Observability
- **Logs:** Loki (temps réel)
- **Metrics:** Prometheus
- **Tracing:** Optional Jaeger
- **Alerting:** AlertManager

---

## 3. Stack Technique Final

### Backend Framework
**Recommandation: FASTIFY** (vs Express/NestJS)

| Aspect | Fastify | Express | NestJS |
|--------|---------|---------|--------|
| **Performance** | ⭐⭐⭐⭐⭐ 4x+ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Validation** | ⭐⭐⭐⭐⭐ JSON Schema | ⭐⭐ Manual | ⭐⭐⭐⭐⭐ |
| **Async** | Native Promise | Callback | Strong |
| **TypeScript** | Native support | Via ts-node | Parfait |
| **Plugins** | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Learning** | Facile | Très facile | Complexe |
| **Production** | Excellent | OK | Excellent |
| **Poids** | 105 KB | 440 KB | 1.4 MB |

**Verdict:** Fastify pour performance + TypeScript + plugins robustes

### Stack Complet Recommandé

```yaml
Backend:
  Runtime: Node.js 22 LTS
  Framework: Fastify 5.x
  Language: TypeScript 5.x
  ORM: Prisma 6.x
  Validation: Zod
  Queue: Bull 5.x (avec Redis)
  Auth: JWT + refresh tokens
  API Docs: Swagger/OpenAPI
  Testing: Vitest + Supertest

Frontend Mobile:
  Framework: React Native 0.75
  State Mgmt: Redux Toolkit
  API Client: TanStack Query
  Offline: WatermelonDB
  Sync: Custom conflict-free sync
  Push: Expo Notifications
  Maps: React Native Maps

Frontend Web:
  Framework: Next.js 14
  Styling: Tailwind CSS 4
  State: TanStack Query + Zustand
  Forms: React Hook Form
  UI Components: shadcn/ui
  Charts: Recharts
  Analytics: PostHog

Databases:
  Transactional: PostgreSQL 16
  Cache: Redis 7
  Search: Elasticsearch 8 (optional)
  Files: S3 / MinIO

Infrastructure:
  Containerization: Docker + Docker Compose
  Orchestration: Optional Kubernetes (future)
  VPS: Hetzner / OVH / AWS
  CDN: CloudFlare
  DNS: CloudFlare
  Email: SendGrid
  SMS: Twilio

DevOps:
  CI/CD: GitHub Actions
  Container Registry: Docker Hub / GHCR
  Monitoring: Prometheus + Grafana
  Logging: Loki + Promtail
  Secrets: HashiCorp Vault (prod)

Payment Providers:
  Mobile Money: Moov / Orange Money / Wagni
  Card Payments: Stripe / Wise
  Webhooks: Secure signed+encrypted

Development:
  Version Control: Git + GitHub
  Package Manager: pnpm
  Task Runner: Turborepo (monorepo)
  Linting: ESLint + Prettier
  Secrets: .env.local + Doppler (prod)
```

---

## 4. Base de Données

### Schéma PostgreSQL Complet

```sql
-- ============================================
-- AUTHENTIFICATION & UTILISATEURS
-- ============================================

CREATE TABLE users (
  id BIGSERIAL PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  phone VARCHAR(20) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  password_salt VARCHAR(255) NOT NULL,
  first_name VARCHAR(100) NOT NULL,
  last_name VARCHAR(100) NOT NULL,
  avatar_url TEXT,
  
  -- Profil
  user_type VARCHAR(20) NOT NULL, -- 'customer', 'seller', 'rider', 'admin'
  status VARCHAR(20) NOT NULL DEFAULT 'active', -- 'active', 'suspended', 'banned'
  
  -- Localisation
  country_code VARCHAR(2) NOT NULL DEFAULT 'CM', -- ISO 3166-1 alpha-2
  timezone VARCHAR(50) DEFAULT 'Africa/Douala',
  preferred_language VARCHAR(2) DEFAULT 'fr',
  
  -- Données sensibles chiffrées
  national_id_encrypted BYTEA, -- Chiffré avec nacl.secretbox
  national_id_hash VARCHAR(255) UNIQUE,
  
  -- KYC
  kyc_status VARCHAR(20) DEFAULT 'pending', -- 'pending', 'verified', 'rejected', 'recheck'
  kyc_verified_at TIMESTAMP,
  kyc_data JSONB, -- Métadonnées KYC {provider, score, checks}
  
  -- Sécurité
  mfa_enabled BOOLEAN DEFAULT FALSE,
  mfa_secret VARCHAR(255),
  email_verified BOOLEAN DEFAULT FALSE,
  email_verified_at TIMESTAMP,
  phone_verified BOOLEAN DEFAULT FALSE,
  phone_verified_at TIMESTAMP,
  
  -- Audit
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  deleted_at TIMESTAMP, -- Soft delete
  last_login_at TIMESTAMP,
  
  CONSTRAINT email_not_empty CHECK (email != ''),
  CONSTRAINT phone_not_empty CHECK (phone != '')
);

CREATE INDEX idx_users_email ON users(email) WHERE deleted_at IS NULL;
CREATE INDEX idx_users_phone ON users(phone) WHERE deleted_at IS NULL;
CREATE INDEX idx_users_user_type ON users(user_type) WHERE deleted_at IS NULL;
CREATE INDEX idx_users_kyc_status ON users(kyc_status);
CREATE INDEX idx_users_created_at ON users(created_at DESC);

-- ============================================
-- AUTHENTIFICATION - SESSIONS & TOKENS
-- ============================================

CREATE TABLE refresh_tokens (
  id BIGSERIAL PRIMARY KEY,
  user_id BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  token_hash VARCHAR(255) UNIQUE NOT NULL,
  token_family VARCHAR(255), -- Pour détecter token rotation attacks
  
  expires_at TIMESTAMP NOT NULL,
  revoked_at TIMESTAMP,
  revoked_reason VARCHAR(255),
  
  -- Contexte
  ip_address INET,
  user_agent TEXT,
  device_fingerprint VARCHAR(255),
  
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT not_expired_or_revoked CHECK (expires_at > CURRENT_TIMESTAMP OR revoked_at IS NOT NULL)
);

CREATE INDEX idx_refresh_tokens_user_id ON refresh_tokens(user_id);
CREATE INDEX idx_refresh_tokens_expires_at ON refresh_tokens(expires_at);

CREATE TABLE audit_logs (
  id BIGSERIAL PRIMARY KEY,
  user_id BIGINT REFERENCES users(id) ON DELETE SET NULL,
  action VARCHAR(100) NOT NULL,
  resource_type VARCHAR(50),
  resource_id BIGINT,
  changes JSONB, -- {before, after}
  ip_address INET,
  user_agent TEXT,
  status VARCHAR(20), -- 'success', 'failure'
  error_message TEXT,
  
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_audit_logs_user_id ON audit_logs(user_id);
CREATE INDEX idx_audit_logs_action ON audit_logs(action);
CREATE INDEX idx_audit_logs_created_at ON audit_logs(created_at DESC);
CREATE INDEX idx_audit_logs_resource ON audit_logs(resource_type, resource_id);

-- ============================================
-- RÔLES & PERMISSIONS
-- ============================================

CREATE TABLE roles (
  id BIGSERIAL PRIMARY KEY,
  code VARCHAR(50) UNIQUE NOT NULL, -- 'admin', 'seller', 'rider', 'customer'
  name VARCHAR(100) NOT NULL,
  description TEXT,
  is_system BOOLEAN DEFAULT TRUE, -- Immuable
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE permissions (
  id BIGSERIAL PRIMARY KEY,
  code VARCHAR(100) UNIQUE NOT NULL, -- 'users.create', 'orders.read', etc.
  name VARCHAR(100) NOT NULL,
  resource VARCHAR(50), -- 'users', 'orders', 'products'
  action VARCHAR(20), -- 'create', 'read', 'update', 'delete'
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE role_permissions (
  role_id BIGINT NOT NULL REFERENCES roles(id) ON DELETE CASCADE,
  permission_id BIGINT NOT NULL REFERENCES permissions(id) ON DELETE CASCADE,
  PRIMARY KEY (role_id, permission_id)
);

CREATE TABLE user_roles (
  user_id BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  role_id BIGINT NOT NULL REFERENCES roles(id) ON DELETE CASCADE,
  assigned_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  assigned_by BIGINT REFERENCES users(id) ON DELETE SET NULL,
  PRIMARY KEY (user_id, role_id)
);

-- ============================================
-- VENDEURS (SELLERS)
-- ============================================

CREATE TABLE sellers (
  id BIGSERIAL PRIMARY KEY,
  user_id BIGINT UNIQUE NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
  
  -- Informations commerciales
  business_name VARCHAR(255) NOT NULL,
  business_registration VARCHAR(255),
  business_registration_verified BOOLEAN DEFAULT FALSE,
  
  -- Localisation
  country_code VARCHAR(2) NOT NULL,
  city VARCHAR(100) NOT NULL,
  latitude DECIMAL(10, 8),
  longitude DECIMAL(11, 8),
  address TEXT,
  
  -- Paiements
  primary_payment_method VARCHAR(50), -- 'mobile_money', 'bank_transfer'
  mobile_money_number VARCHAR(20),
  mobile_money_operator VARCHAR(50), -- 'moov', 'orange', 'wagni'
  bank_account_encrypted BYTEA,
  
  -- Vérification
  verification_status VARCHAR(20) DEFAULT 'pending', -- 'pending', 'verified', 'rejected'
  verification_data JSONB,
  verified_at TIMESTAMP,
  
  -- Stats
  total_sales DECIMAL(15, 2) DEFAULT 0,
  total_orders INT DEFAULT 0,
  rating DECIMAL(3, 2) DEFAULT 0,
  rating_count INT DEFAULT 0,
  
  -- Documents
  identity_document_url TEXT,
  proof_of_address_url TEXT,
  
  -- Activation
  is_active BOOLEAN DEFAULT FALSE,
  suspended_at TIMESTAMP,
  suspension_reason TEXT,
  
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  deleted_at TIMESTAMP
);

CREATE INDEX idx_sellers_user_id ON sellers(user_id);
CREATE INDEX idx_sellers_verification_status ON sellers(verification_status);
CREATE INDEX idx_sellers_country_code ON sellers(country_code);
CREATE INDEX idx_sellers_coordinates ON sellers USING GIST (ll_to_earth(latitude, longitude));

-- ============================================
-- PRODUITS & CATALOGUE
-- ============================================

CREATE TABLE categories (
  id BIGSERIAL PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  slug VARCHAR(255) UNIQUE NOT NULL,
  description TEXT,
  icon_url TEXT,
  parent_id BIGINT REFERENCES categories(id) ON DELETE SET NULL,
  position INT DEFAULT 0,
  is_active BOOLEAN DEFAULT TRUE,
  
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_categories_slug ON categories(slug);
CREATE INDEX idx_categories_parent_id ON categories(parent_id);

CREATE TABLE products (
  id BIGSERIAL PRIMARY KEY,
  seller_id BIGINT NOT NULL REFERENCES sellers(id) ON DELETE RESTRICT,
  category_id BIGINT NOT NULL REFERENCES categories(id) ON DELETE RESTRICT,
  
  -- Informations produit
  name VARCHAR(255) NOT NULL,
  slug VARCHAR(255) NOT NULL,
  description TEXT,
  short_description VARCHAR(500),
  
  -- Tarification
  base_price DECIMAL(15, 2) NOT NULL CHECK (base_price > 0),
  cost_price DECIMAL(15, 2),
  discount_price DECIMAL(15, 2),
  discount_percentage DECIMAL(5, 2),
  
  -- Inventaire
  stock_quantity INT NOT NULL DEFAULT 0,
  low_stock_threshold INT DEFAULT 10,
  
  -- Images
  thumbnail_url TEXT,
  images JSONB, -- [{url, alt, position}]
  
  -- SEO
  meta_title VARCHAR(255),
  meta_description VARCHAR(500),
  meta_keywords TEXT,
  
  -- Status
  status VARCHAR(20) DEFAULT 'draft', -- 'draft', 'active', 'archived'
  
  -- Stats
  views_count INT DEFAULT 0,
  rating DECIMAL(3, 2) DEFAULT 0,
  rating_count INT DEFAULT 0,
  
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  deleted_at TIMESTAMP
);

CREATE UNIQUE INDEX idx_products_seller_slug ON products(seller_id, slug) WHERE deleted_at IS NULL;
CREATE INDEX idx_products_category_id ON products(category_id);
CREATE INDEX idx_products_status ON products(status);
CREATE INDEX idx_products_created_at ON products(created_at DESC);

CREATE TABLE product_variants (
  id BIGSERIAL PRIMARY KEY,
  product_id BIGINT NOT NULL REFERENCES products(id) ON DELETE CASCADE,
  
  -- Variants
  name VARCHAR(255), -- 'Taille', 'Couleur', etc.
  value VARCHAR(100), -- 'M', 'Rouge', etc.
  sku VARCHAR(100) UNIQUE NOT NULL,
  
  -- Pricing
  price_adjustment DECIMAL(15, 2) DEFAULT 0,
  
  -- Stock
  stock_quantity INT DEFAULT 0,
  
  -- Images
  image_url TEXT,
  
  position INT DEFAULT 0,
  
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_product_variants_product_id ON product_variants(product_id);
CREATE INDEX idx_product_variants_sku ON product_variants(sku);

-- ============================================
-- COMMANDES & PAIEMENTS
-- ============================================

CREATE TABLE orders (
  id BIGSERIAL PRIMARY KEY,
  order_number VARCHAR(50) UNIQUE NOT NULL, -- 'ORD-2024-001234'
  customer_id BIGINT NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
  seller_id BIGINT NOT NULL REFERENCES sellers(id) ON DELETE RESTRICT,
  
  -- Status
  status VARCHAR(20) NOT NULL DEFAULT 'pending', -- 'pending', 'confirmed', 'processing', 'shipped', 'delivered', 'cancelled', 'refunded'
  status_history JSONB, -- [{status, timestamp, reason}]
  
  -- Adresse livraison
  delivery_address_id BIGINT REFERENCES addresses(id),
  
  -- Articles
  items_count INT NOT NULL,
  subtotal DECIMAL(15, 2) NOT NULL,
  delivery_fee DECIMAL(15, 2) NOT NULL,
  discount_amount DECIMAL(15, 2) DEFAULT 0,
  discount_reason VARCHAR(255),
  tax_amount DECIMAL(15, 2) DEFAULT 0,
  total DECIMAL(15, 2) NOT NULL,
  
  currency VARCHAR(3) DEFAULT 'XAF',
  
  -- Notes
  customer_notes TEXT,
  seller_notes TEXT,
  
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  delivered_at TIMESTAMP,
  cancelled_at TIMESTAMP,
  
  CONSTRAINT total_positive CHECK (total > 0)
);

CREATE INDEX idx_orders_customer_id ON orders(customer_id);
CREATE INDEX idx_orders_seller_id ON orders(seller_id);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_orders_created_at ON orders(created_at DESC);
CREATE INDEX idx_orders_order_number ON orders(order_number);

CREATE TABLE order_items (
  id BIGSERIAL PRIMARY KEY,
  order_id BIGINT NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
  product_id BIGINT NOT NULL REFERENCES products(id) ON DELETE RESTRICT,
  product_variant_id BIGINT REFERENCES product_variants(id) ON DELETE SET NULL,
  
  product_name VARCHAR(255) NOT NULL,
  product_price DECIMAL(15, 2) NOT NULL,
  product_image_url TEXT,
  
  quantity INT NOT NULL CHECK (quantity > 0),
  unit_price DECIMAL(15, 2) NOT NULL,
  total_price DECIMAL(15, 2) NOT NULL,
  
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_order_items_order_id ON order_items(order_id);
CREATE INDEX idx_order_items_product_id ON order_items(product_id);

-- ============================================
-- PAIEMENTS (Mobile Money & Autres)
-- ============================================

CREATE TABLE payment_methods (
  id BIGSERIAL PRIMARY KEY,
  user_id BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  
  method_type VARCHAR(50) NOT NULL, -- 'mobile_money', 'card', 'bank_transfer'
  provider VARCHAR(50), -- 'moov', 'orange', 'stripe', 'wise'
  
  -- Données sensibles chiffrées
  data_encrypted BYTEA, -- {phone, cardToken, etc.}
  
  -- Métadonnées
  is_default BOOLEAN DEFAULT FALSE,
  display_name VARCHAR(255),
  last_four VARCHAR(4),
  
  is_active BOOLEAN DEFAULT TRUE,
  verified_at TIMESTAMP,
  
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_payment_methods_user_id ON payment_methods(user_id);
CREATE INDEX idx_payment_methods_is_default ON payment_methods(user_id, is_default);

CREATE TABLE payments (
  id BIGSERIAL PRIMARY KEY,
  payment_id VARCHAR(100) UNIQUE NOT NULL, -- UUID v4 pour idempotence
  order_id BIGINT NOT NULL REFERENCES orders(id) ON DELETE RESTRICT,
  user_id BIGINT NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
  
  amount DECIMAL(15, 2) NOT NULL,
  currency VARCHAR(3) DEFAULT 'XAF',
  
  payment_method_id BIGINT REFERENCES payment_methods(id) ON DELETE SET NULL,
  payment_method_type VARCHAR(50),
  provider VARCHAR(50), -- 'moov', 'orange', 'stripe', etc.
  provider_transaction_id VARCHAR(255) UNIQUE,
  
  -- Status détaillé
  status VARCHAR(30) DEFAULT 'pending', -- 'pending', 'processing', 'authorized', 'captured', 'failed', 'cancelled', 'refunded'
  status_history JSONB, -- [{status, timestamp, provider_status}]
  
  -- Retry logic
  attempt_count INT DEFAULT 0,
  last_attempt_at TIMESTAMP,
  last_error_code VARCHAR(50),
  last_error_message TEXT,
  
  -- Webhook
  webhook_received_at TIMESTAMP,
  webhook_signature_verified BOOLEAN,
  webhook_data JSONB,
  
  -- Audit
  ip_address INET,
  
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  completed_at TIMESTAMP
);

CREATE INDEX idx_payments_order_id ON payments(order_id);
CREATE INDEX idx_payments_user_id ON payments(user_id);
CREATE INDEX idx_payments_status ON payments(status);
CREATE INDEX idx_payments_provider ON payments(provider);
CREATE INDEX idx_payments_created_at ON payments(created_at DESC);
CREATE INDEX idx_payments_payment_id ON payments(payment_id);

-- ============================================
-- RÉCONCILIATION & FRAUDE
-- ============================================

CREATE TABLE payment_reconciliation (
  id BIGSERIAL PRIMARY KEY,
  payment_id BIGINT NOT NULL REFERENCES payments(id) ON DELETE RESTRICT,
  
  local_amount DECIMAL(15, 2),
  provider_amount DECIMAL(15, 2),
  difference DECIMAL(15, 2),
  
  status VARCHAR(20) DEFAULT 'pending', -- 'pending', 'reconciled', 'discrepancy', 'investigated'
  
  notes TEXT,
  
  reconciled_at TIMESTAMP,
  reconciled_by BIGINT REFERENCES users(id) ON DELETE SET NULL,
  
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE fraud_alerts (
  id BIGSERIAL PRIMARY KEY,
  user_id BIGINT REFERENCES users(id) ON DELETE SET NULL,
  payment_id BIGINT REFERENCES payments(id) ON DELETE SET NULL,
  
  alert_type VARCHAR(50), -- 'unusual_amount', 'velocity_check', 'country_mismatch', 'device_change'
  severity VARCHAR(20), -- 'low', 'medium', 'high', 'critical'
  
  risk_score DECIMAL(5, 2), -- 0-100
  risk_factors JSONB,
  
  action_taken VARCHAR(50), -- 'none', 'review', 'block', 'mfa_required'
  
  is_false_positive BOOLEAN,
  
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  resolved_at TIMESTAMP
);

-- ============================================
-- LIVRAISON & TRACKING
-- ============================================

CREATE TABLE delivery_zones (
  id BIGSERIAL PRIMARY KEY,
  country_code VARCHAR(2) NOT NULL,
  city VARCHAR(100) NOT NULL,
  area_name VARCHAR(255) NOT NULL,
  
  -- Zone géographique
  coordinates GEOMETRY(Polygon), -- PostGIS
  
  -- Frais
  base_delivery_fee DECIMAL(15, 2),
  per_km_fee DECIMAL(15, 2),
  
  -- Délais
  estimated_delivery_hours_min INT,
  estimated_delivery_hours_max INT,
  
  is_active BOOLEAN DEFAULT TRUE,
  
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_delivery_zones_country_city ON delivery_zones(country_code, city);

CREATE TABLE deliveries (
  id BIGSERIAL PRIMARY KEY,
  order_id BIGINT UNIQUE NOT NULL REFERENCES orders(id) ON DELETE RESTRICT,
  rider_id BIGINT REFERENCES users(id) ON DELETE SET NULL,
  
  -- Adresses
  pickup_address_id BIGINT NOT NULL REFERENCES addresses(id),
  delivery_address_id BIGINT NOT NULL REFERENCES addresses(id),
  
  status VARCHAR(20) DEFAULT 'pending', -- 'pending', 'assigned', 'picked_up', 'in_transit', 'delivered', 'failed', 'cancelled'
  status_history JSONB,
  
  -- Localisation temps réel
  current_latitude DECIMAL(10, 8),
  current_longitude DECIMAL(11, 8),
  last_location_update TIMESTAMP,
  
  -- Estimés vs réalisé
  estimated_delivery_time TIMESTAMP,
  actual_delivery_time TIMESTAMP,
  
  -- Preuve
  delivery_signature_url TEXT,
  delivery_photo_url TEXT,
  delivery_otp VARCHAR(6),
  
  distance_km DECIMAL(10, 2),
  
  -- Notes
  notes TEXT,
  
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_deliveries_order_id ON deliveries(order_id);
CREATE INDEX idx_deliveries_rider_id ON deliveries(rider_id);
CREATE INDEX idx_deliveries_status ON deliveries(status);

CREATE TABLE delivery_tracking (
  id BIGSERIAL PRIMARY KEY,
  delivery_id BIGINT NOT NULL REFERENCES deliveries(id) ON DELETE CASCADE,
  
  event_type VARCHAR(50), -- 'location_update', 'status_change', 'delay_alert', 'arrival_notification'
  latitude DECIMAL(10, 8),
  longitude DECIMAL(11, 8),
  description TEXT,
  
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_delivery_tracking_delivery_id ON delivery_tracking(delivery_id);

-- ============================================
-- ADRESSES
-- ============================================

CREATE TABLE addresses (
  id BIGSERIAL PRIMARY KEY,
  user_id BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  
  type VARCHAR(20) DEFAULT 'home', -- 'home', 'work', 'other'
  label VARCHAR(100),
  
  country_code VARCHAR(2) NOT NULL,
  city VARCHAR(100) NOT NULL,
  district VARCHAR(100),
  street_address VARCHAR(255) NOT NULL,
  building_name VARCHAR(255),
  apartment_number VARCHAR(50),
  landmark VARCHAR(255),
  
  latitude DECIMAL(10, 8),
  longitude DECIMAL(11, 8),
  
  full_address_text TEXT, -- Texte complète pour recherche
  
  is_default BOOLEAN DEFAULT FALSE,
  is_active BOOLEAN DEFAULT TRUE,
  
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_addresses_user_id ON addresses(user_id);
CREATE INDEX idx_addresses_coordinates ON addresses USING GIST (ll_to_earth(latitude, longitude));
CREATE INDEX idx_addresses_city ON addresses(city);

-- ============================================
-- AVIS & NOTATION
-- ============================================

CREATE TABLE reviews (
  id BIGSERIAL PRIMARY KEY,
  order_id BIGINT UNIQUE REFERENCES orders(id) ON DELETE SET NULL,
  product_id BIGINT NOT NULL REFERENCES products(id) ON DELETE RESTRICT,
  reviewer_id BIGINT NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
  seller_id BIGINT NOT NULL REFERENCES sellers(id) ON DELETE RESTRICT,
  
  rating INT NOT NULL CHECK (rating >= 1 AND rating <= 5),
  title VARCHAR(255),
  comment TEXT,
  
  -- Images
  images JSONB,
  
  -- Modération
  status VARCHAR(20) DEFAULT 'pending', -- 'pending', 'approved', 'rejected'
  
  is_verified_purchase BOOLEAN DEFAULT TRUE,
  
  helpful_count INT DEFAULT 0,
  unhelpful_count INT DEFAULT 0,
  
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_reviews_product_id ON reviews(product_id);
CREATE INDEX idx_reviews_seller_id ON reviews(seller_id);
CREATE INDEX idx_reviews_reviewer_id ON reviews(reviewer_id);
CREATE INDEX idx_reviews_status ON reviews(status);

-- ============================================
-- WALLET & PAYOUT
-- ============================================

CREATE TABLE wallets (
  id BIGSERIAL PRIMARY KEY,
  user_id BIGINT UNIQUE NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
  
  balance DECIMAL(15, 2) NOT NULL DEFAULT 0,
  currency VARCHAR(3) DEFAULT 'XAF',
  
  reserved_for_orders DECIMAL(15, 2) DEFAULT 0, -- Fonds pour commandes en cours
  
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE wallet_transactions (
  id BIGSERIAL PRIMARY KEY,
  transaction_id VARCHAR(100) UNIQUE NOT NULL,
  wallet_id BIGINT NOT NULL REFERENCES wallets(id) ON DELETE RESTRICT,
  
  type VARCHAR(20), -- 'credit', 'debit', 'refund', 'fee', 'payout'
  amount DECIMAL(15, 2) NOT NULL,
  
  reason VARCHAR(255),
  related_order_id BIGINT REFERENCES orders(id) ON DELETE SET NULL,
  related_payment_id BIGINT REFERENCES payments(id) ON DELETE SET NULL,
  
  balance_before DECIMAL(15, 2),
  balance_after DECIMAL(15, 2),
  
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_wallet_transactions_wallet_id ON wallet_transactions(wallet_id);
CREATE INDEX idx_wallet_transactions_type ON wallet_transactions(type);
CREATE INDEX idx_wallet_transactions_created_at ON wallet_transactions(created_at DESC);

CREATE TABLE payouts (
  id BIGSERIAL PRIMARY KEY,
  payout_id VARCHAR(100) UNIQUE NOT NULL,
  seller_id BIGINT NOT NULL REFERENCES sellers(id) ON DELETE RESTRICT,
  wallet_id BIGINT NOT NULL REFERENCES wallets(id) ON DELETE RESTRICT,
  
  amount DECIMAL(15, 2) NOT NULL,
  currency VARCHAR(3) DEFAULT 'XAF',
  
  status VARCHAR(20) DEFAULT 'pending', -- 'pending', 'processing', 'completed', 'failed'
  
  payment_method_type VARCHAR(50), -- 'mobile_money', 'bank_transfer'
  payment_method_details_encrypted BYTEA,
  
  -- Dates
  requested_at TIMESTAMP,
  processed_at TIMESTAMP,
  completed_at TIMESTAMP,
  
  -- Audit
  error_message TEXT,
  retry_count INT DEFAULT 0,
  
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_payouts_seller_id ON payouts(seller_id);
CREATE INDEX idx_payouts_status ON payouts(status);
CREATE INDEX idx_payouts_created_at ON payouts(created_at DESC);

-- ============================================
-- SUPPORT CLIENT
-- ============================================

CREATE TABLE support_tickets (
  id BIGSERIAL PRIMARY KEY,
  ticket_number VARCHAR(50) UNIQUE NOT NULL,
  user_id BIGINT NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
  
  category VARCHAR(50), -- 'payment', 'delivery', 'product', 'account', 'other'
  subject VARCHAR(255) NOT NULL,
  description TEXT NOT NULL,
  
  status VARCHAR(20) DEFAULT 'open', -- 'open', 'in_progress', 'pending_user', 'resolved', 'closed'
  priority VARCHAR(20) DEFAULT 'normal', -- 'low', 'normal', 'high', 'critical'
  
  related_order_id BIGINT REFERENCES orders(id) ON DELETE SET NULL,
  
  assigned_to BIGINT REFERENCES users(id) ON DELETE SET NULL,
  
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  resolved_at TIMESTAMP
);

CREATE INDEX idx_support_tickets_user_id ON support_tickets(user_id);
CREATE INDEX idx_support_tickets_status ON support_tickets(status);

-- ============================================
-- CONFIGURATION & SYSTÈME
-- ============================================

CREATE TABLE system_config (
  id BIGSERIAL PRIMARY KEY,
  key VARCHAR(255) UNIQUE NOT NULL,
  value JSONB NOT NULL,
  type VARCHAR(50), -- 'string', 'number', 'boolean', 'json'
  description TEXT,
  updated_by BIGINT REFERENCES users(id) ON DELETE SET NULL,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE feature_flags (
  id BIGSERIAL PRIMARY KEY,
  name VARCHAR(255) UNIQUE NOT NULL,
  description TEXT,
  is_enabled BOOLEAN DEFAULT FALSE,
  rollout_percentage INT DEFAULT 100,
  target_users BIGINT[], -- User IDs for targeted rollout
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- ============================================
-- ÉVÉNEMENTS & WEBHOOKS
-- ============================================

CREATE TABLE webhook_endpoints (
  id BIGSERIAL PRIMARY KEY,
  seller_id BIGINT REFERENCES sellers(id) ON DELETE CASCADE,
  
  url TEXT NOT NULL,
  events JSONB, -- ['order.created', 'order.shipped', ...]
  
  secret_encrypted BYTEA, -- Signing secret
  
  is_active BOOLEAN DEFAULT TRUE,
  
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE webhook_logs (
  id BIGSERIAL PRIMARY KEY,
  webhook_endpoint_id BIGINT NOT NULL REFERENCES webhook_endpoints(id) ON DELETE CASCADE,
  event_type VARCHAR(100),
  
  payload JSONB,
  
  response_status_code INT,
  response_body TEXT,
  
  retry_count INT DEFAULT 0,
  
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- ============================================
-- BULKS & JOBS
-- ============================================

CREATE TABLE background_jobs (
  id BIGSERIAL PRIMARY KEY,
  job_id VARCHAR(100) UNIQUE NOT NULL,
  queue_name VARCHAR(100),
  job_type VARCHAR(100),
  
  data JSONB,
  
  status VARCHAR(20) DEFAULT 'pending', -- 'pending', 'processing', 'completed', 'failed'
  
  error_message TEXT,
  error_stack TEXT,
  
  attempt INT DEFAULT 0,
  max_attempts INT DEFAULT 3,
  
  started_at TIMESTAMP,
  completed_at TIMESTAMP,
  
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_background_jobs_status ON background_jobs(status);
CREATE INDEX idx_background_jobs_queue ON background_jobs(queue_name);

-- ============================================
-- EXTENSIONS & SÉCURITÉ
-- ============================================

-- PostGIS pour géolocalisation
CREATE EXTENSION IF NOT EXISTS postgis;
CREATE EXTENSION IF NOT EXISTS postgis_topology;

-- pg_trgm pour recherche textuelle
CREATE EXTENSION IF NOT EXISTS pg_trgm;

-- uuids
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- ============================================
-- TRIGGERS & FUNCTIONS
-- ============================================

CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = CURRENT_TIMESTAMP;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Apply to all updatable tables
CREATE TRIGGER update_users_timestamp BEFORE UPDATE ON users
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();

CREATE TRIGGER update_sellers_timestamp BEFORE UPDATE ON sellers
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();

CREATE TRIGGER update_products_timestamp BEFORE UPDATE ON products
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();

CREATE TRIGGER update_orders_timestamp BEFORE UPDATE ON orders
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();

-- ============================================
-- VUES UTILES
-- ============================================

CREATE VIEW seller_stats AS
SELECT 
  s.id,
  s.business_name,
  COUNT(DISTINCT o.id) as total_orders,
  SUM(o.total) as total_revenue,
  AVG(r.rating) as average_rating,
  COUNT(DISTINCT p.id) as total_products
FROM sellers s
LEFT JOIN orders o ON s.id = o.seller_id AND o.deleted_at IS NULL
LEFT JOIN reviews r ON s.id = r.seller_id AND r.status = 'approved'
LEFT JOIN products p ON s.id = p.seller_id AND p.deleted_at IS NULL
GROUP BY s.id, s.business_name;

--
-- Fin schéma PostgreSQL
--
```

### Modèles Prisma Correspondants

```prisma
// schema.prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model User {
  id                    BigInt @id @default(autoincrement())
  email                 String @unique
  phone                 String @unique
  passwordHash          String
  passwordSalt          String
  firstName             String
  lastName              String
  avatarUrl             String?
  
  userType              String // customer, seller, rider, admin
  status                String @default("active")
  
  countryCode           String @default("CM")
  timezone              String @default("Africa/Douala")
  preferredLanguage     String @default("fr")
  
  nationalIdEncrypted   Bytes?
  nationalIdHash        String? @unique
  
  kycStatus             String @default("pending")
  kycVerifiedAt         DateTime?
  kycData               Json?
  
  mfaEnabled            Boolean @default(false)
  mfaSecret             String?
  emailVerified         Boolean @default(false)
  emailVerifiedAt       DateTime?
  phoneVerified         Boolean @default(false)
  phoneVerifiedAt       DateTime?
  
  createdAt             DateTime @default(now())
  updatedAt             DateTime @updatedAt
  deletedAt             DateTime?
  lastLoginAt           DateTime?
  
  // Relations
  refreshTokens         RefreshToken[]
  auditLogs             AuditLog[]
  userRoles             UserRole[]
  addresses             Address[]
  orders                Order[] @relation("CustomerOrders")
  payments              Payment[]
  paymentMethods        PaymentMethod[]
  reviews               Review[] @relation("ReviewerReviews")
  supportTickets        SupportTicket[]
  wallet                Wallet?
  seller                Seller?
  
  @@index([email])
  @@index([phone])
  @@index([userType])
  @@index([kycStatus])
  @@index([createdAt])
}

model Seller {
  id                        BigInt @id @default(autoincrement())
  userId                    BigInt @unique
  
  businessName              String
  businessRegistration      String?
  businessRegistrationVerified Boolean @default(false)
  
  countryCode               String
  city                      String
  latitude                  Decimal? @db.Decimal(10, 8)
  longitude                 Decimal? @db.Decimal(11, 8)
  address                   String?
  
  primaryPaymentMethod      String? // mobile_money, bank_transfer
  mobileMoneyNumber         String?
  mobileMoneyOperator       String? // moov, orange, wagni
  bankAccountEncrypted      Bytes?
  
  verificationStatus        String @default("pending")
  verificationData          Json?
  verifiedAt                DateTime?
  
  totalSales                Decimal @default(0) @db.Decimal(15, 2)
  totalOrders               Int @default(0)
  rating                    Decimal @default(0) @db.Decimal(3, 2)
  ratingCount               Int @default(0)
  
  identityDocumentUrl       String?
  proofOfAddressUrl         String?
  
  isActive                  Boolean @default(false)
  suspendedAt               DateTime?
  suspensionReason          String?
  
  createdAt                 DateTime @default(now())
  updatedAt                 DateTime @updatedAt
  deletedAt                 DateTime?
  
  // Relations
  user                      User @relation(fields: [userId], references: [id], onDelete: Restrict)
  products                  Product[]
  orders                    Order[]
  reviews                   Review[]
  payouts                   Payout[]
  wallet                    Wallet?
  webhookEndpoints          WebhookEndpoint[]
  
  @@index([userId])
  @@index([verificationStatus])
  @@index([countryCode])
  @@index([createdAt])
}

model Product {
  id                 BigInt @id @default(autoincrement())
  sellerId           BigInt
  categoryId         BigInt
  
  name               String
  slug               String
  description        String?
  shortDescription   String?
  
  basePrice          Decimal @db.Decimal(15, 2)
  costPrice          Decimal? @db.Decimal(15, 2)
  discountPrice      Decimal? @db.Decimal(15, 2)
  discountPercentage Decimal? @db.Decimal(5, 2)
  
  stockQuantity      Int @default(0)
  lowStockThreshold  Int @default(10)
  
  thumbnailUrl       String?
  images             Json? // [{url, alt, position}]
  
  metaTitle          String?
  metaDescription    String?
  metaKeywords       String?
  
  status             String @default("draft")
  
  viewsCount         Int @default(0)
  rating             Decimal @default(0) @db.Decimal(3, 2)
  ratingCount        Int @default(0)
  
  createdAt          DateTime @default(now())
  updatedAt          DateTime @updatedAt
  deletedAt          DateTime?
  
  // Relations
  seller             Seller @relation(fields: [sellerId], references: [id], onDelete: Restrict)
  category           Category @relation(fields: [categoryId], references: [id], onDelete: Restrict)
  variants           ProductVariant[]
  orderItems         OrderItem[]
  reviews            Review[]
  
  @@unique([sellerId, slug], where: { deletedAt: null })
  @@index([categoryId])
  @@index([status])
  @@index([createdAt])
}

model Order {
  id              BigInt @id @default(autoincrement())
  orderNumber     String @unique
  customerId      BigInt
  sellerId        BigInt
  
  status          String @default("pending")
  statusHistory   Json? // [{status, timestamp, reason}]
  
  deliveryAddressId BigInt?
  
  itemsCount      Int
  subtotal        Decimal @db.Decimal(15, 2)
  deliveryFee     Decimal @db.Decimal(15, 2)
  discountAmount  Decimal @default(0) @db.Decimal(15, 2)
  discountReason  String?
  taxAmount       Decimal @default(0) @db.Decimal(15, 2)
  total           Decimal @db.Decimal(15, 2)
  
  currency        String @default("XAF")
  
  customerNotes   String?
  sellerNotes     String?
  
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
  deliveredAt     DateTime?
  cancelledAt     DateTime?
  
  // Relations
  customer        User @relation("CustomerOrders", fields: [customerId], references: [id], onDelete: Restrict)
  seller          Seller @relation(fields: [sellerId], references: [id], onDelete: Restrict)
  deliveryAddress Address? @relation(fields: [deliveryAddressId], references: [id])
  items           OrderItem[]
  payments        Payment[]
  delivery        Delivery?
  review          Review?
  supportTickets  SupportTicket[]
  
  @@index([customerId])
  @@index([sellerId])
  @@index([status])
  @@index([createdAt])
}

model Payment {
  id                BigInt @id @default(autoincrement())
  paymentId         String @unique
  orderId           BigInt
  userId            BigInt
  
  amount            Decimal @db.Decimal(15, 2)
  currency          String @default("XAF")
  
  paymentMethodId   BigInt?
  paymentMethodType String?
  provider          String? // moov, orange, stripe, etc.
  providerTransactionId String? @unique
  
  status            String @default("pending")
  statusHistory     Json? // [{status, timestamp, provider_status}]
  
  attemptCount      Int @default(0)
  lastAttemptAt     DateTime?
  lastErrorCode     String?
  lastErrorMessage  String?
  
  webhookReceivedAt DateTime?
  webhookSignatureVerified Boolean?
  webhookData       Json?
  
  ipAddress         String?
  
  createdAt         DateTime @default(now())
  updatedAt         DateTime @updatedAt
  completedAt       DateTime?
  
  // Relations
  order             Order @relation(fields: [orderId], references: [id], onDelete: Restrict)
  user              User @relation(fields: [userId], references: [id], onDelete: Restrict)
  paymentMethod     PaymentMethod? @relation(fields: [paymentMethodId], references: [id])
  reconciliation    PaymentReconciliation?
  walletTransaction WalletTransaction?
  
  @@index([orderId])
  @@index([userId])
  @@index([status])
  @@index([provider])
  @@index([createdAt])
}

model Delivery {
  id                    BigInt @id @default(autoincrement())
  orderId               BigInt @unique
  riderId               BigInt?
  
  pickupAddressId       BigInt
  deliveryAddressId     BigInt
  
  status                String @default("pending")
  statusHistory         Json? // [{status, timestamp}]
  
  currentLatitude       Decimal? @db.Decimal(10, 8)
  currentLongitude      Decimal? @db.Decimal(11, 8)
  lastLocationUpdate    DateTime?
  
  estimatedDeliveryTime DateTime?
  actualDeliveryTime    DateTime?
  
  deliverySignatureUrl  String?
  deliveryPhotoUrl      String?
  deliveryOtp           String?
  
  distanceKm            Decimal? @db.Decimal(10, 2)
  
  notes                 String?
  
  createdAt             DateTime @default(now())
  updatedAt             DateTime @updatedAt
  
  // Relations
  order                 Order @relation(fields: [orderId], references: [id], onDelete: Restrict)
  rider                 User? @relation(fields: [riderId], references: [id], onDelete: SetNull)
  pickupAddress         Address @relation("PickupAddresses", fields: [pickupAddressId], references: [id])
  deliveryAddress       Address @relation("DeliveryAddresses", fields: [deliveryAddressId], references: [id])
  tracking              DeliveryTracking[]
  
  @@index([orderId])
  @@index([riderId])
  @@index([status])
}

// ... (autres modèles à suivre)
```

---

## 5. Authentification & Sécurité

### Architecture Sécurité Production

#### 1. Flux d'Authentification (JWT + Refresh Token Rotation)

```typescript
// Backend: Authentification flow
UserLogin → Validate Credentials → Generate JWT (short-lived, 15min)
→ Generate Refresh Token (long-lived, 30 days) 
→ Store RefreshToken in DB with family tracking
→ Return both tokens + device fingerprint

// Client: Token handling
JWT in memory (vulnerable à XSS mais acceptable)
RefreshToken in secure HTTP-only cookie
DeviceFingerprintinclude dans requests
```

#### 2. Sécurité API

```yaml
Middleware Sécurité:
  1. CORS:
     - Whitelist domaines
     - Credentials: true
     - préflight caching

  2. Rate Limiting:
     - Globale: 100 req/min par IP
     - Authentification: 5 tentatives / 15 min
     - Paiement: 10 req/min par user

  3. JWT Validation:
     - Signature verification
     - Expiration check
     - Blacklist check (token révoqué)

  4. Headers Sécurité:
     - Content-Security-Policy
     - X-Frame-Options: DENY
     - X-Content-Type-Options: nosniff
     - Strict-Transport-Security
     - X-XSS-Protection

  5. Input Validation:
     - Zod schemas sur toutes entrées
     - Sanitization XSS
     - SQL injection prevention (ORM)

  6. RBAC:
     - Permissions granulaires
     - Basées rôles + resource-specific
     - Audit logging
```

#### 3. Protection Données Sensibles

```typescript
// Données à chiffrer (NaCl secretbox)
- National IDs
- Bank accounts
- Mobile Money credentials
- Payment details

// Hashing (Argon2id)
- Passwords
- National ID (pour checksums)
- Tokens (pour DB lookup)

// Encryption keys
- Master key in Vault (prod)
- Per-user keys optional (future)
- Key rotation strategy
```

#### 4. Sécurité Mobile Money

```yaml
Mobile Money Security:
  1. Idempotence:
     - Tous les paiements ont UUID unique
     - Retry-safe même + webhooks
     - DB check duplicates

  2. Anti-Fraude:
     - Velocity checks (5 paiements/jour par user)
     - Amount checks (max 500K XAF/transaction)
     - Device fingerprinting
     - Unusual patterns detection
     - IP geolocation verification

  3. Webhook Security:
     - HMAC-SHA256 signatures
     - Timestamp validation (anti-replay)
     - Retry logic avec exponential backoff
     - Webhook timeout: 30s

  4. PCI Compliance:
     - Zoning network
     - Tokenization Cards si présent
     - Regular security audits
```

#### 5. Stratégie Monitoring Sécurité

```yaml
Real-time Alerts:
  - Brute force attempts
  - Unusual payment amounts
  - Multiple failed payments
  - Impossible geography (user travels)
  - Rate limit breaches
  - Admin action anomalies

Dashboards:
  - Failed login attempts
  - Fraud scores distribution
  - Webhook failures
  - Token revocation rate
  - Permission violations
```

---

## 6. Architecture Backend

### Comparaison Framework: Express vs Fastify vs NestJS

| Critère | Fastify | Express | NestJS |
|---------|---------|---------|--------|
| **Performance** | ~4x Express | Baseline | ~3x Express |
| **Throughput** | 220K req/s | 50K req/s | 150K req/s |
| **Startup** | <100ms | ~50ms | ~500ms |
| **Plugins** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Validation** | Native JSON Schema | Manual | Decorators |
| **TypeScript** | Excellent | OK | Excellent |
| **Learning** | Facile | Très facile | Complexe |
| **Scalability** | Très bon | OK | Excellent |
| **Poids** | 105 KB | 440 KB | 1.4 MB |
| **Request/sec at 100K users** | ~80K | ~20K | ~60K |

**Verdict: Fastify** = Performance + simplicité + TypeScript + scalabilité

### Structure Backend Recommandée

```
backend/
├── src/
│   ├── config/
│   │   ├── database.ts
│   │   ├── auth.ts
│   │   ├── payment.ts
│   │   └── env.ts
│   │
│   ├── plugins/
│   │   ├── database.ts (Prisma)
│   │   ├── cache.ts (Redis)
│   │   ├── auth.ts (JWT)
│   │   ├── cors.ts
│   │   └── rateLimit.ts
│   │
│   ├── middleware/
│   │   ├── errorHandler.ts
│   │   ├── validation.ts
│   │   ├── authentication.ts
│   │   ├── authorization.ts
│   │   ├── logging.ts
│   │   └── security.ts
│   │
│   ├── modules/
│   │   ├── auth/
│   │   │   ├── auth.controller.ts
│   │   │   ├── auth.service.ts
│   │   │   ├── auth.routes.ts
│   │   │   ├── auth.schemas.ts
│   │   │   └── auth.repository.ts
│   │   │
│   │   ├── users/
│   │   │   ├── users.controller.ts
│   │   │   ├── users.service.ts
│   │   │   ├── users.routes.ts
│   │   │   ├── users.schemas.ts
│   │   │   ├── users.repository.ts
│   │   │   └── users.types.ts
│   │   │
│   │   ├── products/
│   │   │   ├── products.controller.ts
│   │   │   ├── products.service.ts
│   │   │   ├── products.routes.ts
│   │   │   ├── products.schemas.ts
│   │   │   ├── products.repository.ts
│   │   │   ├── search.service.ts
│   │   │   └── cache.service.ts
│   │   │
│   │   ├── orders/
│   │   │   ├── orders.controller.ts
│   │   │   ├── orders.service.ts
│   │   │   ├── orders.routes.ts
│   │   │   ├── orders.schemas.ts
│   │   │   ├── orders.repository.ts
│   │   │   └── stateMachine.ts
│   │   │
│   │   ├── payments/
│   │   │   ├── payments.controller.ts
│   │   │   ├── payments.service.ts
│   │   │   ├── payments.routes.ts
│   │   │   ├── payments.schemas.ts
│   │   │   ├── payments.repository.ts
│   │   │   ├── providers/
│   │   │   │   ├── moov.provider.ts
│   │   │   │   ├── orange.provider.ts
│   │   │   │   └── stripe.provider.ts
│   │   │   ├── webhook.handler.ts
│   │   │   └── reconciliation.service.ts
│   │   │
│   │   ├── delivery/
│   │   │   ├── delivery.controller.ts
│   │   │   ├── delivery.service.ts
│   │   │   ├── delivery.routes.ts
│   │   │   ├── delivery.schemas.ts
│   │   │   ├── delivery.repository.ts
│   │   │   ├── tracking.service.ts
│   │   │   └── zones.service.ts
│   │   │
│   │   ├── sellers/
│   │   │   ├── sellers.controller.ts
│   │   │   ├── sellers.service.ts
│   │   │   ├── sellers.routes.ts
│   │   │   ├── sellers.schemas.ts
│   │   │   └── sellers.repository.ts
│   │   │
│   │   └── admin/
│   │       ├── admin.controller.ts
│   │       ├── admin.routes.ts
│   │       ├── dashboard.service.ts
│   │       └── moderation.service.ts
│   │
│   ├── workers/
│   │   ├── index.ts
│   │   ├── queues/
│   │   │   ├── email.queue.ts
│   │   │   ├── payment-reconciliation.queue.ts
│   │   │   ├── delivery-sync.queue.ts
│   │   │   ├── notification.queue.ts
│   │   │   └── image-processing.queue.ts
│   │   ├── jobs/
│   │   │   ├── sendVerificationEmail.ts
│   │   │   ├── reconcilePayments.ts
│   │   │   ├── syncDeliveryStatus.ts
│   │   │   └── generateReports.ts
│   │   └── processors/
│   │       ├── email.processor.ts
│   │       ├── payment.processor.ts
│   │       └── delivery.processor.ts
│   │
│   ├── webhooks/
│   │   ├── index.ts
│   │   ├── moov.webhook.ts
│   │   ├── orange.webhook.ts
│   │   └── stripe.webhook.ts
│   │
│   ├── services/
│   │   ├── cache.service.ts
│   │   ├── email.service.ts
│   │   ├── sms.service.ts
│   │   ├── notification.service.ts
│   │   ├── encryption.service.ts
│   │   ├── storage.service.ts (S3)
│   │   ├── logging.service.ts
│   │   ├── metrics.service.ts
│   │   └── audit.service.ts
│   │
│   ├── utils/
│   │   ├── errors.ts
│   │   ├── validators.ts
│   │   ├── formatters.ts
│   │   ├── crypto.ts
│   │   ├── pagination.ts
│   │   ├── cache-keys.ts
│   │   └── constants.ts
│   │
│   ├── types/
│   │   ├── index.ts
│   │   ├── user.types.ts
│   │   ├── order.types.ts
│   │   ├── payment.types.ts
│   │   └── common.types.ts
│   │
│   └── app.ts
│
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
│
├── Dockerfile
├── docker-compose.yml
├── .env.example
├── package.json
├── tsconfig.json
├── vitest.config.ts
└── README.md
```

### Exemple: Module Utilisateurs (Fastify)

```typescript
// src/modules/users/users.controller.ts
import { FastifyReply, FastifyRequest } from "fastify";
import { UsersService } from "./users.service";
import { CreateUserSchema, UpdateUserSchema } from "./users.schemas";

export class UsersController {
  constructor(private usersService: UsersService) {}

  async getProfile(req: FastifyRequest, reply: FastifyReply) {
    const user = await this.usersService.getById(req.user.id);
    if (!user) return reply.code(404).send({ error: "User not found" });
    reply.send({ data: user });
  }

  async updateProfile(req: FastifyRequest<{ Body: typeof UpdateUserSchema }>, reply: FastifyReply) {
    const updated = await this.usersService.update(req.user.id, req.body);
    reply.send({ data: updated });
  }

  async listUsers(req: FastifyRequest, reply: FastifyReply) {
    // Admin only
    const { page = 1, limit = 20 } = req.query as any;
    const users = await this.usersService.list({ page, limit });
    reply.send({ data: users });
  }
}

// src/modules/users/users.service.ts
import { PrismaClient } from "@prisma/client";
import { CacheService } from "@/services/cache.service";
import { EncryptionService } from "@/services/encryption.service";
import { AuditService } from "@/services/audit.service";

export class UsersService {
  constructor(
    private prisma: PrismaClient,
    private cache: CacheService,
    private encryption: EncryptionService,
    private audit: AuditService
  ) {}

  async getById(id: bigint) {
    // Try cache first
    const cacheKey = `user:${id}`;
    let user = await this.cache.get(cacheKey);
    
    if (!user) {
      user = await this.prisma.user.findUnique({ where: { id } });
      if (user) {
        await this.cache.set(cacheKey, user, { ex: 3600 }); // 1 hour TTL
      }
    }
    
    return user;
  }

  async update(id: bigint, data: any) {
    const before = await this.getById(id);
    
    const updated = await this.prisma.user.update({
      where: { id },
      data
    });
    
    // Invalidate cache
    await this.cache.del(`user:${id}`);
    
    // Audit log
    await this.audit.log({
      userId: id,
      action: "user.updated",
      changes: { before, after: updated }
    });
    
    return updated;
  }

  async list({ page, limit }: { page: number; limit: number }) {
    const skip = (page - 1) * limit;
    return this.prisma.user.findMany({
      skip,
      take: limit,
      select: {
        id: true,
        email: true,
        firstName: true,
        lastName: true,
        userType: true,
        status: true,
        createdAt: true
      },
      orderBy: { createdAt: "desc" }
    });
  }
}

// src/modules/users/users.routes.ts
import { FastifyInstance } from "fastify";
import { UsersController } from "./users.controller";

export async function usersRoutes(fastify: FastifyInstance) {
  const controller = new UsersController(fastify.usersService);

  fastify.get(
    "/profile",
    { onRequest: [fastify.authenticate] },
    (req, reply) => controller.getProfile(req, reply)
  );

  fastify.patch(
    "/profile",
    {
      onRequest: [fastify.authenticate],
      schema: { body: UpdateUserSchema }
    },
    (req, reply) => controller.updateProfile(req, reply)
  );

  fastify.get(
    "/users",
    {
      onRequest: [fastify.authenticate, fastify.authorize({ resource: "users", action: "read" })]
    },
    (req, reply) => controller.listUsers(req, reply)
  );
}
```

---

## 7. Mobile Money & Paiements

### Architecture Paiement Robuste

#### State Machine Transactional

```
┌─────────┐
│ PENDING │  User initiates payment
└────┬────┘
     │
     ▼
┌─────────────┐
│ PROCESSING  │  Payment en cours chez provider
└────┬────┘
     │
     ├──→ AUTHORIZED (Cartes)
     │         │
     │         ▼
     │    ┌─────────┐
     │    │CAPTURED │  Fonds capturés
     │    └────┬────┘
     │         │
     ▼         ▼
┌──────────┐
│ COMPLETED│  Paiement réussi
└────┬─────┘
     │
     └──→ REFUNDED (si demande)
           │
           ▼
        ┌────────┐
        │ REFUND │
        │PENDING │
        └────┬───┘
             │
             ▼
        ┌──────────┐
        │ REFUNDED │
        └──────────┘

     ▼
┌─────────┐
│ FAILED  │  Payment failed
└─────────┘
 
     ▼
┌───────────┐
│ CANCELLED │  User cancelled
└───────────┘
```

#### Workflow Mobile Money (Moov/Orange)

```
User initiates payment
        │
        ▼
CREATE ORDER + PAYMENT RECORD (status: pending)
        │
        ▼
Call Mobile Money API (USSD/RTC)
        │
        ├─ Success → Payment Record updated (processing)
        │     │
        │     ▼
        │  USSD Sent to User
        │     │
        │     ▼ (User enters PIN)
        │  Webhook received from provider
        │     │
        │     ├─ success → status: authorized
        │     │     │
        │     │     ▼
        │     │  Capture funds
        │     │     │
        │     │     ▼
        │     │  status: completed
        │     │     │
        │     │     ▼
        │     │  Update order status
        │     │     │
        │     │     └─→ ORDER_CONFIRMED
        │     │
        │     └─ declined → status: failed
        │           │
        │           ▼
        │       Retry logic or cancel
        │
        └─ Error → status: failed
               │
               ▼
          Retry logic (exponential backoff)
          Attempt count++
          
          Max retries reached → Manual review
```

#### Implémentation Robuste

```typescript
// src/modules/payments/payments.service.ts

export class PaymentsService {
  constructor(
    private prisma: PrismaClient,
    private paymentProviders: PaymentProviders,
    private queue: Queue,
    private encryption: EncryptionService,
    private logger: Logger
  ) {}

  async initiatePayment(orderId: bigint, paymentMethodId: bigint, userId: bigint) {
    // 1. Validation
    const order = await this.prisma.order.findUnique({ where: { id: orderId } });
    if (!order) throw new NotFoundError("Order not found");
    if (order.customerId !== userId) throw new UnauthorizedError();

    // 2. Vérifier idempotence (UUID unique = same payment initiated twice)
    const existingPayment = await this.prisma.payment.findFirst({
      where: { orderId, status: { not: "failed" } }
    });
    if (existingPayment) return existingPayment; // Idempotent!

    // 3. Create Payment record
    const paymentId = crypto.randomUUID();
    const payment = await this.prisma.payment.create({
      data: {
        paymentId,
        orderId,
        userId,
        amount: order.total,
        currency: order.currency,
        status: "pending",
        paymentMethodId,
        attemptCount: 0
      }
    });

    // 4. Get payment method (decrypted)
    const method = await this.decryptPaymentMethod(paymentMethodId);

    // 5. Initiate with provider
    try {
      const providerResponse = await this.paymentProviders[method.provider].initiate({
        amount: order.total,
        currency: order.currency,
        phone: method.phoneNumber,
        externalId: paymentId, // Pour idempotence
        description: `Order #${order.orderNumber}`
      });

      // 6. Update payment with provider info
      await this.prisma.payment.update({
        where: { id: payment.id },
        data: {
          status: "processing",
          providerTransactionId: providerResponse.transactionId,
          lastAttemptAt: new Date()
        }
      });

      // 7. Schedule webhook timeout check (if not received in 2 min)
      await this.queue.add(
        "payment-timeout-check",
        { paymentId: payment.id },
        { delay: 120000, attempts: 1 }
      );

      return payment;
    } catch (error) {
      // Failed to initiate
      await this.handlePaymentFailure(payment.id, error.message);
      throw error;
    }
  }

  async handleWebhook(provider: string, payload: any, signature: string) {
    // 1. Verify signature
    if (!this.verifyWebhookSignature(provider, payload, signature)) {
      throw new SecurityError("Invalid webhook signature");
    }

    const externalId = payload.externalId; // Notre paymentId
    const providerStatus = payload.status;

    // 2. Find payment (by externalId du provider)
    const payment = await this.prisma.payment.findUnique({
      where: { paymentId: externalId }
    });
    if (!payment) {
      this.logger.warn("Webhook for unknown payment", { externalId });
      return; // Ignore unknown payments (could be from old system)
    }

    // 3. Idempotence: check if already processed
    const existingWebhook = await this.prisma.webhookLog.findUnique({
      where: { provider_externalId: `${provider}:${payload.id}` }
    });
    if (existingWebhook) {
      this.logger.info("Duplicate webhook, ignoring", { externalId });
      return;
    }

    // 4. Update payment status + store webhook
    const newStatus = this.mapProviderStatus(provider, providerStatus);

    await this.prisma.$transaction(async (tx) => {
      // Atomic update
      await tx.payment.update({
        where: { id: payment.id },
        data: {
          status: newStatus,
          statusHistory: [...(payment.statusHistory || []), {
            status: newStatus,
            timestamp: new Date(),
            providerStatus
          }],
          webhookReceivedAt: new Date(),
          webhookSignatureVerified: true,
          webhookData: payload
        }
      });

      // Log webhook for idempotence
      await tx.webhookLog.create({
        data: {
          provider_externalId: `${provider}:${payload.id}`,
          paymentId: payment.id,
          data: payload
        }
      });

      // If successful, complete order
      if (newStatus === "completed") {
        await tx.order.update({
          where: { id: payment.orderId },
          data: { status: "confirmed" }
        });

        // Queue notifications
        await this.queue.add("send-notification", {
          userId: payment.userId,
          type: "payment_success",
          orderId: payment.orderId
        });
      }
    });
  }

  async handlePaymentFailure(paymentId: bigint, errorMessage: string) {
    const payment = await this.prisma.payment.findUnique({ where: { id: paymentId } });

    if (payment.attemptCount < 3) {
      // Retry with exponential backoff
      const delay = Math.pow(2, payment.attemptCount) * 5000; // 5s, 10s, 20s
      await this.queue.add(
        "payment-retry",
        { paymentId },
        { delay, attempts: 1 }
      );
    } else {
      // Max retries reached - alert support
      await this.queue.add("payment-manual-review", { paymentId });
    }

    await this.prisma.payment.update({
      where: { id: paymentId },
      data: {
        status: "failed",
        lastErrorCode: "PROVIDER_ERROR",
        lastErrorMessage: errorMessage,
        attemptCount: { increment: 1 }
      }
    });
  }

  // Reconciliation (nightly)
  async reconcilePayments() {
    const yesterday = new Date();
    yesterday.setDate(yesterday.getDate() - 1);

    const payments = await this.prisma.payment.findMany({
      where: {
        createdAt: {
          gte: yesterday.toISOString()
        },
        status: { in: ["completed", "failed"] }
      }
    });

    for (const payment of payments) {
      const provider = this.paymentProviders[payment.provider];
      const providerStatus = await provider.getStatus(payment.providerTransactionId);

      if (providerStatus !== payment.status) {
        // Discrepancy!
        this.logger.error("Payment reconciliation mismatch", {
          paymentId: payment.id,
          localStatus: payment.status,
          providerStatus
        });

        await this.prisma.paymentReconciliation.create({
          data: {
            paymentId: payment.id,
            status: "discrepancy",
            notes: `Local: ${payment.status}, Provider: ${providerStatus}`
          }
        });
      }
    }
  }
}
```

#### Providers (Moov, Orange, Stripe)

```typescript
// src/modules/payments/providers/moov.provider.ts

export class MoovProvider implements PaymentProvider {
  private apiKey: string;
  private apiSecret: string;
  private baseUrl = "https://api.moov.cm";

  constructor(credentials: { apiKey: string; apiSecret: string }) {
    this.apiKey = credentials.apiKey;
    this.apiSecret = credentials.apiSecret;
  }

  async initiate(params: PaymentInitParams): Promise<PaymentProviderResponse> {
    const response = await fetch(`${this.baseUrl}/payment/initiate`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "Authorization": `Bearer ${this.apiKey}`
      },
      body: JSON.stringify({
        amount: params.amount,
        currency: params.currency,
        phone: params.phone,
        externalId: params.externalId, // Pour idempotence
        description: params.description
      })
    });

    if (!response.ok) {
      throw new Error(`Moov API error: ${response.statusText}`);
    }

    const data = await response.json();
    return {
      transactionId: data.transactionId,
      status: "processing"
    };
  }

  async getStatus(transactionId: string): Promise<string> {
    const response = await fetch(`${this.baseUrl}/payment/status/${transactionId}`, {
      headers: { "Authorization": `Bearer ${this.apiKey}` }
    });

    const data = await response.json();
    return data.status; // 'success', 'failed', 'pending'
  }

  verifyWebhookSignature(payload: any, signature: string): boolean {
    const hash = crypto
      .createHmac("sha256", this.apiSecret)
      .update(JSON.stringify(payload))
      .digest("hex");

    return crypto.timingSafeEqual(Buffer.from(hash), Buffer.from(signature));
  }
}
```

---

## 8. Livraison & Logistique

### Système de Zones & Frais Dynamiques

```typescript
// src/modules/delivery/delivery.service.ts

export class DeliveryService {
  async calculateDeliveryFee(
    fromLatitude: number,
    fromLongitude: number,
    toLatitude: number,
    toLongitude: number,
    cityCode: string
  ): Promise<{ fee: Decimal; estimatedHours: number }> {
    // 1. Get delivery zone
    const zone = await this.prisma.deliveryZone.findUnique({
      where: { cityCode }
    });

    if (!zone) throw new NotFoundError("Delivery zone not available");

    // 2. Calculate distance
    const distance = this.haversineDistance(
      fromLatitude, fromLongitude,
      toLatitude, toLongitude
    );

    // 3. Calculate fee: base + per-km
    const fee = zone.baseDeliveryFee + (distance * zone.perKmFee);

    // 4. Return with estimated hours
    return {
      fee,
      estimatedHours: zone.estimatedDeliveryHoursMin
    };
  }

  async assignDelivery(orderId: bigint) {
    const order = await this.prisma.order.findUnique({ where: { id: orderId } });
    const delivery = await this.prisma.delivery.findUnique({ where: { orderId } });

    // 1. Find nearest available riders
    const riders = await this.findNearbyRiders(
      delivery.pickupAddress.latitude,
      delivery.pickupAddress.longitude,
      radius: 5_000 // 5km
    );

    if (riders.length === 0) {
      // No riders available - mark as pending
      await this.queue.add("retry-rider-assignment", { orderId }, { delay: 300000 }); // 5 min
      return;
    }

    // 2. Select best rider (closest + lowest active orders)
    const bestRider = this.selectBestRider(riders);

    // 3. Assign
    await this.prisma.$transaction([
      this.prisma.delivery.update({
        where: { orderId },
        data: {
          riderId: bestRider.id,
          status: "assigned"
        }
      }),
      // Notify rider via push notification + SMS
      this.queue.add("notify-rider-assignment", {
        riderId: bestRider.id,
        orderId
      })
    ]);
  }

  async updateDeliveryLocation(deliveryId: bigint, latitude: number, longitude: number) {
    const delivery = await this.prisma.delivery.findUnique({ where: { id: deliveryId } });

    // 1. Update location
    await this.prisma.delivery.update({
      where: { id: deliveryId },
      data: {
        currentLatitude: latitude,
        currentLongitude: longitude,
        lastLocationUpdate: new Date()
      }
    });

    // 2. Calculate progress
    const distanceRemaining = this.haversineDistance(
      latitude, longitude,
      delivery.deliveryAddress.latitude,
      delivery.deliveryAddress.longitude
    );

    const progressPercent = ((delivery.distanceKm - distanceRemaining) / delivery.distanceKm) * 100;

    // 3. Emit real-time update via WebSocket
    this.io.to(`order:${delivery.orderId}`).emit("delivery_update", {
      latitude,
      longitude,
      progressPercent,
      estimatedArrivalMinutes: Math.round(distanceRemaining / 1000 * 2) // ~2 min per km
    });

    // 4. Log tracking
    await this.prisma.deliveryTracking.create({
      data: {
        deliveryId,
        eventType: "location_update",
        latitude,
        longitude,
        description: `Rider location updated: ${progressPercent.toFixed(0)}% complete`
      }
    });
  }

  async completeDelivery(deliveryId: bigint, photoUrl: string, signatureUrl: string, otp: string) {
    const delivery = await this.prisma.delivery.findUnique({ where: { id: deliveryId } });

    // 1. Verify OTP
    if (!this.verifyDeliveryOTP(delivery.deliveryOtp, otp)) {
      throw new ValidationError("Invalid delivery OTP");
    }

    // 2. Record proof
    const updatedDelivery = await this.prisma.delivery.update({
      where: { id: deliveryId },
      data: {
        status: "delivered",
        deliveryPhotoUrl: photoUrl,
        deliverySignatureUrl: signatureUrl,
        actualDeliveryTime: new Date()
      }
    });

    // 3. Update order status
    await this.prisma.order.update({
      where: { id: delivery.orderId },
      data: { status: "delivered", deliveredAt: new Date() }
    });

    // 4. Credit seller wallet
    const order = await this.prisma.order.findUnique({ where: { id: delivery.orderId } });
    await this.creditSellerWallet(order.sellerId, order.total);

    // 5. Notifications
    await this.queue.add("send-notification", {
      userId: order.customerId,
      type: "delivery_completed",
      orderId: delivery.orderId
    });
  }

  private haversineDistance(lat1: number, lon1: number, lat2: number, lon2: number): number {
    const R = 6371; // Earth radius in km
    const dLat = (lat2 - lat1) * Math.PI / 180;
    const dLon = (lon2 - lon1) * Math.PI / 180;
    const a = Math.sin(dLat/2) * Math.sin(dLat/2) +
              Math.cos(lat1 * Math.PI / 180) * Math.cos(lat2 * Math.PI / 180) *
              Math.sin(dLon/2) * Math.sin(dLon/2);
    const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
    return R * c;
  }
}
```

---

## 9. Frontend

### Architecture Frontend Recommandée

**Stack:** Next.js 14 + React 18 + TanStack Query + TypeScript

#### Structure Frontend Web

```
web/
├── src/
│   ├── app/
│   │   ├── (public)/
│   │   │   ├── layout.tsx
│   │   │   ├── page.tsx (home)
│   │   │   ├── about/page.tsx
│   │   │   └── help/page.tsx
│   │   │
│   │   ├── (auth)/
│   │   │   ├── layout.tsx
│   │   │   ├── login/page.tsx
│   │   │   ├── register/page.tsx
│   │   │   └── forgot-password/page.tsx
│   │   │
│   │   ├── (dashboard)/
│   │   │   ├── layout.tsx
│   │   │   ├── page.tsx (dashboard)
│   │   │   ├── profile/page.tsx
│   │   │   ├── orders/page.tsx
│   │   │   ├── orders/[id]/page.tsx
│   │   │   ├── wishlist/page.tsx
│   │   │   └── settings/page.tsx
│   │   │
│   │   ├── (seller)/
│   │   │   ├── layout.tsx
│   │   │   ├── page.tsx (seller dashboard)
│   │   │   ├── products/page.tsx
│   │   │   ├── products/new/page.tsx
│   │   │   ├── products/[id]/edit/page.tsx
│   │   │   ├── orders/page.tsx
│   │   │   ├── analytics/page.tsx
│   │   │   └── payouts/page.tsx
│   │   │
│   │   ├── (admin)/
│   │   │   ├── layout.tsx
│   │   │   ├── page.tsx (admin dashboard)
│   │   │   ├── users/page.tsx
│   │   │   ├── sellers/page.tsx
│   │   │   ├── payments/page.tsx
│   │   │   ├── disputes/page.tsx
│   │   │   └── analytics/page.tsx
│   │   │
│   │   ├── products/page.tsx (catalog)
│   │   ├── products/[id]/page.tsx (PDP)
│   │   ├── seller/[id]/page.tsx (seller profile)
│   │   ├── checkout/page.tsx
│   │   ├── cart/page.tsx
│   │   └── api/ (API Routes)
│   │
│   ├── components/
│   │   ├── common/
│   │   │   ├── Header.tsx
│   │   │   ├── Footer.tsx
│   │   │   ├── Navigation.tsx
│   │   │   ├── Loading.tsx
│   │   │   ├── Error.tsx
│   │   │   └── Modal.tsx
│   │   │
│   │   ├── products/
│   │   │   ├── ProductCard.tsx
│   │   │   ├── ProductGallery.tsx
│   │   │   ├── ProductFilter.tsx
│   │   │   ├── ProductSearch.tsx
│   │   │   └── Reviews.tsx
│   │   │
│   │   ├── orders/
│   │   │   ├── OrderCard.tsx
│   │   │   ├── OrderTimeline.tsx
│   │   │   ├── OrderTracking.tsx
│   │   │   └── DeliveryMap.tsx
│   │   │
│   │   ├── payment/
│   │   │   ├── PaymentForm.tsx
│   │   │   ├── MobileMoneyFlow.tsx
│   │   │   └── PaymentStatus.tsx
│   │   │
│   │   └── seller/
│   │       ├── SellerProfile.tsx
│   │       ├── ProductForm.tsx
│   │       └── Analytics.tsx
│   │
│   ├── hooks/
│   │   ├── useAuth.ts
│   │   ├── useProducts.ts
│   │   ├── useOrders.ts
│   │   ├── usePayment.ts
│   │   ├── useUser.ts
│   │   ├── useInfiniteScroll.ts
│   │   └── useLocalStorage.ts
│   │
│   ├── services/
│   │   ├── api.client.ts (Fetch wrapper)
│   │   ├── auth.service.ts
│   │   ├── products.service.ts
│   │   ├── orders.service.ts
│   │   ├── payment.service.ts
│   │   └── user.service.ts
│   │
│   ├── store/
│   │   ├── auth.store.ts (Zustand)
│   │   ├── cart.store.ts
│   │   ├── ui.store.ts
│   │   └── theme.store.ts
│   │
│   ├── types/
│   │   ├── index.ts
│   │   ├── user.ts
│   │   ├── product.ts
│   │   ├── order.ts
│   │   └── payment.ts
│   │
│   ├── utils/
│   │   ├── currency.ts
│   │   ├── date.ts
│   │   ├── validation.ts
│   │   ├── file.ts
│   │   └── constants.ts
│   │
│   ├── styles/
│   │   ├── globals.css
│   │   └── tailwind.config.ts
│   │
│   ├── config/
│   │   ├── env.ts
│   │   ├── api.ts
│   │   └── constants.ts
│   │
│   └── middleware.ts (Auth middleware)
│
├── public/
│   ├── images/
│   ├── icons/
│   └── fonts/
│
├── Dockerfile
├── next.config.js
├── tailwind.config.ts
├── tsconfig.json
├── package.json
└── README.md
```

#### Exemple: Page Catégorique (Next.js 14)

```typescript
// app/(public)/products/page.tsx
'use client';

import { useState, useEffect } from 'react';
import { useSearchParams } from 'next/navigation';
import { useInfiniteQuery } from '@tanstack/react-query';
import ProductCard from '@/components/products/ProductCard';
import ProductFilter from '@/components/products/ProductFilter';
import { productsService } from '@/services/products.service';

export default function ProductsPage() {
  const searchParams = useSearchParams();
  const [filters, setFilters] = useState({
    category: searchParams.get('category') || '',
    priceMin: parseInt(searchParams.get('priceMin') || '0'),
    priceMax: parseInt(searchParams.get('priceMax') || '1000000'),
    sortBy: searchParams.get('sortBy') || 'newest'
  });

  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
    isLoading
  } = useInfiniteQuery({
    queryKey: ['products', filters],
    queryFn: ({ pageParam = 1 }) =>
      productsService.list({
        page: pageParam,
        limit: 20,
        ...filters
      }),
    getNextPageParam: (lastPage) =>
      lastPage.hasMore ? lastPage.nextPage : undefined,
    initialPageParam: 1
  });

  const products = data?.pages.flatMap(page => page.products) || [];

  return (
    <div className="grid grid-cols-1 md:grid-cols-4 gap-6 p-4">
      <aside className="md:col-span-1">
        <ProductFilter
          filters={filters}
          onFilterChange={setFilters}
        />
      </aside>

      <main className="md:col-span-3">
        {isLoading ? (
          <div>Loading...</div>
        ) : (
          <>
            <div className="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
              {products.map(product => (
                <ProductCard key={product.id} product={product} />
              ))}
            </div>

            {hasNextPage && (
              <button
                onClick={() => fetchNextPage()}
                disabled={isFetchingNextPage}
                className="mt-8 w-full py-3 bg-blue-600 text-white rounded"
              >
                {isFetchingNextPage ? 'Loading...' : 'Load More'}
              </button>
            )}
          </>
        )}
      </main>
    </div>
  );
}
```

### Performance & Optimisation Mobile Afrique

```yaml
Optimisations:
  1. Images:
     - Next.js Image component
     - WebP format avec fallback
     - Lazy loading + LQIP (Low Quality Image Placeholder)
     - Responsive srcset
     - Max 50KB pour thumbnails
     - Compression aggressive

  2. Code:
     - Route lazy loading
     - Component code splitting
     - Tree shaking
     - Minification + gzip
     - Remove unused CSS

  3. Network:
     - Request deduplication
     - API response caching (TanStack Query)
     - Service Worker precaching
     - Manifest for offline support

  4. DevTools:
     - Lighthouse CI
     - Bundle analysis (webpack-bundle-analyzer)
     - Performance monitoring (Sentry)

  5. Mobile Afrique Spécifique:
     - Offline-first architecture
     - SQLite local DB (WatermelonDB)
     - Delta sync (envoyer changements, pas full copy)
     - Compression aggressive (gzip + brotli)
     - Asset storage mobile
```

---

## 10. DevOps & Infrastructure

### Architecture VPS Recommended (Hetzner/OVH)

```yaml
Server Spec:
  vCPU: 4-8 cores
  RAM: 16-32 GB
  Storage: 200+ GB SSD
  Bandwidth: 20 Tbps

Container Setup:
  Docker + Docker Compose
  Nginx reverse proxy
  PostgreSQL container
  Redis container
  Fastify API containers (3x replicas)
  Worker containers (Bull processing)
  Webhook handler container

Networking:
  Internal Docker network
  Nginx on port 80/443 (exposed)
  All services internal only
```

### Docker Compose (Production-like)

```yaml
# docker-compose.yml
version: '3.9'

services:
  # Reverse Proxy
  nginx:
    image: nginx:latest
    ports:
      - '80:80'
      - '443:443'
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./certs:/etc/nginx/certs:ro
    depends_on:
      - api1
      - api2
      - api3
    networks:
      - koloshop
    restart: unless-stopped

  # PostgreSQL Database
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: koloshop
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - '5432:5432'
    networks:
      - koloshop
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -U postgres']
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  # Redis Cache
  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis_data:/data
    ports:
      - '6379:6379'
    networks:
      - koloshop
    healthcheck:
      test: ['CMD', 'redis-cli', 'ping']
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  # API Instances (Load Balanced)
  api1:
    build:
      context: ./backend
      dockerfile: Dockerfile
    environment:
      NODE_ENV: production
      DATABASE_URL: postgresql://postgres:${DB_PASSWORD}@postgres:5432/koloshop
      REDIS_URL: redis://:${REDIS_PASSWORD}@redis:6379
      JWT_SECRET: ${JWT_SECRET}
      PORT: 3000
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - koloshop
    restart: unless-stopped

  api2:
    build:
      context: ./backend
      dockerfile: Dockerfile
    environment:
      NODE_ENV: production
      DATABASE_URL: postgresql://postgres:${DB_PASSWORD}@postgres:5432/koloshop
      REDIS_URL: redis://:${REDIS_PASSWORD}@redis:6379
      JWT_SECRET: ${JWT_SECRET}
      PORT: 3000
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - koloshop
    restart: unless-stopped

  api3:
    build:
      context: ./backend
      dockerfile: Dockerfile
    environment:
      NODE_ENV: production
      DATABASE_URL: postgresql://postgres:${DB_PASSWORD}@postgres:5432/koloshop
      REDIS_URL: redis://:${REDIS_PASSWORD}@redis:6379
      JWT_SECRET: ${JWT_SECRET}
      PORT: 3000
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - koloshop
    restart: unless-stopped

  # Background Worker
  worker:
    build:
      context: ./backend
      dockerfile: Dockerfile.worker
    environment:
      NODE_ENV: production
      DATABASE_URL: postgresql://postgres:${DB_PASSWORD}@postgres:5432/koloshop
      REDIS_URL: redis://:${REDIS_PASSWORD}@redis:6379
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - koloshop
    restart: unless-stopped

  # Webhook Handler
  webhook:
    build:
      context: ./backend
      dockerfile: Dockerfile.webhook
    environment:
      NODE_ENV: production
      DATABASE_URL: postgresql://postgres:${DB_PASSWORD}@postgres:5432/koloshop
      REDIS_URL: redis://:${REDIS_PASSWORD}@redis:6379
      PORT: 3001
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - koloshop
    restart: unless-stopped

networks:
  koloshop:
    driver: bridge

volumes:
  postgres_data:
  redis_data:
```

### CI/CD (GitHub Actions)

```yaml
# .github/workflows/deploy.yml
name: Deploy Production

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install

      - name: Run tests
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/koloshop_test
        run: pnpm run test

      - name: Run linting
        run: pnpm run lint

  build:
    needs: test
    runs-on: ubuntu-latest
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'

    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to registry
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.REGISTRY_USERNAME }}
          password: ${{ secrets.REGISTRY_PASSWORD }}

      - name: Build and push backend
        uses: docker/build-push-action@v5
        with:
          context: ./backend
          push: true
          tags: myregistry/koloshop-api:${{ github.sha }},myregistry/koloshop-api:latest

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'

    steps:
      - name: Deploy to production
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.PROD_HOST }}
          username: ${{ secrets.PROD_USER }}
          key: ${{ secrets.PROD_KEY }}
          script: |
            cd /home/app/koloshop
            docker-compose pull
            docker-compose up -d
            docker-compose exec -T api npm run migrate
```

---

## 11. Observabilité & Monitoring

### Stack: Prometheus + Grafana + Loki

```yaml
# monitoring/prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'koloshop-api'
    static_configs:
      - targets: ['api1:3000', 'api2:3000', 'api3:3000']

  - job_name: 'postgres'
    static_configs:
      - targets: ['postgres:5432']
    relabel_configs:
      - source_labels: [__address__]
        target_label: instance

  - job_name: 'redis'
    static_configs:
      - targets: ['redis:6379']

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']
```

### Métriques Importantes

```typescript
// Backend: Prometheus metrics
import prom from 'prom-client';

export const httpRequestDuration = new prom.Histogram({
  name: 'http_request_duration_ms',
  help: 'Duration of HTTP requests in ms',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.1, 5, 15, 50, 100, 500]
});

export const paymentProcessingTime = new prom.Histogram({
  name: 'payment_processing_time_ms',
  help: 'Time to process payment',
  labelNames: ['provider', 'status'],
  buckets: [100, 500, 1000, 5000, 10000]
});

export const orderCounter = new prom.Counter({
  name: 'orders_total',
  help: 'Total orders created',
  labelNames: ['seller_id', 'status']
});

export const activeConnections = new prom.Gauge({
  name: 'active_connections',
  help: 'Number of active connections',
  labelNames: ['service']
});
```

### Dashboards Grafana

**Dashboard 1: Application Health**
- Request rate (RPS)
- Error rate (%)
- Response time (p50/p95/p99)
- Active connections

**Dashboard 2: Payment Processing**
- Payments by provider
- Success rate
- Average processing time
- Webhook latency

**Dashboard 3: Database**
- Query performance
- Connection pool
- Replica lag
- Disk usage

**Dashboard 4: Business Metrics**
- Orders per hour
- Revenue per day
- Average order value
- Seller signup rate

---

## 12. Scalabilité

### Stratégie Scaling par Étape

#### Phase 1: MVP (1K users)
- 1× VPS 2-core
- PostgreSQL single instance
- Redis single instance
- Manual backups

```
├──── API (FastAPI) → DB → Redis
└──── Worker → Queue → Notifications
```

#### Phase 2: Growth (10K users)
- 1× VPS 4-core
- PostgreSQL with read replicas
- Redis with RDB persistence
- Automated backups hourly
- Basic monitoring

```
┌──────────────────────────────┐
│   Nginx Load Balancer        │
└─────────┬──────────┬─────────┘
          │          │
    ┌─────▼──┐  ┌───▼──────┐
    │  API 1 │  │   API 2  │
    └────────┘  └──────────┘
          │          │
    ┌─────▼──────────▼─────┐
    │  PostgreSQL Primary  │
    │  + 1x Read Replica   │
    └──────────────────────┘
```

#### Phase 3: Scale (100K users)
- 2× VPS (load balancer + backend)
- PostgreSQL cluster (streaming replication)
- Redis cluster
- CDN (CloudFlare)
- S3-compatible storage
- Kubernetes ready

```
┌──────────────────────────────────┐
│     CloudFlare CDN / WAF         │
└─────────────────┬────────────────┘
                  │
          ┌───────▼────────┐
          │ Nginx + Lua    │
          │ (Hetzner LB)   │
          └───┬──┬───┬─────┘
              │  │   │
         ┌────▼┐ │ ┌─▼────┐
         │API1 │ │ │API2  │
         └────┬┘ │ └─┬────┘
              │  │   │
    ┌─────────▼──▼───▼──────┐
    │ PostgreSQL Cluster    │
    │ (3x nodes, quorum)    │
    └──┬────────┬───────┬──┘
       │        │       │
    ┌──▼─┐  ┌──▼─┐  ┌──▼─┐
    │Rep │  │Rep │  │Rep │
    └────┘  └────┘  └────┘
```

### Caching Strategy

```typescript
// Multi-layer caching
const cacheHierarchy = {
  // L1: Browser cache (HTTP caching headers)
  http: {
    public: "max-age=3600",
    products: "max-age=86400"
  },

  // L2: Redis (app-level cache)
  redis: {
    userProfile: 3600,        // 1 hour
    productDetails: 86400,    // 1 day
    categoryList: 604800,     // 1 week
    sellerStats: 3600,        // 1 hour
    orderStatus: 1800         // 30 min
  },

  // L3: Database query optimization
  database: {
    indexes: "on_most_queries",
    resultCache: "Redis"
  }
};
```

### Database Scaling

```sql
-- Connection pooling (PgBouncer)
-- Max 100 connections per API instance
-- RDS Connection proxy

-- Partitioning (for >100M records)
-- PARTITION orders BY RANGE (created_at)
-- PARTITION payments BY RANGE (created_at)
-- PARTITION order_items BY RANGE (created_at)

-- Read replicas for analytics
SELECT * FROM orders WHERE created_at > '2024-01-01';
-- Executed on replica to reduce primary load
```

---

## 13. Business Logic

### Modèles de Revenus

```yaml
Revenue Streams:
  1. Commission Vendeurs:
     - 8-12% par transaction
     - Variable selon catégorie (premium: 15%)
     - KoloShop garde ~10-12%
     - Seller reçoit ~88-90%

  2. Premium Seller:
     - Subscription: 5,000 XAF/mois
     - Featured listings
     - Analytics avancé
     - Priority support

  3. Advertising:
     - Featured products: 1,000 XAF/jour
     - Promoted listings: 500 XAF/listing
     - Featured categories: 10,000 XAF/mois

  4. Delivery Network (Future):
     - 15-20% des frais de livraison
     - Premium delivery service
     - White label integration

Total Target Margin: 12-15%
```

### Commission & Payout Flow

```typescript
// Payout calculation (nightly reconciliation)
Order Total: 100,000 XAF
├─ Seller receives: 90,000 XAF (90% after commission)
├─ KoloShop commission: 10,000 XAF (10%)
└─ Platform costs:
   ├─ Payment processing (2%): 2,000 XAF
   ├─ Delivery (20%): 20,000 XAF already factored in order
   └─ Infrastructure: ~1,000 XAF (hosting, monitoring)
   └─ Net KoloShop profit: ~7,000 XAF

Monthly at 1,000 orders/day:
Total GMV: 3B XAF
KoloShop revenue: 300M XAF (10% commission)
Gross profit (after delivery): ~45M XAF after costs
```

### Gestion Litiges & Remboursements

```yaml
Dispute Resolution:
  1. Automatic Holdback:
     - Order refund amount held for 7 days
     - If no dispute → payout to seller
     - If dispute → investigation starts

  2. Dispute Process:
     - Customer initiates within 30 days
     - Automatic refund if no evidence
     - Manual review if contested
     - Escalation to admin if needed

  3. Refund Flow:
     Order.status = REFUNDED
     ├─ Payment.status = REFUNDED
     ├─ Seller.wallet -= amount
     ├─ Customer.wallet += amount
     └─ AuditLog created

  4. Prevention:
     - Seller ratings & trust score
     - Product review scores
     - Dispute rate per seller
     - Auto-suspend if >5% dispute rate
```

---

## 14. Production Readiness

### Checklist Pre-Production

#### ✅ Sécurité
- [ ] Authentification 2FA en place
- [ ] JWT avec refresh token rotation
- [ ] Rate limiting actif
- [ ] CORS configuré
- [ ] HTTPS/TLS forcé
- [ ] Headers sécurité (CSP, X-Frame-Options, etc.)
- [ ] Input validation + sanitization
- [ ] SQL injection prevention (ORM)
- [ ] XSS protection active
- [ ] CSRF tokens
- [ ] Secrets vault en place
- [ ] No hardcoded secrets
- [ ] Database encryption at rest
- [ ] Payment data tokenization
- [ ] Audit logging actif
- [ ] Fraud detection système

#### ✅ Performance
- [ ] API response time < 200ms (p95)
- [ ] Database queries optimized (< 100ms)
- [ ] Redis cache configured
- [ ] CDN for static assets
- [ ] Image optimization
- [ ] Code minification
- [ ] Gzip compression
- [ ] Database connection pooling
- [ ] Load testing done (>10K concurrent)
- [ ] Caching strategy documented
- [ ] Database indexes analyzed
- [ ] Slow query log reviewed

#### ✅ Reliability
- [ ] Database backups automatisés (hourly)
- [ ] Backup restoration tested
- [ ] Error handling + retries
- [ ] Circuit breakers en place
- [ ] Health checks actifs
- [ ] Graceful degradation
- [ ] Rate limiting actif
- [ ] Queue processing robust
- [ ] Webhook retry logic
- [ ] Idempotence guaranteed
- [ ] State machine tested
- [ ] Failover tested

#### ✅ Observabilité
- [ ] Logging structured (JSON)
- [ ] Log retention: 30 days
- [ ] Metrics collection (Prometheus)
- [ ] Alerting rules
- [ ] Dashboards Grafana
- [ ] Error tracking (Sentry)
- [ ] Distributed tracing
- [ ] Business metrics monitored
- [ ] SLA targets defined
- [ ] On-call rotation
- [ ] Post-mortem process
- [ ] Incident response plan

#### ✅ Base de Données
- [ ] Indexes sur all queries
- [ ] Foreign keys avec ON DELETE
- [ ] Constraints validées
- [ ] Partitioning stratégie
- [ ] Read replicas configured
- [ ] Point-in-time recovery possible
- [ ] Schema versioning (Flyway/Liquibase)
- [ ] Data validation scripts
- [ ] Migration testing

#### ✅ Paiements
- [ ] PCI compliance review
- [ ] Payment provider integration tested
- [ ] Webhook signatures verified
- [ ] Reconciliation automatisée
- [ ] Fraud prevention active
- [ ] Refund process tested
- [ ] Payment retry logic
- [ ] Idempotent payment IDs
- [ ] Encryption for sensitive data
- [ ] Rate limiting on payment endpoint

#### ✅ DevOps
- [ ] Docker images optimized
- [ ] Docker Compose production-ready
- [ ] Health checks configured
- [ ] Resource limits set
- [ ] Volume management
- [ ] Log aggregation
- [ ] Network isolation
- [ ] Secrets management
- [ ] CI/CD pipeline
- [ ] Automated testing
- [ ] Automated deployment
- [ ] Rollback strategy
- [ ] Infrastructure as Code
- [ ] Disaster recovery tested

#### ✅ Compliance & Legal
- [ ] Privacy policy published
- [ ] Terms of service agreed
- [ ] GDPR compliance (if EU users)
- [ ] Data retention policy
- [ ] Right to be forgotten implemented
- [ ] KYC/AML process documented
- [ ] Payment provider contracts signed
- [ ] SLA agreements

#### ✅ Support & Operations
- [ ] Support system in place
- [ ] FAQ documented
- [ ] Knowledge base created
- [ ] Escalation path clear
- [ ] On-call team trained
- [ ] Runbooks created
- [ ] Incident templates
- [ ] Communication plan

---

## 15. Risques Critiques

| Risque | Impact | Probabilité | Mitigation |
|--------|--------|-------------|-----------|
| **Fraude paiement** | Perte $$$, légal | Haute | Velocity checking, 3D Secure, Fraud detection ML |
| **DB outage** | Total downtime | Moyenne | Replicas, automated backups, failover |
| **DDoS attaque** | Service indisponible | Moyenne | CloudFlare, rate limiting, DDoS protection |
| **Payment provider down** | Commandes impossibles | Basse | Fallback provider, offline order queue |
| **Data breach** | Réputation, légal, GDPR | Basse | Encryption, Network security, Audit |
| **Rider fraud** | Perte orders/argent | Moyenne | Verification, GPS tracking, OTP delivery |
| **Seller fraud** | Chargebacks, disputes | Moyenne | Seller verification, Dispute resolution |
| **Poor UX** | Users churn | Moyenne | A/B testing, Analytics, User feedback |
| **Scaling issues** | Performance -> users leave | Moyenne | Load testing, Architecture reviewed |
| **Vendor lock-in** | Switching costs élevés | Basse | Containerization, Cloud-agnostic design |

---

## Recommendations Finales

### Stack Recommandé Final

```yaml
Backend:
  Framework: Fastify + TypeScript
  Database: PostgreSQL 16
  Cache: Redis 7
  Queue: Bull (Redis-backed)
  ORM: Prisma
  Validation: Zod
  Auth: JWT + refresh tokens

Frontend Web:
  Framework: Next.js 14
  Styling: Tailwind CSS
  State: TanStack Query + Zustand
  UI: shadcn/ui

Mobile:
  Framework: React Native
  State: Redux Toolkit
  Offline: WatermelonDB
  Push: Expo

Infrastructure:
  VPS: Hetzner / OVH
  Container: Docker + Docker Compose
  CI/CD: GitHub Actions
  CDN: CloudFlare
  Monitoring: Prometheus + Grafana + Loki
```

### Roadmap MVP → Production

**Phase 1 (Semaines 1-4): MVP Core**
- User auth + KYC
- Product catalog search
- Order creation
- Basic delivery assignment
- Mobile Money integration (Moov)

**Phase 2 (Semaines 5-8): Seller Platform**
- Seller registration + KYC
- Product upload
- Order management
- Seller analytics
- Payout system

**Phase 3 (Semaines 9-12): Operations**
- Admin dashboard
- Fraud detection
- Payment reconciliation
- Support system
- Monitoring + Alerts

**Phase 4 (Semaines 13-16): Scale + Polish**
- Performance optimization
- Security audit
- Load testing
- UX improvements
- Production deployment

---

## Conclusion

Cette architecture production-ready:
✅ Est réaliste & faisable par petite équipe (5-7 devs)
✅ Scalable jusqu'à 100K+ utilisateurs
✅ Sécurisée (auth, payments, audit)
✅ Observable (logging, metrics, alerting)
✅ Maintenable (clear structure, documentation)
✅ Monétisable (commission, premium features)

**Timeline estimée:** 4 mois de MVP → production
**Équipe requise:** 2 backend + 2 frontend + 1 DevOps + 1 PM
**Coûts infrastructure (mois 1):** ~$500-1000
**Coûts à 10K users:** ~$2000-3000/mois
**Break-even:** ~6-9 mois à 10K users actifs

---

**Document prepared:** May 27, 2026  
**Senior Architect Review:** REQUIRED before implementation  
**Next Step:** Detailed API specification + Data models
