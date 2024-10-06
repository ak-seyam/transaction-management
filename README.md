# Transaction Processing System

## Design Key Points
* **Event sourced balance storage**
A checkpoint is created for user balance and balance is calculated throught this log

* **Durable communication**
Using _Listen to yourself_ pattern 
![Untitled-2024-08-13-1353](https://github.com/user-attachments/assets/d3100012-28af-43fd-a0c4-819958165201)

* **Simple immutable ledger**
No update to transaction statuses only addition which is suitable for financial systems

* **Race condition prevention**
Through having a lock on card to prevent another concurrent authorizations

* **Horizontally scalable**
The application is stateless by design

* **Leveraging NestJs design patterns for building decoupled systems**
Through usage of observable pattern for example to decouple authorization handler from event listeners

## ERD
![2024-10-06_07-27](https://github.com/user-attachments/assets/82d5f820-eba0-4c81-a5cc-9ec17ec63c65)


## Sequences
⚠️ **TPS** is the transaction processing serive behind the gateway 
### Authorization
```mermaid
sequenceDiagram
  PSP->>Gateway: Auth confirmed
  Gateway->>TPS: Submit Auth event
  TPS-->>Kafka: Create event
  TPS->>Gateway: Ack
  Gateway->>PSP: Final status
  Kafka-->>TPS: Get transaction event
  TPS->>TPS: Check user balance
  alt User have enough bakance
    TPS->>TPS: Submit transaction
    TPS->>Notificaiton: Notify the rest of the compoenents
    TPS->>Analytics: Send analytics events
  else User don't have balance
    TPS->>Notification: Publish failure event
    TPS->>Analytics: Send analytics events
  end
```
