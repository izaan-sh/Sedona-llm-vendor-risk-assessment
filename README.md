# 🛡️ Sedona: LLM-Assisted Vendor Risk Assessment

**Platform:** Air-gapped LLM + RAG (Gemma 3 4B, ChromaDB, FastAPI, React)  
**Author:** Izaan Shumaiz | Cybersecurity & AI Graduate  
**Context:** ICT302 IT Professional Practice Project (Capstone), Murdoch University  
**Team:** Team Aegis (6 students) | **Client:** Murdoch University IT Department  

![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=flat&logo=fastapi&logoColor=white)
![React](https://img.shields.io/badge/React-20232A?style=flat&logo=react&logoColor=61DAFB)
![Vite](https://img.shields.io/badge/Vite-646CFF?style=flat&logo=vite&logoColor=white)
![Ollama](https://img.shields.io/badge/Ollama-000000?style=flat&logo=ollama&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=flat&logo=ubuntu&logoColor=white)

> **Note:** This repo is a portfolio write-up of the project and my part in it. Murdoch policies, internal risk framework material, vendor data, and infrastructure details have been removed.

---

## 📌 Executive Summary

Sedona is a prototype platform that helps IT and security staff review vendor security questionnaires (**HECVAT**) against a university's internal security policies. It runs fully offline on a locally hosted LLM, so no vendor or policy data ever leaves the environment.

The project goes beyond a simple chatbot. It combines retrieval, a structured evaluation method, and risk scoring to turn a 267-control questionnaire into a traceable, reviewable risk assessment. Key achievements include:

- Automating review of a **267-control, 25-section** HECVAT against internal security policies.
- Designing a **two-layer evaluation method** (vendor answer vs. HECVAT expectation, and policy alignment via RAG).
- Running a **dual-context RAG pipeline** on **ChromaDB** over the policy knowledge base.
- Deploying a **fully air-gapped** stack on a university-provisioned Ubuntu GPU VM with **no external API calls**.
- Scoring risk using the university's **Risk Management Framework (RMF)**.
- Generating **timestamped reports** with traceability from each risk back to its source.

---

## 🎯 The Problem

Reviewing a HECVAT is slow and inconsistent. Each answer has to be checked against internal policy before a risk rating can be assigned, results depend on the individual reviewer, and vendor answers are often vague or incomplete. Sending that material to a hosted LLM would also create a data handling risk of its own, which defeats the purpose of a risk tool.

---

## 📐 Architecture & Environment

The whole system runs inside a single university-provisioned VM boundary.

| Layer | Role / Specifications |
| :--- | :--- |
| **LLM Engine** | Gemma 3 4B via **llama-cpp-python** (GPU, primary), **Ollama** (CPU fallback, config toggle) |
| **Retrieval** | **ChromaDB** dual-context retrieval over the internal policy knowledge base |
| **Scoring** | RMF scoring module (likelihood x impact) |
| **Backend** | FastAPI (Python) with session management and follow-up Q&A |
| **Frontend** | React + Vite web interface |
| **Reporting** | Timestamped reports in PDF, Excel and JSON |
| **Deployment** | Ubuntu VM, T4-class GPU, 64 GB RAM, fully offline |

<p align="center">
  <img src="docs/architecture.png" alt="Sedona architecture" width="900">
  <br><em>Figure 1: Sedona system architecture inside the VM production boundary.</em>
</p>

**Flow:** the assessor uploads a HECVAT through the interface, which is parsed and matched against policy context in ChromaDB. The local LLM evaluates each control, the RMF scoring module rates the risk, and the reporting service produces timestamped reports.

### 🔒 Why air-gapped?

Vendor security submissions and internal policies are sensitive. Running the model locally removes the exposure that a hosted API would create. This was a deliberate design decision, not a limitation we fell back on.

---

## 🧠 The Two-Layer Evaluation

Each HECVAT control is assessed in two layers, handled in a **single LLM inference pass** rather than two separate calls. This keeps the pipeline simpler and faster on a small local model.

| Layer | Question it answers | How |
| :--- | :--- | :--- |
| **Layer 1** | Does the vendor's answer meet what the HECVAT considers a compliant response? | Direct comparison against the expected HECVAT answer |
| **Layer 2** | Does the answer align with the university's own security policies? | Relevant policy text retrieved from ChromaDB and given to the model as context |

---

## 🚀 What Sedona Does

### Phase 1: Ingestion & Parsing
- Accepts a completed HECVAT and extracts the key security and privacy answers.
- Vendor submissions are **processed at runtime only and not persisted**.

### Phase 2: Evaluation
- Runs the two-layer evaluation on every applicable control.
- Flags gaps and unclear answers, then asks the assessor follow-up questions.

### Phase 3: Risk Scoring
- Rates risks using the RMF (likelihood and impact) and maps them to rating bands.

### Phase 4: Reporting
- Produces a written risk assessment with traceability from each risk statement to the source answer and policy.
- Exports timestamped reports as **PDF, Excel and JSON**.

---

## 🧩 Handling Coverage Gaps

Not every control can be assessed the same way. Sedona handles these cases explicitly instead of letting the model improvise:

| Situation | Example | How Sedona handles it |
| :--- | :--- | :--- |
| **Policy-covered control** | Most security controls | Full two-layer evaluation with policy context |
| **No matching policy** | ~18% of controls, mainly AI and IT accessibility sections | Retrieval returns nothing useful, so the control is flagged as a gap for human review |
| **Conditional section** | HIPAA, PCI DSS, on-premises | Gated by earlier vendor answers and treated as N/A when not applicable |
| **Vague vendor answer** | Unclear or incomplete response | Follow-up question generated for the assessor |

---

## 👤 My Role: Security and Compliance Engineer

I was one of six on the team. My contributions:

- **HECVAT evaluation framework:** designed the two-layer assessment method the pipeline is built around.
- **Risk management:** wrote the risk management section of the project plan and the glossary.
- **Architecture and infrastructure design:** wrote the architecture and infrastructure section of the design document.
- **Security audit:** carried out a security review of the project VM.
- **Documentation and communication:** wrote the executive summary, contributed to the requirements and analysis document, and built parts of the final presentation.

---

## ⚖️ Design Decisions

1. **Dual-context retrieval.** The model sees the vendor's answer and the relevant policy text together, so it can judge alignment directly instead of guessing what the policy says.
2. **Small local model.** Gemma 3 4B is far less capable than large hosted models, but it fits the offline requirement and the VM hardware.
3. **llama-cpp-python as primary, Ollama as fallback.** GPU inference is the normal path, with a config toggle for CPU when needed. They are not interchangeable in performance.
4. **Human in the loop.** Outputs are drafts for reviewers, not decisions.

---

## 🔐 Ethical, Privacy & Reliability Considerations

- No sensitive data leaves the host, by design.
- Vendor HECVAT submissions are runtime only and not stored. Only the internal policy knowledge base is persistent.
- LLM hallucination is the main reliability concern, so every risk statement needs human review.
- Traceability lets reviewers check where each claim came from.

---

## 🧰 Skills & Technologies Demonstrated

- **GRC & Risk Assessment:** HECVAT analysis, RMF-based risk scoring, control-to-policy mapping.
- **LLM Engineering:** local model deployment, prompt design, single-pass multi-criteria evaluation.
- **RAG Systems:** ChromaDB vector retrieval, dual-context pipeline design.
- **Secure Architecture:** air-gapped design, data minimisation (runtime-only vendor data), VM security review.
- **Full-Stack Prototyping:** FastAPI backend, React + Vite interface, report generation.
- **Professional Practice:** working with a real client and a six-person team, technical documentation.

---

## ⚠️ Limitations

- HECVAT only covers so much. It cannot show a vendor's real security posture, so Sedona flags gaps rather than filling them in.
- A 4B model has limited reasoning depth compared to larger hosted models.
- Controls without matching internal policy cannot be meaningfully assessed through retrieval.
- Output quality depends on the quality and completeness of the vendor's answers.

---

## 💡 What I Learned

- Evaluation design has to reflect what retrieval can and cannot actually cover. That matters more than prompt tweaking.
- Air-gapped deployment is a real trade-off: you gain data control and give up model capability.
- Security assurance is mostly about gaps and ambiguity, not clean yes/no answers.
- In a team with a real client, figures and claims must be verified before they go into a deliverable, because unchecked numbers cause rework.

---

## 🎯 Possible Next Steps

- [ ] Add policy coverage for the AI and IT accessibility sections.
- [ ] Test larger local models as hardware allows.
- [ ] Add reviewer feedback capture to improve follow-up questions.

---

## 🙏 Acknowledgements

Team Aegis, Murdoch University IT (client), and our project supervisor.

*Built for ICT302, Murdoch University.*
