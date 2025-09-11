# Referral System - High Level Design (HLD)

## 1. Executive Summary

The Referral System is designed to incentivize existing customers to bring new customers to the travel platform. The system provides a win-win mechanism where referrers earn points and referees get discounts on their bookings.

### Key Business Rules
- **Referral Code Generation**: Activated after first payment
- **Referee Benefit**: ₹2000 discount on booking
- **Referrer Benefit**: 2000 points after referee completes booking
- **Usage Constraints**: One-time use per customer, one code per booking
- **Points Validity**: 6 months from credit date

## 2. System Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                         External Systems                        │
├─────────────────┬───────────────────┬───────────────────────────┤
│   Mobile App    │    Web Portal     │     Admin Dashboard       │
└────────┬────────┴─────────┬─────────┴───────────┬───────────────┘
         │                  │                     │
         └──────────────────┼─────────────────────┘
                           │
                    ┌──────┴──────┐
                    │  API Layer  │
                    └──────┬──────┘
                           │
┌──────────────────────────┴─────────────────────────────────────┐
│                     Business Logic Layer                       │
├─────────────┬──────────────┬───────────────┬───────────────────┤
│  Referral   │   Points     │    Deal       │   Notification    │
│  Service    │   Service    │  Integration  │    Service        │
└─────────────┴──────────────┴───────────────┴───────────────────┘
                           │
┌──────────────────────────┴──────────────────────────────────────┐
│                      Data Access Layer                          │
├─────────────────────────┬───────────────────────────────────────┤
│      MongoDB            │           BigQuery                    │
│  (Customer & Points)    │       (Deals & Versions)              │
└─────────────────────────┴───────────────────────────────────────┘
```

## 3. Database Schema Design

### 3.1 Schema Relationships

```
┌──────────────────────────┐         ┌──────────────────┐
│       Customers          │         │      Deals       │
│      (MongoDB)           │         │    (BigQuery)    │
│ ┌──────────────────────┐ │         └─────────┬────────┘
│ │ Embedded:            │ │                   │
│ │ - referral_code      │ │                   │
│ │   - code             │ │                   │
│ │   - status           │ │                   │
│ │   - stats            │ │                   │
│ └──────────────────────┘ │                   │
└────────────┬─────────────┘                   │
             │                                 │
             │      ┌─────────────────┐        │
             ├──────┤ ReferralUsage  ├────────┤
             │      └────────┬────────┘        │
             │               │                 │
             │      ┌────────┴────────┐        │
             ├──────┤ ReferralPoints  ├────────┘
             │      └────────┬────────┘
             │               │
             │      ┌────────┴────────┐
             └──────┤  PointsLedger   │
                    └─────────────────┘
```

### 3.2 Data Model Overview

#### **3.2.1 Customer Collection (MongoDB)** 👤
**Purpose**: Master record for all customers with embedded referral codes

**Schema Structure**:
```javascript
{
  customer_id: "CUST123",
  name: "John Doe",
  email: "john@example.com",
  phone_number: "+919876543210",
  is_disabled: "",
  disabled_logs: "",
  
  // Embedded referral code document
  referral_code: {
    referral_code: "JOH-X7K9",
    status: "active",
    generated_at: "2025-01-10T10:00:00Z",
    first_payment_deal_id: "DEAL456",
    stats: {
      total_uses: 5,
      successful_referrals: 3,
      pending_referrals: 2,
      total_points_earned: 6000,
      total_discount_given: 10000
    },
    deactivated_at: null,
    deactivation_reason: null
  }
}
```

**Importance**:
- **Single Source of Truth** for customer and their referral code
- **Atomic Updates** for referral statistics
- **Embedded Design** eliminates joins and improves performance
- **Simplified Management** with referral code lifecycle tied to customer

**Key Responsibilities**:
- Manage customer profile information
- Generate and store referral codes after first payment
- Track referral performance metrics
- Enable/disable referral codes

#### **3.2.2 ReferralUsage Collection (MongoDB)** 🔄
**Purpose**: Transaction log of referral code applications

**Importance**:
- **Enforces Business Rules** (one-time use, no self-referral)
- **Tracks Full Lifecycle** from application to completion
- **Prevents Fraud** through unique constraints
- **Audit Trail** for all referral activities

**Key Responsibilities**:
- Record when a code is applied to a booking
- Enforce one-code-per-customer rule
- Track payment and booking completion
- Link referrer and referee for point credits

#### **3.2.3 ReferralPoints Collection (MongoDB)** 💎
**Purpose**: Manages earned points and their lifecycle

**Importance**:
- **Points Wallet** for each customer
- **Expiry Management** with automatic tracking
- **Redemption Control** with partial usage support
- **Financial Liability** tracking for business

**Key Responsibilities**:
- Store credited points with expiry dates
- Track redemption history
- Support partial redemptions
- Manage point expiration

#### **3.2.4 PointsLedger Collection (MongoDB)** 📊
**Purpose**: Immutable audit log of all point transactions

**Importance**:
- **Complete Audit Trail** for compliance
- **Balance Verification** through transaction history
- **Dispute Resolution** with detailed records
- **Financial Reconciliation** support

**Key Responsibilities**:
- Record every point transaction
- Maintain balance integrity
- Provide transaction history
- Support financial audits

#### **3.2.5 Deals Table (BigQuery)** 📝
**Purpose**: Stores deal information including applied referral codes

**Key Fields**:
- `deal_id`: Unique identifier
- `customer_id`: Link to customer
- `referral_code_applied`: Stores the applied referral code
- `is_converted`: Booking completion status
- `sold_ts`: Timestamp when deal was sold

## 4. Data Flow Diagrams



### 4.1 Referral Application Flow

```
Customer Applies Code → Find Customer by Code → Validate Rules
         │                      │                     │
         ↓                      ↓                     ↓
   Update Deal ←──── Apply Discount ←──── Check Embedded Code
                                              
                                              
                                      
```

### 4.2 Points Credit Flow

```
Deal Converted → Check Referral in Deal → Find Referrer Customer
       │                 │                        │
       ↓                 ↓                        ↓
Payment Done ──→ Create ReferralUsage ──→ Update Embedded Stats
                         │                        │
                         ↓                        ↓
                 Create ReferralPoints    Update in Customer Model
                         │
                         ↓
                  Add to PointsLedger
```

## 5. Key Business Flows

### 5.1 Complete Referral Journey with Embedded Architecture

```
1. Customer A completes first payment
   └─→ Customer.referral_code embedded document created
   └─→ Code: JOH-X7K9 (will be the booking id) with initial stats

2. Customer A shares code with Friend B
   └─→ Code retrieved from Customer.referral_code
   └─→ Available in app/web

3. Friend B applies code during booking
   └─→ Find Customer by referral_code.referral_code
   └─→ Validate Customer.referral_code
   └─→ Apply ₹2000 discount to deal
   └─→ Update deal.referral_code_applied

4. Friend B completes payment (Deal Conversion)
   └─→ ReferralUsage record created
   └─→ Customer.referral_code.stats updated in-place
   └─→ ReferralPoints created (2000 points)
   └─→ PointsLedger entry added
```