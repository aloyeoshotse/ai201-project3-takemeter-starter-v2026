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

## 1.

**Overall accuracy is at least 0.70 on the held-out test set, on every one of the three seeds (42, 7, 2024).**

**Why this target:**

I have four labels (`news`, `question`, `opinion`, `promo`), so a guesser gets about 25%, and always predicting the biggest label gets maybe 30–35%. 0.70 means roughly 7 of 10 posts sorted correctly, which I'd call useful for a first-pass sorter and far above chance. I didn't go higher because my `opinion`/`news` boundary is the one I found hardest to write down (the "controversial" headline case), and with only ~30 test posts one seed can swing by several points. It has to hold on all three seeds, so I set it where a good-but-not-lucky run still clears it.

---

## 2.

**Every label has an F1 of at least 0.55 on the held-out set, on every seed.**

**Why this target:**

Overall accuracy can hide one label being broken. With a ~200-post dataset and a 70/15/15 split, each label gets only about 7–8 test posts, so a single mistake moves that label's F1 by 10+ points. That is why this is looser than the accuracy target. 0.55 is still well above the ~0.25 chance level. If any label falls under it, the model is effectively not recognising that category, and I'd want to know which one.

---

## 3.

**No label makes up less than 10% of `labels.csv` (at least 20 of 200 posts), and the largest label is no more than 3× the smallest.**

**Why this target:**

This is the cause I can check before training. A label with under 20 posts leaves about 3 in the test split, and its F1 will be noise. I expect `promo` to be my smallest label because developer self-announcements are rarer in the feed than opinions or news, so this criterion is aimed at it. 3× is lopsided enough that the model could do well by leaning toward the big labels, but not so strict that I'd have to throw away real posts to hit it.

---

## 4.

**On the 30 staff posts, labelled under the staff taxonomy (not mine), I agree with staff on at least 24 of 30 (0.80).**

**Why this target:**

If someone applying a written rule can't match me 80% of the time, then the labels are partly my taste and the model has nothing stable to learn. I can't expect more than the model will reach, so 0.80 sits above my accuracy target of 0.70: the model should not be held to a standard of consistency higher than I can meet myself. It's not 0.90 because `opinion` and `news` have a genuinely fuzzy edge, and a handful of those 30 posts will sit on it.

---

## 5.

**When the model's top confidence is at least 0.80, it is correct at least 85% of the time, and its average confidence on correct predictions is higher than on wrong ones.**

**Why this target:**

A classifier that says it is sure and is wrong is worse than one that hedges, because I'd trust the confident ones and skip checking them. 85% is deliberately above the 0.70 overall target: if confidence means anything, the confident subset should beat the average. I know the number of posts at ≥0.80 will be small on a ~30-post test set (maybe 10–20), so one error moves it a lot. That is the reason I paired it with the average-confidence comparison, which uses every prediction.

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
