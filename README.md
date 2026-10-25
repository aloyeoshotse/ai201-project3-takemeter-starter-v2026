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

TakeMeter reads a public gaming post from Reddit (r/pcgaming, r/Games, r/gamedev, r/patientgamers, r/IndieGaming and r/gamingsuggestions) and sorts it into one of four labels: `news` (reports something that happened, with no opinion), `question` (asks the community for something), `opinion` (the poster's own judgment, praise or complaint) and `promo` (a developer or publisher promoting their own game or company). A gaming feed mixes all four kinds of post, and the useful thing to know about a post is usually which kind it is, such as whether it is someone selling a game or someone reviewing one. The model is DistilBERT fine-tuned on 230 posts that I labelled using the definitions below, and it trains and runs on a laptop CPU.


---

## Label Taxonomy

<!-- Each label: a one-sentence definition and two real examples from your
     reading. Then your decision rule for the hardest boundary.

     The decision rule is worth a point on its own and it's the thing most
     people leave out. Every taxonomy has a hardest boundary. Name yours. -->

### `news`

**Definition:**
news - reports something that happened or was announced, and states no opinion about it.

**Example 1:**
>After Star Wars: Zero Company and Galactic Racer, Lucasfilm says it will "continue to surprise folks" and is "more flexible" with the franchise

**Example 2:**
>Steam Wallet funds in Japan will expire after 6 months if left unused. Valve updated its Japanese Steam terms, introducing a rule that makes unused wallet funds expire six months after they’re added. There’s no confirmation that it will affect Steam users in other countries

### `question`

**Definition:**
question - asks the community for something, like a recommendation, a fix, or an opinion.

**Example 1:**
> what is your favourite unpopular game on pc ?

**Example 2:**
> Help me find the game


### `opinion`

**Definition:**
opinion - states the poster’s own judgment, praise, or complaint, whether or not it’s tied to news.

**Example 1:**
> Razer Wolverine V4 Pro review

**Example 2:**
> Dishonored ruined other games for me lol.

### `promo`

**Definition:**
promo - a post from a developer or publisher promoting or announcing something about their own game or company, such as a release, milestone, sale, or thank-you message.

**Example 1:**
> GOOD HEAVENS! – Whimsical Fantasy Crafting RPG
> Hey everyone! We’ve just released Good Heavens! Our journey started 5 years ago and I wanted to share this experience with you.
>
> What is the game about?
>
> So in the game, there is a clumsy god; Rar. Basically he creates a world and everything goes downhill. He picks a chosen one to fix everything, then another, then another... So yeah, we are the 25,004th chosen one. While we spawn into this world, what we need to do is survive, improve our skills, and complete the tasks.
>
> There is an entire procedurally generated world at your fingertips, with so many different cities and events to discover. It has a humorous and colorful tone, and you can play it either with friends or on your own. It is co-op up to eight players. You can all upgrade your techs if you want, or each of you can assign roles and play accordingly. There are 3 combat and 3 utility tech trees. It's entirely up to you how you want to customize. Also, no generative AI was used, everything was made with blood and tears of our artists.
>
> If you like to check our game, here is the link to our Steam page.
>
> Hope you’ll like it!

**Example 2:**
> The Witcher: 11 years after launch, more witchers are now walking the Path across the Continent than ever before. Seeing so many of you set out on this journey for the very first time, or relive it once more, means the world to us. Thank you for putting your trust in us, all these years later.

### The hardest boundary

**Which two labels:** `news` and `opinion`

**The post that sits between them:**
> ‘This is horrifying’ Nvidia’s controversial DLSS 5 AI filter leaks and players are inserting it into every game

It reports events (a leak, players adding the filter to games), which fits `news`. It also carries evaluative wording ("controversial", and the quoted "This is horrifying"), which fits `opinion`.

**The decision rules I used every time:**
1. `news` vs `opinion`: if the evaluative wording is in the writer's own voice ("is a disaster", "is awesome"), label it `opinion`, even when it also reports a fact. If the judgment is attributed to someone else ("according to players", a quoted reaction), the writer is only reporting it, so label it `news`. If it is still unclear, label it `opinion`.
2. `promo` vs `news`: if the poster is the developer or publisher talking about its own game or company in the first person ("we", "our", "we've"), label it `promo`, even for patch notes or price changes. A third party reporting the same event in the third person is `news`.

Applied to the examples: "Dishonored ruined other games for me lol." is `opinion` because the judgment is in the poster's own voice. The DLSS 5 headline is `opinion` because "controversial" is the writer's own wording, not attributed to anyone, and the quoted "This is horrifying" does not change that. Without "controversial" it would be `news`.

I chose rules based on voice and attribution because I can check them on the text itself. I tested them with eight AI-written boundary posts: I disagreed with my first draft rule on one of them (an attributed quote, "according to players"), so I changed rule 1, and I added rule 2 after I labeled a first-person patch-notes post `news`.

---

## The Dataset

<!-- Where you collected from, how you labelled, your counts, and three hard
     cases. -->

**Where the posts came from:** 230 public Reddit posts from r/pcgaming, r/Games, r/gamedev, r/patientgamers, r/IndieGaming and r/gamingsuggestions. Text posts keep the full title and body; link posts keep only the Reddit title. I excluded image/video/gallery posts, crossposts, recurring moderator threads, duplicates, deleted posts and posts whose body did not load. This is a convenience sample of new posts, not a random sample of Reddit, and some r/Games posts came from its developer showcase that day (which is where most `promo` comes from). r/gamingsuggestions was added to get enough `question` posts. Details are in `data/collection_notes.md`.

**How I labelled them:** I labelled 27 posts cold (marked `cold` in `note`): 20 first, then 7 more with full text after I found that title-only labelling was a mistake. An AI assistant pre-labelled the other 203 using my four definitions and two decision rules (marked `pre-labelled`). Three of the cold rows took the AI's label for posts I was unsure about, and I corrected four of my own cold labels after the AI flagged that they contradicted my `opinion` definition (see How I Used AI). I have not yet finished reading and correcting every pre-labelled row, so some of those labels may still be wrong.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `news` | 84 | 36.5% |
| `opinion` | 60 | 26.1% |
| `question` | 47 | 20.4% |
| `promo` | 39 | 17.0% |
| **Total** | **230** | 100% |

`promo` is the smallest label, as predicted in `criteria.md`. The largest-to-smallest ratio is 84/39 = 2.2x, inside my 3x limit.

**Three hard cases**

**1.**
> *The post:* "Shogun 2 remaster hits "overwhelmingly negative" reviews on Steam over AI and performance"
>
> *Could have been:* `news` or `opinion`
>
> *I chose `news`, because:* "overwhelmingly negative" is Steam's rating label, quoted and attributed to someone else, so the writer is reporting a reaction, not giving one. Rule 1 says attributed judgment is `news`.

**2.**
> *The post:* "Do you see a future for classic game dev two years from now?" followed by a long first-person account of 15 years of programming, how AI changed it, and a prediction.
>
> *Could have been:* `question` or `opinion`
>
> *I chose `question`, because:* the title asks the community directly and the body is context for that ask. The prediction is the poster's own view, but the post exists to start a discussion, not to state a verdict.

**3.**
> *The post:* "Wine (for Linux) might mark dgvoodoo2 as WONTFIX due to malicious and intentional sabotage by developer..." with a bug-tracker link and a long technical write-up.
>
> *Could have been:* `news` or `opinion`
>
> *I chose `news`, because:* the "sabotage" wording is the bug report's allegation, and the post mainly reports what Wine might do and what a user found. It is close to `opinion` because of the strong wording, and I used rule 1's attribution test to decide.

---

## The Training Run

<!-- Your starting model, your settings, and anything you changed and why. -->

**Base model:** `distilbert-base-uncased` (DistilBERT), fine-tuned on CPU for four-way classification.

**Settings:** seed 42, 3 epochs, learning rate 2e-5, batch size 16, max sequence length 128 tokens. Stratified 70/15/15 split.

**Anything I changed from the defaults, and why:** Nothing. I kept the notebook defaults for the first run so I would have a baseline to diagnose in unit 6.

**Split sizes:** train 160 / val 35 / test 35. Test per label: `news` 13, `opinion` 9, `question` 7, `promo` 6. `question` and `promo` have fewer than about 8 test posts, so their F1 will move a lot between seeds. This first run (seed 42, before I finished reviewing the pre-labelled rows) got 0.571 accuracy and 0.343 macro F1, with `question` and `promo` both at F1 0.00.

---

## How I Used AI

<!-- Two specific moments — what you asked, what came back, what you changed.

     ⚠️ Plus disclosure of any pre-labelling. If you had a model pre-label a
     batch and then read and corrected every one, say that. It's an allowed
     workflow and disclosing it costs you nothing. Not disclosing it is the
     problem. -->

**Moment 1**

- *What I asked for:* I asked an AI assistant (Claude, in Claude Code) to show me the first 20 posts in `labels.csv` so I could label them cold.
- *What came back:* It pointed out that the file is ordered by subreddit, so the first 20 rows would all be r/pcgaming headlines and my cold set would miss most of my labels. Instead it picked 20 rows at random with a fixed seed (201), so the sample could be repeated, and showed them to me with the long posts shortened. Later it drew 7 more at random (seed 202) from the rows I had not seen, and showed me the full text of those.
- *What I changed:* I labelled the first 20 from the titles only, since the long posts were shortened, which was a mistake because some bodies decided the label. That is why I asked for the second set of 7 with full text. I labelled those 7 with no AI input, and together with the first 20 they give 27 rows marked `cold` in `labels.csv`.

**Moment 2**

- *What I asked for:* I sent the AI my labels for the first 20 posts, three of them marked "unsure", and asked it to record them in `labels.csv`. When it asked me to pick labels for the unsure ones, I asked what it would pick.
- *What came back:* Without my asking, it pointed out that I had labelled #17, a personal review, as `opinion`, while I had labelled four similar long personal write-ups (#14, #15, #16 and #18) as `news`, which did not match my own definition of `opinion` ("the poster's own judgment, praise, or complaint"). It also gave labels for the three I was unsure about (#3 `news`, #6 `promo`, #19 `opinion`).
- *What I changed:* I had labelled those posts from their titles only, so I changed #14, #15, #16 and #18 to `opinion` after reading the feedback. I used the AI's labels for #3, #6 and #19. Because of that I do not count those 7 rows as a clean cold check.

**Pre-labelling disclosure:** 27 rows are marked `cold` in the `note` column. I labelled 24 of them myself, and the AI picked the labels for three (see Moment 2). The AI pre-labelled the other 203 rows, using my four definitions and my two decision rules from the Label Taxonomy section, and I marked those rows `pre-labelled`. It also gave a confidence and a second-choice label for each row, which I used to decide which rows to look at first. A separate AI browser tool (in my signed-in Chrome) collected the 230 posts; whole posts were copied from Reddit (see `data/collection_notes.md`). I have not yet finished reading and correcting every one of the 203 pre-labelled rows, so those labels may still contain mistakes and the notes will change to `pre-labelled, checked` as I review them. My first training run happened before that review, so `results.json` and `test_split.csv` will need to be re-run if I change any labels.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.429 | 0.571 | +0.143 |
| Macro F1 | 0.420 | 0.343 | -0.077 |
| F1 — `news` | 0.000 | 0.812 | +0.812 |
| F1 — `opinion` | 0.625 | 0.560 | -0.065 |
| F1 — `promo` | 0.348 | 0.000 | -0.348 |
| F1 — `question` | 0.706 | 0.000 | -0.706 |

**What I predicted before I looked:**
I predict my fine-tuned model will beat the zero-shot baseline most on opinion vs news, because my labels follow quirks of my own rules 
(a headline like “Razer Wolverine V4 Pro review” counts as opinion, and unclear cases default to opinion), and a zero-shot model that 
sees only my definitions can’t learn those quirks.

**What the gap actually means:**
I compared against the zero-shot baseline using my definitions, and also ran it once with bare label names: the definitions 
made it worse (0.429 vs 0.714 accuracy), because with them it predicted promo for 17 of 35 posts. My fine-tuned model beat the definitions baseline 
on accuracy (+0.143) but lost on macro F1 (−0.077), and against the bare-names baseline it loses on accuracy too (0.571 vs 0.714). Almost all of its 
gain is news (0.000 → 0.812), and that gain exists mainly because the baseline over-predicted promo. My prediction was only half right: news improved most, 
but opinion got slightly worse (−0.065), so I can’t claim the model learned my rule for that boundary. The model scored F1 0.00 on promo and question, which 
means it never predicted them, and the confusion matrix below confirms it: the `question` and `promo` columns are all zeros. These numbers are from seed 42 only, and with 6–7 test posts for each of those labels they are noisy.



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

Spread across three seeds (section 6 of the notebook). All three seeds ran on CPU (torch 2.14.1+cu130, max length 128):

| Measure | Seed 42 | Seed 7 | Seed 2024 | Spread |
|---|---|---|---|---|
| Overall accuracy | 0.571 | 0.657 | 0.543 | 0.114 |
| Macro F1 | 0.343 | 0.510 | 0.332 | 0.178 |
| F1 — `news` | 0.812 | 0.788 | 0.828 | 0.040 |
| F1 — `question` | 0.000 | 0.824 | 0.000 | 0.824 |
| F1 — `opinion` | 0.560 | 0.429 | 0.500 | 0.131 |
| F1 — `promo` | 0.000 | 0.000 | 0.000 | 0.000 |

Accuracy swings 0.114 across seeds, which is over the ten-point line, so this dataset is too small for a stable measure. The `question` score is the clearest case: 0.000, 0.824, 0.000. Seed 7 happened to predict it and the other two seeds never did.

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.70 on every seed | 0.571 | 0.657 | 0.543 | MISSED |
| 2. Every label F1 | ≥ 0.55 on every seed | lowest 0.000 (`question`, `promo`) | lowest 0.000 (`promo`) | lowest 0.000 (`question`, `promo`) | MISSED |
| 3. Balance | smallest label ≥ 10% and largest ≤ 3× smallest | 17.0% (`promo`), 2.2× | same | same | MET |
| 4. Agreement with staff | ≥ 24 of 30 (0.80) | 29 / 30 (0.97) | same | same | MET |
| 5. Confidence | ≥ 85% correct when confidence ≥ 0.80, and mean confidence higher on correct than on wrong | 0 predictions ≥ 0.80 (max 0.487); mean 0.370 right vs 0.307 wrong | 0 predictions ≥ 0.80 (max 0.427); 0.338 vs 0.304 | 0 predictions ≥ 0.80 (max 0.345); 0.323 vs 0.287 | MISSED |

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

| true \ predicted | news | question | opinion | promo |
|---|---|---|---|---|
| **news** | 13 | 0 | 0 | 0 |
| **question** | 2 | 0 | 5 | 0 |
| **opinion** | 2 | 0 | 7 | 0 |
| **promo** | 2 | 0 | 4 | 0 |

**My biggest off-diagonal number, and what it means:**
5 of the 7 `question` posts were called `opinion` and none went the other way, so on seed 42 the model never learned `question` (or `promo`) and sent nearly everything to `opinion` or `news`.



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Overall accuracy | ≥ 0.70 on every seed | MISSED | Seeds scored 0.571, 0.657 and 0.543. All three are below 0.70, and the best is 0.043 short. |
| 2 | Every label F1 | ≥ 0.55 on every seed | MISSED | `promo` scored F1 0.000 on all three seeds. `question` scored 0.000 on seeds 42 and 2024 and 0.824 on seed 7. |
| 3 | Balance | smallest label ≥ 10% and largest ≤ 3× smallest | MET | `promo` is 39 of 230 posts (17.0%) and the largest label (`news`, 84) is 2.2× the smallest. |
| 4 | Agreement with staff | ≥ 24 of 30 (0.80) | MET | I agreed with staff on 29 of 30 posts (97%). The one miss was post 8. This criterion doesn't depend on the training seed. |
| 5 | Confidence | ≥ 85% correct when confidence ≥ 0.80, and mean confidence higher on correct than on wrong | MISSED | The second half held on all three seeds (right vs wrong: 0.370/0.307, 0.338/0.304, 0.323/0.287). The first half failed: I re-ran the three seeds and counted, and no prediction reached 0.80 on any seed (highest 0.487, 0.427, 0.345), so there was no confident subset to be 85% right. |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->

**Criterion 2 (every label F1 ≥ 0.55): too few examples for `promo` and `question`.** `promo` has 39 posts and `question` has 47, so the 160-post training split holds about 27 and 33 of them, and the test split holds 6 and 7. The confusion matrix shows the model never predicted either label on seed 42. `promo` was 0.000 on all three seeds, and `question` was 0.000, 0.824 and 0.000. A label that flips between 0 and 0.82 across seeds is near the edge of being learnable. The training run is also short: 160 posts at batch size 16 is 10 steps per epoch, so 3 epochs is only 30 updates, and I suspect the model never moved off predicting the two biggest labels. The zero-shot baseline did score 0.348 on `promo` and 0.706 on `question`, so the labels are learnable in principle, and it is my training data that is too small.

The agreement report can't confirm or rule out a labelling problem on these two labels, because it used the staff taxonomy (`analysis`, `hot_take`, `reaction`) and not my four labels. It does show I can apply a written rule consistently: I matched staff on 29 of 30 posts, and my one miss was loose application of the Rule 1 fact-versus-flourish test, not a confusion between two labels. That makes "I applied the boundaries inconsistently" less likely, but it doesn't test my `question` and `promo` boundaries. The confusion matrix points to the model, not my boundaries: the misses go to `opinion` and `news` in one direction only, 5 of 7 `question` and 4 of 6 `promo` posts to `opinion` on seed 42, and all 7 `question` posts to `opinion` on seed 2024. If my `question`/`opinion` boundary were inconsistent I'd expect `opinion` posts to be called `question` too, and none were on seeds 42 and 2024. One thing I can't rule out: 203 of my 230 labels were AI pre-labelled and I only reviewed them, and only 27 were labelled cold. If the pre-labelling applied the `promo` boundary differently from how I'd have applied it, that would be a labelling problem the agreement report didn't test.

**Cause: too few examples for `promo` and `question`, and a short training run.** Evidence: F1 of 0.000 on `promo` on all three seeds, `question` flipping between 0.000 and 0.824, one-directional misses in the matrix, and a baseline that scores both labels above zero. This is a model and data-size problem, and I have not ruled out the pre-labelling caveat above.

**Criterion 5 (confidence ≥ 0.80 and 85% right): the model is never that sure, a symptom of too little training.** The highest confidence on any seed was 0.487, 0.427 and 0.345. With four labels, 0.25 is the floor for the top class, so seed 2024's best prediction was barely above a uniform guess. The training run is 30 updates (160 training posts at batch size 16 is 10 steps an epoch, for 3 epochs), and I suspect the model hasn't moved far from where it started, though I haven't tested that. The confidence is directionally meaningful: it is higher on correct predictions than wrong ones on all three seeds (0.370 vs 0.307, 0.338 vs 0.304, 0.323 vs 0.287), and the most-confident third of predictions was 1.000, 0.818 and 0.909 correct. So the ordering is useful, but the values never get high enough to hit an absolute 0.80 threshold. I'd call this a symptom of the same cause as criterion 2, not a separate problem, and I haven't checked it against the agreement report because that measures my labelling, not the model's confidence. The threshold itself may also have been set too high for a model this lightly trained, but I'm not revising it: it stays as written, and I'll see whether it moves after the fix.

**Criterion 1 (accuracy ≥ 0.70): follows from the criterion 2 miss.** On seed 42, a model that predicts only `news` and `opinion` can get at most (13 + 9) / 35 = 0.629 right, because 13 of the 35 test posts are `question` or `promo`. Seed 7 got 0.657 when it recovered `question`. So the accuracy target can't be met until the model predicts the two small labels, and fixing criterion 2 is the way to fix this one.



---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** 29 / 30 = 97%

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
> *The post:* "Went back through the club results going back to 2012. Every single year the number of people who enter the spring marathon and don't start it sits between 18 and 23 percent. It's not a bad-luck year, it's the base rate."
>
> *Staff said / I said:* Staff said `analysis`; I said `hot_take`.
>
> *My call, and why:* Staff is right. Rule 1 asks whether the checkable fact carries the argument. Here it does: the 2012 span and the 18-23% range are the evidence, and the closing line ("it's the base rate") only follows from them. The numbers are not decoration on a conclusion the writer had already reached, which is what makes a `hot_take` with a statistic in it. I read "every single year" as emphasis and didn't check whether the data was doing the work.
>
> *Which pile:* (a). The rule covered it and I applied it loosely.

**What the pattern in my disagreements tells me:** I only disagreed on one post, but it matches a pattern in how I labelled. When I reviewed the five posts I was unsure about (1, 2, 4, 7 and 10), I moved three of them (4, 7 and 10) from `hot_take` or `reaction` to `analysis`, so my first pass leaned that way when a post argued from its own data. Post 8 is the same mistake: I let confident wording ("every single year", "it's not a bad-luck year") outweigh the fact that the numbers were the argument. If I did this on my own 200 posts, some of my `opinion` labels may be posts that actually argue from a specific fact.



---

## The Improvement

**What I changed:** One hyperparameter: `EPOCHS` from 3 to 10. Nothing else changed: same `labels.csv`, same learning rate (2e-5), batch size (16), max length (128), seeds (42, 7, 2024) and splits. That takes training from 30 updates to 100. I ran it with a standalone copy of the notebook's training code and wrote `results_three_seeds_after.json`, so `results_three_seeds_before.json` is untouched.

**Which diagnosis pointed at it:** The criterion 2 and criterion 5 diagnoses. Both blamed a short training run (30 updates) together with too few `promo` and `question` examples: `promo` F1 was 0.000 on all three seeds and no prediction ever reached 0.80 confidence. More epochs is the cheapest way to test whether the model was under-trained, and it doesn't touch my labels.

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.70 on every seed | 0.800 | 0.886 | 0.914 | MET |
| 2. Every label F1 | ≥ 0.55 on every seed | lowest 0.625 (`question`) | lowest 0.714 (`opinion`) | lowest 0.875 (`opinion`) | MET |
| 3. Balance | smallest label ≥ 10% and largest ≤ 3× smallest | 17.0% (`promo`), 2.2× | same | same | MET |
| 4. Agreement with staff | ≥ 24 of 30 (0.80) | 29 / 30 (0.97) | same | same | MET |
| 5. Confidence | ≥ 85% correct when confidence ≥ 0.80, and mean confidence higher on correct than on wrong | 6 / 6 correct at ≥ 0.80; mean 0.659 right vs 0.472 wrong | 7 / 7 correct; 0.705 vs 0.671 | 11 / 12 correct (92%); 0.696 right vs 0.704 wrong | MISSED |

**Did it help, and how do I know:** Yes, a lot. Accuracy went from 0.571 / 0.657 / 0.543 to 0.800 / 0.886 / 0.914, and the lowest label F1 went from 0.000 to 0.625 or better on every seed, so criteria 1 and 2 are now MET. `promo` F1 went from 0.000 on all three seeds to 0.800, 1.000 and 0.909, and `question` went from 0.000, 0.824, 0.000 to 0.625, 0.875, 0.933. That supports the diagnosis that the model was under-trained, and that the `promo` and `question` labels were learnable all along. It also makes a labelling problem on those two labels less likely, though it doesn't rule one out for `opinion`.

Criterion 5 is still MISSED, but only on seed 2024, and by very little. The 0.80 half now holds on every seed (100%, 100% and 92% correct). The second half failed on seed 2024, where mean confidence on wrong predictions (0.704) was higher than on correct ones (0.696). That gap is 0.008 on a 35-post test split with 3 wrong predictions, so it's well inside the noise, but the target says every seed and I'm not revising it.

Two limits on this result. First, the test split is 35 posts, so one post is about 3 points of accuracy, and seed-to-seed spread is still wide (0.800 to 0.914). Second, I only tried one setting. I didn't tune the number of epochs, so I don't know whether 10 is the best choice.

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->

**Criterion 5 (confidence ≥ 0.80 and 85% right, with higher confidence on right than wrong): missed on seed 2024 only.** After the fix, 100%, 100% and 92% of the predictions at 0.80 or higher were correct, so the first half holds on every seed. The second half failed on seed 2024, where wrong predictions averaged 0.704 confidence and correct ones 0.696. That is 3 wrong predictions out of 35 and a gap of 0.008, so I can't tell this from noise. I stopped because the next step is more test data, not another change: with 35 test posts, I can't show whether the model's confidence ordering is real on a seed with only 3 errors. If I had more time I'd collect another 100 or so posts so the held-out split is large enough that a gap of 0.008 either means something or clearly doesn't. I'd also add a check on the validation split before trusting any single seed. I did not lower the 0.80 threshold or the 85% target.

**`opinion` is my weakest label even though it passes criterion 2.** Its F1 is 0.706, 0.714 and 0.875 across the three seeds, against 0.80 or higher for `promo` and `news` on every seed. That clears the 0.55 target, so it isn't a miss, but it is what I'd fix next. After the 10-epoch run, the `opinion` errors go to `news` (2 posts on seeds 7 and 2024) and `question` (3 posts on seed 42, 2 on seed 7).

**The gap between what I meant my labels to capture and what the model learned:**
I defined `opinion` as a judgment, praise or complaint "whether or not it's tied to news", and `question` as asking for "a recommendation, a fix, or an opinion", so a post asking what people think is an `opinion` post phrased as a `question`, and my own definitions overlap. The matrix shows the model finds exactly that boundary hard: on seed 42, 3 of 9 `opinion` posts were called `question`, and on seeds 7 and 2024 `opinion` posts went to `news`, which matches the headline-with-a-take case I flagged in `criteria.md` as the hardest to write down.



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
