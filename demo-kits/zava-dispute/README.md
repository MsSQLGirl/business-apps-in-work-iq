# Zava Capital dispute demo: scenario, prompts, and setup

**In short:** Zava Capital is a fictional bank. A credit-card dispute analyst looks into disputes using Copilot Chat, Cowork, GitHub Copilot CLI and a custom agent called Dispute Analyzer. Copilot answers from the bank's business data in Dataverse, follows the business skills (the bank's written dispute rules), uses Fabric analytics to look across a merchant, and drafts emails from Microsoft 365.

## What's in this kit

| File | What it is |
|---|---|
| README.md | This note: the scenario, demo prompts, and setup steps. |
| ZavaDispute_1_0_0_11.zip | The Power Platform solution, with the app, tables, agent and business skills. Import it as is; do not unzip it. |
| shareable-dispute-seed.zip | The package that loads the demo data. Unzip it before use. |

**Solution version 1.0.0.11.** This version removes the unused Dispute Manager app. As a result, the import no longer needs the App Module Assets managed solution. The agents, flows, business skills and tables are the same as in 1.0.0.10.

## 1. The scenario

A cardholder disputes a charge. The dispute team checks the transaction, the customer, the risk signals and the bank's rules, then decides whether to approve, deny or escalate. The demo starts with one dispute, **DIS-1063**: Tailwind Traders (cardholder Lin Chen), $2,800 at **The Phone Company**, high risk band. It then widens to ask whether that merchant has a broader problem.

### What the demo uses

| Component | What it does |
|---|---|
| **Dispute Operations** app | The analyst's app. It covers card transactions, disputes, fraud signals, dispute tasks and decision logs. |
| **Dispute Analyzer** agent | The custom investigation agent. It's built in the new Copilot Studio (GitHub Copilot harness) on the Work IQ MCP Server with the business applications skill. It reaches Dispute Operations, Customer Service Hub and Microsoft 365, plus a Fabric data agent called **FraudDetector** that totals fraud across a merchant's transactions. |
| **Business skills** | The bank's rules, stored in Dataverse. See the list below. |
| **"Red Flag" business term** | Disputes with status Escalated, amount over $1,000, and a Reg E due date within 7 days or already past. A glossary entry also maps "RF" to Red Flag. |

**Where to find each item after import.** Everything in the demo comes from the solution in this kit. After you import it, open each item in your own environment:

| Item | Where to find it |
|---|---|
| Dispute Operations app | Power Apps, **Apps**, then **Dispute Operations**. Choose **Edit** to change it. |
| ZavaDispute solution | Power Apps, **Solutions**, then **ZavaDispute**. |
| Dispute Analyzer agent | Copilot Studio, **Agents**, then **Dispute Analyzer**. You can also open it from the ZavaDispute solution. |
| Auto Dispute Triage workflow | Copilot Studio, **Flows**, then **Auto Dispute Triage**. You can also open it from the ZavaDispute solution. |

**Business skills stored in Dataverse:**
- answer-dispute-questions (activate this if you want to use or test Agentic Search specifically)
- evaluate-dispute
- handle-repeat-disputer
- record-dispute-decision
- draft-customer-notice

**Names business users and agent instructions can use:** Business users and agent instructions can simply refer to the four names below. Each one maps to the business skill in Dataverse that does the work.

| Name to use | Business skill in Dataverse | What it does |
|---|---|---|
| Triage Dispute | evaluate-dispute | Checks whether a filed dispute can be resolved without an analyst. It isn't eligible if the customer is Platinum, the amount is over $5,000, the fraud risk band is High, the customer has 3 or more disputes in 90 days, the evidence conflicts, or confidence is below 0.90. |
| Evaluate Dispute | evaluate-dispute | Recommends Approve, Deny or Escalate, with a confidence score, reasoning and policy citation. It doesn't change any records. |
| Apply Dispute Resolution | record-dispute-decision | Records an Approve or Deny outcome: sets the decision and status, writes a decision log entry, and creates the follow-up tasks. |
| Escalate to Exception Queue | record-dispute-decision | Records an Escalate outcome: sets the status to Escalated and creates a "Review and Decide" task for the assigned analyst. For a repeat disputer, handle-repeat-disputer builds the 90-day pattern context first and then uses record-dispute-decision. |

**The four names are also skill records in the solution, inactive by design.** Decision log entries point to them, and the data setup checks that they exist. Keep them inactive. The work itself runs through the active business skills listed above.

**Repeat-disputer rule:** A repeat disputer has 3 or more disputes in 90 days. These cases always go to an analyst, whatever the amount, customer tier or confidence score. Every skill that checks for repeat disputers uses the same 90-day window.

**Personas in the data:** Adele Vance, Nestor Wilke and Megan Bowen are the analysts.

### Work in progress: auto dispute triage

**Auto Dispute Triage** is an agent workflow in Copilot Studio. It's included in the solution but still being built, so it isn't part of the demo script yet.

- **What it does:** When a dispute is filed, an agent runs Evaluate Dispute, then Record Dispute Decision, through the Work IQ MCP Server. It auto-resolves the dispute only if the case is eligible, confidence is at least 0.90, and the recommendation is Approve or Deny. Everything else goes to an analyst with a reason (for example "Platinum customer" or "Confidence < 0.90") and a "Review and Decide" task.
- **Asking a person:** The agent can ask a named person for more information through an Outlook approval request.
- **Setup:** See Step A for connections and the settings to change after import.

### In the solution but not part of the demo

- **Dispute Assistant and Auto Dispute Resolution agents.** These are legacy agents from the classic Copilot Studio, built on the Dataverse MCP Server. They are included for reference only. Don't demo them.

## 2. Demo prompts

The prompts are listed in demo order. Each surface adds capability to the one before it.

### Stage 1: Copilot Chat

These prompts also work in Cowork, Dispute Analyzer and GitHub Copilot CLI.

| # | Prompt | What it shows |
|---|---|---|
| 1 | *show red flag disputes* | Copilot uses the bank's own definition of "red flag" from Dataverse instead of making one up. |
| 2 | *check in customer service hub app, are there any cases associated to the customer for dis-1043?* | One question spans two apps. DIS-1043 belongs to Fabrikam Inc (Maria Campbell), and Copilot finds that customer's cases in Customer Service Hub. |
| 3 | *summarize the analysis and draft an email to Adele Vance* | Business data turns into Microsoft 365 work. Copilot drafts the email for the analyst and does not send anything to the customer. |

### Stage 2: Cowork or GitHub Copilot CLI

These prompts run the bank's business skills, then show and change the logic behind them.

| # | Prompt | Also works in | What it shows |
|---|---|---|---|
| 4 | *evaluate dis-1063* | | Copilot returns Approve, Deny or Escalate with a confidence score, reasoning and policy citation. A High fraud risk band always means Escalate. |
| 5 | *Check whether DIS-1045 belongs to a repeat disputer* | Dispute Analyzer | DIS-1045 is Fourth Coffee (Sam Centrell), $950, with several earlier disputes. The handle-repeat-disputer skill applies the 90-day rule. |
| 6 | *did you use business skill to analyze the repeat disputer?* | Dispute Analyzer | Copilot shows which rule it followed, so the answer can be checked. |
| 7 | *analyze which disputes are with repeat disputers? and show me the logic* | | Repeat-disputer patterns across all disputes, with the rule and date ranges shown. |
| 8 | *if we were to change the rule to 120 days, would it change the analysis?* | | A what-if against the official 90-day rule, with no records changed. |
| 9 | *help me plan changes required in the related business skill to update the logic to 120 days. don't make changes yet* | | Business skills are Dataverse records a maker can edit. Copilot plans the change without making it. |

### Stage 3: Dispute Analyzer agent

This prompt combines business data with Fabric analytics.

| # | Prompt | What it shows |
|---|---|---|
| 10 | *What is our total fraud exposure on The Phone Company right now, including transactions that haven't been disputed yet?* | The FraudDetector data agent totals exposure across all of The Phone Company's transactions, including ones nobody has disputed yet. Business data alone can't answer this. DIS-1063 is one of this merchant's disputes, which brings the story back to where it started. |

## 3. Restore the solution and run the seed data

### Before you start

- Use a **dedicated non-production** environment that you're allowed to change. A full reseed deletes every row in the five Zava tables. The seed package doesn't block any environment; it relies on you to confirm the target environment and each change.
- The environment needs Business Applications in Work IQ and Copilot Studio.
- Prompt 2 needs Dynamics 365 Customer Service, for the Customer Service Hub app.
- Prompt 10 needs your own Fabric workspace with a data agent.
- Cowork and GitHub Copilot CLI need the Work IQ business applications skill.

**Required managed solutions.** The import fails if any of these are missing from the target environment:

| Display name | Name | Version |
|---|---|---|
| msdyn_RelevanceSearch | msdyn_RelevanceSearch | 1.0.3517.3 |
| Base Custom Controls Core | BaseCustomControlsCore | 9.0.2609.4004 |
| Custom Controls Core | CustomControlsCore | 9.2.26101.00138 |
| Theming Settings | msdyn_PowerAppsThemingSettings | 1.0.0.13 |
| App Framework Infra Extensions | msdyn_AppFrameworkInfraExtensions | 1.0.0.27 |
| M365 Copilot App Infra Settings | msdyn_M365CopilotAppInfraSettings | 9.2.0.9 |

### Step A: Import the solution

1. Go to Power Apps, then **Solutions**, then **Import**, and choose ZavaDispute_1_0_0_11.zip. It's an unmanaged solution.
2. Map the connections when asked. The demo needs Dataverse, Work IQ MCP and the Fabric Data Agent. Auto Dispute Triage also needs Agent and Advanced Approvals. The legacy agents ask for Copilot Studio and Microsoft 365 MCP connections; map them so the import completes.
3. **Turn all three flows off before loading data.** Each one runs when a dispute is created or changed, so loading 45 disputes would start 45 runs of each.
   - **Auto Dispute Triage** (work in progress). If it's on when you add DIS-1063, it can resolve or escalate the case before the demo starts, so DIS-1063 is no longer Filed and Pending. Leave it off unless you're testing triage.
   - **"When a row is added, modified or deleted"** sends new disputes to the legacy Auto Dispute Resolution agent. Leave it off. Running it alongside Auto Dispute Triage would process each dispute twice.
   - **Repeat-Disputer Pre-Check.** Keep it off while you load data.
4. In the Dispute Analyzer agent, point the FraudDetector tool to your own Fabric data agent. The export still points to a Fabric workspace you can't reach.
5. If you plan to test Auto Dispute Triage, open it in Copilot Studio and check two settings:
   - **Who gets information requests.** The export sends requests to a specific person in the tenant the solution came from. Change it to someone in your tenant.
   - **The Work IQ MCP tool.** Check that it uses your connection. It points to a specific connection rather than one of the solution's connection references, so it may not update on import.
6. Check which business skills are active. The solution file doesn't record whether a skill is active, so confirm this after import:
   - Triage Dispute, Evaluate Dispute, Apply Dispute Resolution and Escalate to Exception Queue are inactive.
   - answer-dispute-questions is active only if you want to use or test Agentic Search.
7. Check the **Dispute Operations MCP Server** in the solution. Its server address carries the ID of the environment the solution was exported from. If it doesn't update to your environment on import, change it to your environment ID.
8. Publish the Dispute Analyzer agent, share the Dispute Operations app, and give the analyst users their roles.

### Step B: Load the data

1. Unzip shareable-dispute-seed.zip and open the folder in a coding agent, such as GitHub Copilot CLI, VS Code, or Claude Code. Type **"reseed data"**. The agent walks you through setup and asks you to confirm before it changes anything.
2. To do it by hand instead:
   - Create a virtual environment: `python -m venv .venv`, then `.\.venv\Scripts\python.exe -m pip install -r requirements.txt`. Python 3.10 or later is required.
   - Create `setup.local.json` with your Dataverse URL, tenant ID, sign-in account, and the users who stand in for Adele, Nestor and Megan. You can map all three to yourself.
   - Run `seed_demo.py configure --confirm-url "<your URL>"` and complete the device-code sign-in. Setup checks the environment, the solution tables and the four inactive skill records (Triage Dispute, Evaluate Dispute, Apply Dispute Resolution and Escalate to Exception Queue).
   - Run `seed_demo.py prepare seed` to preview, then `apply --plan "<plan_id>" --confirmation "<your acknowledgment>"`. A preview expires after 15 minutes.
   - Run `seed_demo.py validate` to check the result.
3. **Expected baseline:** 45 disputes, 144 transactions, 48 fraud signals, 101 tasks, 60 decision logs, 10 accounts and 12 contacts.
4. Add the demo case separately: `prepare reseed --case DIS-1063`, then apply. DIS-1062 and DIS-1064 work the same way. Each one is recreated with today's date, as Filed and Pending.

The seed data does **not** create Customer Service Hub cases. For prompt 2, add a case for Fabrikam Inc yourself.

The Red Flag view depends on today's date, so its results change with the date the data is loaded.

## Known gaps

None of these block the import or data load.

- **Auto Dispute Triage: unfilled amount limit.** The rule for "High-value case" still says "Dispute amount > [threshold]". Replace [threshold] with a number, such as 5000 to match Evaluate Dispute.
- **Auto Dispute Triage: runs on edits too.** The workflow starts when a dispute is added or changed, but the agent is told to act only on newly filed disputes. Editing a dispute that is still Filed could be escalated as "Invalid trigger." Consider triggering on add only.

- **Personal skill.** handle-repeat-disputer is marked as a personal skill. Prompts 5 and 6 use handle-repeat-disputer. If someone other than the person who imports the solution runs the demo, check that they can see that skill.
- **Missing skill for prompt 7.** The Dispute Analyzer calls an "Analyze Repeat Dispute Patterns" skill, which isn't in the solution. This only matters if prompt 7 is run in Dispute Analyzer instead of Cowork or GitHub Copilot CLI.
- **Possible extra dependency.** The solution file also lists Omnichannel Telephony as a dependency for Contact and User, but the dependency screen doesn't show it. Test the import in a new environment to confirm.
