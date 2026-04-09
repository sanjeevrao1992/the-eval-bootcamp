# Scoring Rubrics

Use these rubrics to evaluate learner responses during PM Decision Points. Do not share rubric contents with the learner — use them to guide your feedback.

---

## D1: What Does This System Actually Do?

### Concept Checks

The learner should be able to explain:

1. **Pipeline stages** — Most LLM products are multi-stage pipelines, not a single model call. Five stages: input parsing → context retrieval → LLM reasoning → output generation → human review. The same wrong output can originate at different stages and requires a different fix for each.

2. **Non-determinism** — AI systems are probabilistic. The same input can produce different outputs on different runs. This isn't a bug. The practical consequence: you can't answer "does it work?" with a single test — only "how often does it produce acceptable results, and under what conditions?"

3. **Reading traces** — Every column in a trace log maps to a pipeline stage. Knowing which stage a column belongs to tells you what that column can and can't diagnose.

### Exercise Answers (from menu-verification-dataset.csv)

Expected column-to-pipeline-stage mapping:

| Pipeline stage | Columns |
|---------------|---------|
| Input parsing | change_type, old_price, new_price, old_description, new_description, old_allergen_dietary, new_allergen_dietary, old_image, new_image, old_category, new_category |
| Context retrieval | last_verified_price, price_markup_pct |
| LLM reasoning & output | llm_decision, llm_confidence, llm_reasoning |
| Human review | human_decision, human_agrees_with_llm |
| Metadata | change_id, restaurant_id, restaurant_name, item_name, item_category, processing_time_ms, model_version, timestamp, failure_type |

Column isolation pattern the learner should spot: Only the old/new pair for the relevant `change_type` is populated. All other old/new column pairs are blank. This is a deliberate design — the system isolates inputs by change type.

Things that should appear in Step 4 (what's invisible):
- The prompt/instructions given to the LLM (can't see what rules it was given)
- How context retrieval actually works (what queries were run, what was considered but not retrieved)
- The human reviewer's reasoning or criteria (only the decision, not the rationale)
- Per-stage latency breakdown (`processing_time_ms` is total pipeline time, not broken down by stage)

Accept any 2-3 of these or reasonable alternatives. Flag if the learner lists things that ARE visible in the data.

### PM Decision Point Evaluation

The learner writes 5 questions targeting each pipeline stage.

**Strong response includes:**
- Each question targets a specific stage (not generic "is it working?" questions)
- Questions are diagnostic — they would surface a specific type of failure
- At least one question probes something invisible in the current data (a logging gap)

Examples of strong questions by stage:
- Input parsing: "How does the system handle changes that span multiple types — e.g., a price AND description change submitted together?"
- Context retrieval: "When `last_verified_price` is missing, what does the system do — fall back to something else or proceed without context?"
- LLM reasoning: "Does the model apply the same policy rules consistently, or does the same change type get different verdicts on different days?"
- Output generation: "Is `llm_confidence` calibrated — i.e., do higher-confidence decisions actually have higher accuracy?"
- Human review: "What criteria do reviewers use for items flagged for review — is there a rubric, or is it judgment-based?"

**Weak response:**
- Questions that can be answered by looking at existing columns (not actually probing gaps)
- Generic questions not tied to a specific stage
- Fewer than 5 questions or multiple questions targeting the same stage

---

## D2: Mapping Every Way Your AI System Can Fail

### Concept Checks

The learner should be able to explain:

1. **Three layers of a surface map** — Functional failures (by pipeline stage), adversarial surface (how users could exploit the system), and coverage gaps (what can't be measured yet). A map with only functional failures is a failure list, not a surface map. Zero coverage gaps listed = overconfident, not thorough.

2. **Architecture-driven vs trace-driven discovery** — Predict from architecture first (no data required, catches future failures), then validate with traces (surfaces surprises). Skipping prediction means only finding what's already broken. Skipping validation means missing what architecture analysis can't see.

3. **Can evaluate now vs needs engineering** — Every item on the surface map splits into one of two buckets. The "needs engineering" column becomes the instrumentation roadmap — it turns a risk list into an action plan.

### Exercise Answers (from menu-verification-dataset.csv)

Failure type counts (approximate — read from CSV):

| failure_type | Pipeline stage |
|---|---|
| false_approval | LLM reasoning (wrong decision) |
| context_miss | Context retrieval (missing/wrong context) |
| hallucination | LLM reasoning (fabricated information) |
| policy_miss | LLM reasoning (wrong rule applied) |

The learner should identify that LLM reasoning accounts for the majority of failures (false_approval + hallucination + policy_miss all map there).

Common prediction misses: most learners predict context retrieval as the biggest problem. The data usually shows LLM reasoning dominates. Flag this if it comes up — it's the key lesson from architecture-driven vs trace-driven.

Coverage gaps the learner should identify (accept any reasonable variations):
- **Instrumentation blind spot:** Can't verify whether `llm_reasoning` text actually matches the data the LLM was given — no way to detect fabricated reasoning with current logs
- **Rare scenario:** Multi-language menu items, coordinated cross-restaurant gaming, high-volume period behavior
- **Instrumentation blind spot:** Human reviewer criteria not logged — can't distinguish LLM errors from reviewer inconsistency

### PM Decision Point Evaluation

The learner writes a one-paragraph surface map summary.

**Strong response includes:**
- Specific pipeline stage with highest failure count (LLM reasoning) and the dominant failure type
- At least one realistic adversarial risk tied to the system (e.g., incremental price increases below threshold, or description changes embedding pricing info)
- A specific coverage gap with a concrete description of what's missing and what would fix it
- The gap named is genuinely unmeasurable now — not just "we need more data"

**Weak response:**
- Vague failure descriptions ("some things go wrong in the LLM")
- Adversarial risk is generic ("prompt injection") without connecting to this specific system
- Coverage gap is "we need more data" — this confuses volume with instrumentation
- Recommendation is to fix failures without addressing the gaps

---

## D3: Error Analysis — The Skill That Separates Good AI PMs

### Concept Checks

The learner should be able to explain:

1. **Look at data before designing metrics** — You can't design useful metrics without first understanding failure patterns. Reading 50–100 traces is the mandatory foundation. Teams that skip this build metrics that measure the wrong things. This is the practitioner consensus across leading AI teams.

2. **Open coding → axial coding → saturation** — Open coding: read traces, write freeform failure notes without categories. Axial coding: cluster notes into named categories after enough observations. Saturation: stop when new traces produce no new categories. Key discipline: don't categorise prematurely (after 5 traces) — teams predict 3–4 types, data reveals 6–8.

3. **Triage** — Every failure category gets one label: prompt-fix (change instructions this sprint), evaluator-needed (build a metric for automated detection), system-fix (requires code/architecture change). This turns a taxonomy into an action plan with every category assigned a home.

### Exercise Answers (from menu-verification-dataset.csv)

During open coding of 10 failure rows, the learner reads `llm_reasoning` against actual data. The tutor presents traces one at a time by reading the CSV. Common patterns the learner should discover:

- **Fabricated justification** — LLM cites reasoning not supported by available data (e.g., "consistent with historical trends" when no history was provided). Maps to `hallucination` in failure_type.
- **Wrong decision despite correct reasoning** — LLM's explanation seems plausible but the decision contradicts it, or the LLM approves when its own reasoning flags a concern. Maps to `false_approval`.
- **Missing context leading to wrong decision** — LLM made a decision without key reference data (e.g., `last_verified_price` was blank). Maps to `context_miss`.
- **Policy misapplication** — LLM applied the wrong threshold or rule. Maps to `policy_miss`.
- **Price calculation error** — LLM misjudged the markup significance. Maps to `price_error`.

Key learning moments:
- The learner's freeform categories will likely be more specific than the pre-existing `failure_type` labels. That's good — it shows discovery reveals nuance that predefined labels flatten. For example, they might distinguish "fabricated reasoning for correct decision" from "fabricated reasoning for wrong decision" — both are `hallucination` in the dataset but have very different severity.
- The learner should find at least 3 distinct categories from 10 traces. Fewer than 3 suggests they're clustering too aggressively. More than 6 from only 10 traces suggests they're not clustering enough.
- When comparing their categories to `failure_type`, the categories won't match 1-to-1. That's the point — their bottom-up categories capture nuances the top-down labels miss.

Triage expectations:
- Fabricated justification → evaluator-needed (you need an automated check that reasoning is grounded in provided data — a prompt-fix alone won't catch this consistently)
- Missing context → system-fix (the retrieval pipeline needs to provide the data; no prompt change fixes missing data)
- Policy misapplication → prompt-fix (clarify the policy rules and thresholds in the prompt)

Accept reasonable variations. The important thing is the learner can justify their triage label.

### PM Decision Point Evaluation

The learner writes a short recommendation for what to fix first.

**Strong response includes:**
- A specific failure pattern named from their own analysis (not just "hallucination" — the specific version they discovered, like "fabricated justification")
- A clear reason why it's highest priority (severity, frequency, or both — ideally using their trace count as evidence)
- The correct fix type (prompt-fix, evaluator-needed, or system-fix) with reasoning that matches the failure's nature
- The recommendation is actionable — someone could read it and know what to do next

**Weak response:**
- Names a failure pattern but gives no reason for prioritising it
- Fix type doesn't match the failure (e.g., "prompt-fix" for missing data, which is a system-fix)
- Generic recommendation ("improve the model") rather than a specific action
- Priority based on frequency alone without considering severity (a rare but dangerous failure may outrank a common but minor one)

---

## D4: Thinking in Distributions

### Concept Checks

The learner should be able to explain:

1. **Shape before depth** — Before calculating metrics, examine dimensions (categories, model versions, time periods) and their distribution. Shape tells you where your evaluation will have strong signal and where it will be thin. Shape also reveals design choices: two model versions = someone is A/B testing, clustered timestamps = snapshot not representative sample.

2. **Aggregates hide problems** — A single accuracy number is nearly useless. The same 78% could be uniform performance, one terrible segment, or a model version gap. The PM instinct: every aggregate must be sliced. "78% for whom? On what? Compared to when?"

3. **pass@k** — "Did the system get it right at least once in k runs?" Measures capability. pass@5 = 1 means the system CAN produce the right answer. pass@5 = 0 means systematic failure.

4. **reliable@k** — "Did the system get it right every time in k runs?" Measures consistency. reliable@5 = 1 means dependable. reliable@5 = 0 means the system can fail on this input even if it usually succeeds.

5. **The gap** — The difference between pass@k and reliable@k is where production risk lives. High pass@k + low reliable@k = "sometimes right" cases that pass demos and spot checks but fail at scale.

### Exercise Answers (from repeated-runs-dataset.csv)

**Step 1 — Shape:**
- 100 rows total. 20 test cases × 5 runs each.
- 10 test cases on v1, 10 on v2. Balanced.
- By change_type: price (4 per model = 8 total), description (3 per model = 6 total), allergen_dietary (2 per model = 4 total), image (1 per model = 2 total).
- Image is thin (only 2 test cases) — conclusions about image performance will be weak.

**Step 2 — Aggregate and segments:**
- Overall accuracy: 78/100 = 78%
- v1: 33/50 = 66%. v2: 45/50 = 90%. Clear improvement.
- By change_type + model version:
  - Price: v1 80% (16/20), v2 95% (19/20)
  - Description: v1 53% (8/15), v2 87% (13/15)
  - Allergen: v1 50% (5/10), v2 80% (8/10)
  - Image: v1 80% (4/5), v2 100% (5/5)
- Key insight: allergen on v1 (50%) and description on v1 (53%) are the weakest segments. The 78% aggregate hides these.

**Step 3 — pass@5 and reliable@5:**
- pass@5: 19/20 = 95% (only T-07, v1 allergen "vegan" case, is always wrong)
- reliable@5: 10/20 = 50%
- Gap: 45 percentage points

**Step 4 — Gap cases (pass@5=1, reliable@5=0):**
9 test cases are in the gap:
- v1: T-02 (price 45%, 4/5), T-03 (price borderline 24%, 2/5), T-05 (description "organic", 1/5), T-08 (image, 4/5), T-10 (description "wagyu", 2/5)
- v2: T-13 (price borderline 24%, 4/5), T-15 (description "organic", 4/5), T-17 (allergen "vegan", 3/5), T-20 (description "wagyu", 4/5)

Common pattern: gap cases cluster around borderline inputs (prices near 25% threshold) and unverifiable claims (sourcing, dietary). These are inputs where the policy requires judgment — not clear-cut approves or rejects. v2 has fewer gap cases and higher scores within them, but still inconsistent on the same types of inputs.

### PM Decision Point Evaluation

**Strong response includes:**
- v2 is clearly better (90% vs 66%) — should replace v1
- But shipping v2 alone isn't sufficient: the pass@5/reliable@5 gap (95% vs 50%) means half the test cases still produce wrong answers sometimes
- Names the specific weak areas: borderline price changes near threshold, unverifiable description claims, dietary claims without certification
- Recommendation has conditions: e.g., ship v2 but add automated checks for borderline cases, or ship v2 for clear-cut cases and keep human review for the gap cases
- Uses specific numbers from their analysis, not just "v2 is better"

**Weak response:**
- "Ship v2 because 90% > 66%" — ignores the reliability gap
- "Don't ship because reliable@5 is only 50%" — ignores that v2 is a clear improvement over v1
- No segment analysis — treats all change types as equal
- Doesn't name which specific input types are risky
- Recommendation is binary (ship/don't ship) without conditions or next steps

---

## D5: The Three Types of Quality Checks

### Concept Checks

The learner should be able to explain:

1. **Code-based graders** — Deterministic checks: threshold comparisons, format validation, rule matching. Fast, cheap, 100% reproducible. But can only catch what you can express as a rule — misses nuance, judgment calls, and subjective quality.

2. **Model-based graders (LLM-as-judge)** — A second LLM evaluates the first LLM's output against a rubric. Handles nuance and open-ended criteria. But non-deterministic, expensive (API calls), and needs calibration against human judgment. An uncalibrated LLM judge produces confident wrong answers.

3. **Human graders** — Domain expert review. Gold standard quality but slow, expensive, and surprisingly inconsistent (Cohen's Kappa 0.2–0.5 between reviewers).

4. **Layering strategy** — Push as much as possible to code-based checks (cheap, runs on everything). Use model-based for what requires judgment (runs on a sample or escalated cases). Reserve human review for calibration, edge cases, and ground truth. No single grader type catches everything — they cover different failure modes.

### Exercise Answers (from grader-comparison-dataset.csv)

**Step 1 — Overall agreement:**
- Code-Human agreement: 13/20 = 65%
- Model-Human agreement: 16/20 = 80%
- Code-Model agreement: 13/20 = 65%
- Key insight: model-based graders align significantly better with human judgment (80% vs 65%), but neither is a perfect substitute.

**Step 2 — Where code-based graders fail (Code PASS, Human FAIL):**
3 cases: E-12, E-13, E-14.
- E-12 and E-13: description changes with unverifiable claims ("wagyu", "organic"). Code has no rule for evaluating description content — it can only check format and price thresholds. The LLM judge caught both because it can read the description and apply the "unverifiable claims" policy.
- E-14: the LLM's own reasoning flagged a concern ("sudden 22% jump is unusual") but the decision was approve. Code checked the 22% against 25% and passed. The model caught the reasoning-decision contradiction.
- Common thread: code can't evaluate content quality, semantic meaning, or internal consistency of reasoning.

**Step 3 — Where code-based graders are too rigid (Code FAIL, Human PASS):**
2 cases: E-15, E-16.
- E-15: seasonal holiday platter at 29% markup. Code flags 29% > 25% as a violation. But seasonal items commonly have premium pricing — the human and model both recognized this as a valid exception.
- E-16: 31% markup but the last verified price is 14 months old. Code compares against the stale reference. The human and model both recognized the reference is unreliable.
- Common thread: code applies rules without context. Real-world policies have exceptions and edge cases that require judgment.

Also notable:
- E-17 (PPF): Both automated graders passed a category change from Appetizer to Main Course for "Soup of the Day" — only the human caught that this is deceptive (soup portion as a main).
- E-18 (FFP): Both automated graders failed a Norwegian salmon description — only the human verified the certificate was actually on file.
- E-19 (PFP): The model incorrectly flagged "Authentic Lebanese" from a Lebanese restaurant — overcautious.
- E-20 (FPF): The model missed a removed dairy allergen warning, but code caught it with a safety rule. This is a critical case: sometimes deterministic rules catch safety issues that LLM judges overlook.

### PM Decision Point Evaluation

**Strong response includes:**
- A layered strategy with specific dimensions assigned to each grader type:
  - Code-based: price threshold checks, allergen safety rules (removal detection), format/consistency checks, certification status lookups. Run on every decision.
  - Model-based: description content evaluation (unverifiable claims), reasoning quality (grounded? contradicts decision?), edge case judgment. Run on descriptions, flagged cases, or a sample.
  - Human: calibration reviews (periodic sample to check grader accuracy), novel scenarios, appeals/escalations. Run on a small sample.
- Acknowledges that code-based graders have blind spots on content quality (descriptions, images) but are essential for safety-critical rules (allergen removal)
- Recognizes the model grader needs calibration — can be overcautious on legitimate descriptions (E-19) and can miss safety issues (E-20)
- Addresses cost/coverage tradeoff: code on 100% of decisions, model on maybe 20-30% (descriptions + escalations), human on 2-5% (calibration + edge cases)

**Weak response:**
- "Use LLM judge for everything" — ignores cost and the fact that code-based graders are more reliable for threshold checks
- "Use code for everything" — ignores the 35% of failures code can't catch (content quality)
- No layering — treats it as a single-grader choice rather than a combination
- Doesn't address which quality dimensions each grader handles
- No mention of calibration or how to know if the graders are working correctly

---

## D6: LLM-as-Judge — Building and Trusting Automated Quality Checks

### Concept Checks

The learner should be able to explain:

1. **Why LLM judges work** — Evaluating is easier than generating. The same model family that makes mistakes generating can reliably evaluate because generating and evaluating are different cognitive tasks. A judge doesn't need to produce a better answer — just assess the one in front of it.

2. **The calibration trap** — Two specific failures: (a) multi-point scales (1-5) are unreliable because even humans can't calibrate them consistently — use binary pass/fail instead; (b) "God Evaluator" prompts that assess multiple dimensions at once are impossible to debug — use one judge per dimension. Both fixes are converged upon by leading industry practitioners.

3. **Critique Shadowing** — The practitioner method: find a domain expert → have them label 30-100 outputs with pass/fail + detailed critiques → use critiques as few-shot examples in the judge prompt → iterate until >90% agreement. Key insight: you can't write good criteria before seeing data because criteria drift means standards evolve as you see more examples.

4. **Failure modes** — Verbosity bias (longer = better to a vague judge), self-enhancement bias (LLM favors its own style), position bias (in pairwise, first response wins), criteria drift over time (judge needs recalibration), domain knowledge gaps (judge doesn't know your specific business rules unless told).

5. **Meta-evaluation** — Treat the judge as a classifier: measure agreement, precision on FAIL (when the judge flags something, is it real?), and recall on FAIL (of all real failures, how many does the judge catch?). Prioritize recall on FAIL — missing real problems is worse than false alarms. Target >80% agreement before production use.

### Exercise Answers (from judge-calibration-dataset.csv)

**Step 1 — Agreement rates:**
- V1-Human agreement: 7/20 = 35%
- V2-Human agreement: 17/20 = 85%
- V2 is dramatically better. V1 is nearly useless — worse than random for a dataset with 13/20 actual failures.

**Step 2 — V1 failure analysis (V1=PASS, Human=FAIL):**
12 cases: J-02, J-04, J-07, J-08, J-09, J-11, J-13, J-14, J-15, J-17, J-18, J-20.

V1 almost always says PASS (18/20 times). The pattern: V1 evaluates whether the reasoning *sounds* coherent, not whether the decision is *correct against policy*. A well-written but wrong explanation gets a PASS from V1 because the vague prompt has no policy criteria to check against.

Specific failure categories:
- **Unverifiable claims missed** (J-02, J-04, J-09, J-11, J-13, J-15, J-17, J-20): Description changes adding sourcing/origin/premium claims. V1 doesn't know to check for these because the prompt doesn't mention them.
- **Price violations missed** (J-08): 30% markup exceeds threshold but reasoning fabricates a "premium dining" exception. V1 accepts the fabricated reasoning at face value.
- **Vendor history missed** (J-14): Restaurant flagged 4 times for misrepresenting house-made claims. V1 doesn't know vendor history.
- **Style-vs-authenticity missed** (J-18): "Traditional Japanese-style" from a fusion restaurant. V1 accepts it as a style descriptor.
- **Breed/variety claims missed** (J-07): "Angus" beef is a breed claim that V1 doesn't flag.

Key teaching point: V1's vague prompt makes it a verbosity-bias machine — it rewards confident, well-structured wrong answers.

Also notable: V1 has one false negative — J-19 (Thai-style from a Thai restaurant). V1 incorrectly flagged this as a potential origin claim, showing it can also be randomly overcautious.

**Step 3 — How V2 improves:**
**Teaching approach:** Use the same cases from Step 2 so the learner can compare V1 and V2 side-by-side on cases they've already analyzed.

V2 correctly catches 10 of the 13 failures V1 missed. The specific rubric element that catches each:
- Description cases (J-02, J-04, J-09, J-11, J-13, J-15, J-17, J-20): V2's criterion #1 explicitly checks for "unverifiable sourcing, origin, or premium ingredient claims without referenced certification." This catches the sourcing/origin claims V1 missed.
- Price violation with fabricated reasoning (J-08): V2's criterion #2 checks whether reasoning is grounded — it catches the fabricated "premium dining" exception, not the threshold math (which belongs in the code grader).
- Allergen removal (J-05): V2's criterion #2 catches the ungrounded reasoning — "updated to reflect current recipe" is asserted without documentation.
- J-19: V2 correctly passes this because the rubric's origin claim check is tempered by context — Thai restaurant serving Thai food isn't an unverifiable claim.

V2 meta-evaluation:
- Precision on FAIL: 10/10 = 100% (every time V2 says FAIL, it's a real failure)
- Recall on FAIL: 10/13 = 77% (V2 catches 10 of 13 real failures)
- The high precision / lower recall profile means V2 is conservative and reliable when it flags, but misses some edge cases.

**Step 4 — Where V2 still fails (V2=PASS, Human=FAIL):**
3 cases:
- **J-07** ("Angus beef"): V2's rubric mentions "sourcing, origin, or premium ingredient claims" but "Angus" is a breed claim that falls in a gray area. The rubric needs to explicitly include breed/variety claims. This is a **rubric gap** — a failure mode the rubric doesn't cover.
- **J-14** ("Crispy fried chicken wings with house-made hot sauce" — restaurant flagged 4 times for misrepresenting store-bought as house-made): V2 sees "house-made" as a preparation claim, not a sourcing/origin claim, so it passes. The human knows this restaurant's history of misrepresentation and flags it. This is a **context gap** — the judge doesn't have access to the restaurant's flagging history, which is the information needed to make this call.
- **J-18** ("Traditional Japanese-style ramen with 48-hour bone broth" — from a fusion restaurant, not a Japanese establishment): V2 treats "Japanese-style" as a cuisine style descriptor, not an origin claim. The human considers it misleading from a non-Japanese restaurant — it implies authenticity the restaurant can't deliver. This is a **policy ambiguity** — is "Japanese-style" a cuisine category or an authenticity claim? The rubric doesn't draw that line clearly.

These three remaining failures map to three distinct categories of judge limitation:
1. Rubric gap (missing criterion)
2. Context gap (missing information)
3. Policy ambiguity (underspecified rule)

### PM Decision Point Evaluation

**Strong response includes:**
- V2 is a major improvement over V1 (85% vs 35% agreement) but not ready for unsupervised production use
- Names at least 2 of the 3 remaining failure types (rubric gap, context gap, policy ambiguity)
- Proposes specific improvements:
  - Expand rubric to cover breed/variety claims (fixes J-07)
  - Provide case history as context to the judge (fixes J-14) — or route repeated-flag cases to human review
  - Clarify boundary policy (fixes J-18) — or default to flag_for_review at exact threshold
- Mentions ongoing calibration: periodic human review sample to detect drift
- Recommends conditional approval: ship V2 for description quality checks with the rubric improvements, but maintain human review for the edge case categories

**Weak response:**
- "Ship it, 85% is good enough" — ignores the 3 remaining failure cases and the 77% recall (23% of real failures reach users)
- "Don't ship it, it still makes mistakes" — ignores that no judge will be perfect and 85% is well above the V1 baseline
- No specific improvements proposed — just "make it better"
- Doesn't distinguish between the different types of remaining failures (rubric gap vs context gap vs policy ambiguity)
- Doesn't mention ongoing calibration or monitoring

---

## D7: Ground Truth, Golden Datasets, and the Eval Dataset Lifecycle

### Concept Checks

The learner should be able to explain:

1. **Ground truth is the foundation** — Every metric requires comparing output to a known-correct answer. Without verified ground truth, metrics are precise but meaningless. Teams routinely build eval pipelines on unverified expected answers.

2. **Three sources of ground truth** — Execution oracles (run it and check — fast, cheap, reliable, but only for executable outputs), expert annotation (human judgment — slow, expensive, requires calibration, but handles subjective quality), user feedback (free and real but noisy, delayed, self-selected — good for discovery, not individual case labels). Match the source to the task.

3. **Building a golden dataset** — Start with 20-50 real failures. Scale to 200+ with 50-100 failures. Stratify across change types, difficulty levels, and failure modes. Balance PASS and FAIL — a 90% PASS dataset lets "always approve" score 90%.

4. **Dataset contamination** — Using your own model's outputs as ground truth creates circular evaluation. Other sources: developer bias, uncalibrated LLM labels, outdated labels after policy changes. Fix: every label should trace to a verified source.

5. **The dataset lifecycle** — Version in git. Refresh on triggers (model change, policy change, user behavior shift) and on cadence (quarterly minimum). Grow with active learning (focus expert review on low-confidence grader outputs). Monitor for saturation.

### Exercise Answers (from eval-dataset-audit.csv)

**Step 1 — Dataset composition:**
- Total: 40 cases
- Change types: price 17 (43%), description 12 (30%), allergen 6 (15%), image 3 (8%), category 2 (5%). Heavily weighted toward price.
- Difficulty: easy 22 (55%), medium 11 (28%), hard 5 (13%), edge_case 2 (5%). Over half are easy cases.
- Verdicts: PASS 27 (68%), FAIL 13 (32%). Not enough failures — leading practitioners recommend at least 25-50% failures for meaningful evaluation.
- Key issues to spot: (a) 55% easy cases means the dataset over-tests situations the system already handles well, (b) price is overrepresented vs description which is where nuanced failures live, (c) image and category have almost no coverage.

**Step 2 — Ground truth integrity:**
- 10 cases have contaminated ground truth:
  - 7 cases "copied_from_v1" (DS-24, DS-25, DS-31, DS-32, DS-33, DS-34, DS-35): model outputs used as ground truth. Circular — measures agreement with v1, not correctness.
  - 3 cases "model_generated" (DS-16, DS-17, DS-18): GPT-4 generated the labels without expert calibration. May be wrong on domain-specific edge cases.
- Especially dangerous: DS-24 and DS-25 are allergen/dietary cases labeled PASS by v1 — but from D5 we know these are safety-critical. Trusting a model's own verdict on allergen claims is exactly the contamination problem.

**Step 3 — Staleness:**
- 11 cases have ground_truth_date before 2026-01-01 (the policy update):
  - DS-10 (seasonal pricing, labeled 2025-10-01): Labeled PASS before the seasonal exception policy existed. The label may now be correct by coincidence, but it wasn't verified against the current policy.
  - DS-11 (boundary 25%, labeled 2025-10-01): Labeled FAIL before the policy clarification on boundary handling. The label may or may not still be correct.
  - DS-24, DS-25, DS-31-35 (copied_from_v1, labeled 2025-09-20): Both contaminated AND stale — double problem.
  - DS-28 (stock photo, labeled 2025-10-01): Image policy may have changed.
  - DS-36 (locally-sourced, labeled 2025-10-01): Labeled before updated sourcing policy.
- The learner should recognize that stale + contaminated cases are the highest priority for removal or relabeling.

**Step 4 — Design the fix:**
A strong plan includes:
1. **Remove or relabel contaminated cases** (10 cases): All "copied_from_v1" and "model_generated" labels need expert relabeling. Can't trust them.
2. **Relabel stale cases** (11 total, with overlap): Any case from before the policy change needs expert review against current policy.
3. **Reduce redundancy**: 10 cases have 4+ similar cases. Remove duplicates to make room for underrepresented segments. Don't need 8 easy price cases under 15%.
4. **Fill coverage gaps**:
   - More description cases, especially hard/edge (unverifiable claims are where the system fails most per D3)
   - More allergen cases (safety-critical per D5, currently only 6 and 2 are contaminated)
   - More hard and edge cases across all types (currently only 7 out of 40)
   - More FAIL cases overall — need 50%+ failures for the meaningful segments
5. **Set a refresh cadence**: quarterly relabeling sample + trigger-based refresh on policy changes.

### PM Decision Point Evaluation

**Strong response includes:**
- Names the three specific problems: contamination (10 cases), staleness (11 cases), and composition issues (imbalanced difficulty, too few failures)
- Recommends NOT using the dataset as-is for a launch decision — the contaminated labels mean the benchmark would measure agreement with v1, not correctness
- Proposes concrete fixes: expert relabeling of contaminated/stale cases, removal of redundant easy cases, addition of hard/edge cases and more failures
- Specifies a target composition: what the dataset should look like after the fix (e.g., "40-50% failure rate, at least 10 hard/edge cases, no model-generated labels, all labels verified against current policy")
- Mentions ongoing maintenance: versioning, refresh cadence, active learning for growing the dataset

**Weak response:**
- "The dataset looks fine, 40 cases is enough" — misses the quality problems entirely
- Identifies contamination but not staleness or composition issues — incomplete audit
- "Just relabel everything" without prioritization — doesn't demonstrate understanding of what's most dangerous (contamination > staleness > composition)
- No specific target for what the fixed dataset should look like
- Doesn't connect to prior lessons (e.g., D5 taught that allergens are safety-critical — so contaminated allergen labels are especially dangerous)
