# TakeMeter - Planning Document
**Project 3 | AI201 | Sidhya Ganesh**

---

## Community

**Chosen community:** r/LoveIslandUSA (Reddit)

Love Island USA is a reality dating competition show with an extremely active Reddit fandom. The community is a strong fit for a classification task because its discourse is genuinely varied in type and quality: some posts carefully analyze editing patterns, casting decisions, or contestant behavior over time; others throw out bold unsupported opinions about who deserves to win or who is "the villain"; and many are pure in-the-moment emotional reactions to specific episode events. These distinctions are ones that regular community members actively recognize and respond to differently - an analysis post generates discussion, a reaction post generates solidarity, and a hot take generates debate. This variance makes the classification task meaningful rather than arbitrary.

**Data source:** Public posts and comments from r/LoveIslandUSA, collected manually via Reddit's web interface during an active season.

---

## Label Taxonomy

### Labels

**`analysis`**
A post that makes a structured observation about the show backed by specific evidence - referencing editing patterns, timelines, contestant behavior over multiple episodes, or production decisions. The claim is supported, not just stated. The post would still make sense if you removed the emotional framing.

**`hot_take`**
A bold, confident opinion about a contestant, couple, or outcome stated without supporting evidence. The post asserts rather than argues. This label also covers vibe/identity posts (e.g. "I am so [contestant] coded") that make no structured argument and aren't responding to a specific moment.

**`reaction`**
An immediate emotional response to something that happened in a specific episode. Little to no argument - the post is expressing a feeling in the moment. Typically triggered by a scene, elimination, or line that just aired.

### Edge Case Decision Rule

**The hardest boundary is `hot_take` vs `reaction`.**

Some posts express strong emotion but aren't tied to a specific episode moment - they're standing opinions about a contestant rather than a response to something that just happened. Decision rule: **if the post is not clearly anchored to a specific episode event, classify it as `hot_take` even if it reads as emotional.** Reaction requires a specific trigger. A post like "I genuinely cannot stand [contestant]" with no episode reference → `hot_take`. The same sentiment posted immediately after an elimination with explicit reference to it → `reaction`.

**The second hardest boundary is `hot_take` vs `analysis`.**

Some posts cite one statistic or observation to back up an opinion. Decision rule: **if the evidence provided is specific and verifiable and would support the claim even without the opinion framing, classify as `analysis`. If the evidence feels cherry-picked or decorative - just enough to sound credible but not genuinely reasoning - classify as `hot_take`.**

### Example Posts

| Label | Example 1 | Example 2 |
|---|---|---|
| `analysis` | "I fully admit that I have no kind words to say about Zach but I'm trying so hard to see what positive qualities he gives. He's a bad partner to Kayda, withheld her breakfast and has been constantly whining about the option of exploring. He's a bad friend to Bryce, and I still think it's wild to laugh in your friend's face. He is rude to Trinity and talks behind her back." | "3rd straight season of us doing this to black women instead of yelling at production for potentially picking only muscles as their dark black men option. Black women turned on JaNa for not picking Coye, turned on Olandria for not picking Jalen and now Aniya." |
| `hot_take` | "Sincere is playing the girls and Bryce is playing the audience (and Trinity)" | "To me he's a regular LI antagonist. We get a Sincere every year. Zach is the real evil" |
| `reaction` | "Only 1hr again 😭" | "Now, you're sending THREE home IM SAT" |

---

## Hard Edge Cases

1. **"I believe her mom also said Aniya has dated mostly black men. I'd rather Aniya picks someone she has chemistry and a connection with."** - Could be `hot_take` (expressing a preference without argument) or `analysis` (citing an external fact to support a position). Decision: `analysis`, because it references a specific verifiable piece of information (the mom's statement) and uses it to support a claim about compatibility.

2. **"Okay I slept on it and I woke up still angry about Melanie and Aniya's offensive giggle fit. Nothing was funny! Almost ruined the whole episode but Aniya and Carl's chat turned it back around."** - Could be `reaction` (emotional response) or `hot_take` (opinion about the quality of a scene). Decision: `reaction` - it is clearly anchored to a specific episode moment (the giggle fit scene) and expresses an immediate emotional response rather than a general argument.

3. **"People value a hot take more than they value reality. I too thought 'are you kidding me' during that chat but then when Aniya went out and had fun I thought 'alright, they were just goofing.' But lots of people are frozen in time in that moment because they already had their angry tweet locked and loaded."** - Could be `analysis` (structured observation about fandom behavior) or `hot_take` (bold claim about how people react). Decision: `analysis` - the post provides a specific mechanism and traces a pattern of behavior across multiple audience responses, rather than just asserting a position.

---

## Data Collection Plan

**Source:** r/LoveIslandUSA - public posts and top-level comments from episode discussion threads, daily discussion threads, and general posts during an active season.

**Collection method:** Manual copy-paste into a CSV with columns: `text`, `label`, `notes`. The `notes` column will track cases that required non-obvious decisions.

**Target distribution:** At least 67 examples per label (~33% each), aiming for no label exceeding 70% of the dataset. If any label is underrepresented after 150 examples, additional targeted collection will focus on that label type (e.g., searching for post-episode reaction threads for more `reaction` examples, or meta-discussion threads for more `analysis` examples).

**Volume target:** 210+ labeled examples to leave buffer after the 70/15/15 train/val/test split (which gives ~30 test examples - enough for meaningful per-class metrics at 3 classes).

**AI annotation assistance:** Claude will be used to pre-label batches of collected posts using the label definitions above. Every pre-assigned label will be reviewed and corrected by the student before inclusion in the dataset. Pre-labeled examples that are corrected will be noted in the AI Usage section of the README.

---

## Evaluation Metrics

**Primary metric: overall accuracy** - straightforward to interpret and required by the spec.

**Per-class F1** - this is the right additional metric because the task has 3 labels that may not be equally easy to learn. A model that gets 80% overall accuracy by over-predicting `reaction` (the most emotionally obvious class) could still have near-zero F1 on `analysis`. F1 balances precision and recall per class, which surfaces that kind of failure.

**Confusion matrix** - to identify which specific label pairs the model confuses and in which direction. This is more actionable than aggregate metrics alone.

Accuracy alone is insufficient because class imbalance (even mild) can produce misleadingly high accuracy. Per-class F1 and the confusion matrix together show whether the model has actually learned all three boundaries or just the easy ones.

---

## Definition of Success

The fine-tuned classifier will be considered **good enough for deployment in a community moderation or post-tagging tool** if it meets both of the following thresholds on the held-out test set:

1. **Overall accuracy ≥ 75%**
2. **Fine-tuned model beats the zero-shot Groq baseline by at least 10 percentage points**

A model hitting only one of these conditions is not considered successful - beating a weak baseline by 10 points matters less if absolute accuracy is still poor, and high accuracy that barely exceeds a strong baseline suggests fine-tuning added little.

Per-class F1 ≥ 0.60 on all three labels would be a secondary success indicator, showing the model hasn't completely failed to learn any single boundary.

---

## AI Tool Plan

**Intended use: Annotation assistance**

Claude will be used to pre-label batches of collected posts. The workflow:
1. Paste a batch of 20–30 unlabeled posts into Claude along with the label definitions above
2. Ask Claude to assign one label per post and output in CSV format
3. Review every pre-assigned label independently before accepting it
4. Track which labels were corrected and in what direction - this will be disclosed in the README AI Usage section and may surface systematic disagreements between Claude's interpretation and mine

This approach speeds up annotation while keeping human judgment as the final authority on every label. It also generates disagreement cases that are themselves useful for understanding where the label boundaries are genuinely ambiguous.

---

## Stretch Features
*(Update this section before starting any stretch feature)*

- [ ] Inter-annotator reliability
- [ ] Confidence calibration
- [ ] Error pattern analysis
- [ ] Deployed interface
