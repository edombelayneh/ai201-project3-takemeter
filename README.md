# TakeMeter: A Classifier for Liminal-Space and Simulation-Theory Posts

This project is a text classifier. It reads posts from communities like r/backrooms, r/Glitch_in_the_Matrix, r/SimulationTheory, r/TrueBackrooms, and r/SCP. It sorts each post into one of three types: a real personal story, made-up shared fiction, or a real argument about reality. Full label rules and decisions are in `planning.md`. This file is the short, final summary.

## What This Project Does

People in these communities use similar words ("level," "simulation," "glitch," "parallel"), but they mean very different things by them. Some posts are real stories. Some posts add to a shared made-up world. Some posts are real arguments about whether reality is a simulation. TakeMeter sorts each post into one of these three labels:

- **`personal_anecdote`** — A real, specific thing that happened to the poster, at a real time and place.
- **`lore_worldbuilding`** — Adds to, or argues about, a shared fictional world (like Backrooms levels or SCP canon). It is not a real event.
- **`philosophical_argument`** — A general reasoned claim about reality. It is not about one personal event, and it does not add new fiction.

The model checks the labels in this order: anecdote first, then lore, then argument. See `planning.md` for the full reasoning and tricky examples.

## The Dataset

- **162 labeled posts** collected from Reddit (title and body combined into one text field). The three labels are close to even: personal_anecdote 34.0%, lore_worldbuilding 33.3%, philosophical_argument 32.7%.
- The original goal was 200 posts. We landed at 162 instead. See **Spec Reflection** below for why.
- 4 collected posts were removed and not labeled at all: one post that had nothing to do with these topics, one post asking for feedback on a personal sci-fi story, one post that was just a gaming tech question, and one post that talked about a real suicide attempt. That last one was removed for safety and ethics reasons, no matter what label it might have gotten.
- Some labels were first guessed by an AI (Claude), then checked and fixed by hand. This is tracked in the `pre_labeled_by_llm` column in the dataset file. More on this in **AI Usage** below.

## The Model and Training

- Base model: `distilbert-base-uncased`, fine-tuned on the 162 labeled posts.
- Training ran for 7 rounds (epochs). Accuracy on the validation set went from 0.375 in round 1 up to 0.875 by rounds 6-7. Both the training error and validation error kept going down the whole time, so the model was not badly overfitting given how small the dataset is. But see the Reflection section for a closer, more doubtful look at what this accuracy number really means.
- Baseline: Groq's `llama-3.3-70b-versatile` model, given the same label definitions and one example per label, with no training. Tested on the same set of posts as the fine-tuned model.

## Evaluation Report

### Overall Accuracy

| Model | Accuracy |
|---|---|
| Baseline (Groq, no training) | **0.960** (25 out of 25 usable answers) |
| Fine-tuned (distilbert-base-uncased) | **0.880** (25 out of 25 test posts) |

**The fine-tuned model scored lower than the baseline.** This is the most important result in this whole report, and it is explained in detail in the Reflection section below. This result does not match the goal set in `planning.md`, which expected the fine-tuned model to score at least 10 points higher than the baseline.

### Scores for Each Label

**Baseline (Groq, no training):**

| Label | Precision | Recall | F1 | Count |
|---|---|---|---|---|
| personal_anecdote | 0.90 | 1.00 | 0.95 | 9 |
| philosophical_argument | 1.00 | 1.00 | 1.00 | 8 |
| lore_worldbuilding | 1.00 | 0.88 | 0.93 | 8 |

**Fine-tuned model:**

| Label | Precision | Recall | F1 | Count |
|---|---|---|---|---|
| personal_anecdote | 1.00 | 0.89 | 0.94 | 9 |
| philosophical_argument | 0.88 | 0.88 | 0.88 | 8 |
| lore_worldbuilding | 0.78 | 0.88 | 0.82 | 8 |

The fine-tuned model's weakest score is `lore_worldbuilding` precision (0.78). This means that when the model says "this is lore," it is wrong more often than the baseline ever was. The table below (the confusion matrix) shows this same pattern: lore_worldbuilding shows up in all 3 of the model's mistakes.

### Confusion Matrix (Fine-Tuned Model, Test Posts)

| Real label ↓ / Model said → | personal_anecdote | philosophical_argument | lore_worldbuilding |
|---|---|---|---|
| **personal_anecdote** | 8 | 0 | 1 |
| **philosophical_argument** | 0 | 7 | 1 |
| **lore_worldbuilding** | 0 | 1 | 7 |

### The Wrong Answers, Explained

**#1 — Real label: philosophical_argument. Model said: lore_worldbuilding (confidence 0.45)**
> "I find it hilarious that most stories are about parallel universes caused by someone taking one path or another… and not about the uncountable and effectively identical parallel worlds caused by particles..."

*Why it got this wrong:* This post makes fun of how fiction usually handles parallel universes, then makes a real point about particle physics. The word "stories," and the talk about fiction, probably pulled the model toward "lore." But the post is not adding to a made-up world — it's making an argument about real physics. The model is reading the *topic* (fiction-as-a-subject) instead of the *job* the post is doing (making an argument).

**#2 — Real label: personal_anecdote. Model said: lore_worldbuilding (confidence 0.45)**
> "There is a possibility that I accidentally created the 'Smiler' entity... 6 years ago, when the Backrooms was still a new thing, I made a Backrooms game on Roblox..."

*Why it got this wrong:* This is the exact kind of post flagged as tricky during labeling (see row 65 in the dataset, and the notes in `planning.md`). The poster is telling a real story about something they actually did (made a Roblox game). But the *subject* of that story is the origin of a piece of lore. The model seems to be reacting to words like "Smiler," "Backrooms," and "entity," instead of noticing the first-person, dated, real-life framing that should mark this as a true story. This matches a guess made before training even started: the model has trouble telling apart "written in first person" from "really happened to me."

**#3 — Real label: lore_worldbuilding. Model said: philosophical_argument (confidence 0.46)**
> "Hereditary Effects. What SCPs could theoretically have hereditary effects? Could 006 be passed down? Could 6979-A's reality-warping be inherited via Fetal Alcohol Syndrome? What else?"

*Why it got this wrong:* This post is written exactly like a real-world science question ("could X be passed down," "could Y be caused by Z"). That kind of sentence shape is normally how a `philosophical_argument` post sounds. The only clue that this is fiction is the SCP numbers, which apparently weren't enough on their own. This is the flip side of mistake #1: there, the topic looked fictional but the job was arguing; here, the sentence shape looks like arguing but the topic is fictional.

### Is This a Labeling Mistake or a Model Mistake?

These three posts were labeled the same way as every other post, using the same rules from `planning.md`. So this is not a case of labeling them inconsistently. It is a real gap: **short posts where the sentence shape (a question, a complaint) is the same across labels, and the only real clue is whether the topic is fiction or real.** All three wrong answers got a low confidence score — 0.45 to 0.46, barely above 1-in-3, which is random guessing for three labels. This suggests the model itself was unsure on these, not confidently wrong.

**What would help fix this:** more training posts that are short, one-idea, and shaped like a question ("could X cause Y?") for all three labels — not just for `philosophical_argument`. Right now the dataset's existing examples may have "question-shaped post" mostly under philosophical_argument (like "Could we be the Programmers...", "Is it illogical to assume..."), so the model may have learned a shortcut: "sounds like a question → argument." That shortcut breaks on SCP-style fiction questions.

### Sample Classifications

| Post (shortened) | Real Label | Model Said | Confidence |
|---|---|---|---|
| "I find it hilarious that most stories are about parallel universes..." | philosophical_argument | lore_worldbuilding | 0.45 |
| "There is a possibility that I accidentally created the 'Smiler' entity..." | personal_anecdote | lore_worldbuilding | 0.45 |
| "Hereditary Effects. What SCPs could theoretically have hereditary effects?..." | lore_worldbuilding | philosophical_argument | 0.46 |
| *[ADD: a post the model got RIGHT, with its confidence score]* | | | |
| *[ADD: a second post the model got RIGHT, with its confidence score]* | | | |

*[ADD: one sentence here saying why one of the correct answers above makes sense — for example: "The model correctly called [post] a personal_anecdote with [X] confidence, because the post is tied to one specific dated event, with no fiction words and no general reasoning language."]*

## Reflection: What the Model Actually Learned vs. What It Was Supposed to Learn

The labels were built around one main idea: *is this a real event, made-up shared fiction, or a real argument about reality?* This idea is supposed to work no matter what topic or words a post uses. But what the model actually seems to have learned is more like **a word-matching guesser with a weak sense of structure on top.** It does fine when the topic and the structure agree (a dated real story, with no fiction words; a Bostrom-style argument, with no story-telling). But it breaks exactly when topic and structure point in different directions (a real story about how a piece of lore was created; a fiction question that sounds like a real-world hypothesis).

This shows up clearly in the **baseline comparison** too: the much bigger 70B model (with zero training) beat the small fine-tuned model (0.96 vs 0.88) on this same test. As noted earlier, the baseline's near-perfect score is probably easy partly because the labels in this dataset have very different, easy-to-spot words (SCP numbers, named theorists, dated first-person stories). This suggests the task, as currently set up, may be easier to solve by word-matching than by truly understanding the structure the labels were meant to test — and a much bigger pretrained model is simply better at that kind of word-matching than a small DistilBERT trained on only about 130 examples. The fine-tuned model did not get *better* at the deeper structural job the taxonomy wanted; if anything, both models are partly solving an easier version of the task than the one `planning.md` was aiming for.

**What the model leaned on too much:** topic words as a stand-in for the real label (SCP/Backrooms words → lore; Bostrom/Mandela-effect words → argument; dated first-person storytelling → anecdote).

**What it missed:** posts where the sentence shape does not match the topic-typical words — a real story about how a piece of lore began, or a fiction question shaped like a real hypothesis.

## Spec Reflection

**One way the project guide helped:** the instruction to read 30-40 real posts before picking labels, and to find one truly tricky post per label before labeling 200 posts, stopped a weak set of labels from happening. The decision order (check anecdote first, then lore, then argument) came directly from trying to settle one tricky example before any real labeling began. That order is the reason later tricky posts (like the Roblox/Smiler one) could be settled at all, instead of staying stuck.

**One way the project diverged from the guide, and why:** the project did not reach the 200-post goal — it stopped at 162. While this project was happening, Reddit made it much harder to scrape posts automatically. Both plain web requests and the official PRAW tool got blocked, even from a personal computer, which the project guide's tool section did not expect. Lore posts specifically were also harder to find than expected, since r/backrooms has recently been full of talk about the 2024 Backrooms movie instead of the level/entity lore-building the sub was originally picked for. Given limited time, the choice was to keep all three labels above the 20% floor with a smaller, balanced dataset (162 posts, 32.7-34.0% each) instead of forcing 200 posts with one label way too small — this matches the backup plan already written in `planning.md` for an underrepresented label.

## AI Usage

AI tools were used in a few specific, clear ways in this project:

1. **Testing and fixing the labels.** Early on, the label definitions were checked against tricky example posts with Claude (for example: a post about a real dream described using liminal-space words; a post using lore words to describe a real-world theory about abandoned malls). This showed that the three labels needed a strict checking order (anecdote, then lore, then argument), not three separate definitions, since separate definitions overlapped too much. The final order in `planning.md` came directly from this process.

2. **Pre-labeling posts.** A large part of the 162 labeled posts were first guessed by Claude, using the exact label order and rules from `planning.md`, then checked and corrected by hand. This is tracked row-by-row using the `pre_labeled_by_llm` column in the dataset file. During this check, three posts were flagged by the AI as genuinely unclear (a post about the "void state"/shifting idea, a post quoting a channeled-entity conversation, and a post presenting a personal discovery as proof of a theory). These were kept, but flagged as uncertain, instead of being forced into a clean answer — and two of them turned out to directly explain the fine-tuned model's actual mistakes later.

3. **Checking content for fit and safety.** While reviewing the AI's label guesses, Claude also flagged posts that did not belong in this dataset at all, instead of forcing them into a label: one post with nothing to do with this topic, one post asking for personal feedback on a sci-fi story, one gaming tech-support question, and one post describing a real suicide attempt. All four were removed from the final dataset (kept in a separate "excluded" file) instead of being labeled — because they did not fit the task, and in the suicide-attempt case, because it was the right thing to do regardless of labeling.

4. **Writing the Groq baseline prompt.** The instructions given to the baseline model (Groq) were written by Claude, based directly on the label definitions and checking order from `planning.md`. This turned the step-by-step decision process into clear instructions a model could follow with no other context, since just listing three definitions (without the order) would not have explained the same decision process.

5. **Looking for patterns in the wrong answers.** After getting the fine-tuned model's 3 wrong answers, they were reviewed with Claude to look for a shared pattern. The pattern it suggested (a mix-up between question-style sentences and lore-vs-argument topics, plus a mix-up between first-person writing and fictional subject matter) was checked against the actual wrong posts before being written into this report — both parts of the pattern held up after a closer look, rather than being accepted without checking.

What was changed or corrected: not every AI label guess was kept as-is — several were corrected by hand during review (the exact number was not tracked separately, but a `pre_labeled_by_llm=True` row means "this started as a guess," not "this is unreviewed"). The decision to remove the off-topic and sensitive posts was one place the AI's suggestion was followed directly, since the reasoning behind it (it doesn't fit the labels; it's the safe and ethical choice) held up on its own.
