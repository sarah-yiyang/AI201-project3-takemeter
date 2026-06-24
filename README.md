# TakeMeter — r/StardewValley Post-Intent Classifier

TakeMeter classifies posts from the [r/StardewValley](https://www.reddit.com/r/StardewValley/) subreddit by the **author's primary intent** into three labels: **Discussion/Opinion**, **Help/Question**, and **Showcase**. The goal is a classifier that could route community posts to the kind of response each one actually wants — answers for help-seekers, conversation for discussion-starters, and appreciation for people showing off their work.

This README is the full project report. Design rationale for the labels, data plan, and success criteria lives in [planning.md](planning.md); this document covers the build, the evaluation, and the reflection.

---

## 1. Community & Task

**Community:** r/StardewValley — the subreddit for the open-ended farming/life-sim *Stardew Valley*.

**Why it fits a classification task:** because the game is deliberately open-ended (farming, mining, fishing, relationships, decorating, combat), people use the subreddit for genuinely different purposes. The discourse splits cleanly into three communicative intents — asking for help, starting a conversation, and showing off — while still having a meaningful gray zone (a screenshot can double as a discussion prompt; a subjective question can be a decision request). That mix of separable classes plus real boundary cases is what makes the task non-trivial.

**Task:** single-label, 3-class text classification on the post's title + body.

---

## 2. Labels

Each label is defined by the author's **primary intent**, not by surface features (a question mark doesn't force Help; an image doesn't force Showcase). Full definitions, examples, and the edge-case tie-breaker are in [planning.md](planning.md).

| Label | One-line definition |
|---|---|
| **Discussion/Opinion** | Main goal is to spark a community conversation by sharing an opinion/experience/observation and inviting others' views *for their own sake*, not to solve a problem of the author's own. |
| **Help/Question** | Main goal is to get assistance with the author's *own* situation (problem, bug, strategy, or pending decision) so they can act on the replies — even when the question is subjective. |
| **Showcase** | Main goal is to display something the author made, did, achieved, or captured (farm, build, achievement, screenshot, creation) for others to see and react to. |

**Examples (2 per label):**

- **Discussion/Opinion**
  1. *"What to tell your past self?"* — "Has anyone ever used signs or some kind of item to remember your first time playing Stardew…? What would you tell them?"
  2. *"Prison for dummies"* (image) — "After almost 2000 hours it still happens from time to time. Guess I'm just stupid 😒"
- **Help/Question**
  1. *"PLEASE answer if you've completed the Desert Skull Mine"* — "I have a chest packed with explosives, food, and weapons. I still cannot get down 100 floors. What was your method?"
  2. *"Woke up with Bream in my inventory?? I have not fished for over a month"* — "My inventory was empty… and when I awoke: fish! Please advise."
- **Showcase**
  1. *"I reached the summit! Rate my farm and my home"* — "This took me like 200+ hours… doing this feels like a weight off my shoulders."
  2. *"A sweet pea grave offering for Grandpa!"* (image) — "The way this sweet pea spawned right in front of grandpa's grave makes it look like someone came by to leave some flowers!"

**Known hard boundaries** (predicted before modeling): **Help ↔ Discussion** (subjective/decision questions that look like opinion prompts) and **Showcase ↔ Discussion** (posts anchored on a screenshot that also invite conversation).

---

## 3. Data Collection

- **Source:** public posts from r/StardewValley (Hot, Top, and New feeds, to avoid oversampling one content type). Public content only — no authenticated or private channels.
- **Format:** [data.csv](data.csv) with columns `text`, `label`, `notes` (the `notes` column records borderline-case reasoning and flags synthetic posts).
- **Size & balance:** **200 labeled examples**, hand-labeled by me. Final distribution is well balanced (no class exceeds 35%):

| Label | Count | Share |
|---|---|---|
| Discussion/Opinion | 69 | 34.5% |
| Showcase | 66 | 33.0% |
| Help/Question | 65 | 32.5% |
| **Total** | **200** | |

- **Synthetic posts:** 10 of the 200 are AI-generated boundary posts created during label stress-testing (tagged `synthetic stress-test` in `notes`). They were labeled by me and kept as documented edge cases — see the AI Usage section for disclosure.
- **Split:** the notebook splits automatically into 70% train / 15% validation / 15% test. The held-out **test set is 30 posts** (Discussion 11, Help 9, Showcase 10).

### Labeling process

I labeled every post myself by reading the title + body and asking **what response the author most wants** (an answer to act on → Help; the community's views for their own sake → Discussion; eyes on what they made/did → Showcase). For genuinely ambiguous posts I applied a fixed **primary-intent tie-breaker** (full version in [planning.md](planning.md)): surface features (question marks, attached images) do not decide the label; intent does. When a post was still split after the tie-breaker, I labeled by dominant intent and recorded a one-line rationale in the `notes` column so my decisions stay consistent and reviewable.

### 3 difficult-to-label examples and my decisions

1. **"How do you guys keep playing after years? … do you do anything to keep it fresh like install mods…? Do you start new farms a lot?"**
   → Labeled **Help/Question** (confused with Discussion). It reads like an open conversation-starter, but the author is asking for concrete strategies to apply to *their own* lapsed motivation — there's a pending personal action, so the tie-breaker puts it in Help.
2. **"Question for all - Stardew Hot-Takes? … why not ask everyone their controversial Stardew opinions?"**
   → Labeled **Discussion/Opinion** (confused with Help). It's literally a question, but the author wants the community's opinions *for their own sake* with no problem of their own to solve — surface form (a question) does not override intent.
3. **"The most disgusting achievement I have ever achieved"** (image) — "I need to take a hot shower… This feels too gross lol."
   → Labeled **Showcase** (confused with Discussion). The casual "lol" phrasing invites chatter, but the post wouldn't exist without the specific achievement + screenshot being displayed, so by tie-breaker rule 3 it's a Showcase.

---

## 4. Methodology

**Baseline (zero-shot LLM):** Groq `llama-3.3-70b-versatile`, temperature 0, `max_tokens=20`. The system prompt gives the three label definitions + one example per label and instructs the model to output only the label name:

```text
You are classifying posts from the Stardew Valley reddit community.
Assign each post to exactly one of the following categories.

Discussion/Opinion: A post whose main goal is to spark a community conversation
by sharing an opinion, hot take, experience, or observation and inviting others'
views for their own sake, rather than seeking help with a problem or decision.
Example: "What to tell your past self? Has anyone ever used signs to remember
your first time playing Stardew…?"

Help/Question: A post whose main goal is to get assistance with the author's own
situation — a gameplay problem, bug, strategy, or pending decision — so they can
act on the replies, even when the question is subjective.
Example: "Woke up with Bream in my inventory?? … Please advise."

Showcase: A post whose main goal is to display something the author made, did,
achieved, or captured for others to see and react to, rather than ask for help
or debate a topic.
Example: "my 100% completed farm! decorating was super hard… thank you pinterest!"

Respond with ONLY the label name. Do not explain your reasoning.
Valid labels: Discussion/Opinion / Help/Question / Showcase
```

**How results were collected:** each test post is sent as a user message (`"Classify this post:\n\n{text}"`), looped over the 30-post test set with a 0.1s delay between calls (free-tier rate limits). The model's reply is lower-cased and matched against the valid labels (longest-first, case-insensitive) so a clean label string maps to a prediction; anything unmatched counts as unparseable. All 30 responses parsed successfully. *(An early run returned 30/30 unparseable due to a case-mismatch bug in the matcher — the response was lower-cased but the labels weren't — which was fixed before collecting these results.)*

**Fine-tuned model:** `distilbert-base-uncased` with a 3-class classification head, fine-tuned using the HuggingFace `transformers` `Trainer` API on **Google Colab (T4 GPU)**. Both models are evaluated on the same 30-post test set.

**Training setup:** 10 epochs, learning rate 3e-5, batch size 16, weight decay 0.01, `warmup_ratio=0.1`, evaluating each epoch and keeping the best checkpoint by validation accuracy (`load_best_model_at_end=True`).

### Hyperparameter changes from the provided skeleton

The notebook skeleton's defaults caused the model to fail to learn (training loss stuck at `ln(3) ≈ 1.10`, i.e. random). The fix and reasoning:

| Setting | Skeleton | Changed to | Why |
|---|---|---|---|
| `warmup_steps` / `warmup_ratio` | `warmup_steps=50` | `warmup_ratio=0.1` | The full run is only **27 optimizer steps** (140 train ÷ batch 16 × 3 epochs). A 50-step warmup meant the learning rate *never finished ramping up* — it trained at near-zero LR the whole time. Using a ratio scales warmup to the actual run length. **This was the core bug.** |
| `num_train_epochs` | 3 | 10 | 27 steps is far too few for a tiny dataset; more passes are needed to converge. |
| `learning_rate` | 2e-5 | 3e-5 | Small bump to help the model move on limited data. |

`load_best_model_at_end=True` with `metric_for_best_model="accuracy"` keeps the best checkpoint, which mitigates the overfitting visible in the later epochs (training loss → ~0.02 while validation loss climbs).

---

## 5. Evaluation Report

### 5.1 Overall accuracy

| Model | Accuracy | Macro-F1 |
|---|---|---|
| Baseline (Llama-3.3-70B zero-shot) | **0.800** | 0.80 |
| Fine-tuned (DistilBERT) | **0.733** | 0.73 |
| **Δ (fine-tuned − baseline)** | **−0.067** | −0.07 |

**The fine-tuned model did not beat the zero-shot baseline.** This is the headline finding (discussed in §6).

### 5.2 Per-class metrics

**Baseline (zero-shot Llama-3.3-70B):**

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| Discussion/Opinion | 0.73 | 0.73 | 0.73 | 11 |
| Help/Question | 0.88 | 0.78 | 0.82 | 9 |
| Showcase | 0.82 | 0.90 | 0.86 | 10 |
| **Macro avg** | 0.81 | 0.80 | 0.80 | 30 |

**Fine-tuned (DistilBERT):**

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| Discussion/Opinion | 0.82 | 0.82 | 0.82 | 11 |
| Help/Question | 0.60 | 0.67 | 0.63 | 9 |
| Showcase | 0.78 | 0.70 | 0.74 | 10 |
| **Macro avg** | 0.73 | 0.73 | 0.73 | 30 |

The two models have **different weak spots**: the baseline's weakest class was Discussion/Opinion (F1 0.73); the fine-tuned model's weakest is Help/Question (F1 0.63). Fine-tuning didn't just lower accuracy — it **changed the error structure**.

### 5.3 Confusion matrix — fine-tuned model

Rows = **true** label, columns = **predicted** label. (Also saved as [confusion_matrix.png](confusion_matrix.png).)

| True ↓ / Pred → | Discussion/Opinion | Help/Question | Showcase | Total |
|---|---|---|---|---|
| **Discussion/Opinion** | **9** | 1 | 1 | 11 |
| **Help/Question** | 2 | **6** | 1 | 9 |
| **Showcase** | 0 | 3 | **7** | 10 |
| **Total (predicted)** | 11 | 10 | 9 | 30 |

**Reading the matrix — the dominant error directions:**
- **Showcase → Help/Question (3 errors):** the single largest off-diagonal cell. Every one of Showcase's 3 misses was sent to Help/Question.
- **Help/Question → Discussion/Opinion (2 errors):** the canonical decision-question boundary.
- Help/Question is **over-predicted** (10 predicted, only 6 correct → precision 0.60): it acts as a *sink* that absorbs emotional/feedback-seeking Showcase posts and is itself confused for Discussion. There are **zero** Showcase↔Discussion confusions in either direction, and only 1 Help↔Showcase — so errors concentrate on the **Help/Question boundary**, not the Showcase↔Discussion boundary I originally predicted.

### 5.4 AI-assisted pattern surfacing (and what I corrected)

Per the project instructions, I pasted all 8 misclassified examples into an LLM and asked it to surface common themes, then re-read the examples myself to verify.

**Patterns the AI proposed → my verification:**
- ⚠️ **All errors are low-confidence — but so are the correct predictions.** Every wrong prediction scored **0.36–0.49**, which initially looked like good calibration (mistakes = uncertain calls). On checking the *correct* predictions too, they sit at **0.39–0.54** — the same range. So low confidence is **not** a useful error signal here; the model is uniformly under-confident rather than well-calibrated (see §5.6).
- ✅ **Help/Question is an over-prediction sink for first-person / feedback-seeking posts.** Showcase posts with emotional narration ("I made mayonnaise, and now I'm crying") or an explicit feedback ask ("looking for thoughts/feedback") get pulled into Help. Verified — 3 of 3 Showcase errors fit this.
- ✅ **Decision questions ("who's your favourite…", "idk who to choose…") read as Discussion.** Verified — matches the Help↔Discussion boundary from planning.md.
- ❌ **Discarded: "short / low-information posts."** The AI initially suggested short posts were a driver. On re-reading, the misclassified posts are mostly *long* narratives — length is not the pattern, so I dropped this claim.

### 5.5 Specific wrong predictions — analysis

**Example A — Help misread as Discussion (the decision-question boundary)**
> *"Who's you guys' favourite bachelor/bachelorette? i have married harvey in almost all of my playthroughs … and i was wondering…"*
> True: **Help/Question** · Predicted: **Discussion/Opinion** (conf 0.39)

- **Which labels / which direction:** Help → Discussion, one of the 2 errors in that cell.
- **Why it's hard:** the surface form ("who's your favourite?") is identical to an opinion-gathering prompt. I labeled it Help because the author is collecting input to *make their own romance decision* — an intent signal, not a surface one.
- **Labeling vs. data problem:** **data problem, not annotation inconsistency.** I labeled all "which should I pick" decision posts as Help consistently. The model simply hasn't seen enough of them to learn that "favourite/which" + a personal pending choice = Help rather than Discussion.
- **Fix:** more training examples of subjective decision-questions labeled Help, so the boundary is represented rather than inferred from a keyword.

**Example B — Showcase misread as Help (the over-prediction sink)**
> *"Sam/Seb Portraits mod progress - looking for thoughts/feedback … I did some edits and now I'm posting second drafts…"*
> True: **Showcase** · Predicted: **Help/Question** (conf 0.37)

- **Which labels / which direction:** Showcase → Help, part of the largest error cell.
- **Why it's hard:** the phrase "looking for thoughts/feedback" is a strong lexical cue for Help. But the post's primary intent is to *display a finished creation* (a mod the author made) — feedback is secondary. The model keyed on the surface phrase, exactly the mistake the annotation guide warns against.
- **Labeling vs. data problem:** **data/boundary problem.** Showcase posts that invite feedback are underrepresented, so the model defaults to the "feedback → Help" shortcut.
- **Fix:** add Showcase examples that contain feedback requests ("rate my farm", "thoughts?"), so the model learns that displaying a creation outranks a feedback hook.

**Example C — Showcase misread as Help (emotional narrative)**
> *"I made mayonnaise, and now I'm crying. Guys. It happened. I've been playing since the beginning and I'm so happy right now with my farm layout…"*
> True: **Showcase** · Predicted: **Help/Question** (conf 0.38)

- **Which labels / which direction:** Showcase → Help.
- **Why it's hard:** there's no explicit "look at this" framing and no image text — just an emotional first-person story. With nothing lexically "showcase-y," the model fell back to its Help default. A human reads the celebratory tone as showing off a moment; the model couldn't.
- **Labeling vs. data problem:** **data problem.** Narrative-style Showcase posts (a story rather than "here's my farm [image]") are a minority of the Showcase examples, so the model didn't learn them.
- **Fix:** more diverse Showcase examples — especially narrative/emotional ones without explicit display language.

**Common thread:** all three failures are the model relying on **surface lexical cues** ("favourite", "feedback", emotional first-person) instead of **primary intent**, with Help/Question as the fallback bucket. That is precisely the failure mode the label definitions were written to avoid — but ~140 training examples weren't enough to teach the intent-over-form distinction.

### 5.6 Sample Classifications

Five posts run through the fine-tuned model, with predicted label and softmax confidence (3 wrong, 2 correct):

| Post (truncated) | True | Predicted | Confidence | Correct? |
|---|---|---|---|---|
| "Who's you guys' favourite bachelor/bachelorette? …" | Help/Question | Discussion/Opinion | 0.39 | ✗ |
| "Sam/Seb Portraits mod progress - looking for thoughts/feedback…" | Showcase | Help/Question | 0.37 | ✗ |
| "I love you, Iridium Scythe. Harvesting \$300,000 worth of crops…" | Discussion/Opinion | Showcase | 0.49 | ✗ |
| "I finally completed all the achievements for my favorite game!" | Showcase | Showcase | 0.54 | ✓ |
| "Tell me your hottest Stardew Valley take. I don't mean 'Abigail is the Wizard's daughter'…" | Discussion/Opinion | Discussion/Opinion | 0.44 | ✓ |

**Why a correct prediction is reasonable:** *"I finally completed all the achievements for my favorite game!"* is correctly classified **Showcase** (conf 0.54, the model's most confident call in this sample) — it announces a completed personal accomplishment for others to see, with no question asked and no topic to debate, so surface form and intent both point to Showcase and the model has many similar training examples.

**Calibration takeaway (corrected against the data):** the model is **uniformly under-confident, not well-calibrated.** Its correct predictions (0.39–0.54) sit in essentially the *same* range as its wrong ones (0.36–0.49), and **no prediction in the test set reaches even 0.55**. So confidence here does **not** separate right from wrong and cannot be used as a reliability signal — this *fails* the calibration goal in planning.md (which wanted confident predictions to be trustworthy). This is expected for a small model fine-tuned on ~140 examples: it never becomes sure of anything, so its softmax probabilities are compressed toward chance (≈0.33).

---

## 6. Reflection — what the model captured vs. what I intended

**What I intended:** a classifier that decides by the author's *primary intent*, explicitly ignoring surface features (question marks, images, keywords).

**What the model actually captured:** a decision boundary built largely on **surface lexical cues**, with **Help/Question as a default bucket**. Concretely:
- It learned the **easy, form-aligned cases well** — Discussion/Opinion reached F1 0.82, because most Discussion posts both *are* opinion-sharing and *look* like it.
- It **overfit to lexical shortcuts**: "feedback/thoughts/looking for" → Help; "favourite/which should I" → Discussion; explicit display language → Showcase. When a post's *form* contradicted its *intent* (a finished mod that asks for feedback; a celebratory story with no "look at this"), the model followed the form and lost.
- It **missed intent-over-form** — exactly the distinction the label definitions were designed around. The model could not tell that a feedback request on a finished creation is still Showcase, or that a "favourite?" poll aimed at a personal decision is Help.

**What it overfit to:** keyword/phrase patterns and first-person emotional tone (routing both toward Help). **What it missed:** the abstract notion of *what response the author wants* — the very thing that separates these labels.

This gap is the core lesson: with only ~140 training examples, the model learned the **surface correlates** of the labels rather than their **defining intent**, which is why a 70B model that already "understands" intent zero-shot still beat it. The fix is not a better label definition (the definitions are consistent and were validated by stress-testing) but **more — and more form-contradicting — training data** for the Help/Question boundary.

---

## 7. Spec Reflection

**One way the spec guided implementation:** the spec's required structure — *define labels and success criteria before modeling*, then baseline → fine-tune → evaluate — directly shaped the work. Because [planning.md](planning.md) committed to specific, checkable success criteria (e.g. "Help/Question recall ≥ 0.70", "errors should fall on the known boundary pairs") *before* I saw results, the evaluation became a matter of ticking boxes rather than rationalizing whatever numbers came out. The pre-registered "hard edge cases" also gave me a hypothesis to test against the confusion matrix instead of just reporting accuracy.

**One way the implementation diverged from the spec:** the spec's provided training hyperparameters (`warmup_steps=50`, `num_train_epochs=3`) did not work for a dataset this small — with only 27 optimizer steps, a 50-step warmup left the model training at near-zero learning rate, and it never learned (accuracy ~0.40, loss stuck at random). I diverged by replacing `warmup_steps=50` with `warmup_ratio=0.1` and raising epochs to 10. The reason is dataset-size-dependence: a fixed step count that's reasonable for a large dataset is longer than the *entire run* on 140 examples, so warmup had to be expressed as a fraction of the run rather than an absolute count.

---

## 8. AI Usage

I used AI assistance (Claude) at several points. The model was treated as a fallible assistant whose output I verified and frequently overrode.

**Instance 1 — Label stress-testing (definition refinement).** I directed Claude to generate 10 posts deliberately sitting on the Help↔Discussion and Showcase↔Discussion boundaries, then I classified each myself. It produced realistic boundary posts; classifying them exposed a flaw in my original definitions (I had written Help as requiring "a single correct answer," but I labeled subjective decision-questions as Help). **What I changed:** I rewrote the Help/Question definition and tie-breaker rule 4 to key on "the author has a pending decision and wants replies they'll act on" rather than "a correct answer exists." I kept my own labels on the synthetic posts and tagged them `synthetic` in the data.

**Instance 2 — Debugging the training collapse.** I gave Claude my fine-tuning code after the model dropped to ~0.40 accuracy and asked what was wrong. It identified that `warmup_steps=50` exceeded the total of 27 optimizer steps, so the learning rate never ramped up (confirmed by the loss sitting at `ln(3)`). **What I did:** applied the recommended `warmup_ratio=0.1` + more epochs, which fixed training (accuracy rose to 0.73). I verified the diagnosis against the loss curve rather than taking it on faith. (Claude also caught an earlier baseline bug — a case-mismatch in the label-matching code that made all 30 responses unparseable.)

**Instance 3 — Failure-pattern surfacing.** Per the assignment, I pasted the 8 misclassified examples into an LLM to surface themes, then verified each against the examples (see §5.4). **What I overrode:** I discarded its "short/low-information posts" hypothesis after re-reading showed the errors were mostly long posts, and kept only the patterns I could confirm in the data.

**Annotation disclosure.** The ~190 real posts were **collected and labeled manually by me** — I did *not* use an LLM to pre-label them. AI was used only to (a) generate the 10 synthetic stress-test posts (which I then labeled myself) and (b) help format/validate the CSV (fixing quote-escaping and counting labels). No label in the dataset was assigned by an AI without my review.

---

## 9. Repository structure

| File | Purpose |
|---|---|
| [planning.md](planning.md) | Pre-modeling design: community, label definitions, edge cases, data plan, evaluation metrics, success criteria, AI tool plan, baseline reflection. |
| [data.csv](data.csv) | 200 labeled posts (`text`, `label`, `notes`). |
| [evaluation_results.json](evaluation_results.json) | Baseline vs. fine-tuned accuracy, label map, model name. |
| [confusion_matrix.png](confusion_matrix.png) | Fine-tuned model confusion matrix (also written as a table in §5.3). |
| README.md | This evaluation report. |
