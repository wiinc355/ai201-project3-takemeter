Community: Hacker News
Label 1: news

Definition: Posts primarily sharing a new article, announcement, release, research paper, company update, or event, where the main goal is informing readers.

Example Posts:

"SQLite 4.0 Released with Experimental MVCC Support"
"Google Announces New Open-Source AI Model"

Uncertain Case:

"Why Rust Is Replacing C in Systems Programming"
Is this news because it links to a recent article, or analysis because it mainly presents an argument?
Label 2: analysis

Definition: Posts that provide interpretation, explanation, technical deep-dives, opinions, or critiques rather than simply reporting information.

Example Posts:

"Lessons Learned from Running PostgreSQL at Scale"
"A Detailed Breakdown of Apple's New Privacy Architecture"

Uncertain Case:

"OpenAI's Latest Model Benchmarks Explained"
Could be news if focused on announcing the model, or analysis if focused on interpreting results.
Label 3: discussion

Definition: Posts intended to spark open-ended conversation, debate, or community reflection on a topic rather than answer a specific question.

Example Posts:

"What Programming Trend Do You Think Is Overhyped?"
"Will Remote Work Remain the Default for Engineers?"

Uncertain Case:

"The Future of Software Engineering in an AI World"
Could be discussion if asking for opinions, or analysis if presenting a strong thesis.
Label 4: ask_community

Definition: Posts where the author is explicitly seeking advice, recommendations, experiences, or answers from the Hacker News community.

Example Posts:

"Ask HN: Best Resources for Learning Distributed Systems?"
"Ask HN: How Do You Manage Burnout as a Developer?"

Uncertain Case:

"Ask HN: Is AI Replacing Junior Developers?"
Could be ask_community if seeking opinions, but may also function as a broad discussion thread.
Mutual Exclusivity Check

To reduce overlap, use the following decision rules:

ask_community → Any post explicitly asking a question or requesting advice belongs here, even if it starts a discussion.
discussion → Opinion-oriented prompts seeking conversation but not direct answers.
analysis → Posts whose primary content is interpretation, explanation, critique, or deep technical examination.
news → Posts primarily sharing new information, announcements, releases, research, or events.

A good test:

"Ask HN: What laptop should I buy?" → ask_community
"Why ARM Laptops Are Winning" → analysis
"Are Laptops Becoming Too Expensive?" → discussion
"Apple Releases New MacBook Pro" → news

These rules make it possible to assign most Hacker News posts to exactly one label.

Planning.md Description

Hacker News is a technology-focused community where users share industry news, technical articles, startup stories, research, and questions for fellow developers, entrepreneurs, and technology enthusiasts. The dataset uses four labels: news (information sharing), analysis (interpretation and deep dives), discussion (conversation starters), and ask_community (requests for advice or answers). These distinctions matter because users engage with content differently depending on whether they are learning about an event, evaluating an argument, debating an issue, or seeking help from the community.