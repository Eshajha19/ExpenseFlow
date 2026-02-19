# Asynchronous Event-Driven Architecture (EDA)

## 🚀 Overview
Issue #711 transitions the application from a tightly coupled "Monolithic Linkage" model to a decoupled **Asynchronous Event-Driven Architecture**. By using an internal Event Bus, services no longer need to know about each other's existence to trigger side-effects (like emails, audit logs, or analytics updates).

## 🏗️ Technical Components

### 1. Centralized Event Bus (`utils/AppEventBus.js`)
An enhanced wrapper around Node.js `EventEmitter` that provides:
- **Asynchronous Execution**: Listeners are executed in their own event loop cycles.
- **Error Boundaries**: Failures in a side-effect listener (e.g., mail server down) do NOT crash the primary request or bubble up to the end user.
- **Telemetry**: Built-in metrics tracking for event counts and listener failures.

### 2. Global Event Registry (`config/eventRegistry.js`)
A single source of truth for all system events.
- **Naming Convention**: `[domain].[action]` (e.g., `user.registered`, `transaction.deleted`).
- Prevents "Magic Strings" from polluting the business logic.

### 3. Decoupled Listeners (`listeners/`)
Stateless classes that "react" to system events.
- **`EmailListeners.js`**: Replaces direct `emailService` calls. Handles all outbound communication.
- **`AuditListeners.js`**: Handles compliance, forensic logging, and background analytics updates.

## 🔄 The Flow
1. **User Request**: Client hits `/api/auth/register`.
2. **Business Logic**: `AuthRoute` saves the user to the database.
3. **Publication**: `AuthRoute` calls `AppEventBus.publish(USER.REGISTERED, user)`.
4. **Decoupled Handlers**: 
   - `EmailListener` sees the event and queues a welcome email.
   - `AuditListener` sees the event and logs the registration in the forensic store.
5. **Immediate Response**: The server returns `201 Created` to the user without waiting for the email to actually send.

## ✅ Benefits
- **Performance**: Faster response times as side-effects move out of the request/response path.
- **Reliability**: If the email service fails, the user registration still succeeds.
- **Scalability**: New features (e.g., a "Slack Notification" service) can be added simply by creating a new listener, without touching existing code.

## 🧪 Testing
Run the event bus integration tests:
```bash
npx mocha tests/eventBus.test.js
```
