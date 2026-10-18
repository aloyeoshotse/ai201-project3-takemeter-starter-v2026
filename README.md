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
