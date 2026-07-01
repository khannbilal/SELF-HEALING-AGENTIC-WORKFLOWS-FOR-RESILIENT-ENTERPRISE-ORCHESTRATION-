# Self-Healing Agentic Workflows for Resilient Enterprise Orchestration

## 1. Overview

Enterprise AI agents frequently suffer from operational brittleness due to sudden changes in external API schemas (e.g., vendor updates). When an upstream data payload layout shifts, standard hardcoded or naive agentic integrations encounter unhandled exceptions and crash. 

This project introduces a dual-system architecture designed to mitigate this issue. By pairing a research-focused **Counterfactual Probing loop** with an industry-grade production runtime traced via **OpenTelemetry**, the system dynamically diagnoses tool failures, determines root causes, and synthesizes runtime Python adapter code to achieve automated recovery without human intervention.

---

## 2. Framework

The framework operates as a multi-agent orchestration layer built using the `CrewAI` platform and executing locally via `Ollama` (`Llama 3.2`). The orchestration is defined by three distinct operational roles:

* **Counterfactual Researcher:** A diagnostic agent responsible for initial tool execution and trace log analysis to isolate the origin of execution errors.
* **Adapter Coder:** An integration engineer capable of inspecting raw payloads and generating real-time, temporary Python dictionary mappings (`patch_schema`) to bypass schema mutations.
* **Logic Reviewer:** A quality assurance gatekeeper tasked with validating and verifying that the generated patch code correctly re-maps mutated data structures back to the original expected format.

---

## 3. Scope

The scope of this implementation encompasses:

* The configuration of a localized environment utilizing an asynchronous agentic architecture running a local LLM server (`llama3.2`).
* The design of a counterfactual validation loop testing varying prompts to isolate model hallucinations from definitive external API changes.
* The creation of a self-patching runtime engine capable of generating and executing python-based adapter functions.
* Production-grade instrumentation using the **OpenTelemetry API/SDK** to output tracing data directly to console spans.
* Empirical benchmarking comparing the autonomous resilience of the sentinel setup against standard, non-healing agent variations.

---

## 4. Dataset

The environment integrates and stages two key datasets within the workspace environment:

1. **Berkeley Function Calling Leaderboard (BFCL) Dataset:** Staged locally from `gorilla-llm/Berkeley-Function-Calling-Leaderboard` to evaluate function calling capabilities.
2. **Tau2 Bench Stream:** Integrated via the Hugging Face dataset stream `HuggingFaceH4/tau2-bench-data` for agentic benchmark tracking.

---

## 5. Methodology

The execution methodology follows a strict sequential pipeline divided into distinct diagnostic and mitigation phases:

1. **Environment Preparation:** Local deployment of dependencies, downloading datasets, and initializing a local Ollama server hosting the `llama3.2` model weight collection.
2. **Initial Invocation & Failure Catching:** The *Counterfactual Researcher* executes a data fetch command against a mock API endpoint where the JSON schema key has been mutated (e.g., changing `"price"` to `"cost_usd"`). The resulting `KeyError` is captured.
3. **Counterfactual Ablation Testing:** The agent re-runs the failed step across three distinct prompt variations (*"Fetch data for record 'REC-101'."*, *"Query data for 'REC-101'."*, *"Extract price for 'REC-101'."*). If all three variations return an identical schema execution error, the system confirms an external schema change with high fidelity, ruling out a random model hallucination.
4. **Schema Inspection and Code Generation:** Upon confirmation, the *Adapter Coder* invokes a secondary `inspect_schema_tool` to pull the raw, mutated payload string. It evaluates the structural changes and auto-generates a specific Python function named `patch_schema(raw_dict)`.
5. **Review and Verification:** The *Logic Reviewer* validates the function string to ensure it correctly pops the mutated key and re-assigns the target value back to the expected key without injecting placeholders or hallucinated data.
6. **Observability Logging:** The entire process is wrapped inside OpenTelemetry context managers, producing explicit spans (`Agentic_Healing_Workflow`, `CrewAI_Agent_Execution`, and `Validation_And_Patching`) to record events and execution states.

---

## 6. Project Architecture Diagram

        [User / Client Request]
        │
        ▼
        [OpenTelemetry Traced Workflow Span]
        │
        ├─► [Tool Execution Layer] ──► (API Schema Change: 'price' -> 'cost_usd') ──► [KeyError Raised]
        │
        ├─► [Counterfactual Probing Loop]
        │         ├── Run Variation 1: Fetch Data ──► [Fails]
        │         ├── Run Variation 2: Query Data ──► [Fails]
        │         └── Run Variation 3: Extract Price ──► [Fails]
        │                │
        │                └─► [All Failures Match] ──► Confirm: High Reasoning Trace Mutation
        │
        ├─► [Multi-Agent Healing Pipeline (Sequential Process)]
        │         ├── Counterfactual Researcher ──► Reports Failure State
        │         └── Adapter Coder ──► Inspects Raw API Payload ──► Generates 'patch_schema()'
        │         └── Logic Reviewer ──► Validates Python Adapter Code & Deploys
        │
        ▼
        [Traced Pipeline Successfully Patched] ──► Output Cleaned JSON ──► [Workflow Status: OK]


---

## 7. Results

The performance of the self-healing workflow was evaluated through a randomized benchmark over 20 distinct trial interactions. The quantitative metrics recorded are detailed below:

### Evaluation Metrics
* **Total Benchmark Trials:** 20
* **Standard Agent Autonomous Recovery Rate (ARR):** 10.0%
* **Sentinel Agent Autonomous Recovery Rate (ARR):** 95.0%

### Comparative Performance Table

| Metric | Standard Agent Baseline | Sentinel Self-Healing Agent | Performance Delta |
| :--- | :---: | :---: | :---: |
| **Autonomous Recovery Rate (ARR)** | 10.0% | 95.0% | **8.5x Improvement** |
| **Error Handling Capability** | Crashes on schema mutation | Real-time adaptive Python patching | *Adaptive* |
| **Observability Output** | Standard unhandled stack trace | Structured OpenTelemetry spans | *Traced* |

>  **Key Outcome:** The Sentinel configuration successfully recovered from randomized keys, nested objects, and modified data structures, resulting in a **95% task completion rate** compared to only 10% for the baseline configuration.

---

## 8. Conclusion

The implementation demonstrates that enterprise-level tool brittleness can be effectively mitigated through multi-agent diagnostic orchestration. Utilizing counterfactual ablation effectively isolates external environment changes from internal model fluctuations, achieving a high reasoning trace fidelity. Integrating real-time patch generation with rigorous multi-agent validation allows the system to overcome dead-end API errors without requiring human debugging intervention, resulting in an 8.5x performance boost over conventional baseline architectures.

---

## 9. Future Work

* **Deep Structural Transformations:** Expanding the testing framework to support deeply nested object transformations and multi-layer structural alterations across a broader set of enterprise tools.
* **Model Optimization:** Evaluating alternative open-source model weights via the local Ollama backend to optimize reasoning trace latency during the ablation loop phase.
* **Patch Caching Layer:** Implementing persistent runtime caching mechanisms for successfully compiled Python adapter code blocks to skip iterative patching cycles on recurrent broken endpoint requests.

---

## 10. References

* **Berkeley Function Calling Leaderboard (BFCL):** Gorilla LLM Project, Dataset Repository.
* **Tau2 Bench Data Stream:** Hugging Face H4 Dataset Collection.
* **CrewAI Orchestration Framework:** Multi-Agent Sequencing Library.
* **OpenTelemetry Standard:** Cloud Native Computing Foundation Observability Specifications
