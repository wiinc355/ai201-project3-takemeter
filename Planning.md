# TakeMeter Planning Document — Hacker News Classification

## 1. Community

**Community chosen:** Hacker News (news.ycombinator.com)

**Why this community?**
Hacker News is a technology-focused discussion platform with a highly engaged community of developers, entrepreneurs, and technology enthusiasts. The discourse is text-heavy, substantive, and genuinely varied in quality and intent — users post everything from direct announcements of new projects, to critiques and analyses of technology trends, to open questions seeking community advice. This variation makes it an ideal candidate for a classification task. The community is large enough to collect 200+ examples without difficulty, the boundaries between post types are real and meaningful (not arbitrary), and the classification task directly reflects how Hacker News users themselves think about content quality and engagement.

**Why these distinctions matter:** On Hacker News, different post types serve different functions. A reader approaching the site might ask: "What new projects/releases/research should I know about?" (news), "What are the thoughtful takes on tech trends?" (analysis), "What's the community debating right now?" (discussion), or "Where can I get specific advice?" (ask_community). A classifier that distinguishes these types could help users find the content they're looking for and help community moderators understand post quality and intent.

---

## 2. Labels

### Label 1: news

Definition: Posts primarily sharing a new article, announcement, release, research paper, company update, or event, where the main goal is informing readers.

Example Posts:
- "SQLite 4.0 Released with Experimental MVCC Support"
- "Google Announces New Open-Source AI Model"

Uncertain Case:
- "Why Rust Is Replacing C in Systems Programming" — Is this news because it links to a recent article, or analysis because it mainly presents an argument? **Decision rule:** If the title is reporting an event or release (e.g., "Rust adopted by CISA"), it's news. If the title is making an interpretive claim about trends ("Rust is *replacing*"), it's analysis.

---

### Label 2: analysis

Definition: Posts that provide interpretation, explanation, technical deep-dives, opinions, or critiques rather than simply reporting information.

Example Posts:
- "Lessons Learned from Running PostgreSQL at Scale"
- "A Detailed Breakdown of Apple's New Privacy Architecture"

Uncertain Case:
- "OpenAI's Latest Model Benchmarks Explained" — Could be news if focused on announcing the model, or analysis if focused on interpreting results. **Decision rule:** If the post primarily reports what was released/announced, it's news. If the post primarily explains what the benchmark results mean or compares them to prior work, it's analysis.

---

### Label 3: discussion

Definition: Posts intended to spark open-ended conversation, debate, or community reflection on a topic rather than answer a specific question.

Example Posts:
- "What Programming Trend Do You Think Is Overhyped?"
- "Will Remote Work Remain the Default for Engineers?"

Uncertain Case:
- "The Future of Software Engineering in an AI World" — Could be discussion if asking for opinions, or analysis if presenting a strong thesis. **Decision rule:** If the post is framed as a question or explicitly invites multiple viewpoints ("what do you think?"), it's discussion. If the post presents a specific argument or interpretation ("here's why..."), it's analysis.

---

### Label 4: ask_community

Definition: Posts where the author is explicitly seeking advice, recommendations, experiences, or answers from the Hacker News community.

Example Posts:
- "Ask HN: Best Resources for Learning Distributed Systems?"
- "Ask HN: How Do You Manage Burnout as a Developer?"

Uncertain Case:
- "Ask HN: Is AI Replacing Junior Developers?" — Could be ask_community if seeking opinions, but may also function as a broad discussion thread. **Decision rule:** The "Ask HN:" prefix means it's ask_community, even if the actual content reads more like a discussion. The marker indicates explicit intent to solicit community input.

---

## 3. Hard Edge Cases & Decision Rules

To reduce overlap, use the following decision hierarchy:

1. **ask_community** → Any post explicitly asking a question or requesting advice (especially those with "Ask HN:" prefix) belongs here, even if it starts a discussion.
2. **discussion** → Opinion-oriented prompts seeking conversation but not direct answers (no "Ask HN:" prefix, not reporting news, not explaining something).
3. **analysis** → Posts whose primary content is interpretation, explanation, critique, or deep technical examination.
4. **news** → Posts primarily sharing new information, announcements, releases, research, or events.

**Classification test cases:**
- "Ask HN: What laptop should I buy?" → ask_community ✓
- "Why ARM Laptops Are Winning" → analysis ✓
- "Are Laptops Becoming Too Expensive?" → discussion ✓
- "Apple Releases New MacBook Pro" → news ✓

These rules make it possible to assign most Hacker News posts to exactly one label.

---

## 4. Data Collection Plan

**Source:** Hacker News (news.ycombinator.com) — specifically the front page, best, and new sections.

**Collection method:** Manual collection of 200 posts from various timeframes (past 6 months to ensure diversity of topics and events).

**Target distribution:** 50 posts per label (balanced dataset) — this was selected to avoid heavily skewed label distributions that would bias the model during training.

**How collected:** Posts were manually selected and labeled to represent realistic examples of each category as they appear on Hacker News.

**Handling underrepresentation:** The dataset already achieves perfect balance (50/50/50/50). If this had not been the case, additional collection would have focused on underrepresented labels until reaching at least 40 examples per label (80% of the target minimum).

**Data already collected:** ✅ 200 examples collected and annotated (50 per label).

---

## 5. Evaluation Metrics

**Metrics chosen:**
1. **Accuracy** — overall percentage of correct predictions across all labels
2. **Per-class precision, recall, and F1-score** — to detect if the model performs better on some labels than others
3. **Confusion matrix** — to identify which labels are frequently confused with each other
4. **Baseline comparison** — zero-shot Groq classification vs. fine-tuned DistilBERT

**Why these metrics?**

- **Accuracy alone is insufficient** because a balanced dataset can mask poor performance on minority labels. With 4 balanced labels, accuracy is a reasonable high-level metric, but per-class metrics reveal whether the model struggles with specific types.
  
- **Precision vs. recall tradeoffs matter here:** If this classifier were deployed as a content moderation tool, false positives (mislabeling a discussion post as ask_community) might be less costly than false negatives (failing to identify actual ask_community posts). Per-class metrics help identify these tradeoffs.
  
- **Confusion matrix** is critical for error analysis — it shows which label-pairs are commonly confused, which directly points to where label definitions or boundaries need clarification.
  
- **Baseline comparison** is essential because accuracy is meaningless without context. A 90% accuracy is excellent if the baseline is 25% (random) but concerning if the baseline is 95% (a simpler heuristic works better).

**Success threshold:** See section 6 below.

---

## 6. Definition of Success

**Performance thresholds:**

- **Acceptable (minimum):** Accuracy ≥ 75% on test set, with each label achieving at least 70% recall. This would demonstrate that the fine-tuned model beats random guessing (25%) by a meaningful margin and can reliably identify each label type.

- **Good:** Accuracy ≥ 80%, all labels ≥ 75% recall. This would indicate robust performance across all categories.

- **Excellent:** Accuracy ≥ 85%, all labels ≥ 80% recall. This would represent a classifier ready for real-world deployment or community tool integration.

**Baseline comparison:** The fine-tuned model must outperform the zero-shot Groq baseline by at least 5 percentage points on accuracy. If Groq baseline achieves 85% accuracy, fine-tuning must reach 90% to be considered worthwhile (demonstrating that the annotation effort produced value).

**Real-world usefulness criterion:** The classifier should achieve ≥ 80% accuracy overall. Below 80%, the error rate is high enough to undermine user trust in automated categorization; above 80%, the tool becomes practical for a real community feature.

---

## 7. AI Tool Plan

### Label Stress-Testing

**What:** Use an LLM to generate 5–10 posts that sit at the boundary between two labels (e.g., a post that could be news or analysis). Then manually try to classify each one using the definitions above.

**Tool:** GPT-4 or Claude with the label definitions and edge cases provided.

**Execution:** After dataset collection, before annotation begins. If the LLM generates posts you struggle to classify cleanly, it's a signal that the definitions need refinement NOW (before annotating 200 examples and discovering the same ambiguity repeatedly).

**Success criterion:** You can classify at least 8 of the 10 generated posts cleanly using the decision rules above. If fewer, revise label definitions and try again.

### Annotation Assistance

**Decision:** Will NOT use LLM pre-labeling for this project. Reason: The dataset is already labeled (200 examples collected). If this were a larger collection phase, would consider using Claude to pre-label a sample with explicit disclosure of which examples were pre-labeled vs. manual.

**Alternative use:** Will use LLMs to validate edge cases during annotation — if a post seems ambiguous, ask Claude to classify it using the prompt and see if it agrees with manual judgment. This helps identify whether the definitions are clear enough.

### Failure Analysis

**Plan:** After fine-tuning completes and test predictions are generated, extract the 15–20 wrong predictions. Provide them to Claude along with the label definitions and ask it to:
1. Identify patterns in why the model failed (e.g., "mislabels ask_community as discussion when the post doesn't have 'Ask HN:' prefix")
2. Suggest whether the pattern reflects a label definition gap or model limitation
3. Propose refinements

**Verification:** Review Claude's pattern identification against your own manual inspection of the wrong predictions. Look for genuine patterns (not spurious correlations) before writing up findings in your evaluation report.

---

## Summary

This plan ensures that label definitions are sharp, data collection is systematic, and evaluation criteria are objective and measurable. The AI tool usage is focused on validation and error analysis rather than replacing manual judgment.
