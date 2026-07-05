# ThreatGraphX 🕸️🔐

**Explainable Cyber Threat Intelligence Using Knowledge Graphs and Graph Neural Networks**

ThreatGraphX unifies four major cybersecurity knowledge sources — **NVD CVE, MITRE CWE, CAPEC, and ATT&CK** — into a single heterogeneous knowledge graph, then uses Graph Neural Networks to predict missing threat relationships and explain *why* a vulnerability maps to a given attack technique and mitigation.

Instead of manually chasing a vulnerability through four disconnected databases, ThreatGraphX walks the full chain automatically:

```
CVE  →  CWE  →  CAPEC  →  ATT&CK Technique  →  Mitigation
(vulnerability) (weakness) (attack pattern) (adversary behavior) (defense)
```

---

## ✨ Key Features

- **Unified Knowledge Graph** — 7,502 nodes, 15,573 edges, 5 relation types built from raw CVE/CWE/CAPEC/ATT&CK feeds
- **Link Prediction with GNNs** — GraphSAGE, R-GCN, and a custom Transformer-enhanced model (**ThreatGraphX-Tx**)
- **Zero-Shot Evaluation** — tests how well models predict relationships for CVEs never seen during training, mimicking real-world newly disclosed vulnerabilities
- **Two-Phase (TxGNN-style) Training** — pretrain on the full graph, then fine-tune specifically on CVE-relevant edges for stronger zero-shot generalization
- **Explainable Output** — every prediction comes with a traceable multi-hop path from vulnerability to mitigation, not just a black-box score

---

## 📊 Results Snapshot

**Standard Evaluation (transductive)**

| Model | Accuracy | F1 | AUC-ROC |
|---|---|---|---|
| GraphSAGE | 90.65% | 0.9129 | 0.9527 |
| R-GCN | **92.73%** | 0.9306 | **0.9735** |
| ThreatGraphX-Tx | 92.73% | **0.9316** | 0.9716 |

**Strict Zero-Shot Evaluation (unseen CVE nodes)**

| Model | Accuracy | F1 | AUC-ROC |
|---|---|---|---|
| GraphSAGE | 76.06% | 0.7149 | 0.8781 |
| R-GCN | 78.23% | 0.7498 | 0.8995 |
| **ThreatGraphX-Tx (two-phase)** | **92.02%** | **0.9208** | **0.9739** |

Translational baselines (TransE, DistMult) were also tested for comparison but performed poorly (Hits@10 ≈ 0), confirming that relation-aware GNNs are necessary for this heterogeneous graph.

---

## 🏗️ Pipeline Overview

The project is built as a 20-step pipeline:

1. **Data Inspection** — validate raw CVE/CWE/CAPEC/ATT&CK files
2. **Entity Extraction** — parse CVE, CWE, CAPEC, ATT&CK techniques & mitigations, software/products
3. **Edge Construction** — build has_weakness, mapped_to, maps_to, mitigated_by, affects relations
4. **Knowledge Graph Construction** — merge into a single graph with node2id/relation2id mappings
5. **Graph Visualization** — inspect relation and node-type distributions
6. **Data Splitting** — 70/15/15 train/val/test split, converted to (head, relation, tail) triples
7. **Baseline Embeddings** — TransE and DistMult (PyKEEN)
8. **GNN Models** — GraphSAGE, R-GCN, and ThreatGraphX-Tx (Transformer-enhanced)
9. **Zero-Shot & Strict Zero-Shot Evaluation** — including two-phase fine-tuning
10. **Explainability Module** — multi-hop threat path generation for any CVE

---

## 📁 Datasets Used

| Source | Format | Role |
|---|---|---|
| [NVD CVE](https://nvd.nist.gov/vuln/data-feeds) | JSON (NVD 2.0) | Vulnerability nodes |
| [MITRE CWE](https://cwe.mitre.org/data/downloads.html) | XML (CWE v7.3) | Weakness category nodes |
| [MITRE CAPEC](https://capec.mitre.org/data/downloads.html) | XML (CAPEC v3.5) | Attack pattern nodes |
| [MITRE ATT&CK](https://attack.mitre.org) | STIX 2.1 JSON | Technique & mitigation nodes |

---

## 🧠 Models

| Model | Type | Highlight |
|---|---|---|
| TransE / DistMult | Translational KG embedding | Baselines — insufficient for this heterogeneous graph |
| GraphSAGE | Neighborhood aggregation GNN | Strong recall, weaker precision |
| R-GCN | Relation-aware GNN | Best standard-evaluation performance |
| **ThreatGraphX-Tx** | R-GCN embeddings + Transformer encoder | Best zero-shot generalization to unseen CVEs |

---

## 🔍 Explainability Example

For `CVE-2026-12212`, ThreatGraphX traces the following threat path:

```
CVE-2026-12212 → CWE-266 (Incorrect Privilege Assignment)
              → CAPEC-19 (Embedding Scripts within Scripts)
              → T1037 (Boot or Logon Initialization Scripts)
              → Mitigation: Execution Prevention
```

This lets an analyst see **why** a technique is relevant — not just a confidence score.

---

## 🛠️ Tech Stack

- Python, PyTorch, PyTorch Geometric
- PyKEEN (TransE, DistMult baselines)
- NetworkX (graph construction & visualization)
- Pandas / NumPy / scikit-learn

---

## 📂 Repository Structure

```
data/
 ├── raw/                # Original CVE, CWE, CAPEC, ATT&CK source files
 ├── processed/           # Extracted node & edge CSVs
 └── splits/               # Train/val/test edge & triple files
notebooks/               # Step-by-step pipeline notebooks (Steps 01–20)
results/                    # Model metrics, threat discovery tables
```

---

## 📚 References

1. Usama, M., Nagaty, K. and Elmaghawry, N., 2025. *Construction of a Unified Knowledge Graph for Cyber Threat Intelligence.* ICCA 2025.
2. Liang, K., Liu, Y., Zhou, S., 2024. *Knowledge graph contrastive learning based on relation-symmetrical structure.* IEEE TKDE.
3. Ji, S., Pan, S., Cambria, E., Marttinen, P., Yu, P.S., 2021. *A survey on knowledge graphs.* IEEE TNNLS.
4. Alsaheel, S. et al., 2021. *ATT&CK-KG: A knowledge graph of adversary techniques and tactics.* IEEE Big Data.

---

## 👤 Author

**Abdu Samad Shah**
Submitted to Dr. Muhammad Ishaq (Assistant Professor), IM Sciences
