---
name: english-grammar-in-use-summary
description: Generates structured English Grammar in Use unit summary blog posts for a Jekyll Chirpy blog, including grammar rules, form tables, examples, and common mistakes.
metadata:
  labels: [english, grammar, english-grammar-in-use, study-notes]
triggers:
  keywords:
    - "english grammar in use"
    - "grammar in use"
    - "grammar unit"
    - "summarize unit"
    - "grammar summary"
---

# English Grammar in Use — Unit Summary Generator

## Overview

This skill generates **structured grammar unit summaries** based on the book  
**_English Grammar in Use_ by Raymond Murphy**.

To keep the number of posts manageable (there are 145 units in total), this skill **combines three consecutive units** into a single summary post.

Each summary is saved as a **Jekyll Chirpy blog post** in `_posts/english/english-grammar-in-use-book/`.

The generated post must:

* accurately reflect the grammar rules taught in the three units
* use clear, beginner-friendly language
* include form tables, example sentences, and common mistakes for each unit
* be useful for both study and future review
* follow the Jekyll Chirpy blog format

**Target length:** 1500–2500 words.

---

# Site Context

| Property            | Value                 |
| ------------------- | --------------------- |
| Jekyll theme        | `jekyll-theme-chirpy` |
| Post directory      | `_posts/english/english-grammar-in-use-book/`     |
| Default language    | English               |
| Table of contents   | Enabled globally      |
| Mermaid diagrams    | Supported             |
| Syntax highlighting | Rouge                 |

---

# Front Matter Template

Every generated post MUST begin with:

```yaml
---
title: "Units {Start_Unit}-{End_Unit}: {Combined Unit Topics} — English Grammar in Use"
description: "{2–3 sentence summary of the grammar points covered in these units.}"
date: YYYY-MM-DD 00:00:00
categories: [English, Grammar]
tags: [grammar, english-grammar-in-use, units-{Start_Unit}-{End_Unit}, {grammar-topic-tags}]
---
```

### Rules

**Title**

Follow this pattern exactly:

```
Units 5-7: Present Simple and Continuous — English Grammar in Use
```

**Description**

Summarize what grammar points are taught across the three units and what the reader will learn.

Example:

```
Learn when to use the Present Simple and Present Continuous tenses, their forms, and how they differ from each other.
```

**Tags**

* lowercase
* hyphen-separated
* always include `grammar`, `english-grammar-in-use`, and `units-{Start_Unit}-{End_Unit}`
* add relevant grammar-topic-specific tags

Example:

```
grammar
english-grammar-in-use
units-5-7
present-simple
present-continuous
tense
```

Do NOT include `layout: post`.

---

# Post Structure

Follow this exact section order.

---

## 1. Introduction

In 2–4 sentences:

* state the grammar topics of the combined units
* explain why they matter for English learners
* explicitly list the three units being covered

Keep it brief and motivating.

---

## 2. Unit {N}: {First Unit Topic}

*(For each of the three units, include the following subsections using `###`)*

### Grammar Rule

Explain the grammar rule clearly and concisely.

Prefer:

* short paragraphs
* bullet points
* plain language over linguistic jargon

### Form Table

Always include a **form table** showing the grammatical structure.

Example for Present Simple:

| Subject         | Positive          | Negative              | Question            |
| --------------- | ----------------- | --------------------- | ------------------- |
| I / You / We / They | work          | don't work            | Do you work?        |
| He / She / It   | works             | doesn't work          | Does she work?      |

### Key Examples

Provide **6–10 example sentences** that illustrate the grammar rule.
Group them by positive, negative, and question forms when relevant.

### When to Use

Explain the **specific situations** where this grammar rule applies.
Use a table when multiple use cases exist.

### Common Mistakes

Highlight the mistakes learners typically make with this grammar point.

Format:

```
❌ She don't like coffee.
✅ She doesn't like coffee.
```

### Comparison (if applicable)

Include when the unit contrasts two similar grammar points.

---

## 3. Unit {N+1}: {Second Unit Topic}

*(Repeat the subsections: Grammar Rule, Form Table, Key Examples, When to Use, Common Mistakes, Comparison)*

---

## 4. Unit {N+2}: {Third Unit Topic}

*(Repeat the subsections: Grammar Rule, Form Table, Key Examples, When to Use, Common Mistakes, Comparison)*

---

## 5. Quick Summary

End the post with a concise bullet-point recap for all three units.

Format:

```markdown
## 📝 Quick Summary

**Unit {N}:**
- Use [grammar point] to [main use case].
- Don't forget: [one key reminder].

**Unit {N+1}:**
- Use [grammar point] to [main use case].
- Don't forget: [one key reminder].

**Unit {N+2}:**
- Use [grammar point] to [main use case].
- Don't forget: [one key reminder].
```

---

# Writing Style

The post should be:

* **clear and simple** — written for intermediate English learners
* **example-driven** — examples first, theory second
* **concise** — no unnecessary padding
* **encouraging** — supportive tone suitable for self-study

Avoid heavy linguistic terminology. Prefer plain explanations.

---

# File Naming Convention

```
_posts/english/english-grammar-in-use-book/YYYY-MM-DD-grammar-in-use-units-{Start_Unit}-{End_Unit}.md
```

Example:

```
_posts/english/english-grammar-in-use-book/2026-03-09-grammar-in-use-units-5-7.md
```

Rules:

* lowercase
* kebab-case
* use today's date
* always include the combined unit numbers

---

# Generation Workflow

When generating a unit summary:

1. Identify the three consecutive unit numbers and their grammar topics
2. Research the grammar rules thoroughly for all three (if not already known)
3. Plan the form tables and example sentences for each unit
4. Write the post following the structure above, covering all three units
5. Validate:
   * correct front matter format, combining the three units
   * form tables are accurate and present for each unit
   * examples are grammatically correct
   * correct directory: `_posts/english/`
   * correct date and unit numbers in filename
