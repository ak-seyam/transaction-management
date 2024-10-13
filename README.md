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
  Gateway->>PSP: REC status
  Kafka-->>TPS: Get transaction event
  TPS->>TPS: Check user balance
  alt User have enough bakance
    TPS->>TPS: Submit transaction
    TPS->>DB: Create new balance checkpoint
    TPS->>Notificaiton: Notify the rest of the compoenents
    TPS->>Analytics: Send analytics events
  else User don't have balance
    TPS->>Notification: Publish failure event
    TPS->>Analytics: Send analytics events
  end
```

### Balance check
```mermaid
sequenceDiagram
  TPS->>DB: Get the last checkpoint
  DB->>TPS: latest checkpoint
  alt there is existing checkpoint
    TPS->>DB: Get all transactions after this checkpoint
    DB->>TPS: Trasactions
    TPS->>TPS: Calculate balance
  else no checkpoints
    TPS->>DB: Get all transactions for this user (In this case it should be none)
    TPS->>TPS: calculcate the balance from card provisioned limit
  end
```

### Clearing
```mermaid
sequenceDiagram
  PSP->>Gateway: Clearing confirmed
  Gateway->>TPS: Submit Clearing event
  TPS-->>Kafka: Create event
  TPS->>Gateway: Ack
  Gateway->>PSP: Final status
  Kafka-->>TPS: Get transaction event
  TPS->>TPS: Check if user got auth for this clear
  alt Got Auth
    TPS->>TPS: Submit clreaing
    TPS->>Notificaiton: Notify the rest of the compoenents
    TPS->>Analytics: Send analytics events
  else User don't have balance
    TPS->>Notification: Publish clear reject event
    TPS->>Analytics: Send analytics events
  end
```

### Getting transaction history
```mermaid
sequenceDiagram
  Service->>Gateway: Get user's transactions
  Gateway->>TPS: Get transactions
  TPS->>TPS: Get events for the user
  TPS->>TPS: Set the latest status to the response
  TPS->>Gateway: Replay back with transactions (I have omitted pagination in the imeplementation for simplicity)
```

## Running the project
### System Requirements
- protoc
- docker & docker compose
- node > v18.19.x 
### How to run
1. `docker compose up` to start postgresql and kafka
2. run `npm install`in every submodule
3. run `npm run start:dev` to start the dev server
