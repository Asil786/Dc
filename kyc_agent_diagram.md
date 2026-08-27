# Global KYC Compliance Agent — Technical Architecture & Workflow

```mermaid
flowchart TD
    %% ─────────────────────────────────────────────
    %% STAGE 1: INTAKE & SECURITY QUARANTINE
    %% ─────────────────────────────────────────────
    subgraph STAGE1 ["Stage 1: Case Intake & Document Security Quarantine"]
        IN1["<b>Case Submission Payload</b><br/>• Customer Metadata & Jurisdiction (US / UK / SG)<br/>• Idempotency Key & Document Payloads"]
        
        IN2["<b>CaseValidator & RBAC Guard</b><br/>• Pydantic v2 Schema Enforcement<br/>• Idempotency Cache Verification<br/>• Tool Permission Pre-Authorization"]
        
        IN3["<b>DocumentParser & Security Quarantine</b><br/>• SHA-256 State Fingerprinting<br/>• Isolated in XML Boundary Tags<br/>• Adversarial Prompt Injection Defense<br/>• Structured Extraction: Name, Reg No, Address, UBO %"]
        
        IN1 --> IN2 --> IN3
    end

    %% ─────────────────────────────────────────────
    %% STAGE 2: CROSS-DOCUMENT & REGISTRY
    %% ─────────────────────────────────────────────
    subgraph STAGE2 ["Stage 2: Cross-Document & Official Registry Verification"]
        CD1["<b>Cross-Document Verification Engine</b><br/>• Levenshtein Name & Address Similarity<br/>• Cross-Filing ID Reconciliation<br/>• Contradiction & Multi-State Conflict Detection"]
        
        REG1["<b>Official Corporate Registry Layer</b><br/>• SEC EDGAR API (US) / Companies House API (UK)<br/>• Non-Authoritative Fallback Cache"]
        
        REG_STATUS{"<b>Registry Corporate Status</b>"}
        
        REG_ACT["<b>Active Corporate Standing</b><br/>Official Status Verified"]
        REG_DIS["<b>Dissolved / Inactive Status</b><br/>High Risk Penalty: +80 Points"]
        
        IN3 --> CD1 --> REG1 --> REG_STATUS
        REG_STATUS -->|"Active"| REG_ACT
        REG_STATUS -->|"Dissolved / Inactive"| REG_DIS
    end

    %% ─────────────────────────────────────────────
    %% STAGE 3: SANCTIONS, PEP & RISK ENGINE
    %% ─────────────────────────────────────────────
    subgraph STAGE3 ["Stage 3: Sanctions Screening & Declarative Risk Engine"]
        SANC1["<b>Sanctions & PEP Screening Engine</b><br/>• OFAC SDN List (US) & UN Sanctions (Global)<br/>• Tier 1 & Tier 2 PEP Database Screening<br/>• Fuzzy Matching Engine (0.85 Threshold)"]
        
        POL1["<b>Declarative Policy Loader (YAML)</b><br/>• Evaluates US / UK Jurisdictional Rules<br/>• Condition Registry with Zero-Eval Execution"]
        
        RISK1["<b>Quantitative Risk Calculator</b><br/>• Computes 0-100 Weighted Risk Score<br/>• Stacking Cap Applied (Max 30 pts for Fallbacks)<br/>• Assigns Tier: LOW / MEDIUM / HIGH / CRITICAL"]
        
        REG_ACT --> SANC1
        REG_DIS --> SANC1
        SANC1 --> POL1 --> RISK1
    end

    %% ─────────────────────────────────────────────
    %% STAGE 4: DECISION ROUTING & DUAL-PATH RESOLUTION
    %% ─────────────────────────────────────────────
    subgraph STAGE4 ["Stage 4: State Machine Routing & Review Resolution"]
        GATE{"<b>Risk Routing Gate</b><br/>Score >= 70 OR Sanctions Hit OR PEP Tier 1?"}
        
        AUTO_PATH["<b>Automated Clearance Path (Low / Medium Risk)</b><br/>• Score < 70 & Clean Screening<br/>• Verdict: RECOMMENDATION_APPROVE<br/>• Zero Human Intervention Required"]
        
        HITL_PAUSE["<b>Human-in-the-Loop MLRO Review Path (High / Critical Risk)</b><br/>• Workflow PAUSES at WAITING_FOR_HUMAN_REVIEW Checkpoint<br/>• Generates Structured MLRO Packet with Evidence & Violations"]
        
        HITL_ACTION{"<b>MLRO Review Decision</b><br/>Submitted via resume_kyc_case"}
        
        DEC_APP["<b>APPROVED_WITH_CONDITIONS</b><br/>Conditions Appended to Record"]
        DEC_EDD["<b>ESCALATED_TO_EDD</b><br/>Referred for Enhanced Due Diligence"]
        DEC_REJ["<b>REJECTED</b><br/>Onboarding Formally Declined"]
        
        RISK1 --> GATE
        GATE -->|"No: Score < 70 & Clean"| AUTO_PATH
        GATE -->|"Yes: Score >= 70 / Sanctions / PEP"| HITL_PAUSE
        
        HITL_PAUSE --> HITL_ACTION
        HITL_ACTION -->|"Approved with Conditions"| DEC_APP
        HITL_ACTION -->|"Escalate to EDD"| DEC_EDD
        HITL_ACTION -->|"Reject"| DEC_REJ
    end

    %% ─────────────────────────────────────────────
    %% STAGE 5: DOSSIER SYNTHESIS & AUDIT
    %% ─────────────────────────────────────────────
    subgraph STAGE5 ["Stage 5: Dossier Synthesis & Cryptographic Audit"]
        DOSSIER["<b>Decision Engine: Dossier Synthesis</b><br/>• Assembles Executive Summary & Case Timeline<br/>• Formulates Final Recommendations & Evidence Graph"]
        
        AUDIT["<b>Cryptographic Audit Trail (LocalAuditLogger)</b><br/>• SHA-256 Tamper-Evident Hash Chain<br/>• Previous Hash Pointer Linking on Every Event<br/>• Logs Phase, Tool, Role, Timestamp & Execution ID<br/>• Standalone Integrity Verification: verify_log_integrity"]
        
        AUTO_PATH --> DOSSIER
        DEC_APP --> DOSSIER
        DEC_EDD --> DOSSIER
        DEC_REJ --> DOSSIER
        DOSSIER --> AUDIT
    end

    %% ─────────────────────────────────────────────
    %% VISUAL STYLING (GITHUB LIGHT & DARK COMPLIANT)
    %% ─────────────────────────────────────────────
    classDef default fill:#ffffff,stroke:#cbd5e1,stroke-width:1px,color:#0f172a;
    classDef intakeCard fill:#f0f9ff,stroke:#0284c7,stroke-width:1.5px,color:#0f172a;
    classDef processCard fill:#f8fafc,stroke:#475569,stroke-width:1.5px,color:#0f172a;
    classDef gateDiamond fill:#fffbeb,stroke:#d97706,stroke-width:2px,color:#0f172a;
    classDef successCard fill:#ecfdf5,stroke:#059669,stroke-width:1.5px,color:#0f172a;
    classDef warningCard fill:#fff7ed,stroke:#ea580c,stroke-width:1.5px,color:#0f172a;
    classDef dangerCard fill:#fef2f2,stroke:#dc2626,stroke-width:1.5px,color:#0f172a;
    classDef auditCard fill:#0f172a,stroke:#0f172a,stroke-width:2px,color:#ffffff;

    class IN1,IN2,IN3 intakeCard;
    class CD1,REG1,SANC1,POL1,RISK1,DOSSIER processCard;
    class REG_STATUS,GATE,HITL_ACTION gateDiamond;
    class REG_ACT,AUTO_PATH,DEC_APP,DEC_EDD successCard;
    class HITL_PAUSE warningCard;
    class REG_DIS,DEC_REJ dangerCard;
    class AUDIT auditCard;
```
