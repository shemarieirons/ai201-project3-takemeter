# TakeMeter: Discourse Classifier for r/Jamaica

TakeMeter is a fine-tuned text classifier that labels r/Jamaica comments by the kind of contribution they make to conversations about Jamaican identity, the diaspora, Patois, and belonging. It fine-tunes `distilbert-base-uncased` on 276 hand-labeled comments and compares it against a zero-shot LLM baseline on the same held-out test set.

**Headline result:** the fine-tuned model reaches 0.738 accuracy against the baseline's 0.690, but the gain is only two comments, and the model never predicts the rare Identity gatekeeping label (F1 = 0.00). It is a clear improvement on Lived experience and a regression on the hardest class.

- **Demo video:** [INSERT LINK]
- **Dataset:** [`dataset.csv`](dataset.csv) (276 labeled comments)
- **Planning document:** [`planning.md`](planning.md)

---

## Community

I chose **r/Jamaica**, a public subreddit with active, text-heavy discussion of Jamaican culture, language, national identity, and migration. Threads such as "Are You Really Jamaican?" and debates over whether Patois is required to count as Jamaican produce a wide range of discourse in one place: short jokes, personal testimony, strict eligibility rules, and long arguments about citizenship versus ancestry.

That variety is what makes it a good classification target. "I grew up abroad and felt caught between two places" and "you cannot claim Jamaica if you were born abroad" are about the same topic but do very different things, and regular participants recognize the difference. The task is about **discourse function, not topic**. To keep the model from learning topic keywords alone, I sampled identity, diaspora, and Patois threads and also other culturally relevant discussions (dancehall, tourism, local issues) where the same discourse types occurred naturally.

---

## Label Taxonomy

Each comment gets exactly one label, based on its primary communicative function, not on whether I agree with it.

| Label | Definition | Examples |
|---|---|---|
| **Reasoned analysis** | Makes a developed claim about Jamaican identity, diaspora, language, or belonging and supports it with explanation, distinctions, evidence, or multiple perspectives. | "Hapana, that's just your opinion. Luckily for us you aren't the arbiter of who is and isn't Jamaican. Its a hard sell to deny people born to Jamaicans and raised by Jamaicans among other Jamaicans outside Jamaica arent Jamaican."<br><br>"I am not saying you're not Jamaican... what I am posting about are people who don't know what is happening here...telling you" |
| **Lived-experience contribution** | Uses the commenter's own personal or family experience as the main way of responding to a question about identity, culture, language, or belonging. | "I grew up with my granny and my aunts, uncles are all teachers. I know that I sound crazy when I speak patois because granny would make me correct every word that came out my mouth if it was not proper English."<br><br>"As a person who was born here (in the St. Ann's Bay hospital), raised here and still lives here, I just can't speak patois. Everytime I try, I just sound like a white person doing one of those phony imitations." |
| **Identity gatekeeping** | States or implies that a person must meet a specific requirement to legitimately identify as Jamaican, or that another person does not qualify because they fail it. | "Jamaican is not a race. You can say you are of Jamaican descent. But you can't say you're Jamaican."<br><br>"It's not about opinions but rather pretentious POS acting like they're one of us when the fact of the matter is they were not born here so they're NOT Jamaicans" |
| **Reaction or affirmation** | Primarily expresses a brief emotional response, agreement, disagreement, humor, or frustration, without developing an argument, giving substantial personal experience, or setting an identity rule. | "Lol just goes to show the truth hurts really"<br><br>"Don't be so sensitive. We're all Jamaican." |

---

## Dataset

**Source:** public posts and comments from r/Jamaica, collected manually. Before finalizing labels I read 30 to 40 comments to check the definitions applied cleanly. The CSV has columns `text`, `label`, and `notes` (for difficult cases).

| Label | Count | Share |
|---|---:|---:|
| Reasoned analysis | 100 | 36.2% |
| Lived-experience contribution | 76 | 27.5% |
| Reaction or affirmation | 76 | 27.5% |
| Identity gatekeeping | 24 | 8.7% |
| **Total** | **276** | **100%** |

No label exceeds 70%, but **Identity gatekeeping is well under the 20% target I set in planning**. Explicit eligibility rules were rarer than expected in the sampled threads, and I did not relabel other comments to fill a quota. This turned out to matter a great deal for the results.

The stratified 70/15/15 split (`random_state=42`) gives:

| Split | Total | Reasoned | Lived experience | Gatekeeping | Reaction |
|---|---:|---:|---:|---:|---:|
| Train | 193 | 70 | 53 | 17 | 53 |
| Validation | 41 | 15 | 11 | 4 | 11 |
| Test | 42 | 15 | 12 | 3 | 12 |

The test set has only **3 gatekeeping examples**, so every gatekeeping number below is very noisy.

**Annotation process.** Claude pre-labeled the comments using my label definitions and decision rules. I reviewed each proposed label against the definitions and changed 2 of them. Difficult calls were recorded in the `notes` column. The very low change rate has a downside I want to flag: my final labels sit close to one LLM's judgment, and I did not measure agreement with an independent human annotator, so any systematic bias in that pre-labeling is likely baked into the dataset.

### Difficult labeling decisions

The hardest boundary was **lived experience vs. identity gatekeeping**: a personal story that also implies who counts as Jamaican. My rule: it is *lived experience* when the writer describes their own identity, background, or feelings without applying a standard to others, and *gatekeeping* when they use their experience to set a general rule. Cues for lived experience are "for me," "I feel," and "in my experience." Cues for gatekeeping are "real Jamaicans," "you cannot," "only," "must," "does not count," and "nobody."

1. **Kept as Lived experience.** A commenter who left Jamaica for school overseas said that if someone hasn't been back in years, it is better to leave certain conversations alone. Gatekeeping was a candidate because of the advice to others, but they applied the caution to themself, and the main move was describing personal distance from home.
2. **Kept as Lived experience.** A long comment about the writer once denying other people's Jamaican identity and later changing their mind. Gatekeeping language appears, but only in the past tense inside the writer's own story, so it is a personal trajectory and not a current eligibility rule.
3. **Kept as Identity gatekeeping.** An anecdote about a returned resident who lived in Jamaica for 50 years, left for New York, and was later treated as a foreigner by taxi drivers. Lived experience was a candidate because it is a story, but the anecdote supports a broader claim about who gets treated as Jamaican.

---

## Zero-Shot Baseline

**Model:** `openai/gpt-oss-20b` via Groq (the model configured in my notebook). This differs from the model named in the assignment (`meta-llama/llama-4-scout-17b-16e-instruct`) and from the one in my planning document (`openai/gpt-oss-120b`), so the baseline numbers here apply to gpt-oss-20b only.

**Method:** A system prompt described the task, gave the four definitions with one example each plus the borderline decision rules, and told the model to output only a label name. Each of the 42 test comments was classified independently at `temperature=0`, with no training examples shown. Groq's free tier caps tokens per minute, so the loop retries with backoff on HTTP 429 errors. All 42 responses parsed (0 unparseable).

**Where it struggled.** Working back from precision and recall, the baseline predicted Identity gatekeeping 8 times and only 2 were correct, so six other comments were pulled into that label. It predicted Reaction 16 times against 12 true examples. It missed half of the Lived-experience comments (recall 0.50) even though every Lived-experience prediction it made was right (precision 1.00).

**Hypothesis:** the zero-shot model reacts to surface features, reading opinionated or exclusionary wording as gatekeeping and only recognizing lived experience when a comment is an unmistakable first-person story. Fine-tuning supports part of this: Lived-experience recall rose from 0.50 to 0.83, consistent with the baseline missing less obvious testimony. I did not read the baseline's individual errors, so the claim that it over-reads exclusionary wording as gatekeeping is untested at the example level and rests only on its prediction counts.

---

## Fine-Tuning

**Base model:** `distilbert-base-uncased` with a new 4-way classification head. **Training:** Hugging Face `Trainer` on a Colab T4 GPU, 193 training examples, max sequence length 256, evaluated on the validation set each epoch. `load_best_model_at_end=True` with validation accuracy as the metric, so the test set is scored with the best epoch.

### Hyperparameter decision

I started with the notebook defaults: 3 epochs, learning rate 2e-5, batch size 16, 50 warmup steps.

| Epoch | Training loss | Validation loss | Validation accuracy |
|---:|---:|---:|---:|
| 1 | 1.391 | 1.366 | 0.268 |
| 2 | 1.362 | 1.297 | 0.463 |
| 3 | 1.285 | 1.199 | 0.415 |

This run failed. 193 examples at batch size 16 is about 13 steps per epoch, **39 steps in total**. Training loss barely moved (chance for four classes is ln 4 ≈ 1.386), and the model predicted "Reasoned analysis" for all 42 test comments (accuracy 15/42 = 0.357):

| Actual \ Predicted | Reasoned | Lived experience | Gatekeeping | Reaction |
|---|---:|---:|---:|---:|
| **Reasoned** | 15 | 0 | 0 | 0 |
| **Lived experience** | 12 | 0 | 0 | 0 |
| **Gatekeeping** | 3 | 0 | 0 | 0 |
| **Reaction** | 12 | 0 | 0 | 0 |

The cause was warmup. A 50-step warmup is longer than the entire 39-step run, so the learning rate never reached its target and the new classification head barely trained. I changed three settings:

| Setting | Default | Final | Reason |
|---|---:|---:|---|
| `warmup_steps` | 50 | 10 | Warmup must finish early in the run (the main fix). |
| `num_train_epochs` | 3 | 8 | About 104 updates instead of 39. |
| `learning_rate` | 2e-5 | 5e-5 | A tiny dataset needs larger updates to the new layers. |

Batch size (16) and weight decay (0.01) were unchanged. Retrained run:

| Epoch | Training loss | Validation loss | Validation accuracy |
|---:|---:|---:|---:|
| 1 | 1.240 | 1.121 | 0.463 |
| 2 | 1.142 | 0.880 | 0.683 |
| **3** | **0.767** | **0.824** | **0.707** |
| 4 | 0.364 | 0.985 | 0.659 |
| 5 | 0.272 | 0.984 | 0.585 |
| 6 | 0.229 | 1.116 | 0.634 |
| 7 | 0.122 | 0.952 | 0.659 |
| 8 | 0.090 | 1.032 | 0.634 |

Training now works, but the model **overfits after epoch 3**: training loss keeps falling while validation loss rises. The model scored on the test set is the **epoch 3 checkpoint**. Caveat: the validation set has 41 examples (one comment is about 2.4 accuracy points), and the best epoch was chosen on those same examples, so 0.707 is probably slightly optimistic.

---

## Evaluation Report

Everything below uses the same held-out test set of 42 comments.

### Accuracy and per-class metrics

| Model | Accuracy | Macro F1 |
|---|---:|---:|
| Zero-shot baseline (`gpt-oss-20b`) | 0.690 (29/42) | 0.64 |
| Fine-tuned, default settings (failed run) | 0.357 (15/42) | n/a |
| **Fine-tuned DistilBERT, final (epoch 3)** | **0.738 (31/42)** | **0.58** |

| Label | Support | Baseline P / R / F1 | Fine-tuned P / R / F1 |
|---|---:|---|---|
| Reasoned analysis | 15 | 0.83 / 0.67 / 0.74 | 0.69 / 0.73 / 0.71 |
| Lived-experience contribution | 12 | 1.00 / 0.50 / 0.67 | 0.91 / 0.83 / 0.87 |
| Identity gatekeeping | 3 | 0.25 / 0.67 / 0.36 | 0.00 / 0.00 / 0.00 |
| Reaction or affirmation | 12 | 0.69 / 0.92 / 0.79 | 0.67 / 0.83 / 0.74 |
| **Macro average** | 42 | 0.69 / 0.69 / 0.64 | 0.57 / 0.60 / 0.58 |

### Confusion matrix (fine-tuned, final run)

Rows are true labels, columns are predicted labels. A copy is in `confusion_matrix.png`.

| Actual \ Predicted | Reasoned | Lived experience | Gatekeeping | Reaction |
|---|---:|---:|---:|---:|
| **Reasoned** | 11 | 1 | 0 | 3 |
| **Lived experience** | 1 | 10 | 0 | 1 |
| **Gatekeeping** | 2 | 0 | 0 | 1 |
| **Reaction** | 2 | 0 | 0 | 10 |

The Gatekeeping column is all zeros: the model never predicted that label for any of the 42 comments. Of the 11 errors, 9 involve Reasoned analysis as either the true label or the prediction.

### Fine-tuned vs. baseline

Fine-tuning improved accuracy from 0.690 to 0.738, but that is **31 versus 29 correct comments**, a difference too small to call reliable on a 42-example test set. The per-class picture is more informative than the headline number. Fine-tuning clearly helped Lived experience (F1 0.67 to 0.87, recall 0.50 to 0.83). It slightly hurt Reasoned analysis (0.74 to 0.71) and Reaction (0.79 to 0.74), and it eliminated Identity gatekeeping entirely (0.36 to 0.00). As a result, **macro F1 is lower for the fine-tuned model (0.58) than for the baseline (0.64)**. The two models fail in opposite ways on the rare class: the baseline over-predicts gatekeeping, while the fine-tuned model never predicts it. The fine-tuned model's accuracy edge comes from the larger classes, at the expense of the smallest one.

### Sample classifications

| Post (truncated) | True label | Predicted | Confidence | Correct? |
|---|---|---|---:|---|
| Are you trying to say ethnicity is different from nationality, or are you trying to say be... | Reasoned analysis | Reasoned analysis | 0.73 | yes |
| I met a man from Jamaica who has lived in The Bronx for 40 plus years and you'd think he j... | Lived-experience contribution | Lived-experience contribution | 0.81 | yes |
| Why do you feel the need to say something like this? | Reaction or affirmation | Reaction or affirmation | 0.79 | yes |
| It's not a brag to not be able to speak the language of the country. It's actually a shame... | Reasoned analysis | Reaction or affirmation | 0.40 | no |
| As someone who was born and raised in Jamaica and lived overseas I can say Afro-Jamaicans ... | Lived-experience contribution | Reasoned analysis | 0.78 | no |

**Why a correct prediction is reasonable.** "Why do you feel the need to say something like this?" is a short rhetorical question that expresses frustration without building an argument, telling a personal story, or setting a rule about who counts as Jamaican, which matches the Reaction definition exactly. The model predicted it with high confidence (0.79). Short, plainly expressive comments like this are where the model is most reliable: 8 of the 9 comments of 15 words or fewer were classified correctly (see Error Pattern Analysis).

### Three wrong predictions

**Error 1: Identity gatekeeping predicted as Reasoned analysis (confidence 0.66).**
> "This is a huge problem. A lot of the people online claiming to be Jamaican are 2nd or 3rd generation "Jamaicans" in a foreign land, and some have never spent a month combined in the country. They tend..."

*Why the boundary is hard:* I labeled this gatekeeping because it implies those commenters do not qualify as Jamaican, but it never states a rule outright. It is written in an expository, evidence-flavored register ("a lot of the people... some have never spent a month combined"), which looks like Reasoned analysis on the surface. *Labeling or data problem?* Mostly a data problem. I labeled this consistently with other implicit-gatekeeping comments, but the model saw only 17 gatekeeping training examples, too few to learn that the label can be carried by implication and not by explicit "you cannot" wording. *What would fix it:* many more gatekeeping examples, especially implicit ones phrased as complaint or explanation, and possibly a definition that says explicitly how to treat claims about who does not qualify that are phrased as observations.

**Error 2: Lived-experience contribution predicted as Reasoned analysis (confidence 0.78).**
> "As someone who was born and raised in Jamaica and lived overseas I can say Afro-Jamaicans are indeed a verifiable ethnicity... The problem is they just don't want to be viewed as an ethnicity. We are..."

*Why the boundary is hard:* This is the edge case from my planning, in a different form. The writer opens with personal credentials, but then makes a general claim about an ethnic group, so it mixes testimony with argument. It is also long (119 words), and the model leans toward Reasoned analysis for long comments. *Labeling or data problem?* A boundary problem more than an annotation inconsistency: my definitions say lived experience is "the main way" the writer responds, and here the personal framing is only the opener. This is the most confident error in the whole test set (0.78), so the model is not unsure; it has learned that long, claim-making text means analysis. *What would fix it:* more training examples where personal credentials introduce a general claim, with a clearer rule for them, such as "labeled by what most of the comment does, not how it opens".

**Error 3: Reasoned analysis predicted as Reaction or affirmation (confidence 0.49).**
> "Then what is "Jamaican"? An ethnicity, a natiojality, or both? And what type of nationality?"

*Why the boundary is hard:* This is a short (15-word) question that raises the central conceptual issue of the thread but does not argue anything. It could be read as an engaged contribution to the discussion or as a brief interjection. *Labeling or data problem?* Partly a labeling problem. Another test comment, "Ok so the minivan is more akin to taking public transportation vs the upper class having their own school transport system...?", is also a question, but I labeled that one Reaction, and the model predicted Reasoned analysis for it (confidence 0.47), the exact opposite of this error. My definitions say nothing about how to handle questions, so the two were labeled differently and the model's guesses mirror that inconsistency. *What would fix it:* add an explicit rule for questions (for example, a question is Reasoned analysis only if it advances a position) and re-check the labels of all question-form comments.

---

## Stretch Feature: Confidence Calibration

I took the softmax probability of the predicted class as the model's confidence and grouped the 42 test predictions into bins.

| Confidence bin | Predictions | Correct | Accuracy | Mean confidence |
|---|---:|---:|---:|---:|
| below 0.50 | 7 | 3 | 43% | 0.45 |
| 0.50 to 0.69 | 11 | 5 | 45% | 0.61 |
| 0.70 to 0.79 | 12 | 11 | 92% | 0.77 |
| 0.80 and above | 12 | 12 | 100% | 0.84 |

Mean confidence is 0.74 on correct predictions and 0.57 on wrong ones.

**Finding.** The confidence scores are meaningful, especially at the top: every one of the 12 predictions at 0.80 or above was correct, and 23 of 24 predictions at 0.70 or above (96%) were correct. Below 0.70, only 8 of 18 were correct (44%). A practical use is a 0.70 threshold: the model answers the 24 comments (57%) it is confident about with 96% accuracy, and sends the rest to a human.

The model is **not well calibrated in the middle**. In the 0.50 to 0.69 bin, mean confidence is 0.61 but accuracy is 45%, so it is overconfident there. Within that range, the 0.60 to 0.69 sub-bin is right only 2 of 6 times (33%) at a mean confidence of 0.67, and that sub-bin contains all three Identity gatekeeping errors (confidence 0.66, 0.67, 0.67), where the model is wrong without being unsure. There is also one confident error above 0.70, the Lived experience comment predicted as Reasoned analysis at 0.78 (Error 2). Caveats: with 42 predictions, each bin holds only 7 to 12 examples, so these percentages could shift a lot with a larger test set, and the model is the epoch 3 checkpoint, so these numbers would likely look different for a longer-trained, more overfit model.

---

## Stretch Feature: Error Pattern Analysis

There are 11 wrong predictions out of 42. I looked for patterns that hold across the error set, not just in single examples. Claude proposed candidate patterns from the list of wrong predictions and per-example confidences, and the counts below were computed directly from the test-set output.

**Pattern 1: the model never learned Identity gatekeeping, and absorbs it into Reasoned analysis or Reaction.**
All 3 gatekeeping comments were misclassified (2 as Reasoned analysis, 1 as Reaction), and the label was predicted 0 times out of 42, each time with confidence of 0.66 to 0.67. The gatekeeping comments in the test set carry the eligibility rule by implication inside an explanatory or venting register: Error 1 above, and "I agree, those unpatriotic turncoats need to leave Jamaican affairs to actual Jamaicans...", which reads as agreement plus complaint. The model appears to classify by register (explaining versus venting) rather than by the function the label is defined by. With only 17 training examples, it had too little evidence to separate gatekeeping from those registers.

**Pattern 2: Reasoned analysis is the hub of the confusion.**
9 of 11 errors involve Reasoned analysis as either the true or the predicted label. The model predicted it 16 times against 15 true instances (precision 0.69). It both absorbs comments from other classes (5 of the 11 errors are other labels predicted as Reasoned analysis: one Lived, two Gatekeeping, two Reaction) and loses its own: 3 of its 4 misses went to Reaction, all at low confidence (0.40 to 0.50). Reasoned analysis is the broadest and largest class, so the model's boundary around it is the least sharp.

**Pattern 3 (weaker): mid-length comments are hardest.**

| Comment length | Comments | Wrong | Error rate |
|---|---:|---:|---:|
| 15 words or fewer | 9 | 1 | 11% |
| 16 to 50 words | 16 | 6 | 38% |
| over 50 words | 17 | 4 | 24% |

Very short comments are mostly clear reactions, and very long ones are mostly clear testimony (the model gets the longest Lived-experience comments, 388 and 200 words, right at 0.85 and 0.86). Comments in the middle are where register is ambiguous. I treat this as suggestive only: the groups are small (9, 16, 17) and the 16 to 50 word group is partly the same comments as Pattern 2.

**What I conclude.** Errors are concentrated on the boundaries I defined by function rather than form: the model separates comments by tone and length, which works for clearly short reactions and clearly long stories, and breaks down for implicit gatekeeping and for question-form comments (see Error 3). The most useful fix is more and more varied gatekeeping data, followed by explicit rules for questions.

---

## Reflection: What the Model Learned vs. What I Intended

I intended the model to classify by **discourse function**: personal testimony as Lived experience, developed argument as Reasoned analysis, general eligibility rules as Identity gatekeeping, and short expressive comments as Reaction.

What it appears to have learned is closer to **register and length**. Where function and register coincide, it does well: first-person stories are Lived experience (F1 0.87, and its confidences are the highest of any class), and short punchy comments are Reaction. Where function and register diverge, it follows register. A gatekeeping comment written like an explanation gets labeled analysis; a gatekeeping comment written like a complaint gets labeled reaction; a comment that opens with personal credentials but then argues gets labeled analysis. It also effectively learned a prior against the rare class: Identity gatekeeping makes up 9% of the data, and a model trained for few steps on 17 examples can lower its loss by never predicting it.

What it **overfit to**: after epoch 3 training loss kept dropping while validation loss rose, and I used the epoch 3 checkpoint to limit this. With 193 examples, it likely memorized phrasings rather than structure.

What it **missed** is the thing I designed the taxonomy around. Gatekeeping is defined by an eligibility claim (who counts), which can be implicit and can appear inside any register, and 17 examples were not enough for a model to find that signal beneath the style differences. A second miss is that my own definitions underspecify questions, so part of the model's difficulty on those comments reflects my labels, not just its limits.

### Success criteria check

Thresholds set in `planning.md` before training:

| Criterion | Target | Result | Met? |
|---|---|---|---|
| Fine-tuned accuracy | at least 0.70 | 0.738 | Yes |
| Per-label F1 | at least 0.60 for every label | 0.71, 0.87, 0.00, 0.74 | No (Gatekeeping 0.00) |
| Beats baseline | above 0.690 | 0.738 | Yes (by 2 comments) |
| No label effectively failed | no label near F1 0 | Gatekeeping F1 = 0.00 | No |

Two of four criteria were met. I would not deploy this classifier as a community tool, because the label I care most about for moderation-style use (gatekeeping) is the one it cannot detect. It could be useful for Lived experience, where precision is 0.91, and the confidence scores could route low-confidence comments to a human. The fix I would try first is collecting several times more gatekeeping examples, since the failure traces back to the 17 training examples.

---

## Spec Reflection

**One way the spec helped.** Writing the lived-experience vs. gatekeeping decision rule in `planning.md` before labeling at scale let me resolve borderline comments consistently. The same rule went into the baseline prompt, so the baseline and I worked from the same boundary. Setting numeric success thresholds before training also kept me from moving the goalposts afterward, and it made the verdict above easy to state: two of four targets met.

**Where the implementation diverged.**
- *Class balance.* The plan targeted about 50 examples per label. I ended with 24 gatekeeping examples because explicit eligibility rules were rarer than expected, and I did not force other comments into that label. This is the divergence that mattered most: it left only 17 training and 3 test gatekeeping examples and led directly to the model's biggest failure.
- *Training configuration.* The plan said to use the notebook defaults. They collapsed to one class on this dataset, so I changed warmup, epochs, and learning rate.
- *Baseline model.* The plan named `openai/gpt-oss-120b`, the assignment names `meta-llama/llama-4-scout-17b-16e-instruct`, and the baseline I ran used `openai/gpt-oss-20b`.
- *Questions.* The spec had no rule for question-form comments, and the error analysis showed I labeled two of them inconsistently.

---

## AI Usage

1. **Label stress-testing.** I gave an LLM my definitions, examples, and the lived-experience vs. gatekeeping edge case and asked for borderline posts between label pairs. I classified them myself and used the results to settle the decision rule that personal framing stays lived experience unless the writer applies a standard to others. I did not keep a log of individual suggestions I rejected.
2. **Annotation assistance (disclosure).** Claude pre-labeled the collected comments from my four definitions and decision rules. I reviewed the proposed labels and changed 2 of them. Because I changed so few, my dataset labels are close to Claude's judgment, which is a limitation noted in the Dataset section. The baseline is a different model (`gpt-oss-20b`), so it is not scoring its own labeler's work.
3. **Debugging the baseline (Claude).** My baseline returned 42 of 42 unparseable responses. I pasted the code and output, and Claude identified that the parser compared lowercase model output against my `LABEL_MAP` keys (capitalized, with spaces) while the prompt told the model to answer in snake_case, so nothing could match. It suggested a translation dictionary. I placed it inside `classify_with_groq` and merged the function and loop into one cell because I needed the whole cell to work together. Later, rate-limit errors left single comments unparsed, and I used a bounded retry with backoff so all 42 were scored.
4. **Diagnosing the training collapse (Claude).** I shared the all-"Reasoned analysis" confusion matrix and my training log. Claude noticed the 50-step warmup exceeded the 39 total steps and proposed `warmup_steps=10`, 8 epochs, and `learning_rate=5e-5`. I ran the retrain, kept the failed run as evidence, and reported that the best checkpoint is epoch 3 and that the validation set is small, rather than presenting 0.707 as a clean result.
5. **Failure analysis (Claude).** I pasted the 11 wrong predictions and the per-example confidences into Claude and asked for patterns. Claude proposed the patterns in the Error Pattern Analysis section and computed the confidence and length breakdowns from my output. I kept only patterns supported by counts, and labeled the length pattern as weak because its groups are small. The finding that two question-form comments were labeled inconsistently came out of this analysis and was not something I had noticed while annotating.

---

## Files

| File | Purpose |
|---|---|
| `planning.md` | Spec written before data collection, with an update before each stretch feature |
| `README.md` | This report |
| `dataset.csv` | 276 labeled comments (`text`, `label`, `notes`) |
| `evaluation_results.json` | Exported accuracy for both models |
| `confusion_matrix.png` | Fine-tuned confusion matrix (final run) |
| `confusion_matrix_default_run.png` | Failed default-settings run (optional) |
| `[notebook].ipynb` | Colab notebook for training and baseline |
