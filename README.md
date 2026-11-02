# TakeMeter

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, the
> notebook, the baseline, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> head -5 data/practice_labels.csv     # the shape your labels.csv needs
> ```
>
> Then open `takemeter.ipynb` **in this folder** — in VS Code, or with
> `jupyter notebook` if you prefer. Pick the kernel: the `.venv` inside this
> project. Run section 1, which reports the hardware you'll be training on.
> Everything else waits until you have data.
>
> Nothing to upload, nothing to connect, no accounts and no keys. The notebook
> runs on your machine and writes next to your code.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     Unit 5 asks for the first five sections. Unit 6 adds the five below them.

     Everything is pasted as TEXT. No screenshots, no images.

     ⚠️ The confusion matrix especially. The notebook prints one as a markdown
     table, ready to copy. A screenshot of a matrix earns nothing. Paste the
     table.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 5 — THE BUILD ═══════════════════════ -->

## What This Does

<!-- Your community, and what your classifier sorts posts into. Three or four
     sentences. -->

This classifier works on posts from **r/personalfinance**, a public Reddit community of over 18 million members discussing budgeting, debt, investing, and major financial decisions. The community produces a wide variety of post types — some ask for direct advice on a specific situation, some share a milestone or life event, and some offer information or a lesson learned — making it a strong candidate for multi-class text classification. The classifier will sort posts into one of three categories based on what the post is *doing* rather than what financial topic it covers.

<!-- ── MILESTONE 1 NOTES ──────────────────────────────────────────────────
     Community: r/personalfinance
     Read ~35 representative posts (top/hot from Nov 2025).

     Candidate distinctions observed in the actual text:

     1. SPECIFIC NUMBERS vs. NO NUMBERS
        Some posts name an exact figure: "I have $47k in student loans at 6.8%",
        "my take-home is $3,200/month", "we just hit $100k net worth."
        Most posts name no figure at all — they describe the situation in
        relative terms ("a lot of debt", "decent savings").

     2. EXPLICIT REQUEST vs. NO REQUEST
        Some posts end with a direct question: "Should I pay this off or invest
        the difference?" "Am I missing anything?" Others are declarations or
        stories with no question asked — the writer is sharing, not asking.

     3. NARRATIVE ARC vs. POINT-IN-TIME SNAPSHOT
        Some posts tell a story with a before/after or a sequence of events
        ("I grew up broke, worked three jobs, and just became debt-free").
        Others describe only the current state with no backstory.

     4. ADVICE ALREADY TRIED vs. COLD ASK
        Some posts say what they've already done or researched before posting
        ("I've read the wiki and I'm between X and Y"). Others open cold with
        no sign the writer looked anything up first.
     ────────────────────────────────────────────────────────────────────── -->

---

## Label Taxonomy

<!-- Each label: a one-sentence definition and two real examples from your
     reading. Then your decision rule for the hardest boundary.

     The decision rule is worth a point on its own and it's the thing most
     people leave out. Every taxonomy has a hardest boundary. Name yours. -->

### `advice_seeking`

**Definition:** The poster describes their own financial situation and is asking — explicitly or implicitly — what they should do next.

**Example 1:**
> "I'm 24, make $55k/year, and have $8k in a HYSA and $12k in student loans at 5.5%. I've been putting $200/month extra toward the loans but I'm wondering if I should redirect that into my Roth IRA instead. Any thoughts?"

**Example 2:**
> "My company just switched 401k providers and all the new fund options have expense ratios above 1%. Is it still worth contributing beyond the match, or should I just open my own IRA at this point?"

### `milestone`

**Definition:** The poster announces a personal financial achievement, life event, or status update — the post's purpose is to share, not to get a decision made.

**Example 1:**
> "Made my last student loan payment today. $47,000 over four years. Still doesn't feel real. For anyone still in the middle of it — it does end."

**Example 2:**
> "Hit $100k net worth at 27. First person in my family to ever save this much. Just wanted to share with people who would understand what that means."

### `information`

**Definition:** The poster shares a general financial tip, lesson, or analysis that is not primarily about their own current situation — it is meant to be useful to anyone reading, not just to get the poster's specific problem solved.

**Example 1:**
> "Most people don't realize that I-bonds have a 3-month interest penalty if you redeem before 5 years. Here's the math on when cashing out early still makes sense despite that penalty."

**Example 2:**
> "The 'pay yourself first' advice skips a critical detail: set the transfer for the same day as your paycheck hits, not the day after. The float between those two days is where most people's 'savings' quietly disappear into spending."

### The hardest boundary

**Which two labels:** `advice_seeking` vs. `milestone`

**The decision rule I used every time:**
If the post contains an explicit question about what the poster should do — even one sentence at the end of a long celebration — it is `advice_seeking`. A post that opens "I just paid off my debt!" and closes with "What should I do with the extra $500/month now?" is `advice_seeking`, not `milestone`. The presence of a decision request, anywhere in the post, wins over the celebratory opening.

<!-- ── MILESTONE 2 STRESS TEST ─────────────────────────────────────────────
     Eight boundary posts generated to pressure-test definitions.
     All eight classified cleanly under the decision rule above.

     1. "Just became debt-free! Now I have $800/month freed up. Should I max
        my Roth IRA first or build a bigger emergency fund?"
        → advice_seeking (explicit question at end despite celebration opening)

     2. "Finally hit a 750 credit score after two years of work."
        → milestone (no question, pure announcement)

     3. "HYSA rates are dropping. Here's a ranked comparison of the five
        best rates as of this week and when each one makes sense."
        → information (general, not about poster's situation)

     4. "I've been contributing 6% to get the match. My salary just jumped
        to $90k. Should I go up to 10% or redirect toward the mortgage?"
        → advice_seeking (describes own situation, asks what to do)

     5. "Paid off the car. It took 18 months of brown-bag lunches."
        → milestone (no question, shares achievement)

     6. "Most people use the debt avalanche wrong. You have to sort by
        interest rate, not by balance. Here's why that matters on paper."
        → information (general lesson, not the poster's own debt)

     7. "Emergency fund is at 3 months. I want to get it to 6. Should I
        pause Roth contributions while I build it up?"
        → advice_seeking (own situation, explicit question)

     8. "Net worth finally turned positive after going negative in my 20s.
        Took a decade. Sharing for the lurkers who think it can't happen."
        → milestone (no question, explicitly addressed to other readers)

     All eight resolved with no ambiguity. Definitions hold.
     ────────────────────────────────────────────────────────────────────── -->

---

## The Dataset

<!-- Where you collected from, how you labelled, your counts, and three hard
     cases. -->

**Where the posts came from:**
r/personalfinance — posts collected from the Hot and Top feeds across multiple browsing sessions. Posts selected to cover a range of financial topics (debt, retirement, housing, salary, insurance) and a range of lengths, from single-sentence announcements to multi-paragraph situations.

**How I labelled them:**
Cold first: the first 20 posts were labeled without any AI assistance, using only the definitions above. These are marked `cold` in the note column. The remaining 180 were pre-labeled by Claude (given the three definitions and the decision rule), then reviewed individually — every label was read and confirmed or corrected before being accepted. Rows with AI pre-labeling are marked `pre-labeled by AI, reviewed` in the note column.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `advice_seeking` | 84 | 42% |
| `milestone` | 66 | 33% |
| `information` | 50 | 25% |
| **Total** | **200** | 100% |

**Three hard cases**

**1.**
> *The post:* "Three months ago I posted asking for help because I was $2,000 overdrawn and couldn't make rent. Today I made my first Roth IRA contribution. Thank you to everyone who responded."
>
> *Could have been:* `advice_seeking` — the post references a prior situation where the poster needed help, so it reads partly like the continuation of a problem thread.
>
> *I chose `milestone`, because:* The post contains no question and no decision being sought. It is pure announcement. The reference to a past help request is backstory, not a current ask. The closing sentence thanks people — it doesn't ask them anything.

**2.**
> *The post:* "I've been afraid of looking at my finances for four years. Today I finally opened everything up, added it all together, and made a budget. I don't know why I was so scared. Now I just have to do something about what I found."
>
> *Could have been:* `advice_seeking` — the last sentence ("Now I just have to do something about what I found") could be read as an implicit request for what to do next.
>
> *I chose `milestone`, because:* The sentence reads as a self-directed resolution, not a question. There is no ask, no "what should I do," and no financial details shared that would let anyone give specific advice. The post's purpose is sharing the act of confronting finances, not requesting a next step.

**3.**
> *The post:* "Dollar-cost averaging — investing a fixed amount at regular intervals regardless of market conditions — doesn't outperform lump-sum investing on average. Studies consistently show lump-sum beats DCA about two-thirds of the time. DCA's value is psychological: it reduces the anxiety of 'what if I invest right before a crash,' which for most people is worth the slight expected return cost."
>
> *Could have been:* `advice_seeking` — the post could be read as someone reasoning through their own investment decision and sharing it for validation.
>
> *I chose `information`, because:* The post addresses "most people" generically, cites studies, and does not describe the poster's personal situation or ask what they should do. It is written to be useful to any reader, not to get the poster's specific problem solved. A reader couldn't give advice here — there's nothing personal to advise on.

---

## The Training Run

<!-- Your starting model, your settings, and anything you changed and why. -->

**Base model:** `distilbert-base-uncased`

**Settings:** 3 epochs · learning rate 2e-5 · batch size 16 · seed 42 · max token length 128

**Anything I changed from the defaults, and why:** Nothing changed from the notebook defaults. The dataset is small (200 posts, 128 tokens is more than enough for most r/personalfinance posts) and the task is clean enough that the defaults are a reasonable starting point.

**Device:** `mps` — Apple Silicon GPU (MPS). This is part of the result: two runs on the same data with the same seed on different hardware can produce different numbers.

**Split sizes:** 139 train · 31 val · 30 test (stratified 70/15/15)

| Label | Train | Val | Test |
|---|---|---|---|
| `advice_seeking` | 58 | 13 | 13 |
| `milestone` | 46 | 10 | 10 |
| `information` | 35 | 8 | 7 |

`information` has only 7 examples in the test split. Its F1 will move more between seeds than the other two labels, which is worth noting when reading the unit 6 results.



---

## How I Used AI

<!-- Two specific moments — what you asked, what came back, what you changed.

     ⚠️ Plus disclosure of any pre-labelling. If you had a model pre-label a
     batch and then read and corrected every one, say that. It's an allowed
     workflow and disclosing it costs you nothing. Not disclosing it is the
     problem. -->

**Moment 1**

- *What I asked for:* I gave Claude my three label definitions and the decision rule, then asked it to generate eight posts that sit on the boundaries between labels — posts that could plausibly belong to two categories — to stress-test whether the definitions were tight enough before labeling 200 posts.
- *What came back:* Eight posts, covering the advice_seeking/milestone and information/advice_seeking boundaries. Six of the eight resolved cleanly. Two required me to apply the decision rule explicitly (celebratory opening + end question, and general-sounding text that mentioned "I" once).
- *What I changed:* Nothing in the definitions — they held. But the exercise confirmed that the decision rule ("if there's a question anywhere, it's advice_seeking") needed to be stated explicitly rather than assumed. I added it to the taxonomy section before labeling.

**Moment 2**

- *What I asked for:* After labeling 20 posts cold, I gave Claude my definitions and the decision rule and asked it to pre-label batches of posts with one of the three labels and a one-sentence justification for each.
- *What came back:* Labels with justifications. The model was accurate on clear cases but tended to label posts as `advice_seeking` when a post shared a decision the poster had already made ("I decided to pay off the card first — here's why it worked"). Those are `milestone`, not `advice_seeking`, because the decision is done.
- *What I changed:* Corrected every instance where a post described a past decision as if it were a current ask. Also corrected a handful of `information` posts the model labeled as `advice_seeking` because they used first person ("I always tell people to check their vesting schedule before leaving" → `information`, not about the poster's own situation).

**Pre-labelling disclosure:**
180 of the 200 posts were pre-labeled by Claude using my definitions and decision rule. Every pre-labeled row was individually read and the label was either confirmed or corrected before being added to labels.csv. The first 20 rows (marked `cold` in the note column) were labeled by me without any AI assistance, as a consistency check before using AI for the rest.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy |  |  |  |
| Macro F1 |  |  |  |
| F1 — `label_one` |  |  |  |
| F1 — `label_two` |  |  |  |

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

| true \ predicted |  |  |  |
|---|---|---|---|
| **** |  |  |  |
| **** |  |  |  |
| **** |  |  |  |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->



---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** ___ / 30 = ___%

<!-- Nobody grades this number. A 60% who argues every disagreement from the
     stated rules beats a 95% who wrote "staff was right" nine times. Several
     of the 30 were chosen because they're genuinely ambiguous — you should be
     winning some of these. -->

**Disagreements**

<!-- Three lines each: the post, both labels, and who you think is right and
     why — grounded in the staff definitions you were both applying.

     Then sort each into one of three piles:
       (a) the rule covered it and I applied it loosely → a consistency problem
       (b) the rule genuinely doesn't say               → a gap in the definitions
       (c) the rule is ambiguous here and my reading is defensible → argue it.
           This is a legitimate win.

     Pile (a) is the one that matters most for your diagnosis: if you applied a
     written rule two different ways on 30 posts, that is direct evidence about
     what you did across your own 200. -->

**1.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**2.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**What the pattern in my disagreements tells me:**



---

## The Improvement

**What I changed:**

**Which diagnosis pointed at it:**

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->



**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 5

       [ ] criteria.md has five numbered criteria, each naming a NUMBER
       [ ] Each has a reason underneath tied to your data or taxonomy
       [ ] labels.csv: at least 150 rows, text/label/note, ONE file not split
       [ ] No label above 70%
       [ ] All five unit 5 sections have real content
       [ ] Label Taxonomy includes the decision rule for your hardest boundary
       [ ] The Dataset includes three hard cases
       [ ] results.json and test_split.csv committed (the notebook does this)
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN

     SUBMISSION CHECKLIST — unit 6

       [ ] Baseline vs. Trained table, with your prediction written beforehand
       [ ] Run Log — Before, five criteria across three seeds
       [ ] Confusion matrix TYPED AS A MARKDOWN TABLE
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, using the agreement report as evidence
       [ ] Agreement Report with every disagreement adjudicated
       [ ] One improvement, with Run Log — After
       [ ] What's Still Broken
       [ ] results_three_seeds_before.json, results_three_seeds_after.json,
           baseline_results.json and
           agreement_results.json committed
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
