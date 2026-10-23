International Cybersecurity and Digital Forensics Academy (ICDFA)

GRC205: Incident Management and Business Continuity (Week 21)

# BUSINESS CONTINUITY AND DISASTER RECOVERY PLAN


|                           |                                                                      |
|---------------------------|----------------------------------------------------------------------|
| **Student name**          | Agyeman Emmanuel Atta Senior                                         |
| **Course and module**     | GRC205: Incident Management and Business Continuity (Week 21)        |
| **Assessment**            | Individual Practical Assessment (30% weighting, 100 marks)           |
| **Scenario organisation** | Northbridge Skills Institute                                         |
| **Scenario**              | Day 0, 08:00 WAT main site outage (72-hour flooding / power failure) |
| **Submission date**       | 23 October 2026                                                      |

## Executive Mandate and Scope

At 08:00 WAT on Day 0 (Monday), flooding and a severe power failure put Northbridge Skills Institute’s main site out of action for an estimated 72 hours. LMS01, FIN01, REG01, HR01, FS01 and AUTH01 are inaccessible.

This plan follows ISO 22301:2019 and NIST SP 800-34 Rev. 1. It covers a five-function business impact analysis (BIA), dependency analysis, warm site recovery scheduling and a tabletop test, so critical services return within their recovery time objectives (RTOs) without losing data integrity.

## Task 1: Business Impact Analysis

### 1.1 Five-Function BIA Matrix

| **Function and systems**                          | **Owner and users**                     | **Dependencies**                          | **Impact and constraints**                                                                               | **Minimum acceptable service**                                                    | **Priority**        |
|---------------------------------------------------|-----------------------------------------|-------------------------------------------|----------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------|---------------------|
| **F1: Teaching and Assessment (LMS01, DB01)**     | Registrar (1,800 learners, 90 faculty)  | AUTH01, DB01, network, M365, laptops/BYOD | 11:00 WAT exam for 120 learners. NGN 120k reschedule fee if missed. Severe academic loss.                | Online exam portal; 15 centre PCs + 80 BYOD spots; paper attendance roster.       | **1 (Critical)**    |
| **F2: Fees and Finance (FIN01, DB01)**            | Finance Lead (CFO, accounts, payers)    | AUTH01, DB01, network, external gateway   | 16:00 WAT Day 0 fee confirmation deadline. Payment provider is live but the local ledger is not updated. | Offline paper receipt log; payment gateway web portal checks; manual batch entry. | **2 (High)**        |
| **F3: Admissions and Registration (REG01, DB01)** | Admissions Head (new applicants)        | AUTH01, DB01, network, M365 email         | 18:00 WAT Day 0 admissions close. New applications cannot enter the local database.                      | M365 shared spreadsheet for intake; manual timestamping.                          | **3 (Medium-High)** |
| **F4: Staff Payroll (HR01, DB01)**                | HR Manager (90 staff)                   | AUTH01, DB01, network, bank portal        | 12:00 WAT Day 1 bank submission. Payment delays hurt staff welfare and trust.                            | Manual payroll check using the pre-disruption exported sheet.                     | **4 (Medium)**      |
| **F5: Records and Certificates (FS01)**           | Records Lead (graduates, third parties) | AUTH01, FS01, network, storage            | 48-hour internal service commitment. Certificate requests delayed; low immediate loss.                   | Urgent requests logged by M365 email; manual verification once FS01 is restored.  | **5 (Low)**         |

### 1.2 Disruption Cost (8-Hour and 24-Hour Scenarios)

Costs use the supplied hourly NGN rates plus the one-off NGN 120,000 exam rescheduling fee. They assume no continuity measures from the 08:00 WAT baseline.

| **Function**                        | **Hourly rate (NGN)** | **8-hour cost**   | **24-hour cost**  | **Calculation**                                            |
|-------------------------------------|-----------------------|-------------------|-------------------|------------------------------------------------------------|
| **F1: Teaching and Assessment**     | 60,000                | NGN 600,000       | NGN 1,560,000     | (8h × 60k) + 120k = 600k; (24h × 60k) + 120k = 1.56M       |
| **F2: Fees and Finance**            | 40,000                | NGN 320,000       | NGN 960,000       | 8h × 40k; 24h × 40k. 16:00 fee deadline breached.          |
| **F3: Admissions and Registration** | 20,000                | NGN 160,000       | NGN 480,000       | 8h × 20k; 24h × 20k. 18:00 deadline breached.              |
| **F4: Staff Payroll**               | 10,000                | NGN 80,000        | NGN 240,000       | 8h × 10k; 24h × 10k. Day 1 12:00 bank deadline approached. |
| **F5: Records and Certificates**    | 5,000                 | NGN 40,000        | NGN 120,000       | 8h × 5k; 24h × 5k. Within the 48h commitment.              |
| **Total**                           | **135,000**           | **NGN 1,200,000** | **NGN 3,360,000** | **Combined 8h = NGN 1.20M; 24h = NGN 3.36M**               |

**Limits of the financial estimates**

| **Limitation**                 | **Effect**                                                                                                |
|--------------------------------|-----------------------------------------------------------------------------------------------------------|
| **Non-linear cost spikes**     | The model treats cost as linear, but real impact jumps when a regulatory or contractual deadline passes.  |
| **Qualitative costs left out** | Candidate anxiety, brand damage, regulatory scrutiny and possible loss of accreditation are not captured. |

Missing the 11:00 WAT exam harms assessment integrity and the welfare of 1,800 learners. The reputational loss far exceeds the NGN 120,000 fee.

### 1.3 Priority Sequence and Technical Dependencies

Business priority follows deadlines and impact. Technical restoration follows infrastructure dependencies. F1 is the top priority because of the 11:00 WAT exam, but LMS01 cannot start until the recovery network, AUTH01 and DB01 are up.

| **Restoration stage**                          | **Duration**          | **Depends on**             |
|------------------------------------------------|-----------------------|----------------------------|
| 1\. Recovery network                           | 30 min                | None                       |
| 2\. AUTH01 directory                           | 30 min                | Recovery network           |
| 3\. Security and backup checks                 | 30 min                | AUTH01                     |
| 4\. DB01 database                              | 90 min                | Security checks            |
| 5\. Applications: LMS01 / FIN01 / REG01 / HR01 | 45 / 30 / 30 / 45 min | DB01                       |
| FS01 file server                               | 60 min                | AUTH01 and security checks |

**Three critical dependency risks**

| **Risk**                                 | **Why it matters**                                 |
|------------------------------------------|----------------------------------------------------|
| **Single point of failure: DB01**        | It hosts all four application databases.           |
| **Shared AUTH01 directory**              | If it fails, every system login is blocked.        |
| **Single network path to the warm site** | One link failure cuts off the whole recovery site. |

## Task 2: Recovery Objectives and Backup Design

### 2.1 Recovery Objective Matrix

Each function has a safety buffer between its RTO and its maximum tolerable downtime (MTD).

| **Function**                        | **Target RTO**         | **Target RPO**          | **MTD**                | **Buffer** | **Declared service level**                                               |
|-------------------------------------|------------------------|-------------------------|------------------------|------------|--------------------------------------------------------------------------|
| **F1: Teaching and Assessment**     | 4 hours (12:00 WAT)    | 1 hour (07:00 WAT)      | 6 hours (14:00 WAT)    | 2 hours    | Core exam portal restored at the warm site; BYOD + 15 centre PCs live.   |
| **F2: Fees and Finance**            | 6 hours (14:00 WAT)    | 2 hours (06:00 WAT)     | 10 hours (18:00 WAT)   | 4 hours    | FIN01 online before the 16:00 fee deadline; transaction logs reconciled. |
| **F3: Admissions and Registration** | 8 hours (16:00 WAT)    | 4 hours (04:00 WAT)     | 12 hours (20:00 WAT)   | 4 hours    | REG01 online before 18:00 admissions close; M365 intake synced.          |
| **F4: Staff Payroll**               | 18 hours (02:00 Day 1) | 12 hours (20:00 Day -1) | 28 hours (12:00 Day 1) | 10 hours   | HR01 online well before the Day 1 12:00 bank deadline.                   |
| **F5: Records and Certificates**    | 24 hours (08:00 Day 1) | 24 hours (08:00 Day -1) | 48 hours (08:00 Day 2) | 24 hours   | FS01 file shares restored within the 48h commitment.                     |

### 2.2 Backup Checkpoint Analysis

At the 08:00 WAT disruption, the offsite catalogue held four recovery items. The table compares each item’s age with its target RPO.

| **Backup item**           | **Checkpoint (WAT)** | **Age at disruption** | **Target RPO** | **Data loss window**   | **Feasibility and gap**                                                        |
|---------------------------|----------------------|-----------------------|----------------|------------------------|--------------------------------------------------------------------------------|
| **B01: DB01 full backup** | 23:00 Day -1         | 9 hours               | 1–4 hours      | 9 hours (if B02 fails) | Unacceptable for F1/F2. Heavy manual re-entry if the incremental fails.        |
| **B02: DB01 incremental** | 07:00 Day 0          | 1 hour                | 1 hour (F1)    | 1 hour (07:00–08:00)   | Meets F1/F2 RPO. One hour of transactions reconciled via gateway logs.         |
| **B03: AUTH01 snapshot**  | 00:00 Day 0          | 8 hours               | 12 hours       | 8 hours (00:00–08:00)  | Meets RPO. Directory changes made 00:00–08:00 are re-entered.                  |
| **B04: FS01 full backup** | 23:00 Day -1         | 9 hours               | 24 hours (F5)  | 9 hours (23:00–08:00)  | Fully meets the 24h RPO. Files created after 23:00 are re-uploaded from email. |

### 2.3 Backup Schedule and Verification

To close the RPO gaps and remove single-point-of-failure administration, the plan uses a 3-2-1-1-0 design: 3 copies, 2 media types, 1 offsite, 1 offline/immutable, and 0 unverified restore errors.

| **Element**                        | **Design**                                                                                                                                                                                         |
|------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Schedule**                       | DB01 transaction log backups hourly (08:00–22:00), daily full backup at 23:00, and 15-minute log shipping to an isolated cloud vault.                                                              |
| **Restoration chain**              | B01 (full) plus all sequential transaction logs up to the chosen recovery point.                                                                                                                   |
| **Automated sandbox verification** | Offsite replication alone does not prove recoverability. Sandbox restore tests run weekly, checking database checksums, table consistency and access independence before a backup is marked ready. |

## Task 3: Recovery Strategy and Budget

### 3.1 Strategy Comparison (Cold vs Warm vs Hot Site)

| **Package**   | **First critical service** | **Data-loss window** | **Staffing and technical fit**                             | **Malware / propagation risk**           | **RTO/RPO alignment**                                          |
|---------------|----------------------------|----------------------|------------------------------------------------------------|------------------------------------------|----------------------------------------------------------------|
| **Cold site** | 24–48 hours                | Up to 24 hours       | Exceeds the 2-technician capacity; heavy bare-metal build. | Low; clean hardware rebuild.             | **Fails F1, F2, F3 RTOs.** Unacceptable for the 11:00 exam.    |
| **Warm site** | 3–5 hours                  | 1–6 hours            | Fits 2 technicians; pre-configured hardware ready.         | Medium; reduced by sandbox EDR scanning. | **Meets all RTO/RPO targets** with minor business workarounds. |
| **Hot site**  | About 1 hour               | Up to 15 minutes     | Needs dedicated cloud ops staff; high complexity.          | High; real-time sync can spread malware. | Exceeds targets but uses 100% of the budget.                   |

### 3.2 Annual Budget Analysis

Each package is tested against the NGN 3,000,000 annual limit, plus mandatory NGN 400,000 for protected backups and NGN 200,000 for training and drills.

| **Package**   | **Base cost (NGN)** | **Protected backups** | **Training and drills** | **Total annual cost** | **Variance vs NGN 3.0M cap**                      |
|---------------|---------------------|-----------------------|-------------------------|-----------------------|---------------------------------------------------|
| **Cold site** | 600,000             | 400,000               | 200,000                 | **NGN 1,200,000**     | +NGN 1,800,000 (under budget, capability deficit) |
| **Warm site** | 1,500,000           | 400,000               | 200,000                 | **NGN 2,100,000**     | **+NGN 900,000 (recommended; optimal buffer)**    |
| **Hot site**  | 2,400,000           | 400,000               | 200,000                 | **NGN 3,000,000**     | NGN 0 (at limit, no contingency)                  |

**Recommendation and return on investment**

| **Item**           | **Justification**                                                                                                                                                                |
|--------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Recommendation** | Warm site package at NGN 2,100,000 a year. It balances capability, feasibility for a 2-technician team and cost, and leaves a NGN 900,000 contingency buffer.                    |
| **ROI**            | One unmitigated 24-hour outage costs NGN 3,360,000 in direct disruption. At NGN 135,000 an hour, the package pays for itself after about 16 hours of downtime avoided in a year. |

## Task 4: Business Continuity and Disaster Recovery Plan

### 4.1 Governance and Activation

The plan is activated when a site outage is confirmed or expected to exceed 4 hours. The Incident Commander (Operations Lead) holds activation authority, supported by the Deputy Commander (IT Lead).

### 4.2 BC/DR RACI Matrix

R = Responsible; A = Accountable; C = Consulted; I = Informed. Each activity has one A.

| **Activity**                                     | **Incident Cmd** | **IT Recovery** | **Facilities** | **Admissions/Reg** | **Comms** | **CFO** |
|--------------------------------------------------|------------------|-----------------|----------------|--------------------|-----------|---------|
| **1. Plan activation and site declaration**      | **A/R**          | C               | C              | I                  | I         | I       |
| **2. Alternate facility setup (25 PCs)**         | **A**            | R               | R              | I                  | I         | I       |
| **3. Temporary exam execution (BYOD/staggered)** | **A**            | C               | I              | R                  | R         | I       |
| **4. Technical backup restore and validation**   | I                | **A/R**         | I              | I                  | I         | I       |
| **5. Stakeholder and learner communication**     | **A**            | I               | I              | C                  | R         | I       |
| **6. Primary site demobilisation and failback**  | **A**            | R               | R              | C                  | I         | C       |

### 4.3 Temporary Workaround Procedures (72-Hour Outage)

The alternate facility provides 25 workstations for the 120-learner exam.

| **Item**               | **Arrangement**                                                                                                                                 |
|------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------|
| **BYOD learners**      | 80 learners use personal devices over secure guest Wi-Fi.                                                                                       |
| **Centre learners**    | 40 learners in two staggered 20-learner shifts (Shift 1: 12:00 WAT, Shift 2: 13:30 WAT) on 20 of the 25 workstations. 5 are kept as hot spares. |
| **Academic integrity** | Randomised question banks and paper attendance sheets.                                                                                          |

### 4.4 Technical DR Runbook

| **Step** | **Action and target system**            | **Prerequisite**                   | **Owner**    | **Verification and rollback**                                                           |
|----------|-----------------------------------------|------------------------------------|--------------|-----------------------------------------------------------------------------------------|
| **1**    | Bring up the warm site recovery network | Warm site power and gateway active | Tech A       | Ping gateway and DNS test. **Rollback:** switch to the backup WAN interface.            |
| **2**    | Restore AUTH01 directory (B03)          | Step 1 complete                    | Tech A       | Authenticate a test user. **Rollback:** re-apply snapshot B03.                          |
| **3**    | Run security and backup checks          | Step 2 complete                    | Tech A       | AV/EDR scan passes; B01/B02 hashes verified. **Rollback:** isolate the network segment. |
| **4**    | Restore DB01 database (B01 + B02)       | Step 3 complete                    | Tech A       | DBCC CHECKDB passes; test table queries. **Rollback:** fall back to B01 full.           |
| **5**    | Restore applications (LMS01 / FIN01)    | Step 4 complete                    | Tech A and B | Login test and query run. **Rollback:** re-mount the app container.                     |

### 4.5 Stakeholder Communication Matrix and Learner Notice

| **Stakeholder**    | **Message**                                 | **Owner**              | **Channel**              | **Frequency**                      |
|--------------------|---------------------------------------------|------------------------|--------------------------|------------------------------------|
| **Learners**       | Exam rescheduling and instructions          | Registrar / Comms Lead | SMS and email            | At declaration, then every 2 hours |
| **Executive Team** | Status, decisions needed, recovery progress | Incident Cmd           | Phone / secure messaging | Every 2 hours                      |
| **Staff**          | Site closure and working arrangements       | Comms Lead             | SMS / staff group        | Twice daily                        |
| **Bank**           | Payroll deadline risk and payment timing    | Finance Lead (CFO)     | Phone and email          | Before the Day 1 12:00 deadline    |
| **Regulators**     | Disruption notice and any data impact       | Incident Cmd           | Official email           | As regulations require             |

**Learner notice** (Word count: 102)

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr class="odd">
<td><p><strong>IMPORTANT LEARNER NOTICE: ONLINE EXAMINATION RESCHEDULING</strong></p>
<p><strong>TO:</strong> All registered examination candidates (120 learners)</p>
<p><strong>FROM:</strong> Office of the Registrar, Northbridge Skills Institute | <strong>DATE:</strong> Day 0, 08:30 WAT</p>
<p>Due to a power and flooding disruption at our main site, today’s 11:00 WAT examination is <strong>rescheduled</strong> to 12:00 WAT. It will run at our Alternate Facility (Community Tech Centre) and on the BYOD online portal.</p>
<p><strong>What to do:</strong></p>
<ol type="1">
<li><p>BYOD candidates: log into the LMS portal from 11:45 WAT for a system check.</p></li>
<li><p>Candidates needing a centre workstation: report to the Alternate Facility by 11:30 WAT. Exams run in two sessions (Shift 1: 12:00 WAT, Shift 2: 13:30 WAT).</p></li>
<li><p>No candidate will be penalised for delays caused by site transit.</p></li>
</ol>
<p>We will send further updates by SMS. Thank you for your cooperation.</p></td>
</tr>
</tbody>
</table>

## Task 5: Recovery Schedule and Exercise

### 5.1 Warm Site Recovery Schedule (2 Technicians)

The schedule uses the warm site drill durations. Tasks run in parallel only when prerequisites and technician availability allow.

| **Action**                     | **Duration** | **Prerequisite** | **Tech** | **Start (WAT)** | **Finish (WAT) and status** |
|--------------------------------|--------------|------------------|----------|-----------------|-----------------------------|
| **Recovery network**           | 30 min       | None             | A        | 08:00           | 08:30 (network live)        |
| **AUTH01 restoration**         | 30 min       | Recovery network | A        | 08:30           | 09:00 (auth live)           |
| **Security and backup checks** | 30 min       | AUTH01           | A        | 09:00           | 09:30 (checks pass)         |
| **FS01 restoration**           | 60 min       | Security checks  | B        | 09:30           | 10:30 (F5 restored)         |
| **DB01 restoration**           | 90 min       | Security checks  | A        | 09:30           | 11:00 (DB live)             |
| **FIN01 restoration**          | 30 min       | DB01             | B        | 11:00           | 11:30 (F2 restored)         |
| **LMS01 restoration**          | 45 min       | DB01             | A        | 11:00           | 11:45 (F1 restored)         |
| **REG01 restoration**          | 30 min       | DB01             | B        | 11:30           | 12:00 (F3 restored)         |
| **HR01 restoration**           | 45 min       | DB01             | A        | 11:45           | 12:30 (F4 restored)         |

**Schedule path analysis**

| **Item**            | **Result**                                                                                                                                                                          |
|---------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Critical path**   | Recovery network (30m) → AUTH01 (30m) → security checks (30m) → DB01 (90m) → LMS01 (45m) = 225 minutes (3h 45m). LMS01 goes live at 11:45 WAT.                                      |
| **Against targets** | F1 is restored at 11:45 WAT (3h 45m elapsed), inside the 4-hour RTO (12:00 WAT) and 6-hour MTD (14:00 WAT). All five functions are running by 12:30 WAT (4h 30m), within every MTD. |

### 5.2 Tabletop Decision Log (4 Required Injects + 1 Custom)

A 45-minute tabletop exercise tested how well the plan adapts to changing conditions.

| **Inject**                                        | **Decision**                                                                | **Owner**        | **Rationale**                                                              | **Communication**           | **Effect on plan**                                               |
|---------------------------------------------------|-----------------------------------------------------------------------------|------------------|----------------------------------------------------------------------------|-----------------------------|------------------------------------------------------------------|
| **Min 10: Site closure extended to 5 days**       | Keep the warm site running; extend the alternate facility lease.            | Incident Cmd     | The primary site is unusable for a week; the warm site handles the load.   | Notify Board and staff.     | Plan shifts from a 72h sprint to 5 days of sustained operations. |
| **Min 20: Alternate PCs cut from 25 to 15**       | Move 40 centre learners to two 20-learner shifts (12:00 and 13:30 WAT).     | Registrar        | 15 PCs cannot seat 40 learners at once; staggering fixes the shortfall.    | SMS to centre learners.     | Shift 2 exam finishes by 15:00 WAT.                              |
| **Min 30: DB01 B02 fails; unexpected logon**      | Isolate B02; restore B01 full (23:00 Day -1); start a security logon audit. | IT Recovery Lead | Prevents possible malware spread; falls back to a known clean full backup. | Alert the Security Analyst. | RPO rises to 9h; Day 0 23:00–08:00 data must be re-entered.      |
| **Min 40: Payment provider mismatch**             | Reconcile gateway web API logs against FIN01 after the 11:30 restore.       | Finance Lead     | The gateway holds the true record; the local DB was offline.               | Brief the CFO.              | FIN01 ledger updated with zero financial loss by 14:00.          |
| **Min 45 (custom): Main WAN link drops at 10:00** | Fail over to the redundant 5G cellular gateway at the alternate site.       | IT Recovery Lead | Keeps warm site connectivity without delaying the DB01 restore.            | Alert the tech team.        | No delay to the 11:45 LMS01 finish.                              |

### 5.3 Corrective Action Plan

| **Action**                                         | **Priority** | **Owner**        | **Target** | **Measurable evidence**                                |
|----------------------------------------------------|--------------|------------------|------------|--------------------------------------------------------|
| 1\. Automate transaction log shipping (15-min RPO) | High         | IT Recovery Lead | 30 days    | DB01 log shipping active; automated restore test logs. |
| 2\. Expand alternate facility SLA to 40 PCs        | High         | Facilities Lead  | 45 days    | Signed vendor amendment for 40 guaranteed PCs.         |
| 3\. Buy dual cellular 5G failover routers          | Medium       | IT Recovery Lead | 14 days    | Hardware asset tag and automated failover test report. |
| 4\. Automate payment gateway log reconciliation    | Medium       | Finance Lead     | 30 days    | Reconciliation script tested and deployed.             |
| 5\. Hold twice-yearly all-hands BC/DR training     | Medium       | HR Manager       | 60 days    | 85%+ staff attendance records and post-test scores.    |

## Task 6: Governance and Performance Measures

### 6.1 Governance, Review Triggers and Framework Mapping

The Business Continuity Coordinator owns the plan, with annual sign-off by the Board Audit Committee. Reviews are triggered by major infrastructure changes, post-incident reviews and annual drill findings.

| **NIST SP 800-34 Rev. 1 step**              | **ISO 22301:2019 clause**            | **Plan deliverable**               | **Climate / flood consideration**                          |
|---------------------------------------------|--------------------------------------|------------------------------------|------------------------------------------------------------|
| **Step 2: Business impact analysis**        | Clause 8.2: BIA and risk assessment  | Task 1 BIA matrix and calculations | Tests site vulnerability to a local 100-year flood.        |
| **Step 3: Preventive controls**             | Clause 8.3: Strategies and solutions | Task 2 backup design (3-2-1-1-0)   | Offsite backup vault placed outside the local flood plain. |
| **Step 4: Contingency strategies**          | Clause 8.3: Strategies and solutions | Task 3 strategy and budget         | Alternate facility chosen on high ground.                  |
| **Step 5: Plan development**                | Clause 8.4: BC plans and procedures  | Task 4 BC/DR plan                  | Plan assumes severe-weather site loss.                     |
| **Step 6: Testing, training and exercises** | Clause 8.5: Exercise programme       | Task 5 schedule and tabletop log   | Simulates severe weather during the rainy season.          |

### 6.2 Key Performance Indicators

| **KPI**                       | **Formula**                                 | **Data source**         | **Target**      | **Frequency** | **Owner**        |
|-------------------------------|---------------------------------------------|-------------------------|-----------------|---------------|------------------|
| **RTO attainment rate**       | (Achieved recovery time ÷ target RTO) × 100 | Drill and incident logs | ≤ 100%          | Semi-annual   | IT Recovery Lead |
| **Restoration test coverage** | (Systems tested ÷ in-scope systems) × 100   | Backup console audit    | 100% of systems | Quarterly     | IT Recovery Lead |
| **CAP closure rate**          | (Closed actions ÷ total open actions) × 100 | BC/DR audit tracker     | ≥ 90% on time   | Quarterly     | BC Coordinator   |
| **Staff training completion** | (Staff completed ÷ total staff) × 100       | HR LMS reports          | ≥ 95% of staff  | Annual        | HR Manager       |

### 6.3 Maturity Assessment and 12-Month Target

| **Maturity**                              | **Position**                                                                                                                                           |
|-------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Current: Level 1.5 (Initial / ad hoc)** | Backup restore tests had not run in 90 days, documentation was fragmented and site dependencies were undocumented.                                     |
| **12-month target: Level 3.0 (Defined)**  | Needs quarterly automated restore drills, contractually guaranteed alternate facility SLAs, and continuous KPI reporting to the Board Audit Committee. |

## References

**ICDFA.** (2026). *Business Continuity and Disaster Recovery. GRC205: Incident Management and Business Continuity (Week 21 Teaching Deck, Slides 1–15).* International Cybersecurity and Digital Forensics Academy.

**ISO.** (2019). *ISO 22301:2019: Security and resilience – Business continuity management systems – Requirements.* International Organization for Standardization.

**ISO.** (2024). *ISO 22301:2019/Amd 1:2024: Climate action changes.* International Organization for Standardization.

**NIST.** (2010). *Special Publication 800-34 Revision 1: Contingency Planning Guide for Federal Information Systems.* National Institute of Standards and Technology.

**Federal Republic of Nigeria.** (2023). *Nigeria Data Protection Act 2023.* Nigeria Data Protection Commission (NDPC).
