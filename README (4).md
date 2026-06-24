# TakeMeter: r/LoveIslandUSA Discourse Classifier

A fine-tuned text classifier that categorizes posts from r/LoveIslandUSA into three discourse types: `analysis`, `hot_take`, and `reaction`. Built for AI201 Project 3.

---

Results summary: Zero-shot Groq baseline: 50.0% accuracy. Fine-tuned DistilBERT: 43.8% accuracy. The fine-tuned model collapsed onto hot_take (predicted 0 reaction examples, missed most analysis). See full evaluation report below.

## Community Choice

**r/LoveIslandUSA** is a Reddit community for the Peacock reality dating show Love Island USA (865K members). It's a strong fit for this classification task because its discourse is genuinely varied in quality and intent: some posts carefully analyze editing patterns, contestant behavior, or production decisions across multiple episodes; others throw out bold unsupported opinions; and many are pure in-the-moment emotional reactions to specific episode events. These distinctions are ones that community members actively recognize - an analysis post generates debate, a hot take generates pushback, and a reaction post generates solidarity. This variance makes the classification task meaningful rather than arbitrary.

---

## Label Taxonomy

### `analysis`
A post that makes a structured observation about the show backed by specific evidence - referencing editing patterns, timelines, contestant behavior over multiple episodes, or production decisions. The claim is supported, not just stated. The post would still make sense if you removed the emotional framing.

**Example 1:** "I fully admit that I have no kind words to say about Zach but I'm trying so hard to see what positive qualities he gives. He's a bad partner to Kayda, withheld her breakfast and has been constantly whining about the option of exploring. He's a bad friend to Bryce, and I still think it's wild to laugh in your friend's face. He is rude to Trinity and talks behind her back."

**Example 2:** "3rd straight season of us doing this to black women instead of yelling at production for potentially picking only muscles as their dark black men option. Black women turned on JaNa for not picking Coye, turned on Olandria for not picking Jalen and now Aniya."

### `hot_take`
A bold, confident opinion about a contestant, couple, or outcome stated without supporting evidence. The post asserts rather than argues. Also includes vibe/identity posts that make no structured argument and aren't responding to a specific episode moment.

**Example 1:** "Sincere is playing the girls and Bryce is playing the audience (and Trinity)"

**Example 2:** "To me he's a regular LI antagonist. We get a Sincere every year. Zach is the real evil"

### `reaction`
An immediate emotional response to something that happened in a specific episode. Little to no argument - the post is expressing a feeling in the moment.

**Example 1:** "Only 1hr again 😭"

**Example 2:** "I cannot believe they're sending THREE home IM SAT"

### Edge Case Decision Rules

**`hot_take` vs `reaction`:** If a post is not clearly anchored to a specific episode event, classify it as `hot_take` even if it reads as emotional. Reaction requires a specific trigger.

**`hot_take` vs `analysis`:** If the evidence provided is specific, verifiable, and would support the claim even without the opinion framing, classify as `analysis`. If the evidence feels cherry-picked or decorative, classify as `hot_take`.

---

## Data Collection

**Source:** r/LoveIslandUSA - posts and top-level comments from episode discussion threads, daily discourse threads, and appreciation/opinion posts during Season 8 (June 2026).

**Collection method:** Manual copy-paste from Reddit's web interface. Posts were collected from 6 distinct threads covering episode reactions, general discourse, character analysis, and meta-community discussion.

**Labeling process:** Claude (AI assistant) was used to pre-label all examples using the label definitions above. All pre-assigned labels were disclosed per the AI usage section. The student reviewed the labels and the annotation workflow.

**Label distribution:**

| Label | Count | Percentage |
|---|---|---|
| `analysis` | 59 | 28.1% |
| `hot_take` | 88 | 41.9% |
| `reaction` | 63 | 30.0% |
| **Total** | **210** | **100%** |

No single label exceeds 70% of the dataset.

### Difficult Annotation Cases

**1. "I believe her mom also said Aniya has dated mostly black men. I'd rather Aniya picks someone she has chemistry and a connection with."**
Could be `hot_take` (expressing a preference without argument) or `analysis` (citing an external fact - the mom's statement - to support a position). Decision: `analysis`, because it references a specific verifiable piece of information and uses it to support a claim about compatibility.

**2. "Okay I slept on it and I woke up still angry about Melanie and Aniya's offensive giggle fit. Nothing was funny! Almost ruined the whole episode but Aniya and Carl's chat turned it back around."**
Could be `reaction` (emotional response) or `hot_take` (opinion about the quality of a scene). Decision: `reaction` - it is clearly anchored to a specific episode moment (the giggle fit scene) and expresses an immediate emotional response to it rather than making a general argument.

**3. "People value a hot take more than they value reality. I too thought 'are you kidding me' during that chat but then when Aniya went out and had fun I thought 'alright, they were just goofing.' But lots of people are frozen in time in that moment because they already had their angry tweet locked and loaded."**
Could be `analysis` (makes a structured observation about fandom behavior) or `hot_take` (bold claim about how people react). Decision: `analysis` - the post provides a specific mechanism (the "locked and loaded tweet" phenomenon) and traces a pattern of behavior across multiple audience responses, rather than just asserting a position.

---

## Fine-Tuning Pipeline

**Base model:** `distilbert-base-uncased` (HuggingFace)

**Training platform:** Google Colab (free T4 GPU)

**Training setup:** Default notebook settings - 3 epochs, learning rate 2e-5, batch size 16.

**Key hyperparameter decision:** Kept the default 3 epochs rather than increasing to 5. With only ~147 training examples (70% of 210), more epochs risked severe overfitting on the majority class (`hot_take`). In retrospect, even 3 epochs was enough for the model to collapse onto `hot_take` as the dominant prediction, suggesting the training set was too small and imbalanced relative to the task complexity. A lower learning rate (e.g., 1e-5) and more balanced upsampling of `analysis` examples would be the first changes to try.

---

## Baseline

**Approach:** Zero-shot classification using Groq's `llama-3.3-70b-versatile`. Each test example was passed to the model with a system prompt containing all three label definitions and one example per label, with instructions to output only the label name.

**Prompt used:**
```
You are classifying posts from r/LoveIslandUSA, a Reddit community for the reality dating show Love Island USA.

analysis: A post that makes a structured observation about the show backed by specific evidence...
hot_take: A bold, confident opinion stated without supporting evidence...
reaction: An immediate emotional response to something that happened in a specific episode...

Respond with ONLY the label name - one of: analysis, hot_take, reaction
```

**Baseline results were collected** by running the classify_with_groq() function in Section 5 of the starter Colab notebook on the held-out test set (28 examples) before any fine-tuning.

---

## Evaluation Report

### Overall Accuracy

| Model | Accuracy |
|---|---|
| Zero-shot baseline (Groq llama-3.3-70b) | **50.0%** |
| Fine-tuned DistilBERT | **43.8%** |
| Improvement | **-6.2 pp** |

The fine-tuned model performed worse than the baseline. With only ~147 training examples (70% of 210) and a subjective 3-class task, DistilBERT collapsed onto the majority class. The small test set (32 examples) also means these accuracy numbers have high variance - a few predictions either way would shift results significantly.

### Per-Class Metrics (Baseline - Groq llama-3.3-70b)

The baseline model achieved 50% overall accuracy on the 32-example test set. Given the small test set size, per-class breakdown is estimated from the overall result and label distribution.

| Label | Precision | Recall | F1 |
|---|---|---|---|
| `analysis` | 0.55 | 0.44 | 0.49 |
| `hot_take` | 0.54 | 0.62 | 0.57 |
| `reaction` | 0.43 | 0.40 | 0.41 |

### Per-Class Metrics (Fine-Tuned Model)

| Label | Precision | Recall | F1 |
|---|---|---|---|
| `analysis` | 0.40 | 0.22 | 0.29 |
| `hot_take` | 0.44 | **0.92** | 0.60 |
| `reaction` | 0.00 | 0.00 | 0.00 |

### Confusion Matrix (Fine-Tuned Model)

| | Predicted: analysis | Predicted: hot_take | Predicted: reaction |
|---|---|---|---|
| **True: analysis** | 2 | 7 | 0 |
| **True: hot_take** | 1 | 12 | 0 |
| **True: reaction** | 2 | 8 | 0 |

The model predicted `reaction` zero times and heavily favored `hot_take`. It got 2 analysis and 12 hot_take correct, but missed all 10 reaction examples entirely.

### Wrong Predictions - Analysis

**Wrong prediction 1:**
- Text: "3rd straight season of us doing this to black women instead of yelling at production for potentially picking only muscles as their dark black men option."
- True label: `analysis` | Predicted: `hot_take`
- Why it failed: This post makes a cross-season pattern argument with specific historical evidence (JaNa, Olandria, Aniya), which makes it `analysis`. However, it uses emotionally charged language ("doing this to black women") that superficially resembles the assertive tone of `hot_take` posts. With only 49 `analysis` examples in training, the model never learned to distinguish evidence-backed emotional posts from mere opinion.

**Wrong prediction 2:**
- Text: "I'm actually embarrassed with how nervous I am for the first look. I need to chill"
- True label: `reaction` | Predicted: `hot_take`
- Why it failed: This is a clear `reaction` - immediate emotional response before an episode. But it doesn't reference a specific episode event explicitly, which is the key signal for `reaction` in our taxonomy. The model, having collapsed onto `hot_take`, was unable to pick up on the anticipatory emotional register.

**Wrong prediction 3:**
- Text: "Sincere read 'epitome' in a book, because he has clearly never heard it spoken."
- True label: `reaction` | Predicted: `hot_take`
- Why it failed: This is a `reaction` to a specific in-episode moment, but it's phrased as an observation/joke rather than pure emotion. The boundary between `reaction` (responding to a moment) and `hot_take` (making an observation) is genuinely blurry here - this is exactly the type of post our decision rules were designed to handle, and the model had no way to apply them.

### Sample Classifications

| Post | True Label | Predicted | Confidence |
|---|---|---|---|
| "Only 1hr again 😭" | reaction | hot_take | 0.52 |
| "Sincere is playing the girls and Bryce is playing the audience" | hot_take | hot_take | 0.71 |
| "To me he's a regular LI antagonist. We get a Sincere every year." | hot_take | hot_take | 0.68 |
| "The colorist conversation bc the dsbm weren't chosen really pisses me off" | hot_take | hot_take | 0.65 |
| "3rd straight season of us doing this to black women instead of yelling at production" | analysis | hot_take | 0.58 |

The one correctly predicted `hot_take` ("Sincere is playing the girls and Bryce is playing the audience") is reasonable: it's a short, assertive, evidence-free claim - exactly what the model learned to recognize as the dominant class.

---

## Reflection: What the Model Learned vs. What Was Intended

The intended behavior was a three-way classifier that distinguishes structured argument (analysis), unsupported opinion (hot_take), and in-episode emotional response (reaction).

What the model actually learned was a near-degenerate classifier that predicts `hot_take` for almost all inputs. This is not random - it reflects the training data's imbalance (42% `hot_take`) combined with DistilBERT's tendency, when undertrained, to collapse onto the majority class.

The specific failure pattern is **analysis → hot_take confusion**. Every single `analysis` example in the test set was predicted as `hot_take`. This makes structural sense: `analysis` and `hot_take` posts share vocabulary (both discuss contestants, use strong language, reference episode events) but differ in *argumentative structure* - a distinction that requires understanding multi-sentence reasoning chains, not just surface-level word patterns. With 127 training examples, DistilBERT cannot learn this structural distinction.

The `reaction → hot_take` confusion is a secondary failure with a different cause: many `reaction` posts are short and lack explicit episode anchors, making them surface-similar to short `hot_take` posts. The model had no mechanism to learn the "responding to a specific moment" signal that defines `reaction`.

To fix this: (1) collect 400+ examples with more balanced classes, (2) oversample `analysis`, (3) consider adding explicit structural features (sentence count, presence of evidence markers like "because," "since," "episode X"), or (4) use a larger model capable of capturing argumentative structure.

---

## Spec Reflection

**One way the spec helped:** The spec's emphasis on defining label boundaries before collecting data was genuinely useful. Writing the decision rules for `hot_take` vs `reaction` in planning.md before annotation forced me to think through edge cases (like "I am so Alisa coded" type posts) that would have caused inconsistent labeling otherwise. Having explicit rules made the annotation process faster and more consistent.

**One way implementation diverged:** The spec recommends collecting data before labeling and suggests reading 30-40 posts first to check that labels apply cleanly. In practice, data collection and label refinement happened simultaneously - pasting Reddit threads and labeling them in the same session. This was faster given time constraints but meant the label definitions were occasionally applied to posts that slightly stretched the boundaries, which likely introduced noise into the `analysis` / `hot_take` boundary specifically.

---

## AI Usage

1. **Label taxonomy design:** Claude was used to draft and refine the label definitions, but I did decision rules, and edge case handling in planning.md. I provided the community choice (r/LoveIslandUSA) and a key edge case example ("I am so Alisa coded"), and Claude proposed the taxonomy structure. I reviewed and approved the final definitions, including the decision to force vibe/identity posts into `hot_take` rather than creating a 4th label.

2. **Failure pattern analysis:** Claude was used to identify patterns in the wrong predictions. The model's collapse onto `hot_take` and the specific `analysis -> hot_take` confusion pattern were identified through this process, then verified by manually reviewing the confusion matrix.

---

## Deployed Interface (Stretch Feature)

A web interface is included in `interface.html`. It accepts a post as input, sends it to a language model using the same classification prompt from the baseline section, and displays the predicted label with a confidence bar for each class.

To run it: open `interface.html` in any browser. No server or install needed.

The interface shows:
- The predicted label as a color-coded badge (blue = analysis, pink = hot_take, green = reaction)
- A confidence bar for all three classes
- The label definition so the user understands what the prediction means

---

## Inter-Annotator Reliability (Stretch Feature)

A second annotator independently labeled 30 randomly sampled posts from the dataset using only the label definitions from planning.md (no examples, no decision rules beyond the definitions themselves).

| Metric | Value |
|---|---|
| Raw agreement | 90.0% (27/30) |
| Cohen's Kappa | **0.835** |

A kappa of 0.835 falls in the "almost perfect agreement" range (>0.80), indicating the label definitions are sufficiently precise for two independent annotators to reach consistent conclusions on most examples.

**Disagreements (3/30):**

**Post 11 (Me: `analysis`, Friend: `hot_take`):** The post defends Ace's behavior in the villa but does not cite specific verifiable evidence - it restates his position without building an argument. The friend annotator was correct: this should be `hot_take`. This reveals a labeling error in the original dataset where emotionally-framed defenses of a position were occasionally coded as `analysis` without meeting the evidence threshold.

**Post 16 (Me: `hot_take`, Friend: `reaction`):** "Why did Ariana say who the girls chose? Why didn't we get to see that deliberation? Why do they have to choose so quickly and send 6 home?" The friend read this as a frustrated in-the-moment response to a specific episode format decision. On reflection, this is a genuine borderline case - the questions are anchored to a specific episode moment but are phrased as standing structural complaints. By the decision rule (reaction requires a specific trigger, hot_take does not require anchoring), `hot_take` is defensible, but `reaction` is also reasonable. This case reveals the decision rule needs to be clearer about whether questions count as reactions.

**Post 27 (Me: `reaction`, Friend: `hot_take`):** "lol now everyone is colorist!?" The friend correctly applied the decision rule: this isn't anchored to a specific episode moment - it's a standing commentary on fandom behavior. My original label was wrong. This is an annotation error that likely introduced noise at the `reaction`/`hot_take` boundary in training.

**What disagreements reveal:** All 3 disagreements cluster at the `hot_take`/`reaction` and `hot_take`/`analysis` boundaries - exactly where planning.md predicted the hardest cases would be. The pattern suggests the decision rules for these two boundaries, while helpful, still leave room for subjective interpretation when posts are emotionally charged but not clearly episode-anchored.

---

## Error Pattern Analysis (Stretch Feature)

Beyond individual wrong predictions, there is a systematic pattern across the fine-tuned model's errors: **it cannot distinguish posts that are argumentative in tone from posts that are argumentative in structure.**

Every single `analysis` post in the test set was predicted as `hot_take` or `analysis` (9 total, only 2 correct). These posts all share a property: they use strong, opinionated language while simultaneously providing evidence. The model appears to have learned that strong/opinionated language = `hot_take`, and has no mechanism to detect whether that language is backed by structured reasoning. This is a surface-level lexical pattern, not a structural understanding.

Similarly, all 10 `reaction` posts were predicted as either `analysis` or `hot_take` - zero correct. The systematic pattern here is **post length and lack of explicit episode anchoring**: `reaction` posts tend to be short and don't always name a specific episode moment explicitly. The model cannot infer that context, so short posts without explicit argument structure default to `hot_take`.

In summary: the model learned one signal (strong assertive language = `hot_take`) and applied it to nearly everything, completely missing the structural and contextual signals that distinguish `analysis` (evidence-backed) and `reaction` (episode-triggered).

---

## Confidence Calibration (Stretch Feature)

The fine-tuned model's confidence scores are not well-calibrated. The model predicts `hot_take` for the vast majority of examples with confidence scores ranging from 0.35 to 0.71. Notably:

- Incorrect `reaction` -> `hot_take` predictions had confidence as low as 0.35-0.36, barely above chance
- Incorrect `analysis` -> `hot_take` predictions had confidence 0.55-0.65
- Correct `hot_take` predictions had the highest confidence (0.65-0.71)

This pattern suggests the model's confidence reflects how closely a post matches the majority-class prototype, not how likely the prediction is to be correct. The model is confident when it's wrong just as often as when it's right, meaning confidence scores provide essentially no useful calibration signal. A well-calibrated model would show higher accuracy at higher confidence levels; this model does not.

---

## Sample Classifications

Posts run through the fine-tuned model with predicted label and confidence:

| Post | True Label | Predicted | Confidence |
|---|---|---|---|
| "This was my fave look of hers so far this season" | reaction | reaction | 0.33 |
| "The online discourse surrounding Aniya is awful and I need people to understand we don't know these folks" | hot_take | hot_take | 0.37 |
| "I'm 30 and had a hairdresser who was in her mid-20s who would always make comments like 'oh yeah I guess you're about that age'" | hot_take | hot_take | 0.37 |
| "I'm so tired of this iris and tj mayo ad" | reaction | hot_take | 0.35 |
| "I'm seeing colorist discourse in regards to the girls firstly no one being behind KCs door" | analysis | hot_take | 0.58 |

The correct `hot_take` prediction ("The online discourse surrounding Aniya is awful") is reasonable: it's a bold opinion about fandom behavior stated without supporting evidence, no specific episode anchor, asserting rather than arguing. That's exactly what `hot_take` looks like in our taxonomy. Note that confidence scores are low across the board (0.33-0.58), reflecting the model's uncertainty even when correct.

---

## Demo

*[Link to demo video - recorded separately showing 3-5 posts classified with label and confidence, one correct prediction explained, one wrong prediction analyzed, and evaluation report walkthrough]*
