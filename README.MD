## 1. Project Overview (Task 1)
This project transforms a relational banking dataset into a high-fidelity Knowledge Graph using Neo4j. By focusing on the **relationships** between customers and their digital footprints, we unlock advanced fraud detection capabilities that are difficult to achieve in traditional SQL environments.

### Key Entities & Relationships
* **Identity Hubs:** Customers are linked to discrete `Address`, `Phone`, and `Email` nodes. This allows for **Identity Resolution**—finding multiple accounts controlled by the same real-world entity.
* **Financial Flow:** `Accounts` and `Cards` act as the source of transactions. The model captures both `Purchases` (retail spending) and `Transfers` (P2P movements).
* **Analytical Use Cases:**
    * **Anti-Money Laundering (AML):** Detecting circular transfer loops.
    * **First-Party Fraud:** Identifying synthetic identity rings via shared contact points.
    * **Customer 360:** Using PageRank to identify high-value "influencer" accounts in the network.

---

## 2. Graph Data Model (Task 2)

### Schema Visualization
The model was designed in **Arrows.app** to ensure a clean, performant graph structure. 

![Banking Graph Data Model](assets/Task1_ArrowApp.png)

* **Identity Nodes:** * **Customer:** Individual bank clients (CIF, Age, Job Title).
  * **Address / Email / Phone:** Discrete contact points used to identify shared-identity clusters.
* **Financial Nodes:**
  * **Account:** The primary repository for funds.
  * **Card:** The payment instrument used for retail spending.
* **Activity Nodes:**
  * **Transfer:** Peer-to-peer fund movements between accounts.
  * **Purchase:** Retail spending events at point-of-sale.
  * **Merchant:** The commercial destination for purchases.

* **Ownership & Usage:** `(Customer)-[:HAS_ACCOUNT]->(Account)` and `(Customer)-[:HAS_CARD]->(Card)`.
* **The Flow of Value:** `(Account)-[:SENT_TRANSFER]->(Transfer)-[:RECEIVE_TRANSFER]->(Account)`. This structure allows for precise tracing of fund origins and destinations.
* **Retail Behavior:** `(Card)-[:CARD_PURCHASE]->(Purchase)-[:TO_MERCHANT]->(Merchant)`.
* **Identity Proximity:** `(Customer)-[:HAS_ADDRESS]->(Address)`.

---

### Ingestion Logic
The data ingestion is handled via optimized Cypher scripts using `LOAD CSV`. Key design choices include:
* **Constraint Enforcement:** Unique constraints are applied to `CIF`, `AccountNumber`, and `TransactionID` to ensure data integrity and idempotent loads.
* **Property Promotion:** Shared attributes like `Address` and `PhoneNumber` are promoted to Nodes to enable high-speed "Commonality" queries.

### Repository Artifacts
* **Model Schema:** `Task1_ArrowApp.png`
* **Import Script:** `ingestion_script.cypher`
* **Database Dump:** `dbdump.dump` (For full reproduction of nodes, relationships, and GDS results).

---

