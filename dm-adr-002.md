# DM-ADR-002: Adopt Event-Driven Architecture for User Notifications

**Status:** approved
**Date:** 2026-06-15

**Tags:** messaging, kafka, notifications, db, ohh

## Context

Currently, our notification system uses synchronous HTTP calls between microservices to trigger emails and push notifications. This tightly couples the services, leading to cascading failures when the notification service is down or slow, and makes it difficult to scale the core transaction services during peak load.\n\nWe need a way to decouple these services to improve overall system resilience and responsiveness.

## Decision

We will adopt an **Event-Driven Architecture** using **Apache Kafka** as our central message broker for all user notifications.\n\nServices will publish `UserActionCompleted` events to a Kafka topic. The Notification Service will act as a consumer, picking up these events asynchronously and processing the required emails or push notifications.

## Consequences

### Positive\n- **Decoupling**: Core services no longer depend on the uptime of the Notification Service.\n- **Resilience**: Spikes in traffic won't crash the notification pipeline, as messages will queue up safely.\n\n### Negative\n- **Complexity**: Introduces a new piece of infrastructure (Kafka) that we must maintain and monitor.\n- **Eventual Consistency**: Users might experience a slight delay in receiving emails compared to synchronous calls.

## Alternatives Considered

- **RabbitMQ**: Evaluated but rejected because we anticipate very high throughput and want to leverage Kafka's replayability feature for auditing.\n- **Redis Pub/Sub**: Rejected because messages are not persistent, and we cannot risk losing notifications if the consumer crashes.

## Design Changes

### API Changes

Removed `POST /internal/notifications` endpoint from the Notification Service.

### Workflow Changes

Checkout workflow no longer waits for email confirmation before returning `200 OK` to the user.

### Service Changes

Added Kafka Producer library to `BillingService` and `AuthService`.

### Infrastructure Changes

Provisioned a managed Confluent Kafka cluster.

## Major Impacts

### Operational Impact

Need to add Datadog alerts for Kafka consumer lag.

### Testing Impact

End-to-end tests must be updated to poll for asynchronous email delivery.

### Security Impact

Need to configure mTLS for Kafka clients.

### Documentation Impact

Update the Developer Guide with instructions on how to produce and consume events locally using Docker Compose.

### Scalability Impact

The notification service can now be scaled horizontally based on the size of the Kafka consumer group.

## References

### Design Documents

- https://wiki.example.com/architecture/event-driven

### External References

- https://kafka.apache.org/documentation/
