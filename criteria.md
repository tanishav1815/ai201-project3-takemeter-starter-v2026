# Acceptance criteria — TakeMeter

Five criteria that say what "working" means for this classifier, written in
unit 5 **before** anything was trained.

**All five are yours this time.** None are given. You've had two projects of
practice.

An acceptance criterion names a number. *"The model is accurate"* is an
opinion. *"Every label has an F1 of at least 0.60 on the held-out set"* is a
criterion.

Under each, write a sentence or two on **why that number**. A reason that says
something about your data or your taxonomy earns credit — *"I picked 0.60 F1
for `reaction` because it's my smallest label and I only have about 50
examples of it"*. A reason that could be attached to any project does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## Pick numbers you can defend

Not numbers that sound impressive. Three labels means a coin-flip guesser gets
about 33%, so a target of 0.40 is barely a target. Your number should sit
somewhere you'd honestly call useful.

**Cover at least three of these five areas.** They're here as prompts, not as a
form to fill in — a criterion that fits none of them is fine if it names a
number.

| Area | A question it could answer |
|---|---|
| Overall accuracy | How often does it need to be right to be worth using? |
| Per-label performance | Is one label allowed to be much worse than the others? |
| Balance | How lopsided can your label counts get before it's a problem? |
| Consistency | If someone else labelled the same posts, how often should you agree? |
| Confidence | Should a confident prediction be right more often than an unsure one? |

Two things worth knowing before you pick numbers, because both will affect
whether you hit them:

- **Your smallest label will have the jumpiest score.** If a label has 50
  examples, about 8 land in the test split. An F1 computed on 8 examples moves
  a lot between seeds. A target for that label should be looser than one for
  your biggest label, and saying so is a good reason.
- **Unit 6 tests across three seeds, and the target has to hold across all
  three.** A target of 0.65 against results of 0.71, 0.62, 0.68 is a **miss**.
  Pick with that in mind — it is stricter than it first sounds.

---

## 1. Overall accuracy of at least 0.68 across all three seeds.

The model correctly labels at least 68% of held-out posts in every one of the three seed runs.

**Why this target:**
A random guesser on three balanced labels gets 33%. A target of 0.68 means the model needs to be right on roughly two in three posts — high enough to be genuinely useful as a sorting tool, low enough to be reachable with ~140 training examples. I set the floor at 0.68 rather than 0.65 because the three labels in this taxonomy are conceptually distinct: a post that asks "what should I do?" looks very different in text from one that announces "I paid it off." The task is easier than a sentiment or tone taxonomy, so I'm holding the model to a higher standard than I would for a fuzzier problem.

---

## 2. F1 of at least 0.55 for every individual label across all three seeds.

No label has an F1 below 0.55 in any of the three seed runs.

**Why this target:**
With ~200 posts split 70/15/15, each label gets roughly 10 examples in the test set. An F1 computed on 10 examples moves a lot — one misclassified post can swing it 8–10 points. I set the floor at 0.55 rather than 0.60 specifically because of this variance: a target of 0.60 could easily be missed in one seed purely from test-set luck rather than from a real model failure. 0.55 is still meaningfully above chance (a one-label guesser gets an F1 of 0 for every label it never predicts), and it has to hold across all three seeds, which makes it stricter than it first sounds.

---

## 3. No single label exceeds 55% of the total labeled dataset.

The most common label in `labels.csv` accounts for no more than 55% of all rows.

**Why this target:**
r/personalfinance skews heavily toward advice-seeking posts — in a random sample, `advice_seeking` could easily be 65–70% of posts. At that concentration the model learns to guess the majority class and hits reasonable accuracy without learning the boundaries between labels. 55% is the point where the minority labels still have enough examples (~90 each for two labels at 200 total) to train and test meaningfully. This is a collection constraint, not a model metric: if I hit 58%, I need to add more `milestone` or `information` posts before training.

---

## 4. Agreement with the staff-labeled set of at least 65% (at least 20 out of 30 posts).

When I label the 30 staff posts under the staff taxonomy, my labels match theirs on at least 20 of the 30.

**Why this target:**
This criterion is about my own labeling consistency, not the model — if I can't apply a written rule the same way twice, the model gets an inconsistent signal and the F1 numbers mean less. 65% (20/30) is the floor I'd expect for a taxonomy with clear written decision rules applied by someone who read them carefully. I set it at 65% rather than higher because the staff taxonomy is different from mine: I'll be applying someone else's definitions, so some disagreement is expected even if my own labels are internally consistent. Hitting 65% tells me my labeling process is disciplined enough; missing it tells me I need to tighten how I apply rules before collecting the full 200.

---

## 5. Macro F1 of at least 0.62 across all three seeds.

The average of the three per-label F1 scores is at least 0.62 in every seed run.

**Why this target:**
Macro F1 averages over labels equally, so a model that ignores a minority label can't hide behind a high accuracy score. 0.62 sits between my per-label floor (0.55) and what I'd expect a well-trained model to reach if all three labels are reasonably balanced and the taxonomy is tight. I chose 0.62 rather than 0.65 because macro averages are pulled down hard when one label underperforms — with only ~10 test examples per label, one bad seed for one label can drop the macro by 3–4 points without the model actually being broken. The target has to hold across all three seeds, which makes it genuinely ambitious at this dataset size.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 6 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath, like this:

         ## 2. Every label performs acceptably

         The model performs well on all labels.

         **Why this target:** ...

         > **Revised in unit 6:** Every label has an F1 of at least 0.60 on
         > the held-out set.
         >
         > **Why revised:** "performs well" gave me nothing to check. I
         > couldn't produce a verdict from it at all.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "Overall accuracy of at least 0.65" → "at least 0.55", because
            0.65 turned out to be optimistic for 200 examples.

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
