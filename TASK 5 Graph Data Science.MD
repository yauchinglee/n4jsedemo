# Task 5: Advanced Network Intelligence (Graph Data Science)

While standard queries identify known patterns, **Graph Data Science (GDS)** allows us to analyze the global structure of the financial network. By treating the entire database as a single mathematical object, we can identify "Hidden Hubs" and "Fraud Syndicates" that are invisible to traditional row-based analysis.

---

## 1. Centrality Analysis (PageRank)
**Business Value:** Identifies "Influence Hubs." In a banking context, an account with an abnormally high PageRank score—yet no legitimate commercial profile—is a high-probability **Money Laundering Hub** or **Main Mule**.

### A. Memory Projection
Before running algorithms, we project the graph into a high-performance in-memory workspace.

```cypher
// Create a virtual projection of the transaction flow
CALL gds.graph.project(
  'banking_risk_projection',
  ['Account'],
  {
    SENT_TRANSFER: { orientation: 'NATURAL' },
    RECEIVE_TRANSFER: { orientation: 'REVERSE' }
  }
);
```

### 2. Executing the Algorithm
We calculate the "Importance" of every account based on the volume and quality of incoming transfers.

```cypher
// Run PageRank and stream the top 10 most influential accounts
CALL gds.pageRank.stream('banking_risk_projection')
YIELD nodeId, score
RETURN 
    gds.util.asNode(nodeId).AccountNumber AS Account, 
    score AS RiskInfluenceScore
ORDER BY RiskInfluenceScore DESC
LIMIT 10;
```

### 3. Community Detection (Louvain)
Business Value: Fraud is a "team sport." The Louvain algorithm identifies clusters of accounts that interact significantly more with each other than the rest of the network, surfacing potential organized crime syndicates.
```cypher
// Identify isolated communities of high-frequency interaction
CALL gds.louvain.stream('banking_risk_projection')
YIELD nodeId, communityId
RETURN 
    gds.util.asNode(nodeId).AccountNumber AS Account, 
    communityId
ORDER BY communityId ASC;
```

### 4. Writing Scores to the Graph
To make these mathematical scores actionable for investigators, we write the results back to the database and use Neo4j Bloom to create a visual risk heatmap.
```cypher
CALL gds.pageRank.write('banking_risk_projection', {
  writeProperty: 'centralityScore'
});
```

