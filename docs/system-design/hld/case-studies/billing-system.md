---
title: Billing System
tags: [hld, billing, metering, pricing, invoices]
status: draft
---

# Billing System

Designing a billing system like AWS involves managing various services, usage tracking, pricing models and customer billing. AWS's billing system handles metered billing, subscription services, dynamic pricing and invoice generation.

## High-level components

1. **Admin module:** manages pricing updates for different services.
2. **User/customer module:** lets users access pricing details, view metrics and receive invoices.
3. **Service management module:** tracks and logs user activity for different services (compute, storage, etc.).
4. **Pricing engine:** calculates costs dynamically based on usage.
5. **Billing and invoice management:** generates invoices from calculated charges and manages payments.
6. **Notification module:** sends billing notifications and usage alerts to users.
7. **Reporting and analytics:** provides detailed cost breakdowns and analytics.

## Detailed design

### 1. Admin module

**Responsibilities**

- **Update pricing:** admins can update pricing for services, including tiers (on-demand, reserved).
- **Set policies and discounts:** define discount policies for specific users or services.

**Endpoints**

- `POST /pricing/update` — update service prices.
- `POST /discounts/apply` — apply discounts to specific users or services.

**Database**

Pricing table stores pricing information for different services:

```sql
CREATE TABLE Pricing (
    service_id INT PRIMARY KEY,
    service_name VARCHAR(100),
    base_price DECIMAL(10, 2),       -- base price per unit of usage
    tiered_pricing JSON,             -- pricing based on tiers (e.g. per GB, per hour)
    reserved_pricing DECIMAL(10, 2), -- discounted pricing for reserved instances
    tax DECIMAL(5, 2)                -- tax percentage
);
```

### 2. User/customer module

**Responsibilities**

- **View pricing:** check pricing details for each service.
- **View metrics and usage:** view service consumption metrics.
- **Download invoices:** access detailed billing statements.

**Endpoints**

- `GET /pricing/view` — fetch current pricing details.
- `GET /metrics/dashboard` — view the user's usage metrics.
- `GET /invoice/download` — download an invoice for a specific billing cycle.

**Database**

- **User table:** user information and account details.
- **Metrics table:** usage metrics for each user.

```sql
CREATE TABLE Users (
    user_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    plan_id INT,
    created_at TIMESTAMP
);

CREATE TABLE Metrics (
    metric_id INT PRIMARY KEY,
    user_id INT,
    service_id INT,
    usage_amount DECIMAL(10, 2),  -- resource consumed (e.g. GB, compute hours)
    last_updated TIMESTAMP
);
```

### 3. Service management module

**Responsibilities**

- **Track usage:** monitors and logs user activity for services (storage, compute, networking).
- **Update metrics:** continuously updates the Metrics table with resource consumption.

**Endpoints**

- `POST /service/usage-log` — record user activity for a specific service.
- **Metrics update pipeline:** a background job that tracks and updates metrics data regularly.

### 4. Pricing engine

**Responsibilities**

- **Dynamic pricing calculation:** based on usage metrics, calculate cost using pricing tiers, discounts, taxes and reserved rates.
- **Apply taxes and discounts** when calculating final costs.
- **Pricing model:** supports pay-as-you-go, tiered pricing and reserved pricing.

**Endpoints**

- `GET /pricing/calculate` — calculate the price for a user's services based on their metrics.

```java
public class PricingEngine {
    public double calculatePrice(User user, List<Metrics> usageMetrics) {
        double totalCost = 0;
        for (Metrics metric : usageMetrics) {
            Pricing pricing = pricingRepository.findByServiceId(metric.getServiceId());
            double basePrice = pricing.getBasePrice();
            double usage = metric.getUsageAmount();
            double cost = usage * basePrice;

            // Apply tiered pricing (if applicable)
            if (pricing.hasTieredPricing()) {
                cost = applyTieredPricing(usage, pricing);
            }

            // Apply tax
            cost += cost * pricing.getTax();
            // Check if the user has any discounts
            totalCost += cost;
        }
        return totalCost;
    }

    private double applyTieredPricing(double usage, Pricing pricing) {
        // Logic to apply tiered pricing based on usage
        return pricing.calculateTieredCost(usage);
    }
}
```

### 5. Billing and invoice management

**Responsibilities**

- **Generate invoices:** at the end of each billing cycle, generate an invoice with a breakdown of charges.
- **Store invoices:** keep a record of each invoice for reference.
- **Payment handling:** manage payments for each invoice (credit cards, direct debit, prepaid accounts).

**Endpoints**

- `POST /billing/invoice/generate` — generate an invoice based on the user's activity.
- `GET /billing/invoice/view` — view invoice details for a specific billing cycle.

```sql
CREATE TABLE Invoices (
    invoice_id INT PRIMARY KEY,
    user_id INT,
    total_amount DECIMAL(10, 2),
    generated_at TIMESTAMP,
    due_date TIMESTAMP,
    payment_status VARCHAR(50)
);
```

### 6. Notification module

**Responsibilities**

- **Usage alerts:** alert users if they are nearing usage limits or if spending exceeds a threshold.
- **Billing reminders:** remind users of due invoices.

**Endpoints**

- `POST /notifications/alert` — send alerts for usage or billing.

**Database**

Notification table stores notification details (sent, read, type, etc.).

### 7. Reporting and analytics

**Responsibilities**

- **Cost breakdown:** detailed costs for each service used (compute hours, data transfer, etc.).
- **Trends and usage insights:** trends in resource usage over time to help optimize costs.

**Endpoints**

- `GET /analytics/usage-trends` — usage and cost trends over a period.
- `GET /analytics/cost-breakdown` — detailed cost breakdown for each service.

**Database**

Analytics table stores historical usage and cost data.

## High-level architecture

```
+----------------------------------------------------------+
|                        AWS Billing System                |
+----------------------------------------------------------+
|  Admin Module              User/Customer Module          |
|  Service Management Module (tracks EC2, S3, Lambda, ...) |
|  Pricing Engine            Billing & Invoice Mgmt        |
|  Reporting & Analytics     Notification Module           |
+----------------------------------------------------------+
```

## Flow example

1. **User spins up an EC2 instance**
    - Service management logs the activity (e.g. 10 compute hours).
    - Metrics module updates the Metrics table with the latest usage data.
2. **Monthly invoice generation**
    - At month end, billing and invoice management uses the pricing engine to calculate total cost from activity and pricing.
    - The system generates an invoice and stores it in the Invoice table.
3. **User views invoice**
    - User retrieves the invoice via `GET /billing/invoice/view`, with a breakdown of usage and costs.
4. **Notification**
    - Notification module alerts the user if they are nearing a monthly spending limit or if a payment is due.

## Tech stack suggestions

- **Backend:** Java (Spring Boot) or Node.js for REST APIs.
- **Python** for analytics and reporting (optional).

| Module | Database type | Recommended |
|---|---|---|
| Admin (pricing) | Relational (SQL) | Amazon RDS (MySQL/PostgreSQL), Google Cloud SQL |
| User/customer (user data) | Relational (SQL) | Amazon RDS (PostgreSQL), Google Cloud SQL |
| User/customer (metrics) | Time-series | Amazon Timestream, InfluxDB, Prometheus |
| Service management (logs) | NoSQL (document) | Amazon DynamoDB, MongoDB, Cassandra |
| Pricing engine | Relational or NoSQL | Amazon Aurora, DynamoDB |
| Billing & invoice management | Relational (SQL) | Amazon RDS (PostgreSQL/MySQL), Oracle |
| Notification | NoSQL or message queue | DynamoDB, SQS, Kafka |
| Reporting & analytics | Data warehouse | Redshift, BigQuery, Snowflake |

## Added: deep dives

> [!info] Added
> The sections below are not in the original doc. Related notes: [Payment System](payment-system.md), [Digital Wallet](digital-wallet.md).

- **Idempotent metering:** every usage event has an `event_id`; ingest is at-least-once, aggregates are exactly-once *effect*.
- **Price versioning:** stamp `price_version` on each invoice line so retroactive price edits don't rewrite history.
- **Credits and commitments:** prepaid credits and reserved instances are first-class adjustments before tax.
- **Reconciliation:** sum of usage × price should match invoice totals; mismatches go to a reconciliation job.
