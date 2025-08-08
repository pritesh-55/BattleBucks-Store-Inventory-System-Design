# BattleBucks Store & Inventory System Design

This comprehensive design provides a scalable, performant, and maintainable backend system for BattleBucks that can handle 1M+ users and 100K+ daily purchases while maintaining data consistency and providing excellent user experience.

## 1. Schema Design Diagram
- [🔗  Schema Design Diagram](https://mermaid.live/edit#pako:eNq9WEtz4jgQ_isunZPUYsgEuO7WXuYyl71sUeVq7Aa00cMjyzwmyX_flrGxZcsTmKSGC7hb_f661eaFpTpDtmRo_uKwNSBXKqJPWaApopfzg_us-ZYrG_Es-va1pe7BpDsw1XEFEqN_AkyUwEWQk0NRHLTJkh0Uu5aNqpRRYcGWRUu0XCLRZB6lBsFiloANccs887hv5682rOQAQqC9IrqaWgkR6-8OK8OUSxDRFmWRrEGASnHItdqCSKozCEZh9tMjRY7KfnLAWyqKF-lPiugKOKQ6fHjFu7E6njvcokxS4m-14b_omMWjpfQVqeG55Vr5KnIwlMV-ua50r7DaYOKcvAH7t3hXh37q--d4rlR9epXrKmn2lGOPnqHgeyRlPq9BVm54ihWyfDsSjsn3EpTl9tSJUWuBoCJOOLSQPsNadHT-V2gVSbRAUIPf0agus5Baarzc6A0X-LF2HcvvZRCdjSThFoA9RW2S0ghfo8A9ioF9POZI4KaBkOSaCEUwyRluoBT23RR_Sjbz0gVS4OfMvHZkvYOEipjDSbqOpNB2Ohsm1xpQBZWaesUZ7E6a35GSa7u9prZifm5qdjtAQggcdl2T0lJxm4Qa1s95_8TwbuNqT8mmCfOhSv9iNG3qIf1ecjNaGWoR4hYeN1BsLzwkjXlOKm8r2GCQjMTqZ29sThRC29647UTVeBgE3KYUZF5UvUArz1Xr1XWg86AcvEFGbopg0162tqN1O504O0tqe6W66PSb0zkElhzJbeDeGXLa9CmymBi0pDKMmlTLnBa3UVR9YBqcV8KkM4s-NiqrzHYnW_iKBqnL7srX0OuFMlnjhjpwnA8bqtHAIYMbNNX90y3a-GJy5XJkiJUILrm9YTniGeGdb3jXzQvCVFZdkD5MDLURuVMkqZ-b1ssDV5k-uDXFXDVdKIruS83r6_3966v_KrCMVoxaqMbBivXP65fQRtJINYSgXHv3uuMSnunXhRaU6A1yJ6YPBMhq8AUlQvBtnOsSG-Huu0Gtorv6NqKeQe_0SDLcmUE2wpv_iNnmxI9WfkSyT3bS5-X_smX7HnRN9WrTcaAhZRFcPAhIhmtEUpy2u4bs22-BMG491coC79d6IBm4S5x0TSYv1iO2_SDeVUONxvc8KwlePYUBANRKe_d0g6WG3MVvJ4Hjwg0p2hgtV4zdsa3hGVtaU-Idk2gkuEdWjaQVszukFZ45yQzMszP3RjI5qH-1lo2Y0eV2x5YbEAU9na-G-v-PyxEaT2j-dEOILePHuNLBli_syJaT2ePDYjafTyeLx8lsHk-_3LETW97Hi4dFPH-aT_6IF_PpU7yI3-7Yj8ru5GEym82m8y_T-WQeTxbx09v_JFKW8g)

### Core Entities

#### Users & Authentication
```sql
-- Users table
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    status ENUM('active', 'suspended', 'banned') DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    INDEX idx_users_email (email),
    INDEX idx_users_username (username),
    INDEX idx_users_status (status)
);

-- User wallets for virtual currency
CREATE TABLE user_wallets (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    gems_balance DECIMAL(15,2) DEFAULT 0.00,
    total_gems_earned DECIMAL(15,2) DEFAULT 0.00,
    total_gems_spent DECIMAL(15,2) DEFAULT 0.00,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    UNIQUE KEY uk_user_wallet (user_id)
);
```

#### Games & Items
```sql
-- Games supported by the platform
CREATE TABLE games (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    code VARCHAR(20) UNIQUE NOT NULL,
    status ENUM('active', 'maintenance', 'deprecated') DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    INDEX idx_games_status (status)
);

-- Item categories for organization
CREATE TABLE item_categories (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    parent_id INT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (parent_id) REFERENCES item_categories(id)
);

-- Store items
CREATE TABLE store_items (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    category_id INT,
    game_id INT NULL, -- NULL for platform-wide items
    item_type ENUM('consumable', 'non_consumable') NOT NULL,
    delivery_type ENUM('in_game', 'email', 'shopify', 'functional') NOT NULL,
    price_gems DECIMAL(10,2) NOT NULL,
    max_quantity INT NULL, -- NULL for unlimited
    is_stackable BOOLEAN DEFAULT FALSE,
    metadata JSON, -- Flexible storage for item-specific data
    status ENUM('active', 'inactive', 'coming_soon') DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    FOREIGN KEY (category_id) REFERENCES item_categories(id),
    FOREIGN KEY (game_id) REFERENCES games(id),
    INDEX idx_store_items_game (game_id),
    INDEX idx_store_items_status (status),
    INDEX idx_store_items_category (category_id),
    INDEX idx_store_items_type (item_type),
    INDEX idx_store_items_price (price_gems)
);
```

#### Character Profiles
```sql
-- User character profiles
CREATE TABLE character_profiles (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    game_id INT NULL, -- NULL for platform-wide profile
    profile_name VARCHAR(100) NOT NULL,
    avatar_url VARCHAR(500),
    level INT DEFAULT 1,
    experience_points BIGINT DEFAULT 0,
    is_default BOOLEAN DEFAULT FALSE,
    metadata JSON, -- Game-specific character data
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (game_id) REFERENCES games(id),
    INDEX idx_character_user_game (user_id, game_id),
    UNIQUE KEY uk_user_game_default (user_id, game_id, is_default)
);
```

#### Purchase & Inventory System
```sql
-- Purchase transactions
CREATE TABLE purchases (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    total_gems DECIMAL(10,2) NOT NULL,
    status ENUM('pending', 'completed', 'failed', 'refunded') DEFAULT 'pending',
    payment_method ENUM('gems', 'real_money') DEFAULT 'gems',
    transaction_id VARCHAR(100) UNIQUE, -- For idempotency
    metadata JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    FOREIGN KEY (user_id) REFERENCES users(id),
    INDEX idx_purchases_user (user_id),
    INDEX idx_purchases_status (status),
    INDEX idx_purchases_created (created_at),
    INDEX idx_purchases_transaction (transaction_id)
);

-- Purchase line items
CREATE TABLE purchase_items (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    purchase_id BIGINT NOT NULL,
    store_item_id BIGINT NOT NULL,
    quantity INT NOT NULL DEFAULT 1,
    unit_price_gems DECIMAL(10,2) NOT NULL,
    total_price_gems DECIMAL(10,2) NOT NULL,
    
    FOREIGN KEY (purchase_id) REFERENCES purchases(id) ON DELETE CASCADE,
    FOREIGN KEY (store_item_id) REFERENCES store_items(id),
    INDEX idx_purchase_items_purchase (purchase_id)
);

-- User inventory
CREATE TABLE user_inventory (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    store_item_id BIGINT NOT NULL,
    quantity INT NOT NULL DEFAULT 1,
    acquired_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    expires_at TIMESTAMP NULL,
    metadata JSON, -- Item-specific data like serial numbers
    
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (store_item_id) REFERENCES store_items(id),
    UNIQUE KEY uk_user_item_non_stackable (user_id, store_item_id),
    INDEX idx_inventory_user (user_id),
    INDEX idx_inventory_item (store_item_id)
);

-- Equipped items for character profiles
CREATE TABLE equipped_items (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    character_profile_id BIGINT NOT NULL,
    user_inventory_id BIGINT NOT NULL,
    slot_type VARCHAR(50), -- e.g., 'weapon_skin', 'avatar_hat'
    equipped_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (character_profile_id) REFERENCES character_profiles(id) ON DELETE CASCADE,
    FOREIGN KEY (user_inventory_id) REFERENCES user_inventory(id) ON DELETE CASCADE,
    UNIQUE KEY uk_character_slot (character_profile_id, slot_type),
    INDEX idx_equipped_character (character_profile_id)
);
```

#### Fulfillment & Delivery
```sql
-- Fulfillment tracking
CREATE TABLE fulfillment_orders (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    purchase_id BIGINT NOT NULL,
    purchase_item_id BIGINT NOT NULL,
    delivery_type ENUM('in_game', 'email', 'shopify', 'functional') NOT NULL,
    status ENUM('pending', 'processing', 'completed', 'failed', 'cancelled') DEFAULT 'pending',
    external_order_id VARCHAR(255), -- Shopify order ID, email job ID, etc.
    delivery_data JSON, -- Delivery-specific information
    attempts INT DEFAULT 0,
    max_attempts INT DEFAULT 3,
    next_retry_at TIMESTAMP NULL,
    completed_at TIMESTAMP NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    FOREIGN KEY (purchase_id) REFERENCES purchases(id),
    FOREIGN KEY (purchase_item_id) REFERENCES purchase_items(id),
    INDEX idx_fulfillment_status (status),
    INDEX idx_fulfillment_delivery_type (delivery_type),
    INDEX idx_fulfillment_retry (next_retry_at)
);
```

#### Audit & Rate Limiting
```sql
-- Wallet transactions audit
CREATE TABLE wallet_transactions (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    transaction_type ENUM('purchase', 'refund', 'reward', 'adjustment') NOT NULL,
    amount DECIMAL(15,2) NOT NULL, -- Positive for credit, negative for debit
    balance_before DECIMAL(15,2) NOT NULL,
    balance_after DECIMAL(15,2) NOT NULL,
    reference_id BIGINT, -- Purchase ID or other reference
    description TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (user_id) REFERENCES users(id),
    INDEX idx_wallet_transactions_user (user_id),
    INDEX idx_wallet_transactions_type (transaction_type),
    INDEX idx_wallet_transactions_created (created_at)
);

-- Rate limiting
CREATE TABLE rate_limits (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    identifier VARCHAR(255) NOT NULL, -- IP or user_id, etc.
    endpoint VARCHAR(255) NOT NULL,
    requests_count INT NOT NULL DEFAULT 1,
    window_start TIMESTAMP NOT NULL,
    expires_at TIMESTAMP NOT NULL,
    
    UNIQUE KEY uk_rate_limit (identifier, endpoint, window_start),
    INDEX idx_rate_limits_expires (expires_at)
);
```

## 2. Key Entities & Design Decisions

### Key Entities Overview

#### Users & Authentication
The **users** table serves as the central identity hub, with each user having a unique **user_wallets** record tracking their gem balance and spending history. The wallet design separates balance tracking from transaction history for better performance and audit capabilities.

#### Store Items & Multi-Game Support
**store_items** is designed with flexibility in mind. The `game_id` field can be NULL for platform-wide items (like profile avatars) or contain a specific game ID for game-exclusive items (like weapon skins). The **games** table maintains metadata for all supported games, allowing easy expansion to new games without schema changes.

The `metadata` JSON field in store_items provides extensibility for game-specific properties without requiring schema migrations. For example, a weapon skin might store `{"weapon_type": "rifle", "rarity": "legendary"}` while a platform avatar stores `{"animation_type": "idle", "theme": "cyberpunk"}`.

#### Character Profiles & Multi-Game Identity
**character_profiles** supports both game-specific and platform-wide profiles through the nullable `game_id` field. Users can have one default profile per game plus a platform-wide profile. This design accommodates games with different character systems while maintaining a unified platform identity.

#### Purchases & Inventory System
**purchases** acts as the transaction header with **purchase_items** containing line-item details. This separation allows for complex purchases (multiple items in one transaction) while maintaining referential integrity. The **user_inventory** table stores actual owned items with quantity support for stackable items.

### Handling Different Item Types

#### Delivery Types Implementation
The system supports four delivery types through the `delivery_type` enum and **fulfillment_orders** table:

1. **In-Game Delivery**: Items are added to user_inventory immediately upon purchase completion. Game clients poll or receive webhooks about new items.

2. **Email Delivery**: Fulfillment creates background jobs to generate voucher codes and send templated emails. The `delivery_data` JSON field stores voucher details and expiration information.

3. **Shopify Integration**: Physical items trigger Shopify order creation through APIs. The `external_order_id` tracks Shopify order status, with webhooks updating fulfillment status.

4. **Functional Delivery**: Items like "gem packs" or "name change tokens" are applied immediately during purchase processing, with effects recorded in relevant tables.

#### Consumables vs Non-Consumables
The `item_type` field distinguishes between consumable and non-consumable items:

- **Non-Consumables** (skins, avatars): Permanently owned items that can be equipped/unequipped. The `is_stackable` flag is typically FALSE, and inventory quantities remain constant.

- **Consumables** (utility cards, boosters): Items that are "used up" when consumed. They support stacking (multiple quantities in one inventory slot) and are decremented upon use. Usage is tracked through separate consumption logs.

#### Equipping Logic & Item Compatibility
The **equipped_items** table manages what items are currently equipped to character profiles. The `slot_type` field defines equipment categories (e.g., "weapon_skin", "character_hat", "platform_avatar").

**Multi-Game Compatibility Rules**:
- Game-specific items (non-NULL `game_id`) can only be equipped to profiles of the same game
- Platform-wide items (NULL `game_id`) can be equipped to any profile
- Each slot_type allows only one equipped item per character profile (enforced by unique constraint)
- Business logic validates compatibility before equipping (e.g., rifle skins only on rifle slots)

## 3. System Architecture
- [🔗  System Architecture Diagram](https://mermaid.live/edit#pako:eNp9V-tu6kgMfpUoUldnJWghFNqi1ZECpBBtUjhJW9SlVTUlUxg1JGwup4et-hz7QPti68wlTG5t_8DYn-357LHNh7oOPawO1U2E9lvldvQYKPB3cqKMfYKDRLHQAUfsME5fmNajyoX6fu-TNUpIGMSPKtPK_pbGaLXEL5n86Xhqz0emZazs8IX4uCSb6raxmqId5n5jLsOBxz7kcekLU5miBL-jg_KbYoXIU0bIR8FahGmNVoVTbglwz9PlSoL_8RKdfXfgs2KRHUlIsKEneppsIQJ-r6eS-3EYYcUm6yiMcfSTrHFcJSdTcYVUouXONZxn9368ugOs0KA-20rRKz9cROErcBXzr-MtitA6wVEsMefezh2DWnUTybGAoAT54YZ_MxO8i3PbZA1XliyZN_fGDVh7oNbM4CfEE0aHkkUau2zI-Dsl-71gr638SHFEsBzi4s4Zz3SXRblIo_UWxeVAbyMUxHC5rJT40T3yiVeiAxiNi1EvdcsybqnpJfJ9nJQMT_FOsVGANngH92n2pqceSSS713fWtWlZ1PB16kMe_MxAyfo88oCOY2CCk1-QpQD5ihkkGOpCuoSDE-DUCjdk3VjjNhiDgDMq05oCK4mlCpuZ09nzj9WMbLZZgsOIJAemRr0L7uNWgQH5jRoTwNvYI-muzkJeFa28OCW0NV8C2grf66DGDhEfPOvAywEKvVDEs_nCvH4AsLsN9-S1EPT2EMOz8BnXzY1hhNZvmyhMA09ZhtEbqFaIq1FRa8p0OXf-NJxjpXJdGs1__yrdjkKCOMmaS1xTMBwt10zJQL8Wb9i6maMpWWVcrxYnuONIQWAJq1WwVQbHaL2FGm7q-SDFXCax5hgT030eW3fuLXh3oHJi6OFpDA-AVzzrS8VOxLqIKCZ-yN8vb9ziveQNujnwCdh-yTJVH3lJLAU_GT3bOg38m42ykOGE-l1C8WJlvsfs7ca_PxVAjrGwzLHeXX1zMIwaB9M5qHQplt2Xt8F6oFYCasXH1QB2Z7ozAZ8uzAGPO8uIjJVOu9c7qdPWhLYmaffO24NBrXpPqPck9cFFu9vp5PpV-vNu5zaNxKqGWingvHJhRH_x8I9PJevM7J3IPdmF-KYR8c5cw5VnkP5gw3TLVoAFOtBXWVgDMPixwwAflLxJFlaXGxMmo3kzhfUlIJAj0euz2pqEm7Mb_A7Z9L9o6nxnSsK6pQU2JqXd_g7LC_vOdqXCUbYiFQ5yywV7mQNpyRE7EUWyHeiLhQqwxRwyBAWL7aUiyDeQiqSwUVSk8lpQER7HeincbKJGbR4lrGO7XRrwrYnpyGZrTVUUasKs6EjrANMogGoIkgB1LOX3EXxn3NMWyyTCmNI-BXChyTKF3F6jRinCBq0jO40qIi0N4tqryKtLhUy2ptTzRDeQehFdLxrYFQtEKSQ-BcsRsQCKVcjmJ39-WRCFvMtSGgeVyjObp0UEUghL1qilS4youJx9MJJ3_i9k2heyXqVgmJBNvgahmHBNxS7NsoYklzzIdVaVlvNZElcnPZ-cx2efI1idVq7QKNeappm0u8fF5LJ0yhlmcrkcpPqo6SpqC35xE08dJlGKW-oORzDI4Kv6kWk-qvBjcIcf1SF89FD0lo3LT8DsUfBXGO4EDFbZzVYdviI_hm_pHn4t4QlBEPRRBQYRjsaw8ybq8LzTGVAj6vBD_aUOu93L057W0c77nZ52ddG96LbUgzoc9E-v-tqlNuidX2n986vLz5b6D3XbOb0caBe9wRX8X_ZBofv5P-Nx27w)

### Microservices Architecture

#### Core Services
1. **User Service**
   - User authentication & authorization
   - Profile management
   - Character profiles

2. **Store Service**
   - Item catalog management
   - Pricing & promotions
   - Store front APIs

3. **Inventory Service**
   - User inventory management
   - Item equipping logic
   - Inventory queries

4. **Purchase Service**
   - Purchase processing
   - Transaction management
   - Payment integration

5. **Wallet Service**
   - Gem balance management
   - Transaction processing
   - Audit logging

6. **Fulfillment Service**
   - Order fulfillment
   - External integrations (Shopify, Email)
   - Retry mechanisms

## 4. Performance Optimization Strategies

### 4.1 Caching Strategy

#### a. Redis Cache Layers
- **Store Catalog**: Cache popular items, categories (TTL: 1 hour)
- **User Inventory**: Cache frequently accessed inventories (TTL: 30 minutes)
- **Character Profiles**: Cache equipped items and profile data (TTL: 15 minutes)
- **Wallet Balances**: Cache with write-through pattern (TTL: 5 minutes)

#### b. Application-Level Caching
**Service Layer Caching Strategy:**
- Store Service implements decorator-based caching with 1-hour TTL for catalog queries
- Inventory Service uses 30-minute TTL for user inventory with write-through pattern
- Cache keys follow hierarchical pattern: `service:entity:parameters`
- Automatic cache invalidation on data mutations

### 4.2 Background Jobs & Queues

#### Queue System (Redis/RabbitMQ)
- **High Priority Queue**: Purchase processing, gem transactions
- **Medium Priority Queue**: Inventory updates, character profile updates  
- **Low Priority Queue**: Email delivery, analytics, audit logs
- **Shopify Queue**: Physical order processing and webhooks

#### Job Types
**Background Job Categories:**
- **Purchase Completion Jobs**: High priority queue handles inventory updates and transaction finalization
- **Fulfillment Retry Jobs**: Medium priority queue manages failed delivery attempts with exponential backoff
- **Email Delivery Jobs**: Low priority queue processes digital reward notifications
- **Analytics Jobs**: Lowest priority queue handles reporting and metrics collection

### 4.3 Transactional Safety & Idempotency

#### Purchase Flow with Distributed Transactions
**Transactional Safety Design:**
- **Idempotency Protection**: Check for existing transaction_id before processing
- **Database Locking**: Acquire exclusive lock on user wallet using SELECT FOR UPDATE
- **Validation Layer**: Verify sufficient gem balance before proceeding
- **Atomic Operations**: Create purchase record and deduct gems within single transaction
- **Commit Strategy**: Commit transaction only after all critical operations succeed
- **Async Processing**: Queue fulfillment jobs after successful transaction commit
- **Status Management**: Update purchase status to 'completed' after job queueing
- **Error Handling**: Automatic rollback on any transaction failure with client notification

### 4.4 High-Traffic API Optimizations

**GET /store/items**
- **Pagination**: Serve via limit/offset or cursor-based pagination.
- **Denormalize Popularity**: Precompute popular items to reduce live computation.
- **Lightweight Response**: Include only essential fields by default
- **Caching**: Cache full/filtered store lists per region, user segment, or platform.
- **CDN**: Cache static item images and metadata

**GET /user/inventory**
- **Denormalization**: Pre-compute equipped status
- **Lazy Loading**: Load item details on-demand
- **Lightweight Response**: Include only essential fields by default
- **Batch Loading**: Group item metadata queries
- **Compression**: gzip responses for large inventories

### 4.6 Indexing Strategy
```sql
-- Composite indexes for common queries
CREATE INDEX idx_store_items_game_status ON store_items(game_id, status);
CREATE INDEX idx_user_inventory_user_item ON user_inventory(user_id, store_item_id);
CREATE INDEX idx_purchases_user_created ON purchases(user_id, created_at DESC);

-- Covering indexes for read-heavy queries
CREATE INDEX idx_store_items_catalog ON store_items(status, game_id, category_id) 
    INCLUDE (name, price_gems, item_type);
```

### 4.7 Rate Limiting & Abuse Prevention
**Rate Limiting Strategy**:
- **Per-User Limits**: 100 store browsing requests/minute, 10 purchase attempts/minute, 50 inventory operations/minute
- **Global Limits**: 10,000 concurrent purchase operations, Circuit breakers for external services

**Abuse Prevention**:
- **Purchase Velocity**: Max 5 identical item purchases/hour
- **Gem Farming Detection**: Pattern analysis for suspicious earning
- **IP-based Limiting**: Block rapid gem deductions from same IP.
- **Idempotency Keys**: Mandatory for all purchase requests.

### 4.8 Eager Loading/Denormalization
**Eager Loading**:
- For GET /character/{id}, join character_profiles + user_inventory.

**Denormalization**:
- **character_profiles.equipped_item_id**: Avoid joins when rendering characters.
- **user_inventory.item_name**: Reduce reads to store_items.

## 5. Bonus

### 5.1 Scaling Considerations
**Infrastructure Scaling**:
- **Horizontal Scaling**: Microservices with load balancers and API Gateway
- **Database Sharding**: Shard by user_id for inventory/purchases
- **CDN Integration**: Static assets and API responses
- **Auto-scaling**: Container orchestration (Kubernetes)

**Data Archiving**:
- **Hot Data**: Recent purchases, active inventory (SSD)
- **Warm Data**: Historical purchases 3-12 months (Standard storage)
- **Cold Data**: Archives >1 year (Glacier/cheap storage)

**Monitoring & Alerting**:
- **Real-time Metrics**: Purchase success rates, API latency
- **Business Metrics**: Revenue per user, inventory turnover
- **Error Tracking**: Failed fulfillments, payment issues

### 5.2 Optimize gem deduction and purchase flow for Transactional safety
- [🔗  Sequence Diagram for Transactional safety](https://mermaid.live/edit#pako:eNqNVf1v2jAQ_VdO_qlVoU34KBCJVi1QDfWDDpgqbUzIxAfNljiZ47RliP99TgKEkJAuQiTx-Z7fvXd2VsR0GRKD-PgnQG5i16ILQZ0JB3V5VEjLtDzKJXRsC7nMjj8HwnylPk5HKN4sE7MzXqhtozwe7_M3heyK5fEpXSrpTC2SjdwF9tyybUchTL8GGGymxP8x5_LV1dkhSwOeB6MxXHibcVhJQblPTWm5fGqxElgSHf_Hz_U-3pMrEdw3FJDF6zN0PBXn5hI6r2j-jlMOJyoy22IMGPUeep0x7Ei8fOkNe5BmAm24jqGonagN-GH50o8D4bUFLasFsuR64WyLL3ZLJYkZggog1s2AiqbB4B5O8DBd8TqNIdBWZJ7wPQe5SK1bXFgcxkmlBYSUfekWMuBh0LmHwEcxfY8icDcYwrfn7s24lwClk_KEXyiLpzNqU9X4G_Ej0Fj1XNAcdofkUqhXbZCupPbUdH15neAkT6Gtfe4H87nqaSU7zAPO9pzNKSXX4xRGyCENUeRzTfl8SxkMw0PAl0li5O6okFphg_efRr3hXoOf-JLKwG97yJnqp9NPoQ617SILzLg8mAvXgY3OhXLtM4r93O8dX3XDQSuU2xDa9Sm7s8zRZcANY_HZoXwHaxtPQ2XSckS72Gd6BKdQ-03-Tvuwyo38put4qnJkabj0W9H27QweH_vj_P2bRcqhmTm2DYhuqsN2AfjlznyYLYGhbSkey6lcevj_XV3RdOgIpKpOOMmeXFF_c3b0vFBI5WzleyXDVsYEImtsPko_ahDK2NaCHZG8gsrbioboey73kZTIQliMGFIEWCIOCoeGr2QVgkyIfEUHJ8RQj4yK3xMy4WuVoz6Y313X2aYJN1i8EmNO1SYvkcBjSqnN5383KhQxFB034JIYrctmBEKMFfkght7UzpvVhlap661qtanX6yWyVMOV80ZD09VPa9Rqteq6RP5Gq-rnlUqr3mrql7V6vVnTtcv1P0A4xog)

### 5.3 Handle Shopify order tracking and webhooks
- [🔗  Sequence Diagram for handle Shopify order tracking](https://mermaid.live/edit#pako:eNqtVmtv2jAU_SuWP0yblPKmQLQh9cGkfllZoZo0ISGT3IBLYme208IQ_303rxIIoE4tQkB877mPc49tNtSRLlCbavgTgXDglrO5YsFEEHyFTBnu8JAJQ4aRchZMw3QE6pk7UPb4Hvke9_0AhDntNFrIkHvr6c8IojPmq-Fd2XjLDJthCWXLL5gtpFxOB8INJRem7PGoQaWr6ecPaYDIZ1ClxmwyXKw1d5hPuIGAhJndTYGH7hf9_pHGbXKjgGEKb2cjUrl5EUcgGChv0CZ3P0aDh3ERPU3Q5LM2zET6WwjC5WL-5Vy4Pa5tknwRnS6mxZAnOTvOygH2mjnLuZKRcGMICZV0QGssIMXteRcy4xiRzvvRmFSZG3BRTbLqypOWopBR8fkC6fHIHu5OOH7kAuY2xofryFnqrzNV7RdJ4S7hggRgmIvcFVthviH3SY9OMgmX6MiJi0a4v059iqVjyosyZ41anWwyytIJYMptGZ03vpvg4_D2ajw4MsHRYExgZUAJ5r-GtEg-1wNmjyWJtWyTESogm6IjhcdVwAyXgkDAuJ-CwddQJCE2e2jVb22_tVpV26sVGSgl1emCDlAPYNSavHCzwEZDKbB3jrtphhqSnvcf5OHgIQiNtohAxqYqjjtl2e7G9s9LNxFR9pArFjT5hFuAh7q4G_dFHFPR7x8eKbmOX9J1Xc1UkUq6GjKelXMIfLsqcgmcC1WYfcjWybFSnP6H9ZNVCB_VlCOD0AfchhYxCoWAAp-KKJjlIzjfajyxECFESMM9PJsTKScKy6Od2PsM77S4jaoCD0-vvJ938_MaeBfv_Ry9KWb5vhrj-TnHUHO8rdI2dwGOXFeloiIkevrCMLOx8psnjROzKzRzdtI6EbMwqwxZViUaqUXnirvUNioCiwaADvEj3cQuE2oWEMCE2vjTZWo5oROxRQze4L-lDHIY3kPzBbU9hsebRaMQj_78f8urCyYDdYMXlqF2L4lA7Q1dUbverVW6zU6t0a73ms1uvd226BqXG5VOp1bHd63TarWaW4v-TVLWK41Gr93r1i9b7Xa3Va9dbv8B6DQsCw)
