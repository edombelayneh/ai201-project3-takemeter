# TakeMeter: Planning

## 1. Community

I chose communities about liminal spaces and simulation theory: **r/backrooms**, **r/Glitch_in_the_Matrix**, and **r/SimulationTheory**. I picked this topic because it is fun to read, and it is not about politics.

This community is a good fit because people post very different kinds of things. Some posts are made-up stories. Some posts are real arguments about reality. Some posts are personal stories about strange things that happened. These three kinds of posts feel very different when you read them, but they can look similar on the surface (they all use words like "glitch," "simulation," "liminal"). That makes this a good test for a classifier: the model has to learn the real difference, not just spot a keyword.

## 2. Labels

I am using three labels. I check them in this order: anecdote first, then lore, then argument. This order helps keep the labels separate.

### Label 1: `personal_anecdote`
**Definition:** A post is `personal_anecdote` if it tells a true story about something strange that happened to the poster, at a real time and place.

- Example 1: "Took a wrong turn in my office building after hours, ended up in a hallway I swear doesn't exist on the floor plan. Walked for what felt like ten minutes before finding the exit I came in from."
- Example 2: "Drove the same route home I take every day and ended up on a street I've genuinely never seen before. GPS said I was on the right road the whole time. Lasted maybe 30 seconds before everything looked normal again."

### Label 2: `lore_worldbuilding`
**Definition:** A post is `lore_worldbuilding` if it adds new made-up content to the shared fictional world, like a new level, a new creature, or a new rule, and is not a real story about the poster.

- Example 1: "Level 37: The Drowned Archive. Fluorescent lighting replaced by dim emergency red. Entities here are silent and only move when you're not looking directly at them. Survival tip: never check your phone flashlight."
- Example 2: "Proposing a new rule for the Backrooms canon: noclipping happens more often in buildings constructed after 1980 due to inconsistent structural memory in the simulation's render distance."

### Label 3: `philosophical_argument`
**Definition:** A post is `philosophical_argument` if it makes a general reasoned claim about whether reality is a simulation, and is not a personal story and not new fiction.

- Example 1: "If consciousness can in principle be computed (per functionalism), and computing power keeps increasing, then statistically we're far more likely to be one of the simulated minds than the one 'base reality' mind. This is Bostrom's trilemma — happy to walk through the math."
- Example 2: "The Mandela Effect isn't really evidence of timeline shifts — false collective memory is well documented in psychology (e.g. misattribution from similar-but-different stimuli). Here's why I think the 'simulation glitch' explanation is unfalsifiable rather than wrong."

## 3. Hard edge cases

The hardest case is a post that tells a personal story but also uses it to make a bigger argument about reality. For example: "I dreamed I was stuck in an endless yellow hallway, and I woke up sure I had been there before — this is why I think dreams might be glimpses of other simulated layers."

This post is hard because it has a personal story AND an argument. My rule: I check `personal_anecdote` first, always. If the post is about something that happened to the poster, it is `personal_anecdote`, even if the poster also adds a theory about it. The theory part does not change the label, because the core of the post is still "this happened to me."

A second hard case is a post that uses lore words (like "level" or "noclip") to describe something that might be real, like: "I think Backrooms levels are just abandoned malls and buildings that already exist in real life — has anyone mapped real locations to specific levels?" This is hard because it sounds like lore but is actually a claim about the real world. My rule: if the poster is not adding new fiction, but instead making a claim about reality, it goes to `philosophical_argument`, not `lore_worldbuilding`.

## 4. Data collection plan

I will collect posts and comments directly from r/backrooms, r/Glitch_in_the_Matrix, and r/SimulationTheory. I expect each subreddit to lean toward a different label:
- r/backrooms → mostly `lore_worldbuilding`
- r/Glitch_in_the_Matrix → mostly `personal_anecdote`
- r/SimulationTheory → mostly `philosophical_argument`

My goal is at least 200 examples total, with at least 20% (about 40 examples) per label. If one label is underrepresented after collecting 200 examples, I will go back to the subreddit that best matches that label and collect more examples specifically for it, rather than relabeling other posts to force balance. If a label still stays too small after that, I will widen my source to a close subreddit (for example, r/highstrangeness for more philosophical_argument posts) instead of changing the label definitions.

## 5. Evaluation metrics

Accuracy alone is not enough, because if one label has more examples than the others, a model could get high accuracy just by guessing the biggest label every time. So I will also report:

- **Precision and recall per label**, so I can see if the model is bad at one specific label even if it looks good overall.
- **A confusion matrix**, so I can see which labels the model mixes up with each other (for example, mixing up `personal_anecdote` and `philosophical_argument`).
- **F1 score per label**, since it balances precision and recall in one number, which is useful for comparing labels side by side.

These metrics matter for this task because the three labels can look similar on the surface (similar words, similar topics), so I need to know exactly *where* the model gets confused, not just *how often*.

## 6. Definition of success

I want these criteria to be things I can check with a yes/no answer at the end, not just a feeling. So here are exact numbers, checked against my test set:

- **Beats the baseline:** the fine-tuned model's overall accuracy is at least 10 percentage points higher than the Groq zero-shot baseline's accuracy on the same test set. (Pass/fail: yes or no, read directly from the evaluation report.)
- **No weak label:** every label has recall of at least 0.60 and F1 of at least 0.65 for the fine-tuned model. (Pass/fail: check each row of the per-label table.)
- **Confusion is explainable:** when I list the wrong predictions, at least 80% of them are confusions between two labels that I already flagged as hard edge cases in Section 3 (anecdote vs. argument, or lore vs. argument). If most wrong predictions are something else entirely (like lore vs. anecdote, which should not be confusing), that's a sign something is wrong with the labels or the data, not just a hard task.

"Good enough for deployment" means hitting all three checks above. If the model beats the baseline but misses on the recall/F1 check or the confusion check, I will treat that as a partial success and say so directly in the evaluation report, rather than rounding up. It does not need to be perfect — even a model that is right most of the time, with explainable mistakes, can help moderators sort posts faster than reading everything themselves.

## 7. AI Tool Plan

This project has no code-generation step like the implementation projects, so AI tools help in three specific spots: stress-testing my labels, speeding up annotation, and finding patterns in errors.

### Label stress-testing
Before I annotate 200 examples, I will give an AI tool (Claude) my three label definitions and my Section 3 edge cases, and ask it to write 5–10 short posts that sit right on the boundary between two labels (for example, 3 anecdote-vs-argument posts, 3 lore-vs-argument posts, a couple of anecdote-vs-lore posts even though I think that pair is unlikely to overlap). I will then try to label each one myself using only my written definitions, with no extra context. If I can't label a generated post cleanly, that tells me my definitions are not tight enough, and I will rewrite them before I start the real 200-example annotation.

### Annotation assistance
I will use an LLM (Groq's llama-3.3-70b-versatile, same model as my baseline) to pre-label a first pass over my collected posts, using my label definitions as the prompt. I will not just accept these labels — I will review every single one myself and correct any that are wrong, since the whole point of the dataset is that it reflects my own judgment, not the LLM's. To track this for disclosure, I will add a column to my dataset CSV called `pre_labeled_by_llm` (true/false) so it's clear in the final dataset which examples started from an LLM suggestion versus which I labeled from scratch.

### Failure analysis
After I get my test set predictions, I will give the AI tool the full list of wrong predictions (post text, true label, predicted label) and ask it to identify any systematic pattern — for example, "the model struggles with short posts," "the model confuses anecdote and argument when the post mentions dreams," or "the model is biased toward the majority label." I will not put this pattern directly into my evaluation report. Instead, I will go back to the actual wrong examples myself, re-read them, and check whether the suggested pattern actually holds up across most of the errors, or if the AI tool overgeneralized from just one or two examples. Only patterns I've personally verified against the real misclassified posts go into the final report.
