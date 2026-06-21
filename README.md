# AI201 Project 3: r/nba Discourse Classification

## Project Summary

This project builds a text classifier for public Reddit comments from `r/nba`. The goal is to classify NBA discussion comments into three discourse labels:

* `analysis`
* `hot_take`
* `reaction`

The task is not to decide whether a basketball opinion is correct. Instead, the classifier tries to distinguish between comments that explain basketball reasoning, comments that make unsupported confident claims, and comments that mainly express emotion.

I evaluated two approaches:

1. A zero-shot Groq baseline using `llama-3.3-70b-versatile`
2. A fine-tuned `distilbert-base-uncased` model

The final fine-tuned model improved over the zero-shot baseline by 10 percentage points, but it still struggled with the `hot_take` label.

## Community

I chose `r/nba`, a public Reddit community focused on NBA discussion. This community is a good fit because NBA discourse contains a wide range of comment types. Some comments use statistics, tactical reasoning, matchup context, or historical comparison. Other comments are short emotional reactions to games, highlights, trades, injuries, referees, or player performances. A third group consists of confident but weakly supported opinions, which are common in sports debate.

The unit of analysis is an individual Reddit comment.

## Labels

### `analysis`

A comment is labeled `analysis` when it makes a structured basketball argument supported by specific evidence, statistics, tactical reasoning, historical comparison, matchup context, or clear cause-and-effect explanation.

Example:

> The Wolves’ defense worked because they kept sending early help from the weak side and forced Denver’s role players to make quick decisions.

### `hot_take`

A comment is labeled `hot_take` when it makes a bold, confident, or controversial basketball claim without enough supporting evidence. The claim may be true, but the comment mostly asserts instead of argues.

Example:

> Tatum is never winning as the best player on a title team.

### `reaction`

A comment is labeled `reaction` when it mainly expresses emotion, surprise, frustration, hype, sarcasm, or disappointment in response to a basketball event, with little or no argument.

Example:

> I can’t believe they blew that lead again. This team is impossible to watch.

## Dataset

I collected 200 public Reddit comments from `r/nba` and saved them in a single CSV file.

The CSV columns are:

| Column       | Meaning                                            |
| ------------ | -------------------------------------------------- |
| `text`       | Reddit comment text                                |
| `label`      | Final assigned label                               |
| `notes`      | Annotation notes for difficult or borderline cases |
| `source_url` | Source Reddit thread URL                           |

The final label distribution was balanced:

| Label      |   Count |
| ---------- | ------: |
| `analysis` |      67 |
| `hot_take` |      67 |
| `reaction` |      66 |
| **Total**  | **200** |

The notebook automatically split the dataset into train, validation, and test sets.

## Model Setup

I evaluated two models.

| Model                   | Description                                                  |
| ----------------------- | ------------------------------------------------------------ |
| Zero-shot Groq baseline | `llama-3.3-70b-versatile` prompted with my label definitions |
| Fine-tuned DistilBERT   | `distilbert-base-uncased` fine-tuned on the training split   |

The original notebook default used 3 epochs. My first fine-tuned run collapsed the `reaction` class and never predicted it. To fix that, I reran fine-tuning with 6 epochs while keeping the learning rate and batch size unchanged.

Final fine-tuning settings:

| Setting       | Value                     |
| ------------- | ------------------------- |
| Base model    | `distilbert-base-uncased` |
| Epochs        | 6                         |
| Learning rate | 2e-5                      |
| Batch size    | 16                        |

## Evaluation Results

### Overall Accuracy

| Model                   | Accuracy |
| ----------------------- | -------: |
| Zero-shot Groq baseline |    0.500 |
| Fine-tuned DistilBERT   |    0.600 |
| Difference              |   +0.100 |

The fine-tuned DistilBERT model improved over the zero-shot Groq baseline by 10 percentage points.

### Baseline Per-Class Metrics

| Label            | Precision | Recall |   F1 | Support |
| ---------------- | --------: | -----: | ---: | ------: |
| `analysis`       |      1.00 |   0.30 | 0.46 |      10 |
| `hot_take`       |      1.00 |   0.20 | 0.33 |      10 |
| `reaction`       |      0.40 |   1.00 | 0.57 |      10 |
| **accuracy**     |           |        | 0.50 |      30 |
| **macro avg**    |      0.80 |   0.50 | 0.46 |      30 |
| **weighted avg** |      0.80 |   0.50 | 0.46 |      30 |

The baseline was conservative when predicting `analysis` and `hot_take`, but it overpredicted `reaction`. It found all 10 true reaction examples, but many non-reaction examples were also pushed into that category.

### Fine-Tuned Per-Class Metrics

| Label            | Precision | Recall |   F1 | Support |
| ---------------- | --------: | -----: | ---: | ------: |
| `analysis`       |      0.62 |   0.80 | 0.70 |      10 |
| `hot_take`       |      0.50 |   0.10 | 0.17 |      10 |
| `reaction`       |      0.60 |   0.90 | 0.72 |      10 |
| **accuracy**     |           |        | 0.60 |      30 |
| **macro avg**    |      0.57 |   0.60 | 0.53 |      30 |
| **weighted avg** |      0.57 |   0.60 | 0.53 |      30 |

The fine-tuned model improved overall accuracy and macro F1. It performed best on `reaction` and reasonably well on `analysis`, but it performed poorly on `hot_take`.

### Fine-Tuned Confusion Matrix

Rows are true labels. Columns are predicted labels.

| True \ Predicted | `analysis` | `hot_take` | `reaction` |
| ---------------- | ---------: | ---------: | ---------: |
| `analysis`       |          8 |          1 |          1 |
| `hot_take`       |          4 |          1 |          5 |
| `reaction`       |          1 |          0 |          9 |

## Evaluation Analysis

The fine-tuned model improved over the zero-shot baseline, but the improvement was uneven across labels. The model correctly classified 8 out of 10 `analysis` examples and 9 out of 10 `reaction` examples. This suggests that the model learned the two clearer ends of the taxonomy: structured basketball reasoning and emotional response.

The weakest label was `hot_take`. The model only correctly classified 1 out of 10 hot take examples. Four true `hot_take` examples were predicted as `analysis`, and five true `hot_take` examples were predicted as `reaction`.

This shows that `hot_take` is the hardest middle category in the taxonomy. Some hot takes look like analysis because they contain basketball claims, player names, concrete numbers, or strategy language. Other hot takes look like reactions because they use emotional, sarcastic, or exaggerated wording. The model learned the clearer endpoints of the taxonomy, but it did not learn the unsupported-opinion boundary very well.

## Failure Analysis

Before writing this analysis, I used an AI tool to help identify patterns in the wrong predictions. I provided misclassified examples, their true labels, predicted labels, and confidence scores. The tool suggested that the largest pattern was confusion around `hot_take`.

I manually checked this against the confusion matrix and the wrong examples. That pattern was accurate: 9 of the 12 total fine-tuned errors involved true `hot_take` examples. The model usually pushed hot takes toward either `analysis` or `reaction`.

### Wrong Prediction 1

Text:

> Man you cannot have 6 rebounds in 37 mins as a fucking 7 foot 5 center.

True label: `hot_take`

Predicted label: `analysis`

Confidence: 0.4517

Analysis:

This example was likely predicted as `analysis` because it includes concrete basketball details: rebounds, minutes, height, and position. The model appears to have treated those details as evidence. However, the comment does not build a real argument. It uses one stat-like observation to make a strong criticism without explaining context, role, matchup, defensive responsibilities, or why that rebound total happened. Under my labeling rules, evidence without reasoning is still a `hot_take`.

### Wrong Prediction 2

Text:

> He needs to grab the game by the balls are conquer it

True label: `hot_take`

Predicted label: `reaction`

Confidence: 0.4165

Analysis:

This example was likely predicted as `reaction` because the wording is emotional and exaggerated. However, the comment is not only expressing a feeling. It is making a confident claim about what the player needs to do. It does not provide tactical reasoning or evidence, so it fits `hot_take` better than `analysis`. Because it is an unsupported judgment rather than only an emotional response, the final label is `hot_take`.

### Wrong Prediction 3

Text:

> We’ve been fouling the shit out of each other ALL series and now the refs woke up and decided to actually call it all

True label: `analysis`

Predicted label: `hot_take`

Confidence: 0.3660

Analysis:

This example was probably misclassified because of the informal and frustrated tone. The phrase "fouling the shit out of each other" sounds emotional, which makes it look similar to a hot take. However, the comment is making a specific explanation about referee behavior across the series: physical play had been happening throughout, but the officials changed how they called the game. That is a cause-and-effect interpretation of the game context, so I labeled it `analysis`.

### Wrong Prediction 4

Text:

> imagine Wemby with Kyrie's bag

True label: `hot_take`

Predicted label: `reaction`

Confidence: 0.5131

Analysis:

This example is short and low-context, which likely made it hard for the model. It sounds like a quick reaction because it is phrased casually and does not explain much. However, it implies a speculative basketball claim about Wembanyama having Kyrie Irving's handle and shot-creation skill set. That makes it more than a pure emotional reaction. Because it is a confident hypothetical claim without support, I labeled it `hot_take`.

## Sample Classifications

| Comment                                                                                                                                                                                                                     | True Label | Predicted Label | Confidence | Correct? |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------- | --------------- | ---------: | -------- |
| Man you cannot have 6 rebounds in 37 mins as a fucking 7 foot 5 center.                                                                                                                                                     | `hot_take` | `analysis`      |     0.4517 | No       |
| There was that one possession where he lost his dribble, 4 guys hit the floor and Keldon Johnson came up with it miraculously. He gave it back to Castle who literally immediately threw it right to the Thunder again lmao | `analysis` | `analysis`      |     0.5553 | Yes      |
| Don't worry, the Lakers will try to make a trade for Dort this offseason. I can see it coming.                                                                                                                              | `hot_take` | `analysis`      |     0.4998 | No       |
| Thats most 7 game series, 2016 finals was majority shitty games, but remembered for 2-3 classics. but I agree something about this seems worse than usual                                                                   | `analysis` | `analysis`      |     0.5144 | Yes      |
| God I hate Tony Brothers.                                                                                                                                                                                                   | `reaction` | `reaction`      |     0.4624 | Yes      |

One correct prediction that makes sense is:

> God I hate Tony Brothers.

The model predicted `reaction`, which is reasonable because the comment is mainly an emotional response. It does not make a structured basketball argument or an unsupported basketball claim. It is mostly frustration directed at a referee, so it fits the `reaction` label.

Another reasonable correct prediction is:

> There was that one possession where he lost his dribble, 4 guys hit the floor and Keldon Johnson came up with it miraculously. He gave it back to Castle who literally immediately threw it right to the Thunder again lmao

The model predicted `analysis`, which is reasonable because the comment describes a specific sequence of play and explains how the possession unfolded. Even though the tone is casual, the comment contains concrete game context.

## Reflection: Intended vs. Learned Behavior

The intended classifier was supposed to separate three discourse types: structured analysis, unsupported confident claims, and emotional reactions. The final model partially captured this structure. It learned to identify many `analysis` examples and most `reaction` examples, which suggests it picked up on visible signals such as reasoning language, play descriptions, emotional wording, and short reaction-style phrasing.

However, the model did not capture the `hot_take` label well. Instead of learning `hot_take` as its own category, it treated hot takes as a mixture of the other two labels. If a hot take sounded like a basketball claim, the model often moved it toward `analysis`. If it sounded exaggerated, sarcastic, or emotional, the model often moved it toward `reaction`.

This shows a gap between my label definitions and the model’s learned decision boundary. My definition of `hot_take` depends on whether a claim is sufficiently supported. That requires judging the quality of reasoning, not just detecting topic words or emotional tone. With only 200 examples, the model did not learn that boundary reliably.

To improve the model, I would collect more `hot_take` examples and include more borderline cases where hot takes look similar to analysis or reaction. I would also tighten the annotation rules so the difference between emotional hot takes and pure reactions is clearer. For example, I would add more explicit rules for short speculative comments, sarcastic referee comments, and comments that contain numbers but do not actually explain anything.

## Spec Reflection

One way the spec helped was by forcing me to define the labels before collecting the dataset. This made annotation more consistent because I already had decision rules for difficult cases, especially comments that included one statistic but did not explain it.

One way the implementation diverged from the original spec was the training setup. I originally planned to use the notebook defaults, including 3 epochs. After the first fine-tuned run failed to predict the `reaction` class, I changed the training to 6 epochs. This change was necessary because the default training setup did not learn all three labels well enough. The 6-epoch run produced a better final result and improved over the baseline.

## AI Usage

I used AI tools in several parts of this project.

First, I used AI to help draft and stress-test the label taxonomy. I asked for examples of comments that could sit between `analysis`, `hot_take`, and `reaction`. This helped me clarify the decision rule that evidence without explanation should usually be labeled `hot_take`, while evidence connected to a broader argument should be labeled `analysis`.

I also used AI during evaluation to identify patterns in wrong predictions. I provided the model with misclassified examples, true labels, predicted labels, and confidence scores. The AI suggested that the main error pattern was confusion around `hot_take`. I manually verified that pattern using the confusion matrix and the wrong examples before including it in this report.

## Stretch Feature: Error Pattern Analysis

For the stretch feature, I performed a systematic error pattern analysis instead of only listing individual wrong predictions.

The main pattern was confusion around the `hot_take` label. Out of 12 total fine-tuned errors, 9 involved true `hot_take` examples. The model classified 4 hot takes as `analysis` and 5 hot takes as `reaction`.

This shows that `hot_take` was not learned as a stable middle category. When a hot take contained basketball-specific details, the model often treated it as `analysis`. When a hot take used emotional, sarcastic, or exaggerated language, the model often treated it as `reaction`.

This pattern suggests that the model learned surface cues better than the actual label rule. It learned that reasoning-like language often means `analysis` and emotional language often means `reaction`, but it struggled to identify unsupported claims that sit between those two labels.

To improve this, I would collect more borderline `hot_take` examples and add clearer annotation rules for three cases:

1. comments with numbers but no real explanation
2. sarcastic comments that imply a basketball claim
3. short speculative comments about players, trades, or coaching

