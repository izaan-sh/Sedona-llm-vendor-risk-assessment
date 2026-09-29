# Sedona: LLM-Assisted Vendor Risk Assessment

Sedona is a prototype platform that helps IT and security staff review vendor security questionnaires (HECVAT) against a university's internal security policies. It runs fully offline on a local LLM, so no vendor or policy data leaves the environment.

Built by Team Aegis (six students) for Murdoch University's IT Department as the capstone project for ICT302 (IT Professional Practice Project).

> **Note:** This repo is a portfolio write-up of the project and my part in it. Murdoch policies, internal risk framework material, vendor data, and infrastructure details have been removed.
---

## The problem

Reviewing a HECVAT is slow. There are 267 controls across 25 sections, and each answer has to be checked against internal policy before a risk rating can be assigned. Reviews depend heavily on individual reviewers, and vendor answers are often vague or incomplete.

## What Sedona does

- Ingests a completed HECVAT and extracts the key security and privacy answers
- Checks each answer against the university's security policies using retrieval (RAG)
- Flags gaps and unclear answers, then asks the user follow-up questions
- Rates risks using the university's Risk Management Framework (likelihood and impact)
- Generates a written risk assessment as PDF and a PowerPoint summary for decision makers
- Keeps traceability from each risk statement back to the source answer and policy

## Architecture

<img width="1521" height="662" alt="Screenshot 2026-09-29 221438" src="https://github.com/user-attachments/assets/ac043ffb-658c-4af6-ace2-ace78bb40152" />


| Layer | Tech |
|---|---|
| LLM | Gemma 3 4B via llama-cpp-python (Ollama as CPU fallback) |
| Retrieval | ChromaDB, dual-context RAG (vendor answers and policy text) |
| Backend | FastAPI (Python) |
| Frontend | React and Vite |
| Reports | PDF and PowerPoint generation |
| Deployment | Single Ubuntu VM with GPU, no external API calls |

### Why air-gapped?

Vendor security submissions and internal policies are sensitive. Sending them to a hosted LLM API would create a data handling risk of its own, which is a bit ironic for a tool that exists to assess risk. Running the model locally removes that exposure entirely. We chose this deliberately, it wasn't a limitation we fell back on.

## My role: Security and Compliance Engineer

I was one of six on the team. My contributions:

- **HECVAT evaluation framework.** Designed the two-layer assessment method (see below) that the rest of the pipeline is built around.
- **Risk management.** Wrote the risk management section of the project plan and the glossary, using the university's risk rating approach.
- **Architecture and infrastructure design.** Wrote the architecture and infrastructure section of the design document.
- **Security audit.** Carried out a security review of the project VM.
- **Documentation and communication.** Wrote the executive summary, contributed to the requirements and analysis document, and built parts of the final presentation.

## The two-layer evaluation

Each HECVAT control is assessed in two layers:

1. **Layer 1: vendor answer vs. HECVAT expectation.** Does the vendor's response meet what the HECVAT considers a compliant answer?
2. **Layer 2: policy alignment.** Does the answer line up with the university's own security policies? Relevant policy text is retrieved through RAG and given to the model as context.

Both layers are handled in a single LLM inference pass rather than two separate calls, which keeps the pipeline simpler and faster on a small local model.

## Design decisions worth calling out

1. **Dual-context retrieval.** The model sees both the vendor's answer and the relevant policy text together, so it can judge alignment directly instead of guessing what the policy says.
2. **Small local model.** Gemma 3 4B is far less capable than large hosted models, but it fits the offline requirement and the VM hardware. llama-cpp-python is the primary runtime for GPU inference, and Ollama is a CPU fallback switched by a config toggle.
3. **Handling controls with no policy coverage.** Some HECVAT sections (roughly 18% of controls, mainly AI and IT accessibility) have no matching university policy, so retrieval returns nothing useful. Sedona treats these as gaps to flag for a human reviewer rather than letting the model improvise.
4. **Conditional sections.** Three sections (HIPAA, PCI DSS, on-premises) only apply depending on the vendor's earlier answers, so they are handled as N/A when they don't apply.

## Limitations

- HECVAT only covers so much. It can't show a vendor's real security posture, so Sedona flags gaps rather than filling them in.
- LLM output can be wrong or overconfident. Every risk statement needs human review before it is used.
- A 4B model has limited reasoning depth compared to larger hosted models.
- Controls without matching internal policy can't be meaningfully assessed by retrieval.

## Ethical and privacy considerations

- No sensitive data leaves the host, by design
- Outputs are drafts for human reviewers, not decisions
- Traceability is built in so reviewers can check where a claim came from
- Hallucination risk is the main reliability concern, which is why the human review step matters

## What I learned

- Designing evaluation around what the retrieval can and can't actually cover matters more than prompt tweaking
- Air-gapped deployment is a real design trade-off, not just a checkbox: you gain data control and give up model capability
- Mapping questionnaire controls to policy shows how much of security assurance is about gaps and ambiguity, not clean yes/no answers
- Working in a six-person team with a real client means verifying figures and claims before they go into a deliverable, since unchecked numbers cause rework

## Acknowledgements

Team Aegis, Murdoch University IT (client), and our project supervisor.

Built for ICT302, Murdoch University.
