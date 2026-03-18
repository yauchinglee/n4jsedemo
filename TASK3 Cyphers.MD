Analytical Queries & Insights (Task 3)

These queries demonstrate the graph's ability to extract actionable intelligence from connected data.

### A. Money Laundering (Circular Transfers)

Identifies "loops" where money returns to the source account after passing through one or more intermediate accounts to obscure the trail.


```cypher
// Look for a closed loop of transfers between two distinct accounts
MATCH path = (a:Account)-[:SENT_TRANSFER]->(t1:Transfer)-[:RECEIVE_TRANSFER]->(b:Account)-[:SENT_TRANSFER]->(t2:Transfer)-[:RECEIVE_TRANSFER]->(a)
// Ensure we aren't just looking at an account sending money to itself
WHERE a <> b
// Return the specific flow of funds to visualize the 'loop'
RETURN a.AccountNumber AS SourceAccount, b.AccountNumber AS Intermediate, t1.Amount AS Out, t2.Amount AS In;
```

![Run Results](assets/Task3_Cypher1.png)

### B. Social Engineering / Mule Detection (High Velocity)

Identifies accounts that receive a high volume of transfers from multiple unique sources within a short window, suggesting a "money mule" collection point.

Detects multiple distinct customers sharing the same physical address or phone—a primary indicator of synthetic identity "fraud factories."

```cypher
// Traverse from multiple source accounts through their transfers to a single target
MATCH (source:Account)-[:SENT_TRANSFER]->(t:Transfer)-[:RECEIVE_TRANSFER]->(target:Account)
// Aggregate data by the target account to count unique incoming connections
WITH target, count(DISTINCT source) AS uniqueSenders, sum(t.Amount) AS totalReceived
// Filter for 'fan-in' patterns (more than 3 unique senders)
WHERE uniqueSenders > 3
// Sort by the highest number of senders to prioritize investigations
RETURN target.AccountNumber AS MuleCandidate, uniqueSenders, totalReceived
ORDER BY uniqueSenders DESC;
```

![Run Results](assets/Task3_Cypher2.png)

### C. High-Risk Merchant Identification

Business Value: Identifies merchants with unusually high transaction volumes or specific patterns of card spending, helping the bank flag potential compromised Point-of-Sale (POS) locations.

```cypher
// Identify the concentration of purchase activity at specific merchant nodes
MATCH (m:Merchant)<-[:TO_MERCHANT]-(p:Purchase)
// Calculate the volume and frequency of transactions per merchant
RETURN 
    m.Merchant AS MerchantName, 
    count(p) AS TransactionCount, 
    sum(p.Amount) AS TotalVolume
// Order by volume to see where the highest financial risk/exposure lies
ORDER BY TotalVolume DESC 
LIMIT 10;
```


![Run Results](assets/Task3_Cypher3.png)

### D. Direct Account-to-Account Pathfinding

Business Value: Visualizes the direct relationships between accounts, ignoring the "Transfer" node middleman to provide a cleaner view for investigators.

```cypher
// Match the standard transfer chain: Source -> Transfer Node -> Target
MATCH (a1:Account)-[:SENT_TRANSFER]->(t:Transfer)-[:RECEIVE_TRANSFER]->(a2:Account)
// Return the logical connection between the two end-nodes
// This abstracts away the internal Transfer node for a cleaner UI view
RETURN a1.AccountNumber, a2.AccountNumber, t.Amount, t.TransferDatetime AS Timestamp
LIMIT 50;
```

![Run Results](assets/Task3_Cypher4.png)