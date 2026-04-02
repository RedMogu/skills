# 🎯 Lark/Feishu Intelligent Recruitment: Harness Engineering Resume Screening SOP (V3.3 - Open Skill)

> **Core Philosophy**: LLMs have limitations and are easily fooled by resumes stuffed with buzzwords like "Agent, ReAct, RAG, High Concurrency". This skill utilizes **OpenClaw Multi-Agent Collaboration (A2A)** to pioneer a **[Red/Blue Team Audit Mechanism (Bad Cop / Good Cop)]**. The Sub-Agent acts as the "Red Team Attacker" for ruthless debunking; the Main Agent acts as the "Blue Team Pragmatist" for final value extraction and pragmatic calibration. Built on native PDF multi-modal parsing and Lark (Feishu) Bitable state machines.

## 📍 1. Core Architecture (Red/Blue Team Agent Topology)

Strictly follows the **OpenClaw A2A Collaboration Protocol**:
1. **Sub-Agent / The Bad Cop (Advisor Cat)**: Dedicated to "X-ray Toxicity Reviews". Receives factual data and anti-fraud timelines, looking for logical loopholes, timeline fraud, and buzzword stuffing with extreme skepticism. The goal is to ruthlessly expose the candidate's packaging.
2. **Main Agent / The Good Cop (Grand Overseer)**: The workflow orchestrator and **Final Adjudicator**. Coordinates the Lark workflow. Upon receiving the Sub-Agent's "roast report", performs a **Pragmatic Calibration** to prevent the Sub-Agent from "accidentally killing" honest candidates who have solid fundamentals but slightly over-packaged resumes.

---

## 🗄️ 2. Lark Bitable Schema (Multi-JD Support & Dynamic Routing)

> **Auto-Provisioning**: If no table token is provided, the Main Agent will automatically call the API to create the following four tables.

### 📌 Table 1: Candidates (State Machine)
- **Name/Email/Phone** (Text): Unique primary key.
- **Resume File** (Attachment/Link): PDF uploaded by HR or direct Lark Drive link.
- **Applied Role** (Text): The role the candidate explicitly applied for.
- **Status** (Single Select): `[Pending AI]`, `[Processing]`, `[AI Scored]`, `[Pending Human]`, `[Interviewing]`, `[Rejected]`.
- **Matched JD** (Text): The final authentic role determined after the Main Agent's calibration.
- **Fact Layer** (Multi-line Text): Objective facts extracted by AI.
- **Sub-Agent Roast** (Multi-line Text): Flaws and negotiation evidence found by the Bad Cop.
- **Interview Traps** (Multi-line Text): Deadly interview questions prepared by the Bad Cop.
- **Main Agent Decision** (Multi-line Text): The Good Cop's final justification for retaining/grading the candidate.
- **Tier** (Single Select): Tier 1 (Core) / Tier 2 (Cost-effective Downgrade) / Tier 3 (Cheap Labor) / Tier 4 (Eliminated).
- **Confidence Score** (Number): 0-100. Triggers human intervention if below 60.

### 📌 Table 2: Anti-Fraud Timeline & Red Lines (Anti-Fraud Dictionary)
- **Verification Item** (Text): e.g., `DeepSeek-V3` (Tech) or `E-commerce Dividend Period` (Operations).
- **Occurrence/Open Source Date** (Date/Text): e.g., `Dec 2024`.
- **Fraud/Exaggeration Rule** (Multi-line Text): Rigid constraints for the Sub-Agent to cross-reference.

### 📌 Table 3: JD Library
- **JD ID / Role Name** (Text PK): e.g., `Senior AI Architect`, `Overseas Growth Ops`.
- **Core Hard Requirements** (Multi-line Text): The absolute bottom line of authentic capabilities required.

### 📌 Table 4: ⚙️ System Routing Config
Used to decouple underlying logic and implement dynamic file routing:
- **Config Key**: e.g., `PDF Resume Route` or `Markdown Resume Route`.
- **Folder Token**: The corresponding Lark Drive target folder Token.
- **Mechanism & Purpose**: 
  - PDFs must be directed to a **dedicated anti-overwrite directory**, strictly retaining the UUID to prevent file loss.
  - Markdown parsed versions must be sent to the default `candidates` directory for LLM review.

---

## 🎭 3. Red/Blue Team Prompts (The Bad Cop vs The Good Cop)

### 🔴 Layer 1: The Bad Cop's Ultimate Roast Prompt (Sub-Agent)
> **Injected via `sessions_spawn`, universal anti-fraud logic**
```markdown
You are an extremely picky, highly critical auditor (Advisor Cat) who takes pleasure in debunking candidates' resume packaging. You are fluent in the "industry jargon" of tech, product, and operations.
Your goal: Find fraud, exaggeration, and logical loopholes between the lines.

[Absolute Red Lines]:
1. Zero Tolerance for "Empty Buzzwords": Whether tech (RAG, High Concurrency) or non-tech (Ecosystem Loop, 0-to-1, Empowerment), any grand narrative lacking actual data is considered packaging. You MUST look for quantitative metrics in the [Fact Layer] (e.g., real revenue, DAU/MAU, code implementation details). Unsubstantiated "airborne achievements" must be severely penalized.
2. Timeline & Logic Hygiene: Cross-reference the [Anti-Fraud Timeline] and industry common sense. Tech implementations predating open-source releases, or massive logical breaks (e.g., "managed $100M portfolio in freshman internship"), must be instantly tagged as `[FLAG_BS]` fraud.

[Output Requirements (JSON)]:
- `fact_layer`: Pure objective facts stripped of all marketing fluff (data, tools, concrete outputs only).
- `roast_report`: Your toxic attack report, including `evidence_quote` (direct quotes from the resume as hard evidence) and a biting, sarcastic question.
- `interview_traps`: Deadly interview questions designed to instantly pierce the candidate's packaging.
```

### 🔵 Layer 2: The Good Cop's Pragmatic Calibration Prompt (Main Agent)
> **Executed in-memory by the Main Agent after receiving the Bad Cop's JSON**
```markdown
You are a pragmatic, ROI-driven hiring manager. You just received a highly critical, toxic review report (`roast_report`) from your subordinate "Advisor Cat" regarding a candidate.
Your task: **Filter out the Sub-Agent's emotion, mine the candidate's residual value, and make a final decision by cross-referencing the full [JD Library]**.

[Fallback & Retention Logic]:
1. Look Past the Packaging at the Foundation: Even if a candidate blew up "calling an API" into "developing a proprietary LLM architecture" and got roasted by the Sub-Agent, look at their `fact_layer`. If they have a solid 5 years of Java backend experience, we can still downgrade and absorb them as a [Basic Backend Developer].
2. Dimensionality Reduction (Downgrading): If they don't meet the requirements of the senior role they applied for, look downwards in the [JD Library] for lower-level alternatives.
3. No Waste: Only relegate them to Tier 4 (Complete Elimination) if their [fundamental execution skills are garbage] AND [they are blatant liars with zero integrity].

[Final Output (JSON)]:
- `matched_role`: The final authentic matched role after global optimization (often a downgraded result).
- `final_tier`: Final rating (Tier 1~4).
- `decision_reason`: Your justification for retention or grading.
```

---

## 🚨 4. Data Merge & Write-back Protocol (Anti-Omission Guardrail)
When the Main Agent executes the final `batch_update` to Lark Bitable, there is a **high risk of "hallucinating away" or omitting the Sub-Agent's toxic review** due to the Main Agent's tendency to only write its own conclusions. The code and execution workflow MUST forcibly merge and map BOTH layers:
- Layer 1 (Bad Cop)'s `roast_report` MUST be preserved verbatim and written to `[Sub-Agent Roast / Negotiation Room]`.
- Layer 1 (Bad Cop)'s `interview_traps` MUST be written exactly as generated to `[Interview Traps]`.
- Layer 2 (Good Cop)'s `decision_reason` is written to `[Main Agent Decision]`.

**It is strictly prohibited for the Main Agent to filter, summarize, or omit the Bad Cop's attack report during the final database write. The Blue Team's pragmatic calibration is an additive field; the Red Team's toxic review MUST be served in full.**

---

## 🗺️ 5. Workflow Pipeline

```mermaid
sequenceDiagram
    participant HR as Human HR
    participant Feishu as Lark (Bitable / Drive)
    participant Main as Main Agent (Good Cop)
    participant SubAgent as Sub-Agent (Bad Cop)

    HR->>Feishu: 1. Input roles, anti-fraud standards, and routing configs
    HR->>Feishu: 2. Upload resume (Status=Pending AI)
    
    Main->>Feishu: 3. Read [System Routing Config] to get Folder Tokens
    Main->>Main: 4. Fetch pending resumes + Native PDF extraction, push dual formats to respective routes
    Main->>SubAgent: 5. sessions_spawn triggers Red Team Audit
    
    Note over SubAgent: 🔍 Ignores grand narratives, demands hard data
    SubAgent-->>Main: 6. Returns Roast Report & Interview Traps
    
    Note over Main: ⚖️ Blue Team Pragmatic Calibration<br/>Mandatory Data Merge of Bad Cop's Report
    Main->>Main: 7. Evaluates Fact + Roast + Full JD Library
    
    Main->>Feishu: 8. batch_update writes back to Bitable
```

---

## 🛠️ 6. Execution Standards & Guardrails
1. **Source Acquisition**: Use the native multi-modal `pdf` tool (Zero OCR).
2. **Observe Dynamic Routing**: Extracted PDF and MD files MUST be stored in the respective Folder Tokens specified in the **[System Routing Config]**. Retaining the anti-overwrite UUID is mandatory.
3. **Asynchronous Tearing (Sub-Agent)**: Spawn the Bad Cop to generate the aggressive anti-fraud report.
4. **Main Process Consolidation (Main Agent)**: **MUST strictly execute the [Data Merge Guardrail] in Section 4.**
5. **Safe Write-back**: Use `batch_update` with exponential backoff retries.
