# TakeMeter — planning.md

## 1. Community

**Chosen community:** r/Jamaica

### Why I chose this community

I chose r/Jamaica because it has active, public, text-based discussions about Jamaican culture, language, national identity, the diaspora, migration, and what it means to belong to Jamaica. In particular, posts about whether people born or raised outside Jamaica can identify as Jamaican often produce substantial disagreement and a wide range of discourse styles.

Comments in this community range from short emotional reactions and strict identity-based claims to detailed arguments about nationality, ancestry, citizenship, Patois, family connections, residence, migration, and cultural participation. This makes r/Jamaica appropriate for a discourse classification task because the differences between types of communication are meaningful and can be represented using clear labels.

I will focus my data collection on threads about Jamaican identity, diaspora experiences, and Patois. Potential source threads include discussions such as “Are You Really Jamaican?”, “How can the diaspora help Jamaica, vice-versa?”, and conversations about whether speaking Patois is necessary to be considered Jamaican. These threads are appropriate because they contain users directly responding to, disagreeing with, and refining each other’s ideas about cultural belonging.

The community also provides enough public posts and comments to collect at least 200 examples for annotation and evaluation.

---

## 2. Label Taxonomy

I will classify each post or comment into exactly one of the following four labels.

### Label 1: Reasoned analysis

**Definition:** Makes a developed claim about Jamaican identity, diaspora, language, or belonging and supports it with explanation, distinctions, evidence, or consideration of multiple perspectives.

**Example 1:**
> Jamaican is a nationality, but diaspora identity is still real. Someone can be legally American or British while having Jamaican parents, community ties, cultural knowledge, and a connection to the island. That does not mean their experience is identical to someone raised in Jamaica.

**Example 2:**
> Patois matters culturally, but it cannot be the only test for belonging because class, schooling, migration, and family background affect whether someone speaks it fluently.

---

### Label 2: Lived-experience contribution

**Definition:** Uses the commenter’s personal or family experience as the main way of responding to a question about Jamaican identity, culture, language, or diaspora belonging.

**Example 1:**
> I was born in New York to Jamaican parents and went back every summer. My family always called me Jamaican, but when I visited people would call me foreign, so I have always felt caught between both identities.

**Example 2:**
> I grew up in Jamaica but went to a school where speaking proper English was encouraged, so my Patois is not strong. People online have made me feel like that means I am less Jamaican.

---

### Label 3: Identity gatekeeping

**Definition:** States or implies that someone must meet a particular requirement to be legitimately Jamaican, usually without seriously considering competing definitions of Jamaican identity.

**Example 1:**
> If you were born and raised in America, you are American. Having Jamaican parents does not make you Jamaican.

**Example 2:**
> You cannot claim Jamaican culture when you do not speak Patois, do not know what life on the island is like, and only show up for Carnival or reggae events.

---

### Label 4: Reaction or affirmation

**Definition:** Expresses a brief emotional reaction, agreement, disagreement, humor, or frustration without enough explanation to qualify as reasoned analysis, lived experience, or identity gatekeeping.

**Example 1:**
> Finally, somebody said it. People take this identity conversation way too far.

**Example 2:**
> Lol, not another “who is Jamaican” thread.

---

### Label Design Rules

The labels are designed to be mutually exclusive. Every post should receive exactly one label based on the primary structure and communicative function of the post rather than whether I personally agree with its opinion.

A comment will be classified as reasoned analysis when its main purpose is to explain or support a broader claim. It will be classified as a lived-experience contribution when its main purpose is sharing personal experience. It will be classified as identity gatekeeping when its main purpose is setting a rule about who can or cannot claim Jamaican identity. It will be classified as a reaction or affirmation when it is primarily a short expression of emotion, agreement, humor, or disagreement without substantial reasoning.

The labels are also intended to be sufficiently exhaustive that at least 90% of posts from the selected community can be classified without needing an “other” category. If a post initially appears to fit multiple labels, I will use the decision rules in the Hard Edge Cases section rather than assigning multiple labels.

---

## 3. Hard Edge Case

### Anticipated ambiguous case

The hardest anticipated case is a personal story that also makes a claim about who should be considered Jamaican.

For example:

> I was born in Canada, but my parents are Jamaican and I spent every summer in St. Mary. Still, I would not call myself fully Jamaican because I was not raised through the struggles of living there.

This post could reasonably be classified as Lived-experience contribution because it is mainly about the commenter’s own family background and relationship to Jamaica, or Identity gatekeeping because it suggests that being raised in Jamaica is necessary to be fully Jamaican.

### Decision rule

I  will classify this type of post as Lived-experience contribution when the writer is primarily describing their own identity, background, feelings, or experience without applying that standard to everyone else.

I will classify it as Identity gatekeeping when the writer turns their personal experience into a general rule about other people, such as saying that no one born or raised abroad should call themselves Jamaican.

The decision will be based on observable characteristics of the text, especially whether the post uses a personal reflection such as “I feel,” “in my experience,” or “for me,” versus universal or exclusionary language such as “you cannot,” “real Jamaicans,” “only,” “must,” or “does not count.”

---

## 4. Data Collection Plan

I will collect at least 200 public posts or comments from r/Jamaica.

### Source

The examples will be collected from Reddit using publicly accessible posts and comments in r/Jamaica. I will not use private messages, private communities, deleted content that is not publicly available, or content that requires authentication.

I will prioritize comment trees from threads about Jamaican identity, diaspora relationships, cultural belonging, and Patois. I will begin by reading approximately 30–40 comments across several relevant threads before finalizing the dataset. Threads about whether someone is “really Jamaican,” the role of the diaspora, and Patois use are likely to provide the strongest material for this project.

### Dataset format

All examples will be stored in one CSV file with at least:

```text
text,label
```

I will also include a `notes` column to record difficult annotation decisions when useful:

```text
text,label,notes
```

The complete labeled dataset will be included in the GitHub repository.

### Label distribution

I will aim for a reasonably balanced dataset, with approximately equal numbers of examples per label when the community allows it.

My initial target is:

- Reasoned analysis: approximately 50 examples
- Lived-experience contribution: approximately 50 examples
- Identity gatekeeping: approximately 50 examples
- Reaction or affirmation: approximately 50 examples

I will not allow one label to account for more than 70% of the final dataset. If one label is substantially underrepresented after the first 200 examples, I will collect additional examples that naturally represent that label rather than forcing existing examples into it.

### Annotation process

I will read each post individually and assign exactly one label according to the definitions in this document.

While annotating, I will keep track of examples that genuinely cause uncertainty. I will record at least three difficult examples, including the labels I considered and the reasoning behind my final decision. These examples will later be discussed in the README.

If I use an LLM to pre-label examples, I will independently review every pre-labeled example and correct any incorrect labels before including the examples in the final dataset.

---

## 5. Evaluation Metrics

I will evaluate both the zero-shot baseline and the fine-tuned classifier on the **same held-out test set**.

### Accuracy

I will report overall accuracy because it provides a simple measure of how often the classifier assigns the correct label.

However, accuracy alone is not sufficient because the dataset may contain differences in the number of examples per label. A classifier could achieve a reasonable overall accuracy while performing poorly on a smaller class.

### Precision

I will report per-class precision to measure how often predictions of each label are actually correct.

This is useful because it shows whether the model tends to over-predict a particular type of discourse.

### Recall

I will report per-class recall to measure how many examples belonging to each label the model successfully identifies.

This is useful because a model could have high precision for a label while still missing many examples that actually belong to that class.

### F1 score

I will use per-class F1 score because it combines precision and recall into one measure. This will help determine whether the model performs consistently across the different discourse categories.

### Confusion matrix

I will also use a confusion matrix to identify which labels the model confuses with each other. This is especially important for TakeMeter because some categories may have similar language but differ in the type or structure of the discourse.

---

## 6. Definition of Success

I will consider the classifier successful if it demonstrates that fine-tuning provides a meaningful improvement over the zero-shot baseline while performing reasonably across all labels.

My specific success threshold is:

- Overall fine-tuned accuracy of at least **70%**
- F1 score of at least **0.60** for each label
- Fine-tuned accuracy higher than the zero-shot baseline
- No label with an extremely poor F1 score indicating that the model has effectively failed to learn that category

I will consider the classifier not fully successful if it performs well overall but consistently fails on one category or if fine-tuning provides little or no improvement over the zero-shot baseline.

These thresholds are intended to give me an objective way to determine whether the resulting classifier is useful rather than simply deciding afterward that the results "look good."

---

## 7. AI Tool Plan

### A. Label stress-testing

Before collecting and labeling the full dataset, I will use an AI tool to stress-test my proposed labels.

I will provide the AI tool with:

- My label definitions
- My examples for each label
- My anticipated hard edge case

I will ask it to generate approximately 5–10 hypothetical posts that sit near the boundary between two labels.

I will independently attempt to classify these generated examples. If several examples cannot be classified consistently using my definitions, I will revise the label definitions or decision rules before annotating the 200-example dataset.

The purpose of this step is to identify ambiguous label boundaries early rather than discovering the problem after training the model.

### B. Annotation assistance

I will USE an LLM to assist with pre-labeling.

If I use one, I will provide the model with my label definitions and ask it to assign one label to a batch of unlabeled posts. I will then manually review every proposed label and make the final labeling decision myself.

I will keep track of which examples were AI pre-labeled so that this assistance can be disclosed in the final README.

AI-generated labels will not automatically be treated as correct.

### C. Failure analysis

After fine-tuning, I will provide the AI tool with the model's incorrect predictions and ask it to identify possible systematic patterns.

I will specifically ask it to look for patterns involving:

- Post length
- Sarcasm
- Label pairs that are frequently confused
- Missing or weak evidence
- Certain words or phrases
- Short or low-information posts
- Differences between the intended label definitions and the model's apparent decision boundary

I will then manually review the examples and verify whether the suggested pattern is actually present. I will not treat the AI's analysis as evidence without checking the predictions myself.

---

## 8. Dataset and Model Workflow

The overall workflow will be:

1. Choose the online community.
2. Read 30–40 posts from the community.
3. Define 2–4 labels based on patterns actually present in the community.
4. Identify and define a hard edge case.
5. Complete this planning document.
6. Stress-test the labels with AI.
7. Revise the labels if necessary.
8. Collect at least 200 public posts/comments.
9. Label every example.
10. Check the label distribution.
11. Document at least three difficult annotation examples.
12. Upload the complete CSV to the TakeMeter Colab notebook.
13. Allow the notebook to split the data into training, validation, and test sets.
14. Run the zero-shot Groq baseline on the test set.
15. Record the baseline metrics.
16. Fine-tune `distilbert-base-uncased` on the training data.
17. Evaluate the fine-tuned model on the same test set.
18. Compare the fine-tuned model with the zero-shot baseline.
19. Analyze the confusion matrix and incorrect predictions.
20. Use AI to identify possible systematic error patterns and verify them manually.
21. Write the final evaluation report in `README.md`.
22. Export and commit `evaluation_results.json` and `confusion_matrix.png`.
23. Record the 3–5 minute demonstration video.
24. Link the demo video in the README.
25. Submit the GitHub repository through the course portal.

---

## 9. Fine-Tuning Plan

### Base model

I will fine-tune:

`distilbert-base-uncased`

This provides a pre-trained language model that can be adapted to the specific discourse classification task.

### Training environment

I will use Google Colab with a free T4 GPU.

### Initial training configuration

I will initially use the starter notebook's default configuration:

- Epochs: 3
- Learning rate: 2e-5
- Batch size: 16

If I change any of these values, I will document the change and explain why in the README.

### Data split

The complete labeled dataset will be uploaded as one CSV. The provided notebook will split it into:

- 70% training
- 15% validation
- 15% test

The test set will be used for the comparison between the zero-shot baseline and the fine-tuned model.

---

## 10. Baseline Plan

Before fine-tuning, I will run a zero-shot baseline using:

`meta-llama/llama-4-scout-17b-16e-instruct`

through Groq.

The prompt will include the same label definitions used for annotation and will instruct the model to output exactly one label name for each test example.

I will evaluate the baseline on the test set and record:

- Overall accuracy
- Per-class precision
- Per-class recall
- Per-class F1
- Unparseable responses, if any

I will complete the baseline evaluation before evaluating the fine-tuned model so that the comparison is not influenced by the fine-tuned results.

---

## 11. Expected Evaluation Analysis

After training, I will compare the two models using the same test examples.

I will examine:

1. Whether fine-tuning improves overall accuracy.
2. Which labels have the highest and lowest F1 scores.
3. Which label pairs are most frequently confused.
4. Whether errors are concentrated in particular types of posts.
5. Whether the model appears to rely on superficial features rather than the intended discourse characteristics.
6. Whether the difficult cases identified during annotation are also difficult for the model.

I will analyze at least three specific incorrect predictions from the fine-tuned model.

For each error, I will identify:

- The actual label
- The predicted label
- Why the example may have been difficult
- Whether the problem appears related to the label definition, annotation, or training data
- What change could potentially improve the model

---

## 12. Stretch Features

I will decide whether to complete any of the following after the required project is working:

### Inter-annotator reliability

Have another person independently label at least 30 examples and calculate agreement using either Cohen's kappa or percentage agreement.

### Confidence calibration

Evaluate whether the model's confidence scores correspond to actual correctness.

### Error pattern analysis

Identify a systematic error pattern across multiple incorrect predictions rather than analyzing only individual mistakes.

### Deployed interface

Create a simple interface where a user enters a new post and the fine-tuned model returns the predicted label and confidence.

If I complete any stretch feature, I will update this `planning.md` before starting that feature.

---

## 13. Deliverables

The final repository will contain:

```text
planning.md
README.md
dataset.csv
evaluation_results.json
confusion_matrix.png
```

The README will contain:

- Community choice and reasoning
- Label taxonomy
- Two examples per label
- Data collection source
- Labeling process
- Label distribution
- Three difficult annotation examples
- Fine-tuning approach
- Base model
- Training configuration
- Baseline methodology
- Baseline results
- Fine-tuned results
- Accuracy for both models
- Per-class metrics for both models
- Confusion matrix as a Markdown table
- Three analyzed incorrect predictions
- Sample classifications with confidence scores
- Reflection on intended versus learned behavior
- Spec reflection
- AI usage section
- Link to the 3–5 minute demo video