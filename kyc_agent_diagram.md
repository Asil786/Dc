# Global KYC Compliance Agent — Technical Architecture & Workflow

```mermaid
flowchart TD
    %% ─────────────────────────────────────────────
    %%  INPUT INTAKE & VALIDATION
    %% ─────────────────────────────────────────────
    subgraph INTAKE["1. Case Intake & Schema Validation"]
        IN_PAYLOAD["CaseSubmissionPayload\n- Case ID\n- Customer Name & Type\n- Jurisdiction: US / UK / SG\n- Idempotency Key\n- Document Payloads"]
        IN_VALIDATOR["CaseValidator\n- Pydantic v2 Schema Enforcement\n- Idempotency Cache Verification\n- Provenance Metadata Initialization"]
    end

    %% ─────────────────────────────────────────────
    %%  DOCUMENT PROCESSING & SECURITY QUARANTINE
    %% ─────────────────────────────────────────────
    subgraph DOC_ENGINE["2. Document Intake & Security Quarantine (DocumentParser)"]
        DOC_HASH["Cryptographic SHA-256 Hash\nState Fingerprint per Document"]
        DOC_QUARANTINE["Untrusted Content Containment\nEnclosed in untrusted_document_content XML tags"]
        DOC_INJECTION["Adversarial Prompt Injection Scanner\nDetects Direct Overrides, Role Impersonation,\nTool Forgery, Base64/ROT13, Unicode Inversion"]
        DOC_EXTRACT["Structured Field Extractor\n- Entity Name & Registration Number\n- Registered Office Address\n- Beneficial Owners (UBO) & % Shares\n- Issue Dates & 90-Day Freshness Verification"]
    end

    %% ─────────────────────────────────────────────
    %%  CROSS-DOCUMENT VERIFICATION
    %% ─────────────────────────────────────────────
    subgraph CROSS_DOC["3. Cross-Document Verification Engine"]
        CD_NAME["Entity Name Consistency Check\nExact & Levenshtein Similarity Matching"]
        CD_ADDR["Registered Address Cross-Match\nCertificate vs Utility Bill vs Filing"]
        CD_REG["Registration Number Reconciliation\nCross-Doc ID Confirmation"]
        CD_CONTRADICTION["Contradiction & Conflict Detector\nFlags Inconsistent Multi-State / Multi-Doc Filings"]
        CD_EVIDENCE["Evidence Items & Discrepancy Generator\nStructured EvidenceItem Array Output"]
    end

    %% ─────────────────────────────────────────────
    %%  REGISTRY VERIFICATION LAYER
    %% ─────────────────────────────────────────────
    subgraph REGISTRY_LAYER["4. Official Registry Verification Layer (CompositeRegistryProvider)"]
        REG_SEC["SEC EDGAR Provider\nUS CIK / Entity Query via SEC API"]
        REG_CH["Companies House Provider\nUK Company Number Query via CH API"]
        REG_CACHE["Registry Fixture Cache\nDeterministic Testing & Network Fallback\nExplicitly Flagged: is_fallback=True, is_authoritative=False"]
        REG_EVAL{"Entity Status Evaluation"}
        REG_ACTIVE["Status: Active\nAuthoritative Corporate Standing Verified"]
        REG_DISSOLVED["Status: Dissolved / Unknown\nHigh Risk Trigger: +80 Points Penalty"]
    end

    %% ─────────────────────────────────────────────
    %%  SANCTIONS & PEP SCREENING
    %% ─────────────────────────────────────────────
    subgraph SANCTIONS_PEP["5. Sanctions & PEP Screening Engine"]
        SANC_OFAC["OFAC SDN Provider\nUS Treasury Specially Designated Nationals"]
        SANC_UN["UN Sanctions Provider\nUN Consolidated Sanctions List"]
        SANC_PEP["Synthetic PEP Provider\nTier 1: Heads of State, Government Ministers\nTier 2: Senior Officials, Military, Judiciary"]
        SANC_MATCHER["Fuzzy Matching Engine\nConfigurable Similarity Threshold: 0.85\nNormalized Text & Phonetic Alias Scanning"]
        SANC_OUT{"Screening Findings"}
        SANC_HIT["Sanctions Match Identified\nMandatory Escalation: +100 Points"]
        PEP_T1_HIT["PEP Tier 1 Match Identified\nMandatory MLRO Review: +75 Points"]
        PEP_T2_HIT["PEP Tier 2 Match Identified\nEnhanced Scrutiny: +40 Points"]
        SANC_CLEAN["No Sanctions or PEP Matches\nClear Screening Profile"]
    end

    %% ─────────────────────────────────────────────
    %%  DECLARATIVE POLICY & RISK ENGINE
    %% ─────────────────────────────────────────────
    subgraph POLICY_RISK["6. Declarative Policy & Risk Engine"]
        POL_LOADER["PolicyLoader & ConditionRegistry\nLoads risk_policy.yaml, US.yaml, UK.yaml\nDeterministic Function Registry (Zero eval)"]
        RISK_CALC["QuantitativeRiskCalculator\n- Sanctions Match: +100 pts\n- Dissolved Registry: +80 pts\n- PEP Tier 1 Hit: +75 pts\n- Critical Doc Mismatch: +45 pts\n- Missing UBO Disclosure: +45 pts\n- PEP Tier 2 Hit: +40 pts\n- Unsupported Jurisdiction: +40 pts\n- Expired Proof of Address: +25 pts\n- Minor Doc Discrepancy: +15 pts\n- Fallback Penalty Cap: Max 30 pts Total"]
        RISK_TIERS["Deterministic Tier Assignment\n- LOW: 0 - 30 Points\n- MEDIUM: 31 - 69 Points\n- HIGH: 70 - 89 Points\n- CRITICAL: 90 - 100 Points"]
    end

    %% ─────────────────────────────────────────────
    %%  LANGGRAPH STATE MACHINE & ROUTING
    %% ─────────────────────────────────────────────
    subgraph STATE_MACHINE["7. LangGraph Orchestration & Decision Gate"]
        ROUTER{"Risk Threshold Check\nScore >= 70 OR\nSanctions Hit OR\nPEP Tier 1 Hit?"}
    end

    %% ─────────────────────────────────────────────
    %%  PATH A: AUTOMATED CLEARANCE
    %% ─────────────────────────────────────────────
    subgraph PATH_AUTO["8A. Automated Approval Path (Low / Medium Risk)"]
        AUTO_REC["Generate Automated Approval Recommendation"]
        AUTO_SEAL["Compile Evidence & Finalize State"]
        AUTO_VERDICT["Final Verdict: RECOMMENDATION_APPROVE\nWorkflow Status: COMPLETED"]
    end

    %% ─────────────────────────────────────────────
    %%  PATH B: HUMAN-IN-THE-LOOP (MLRO REVIEW)
    %% ─────────────────────────────────────────────
    subgraph PATH_HITL["8B. Human-in-the-Loop MLRO Review Path (High / Critical Risk)"]
        HITL_PAUSE["Workflow Checkpoint Pause\nState: WAITING_FOR_HUMAN_REVIEW\nExecution Halted via LangGraph MemorySaver"]
        HITL_PACKET["Structured MLRO Review Packet\n- Numeric Risk Score & Applied Weights\n- Specific Evidence Items & Violations\n- Sanctions / PEP Match Details\n- Discrepancy Matrix & Cross-Doc Comparison"]
        HITL_RESUME["Workflow Resumption via resume_kyc_case\nMLRO Submits Authorized Decision Payload"]
        HITL_DECISION{"MLRO Decision"}
        DEC_COND["APPROVED_WITH_CONDITIONS\nConditions Appended to Dossier"]
        DEC_EDD["ESCALATED_TO_EDD\nReferred for Enhanced Due Diligence"]
        DEC_REJ["REJECTED\nOnboarding Formally Declined"]
    end

    %% ─────────────────────────────────────────────
    %%  DECISION ENGINE & DOSSIER SYNTHESIS
    %% ─────────────────────────────────────────────
    subgraph DOSSIER_SYNTHESIS["9. Decision Engine & Executive Dossier Synthesis"]
        DE_SYNTH["DecisionEngine.synthesize_dossier\n- Assembles Full Evidence Graph\n- Compiles Executive Summary & Timeline\n- Formulates Final Compliance Recommendations\n- Generates Regulatory Review Packet"]
    end

    %% ─────────────────────────────────────────────
    %%  CRYPTOGRAPHIC AUDIT & GOVERNANCE
    %% ─────────────────────────────────────────────
    subgraph AUDIT_GOVERNANCE["10. Cryptographic Audit Trail & Governance"]
        AUDIT_LOGGER["LocalAuditLogger (Tamper-Evident SHA-256 Hash Chain)\n- Genesis Hash Initialization\n- Previous Hash Pointer Linking on Every Event\n- Logs Phase, Action, Tool, Role, Timestamp, Execution ID\n- Standalone Integrity Verification: verify_log_integrity"]
        RBAC_GUARD["ToolExecutionGuard (RBAC Enforcement)\n- Enforces Declarative TOOL_PERMISSION_REGISTRY\n- Validates Caller Role vs Lifecycle Phase\n- Emits Audit Event & Raises ToolAuthorizationError on Violation"]
    end

    %% ─────────────────────────────────────────────
    %%  WORKFLOW CONNECTIONS
    %% ─────────────────────────────────────────────
    IN_PAYLOAD --> IN_VALIDATOR
    IN_VALIDATOR --> DOC_HASH
    DOC_HASH --> DOC_QUARANTINE --> DOC_INJECTION --> DOC_EXTRACT
    DOC_EXTRACT --> CD_NAME & CD_ADDR & CD_REG & CD_CONTRADICTION
    CD_NAME & CD_ADDR & CD_REG & CD_CONTRADICTION --> CD_EVIDENCE
    CD_EVIDENCE --> REG_SEC & REG_CH
    REG_SEC -.->|API Call / Fallback| REG_CACHE
    REG_CH -.->|API Call / Fallback| REG_CACHE
    REG_SEC & REG_CH & REG_CACHE --> REG_EVAL
    REG_EVAL -->|Active| REG_ACTIVE
    REG_EVAL -->|Dissolved / Inactive| REG_DISSOLVED
    REG_ACTIVE & REG_DISSOLVED --> SANC_OFAC & SANC_UN & SANC_PEP
    SANC_OFAC & SANC_UN & SANC_PEP --> SANC_MATCHER
    SANC_MATCHER --> SANC_OUT
    SANC_OUT -->|Sanctions Hit| SANC_HIT
    SANC_OUT -->|PEP Tier 1| PEP_T1_HIT
    SANC_OUT -->|PEP Tier 2| PEP_T2_HIT
    SANC_OUT -->|Clean| SANC_CLEAN
    SANC_HIT & PEP_T1_HIT & PEP_T2_HIT & SANC_CLEAN --> POL_LOADER
    POL_LOADER --> RISK_CALC --> RISK_TIERS
    RISK_TIERS --> ROUTER

    ROUTER -->|Score < 70 and Clean| AUTO_REC
    AUTO_REC --> AUTO_SEAL --> AUTO_VERDICT --> DE_SYNTH

    ROUTER -->|Score >= 70 or Sanctions or PEP Tier 1| HITL_PAUSE
    HITL_PAUSE --> HITL_PACKET --> HITL_RESUME --> HITL_DECISION
    HITL_DECISION -->|Approve with Conditions| DEC_COND --> DE_SYNTH
    HITL_DECISION -->|Escalate| DEC_EDD --> DE_SYNTH
    HITL_DECISION -->|Reject| DEC_REJ --> DE_SYNTH

    DE_SYNTH --> AUDIT_LOGGER
    RBAC_GUARD -.->|Authorizes & Logs Every Tool Call| DOC_ENGINE
    RBAC_GUARD -.->|Authorizes & Logs Every Tool Call| REGISTRY_LAYER
    RBAC_GUARD -.->|Authorizes & Logs Every Tool Call| SANCTIONS_PEP

    %% ─────────────────────────────────────────────
    %%  STYLING CLASSES (CLEAN ENTERPRISE PALETTE)
    %% ─────────────────────────────────────────────
    classDef default fill:#FFFFFF,stroke:#CBD5E1,stroke-width:1px,color:#0F172A;
    classDef intakeStyle fill:#F8FAFC,stroke:#64748B,stroke-width:1.5px,color:#0F172A;
    classDef docStyle fill:#F0FDF4,stroke:#16A34A,stroke-width:1.5px,color:#14532D;
    classDef crossDocStyle fill:#ECFEFF,stroke:#0891B2,stroke-width:1.5px,color:#155E75;
    classDef regStyle fill:#F5F3FF,stroke:#7C3AED,stroke-width:1.5px,color:#4C1D95;
    classDef sancStyle fill:#FEF2F2,stroke:#DC2626,stroke-width:1.5px,color:#7F1D1D;
    classDef policyStyle fill:#FDF4FF,stroke:#A21CAF,stroke-width:1.5px,color:#701A75;
    classDef routerStyle fill:#FEFCE8,stroke:#CA8A04,stroke-width:2px,color:#713F12;
    classDef autoStyle fill:#ECFDF5,stroke:#059669,stroke-width:1.5px,color:#064E3B;
    classDef hitlStyle fill:#FFF7ED,stroke:#EA580C,stroke-width:1.5px,color:#7C2D12;
    classDef synthStyle fill:#EFF6FF,stroke:#2563EB,stroke-width:1.5px,color:#1E3A8A;
    classDef auditStyle fill:#0F172A,stroke:#0F172A,stroke-width:2px,color:#F8FAFC;

    class INTAKE,IN_PAYLOAD,IN_VALIDATOR intakeStyle;
    class DOC_ENGINE,DOC_HASH,DOC_QUARANTINE,DOC_INJECTION,DOC_EXTRACT docStyle;
    class CROSS_DOC,CD_NAME,CD_ADDR,CD_REG,CD_CONTRADICTION,CD_EVIDENCE crossDocStyle;
    class REGISTRY_LAYER,REG_SEC,REG_CH,REG_CACHE,REG_EVAL,REG_ACTIVE,REG_DISSOLVED regStyle;
    class SANCTIONS_PEP,SANC_OFAC,SANC_UN,SANC_PEP,SANC_MATCHER,SANC_OUT,SANC_HIT,PEP_T1_HIT,PEP_T2_HIT,SANC_CLEAN sancStyle;
    class POLICY_RISK,POL_LOADER,RISK_CALC,RISK_TIERS policyStyle;
    class ROUTER routerStyle;
    class PATH_AUTO,AUTO_REC,AUTO_SEAL,AUTO_VERDICT autoStyle;
    class PATH_HITL,HITL_PAUSE,HITL_PACKET,HITL_RESUME,HITL_DECISION,DEC_COND,DEC_EDD,DEC_REJ hitlStyle;
    class DOSSIER_SYNTHESIS,DE_SYNTH synthStyle;
    class AUDIT_GOVERNANCE,AUDIT_LOGGER,RBAC_GUARD auditStyle;
```
