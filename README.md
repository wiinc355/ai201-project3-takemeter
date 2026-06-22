# AI201 Project 3 - TakeMeter (Hacker News)

## Project Summary

This project classifies Hacker News posts into four discourse types:

- `news`
- `analysis`
- `discussion`
- `ask_community`

I fine-tuned `distilbert-base-uncased` on a manually labeled dataset of 200 Hacker News posts and compared it against a zero-shot Groq baseline on the locked test set. The fine-tuned model reached **0.8000** accuracy, but the zero-shot baseline reached **0.8333**, so fine-tuning did **not** outperform the baseline on this dataset.

## Community and Label Design

**Community:** Hacker News

These labels were chosen to separate the main kinds of intent visible in Hacker News titles:

| Label | Definition | Example 1 | Example 2 |
| --- | --- | --- | --- |
| `news` | Shares new information such as a release, announcement, paper, company update, or event. | `SQLite 4.0 Released with Experimental MVCC Support` | `Google Announces New Open-Source AI Model` |
| `analysis` | Interprets, critiques, explains, or breaks down a topic. | `Lessons Learned from Running PostgreSQL at Scale` | `A Detailed Breakdown of Apple's New Privacy Architecture` |
| `discussion` | Invites open-ended debate or reflection without explicitly asking for advice. | `What Programming Trend Do You Think Is Overhyped?` | `Will Remote Work Remain the Default for Engineers?` |
| `ask_community` | Explicitly asks the community for advice, recommendations, experiences, or answers. | `Ask HN: Best Resources for Learning Distributed Systems?` | `Ask HN: How Do You Manage Burnout as a Developer?` |

### Edge-Case Rules

1. `Ask HN:` posts are always `ask_community`.
2. Question-shaped titles are not automatically `ask_community`; if they mainly invite debate, they belong in `discussion`.
3. Strong claims or opinions are `analysis` only if they function as explanation or critique rather than as prompts for conversation.
4. Straight reporting of an event or release is `news`.

## Dataset

- **Source:** Hacker News
- **Size:** 200 labeled examples
- **Distribution:** 50 examples per label
- **Input file:** `hacker_news_labeled_dataset.csv`

### Labeling Process

I collected titles from Hacker News and manually assigned one label to each post using the decision hierarchy from `planning.md`. The goal was to label the **primary intent** of the post title, not just the topic. I used balanced sampling to reach 50 examples per class, and when a title was ambiguous I compared it against the edge-case rules before assigning a final label.

### Split Strategy

| Split | Size | Percent |
| --- | ---: | ---: |
| Train | 140 | 70% |
| Validation | 30 | 15% |
| Test | 30 | 15% |

The split was stratified by label so each split kept a similar class distribution.

### Three Difficult-to-Label Examples

1. **`Did my old job only exist because of fraud?` -> `analysis`**  
   This is phrased as a question, but it reads more like an interpretive argument than an invitation for community advice or debate.
2. **`The brain was not designed for this much bad news` -> `discussion`**  
   The title makes a strong claim, but its likely function is to provoke conversation and reflection rather than present a technical breakdown.
3. **`Ask HN: What will AI coding look like when today's CS freshmen graduate?` -> `ask_community`**  
   This could look like discussion, but the `Ask HN:` prefix makes it a direct request for community input under the project rules.

## Model and Training Setup

- **Base model:** `distilbert-base-uncased`
- **Epochs:** 3
- **Learning rate:** 2e-5
- **Train batch size:** 16
- **Eval batch size:** 32
- **Weight decay:** 0.01
- **Warmup steps:** 50
- **Selection rule:** best checkpoint by validation accuracy
- **Checkpoint used for evaluation:** `takemeter-model/checkpoint-27`

I did **not** change the default notebook hyperparameters for the final run.

### Hyperparameter Decision

I kept the notebook default of **3 epochs** instead of increasing training time because the dataset is small (200 examples) and a longer run would be more likely to overfit than to add useful signal. Given that the class boundary problem was already concentrated in `discussion`, I preferred a conservative setup over chasing a slightly higher training score.

## Baseline Setup

The baseline used Groq with `llama-3.3-70b-versatile` in a zero-shot classification prompt. The prompt included the project label definitions and instructed the model to return only one exact label name.

### Baseline Prompt Used

```text
Classify each Hacker News post into exactly one label.

Use these label definitions:
- news: posts primarily sharing a new article, announcement, release, research paper, company update, or event, where the main goal is informing readers.
- analysis: posts that provide interpretation, explanation, technical deep-dives, opinions, or critiques rather than simply reporting information.
- discussion: posts intended to spark open-ended conversation, debate, or community reflection on a topic rather than answer a specific question.
- ask_community: posts where the author is explicitly seeking advice, recommendations, experiences, or answers from the Hacker News community.

Decision hierarchy:
1. ask_community -> any post explicitly asking a question or requesting advice belongs here; posts with "Ask HN:" always belong here.
2. discussion -> opinion-oriented prompts seeking conversation but not direct answers.
3. analysis -> interpretation, explanation, critique, or deep technical examination.
4. news -> primarily sharing new information, announcements, releases, research, or events.

Output only one of these exact labels and nothing else:
news
analysis
discussion
ask_community
```

Results were collected by running the prompt on every example in the locked test split, normalizing the returned label text, and then computing accuracy plus per-class metrics on the parseable responses. The official run achieved 30/30 coverage, so the baseline metrics are complete rather than partial.

The official baseline completed with full coverage:

- **Accuracy:** `0.8333`
- **Coverage:** `30 / 30`
- **Unparseable responses:** `0`
- **API failures:** `0`

## Evaluation Report

### Overall Accuracy

| Model | Accuracy |
| --- | ---: |
| Groq zero-shot baseline | 0.8333 |
| Fine-tuned DistilBERT | 0.8000 |

Fine-tuning produced a **-0.0333** regression relative to the baseline.

### Per-Class Metrics

| Label | Baseline P | Baseline R | Baseline F1 | Fine-tuned P | Fine-tuned R | Fine-tuned F1 |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| `news` | 0.89 | 1.00 | 0.94 | 1.00 | 1.00 | 1.00 |
| `analysis` | 0.75 | 0.86 | 0.80 | 0.64 | 1.00 | 0.78 |
| `discussion` | 0.80 | 0.50 | 0.62 | 1.00 | 0.25 | 0.40 |
| `ask_community` | 0.88 | 1.00 | 0.93 | 0.78 | 1.00 | 0.88 |

### Fine-Tuned Confusion Matrix

Rows are true labels and columns are predicted labels.

| True \\ Pred | `news` | `analysis` | `discussion` | `ask_community` |
| --- | ---: | ---: | ---: | ---: |
| `news` | 8 | 0 | 0 | 0 |
| `analysis` | 0 | 7 | 0 | 0 |
| `discussion` | 0 | 4 | 2 | 2 |
| `ask_community` | 0 | 0 | 0 | 7 |

Supplementary image: `confusion_matrix.png`

### What the Metrics Show

The fine-tuned model learned three classes fairly well:

- `news` was perfect.
- `analysis` had perfect recall, but lower precision because the model over-predicted it.
- `ask_community` had perfect recall, helped by the strong `Ask HN:` pattern.

The weak class was `discussion`. Its recall dropped to **0.25**, which means the model found only 2 of the 8 real discussion examples. Most errors flowed from `discussion` into `analysis`, with a smaller number flowing into `ask_community`. That directional pattern matters more than the overall 0.80 accuracy because it shows the model did not actually learn the boundary I cared about most.

## Error Analysis

Before writing this section, I used an AI reviewer to inspect the six wrong predictions and suggest patterns. Its strongest useful point was that the model was not making random mistakes: it was systematically collapsing `discussion` into `analysis` and `ask_community`. I verified that pattern manually against the confusion matrix and the examples below.

### Pattern Summary

1. **Question form pulled some `discussion` posts into `ask_community`.** Titles phrased like direct questions looked like requests for advice even when they were really invitations to debate.
2. **Opinionated or declarative titles pulled `discussion` into `analysis`.** The model learned tone and topic better than conversational intent.
3. **`discussion` behaved like a residual class.** The clearer classes (`news`, `analysis`, `ask_community`) each had stronger surface cues, so the ambiguous class lost.
4. **Confidence was low on both correct and incorrect predictions.** Most confidence scores were around 0.27 to 0.30, which suggests weak separation between classes on title-only inputs.

### Three Specific Wrong Predictions

#### 1. "Are newsletters replacing technical blogs?"

- **True:** `discussion`
- **Predicted:** `ask_community` (0.2724)

This is a good example of the hardest boundary in the taxonomy. The title is phrased as a question, which pushes the model toward `ask_community`, but the actual intent is broader debate rather than a request for actionable advice. This looks more like a **model/data problem** than a bad label: the rule was clear, but the training set likely did not contain enough question-shaped discussion examples that were not `Ask HN:` posts.

**What would help:** add more discussion examples that are phrased as broad questions and contrast them directly with genuine advice-seeking posts.

#### 2. "My Opinion on RL"

- **True:** `discussion`
- **Predicted:** `analysis` (0.2728)

This title contains an explicit opinion marker, and the model appears to have learned that opinion plus technical topic often means `analysis`. The problem is that the title does not promise a deep explanation or critique; it mostly signals a conversation starter. This is partly a **boundary-design issue** because `discussion` and `analysis` can both contain opinions, but it is also a **data issue** because the model did not learn that some opinionated titles are invitations to discuss rather than essays.

**What would help:** tighten the annotation guidance for opinion-driven posts and add more contrastive examples where similar wording maps to different labels based on intent.

#### 3. "Never Too Late"

- **True:** `discussion`
- **Predicted:** `analysis` (0.2677)

This is the least informative title in the error set. Without body text, the model has almost no useful signal about whether the post is news, analysis, or discussion. I would not treat this as strong evidence of a broken decision boundary by itself; it is more a **data-quality limitation** of title-only classification.

**What would help:** either include post body/context in the dataset or exclude ultra-short, context-poor titles from evaluation if the goal is discourse-intent classification rather than title-style classification.

### Additional Observations

- `"Is there still value in learning assembly language?"` failed for the same reason as the first example: question form looked like advice-seeking.
- `"PostGIS pull requests are just a bunch of AI bots"` and `"The notebooks Marie Curie filled with her research are still dangerous to touch"` show that bold claims and topical cues can overpower the intended conversational label.

## Sample Classifications from the Fine-Tuned Model

| Post | Predicted label | Confidence | Notes |
| --- | --- | ---: | --- |
| `Ask HN: How do you decide whether to rewrite a system?` | `ask_community` | 0.3048 | Correct. This is reasonable because the `Ask HN:` prefix is an explicit community-request marker in the label rules. |
| `Ocient Database Sandbox released` | `news` | 0.2736 | Correct. The title is a straightforward release announcement. |
| `Why distributed locks fail in boring systems` | `analysis` | 0.2732 | Correct. The title signals explanation and technical interpretation rather than reporting or advice-seeking. |
| `Are tech layoffs changing how developers think about loyalty?` | `discussion` | 0.2717 | Correct. It invites broad community reflection rather than a concrete answer. |
| `Are newsletters replacing technical blogs?` | `ask_community` | 0.2724 | Incorrect. The model over-weighted question form. |

## Reflection: What the Model Learned vs. What I Intended

I wanted the model to learn **intent**: whether a Hacker News title is informing, analyzing, debating, or asking for advice. What it actually learned was closer to a mixture of **surface format and topic cues**.

- It learned that `Ask HN:` strongly implies `ask_community`.
- It learned that release-style phrasing strongly implies `news`.
- It learned that explanatory or argumentative wording often implies `analysis`.

What it did **not** learn well was the intended definition of `discussion` as a separate intent category. Instead of modeling discussion as "open-ended conversation without explicit advice-seeking," it often treated discussion as whichever clearer class looked most similar on the surface. In practice, that means the model overfit to syntax like question marks and words like "opinion" instead of reliably separating conversation prompts from advice requests or critiques.

## Spec Reflection

### One Way the Spec Helped

The spec requirement to run and save a zero-shot baseline before fine-tuning was extremely useful. Without that comparison, an 0.8000 fine-tuned accuracy would have looked respectable on its own. The baseline showed that the harder question was not "does the model work at all?" but "did fine-tuning add value?" In this case, it did not.

### One Way My Implementation Diverged

The project instructions assumed a Google Colab-first workflow, but I also adapted the notebook and evaluation flow to work in a local Python 3.12 environment with `GROQ_API_KEY` from the environment. I did this because local debugging was easier after notebook/runtime issues, but the final metrics and artifacts remained aligned with the notebook's intended logic and split.

## AI Usage

### 1. Failure-pattern analysis

I used an AI reviewer to analyze the six fine-tuned misclassifications and suggest common themes. It correctly identified the strongest real pattern: `discussion` was collapsing into `analysis` and `ask_community`, especially for question-shaped or opinion-shaped titles. I kept that conclusion because it matched the confusion matrix.

I overrode part of its output where it treated a few titles as definite annotation errors. After re-reading them myself, I think those cases are better described as **ambiguous or low-context titles**, not proven labeling mistakes.

### 2. Coding and evaluation support

I used GitHub Copilot CLI to inspect the notebook, rerun the saved checkpoint evaluation, and synchronize the report artifacts. It produced the updated metrics tables and JSON fields used in this README. I overrode an earlier stale README claim that the baseline was only a partial diagnostic run once the official 30/30 baseline had been recomputed and saved.

### Annotation disclosure

I did **not** use AI to pre-label the 200-example dataset. The labels in `hacker_news_labeled_dataset.csv` were manually assigned.

## Repository Artifacts

- `planning.md` - design decisions, label rules, and milestone planning
- `hacker_news_labeled_dataset.csv` - labeled dataset
- `colab_imported_notebook.ipynb` - working notebook
- `Copy of ai201_project3_takemeter_starter_clean.ipynb` - starter notebook copy
- `takemeter-model/checkpoint-27` - evaluated fine-tuned checkpoint
- `evaluation_results.json` - saved baseline and fine-tuned metrics
- `confusion_matrix.png` - fine-tuned confusion matrix image

## Demo Video

**Demo link:** `https://youtu.be/l_ye3iWKuG8'

The README analysis is complete. The remaining manual deliverable is to record the 3-5 minute demo showing:

1. 3-5 fine-tuned predictions with label and confidence visible
2. One correct prediction and why it is reasonable
3. One incorrect prediction and what went wrong
4. A brief walkthrough of this evaluation report

## Final Takeaway

This project produced a usable Hacker News discourse classifier, but it also exposed an important limitation: with only 200 title-only examples, the `discussion` label was too ambiguous to learn reliably. The most valuable outcome was not the final accuracy number by itself, but the discovery that the model's decision boundary for `discussion` is weaker than the zero-shot baseline and needs either clearer labeling rules, more contrastive examples, or richer post context.
