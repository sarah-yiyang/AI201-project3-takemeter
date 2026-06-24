# TakeMeter — planning.md

---

## Community

I chose **r/StardewValley**, the subreddit for the open-ended country-life RPG Stardew Valley (a farming/life-sim that supports 1–8 players and has a large, long-running player base).

Reasons: First, it's a community I follow and understand, so I can read a post and confidently tell what the author is actually trying to do — which is exactly the judgment a good annotation guide depends on. Second, the game is deliberately open-ended (farming, mining, fishing, relationships, decorating, combat), so people use the subreddit for genuinely different purposes rather than one repetitive activity.

Fit for classification task: there are variety of posts:  people *asking* for concrete help with mechanics, people *sharing* opinions and starting conversations, and people *showing off* farms, screenshots, and achievements. 

---

## Labels

I use three labels. Each is defined by the author's **primary intent**, not by surface features like whether the title ends in a question mark or whether an image is attached.

### 1. Discussion / Opinion
**Definition:** A post whose main goal is to spark a community conversation by sharing an opinion, hot take, experience, or observation and inviting others' views *for their own sake*, rather than seeking help with a problem or decision of the author's own.

**Example posts:**
- *"What to tell your past self?"* — "Has anyone ever used signs or some kind of item to remember your first time playing Stardew (or even just booting up a new save)? If so, what reminds you of your past Stardew playing self and what would you tell them?"
- *"Prison for dummies"* (image post) — "After almost 2000 hours it still happens from time to time. Guess I'm just stupid 😒"

### 2. Help / Question
**Definition:** A post whose main goal is to get assistance with the author's *own* situation — a gameplay problem, bug, strategy, or pending decision — so they can act on the replies, even when the question is subjective and has no single right answer.

**Example posts:**
- *"PLEASE answer if you've completed or trying to complete the Desert Skull Mine"* — "I'm preparing for the desert mine (again)… I have a chest packed with explosives, food, and weapons. I still cannot get down 100 floors. What was your method?"
- *"Woke up with Bream in my inventory?? I have not fished for over a month"* — "My inventory was empty except my tool bar the night before and when I awoke: fish! Please advise."

### 3. Showcase
**Definition:** A post whose main goal is to display something the author made, did, achieved, or captured — a farm, build, achievement, screenshot, memorable moment, or creation/resource — for others to see and react to, rather than to ask for help or debate a topic.

**Example posts:**
- *"I reached the summit! Rate my farm and my home"* — "This took me like 200+ hours… I guess I'm wondering what other people would say about my setup. Either way doing this feels like a weight off my shoulders."
- *"A sweet pea grave offering for Grandpa!"* (image post) — "The way this sweet pea spawned right in front of grandpa's grave makes it look like someone came by to leave some flowers!"

---

## Hard edge cases

The genuinely ambiguous posts sit on the boundaries between these three intents. I've already hit each of the main ones in my examples:

**Help/Question vs. Discussion/Opinion.** A post can be phrased as a question but not actually be help-seeking — it's a prompt with no correct answer.
- *"Dried Mushrooms!?"* — "I found GOLD! Besides aged wine, what are you guys selling? Also I don't dry the common mushrooms…" Reads like a question but is really sharing an observation and inviting others to chime in.
- *"If you could only rely on information inside the game… what would be hardest to find out, and which part would be hardest to get through?"* A question in form, but open-ended and discussion-promoting.

**Showcase vs. Discussion/Opinion.** A post can include a screenshot/achievement *and* invite conversation.
- *"The most disgusting achievement I have ever achieved"* (image) — "I need to take a hot shower… This feels too gross lol." Has discussion-bait phrasing, but the core of the post is displaying a specific achievement + screenshot.

**How I'll handle it during annotation.** I apply a consistent **primary-intent tie-breaker** so I label the same way every time:
1. **Ask what response the author most wants.** A concrete answer that solves a problem → Help/Question. Other people's takes/stories/reactions with no right answer → Discussion/Opinion. Eyes on what they made/did → Showcase.
2. **Don't let surface features decide.** A question mark doesn't force Help/Question; an attached image doesn't force Showcase.
3. **For Showcase vs. Discussion:** if the post wouldn't exist without the specific thing being shown (the achievement, the screenshot, the build), label Showcase even if it ends with a conversational hook.
4. **For Help vs. Discussion:** if the author has a *pending decision or problem of their own* and wants replies they'll act on, label Help/Question — even if the replies are subjective opinions or experiences. If there's no personal action pending and they just want the community's views for their own sake, label Discussion/Opinion.
5. When still split after these steps, I label by the dominant intent and log the post as a borderline case with a one-line note, so my decisions stay reviewable and consistent.

---

## Why the labels matter

These distinctions matter to the community because they map onto how people actually use the subreddit and what kind of reply each post wants. A Help/Question poster is stuck and needs an answer; burying it under casual chat leaves them unhelped. A Discussion/Opinion poster wants conversation, not corrections. A Showcase poster wants appreciation, not advice. Separating these intents is exactly what lets the right kind of response reach the right post — which is also what makes them worth measuring with TakeMeter.

---

## Data collection plan

**Where.** I'll collect posts from r/StardewValley itself, pulling from a mix of feeds so I don't oversample one kind of content: **Hot** and **Top** (which skew toward Showcase and popular Discussion) plus **New** (which surfaces more raw Help/Question posts before they get buried or removed). For each post I'll record the title, the body text, and a note of whether an image was attached, since that context matters for borderline Showcase calls. I'll skip removed/deleted posts, pure crossposts, and posts that are only a title with no usable content.

**How many.** I'm targeting **~200 labeled examples total** across the three labels (Discussion/Opinion, Help/Question, Showcase), with a goal of each label being well enough represented to evaluate on its own. The natural feed is skewed, so I won't force a perfectly even split — I'll label as I collect and watch the per-label counts so I can steer sampling toward whichever label is falling behind.

**If a label is underrepresented after 200 examples.** The natural feed is skewed (Showcase and Discussion are potentially more common than genuine Help/Question), so one or more labels may fall short. My plan, in order:
1. **Targeted sampling** instead of pulling more random posts. For the thin label I'll search the subreddit with intent-specific cues — e.g. for Help/Question, search/sort terms like "help," "how do I," "bug," "stuck," and browse the **New** queue where unanswered questions live; for an underrepresented Showcase, filter by the community's image/flair tags.
2. **Pull from a wider time window** (older Top posts, multiple months) so I'm not limited to whatever is trending this week.
3. **Cap the over-represented labels** so I don't inflate the dataset just to balance — better to have ~50 solid Help/Question examples than to pad another class to match.
4. **Document the imbalance.** If a label genuinely stays rare, I'll report the final per-label counts and treat that class's metrics as lower-confidence rather than pretending the data is balanced. Real class frequency is itself a finding worth noting.

---

## Evaluation metrics

Accuracy alone is misleading here. If the dataset is even mildly imbalanced (e.g. Showcase and Discussion outnumbering Help/Question), a model can score high accuracy by leaning on the common classes while quietly failing the rare one — and the rare class (people who actually need help) is often the one that matters most. So I evaluate the fine-tuned model on the held-out test set with four things, all produced directly from the predictions:

- **Overall accuracy** (`accuracy_score`) — reported as a familiar baseline and sanity check, but explicitly *not* the metric I judge success on.
- **Per-class precision, recall, and F1** (`classification_report`) — the core metrics. These tell me *where* the model fails, not just how often. For each label I care about both directions: **recall** (of all true Help/Question posts, how many did I catch — missing these strands people who need answers) and **precision** (of everything I labeled Help/Question, how many really were — false positives send the wrong kind of response). The report also gives me the **macro-averaged F1**, which weights all three classes equally so a strong majority class can't hide weakness on a minority class — this is my **primary single-number metric**.
- **Confusion matrix** — essential for *this* task because my labels have known gray zones. I expect mistakes to cluster on exactly the boundaries I flagged in Hard edge cases (Help↔Discussion, Showcase↔Discussion). The matrix shows whether errors concentrate on those defensible borderline pairs or scatter in ways that signal a real labeling/model problem.
- **Qualitative error analysis with confidence** — I take the softmax probability of each prediction and list the misclassified test examples (text, true label, predicted label, and the model's confidence). I'll review these and analyze ~3 in depth. Pairing each error with its confidence is the key signal: a wrong prediction made at low confidence is the model honestly hesitating on a genuine gray-zone post, whereas a wrong prediction at high confidence points to a real weakness or a label-definition problem worth fixing.

In short: macro-F1 to judge overall quality fairly across classes, per-class precision/recall to understand the cost of each error type, the confusion matrix to confirm the errors are the "honest" boundary ones rather than something broken, and the confidence-tagged error list to interpret *why* each mistake happened.

---

## Definition of success

**Pass/fail criteria.** I set these *before* seeing the test results, and each is a number I can read straight off my evaluation outputs, so at the end I can tick each box yes/no rather than argue myself into "success":

1. **Beats the majority-class baseline by a clear margin.** The majority classifier (always predict the most common label) has a macro-F1 I can compute exactly from the test-set class split. My model must score **at least +0.20 macro-F1 above that baseline** — proof it's actually learning the three intents, not riding class frequency.
2. **No dead class.** Every class has **F1 ≥ 0.60**, so the model works across all three intents rather than carrying one class with a near-zero score.
3. **Help/Question recall ≥ 0.70.** This is the highest-stakes class — a missed help request strands a person — so I hold its recall to a fixed floor even at some cost to its precision.
4. **Errors are the defensible kind.** Of all misclassified test posts, **at least half fall on the two known borderline pairs** (Help↔Discussion, Showcase↔Discussion) read off the confusion matrix — i.e. mistakes cluster where a human would also hesitate, not on clear-cut posts.
5. **High-confidence predictions are trustworthy.** Among predictions with softmax confidence **≥ 0.90, accuracy ≥ 0.90** — the model is calibrated enough that its confident calls are reliable and its mistakes skew toward low confidence.

I count it **successful** if it meets criteria 1–3 (it works) and **deployment-grade** if it also meets 4–5 (its errors are honest and its confidence is trustworthy). If I miss a target, I report the actual number and treat the gap as a finding rather than rounding it up to a pass.

**Why these numbers and not higher.** Because my own annotation has irreducible ambiguity on the edge cases, I don't expect — or trust — near-perfect scores; a model claiming ~99% would more likely mean leakage or an over-easy split than a genuinely better classifier. The floors above are deliberately modest and reachable on a ~200-example dataset, while still being strict enough that a lazy or broken model would fail them.

---

## AI Tool Plan

This project has no code to generate, so AI tools help in three specific places — all aimed at making my **label definitions and annotation** sharper, not at doing the labeling for me. The model is treated as a fallible assistant whose output I always verify.

### 1. Label stress-testing (before annotation)
**What I'll do.** I'll give an LLM my three label definitions plus the Hard edge cases section and ask it to generate **5–10 posts that deliberately sit on the boundary between two labels** (Help↔Discussion and Showcase↔Discussion specifically). Then I try to classify each one using only my written definitions and tie-breaker rules.

**How it sharpens the guide.** Any generated post I *can't* place cleanly exposes a gap in my definitions — that's the signal to tighten the wording or add a tie-breaker rule **now, before I annotate 200 real examples**, when fixing it is cheap. I'll iterate (regenerate → re-tighten) until the borderline posts are ones my rules can resolve consistently. I'll keep a couple of the trickiest generated posts as documented edge-case examples (clearly marked as synthetic, not counted in the dataset).

### 2. Annotation assistance (pre-labeling)
**Decision: I will *not* use an LLM as the source of truth for labels, but I *may* use it as an optional pre-labeling assist on a batch**, then review every example myself before it enters the dataset. Every label in the final dataset is one I personally confirmed.

**If I do pre-label:**
- **Tool:** Claude, prompted with my exact label definitions and tie-breaker rules so its suggestions follow the same guide I do.
- **Tracking for disclosure:** I'll add a column to my dataset (e.g. `prelabeled_by_ai` = true/false, plus `ai_suggestion` and `final_label`) so I can report exactly which examples were AI-assisted and how often I overrode the suggestion. This row-level record feeds directly into my AI usage disclosure.
- **Guard against anchoring:** for at least a sample of the batch I'll label blind (before seeing the AI's guess) to check whether the AI's suggestion is quietly biasing my own calls.

**If I don't pre-label,** the dataset is fully hand-labeled and I'll state that plainly in the disclosure.

### 3. Failure analysis (after evaluation)
**What I'll do.** After running the model, I'll take the exported list of **wrong predictions** (text, true label, predicted label, confidence — the output my eval script already produces) and give it to an LLM, asking it to **group the errors into patterns** rather than just summarize them.

**What I'll look for:**
- Which **label pair** dominates the errors, and whether that matches the borderline pairs I predicted in Hard edge cases.
- Whether errors cluster on a **surface feature** I warned against (e.g. image posts pushed to Showcase, question marks pushed to Help/Question).
- Whether **high-confidence errors** share a theme — a sign of a label-definition problem rather than honest ambiguity.

**How I'll verify the patterns myself.** The AI's claimed patterns are hypotheses, not findings. For each pattern, I'll go back to the actual misclassified posts and confirm it holds by **reading the examples and checking the counts against the confusion matrix** — only patterns I can see in the real data go into my write-up, with the specific examples cited. If the AI over-generalizes from one or two cases, I drop the claim.

---

## Baseline Run

**Setup.** Using Groq `llama-3.3-70b-versatile` (temperature 0), prompted with my label definitions + one example per label, evaluated on the 30-post test set. All 30 responses parsed cleanly.

**Results.**

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| Discussion/Opinion | 0.73 | 0.73 | 0.73 | 11 |
| Help/Question | 0.88 | 0.78 | 0.82 | 9 |
| Showcase | 0.82 | 0.90 | 0.86 | 10 |
| **Accuracy** | | | **0.80** | 30 |
| **Macro avg** | 0.81 | 0.80 | 0.80 | 30 |

### Reflection

**Where it struggled.** **Discussion/Opinion is clearly the weakest class** (F1 0.73, vs. 0.82 Help and 0.86 Showcase) — and it's weak in both directions (precision 0.73 and recall 0.73), so the model both misses true Discussion posts and wrongly labels other posts as Discussion. Showcase was easiest (recall 0.90), and Help/Question had high precision (0.88) but lower recall (0.78), meaning a couple of real Help posts got pulled into another class.

**Hypothesis (to test after fine-tuning).** Discussion/Opinion is the hub of confusion, exactly as my Hard edge cases predicted that it sits on both gray boundaries, so errors should concentrate there in two specific ways:
1. **Help → Discussion:** subjective or decision-seeking questions (e.g. "which character should I romance?") read like opinion prompts, so true Help posts leak into Discussion. This is what drags Help's recall down to 0.78.
2. **Discussion ↔ Showcase:** Discussion posts anchored on a screenshot/achievement get misread as Showcase (hurting Discussion recall), while some genuine Discussion gets the Showcase label (Showcase precision 0.82, below its 0.90 recall).

In short, I expect the confusion matrix to show most errors on the **Help↔Discussion** and **Showcase↔Discussion** pairs, with very few Help↔Showcase mix-ups (those two are the least similar).

**What I'll check after fine-tuning.** (a) Does Discussion/Opinion's F1 rise the most? (b) Does the confusion matrix confirm errors cluster on the two predicted boundary pairs rather than scattering? (c) Does Help/Question recall improve past my 0.70 success floor? *Caveat:* per-class support is tiny (9–11 posts), so one or two reclassified posts swing these numbers a lot — I'll read changes as directional, not precise.