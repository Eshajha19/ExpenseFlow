# Predictive Subscription Lifecycle & Automated Renewal Engine

## 🚀 Overview
Issue #647 transforms the static subscription tracker into a dynamic, state-aware engine. It predicts future cash flow, automates the transition from trials to active status, and identifies financial leaks through intelligent usage auditing.

## 🏗️ Core Components

### 1. State Machine & Models (`models/Subscription.js`)
Subscriptions now manage their own lifecycle through a formalized state machine:
- **Statuses**: `active`, `trial`, `paused`, `cancelled`, `expired`, `grace_period`.
- **Transitions**: `transitionTo(newStatus, note)` records audit-ready logs of why a state changed (e.g., "Trial converted", "Payment failed - Entering grace period").
- **Predictive Virtuals**: `daysUntilPayment`, `monthlyAmount`, and `yearlyAmount` provide real-time metrics.

### 2. Predictive Forecasting (`utils/predictiveMath.js`)
A specialized utility that projects cash flow requirements:
- **Timeline Projection**: Generates a day-by-day projected bill list for any period (default 30 days).
- **Renewal Probability**: Uses weighted heuristics (Usage Frequency + Value Rating + History) to calculate how likely a user is to keep a subscription.

### 3. Automated Renewal Engine (`services/subscriptionService.js`)
Handles the heavy lifting of keeping the financial record accurate:
- **Auto-Billing**: Detects due subscriptions and automatically creates corresponding `Expense` records.
- **Trial Conversion**: Automatically promotes `trial` status to `active` based on trial end dates.
- **Pattern Detection**: Scans global expenses for recurring merchant names and amounts with low variance, suggesting them as new subscriptions.

### 4. Background Processing (`jobs/renewalWorker.js`)
A precision-timed worker that runs:
- **Daily (00:00)**: Processes all renewals and state transitions.
- **Weekly (Sun 02:00)**: Performs deep data audits to identify high-risk subscriptions.

## 🛠️ API Reference

### `GET /api/subscriptions/forecast?days=X`
Returns a detailed timeline of upcoming charges and total projected cash-flow impact.

### `GET /api/subscriptions/audit`
Returns a "Financial Health" report flagging unused subscriptions and high-impact costs.

### `GET /api/subscriptions/detect-patterns`
Analyzes raw expense history to find hidden recurring payments.

### `POST /api/subscriptions/:id/transition`
Manually moves a subscription through the state machine.

## ✅ Verification
1. Run lifecycle tests:
   ```bash
   npm test tests/subscription.test.js
   ```
2. Manually trigger a renewal batch:
   ```bash
   curl -X POST http://localhost:3000/api/subscriptions/trigger-renewals
   ```
