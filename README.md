# Transaction Processing System
## Design Key Points
### ERD
![2024-10-06_07-27](https://github.com/user-attachments/assets/82d5f820-eba0-4c81-a5cc-9ec17ec63c65)

### Event sourced balance storage
A checkpoint is created for user balance and balance is calculated throught this log

### Durable communication
Using _Listen to yourself_ pattern 
![Untitled-2024-08-13-1353](https://github.com/user-attachments/assets/d3100012-28af-43fd-a0c4-819958165201)

### Simple immutable ledger
No update to transaction statuses only addition which is suitable for financial systems

## Sequences
