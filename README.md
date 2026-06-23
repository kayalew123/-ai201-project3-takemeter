# TakeMeter — r/nba Discourse Classifier
**AI201 Project 3 | Kidus Ayalew**

TakeMeter is a fine-tuned text classifier that evaluates discourse quality in r/nba by categorizing posts and comments into three types: `analysis`, `hot_take`, and `reaction`. The goal is to distinguish between posts that make structured arguments backed by evidence, posts that assert bold opinions without support, and posts that express immediate emotional reactions to recent events.

---

## Community Choice

I chose **r/nba** because it produces a natural and recognizable spectrum of discourse quality. Community members actively distinguish between these types of posts — phrases like "actual analysis" and "hot take" are part of the subreddit's vocabulary. The variety of thread types (trade reactions, player rankings, roster analysis, coaching hires) made it easy to collect diverse examples across all three labels.

---

## Label Taxonomy

**`analysis`** — The post makes a structured argument using specific, verifiable evidence such as stats, historical comparisons, salary cap figures, or tactical observations. The evidence does real argumentative work, not just decoration.

- Example 1: "Randle isn't a bad player but he doesn't offer much flexibility in team building. Wolves net rating over the last two seasons: Ant ON Randle ON +3 in 4157 minutes, Ant ON Randle OFF +3 in 1759 minutes."
- Example 2: "There is way less value in a bad team having their own draft picks after the tanking and lottery reforms. The Thunder post Westbrook and the Nets post KG/Pierce are the teams that did this most clearly."

**`hot_take`** — A bold, confident opinion stated without meaningful supporting evidence. The claim may be true but the post asserts rather than argues.

- Example 1: "Brunson over SGA, Wemby, and Luka is the definition of reactionary."
- Example 2: "The Luka trade continues to be the biggest joke of all time."

**`reaction`** — An immediate emotional response to a specific recent event. The post would not make sense without knowing about the triggering event.

- Example 1: "JAYLEN BROWN YOU ARE SAFE"
- Example 2: "I could've sworn they were going to take the Boston deal."

---

## Data Collection

**Source:** r/nba — public posts and comments collected manually from active threads over a single day during the Giannis Antetokounmpo trade to the Miami Heat (June 2026).

**Threads used:** Giannis trade reaction threads, "who won the trade" analysis threads, player ranking community polls, team roster discussions, coaching hire threads, and free agency speculation threads.

**Labeling process:** Each comment was read individually and assigned one label based on the definitions above. Claude was used to suggest initial labels for batches of comments, which I then reviewed and corrected before adding to the CSV. All final labels represent my own judgment.

**Label distribution:**
| Label | Count | Percentage |
|---|---|---|
| analysis | 76 | 38.4% |
| hot_take | 78 | 39.4% |
| reaction | 44 | 22.2% |
| **Total** | **198** | **100%** |

**3 difficult labeling decisions:**

1. *"Draft picks are the lifeblood of a small market team like Milwaukee which struggles to sign FAs. You draft a Giannis tier player or you trade for one."* — Could be hot_take (bold assertion) or analysis (logical argument). Labeled **analysis** because the post makes a structured causal argument even without specific numbers.

2. *"Giannis didn't want to waste years not competing only to waste a year not competing. Giannis is looking like a clown if this is really the Heat roster."* — Could be analysis (understands the roster situation) or hot_take. Labeled **hot_take** because the framing is primarily rhetorical with no real evidence.

3. *"It has been less than 3 years since Dame joined the Bucks, from favorites in the East to trading your franchise player. I don't think anyone could have guessed how that would play out."* — Could be reaction or analysis. Labeled **reaction** because the post is emotionally processing the trade news, not making a structured argument.

---

## Fine-Tuning Approach

**Base model:** `distilbert-base-uncased` from HuggingFace

**Training setup:**
- Epochs: 3
- Learning rate: 2e-5
- Batch size: 16 (train), 32 (eval)
- Train/val/test split: 70% / 15% / 15%
- Framework: HuggingFace Transformers + Trainer API on Google Colab T4 GPU

**Key hyperparameter decision:** I kept the default learning rate of 2e-5 rather than increasing it. With only ~138 training examples, a higher learning rate risked overfitting quickly. The 2e-5 rate is the standard starting point for fine-tuning BERT-family models on small datasets and provides more stable convergence.

---

## Baseline Description

**Model:** Groq `llama-3.3-70b-versatile` (zero-shot)

**Prompt used:**
```
You are classifying posts from r/nba (NBA basketball community).
Assign each post to exactly one of the following categories.

analysis: The post makes a structured argument using specific, verifiable evidence such as stats, historical comparisons, or detailed tactical observations. The evidence does real argumentative work, not just decoration.
Example: "Giannis' usage rate in Miami's system will drop from 31% to around 24% based on how Spoelstra runs his bigs. This could extend his career."

hot_take: A bold, confident opinion stated without meaningful supporting evidence. The claim may be true but the post asserts rather than argues. Often provocative framing.
Example: "Steph Curry would average 45 a game in the 80s. The defense was terrible."

reaction: An immediate emotional response to a specific recent event such as a game, trade, or injury. Little to no argument — the post is expressing a feeling in the moment.
Example: "LMAOOOO THE BUCKS REALLY DID THAT"

Respond with ONLY the label name. Do not explain your reasoning.

Valid labels:
analysis
hot_take
reaction
```

**Collection method:** Each test example was passed to the Groq API individually with a 0.1s delay between requests to respect rate limits.

---

## Evaluation Report

### Overall Accuracy

| Model | Accuracy |
|---|---|
| Zero-shot baseline (Groq llama-3.3-70b) | 0.700 |
| Fine-tuned DistilBERT | 0.433 |
| Difference | -0.267 (regression) |

### Per-Class Metrics — Fine-Tuned Model

| Label | Precision | Recall | F1-Score | Support |
|---|---|---|---|---|
| analysis | 0.58 | 0.58 | 0.58 | 12 |
| hot_take | 0.43 | 0.50 | 0.46 | 12 |
| reaction | 0.00 | 0.00 | 0.00 | 6 |
| **accuracy** | | | **0.43** | 30 |
| macro avg | 0.34 | 0.36 | 0.35 | 30 |
| weighted avg | 0.40 | 0.43 | 0.42 | 30 |

### Confusion Matrix — Fine-Tuned Model

| | Predicted: analysis | Predicted: hot_take | Predicted: reaction |
|---|---|---|---|
| **True: analysis** | 7 | 4 | 1 |
| **True: hot_take** | 3 | 6 | 3 |
| **True: reaction** | 2 | 4 | 0 |

The most striking result is that the fine-tuned model predicted `reaction` zero times correctly — every true reaction was misclassified as either analysis or hot_take. The model learned to distinguish analysis from hot_take at a modest level (F1 of 0.58 and 0.46 respectively) but completely failed to learn the reaction boundary.

### 3 Wrong Predictions — Analysis

**Wrong prediction #1:**
- Text: "The gap in this team is not the role players. The issue is that Ant himself is an elite catch-fire scorer but not an offensive engine in the mold of SGA Jokic Brunson. He needs to develop elite conditioning..."
- True: `analysis` | Predicted: `hot_take` (confidence: 0.35)
- Why it failed: This post makes a structured argument but uses confident assertive language ("the issue is," "not an offensive engine") that pattern-matches to hot_take framing. The model likely learned surface-level confidence cues rather than the presence of actual reasoning.

**Wrong prediction #2:**
- Text: "I don't think the Dame trade can be considered poor roster construction. Just didn't work out and that's when it really went downhill."
- True: `analysis` | Predicted: `reaction` (confidence: 0.33)
- Why it failed: This short post references a past event (the Dame trade) in a way that resembles a reaction to recent news. The model couldn't distinguish between a brief analytical observation about history and an emotional reaction to current events. Short posts with low confidence scores were systematically misclassified.

**Wrong prediction #3:**
- Text: "A decade of pump faking star trades got them Giannis. Fair play Pat Riley."
- True: `reaction` | Predicted: `hot_take` (confidence: 0.33)
- Why it failed: This reaction post uses confident, evaluative language ("fair play") that resembles a hot take. The model hasn't learned that reaction posts can contain opinions — the defining feature is that they're triggered by a specific event, not that they're purely emotional.

### Sample Classifications

| Text | Predicted Label | Confidence | Notes |
|---|---|---|---|
| "Bam shot 31% last season, Giannis going to be shooting 65% to offset it lol" | hot_take | 0.51 | Correct — sarcastic exaggeration, no real evidence |
| "Wolves net rating with Ant ON Randle ON: +3 in 4157 minutes. Ant ON Randle OFF: +3 in 1759 minutes." | analysis | 0.61 | Correct — specific stats doing argumentative work |
| "JAYLEN BROWN YOU ARE SAFE" | hot_take | 0.44 | Wrong — this is a reaction to the Giannis trade news |
| "It's only a matter of time before we circle back to Jokic is unhappy followed by Wemby is unhappy. The beast must be fed." | reaction | 0.38 | Correct — reacting to the media narrative cycle |
| "Draft picks are the lifeblood of a small market team like Milwaukee. You draft a Giannis tier player or you trade for one." | hot_take | 0.35 | Wrong — this is analysis; model missed the causal argument |

---

## What the Model Learned vs. What I Intended

I intended the model to learn to distinguish posts based on whether they contain evidence-backed reasoning (analysis), freestanding bold assertions (hot_take), or event-triggered emotional responses (reaction).

What the model actually learned was a surface-level approximation of the analysis/hot_take boundary — it picked up on some cues about confident assertive language vs. structured reasoning — but it learned almost nothing about reaction as a distinct category. The model predicted reaction zero times on the test set, which means it found no reliable signal to separate reactions from the other two labels.

The core problem is that reactions in r/nba often look like short hot takes. "JAYLEN BROWN YOU ARE SAFE" is indistinguishable from an opinion to a model that doesn't know a trade just happened. The contextual trigger — the news event — is invisible to the classifier since each post is evaluated in isolation without the thread context. This is a fundamental mismatch between what the label captures (event-triggered response) and what the model can access (the text alone).

Additionally, with only ~44 reaction examples in training, the model had very little signal to learn from. The combination of small class size and context-dependent labels made reaction effectively unlearnable with this setup.

---

## Reflection on Spec

The spec helped most in the label design phase — the requirement to name a hard edge case before annotating forced me to write an explicit decision rule for hot_take vs. analysis before collecting data. This saved significant annotation inconsistency.

My implementation diverged from the spec in one key way: I collected all data in a single day during a major breaking news event (the Giannis trade), which created a topically narrow dataset. The spec recommends reading across different types of threads before committing to labels, which I did, but the actual data collection was dominated by Giannis trade threads. This likely made the model overfit to that specific event's vocabulary and reduced its ability to generalize.

---

## AI Usage

**Instance 1 — Label stress-testing:** I gave Claude my three label definitions and asked it to generate 10 borderline posts between hot_take and analysis. Several were genuinely ambiguous, which led me to tighten the decision rule: evidence must do real argumentative work, not just decorate an opinion. I kept the tightened rule and discarded the generated posts (they were not added to the dataset).

**Instance 2 — Annotation assistance:** I used Claude to suggest initial labels for batches of Reddit comments during data collection. Claude suggested labels based on my definitions, and I reviewed and corrected every single one before adding to the CSV. In several cases I overrode Claude's label — notably, Claude sometimes labeled short reactions as hot_takes because of confident-sounding language, and I corrected these based on the event-trigger rule.

**Instance 3 — Failure analysis:** After seeing the wrong predictions, I asked Claude to identify patterns in the 17 misclassified examples. Claude identified two patterns I verified: (1) nearly all reaction misclassifications went to hot_take, confirming the surface similarity problem, and (2) low confidence scores (all around 0.33-0.36) appeared across almost all wrong predictions, suggesting the model was genuinely uncertain rather than confidently wrong. Both patterns held up when I re-read the examples myself.
