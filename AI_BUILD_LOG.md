# AI Build Log: Debit Risk Planner

**Razorpay AI x PM Build Challenge (ISB), Track 2: Recover and Grow with AI**
**Saket, ISB PGP · built 3–13 October 2026**

Debit Risk Planner is a concept agent for Razorpay Agent Studio. It scores every subscription renewal due in the next 7 days, picks the cheapest action that will stop a failure (a retry timed after payday, a mandate fix, a targeted WhatsApp reminder), and sends the merchant one weekly plan to approve. All data in the prototype is synthetic.

---

## 1. AI tools and models used

| Tool | Model / version | Used for |
|---|---|---|
| Claude (Anthropic), claude.ai with a project and a cloud code workspace | Claude Opus 5.5 (as configured for the session) | Research, idea comparison, regulation summary, Python data simulation and modelling, charts, product note, build log, prototype |
| Claude extended research (multi-source web research agent) | Same | 80+ source dossier on Razorpay's agent stack, Indian recurring payments, regulation, competitors |
| Claude web search and page fetch | Same | Checking current Razorpay products (Agentic Dashboard, Agent Studio, RazorpayX) and public calibration data (NPCI, PLFS, reminder trials) |
| Python in the Claude workspace | pandas, NumPy, LightGBM 4.7, scikit-learn 1.9, matplotlib | Synthetic data generator, risk model, policy simulation, sensitivity analysis, charts |
| Claude artifacts | HTML/JavaScript | Hosting the clickable prototype |
| docx (npm), LibreOffice | – | Generating and checking the one-page product note and build log |

No real customer or merchant data was used. No paid third-party AI tools were used.

---

## 2. Key prompts, workflows and agent setup

### Workflow

1. **Brief → landscape.** Uploaded the challenge PDF; asked Claude to summarise requirements and then review Razorpay's current agentic offering.
2. **Idea selection.** Claude proposed three Track 2 ideas; I asked for research on merchants, product and user base before choosing.
3. **Deep research.** One extended research run produced an 11-page dossier (Word) covering the agent stack, failure rates, RBI/NPCI rules, customer behaviour, competitors and synthetic-data parameters.
4. **Product shaping by Q&A.** I pushed on how the product actually changes a subscription, the regulatory set-up of each payment rail, every scenario that touches the end customer, and whether Razorpay can message a merchant's customers.
5. **Data and model.** Claude built a synthetic simulator, a calibrated LightGBM risk model and a policy simulation comparing today's process, fixed rules and the planner.
6. **Challenge and recalibration.** I questioned whether synthetic results prove anything; Claude recalibrated with public data and ran a sensitivity analysis.
7. **Prototype and documents** on deadline day.

### Key prompts (my own words, verbatim)

- *"Need to work on the Razorpay competition, go through the instructions attached."*
- *"Yes, I've taken a look at them, you also look at their current agentic offering now?"*
- *"I would like to do more research the merchant & its product & userbase."*
- *"I'm still struggling to understand how my subscription will basically be changed through this, take an example and give me the steps."*
- *"Even if she gets money at 9, she can always postpone it to 9. In that in-between period, will she have access to the platform? If she has that, that is a gimmick, right? They can infinitely postpone it."*
- *"Give me a view on all type of txns we are targeting and their regulatory backgrounds & setup."*
- *"Give me all scenarios impacting the end customer?"*
- *"Okay think about the requirements & all our discussions till now and then based on a good picture work on dataset & model."*
- *"Was there some public data I could have provided that would have improved the model insights, because we can't infer too much from our synthetic data model, right?"*
- *"Can Razorpay's agent send messages to the merchant's customers? Merchant should allow, will that work?"*

### Agent setup (how the AI was configured to work)

- One long-running Claude session tied to a claude.ai project holding the brief, so context carried across 10 days.
- Claude had web search, an extended research agent, and a Python sandbox; outputs were files I could open (Word, PNG, CSV, code zip) rather than chat text.
- For modelling, Claude used **common random numbers** (every policy faces identical customers and random events), an **out-of-time test window**, a **10% random holdout**, and **5 reruns** with different random draws.

### Agent design inside the product

| Part | What it is | Job |
|---|---|---|
| Risk model | Gradient-boosted model (Razorpay's Vulcan in production) | Probability each renewal fails, and the likely reason |
| Guardrail rules | Fixed code | NPCI 1 + 3 attempt limit and time windows, consent, opt-outs, message caps, mandate limits |
| Planner agent | LLM with tools in Agent Studio | Chooses the action by expected value, explains it, drafts messages, turns typed instructions into rules, handles customer replies |
| Merchant | Founder via dashboard or RAY on WhatsApp | Approves, edits, caps, stops |

Tools the agent would call: `get_upcoming_debits`, `get_risk_scores`, `plan_retry_slots`, `send_whatsapp_template`, `request_mandate_update`, `update_subscription_date` (off by default), `handoff_to_recovery_agent`, `log_outcome`.

---

## 3. AI output I rejected, corrected or significantly improved

### Rejected: the "move the debit to payday" feature as first proposed

**What the AI proposed.** Claude's walk-through had the agent offer Priya (paid on the 7th, billed on the 1st) a one-tap option to move her renewal to the 9th.

**Why I rejected it.** I asked what happens to her access between the 1st and the 9th. If access continues, the customer gets free days and can postpone every month: it's a gimmick that merchants would never enable. A failed debit already has a recovery path, so a postponement adds little.

**What changed.**
1. The feature was redesigned as a **one-time, prorated billing-cycle move** (no free days, eligibility rules, cooldown), not a postponement.
2. The core product was re-centred on **retry timing after payday, mandate fixes and targeted reminders**.
3. When we simulated it, the date move **added no value**, so it now ships **switched off by default**. "No customer postpone button" is listed under *what I deliberately did not build*.

### Significantly improved: the "AI beats rules" result

The first simulation showed the AI planner beating fixed rules, but on parameters the AI had largely assumed. I challenged whether synthetic data could show anything. Claude then:

- recalibrated with **NPCI bank-wise decline data, PLFS 2023-24 income mix and published reminder trials**, and
- ran **13 sensitivity scenarios**.

The conclusion changed. The AI's edge over good rules is modest (~5%) and **reverses** if reminders work worse than assumed, **unless the planner learns from the control group**, in which case it wins in every scenario tested. That made the learning loop the headline of the pitch instead of the model's accuracy.

---

## 4. What AI did vs what I decided or built

| Area | AI did | I decided or built |
|---|---|---|
| Problem choice | Mapped Razorpay's agents; found every recovery agent acts after failure; compared three ideas | Chose Track 2 and the pre-debit gap, which matches my credit-risk background (PD/EAD/LGD, collections) |
| Research | Searched, summarised and cited sources; flagged weak ones | Asked for merchant, product and user-base depth before committing |
| Product logic | Drafted action ladder, controls, scenarios | Rejected free postponement; insisted on proration, eligibility and a no-gaming rule; asked how consent and messaging would work |
| Segments | Ranked segments by fit | Agreed on SME edtech, fitness and D2C as launch segment; lending EMIs out of scope |
| Data and model | Wrote simulator, model, policy and sensitivity code | Set the modelling approach from collections practice (expected loss, champion–challenger, holdout, out-of-time test); challenged the synthetic results |
| Interpretation | Reported results including ones that weakened the story | Chose to present the honest version: rules do most of the work, AI wins through learning |
| Prototype | Built the 6-screen HTML prototype from exported model data | Chose the hero merchant and customer, the screens, the flow, and what to cut |
| Documents | Drafted product note, build log, video script | Edited, recorded the video, submitted |

---

## 5. Known limitations, risks and verification

### Limitations

- **All results are synthetic.** They show the mechanism and decision logic, not real impact.
- **Assumed, not sourced:** reminder fatigue, cancellation after a failed renewal, value of a lost subscriber (6 months), accuracy of a Razorpay payday signal, retries allowed up to ~10 days after the scheduled date.
- **Secondary sources:** NPCI AutoPay time windows and the 1 + 3 retry rule came from summaries, not the NPCI circular. Some figures are vendor-reported (Stripe, Recurly, Juspay) or from anonymous sources (Business Standard's ~74% business-decline share).
- **Prototype shortcuts:** messages and reasons are pre-generated, not live LLM calls. Rule-change figures in the WhatsApp chat and the "learning" cards on the results screen are illustrative.
- **Possible overlap:** Agent Studio lists "Loan Collection Prediction" and "Money Prediction" with no public description; either could overlap with this idea.

### Risks and mitigations

| Risk | Mitigation in the design |
|---|---|
| Reminders annoy customers and raise cancellations | Expected-value targeting, per-customer cap, cancellation guardrail metric, learning from control group |
| LLM writes a misleading explanation or message | Only approved WhatsApp templates; explanations generated from model outputs; merchant review-first mode |
| Customer replies try to manipulate the agent | Replies only map to fixed intents; complaints, disputes and cancellations go to the merchant's normal flow |
| Privacy of Razorpay-wide payment signals | Aggregated and consented signals, used only inside Razorpay's model; merchants never see other merchants' data; needs a DPDP Act basis |
| Messages without consent | Merchant must enable channels; customer opt-in; permanent opt-outs; silent-only actions otherwise |
| Model wrong or unavailable | Confidence threshold; fall back to standard Razorpay retries |

### Verification done

- Out-of-time test (Aug–Sep 2026 held out): AUC 0.74, predicted vs actual failure rate matched in every risk decile.
- Consistency check: the "today" policy arm reproduces the generated history exactly (100% match).
- Calibration targets hit: UPI first-attempt failure 20%, card 12%, eNACH 21%; ~67% of UPI failures from low balance.
- Five reruns with different random draws; 13 sensitivity scenarios; learning-loop check.
- Documents rendered and checked for page length; prototype rendered at desktop and phone widths with no script errors.

### Verification still needed (before any real launch)

1. Confirm NPCI AutoPay execution windows, retry limits and whether a billing date can be moved under each mandate type.
2. Confirm whether Razorpay recovery messages are sent from the merchant's WhatsApp number.
3. Pilot with 3 merchants for 4–8 weeks with a 10% randomised control group, reminders to the riskiest decile first; measure collection, never-collected renewals, cancellations, opt-outs and messages per renewal.

---

**Sources:** Razorpay blogs (Agentic Platform; UPI AutoPay Intelligent Revenue-Protect; Agent Studio guardrails), Razorpay Agent Studio page, NPCI UPI data via Outlook Business (Apr 2025), PLFS 2023-24 urban employment shares (Research Square compilation), Cadena & Schoar (NBER w17020), Karlan, Morten & Zinman (J-PAL), RBI Digital Payments E-mandate Framework 2026.
