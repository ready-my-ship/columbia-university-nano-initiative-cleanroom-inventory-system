# Columbia University Nano Initiative
## Cleanroom Garment Inventory System

A production-oriented inventory management system developed for the Columbia University Nano Initiative to replace manual cleanroom garment tracking workflows with a centralized database and browser-based application.

The system manages garment checkout and returns, barcode-based inventory tracking, damage accountability, vendor invoice processing, ruin-cost tracking, and operational reporting.

## Business Problem

Cleanroom garment inventory was managed through a largely manual workflow involving spreadsheets, barcode scans, recurring file copies, vendor reports, and manual record verification. This created several operational challenges:

- Garment checkout and return records were distributed across manually maintained files.
- Identifying the student responsible for a damaged garment required searching historical records.
- Vendor-reported damages and invoices had to be manually reconciled with internal garment records.
- Ruin costs were difficult to aggregate and attribute efficiently.
- New and replacement garment SKUs required manual inventory maintenance.
- Repetitive weekly data handling increased administrative workload and the risk of inconsistent records.

The project was designed to centralize these workflows and provide the lab manager with a single operational system for inventory, accountability, and cost tracking.

## System Architecture

The production system runs on an on-site Raspberry Pi 5 that serves as the application and database server. The lab manager accesses the inventory system through a standard web browser, while FastAPI handles application logic and PostgreSQL maintains the centralized transactional data.

The architecture separates the user interface, application logic, database, reporting, source control, and backup components. Barcode scanning and vendor CSV data feed operational workflows, while Power BI provides analytical reporting from the centralized data. The system is also structured to support integration with the laboratory's existing NEMO environment.

<img width="1536" height="1024" alt="columbia_inventory_architecture_diagram" src="https://github.com/user-attachments/assets/9d7ee810-0ddc-4641-b140-c6a8c63f5281" />

## Technology Stack

| Technology | Role in the System |
|---|---|
| **PostgreSQL** | Central relational database for garments, users, checkouts, returns, damages, invoices, transactional history, and archived records |
| **FastAPI** | Python backend providing application logic, database interaction, and REST API endpoints |
| **SQLAlchemy** | Database connectivity and SQL execution between the FastAPI application and PostgreSQL |
| **HTML / CSS / JavaScript** | Browser-based user interface used by the lab manager for operational workflows |
| **Uvicorn** | ASGI server used to run and serve the FastAPI application |
| **Raspberry Pi 5** | On-site Linux application and PostgreSQL database server |
| **Barcode Scanner** | Hardware input for rapid garment identification during checkout and return workflows |
| **Power BI** | Reporting and analytics for inventory, damages, invoice costs, and operational trends |
| **Git / GitHub** | Version control, source-code management, and technical documentation |
| **Linux** | Production operating environment for the Raspberry Pi deployment |


## Core Workflows

### User Tracking and Garment Assignment

1. Cleanroom users, including students and guests, are maintained as individual records using their University ID and identifying information.
2. Garments issued during checkout are linked to the individual user, creating a traceable record of who used each garment.
3. Current and historical garment assignments can be retrieved by user or garment barcode.
4. User-to-garment relationships are preserved through checkout history to support operational tracking, damage accountability, and historical analysis.

### Hanger Assignment and Cleanroom Organization

1. Users are assigned designated hanger numbers to organize garments within the cleanroom.
2. Hanger numbers are recorded during checkout alongside the user and issued garments.
3. Hanger assignments provide the lab manager with a direct relationship between the user, their garments, and their designated cleanroom storage location.
4. The centralized system replaces separate manual records previously used to track these relationships.

### Garment Checkout
1. The lab manager scans the garment barcode and records the user's University ID and assigned hanger number.
2. The application validates the garment and user records.
3. A checkout transaction is created and linked to each garment issued.
4. The garment status is updated to reflect that it is currently checked out.

### Garment Return
1. Returned garments are scanned before being sent for laundering.
2. The application links each returned garment to its originating checkout transaction.
3. A return transaction is recorded while preserving the complete checkout and return history.
4. Garment status is updated as the item progresses through the laundry cycle.

### Send to and Receive from Vestis

1. Returned garments are scanned and recorded before being sent to Vestis for laundering.
2. Shipment records preserve which garments left the cleanroom and their current inventory status.
3. Clean garments are scanned when received back from Vestis.
4. Receipt records update the garment lifecycle while preserving the previous checkout, return, and shipment history.
5. The workflow provides traceability between garments leaving the cleanroom and subsequently returning to inventory.

### Damage Accountability
1. Vestis damage data is imported using the garment SKU.
2. The system matches the SKU to the corresponding garment and historical checkout.
3. The associated user and checkout can be identified without manually searching historical spreadsheets.
4. Damage type and record date are stored for historical reporting and accountability.

### Invoice and Ruin-Cost Tracking
1. Vestis invoice data is imported and matched to garments by SKU.
2. Damage and invoice records are connected to the corresponding garment history.
3. Ruin charges can be attributed to individual garments and their associated checkout records.
4. Costs can be aggregated by reporting period for operational and financial analysis.

### Inventory Search and Operational Lookup

1. The lab manager can search directly by garment barcode to retrieve garment information and associated history.
2. Users can be located by University ID to retrieve their current garment and hanger assignments.
3. Historical relationships between users, garments, checkouts, returns, damages, and costs remain available for investigation and reporting.
4. Centralized search replaces manual lookup across recurring spreadsheets and historical files.

### Historical Records and Traceability
1. Checkout, return, damage, and invoice transactions are retained rather than overwritten as new activity occurs.
2. Historical records preserve the relationship between garments, users, checkout events, returns, damages, and associated costs.
3. Individual garments can be traced by SKU across their operational history.
4. Historical records support damage investigations, user accountability, cost analysis, and long-term inventory reporting.

## Database Design

The system uses PostgreSQL as the centralized transactional database, with operational data organized within the `inventory` schema. The relational model was designed to preserve garment history while connecting users, garments, checkout and return activity, vendor processing, damages, and invoice costs.

Rather than storing the workflow in a single flat table, the database separates major business entities and transactional events into related tables. Primary and foreign keys maintain relationships between records, while unique constraints, validation rules, and database triggers help protect data integrity as garments move through the inventory lifecycle.


### Core Data Model

| Table | Purpose |
|---|---|
| `users` | Stores cleanroom users and identifying information used for garment assignment and lookup |
| `garments` | Maintains the master inventory of individually identifiable garments and their current status |
| `checkouts` | Records garment checkout transactions and associates them with users |
| `checkout_items` | Connects individual garments to checkout transactions and preserves item-level checkout details |
| `returns` | Records garment return transactions |
| `return_items` | Connects individual returned garments to their corresponding return and checkout history |
| `received` | Records garment receipt events when inventory is received back from Vestis |
| `received_items` | Stores the individual garments associated with each receipt event |
| `damages` | Records garment damage and ruin events for accountability and historical tracking |
| `invoices` | Stores Vestis invoice-level information for financial tracking |
| `invoice_items` | Stores individual invoice line items and associated garment costs |
| `damage_invoice_items` | Connects damage records with corresponding invoice charges |
| `pending_garments` | Holds newly encountered garment barcodes requiring review before being incorporated into the inventory |


### Historical Data Architecture

Historical inventory records are maintained separately from the active transactional model through the PostgreSQL `archive` schema.

| Schema / Table | Purpose |
|---|---|
| `archive.inventory_history` | Preserves historical inventory records for long-term traceability, auditing, operational analysis, and recovery of prior garment information |

Separating historical records from the active `inventory` schema allows the operational tables to represent the current transactional system while retaining prior inventory information for investigation and analysis.

Database trigger functions automate key garment lifecycle transitions:

- `set_garment_checked_out()` — updates garment status when an item is checked out.
- `set_garment_sent_to_vestis()` — updates garment status as returned garments enter the Vestis processing workflow.
- `set_garment_available()` — restores garments to available inventory after they are received back.
- `set_garment_retired()` — removes ruined or retired garments from active circulation.

These database-level automations reduce manual status management and help keep garment state synchronized with transactional activity.

### Data Relationships and Traceability

The relational model connects operational events across the full garment lifecycle while avoiding unnecessary duplication of data.

Key relationships include:

- A user can have multiple checkout transactions over time.
- A checkout can contain multiple garments through `checkout_items`.
- Returned garments remain connected to their originating checkout history through `return_items`.
- Garment records provide a consistent identifier for tracing individual items across checkout, return, damage, receipt, and invoice activity.
- Damage records can be associated with invoice charges through `damage_invoice_items`, connecting operational incidents with their financial impact.
- Invoice headers and individual charges are separated through `invoices` and `invoice_items`, allowing multiple line items to belong to a single vendor invoice.
- Historical inventory information is retained separately in `archive.inventory_history` for long-term traceability.

This structure allows the system to answer operational questions such as which user had a garment, when it was checked out and returned, whether it was subsequently damaged or retired, and what costs were associated with that garment.

## Reporting and Analytics

The centralized PostgreSQL database provides a consistent source for operational and analytical reporting. Transactional records can be joined across users, garments, checkouts, returns, damages, receipts, and invoices to analyze both current inventory activity and historical trends.

Power BI provides the reporting layer for translating this operational data into dashboards and management-level metrics.

### Operational Reporting

The reporting layer supports analysis of:

- Current garment inventory and availability.
- Garment checkout and return activity.
- Garments currently associated with individual users.
- Garments sent to and received from Vestis.
- Damage and ruin frequency by garment, damage type, and reporting period.
- Retired garments and inventory replacement activity.
- Historical garment activity and lifecycle trends.

### Cost and Invoice Analytics

Vendor invoice data is connected with garment and damage records to support financial analysis, including:

- Total invoice and ruin costs by reporting period.
- Individual garment damage and replacement costs.
- Cost analysis by garment type and damage type.
- Identification of damage-related charges associated with historical checkout activity.
- Reconciliation of Vestis damage records with corresponding invoice charges.
- Long-term analysis of garment loss, damage, and replacement costs.

This reporting structure allows operational events to be analyzed alongside their financial impact rather than maintaining inventory, damage, and invoice information as separate reporting processes.

## Application and API Design

The inventory system uses a browser-based interface backed by a Python FastAPI application. The application acts as the operational layer between the lab manager and PostgreSQL, translating actions performed in the web interface into validated database transactions.

### Request and Data Flow

A typical application transaction follows this path:

1. The lab manager performs an action in the browser, such as scanning a garment or searching for a user.
2. JavaScript or an HTML request sends the required data to a FastAPI route or API endpoint.
3. FastAPI validates the request and applies the appropriate application logic.
4. SQLAlchemy sends the required SQL operation to PostgreSQL.
5. PostgreSQL reads or modifies the relational records while enforcing applicable constraints and triggers.
6. The result is returned through FastAPI to the browser.
7. The interface displays the updated information to the lab manager.

This separation keeps the browser responsible for user interaction, FastAPI responsible for application and workflow logic, and PostgreSQL responsible for persistent data storage and database-level integrity.

## Deployment and Reliability

The system is designed for on-site Linux deployment within the cleanroom environment, allowing the lab manager to access the application through a standard web browser while keeping operational services local to the facility.

Production reliability includes:

- Scheduled PostgreSQL backups using `pg_dump`.
- Local backup storage for routine recovery.
- Optional AWS S3 off-site backup for disaster recovery.
- Private GitHub source control and version history.
- Separation between development and production environments.

This architecture provides local operational independence while maintaining recoverability and controlled source-code management.

## Testing and Validation

The system was tested across database, application, and end-to-end operational workflows to verify data integrity and expected inventory behavior.

Testing included:

- User lookup and garment assignment.
- Barcode-based garment checkout and return.
- Garment status transitions throughout the inventory lifecycle.
- Sending garments to and receiving garments from Vestis.
- Damage and ruin recording.
- Invoice and cost association.
- Historical record preservation and retrieval.
- Primary key, foreign key, unique, and required-field constraints.
- Database trigger execution and automated status updates.
- API and browser interactions with PostgreSQL.
- Invalid or incomplete transaction handling.
- Removal of test data before production deployment.

End-to-end validation confirmed that transactions remain traceable across users, garments, operational events, damage records, and associated financial data.

## Security and Data Protection

The system was designed for controlled use within the laboratory environment, with operational and user data protected through several layers:

- The application is intended for authorized laboratory personnel rather than direct student or guest access.
- PostgreSQL serves as the centralized source of truth, avoiding independent local copies of operational data.
- Database constraints and transactional controls protect relational integrity and reduce inconsistent records.
- Environment variables are used to keep database credentials and configuration values out of source code.
- Sensitive configuration files and local virtual environments are excluded from GitHub through `.gitignore`.
- The source-code repository is maintained privately with controlled access.
- Scheduled database backups provide recovery protection without altering the production database.

Personally identifiable information is limited to the information required for operational garment tracking and accountability.

## Business Impact and Project Outcomes

The system replaces a fragmented, spreadsheet-based inventory process with a centralized operational platform designed around the cleanroom's existing garment workflow.

- **At least 375 administrative hours expected to be saved annually**, based on the lab manager's estimate of at least 1.5 hours saved per workday. These savings result from reducing manual spreadsheet maintenance, garment reconciliation, historical record searches, vendor data matching, invoice reconciliation, and other repetitive record-management tasks.

Beyond administrative efficiency, the system creates a structured financial dataset that was not previously available in a readily analyzable form. Vendor invoices and ruin charges can be connected to individual garments, damage events, users, and historical transactions, allowing management to quantify garment-related spending and identify previously obscured cost drivers.

### Key Outcomes

- **Approximately 375+ administrative hours saved annually**, based on the lab manager's estimate of at least 1.5 hours saved per workday.
- Reduced repetitive spreadsheet maintenance, manual reconciliation, and recurring record management.
- Faster garment and user lookup through searchable barcode and University ID records.
- End-to-end garment traceability across checkout, return, vendor processing, damage, retirement, and historical records.
- Direct linkage between damaged garments, historical checkout activity, responsible users, and associated costs.
- Automated garment status management through database triggers rather than manual record updates.
- Precise financial analysis by garment, damage type, user, and reporting period.
- Vendor invoice and damage reconciliation provides visibility into recurring damage patterns, replacement costs, and potential sources of avoidable spending.
- Preserved historical records support long-term accountability, investigation, operational analysis, and financial reporting.
- Browser-based workflows consolidate previously fragmented operational tasks into a single management interface.
- A centralized data foundation supports Power BI reporting and future integration with other laboratory systems.

The completed system transforms garment tracking from a recurring administrative process into a persistent, queryable operational and financial data system. In addition to reducing administrative workload, it gives management the ability to measure garment-related spending, identify cost drivers, analyze damage and replacement trends, and identify opportunities for future cost reduction.

## Future Development

The system was designed to support continued expansion without requiring changes to the core transactional architecture.

Planned enhancements include:

- **NEMO Integration** — connect with the laboratory's existing NEMO environment to exchange relevant user and operational data and further reduce duplicate data entry.
- **AI Inventory Assistant** — provide an authorized natural-language interface for querying inventory, garment history, damages, costs, and operational metrics conversationally.
- **Expanded Analytics** — extend Power BI reporting to identify long-term inventory utilization, damage, replacement, vendor, and financial trends.
- **Cost Optimization Analytics** — use accumulated historical and invoice data to identify recurring cost drivers, high-cost garment categories, damage patterns, and opportunities to reduce replacement spending.
- **Additional Workflow Automation** — automate additional operational processes as cleanroom requirements evolve.

