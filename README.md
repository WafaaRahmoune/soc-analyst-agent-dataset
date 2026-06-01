# SOC Tier-2 Analyst Agent Dataset
Supervised fine-tuning (SFT) dataset for an **autonomous SOC Tier-2 analyst agent** operating inside a **Wazuh + OpenSearch + Suricata + Sysmon** environment.

Each instance is a complete tool-calling trajectory in which the agent receives a correlated security alert, investigates it end-to-end through a 4-phase methodology, and produces a final structured verdict (**false positive** or **true positive**) with concrete containment and remediation actions.

The dataset contains **3,057 instances** spanning two alert-input formats (`chained` and `grouped`) and balanced across FP / TP verdicts.

---

## Task

Given a correlated Wazuh alert, the agent must:

1. **Phase 1: nvestigation & Enrichment**: read the full alert, extract every IoC (IPs, hashes, domains, accounts, hosts, paths), enrich external indicators via threat intelligence, and decide whether to close as FP or dig deeper.
2. **Phase 2: Schema Discovery**: discover the exact OpenSearch field paths before hunting.
3. **Phase 3: Threat Hunting**: translate hunting questions into concrete OpenSearch DSL queries, run them, and gather literal evidence.
4. **Phase 4: Recommendation**: synthesize findings into a verdict with severity, scope, immediate actions, remediation, and next steps.

The agent reasons in `<think>...</think>` blocks before every tool call and emits a final JSON verdict.

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

- **`PDFs`**: scenarios hand-authored by a human SOC expert from security playbooks / PDF documents.
- **`AIT_ADS`**:scenarios derived from the AIT-ADS alert data.
- **`Augmented`**: augmented trajectories (paraphrased / perturbed variants, false-positive focused).

## Composition

- Verdicts: **1,572 false positive** / **1,485 true positive**.
