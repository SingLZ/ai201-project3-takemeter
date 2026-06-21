# Project 3 Planning: r/nba Discourse Classification

## 1. Community

For this project, I will study discourse in **r/nba**, a public Reddit community focused on NBA discussion. I chose this community because NBA discussion is active, text-heavy, and varied in quality. Some comments make detailed basketball arguments using statistics or tactical reasoning, while others are mostly emotional reactions, unsupported opinions, or bold claims.

This community is a good fit for a classification task because the discourse naturally contains different types of basketball discussion. Fans often distinguish between serious analysis, reactionary takes, and emotional responses to games or news. That makes the classification problem meaningful instead of arbitrary.

The unit of analysis will be individual Reddit comments, not full threads. Comments are easier to label consistently because each one usually contains one main idea or reaction.

## 2. Labels

I will use three labels:

1. `analysis`
2. `hot_take`
3. `reaction`

These labels are intended to separate substantive basketball reasoning from unsupported opinion and emotional response.

### Label: `analysis`

A comment should be labeled `analysis` when it makes a structured basketball argument supported by specific evidence, tactical reasoning, statistics, historical comparison, or clear cause-and-effect explanation.

Example 1:

> The Wolves’ defense worked because they kept sending early help from the weak side and forced Denver’s role players to make quick decisions. Jokic still got his numbers, but the offense slowed down because the cutters were covered before the pass was available.

Example 2:

> Brunson’s scoring jump is not just higher usage. His footwork in the midrange lets him create clean looks without relying on elite athleticism, and his turnover rate has stayed low even with more defensive attention.

### Label: `hot_take`

A comment should be labeled `hot_take` when it makes a bold, confident, or controversial basketball claim without enough supporting evidence. The claim may be true or interesting, but the comment mostly asserts instead of argues.

Example 1:

> Tatum is never winning as the best player on a title team. He disappears whenever the game actually matters.

Example 2:

> The Lakers should trade everyone except LeBron and AD. This roster is cooked and there is no path forward.

### Label: `reaction`

A comment should be labeled `reaction` when it mainly expresses emotion, surprise, frustration, hype, sarcasm, or disappointment in response to a basketball event. It does not try to build a serious argument.

Example 1:

> I can’t believe they blew that lead again. This team is impossible to watch.

Example 2:

> That dunk was disgusting. He actually ended that man’s career.

## 3. Hard Edge Cases

The hardest edge case will be comments that make a strong claim and include one statistic or piece of evidence.

Example:

> LeBron is overrated because his playoff record against top-seeded teams is below .500.

This comment could look like `analysis` because it includes a statistic. However, it could also be `hot_take` because the statistic is used narrowly to support a provocative claim without enough context.

### Decision Rule

If a comment includes evidence but does not explain why the evidence supports the claim, I will label it `hot_take`.

If the comment connects the evidence to a broader basketball argument, explains context, or compares alternatives fairly, I will label it `analysis`.

Examples:

* `hot_take`: “Player X is overrated because one stat looks bad.”
* `analysis`: “Player X struggles in this matchup because the opposing defense takes away his first option, and the numbers show his efficiency drops when forced into pull-up jumpers.”

Another edge case is a comment that is both emotional and opinionated.

Example:

> This team is embarrassing. The coach has no idea how to manage rotations.

This could be either `reaction` or `hot_take`. I will label it `reaction` if the main purpose is emotional frustration. I will label it `hot_take` if the comment makes a clear basketball claim, such as criticizing coaching, roster construction, or player quality.

## Difficult Annotation Cases From Milestone 3

### Case 1

Text:

> Wembanyama in Spurs wins this series: 25 FGA in Game 1 and 22 FGA in Game 4. Wembanyama in Spurs losses this series: 16 FGA in Game 2, 15 FGA in Game 3, and 15 FGA in Game 5. SAN ANTONIO, WEMBY, YOUR GAMEPLAN COULD NOT BE ANY MORE SIMPLER FOR GAME 6; DEMAND THE BALL

Possible labels: `analysis` or `hot_take`

Final label: `analysis`

Reason:

This comment has emotional and exaggerated wording, especially in the final sentence, which makes it partly look like a `hot_take`. However, it gives a specific field-goal-attempt split between Spurs wins and losses and uses that evidence to support a clear game-plan argument. Because the evidence is connected to a basketball claim, I labeled it `analysis`.

### Case 2

Text:

> Okc was a better team and both stars were so off timing. Also Tony Brothers may just be a supervillain.

Possible labels: `analysis` or `reaction`

Final label: `analysis`

Reason:

This comment includes a joke about Tony Brothers, which makes it look partly like a `reaction`. However, the main part of the comment explains the result by saying OKC was the better team and that both stars were off timing. Since the comment gives a basketball explanation instead of only expressing emotion, I labeled it `analysis`.

### Case 3

Text:

> Mitch Johnson has to play the media game. He has to call out what they were doing to Wemby. He has enough status as the next face of the league that it will become a story. If he doesn’t they’ll do this for the rest of the series.

Possible labels: `hot_take` or `analysis`

Final label: `hot_take`

Reason:

This comment makes a strategic claim about how Mitch Johnson should use the media to influence the series. It has some reasoning, but it does not provide specific evidence, examples, or basketball context showing that this strategy would actually work. Because the comment mostly asserts a confident opinion without enough support, I labeled it `hot_take`.

## 4. Data Collection Plan

I will collect approximately **200 public Reddit comments** from r/nba. I will focus on discussion-heavy threads where comments are more likely to contain meaningful text.

Potential sources include:

* Post-game threads
* Trade discussion threads
* Injury news threads
* Player performance threads
* Award debate threads
* Team analysis threads
* Legacy comparison threads

I will avoid examples that are not useful for classification, including:

* Deleted comments
* Bot comments
* Comments that are only links
* Comments that are only emojis
* Very short comments with no basketball meaning
* Duplicate or near-duplicate comments

### Target Label Balance

I will try to collect a balanced dataset:

| Label      | Target Count |
| ---------- | -----------: |
| `analysis` |        65–70 |
| `hot_take` |        65–70 |
| `reaction` |        65–70 |

A balanced dataset matters because if one label dominates, the model may learn to overpredict that label instead of learning the actual difference between discourse types.

### Underrepresented Label Plan

If one label is underrepresented after collecting 200 examples, I will collect more comments from thread types where that label is more common.

For example:

* If `analysis` is underrepresented, I will collect more from serious discussion threads, post-game breakdowns, and team-analysis threads.
* If `hot_take` is underrepresented, I will collect more from ranking debates, trade debates, award debates, and legacy comparison threads.
* If `reaction` is underrepresented, I will collect more from post-game threads, highlight threads, injury reactions, and breaking news threads.

If a label is still hard to find, I will not force weak examples into that label. Instead, I will either collect more data or adjust the label taxonomy before training.

## 5. Evaluation Metrics

Accuracy alone is not enough for this task because the dataset may not be perfectly balanced, and some labels are easier to predict than others. A model could get decent accuracy by doing well on common or easy labels while still failing on the most important boundary cases.

I will use the following metrics:

### Accuracy

Accuracy will show the overall percentage of correct predictions. This is useful as a general performance measure, but it will not be the only metric.

### Precision, Recall, and F1 Score Per Label

I will evaluate precision, recall, and F1 score for each label.

Precision matters because I want to know whether the model’s predictions for each label are trustworthy. For example, if the model predicts `analysis`, I want that prediction to usually be correct.

Recall matters because I want to know whether the model is finding most examples of each label. For example, if many real `analysis` comments are missed, the model would not be useful for identifying substantive discussion.

F1 score matters because it balances precision and recall. This is especially important if one label is harder to classify than the others.

### Macro F1

Macro F1 will be one of my main metrics because it treats all labels equally. This is important because I care about performance across all three discourse types, not just the most common one.

### Confusion Matrix

I will use a confusion matrix to see which labels the model confuses most often. This is especially important for this task because I expect the hardest distinction to be between `analysis` and `hot_take`.

If the confusion matrix shows that the model frequently confuses `analysis` and `hot_take`, that may mean my label definitions are not clear enough or that the model has trouble recognizing shallow evidence versus real reasoning.

## 6. Definition of Success

A successful classifier should be able to separate serious basketball analysis from unsupported takes and emotional reactions better than a simple baseline.

For this project, I will define success as:

* Overall accuracy of at least **75%**
* Macro F1 score of at least **0.72**
* F1 score of at least **0.70** for each individual label
* No single label should have recall below **0.65**
* The confusion matrix should not show one label being consistently collapsed into another

For a real community tool, I would want stronger performance before deployment. A useful real-world classifier should reach:

* At least **80% accuracy**
* At least **0.78 macro F1**
* Strong performance on the `analysis` label, because misclassifying serious analysis as low-quality discourse would make the tool less trustworthy

I would not use this classifier to automatically remove or punish comments. At most, it could be used as a lightweight sorting, tagging, or research tool to study discussion patterns. Human review would still be needed for any moderation-related use.

## 7. AI Tool Plan

AI tools will be used carefully in this project. Since this project is mostly about labeling, evaluation, and analysis rather than implementation, AI tools are most useful for stress-testing the taxonomy, assisting annotation, and analyzing model failures.

### Label Stress-Testing

Before labeling all 200 examples, I will give an AI tool my label definitions and edge-case rules. I will ask it to generate 5–10 borderline NBA-style comments that sit between two labels.

I will then manually classify those examples.

If I cannot classify the generated examples cleanly, I will revise my definitions before annotating the full dataset. This will help prevent inconsistent labels.

Example prompt:

> Here are my labels for r/nba comments: analysis, hot_take, and reaction. Generate 10 borderline comments that are difficult to classify between two labels. Then explain which two labels each comment is ambiguous between.

I will not include these AI-generated examples in my training dataset. They are only for testing the clarity of my taxonomy.

### Annotation Assistance

I may use an LLM to pre-label some examples, but every final label will be manually reviewed by me. The LLM will not be treated as the source of truth.

If I use LLM pre-labeling, I will track it in the dataset with an extra column such as:

```csv
text,label,llm_suggested_label,reviewed_by_human
```

This will make the AI assistance transparent. The final `label` column will always represent my reviewed human decision.

If LLM suggestions disagree with my decision, I will keep my manual label and use the disagreement as a signal to check whether the example is ambiguous.

### Failure Analysis

After model evaluation, I will collect the model’s incorrect predictions and give them to an AI tool to look for patterns.

I will ask the AI to identify common failure types, such as:

* Confusing `analysis` with `hot_take`
* Treating any statistic as analysis
* Misclassifying sarcastic reactions
* Overpredicting the most common label
* Failing on short comments
* Failing on comments with basketball slang

I will verify these patterns myself by reading the actual wrong predictions. The AI tool will help organize possible explanations, but I will not accept its analysis without checking the examples manually.

## Stretch Feature Plan: Error Pattern Analysis

For my stretch feature, I will perform a systematic error pattern analysis on the fine-tuned model's wrong predictions.

I will not only list individual errors. I will look for repeated patterns in the confusion matrix and wrong predictions, including:

- which true labels are most often misclassified
- which predicted labels they are confused with
- whether mistakes are caused by short comments, sarcasm, emotional wording, weak evidence, or ambiguous basketball claims
- whether the issue appears to be label design, annotation consistency, or model limitations

I will use the wrong predictions from the test set and compare them against the confusion matrix. I will also use an AI tool to suggest possible patterns, then manually verify whether those patterns are supported by the actual examples.