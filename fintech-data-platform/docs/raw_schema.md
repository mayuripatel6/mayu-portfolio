# Raw Data Schemas

This document describes the raw datasets ingested into the fintech data platform.
These datasets are assumed to be ingested daily as Parquet files with minimal transformation applied.

---

## transactions_raw

**Description:**  
Represents all financial transactions made by users across accounts.

**Fields:**
- transaction_id
- account_id
- user_id
- transaction_type
- amount
- currency
- transaction_timestamp
- merchant_id
- country
- ingestion_date

**Assumptions & Notes:**
- `transaction_id` is globally unique
- A given `account_id` can have multiple transactions over time
- `amount` is positive for credits and negative for debits
- `transaction_timestamp` represents event time, not ingestion time
- Late-arriving transactions are possible
- Transaction data may be subject to anonymisation or masking requirements for non-privileged consumers
- Fraud indicators are expected to be derived downstream rather than stored in raw data

**Risks / Design Considerations:**
- Currency conversion may be required for analytics and reporting
- Duplicate transactions may be produced by upstream systems and must be handled downstream
- Transaction reversals and corrections must be supported:
  - It is unclear whether reversals will be emitted as new records or updates to existing records
  - Multiple records may exist for a single transaction to reflect state changes over time

---

## accounts_raw

**Description:**  
Represents bank accounts held by users. Account attributes may change over time and may require historical tracking in curated layers.

**Fields:**
- account_id
- user_id
- account_type
- opened_date
- status

**Assumptions & Notes:**
- One user may hold multiple accounts
- Account status can change over time
- `account_id` can be joined to `transactions_raw.account_id`

**Risks / Design Considerations:**
- It is unclear whether upstream data will be delivered as full refreshes or incremental updates
- The ingestion and curation strategy must account for the expected update pattern

---

## users_raw

**Description:**  
Represents registered users of the platform.

**Fields:**
- user_id
- country
- created_at
- kyc_status

**Assumptions & Notes:**
- KYC status may change over time
- User country may not reflect current residence

**Risks / Design Considerations:**
- The platform may operate across multiple countries, requiring consistent country definitions
- Regulatory requirements may affect how user data can be stored and exposed
