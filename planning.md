# TakeMeter — Planning Document
**AI201 Project 3 | Kidus Ayalew**

---

## Community

I chose **r/nba** as my community. It's one of the most active sports discussion communities on Reddit, with millions of weekly visitors and a wide spectrum of discourse quality. NBA fans are known for posting everything from detailed statistical breakdowns to pure emotional reactions — often in the same thread. The community actively distinguishes between these types of posts, frequently calling out "hot takes" vs. "actual analysis," which makes the labels feel grounded in real community norms rather than imposed from outside. The variety of post types (trade reactions, player rankings, draft analysis, coaching hires) ensures diverse training examples.

---

## Labels

I defined three labels that capture meaningful and distinct types of discourse in r/nba:

**`analysis`** — The post makes a structured argument using specific, verifiable evidence such as stats, historical comparisons, salary cap figures, or detailed tactical observations. The evidence does real argumentative work, not just decoration.

- Example 1: "Randle isn't a bad player but he doesn't offer much flexibility in team building. Wolves net rating over the last two seasons: Ant ON Randle ON +3 in 4157 minutes, Ant ON Randle OFF +3 in 1759 minutes."
- Example 2: "There is way less value in a bad team having their own draft picks after the tanking and lottery reforms. The Thunder post Westbrook and the Nets post KG/Pierce are the teams that did this most clearly."

**`hot_take`** — A bold, confident opinion stated without meaningful supporting evidence. The claim may be true but the post asserts rather than argues. Often uses provocative or definitive framing.

- Example 1: "Brunson over SGA, Wemby, and Luka is the definition of reactionary."
- Example 2: "The Luka trade continues to be the biggest joke of all time."

**`reaction`** — An immediate emotional response to a specific recent event such as a game, trade, injury, or news drop. Little to no argument — the post is expressing a feeling in the moment. The post would not make sense without the triggering event.

- Example 1: "JAYLEN BROWN YOU ARE SAFE"
- Example 2: "I could've sworn they were going to take the Boston deal."

---

## Hard Edge Cases

The hardest anticipated boundary is between **hot_take** and **analysis**. Some posts cite one statistic or make a historical comparison but use it purely for rhetorical effect rather than as part of a genuine argument.

**Example ambiguous post:** "Giannis didn't want to waste years not competing only to waste a year not competing. Giannis is looking like a clown if this is really the Heat roster."

This could be hot_take (bold opinion, no evidence) or a weak analysis (implies understanding of the roster situation). Decision rule: if the post provides specific verifiable evidence that would support the claim even if you removed the opinion framing, label it analysis. If the evidence is vague, cherry-picked, or the post is primarily making a rhetorical point rather than building an argument, label it hot_take. The above example is hot_take — it's opinion-forward with no real evidence.

A second hard case is **reaction vs. hot_take** for posts that respond to a recent event but also express a bold opinion. Decision rule: if the post would not make sense without knowing about a specific recent event (a trade, a game result, a signing), label it reaction even if it contains an opinion. Hot takes are freestanding — they could have been posted on any day.

---

## Data Collection Plan

**Source:** r/nba — public posts and comments collected manually from active threads.

**Thread types used:**
- Giannis trade reaction threads (reactions, hot takes)
- "Who won the trade" analysis threads (analysis, hot takes)
- Player ranking community polls (hot takes, analysis)
- Team roster discussion threads (analysis, reactions)
- Coaching hire threads (reactions, analysis)
- Free agency speculation threads (analysis, hot takes)

**Target distribution:** ~67 per label (200 total). Reactions were harder to find at balanced volume so I targeted game threads and breaking news threads specifically for that label.

**If a label is underrepresented:** Search for threads that naturally generate that type of post. For reactions, game threads and breaking news drops work best. For analysis, "team discussion" and "who won the trade" threads generate more structured posts.

---

## Evaluation Metrics

I used **overall accuracy** and **per-class F1 score** as my primary metrics.

Accuracy alone is insufficient because the dataset, while balanced, could still mask per-class failures. A model that learns to predict hot_take for everything would achieve ~40% accuracy — which looks mediocre but actually means it learned nothing useful about the other two classes. Per-class F1 captures both precision and recall for each label, revealing whether the model is actually learning all three distinctions or just predicting the majority class.

I also examined the **confusion matrix** to identify which specific label pairs the model confuses most, since this reveals where the decision boundaries are failing.

---

## Definition of Success

A classifier is genuinely useful if:
- Overall accuracy ≥ 0.70 on the test set
- All three per-class F1 scores ≥ 0.60
- The confusion matrix shows no single label with F1 = 0.00

Below these thresholds, the model is not reliably distinguishing between the labels and would not be trustworthy in a real community tool.

---

## AI Tool Plan

**Label stress-testing:** Before annotating, I will give Claude my label definitions and ask it to generate borderline posts between hot_take and analysis. If the generated posts are hard to classify, I'll use that as a signal to tighten my definitions before committing to 200 examples.

**Annotation assistance:** I plan to use Claude to pre-label batches of examples before reviewing them myself. I'll provide my label definitions alongside unlabeled posts and ask for one label per post, then review and correct every suggestion. Any AI-assisted pre-labeling will be disclosed in my AI usage section.

**Failure analysis:** After fine-tuning, I plan to paste my wrong predictions into Claude and ask it to identify common patterns across misclassified examples — things like post length, sarcasm, or specific label pairs. I will verify any patterns myself by re-reading the examples before writing them up.

---

## Hard Annotation Decisions (3 Examples)

**1. "Draft picks are the lifeblood of a small market team like Milwaukee which struggles to sign FAs. You draft a Giannis tier player or you trade for one. Either way you need as much draft capital as you can get."**
Could be hot_take (bold assertion) or analysis (makes a logical argument about small market dynamics). Labeled **analysis** because the post makes a structured causal argument even without citing specific numbers — the reasoning is the evidence.

**2. "Giannis didn't want to waste years not competing only to waste a year not competing. Giannis is looking like a clown if this is really the Heat roster."**
Could be analysis (understands the roster situation) or hot_take. Labeled **hot_take** because the framing is primarily rhetorical and no specific evidence supports the claim.

**3. "It has been less than 3 years since Dame joined the Bucks, from favorites in the East to trading your franchise player. I don't think anyone could have guessed how that would play out."**
Could be reaction (triggered by the trade news) or analysis (makes a historical observation). Labeled **reaction** because the post is emotionally processing the Giannis trade news and reflects on recent history as part of that reaction, not as a structured argument.
