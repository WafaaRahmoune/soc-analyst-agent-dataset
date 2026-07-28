# SOC Tier-2 Analyst Agent Dataset

![License](https://img.shields.io/badge/License-CC%20BY%204.0-blue?style=flat-square)
![Instances](https://img.shields.io/badge/Instances-3%2C057-8b5cf6?style=flat-square)
![Format](https://img.shields.io/badge/Format-JSON%20(chat)-34d399?style=flat-square)
![Task](https://img.shields.io/badge/Task-SFT%20%7C%20Tool--calling-orange?style=flat-square)

Supervised fine-tuning (SFT) dataset for training an **autonomous SOC Tier-2 analyst agent** that investigates security alerts inside a **Wazuh + OpenSearch + Suricata + Sysmon** environment.

Each instance is a complete **tool-calling trajectory**: the agent receives a correlated security alert, investigates it end to end through a **4-phase methodology**, and produces a final structured verdict (**false positive** or **true positive**) with concrete containment and remediation actions.

The dataset contains **3,057 instances**, balanced across FP / TP verdicts and spanning two alert-input formats (`chained` and `grouped`).

## Task

Given a correlated Wazuh alert, the agent must:

1. **Phase 1 (Investigation & Enrichment):** read the full alert, extract every IoC (IPs, hashes, domains, accounts, hosts, paths), enrich external indicators via threat intelligence, and decide whether to close as FP or dig deeper.
2. **Phase 2 (Schema Discovery):** discover the exact OpenSearch field paths before hunting.
3. **Phase 3 (Threat Hunting):** translate hunting questions into concrete OpenSearch DSL queries, run them, and gather literal evidence.
4. **Phase 4 (Recommendation):** synthesize findings into a verdict with severity, scope, immediate actions, remediation, and next steps.

The agent reasons inside `<think>...</think>` blocks before every tool call and emits a final JSON verdict.

## Repository structure

```
soc-analyst-agent-dataset/
├── instances/     # 3,057 training instances (one .json trajectory each)  = the dataset
├── AIT_ADS/       # raw AIT-ADS Wazuh alert data (.json.gz per scenario + labels.csv) = source
├── Documents/     # source security playbooks / PDFs (incl. 250 MITRE ATT&CK playbooks) = source
└── README.md
```

- **`instances/`** is the actual dataset you train on.
- **`AIT_ADS/`** and **`Documents/`** are the raw sources the trajectories were derived from, kept for provenance and reproducibility.

## File naming convention

Each instance file name encodes its origin, verdict and format:

- **origin:** `ait_` (derived from AIT-ADS data), `aug_` (augmented / perturbed variant), or `pdf...` (authored from a security playbook / PDF).
- **verdict:** `fp` (false positive) or `tp` (true positive).
- **format:** `grouped` or `chained`.

Example: `ait_fox_batch-0391_fp1_k1_fp_grouped.json` is an AIT-ADS "fox" scenario, false positive, grouped format.

## Alert input formats

- **`grouped`:** the correlated alert is presented as a single grouped block.
- **`chained`:** the alert is presented as a chain of related events.

## Instance format

Each `.json` file follows the OpenAI-style chat schema:

```json
{
  "tools": [ ... ],
  "messages": [
    {"role": "system",    "content": "You are a senior SOC Tier-2 analyst agent..."},
    {"role": "user",      "content": "Investigate this Wazuh alert: { ... }"},
    {"role": "assistant", "content": "<think>...</think>", "tool_calls": [ ... ]},
    {"role": "tool",      "content": "{ ... tool result ... }"},
    {"role": "assistant", "content": "<think>...</think>\n{ \"verdict\": { \"type\": \"...\" }, ... }"}
  ]
}
```

## Data sources

The trajectories come from three origins:

- **Security playbooks (`Documents/`):** scenarios authored from expert SOC playbooks and PDF documents, including a set of 250 MITRE ATT&CK playbooks.
- **AIT-ADS (`AIT_ADS/`):** scenarios derived from the AIT Alert Data Set.
- **Augmented:** paraphrased / perturbed variants (false-positive focused) to improve robustness.

## Composition

| Metric | Count |
|---|---|
| Total instances | **3,057** |
| False positive | 1,572 |
| True positive | 1,485 |
| `chained` format | 1,793 |
| `grouped` format | 1,200 |

## How to use

```python
import json, glob

instances = []
for path in glob.glob("instances/*.json"):
    with open(path, encoding="utf-8") as f:
        instances.append(json.load(f))

print(len(instances), "instances")          # 3057
print(instances[0]["messages"][0]["role"])  # system
```

Each object is a ready-to-use chat trajectory (`messages` + `tools`) that can be fed directly to an SFT / tool-calling fine-tuning pipeline.

## Intended use

This dataset is intended for **research and educational purposes**: fine-tuning and evaluating LLM agents for SOC alert triage, threat hunting, and investigation reasoning.

## Limitations

- Trajectories are generated and augmented, then reviewed against playbooks and AIT-ADS data; they are **not** a substitute for a production SOC and may contain synthetic artifacts.
- Tool results are recorded for the specific `Wazuh + OpenSearch + Suricata + Sysmon` environment and query schema.
- Verdicts reflect the referenced scenarios and should be validated before any operational use.

## License

Released under the **Creative Commons Attribution 4.0 International (CC BY 4.0)** license: you are free to share and adapt the dataset, provided you give appropriate credit.

Part of the data is derived from the **AIT Alert Data Set (AIT-ADS)**, also distributed under CC BY 4.0. Please also credit the original AIT-ADS authors when using the `AIT_ADS`-derived instances.

## How to cite

```bibtex
@misc{rahmoune_soc_analyst_agent_dataset,
  title        = {SOC Tier-2 Analyst Agent Dataset},
  author       = {Rahmoune, Wafaa},
  year         = {2026},
  howpublished = {\url{https://github.com/WafaaRahmoune/soc-analyst-agent-dataset}}
}
```
