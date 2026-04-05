# About
A complete facilitator's script for running seven Domain-Driven Design sessions that transform a broken industrial inspection system into a domain-driven architecture — from sticky notes on Day 1 to a deployable MVP scope and EU AI Act compliance framework on Day 2.

## System Description — Business and Technical Context

### The System in One Sentence
*The Visual Quality Inspection System (VQS)* system is an industrial AI application that detects surface defects on steel panels in real time at the production edge and uses cloud-based large language model analysis to identify the root causes of recurring defect patterns — with the goal of closing the feedback loop between human operator judgement and model improvement.


### Business Context
The system operates on a steel panel production line (Line 3) at manufacturing facility. The line runs three shifts per day, seven days a week. At current throughput, the edge vision node inspects several thousand panels per shift.

*The core business problem* is defect escape: defective panels that pass through the inspection station undetected and are confirmed as defective only downstream — after further processing, packaging, or in the worst case, at the customer. The baseline defect escape rate entering this workshop is 12 escapes per 1,000 inspected units. The business goal is to reduce this to 8.4 or fewer — a 30% improvement — within six months of production deployment.

*Three organisational pressures drive the urgency.*
The first is quality cost. Every escaped defect represents rework, warranty exposure, or customer dissatisfaction. At current volumes, even a marginal reduction in escape rate has a measurable financial impact.
The second is regulatory obligation. The VQS system is classified as a high-risk AI system under the *EU AI Act.*

*This classification triggers obligations around data governance (Article 10), human oversight (Article 14), transparency (Article 13), and technical documentation (Article 12 and Annex IV). The system cannot legally operate in production without these obligations met. They are not optional features — they are go-live criteria.*
The third is operational fragility. The current system has no structured feedback path between operator behaviour and model improvement. When the model drifts — as it did during the lighting-change incident, producing 340 false positives over three days — the only discovery mechanism is an ML Engineer noticing an anomaly in a weekly report. This 72-hour gap is the primary operational risk the workshop is designed to close.

### Technical Context
The VQS system operates across two distinct compute environments with different latency profiles, ownership models, and failure modes.

*The edge environment* runs on a fleet of vision nodes (EV-07) installed at the inspection stations on the production line. Each node captures four overlapping high-resolution frames per panel pass, runs an ML inference pipeline (YOLOv8-based, custom-trained on surface defect classes), and produces a defect classification with a confidence score within 140 milliseconds. The node is connected to the production line PLC, which triggers a physical diverter to route panels to scrap, rework, or pass lanes based on the assigned Decision. The node is also connected to an HMI workstation where the Quality Engineer receives alerts, reviews thumbnail images, and submits overrides. The edge environment is latency-critical and must operate correctly even during brief network interruptions with the cloud.

*The cloud environment* runs the Learning Loop — the set of services responsible for root cause analysis, model retraining, and data governance. These services are asynchronous: they are not in the critical path of the inspection decision and can tolerate higher latency. The cloud environment hosts an LLM/VLM-based root cause analysis pipeline, a case management system that tracks the lifecycle of inspection cases from intake through human-approved remediation, a model operations platform that manages versioned model artifacts and retraining pipelines, and a data and feature engineering layer that curates training datasets and maintains data lineage for EU AI Act compliance.

*The integration between the two environments* is mediated by a small set of domain events. The edge publishes InspectionCaseForwarded and DriftSignal events to the cloud. The cloud pushes new model artifacts and versioned DecisionBand configurations back to the edge fleet. This event-based boundary is deliberate: it decouples the real-time Decision Loop from the asynchronous Learning Loop and allows each to evolve independently.

*The LLM's role is strictly advisory.* The cloud RCA system uses a large language model to generate root cause hypotheses — plausible explanations for why a defect pattern is occurring, supported by evidence references from image batches and override logs. These hypotheses are never actioned directly. They are translated by an Anti-Corruption Layer inside the Case Management context into structured Recommendations, each of which requires explicit human approval from a Case Manager before any corrective action is taken. This architectural principle — keep the LLM advisory — is non-negotiable and is enforced at the integration boundary between the RCA and Case Management contexts.

*The current state before the workshop* is that none of these boundaries, patterns, or ownership rules are formally documented. The system has been built incrementally, with different teams owning different components without a shared language or explicit integration contracts. The EventStorming session makes this visible. The Bounded Context Canvas and Context Mapping sessions formalise it. The Example Mapping session resolves the business rule ambiguities that have been living in people's heads. The User Story Map and Impact Map translate the result into a prioritised delivery plan.

### The Two Loops
The system architecture is built around two distinct processing loops that operate at different timescales and serve different purposes.

*The Decision Loop* runs at the edge, in real time. A panel arrives, the model fires, a Decision is assigned, the operator reviews, an Override may be submitted. This loop completes in under 30 seconds and is the primary mechanism by which the business goal — reducing defect escape rate — is achieved. It is entirely deterministic: given the same panel and the same active DecisionBand, the system will always produce the same Decision.

*The Learning Loop* runs in the cloud, asynchronously. Override patterns accumulate, a threshold is breached, a root cause analysis is triggered, a Recommendation is produced and approved, and eventually a new model version is trained and deployed back to the edge. This loop operates on a timescale of hours to weeks and is the mechanism by which the system improves over time. It is probabilistic: the LLM hypothesis may be correct or incorrect, the retraining may improve the model or degrade it, and human judgement is required at the approval gate to prevent automated propagation of errors.

*The central failure* of the current system is that *these two loops are not connected*. Overrides flow into the Decision Loop but do not reliably reach the Learning Loop. Drift occurs at the edge but is not detected in the cloud until a human happens to notice. Root cause analysis reports are generated but are not linked back to model updates. The workshop's primary structural contribution is to make this connection explicit, formally named, and owned.


## Workshop Index
[Workshop playbook](workshop_playbook.adoc)

[Step 1:Domain Storytelling](step1_domain_storytelling.adoc)

[Step 2:EventStorming](step2_eventstorming.adoc)

[Step 3:Example Mapping](step3_example_mapping.adoc)

[Step 4:Bounded Context Canvas](step4_bounded_context_canvas.adoc)

[Step 5:Context Mapping](step5_context_mapping.adoc)

[Step 6: User Story Mapping](step6_user_story_mapping.adoc)

[Step 7:Impact Mapping](step7_impact_mapping.adoc)
