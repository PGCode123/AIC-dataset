# Concrete Crack Knowledge Graph Dataset

This repository provides a knowledge graph dataset for concrete crack assessment, deterioration mechanism analysis, and maintenance decision support. The knowledge graph integrates information related to materials, environmental conditions, crack characteristics, structural properties, deterioration effects, and maintenance strategies.

The dataset is represented using a graph-based structure and can be directly imported into Neo4j for visualization and Cypher-based querying.

## Knowledge Graph Schema

The knowledge graph contains the following major node types:

- `Material` – concrete materials, reinforcement, fibers, and other material constituents
- `Product` – material and structural products
- `Condition` – environmental, exposure, loading, and service conditions
- `Parameter` – measurable parameters such as crack width
- `Crack` – crack types and characteristics
- `Effect` – deterioration mechanisms and structural consequences
- `Property` – structural or durability-related properties and limits
- `Strategy` – inspection, protection, repair, and maintenance strategies

Relationships connect these entities to represent engineering knowledge, deterioration mechanisms, and maintenance actions.

## Usage

The dataset can be imported into a Neo4j database and queried using Cypher.

The following examples demonstrate three typical applications of the knowledge graph under a marine exposure scenario.

---

## Example 1: Crack-Width Compliance Assessment

This example compares a measured crack width with the allowable crack-width limit under seawater spray and cyclic wet-dry exposure.

```cypher
MATCH (c:Condition {name:"Seawater Spray and Cyclic Wet-Dry"})
MATCH (c)-[:CAUSES]->(p:Property)
MATCH (p)-[:DETERMINES]->(limit:Property)

WITH limit, 0.35 AS measured_value

RETURN
    measured_value AS measured_crack_width,
    limit.value AS allowable_limit,
    CASE
        WHEN measured_value <= limit.value
        THEN "Compliant"
        ELSE "Non-Compliant"
    END AS assessment
```

### Expected Output

| measured_crack_width | allowable_limit | assessment |
|---:|---:|---|
| 0.35 | 0.15 | Non-Compliant |

---

## Example 2: Crack Repair Recommendation

This example retrieves crack-level maintenance strategies based on the measured crack width and the allowable limit.

```cypher
MATCH (c:Condition {name:"Seawater Spray and Cyclic Wet-Dry"})
MATCH (c)-[:CAUSES]->(p:Property)
MATCH (p)-[:DETERMINES]->(limit:Property)

MATCH (cw:Parameter {name:"Crack Width"})
OPTIONAL MATCH (cw)-[:CONTROLLED_BY|PROTECTED_BY|MONITORED_BY]->(s:Strategy)

WITH limit,
     collect(DISTINCT s.name) AS repair_methods,
     0.35 AS measured_value

RETURN
    measured_value AS measured_crack_width,
    limit.value AS allowable_limit,
    CASE
        WHEN measured_value <= limit.value
        THEN ["Periodic Inspection"]
        ELSE repair_methods
    END AS crack_repair_recommendation
```

### Expected Output

| measured_crack_width | allowable_limit | crack_repair_recommendation |
|---:|---:|---|
| 0.35 | 0.15 | Epoxy Injection; Surface Protective Coating |

---

## Example 3: Mechanism-Informed Maintenance Recommendation

This example traces deterioration mechanisms associated with crack width and retrieves both crack-level and structural-level maintenance strategies.

```cypher
MATCH (e:Condition)
MATCH (e)-[:INCLUDES]->(c:Condition {name:"Seawater Spray and Cyclic Wet-Dry"})
MATCH (c)-[:CAUSES]->(p:Property)
MATCH (p)-[:DETERMINES]->(limit:Property)

MATCH (cw:Parameter {name:"Crack Width"})
MATCH (cw)-[:CAUSES|LEADS_TO|RESULTS_IN*1..4]->(m:Effect)

OPTIONAL MATCH (m)-[:MITIGATED_BY]->(s_struct:Strategy)
OPTIONAL MATCH (cw)-[:CONTROLLED_BY|PROTECTED_BY|MONITORED_BY]->(s_crack:Strategy)

WITH e, limit,
     collect(DISTINCT m.name) AS mechanisms,
     collect(DISTINCT s_struct.name) AS structural_repair,
     collect(DISTINCT s_crack.name) AS crack_repair,
     0.35 AS measured_value

RETURN
    e.name AS environment,
    measured_value AS measured_crack_width,
    limit.value AS allowable_limit,
    mechanisms,
    crack_repair,
    structural_repair
```

### Expected Output

| Field | Result |
|---|---|
| Environment | Marine Environment |
| Measured crack width | 0.35 mm |
| Allowable limit | 0.15 mm |
| Mechanisms | Chloride Ingress; Steel Depassivation; Electrochemical Corrosion; Rebar Corrosion; Crack Propagation |
| Crack repair | Epoxy Injection; Surface Protective Coating |
| Structural repair | Cathodic Protection |

---


## Requirements

- Neo4j
- Cypher Query Language
