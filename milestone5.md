## Milestone 5 Fine-Tuning Results

I fine-tuned `distilbert-base-uncased` on the training split of my r/nba labeled comment dataset. I changed the notebook default training setting from 3 epochs to 6 epochs after the first fine-tuned run collapsed the `reaction` label and never predicted it. The learning rate stayed at 2e-5 and the batch size stayed at 16.

### Baseline vs Fine-Tuned Model

| Model | Accuracy |
|---|---:|
| Zero-shot Groq baseline | 0.500 |
| Fine-tuned DistilBERT | 0.600 |
| Difference | +0.100 |

The fine-tuned model improved over the zero-shot baseline by 10 percentage points.

### Fine-Tuned Confusion Matrix

Rows are true labels. Columns are predicted labels.

| True \ Predicted | analysis | hot_take | reaction |
|---|---:|---:|---:|
| analysis | 8 | 1 | 1 |
| hot_take | 4 | 1 | 5 |
| reaction | 1 | 0 | 9 |

### Interpretation

The 6-epoch fine-tuned model performed better than the zero-shot Groq baseline. The biggest improvement was that the model learned to identify `reaction` comments, correctly classifying 9 out of 10 reaction examples in the test set. It also performed reasonably well on `analysis`, correctly classifying 8 out of 10 analysis examples.

The weakest label was `hot_take`. The model only correctly classified 1 out of 10 hot take examples. It split the remaining hot takes between `analysis` and `reaction`, which suggests that hot takes are the hardest middle category in this taxonomy. Some hot takes look analytical because they contain basketball claims, while others look like reactions because they use emotional or exaggerated language.

This result suggests that fine-tuning helped overall, but the label boundary between `hot_take` and the other two classes needs more examples or clearer annotation rules.