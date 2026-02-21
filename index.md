---
layout: default
title: "Workshop: LLM-Powered Survey Coding with Claude"
nav_order: 1
---

# LLM-Powered Survey Coding with Claude
{: .no_toc }

## A Hands-On Workshop for Social Scientists
{: .no_toc }

<div class="code-example" markdown="1">
**Workshop Duration:** 2.5 hours  
**Level:** Intermediate R users  
**Tools:** R, Claude API (Haiku 4.5)
</div>

---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## What You'll Build

In this workshop, you'll learn to use Large Language Models (LLMs) to code open-ended survey responses—transforming a task that would take weeks of human effort into a few hours of automated, consistent coding.

**Real-world example:** We'll work with 1,944 survey responses about racial attitudes, using Claude to apply a detailed 9-variable codebook. By the end, you'll have a production-ready system that:

- ✅ Codes survey responses consistently (temperature=0)
- ✅ Matches human inter-rater reliability (~90-95%)
- ✅ Handles errors and rate limits gracefully
- ✅ Saves progress incrementally (resumable if interrupted)
- ✅ Costs ~$2-3 per 1,000 responses

## What You'll Learn

- **When to use LLMs** for survey coding (and when not to)
- **How modern LLMs work** differently from previous classification models
- **API fundamentals** with Claude using R
- **Prompt engineering** for structured, reproducible coding
- **Cost optimization** through batching and model selection
- **Validation techniques** comparing AI with human coders

## Prerequisites

### Required Knowledge
- Intermediate R (tidyverse, functions, basic data manipulation)
- Familiarity with survey research and coding
- Basic understanding of APIs (we'll cover the specifics)

### Required Setup
- R (version 4.0+) with RStudio
- R packages: `tidyverse`, `httr2`, `jsonlite`
- Anthropic API key (free tier includes $5 credit)
- Sample survey data (provided)

### Recommended Reading
- Basic understanding of what LLMs are
- Familiarity with inter-rater reliability concepts

## Workshop Materials

All materials are available for download:

- 📊 **Sample data:** 1,944 survey responses on racial attitudes
- 📋 **Codebook:** 9-variable coding scheme
- 💻 **Complete R scripts:** From basic to production code
- 📚 **Slides:** Visual explanations of key concepts

## Cost Expectations

Using Claude Haiku 4.5:
- **Test run (100 responses):** ~$0.12
- **Full survey (1,944 responses):** ~$2.50
- **All 4 questions (~8,000 responses):** ~$10

The $5 free credit covers the entire workshop plus your own experiments!

## Workshop Schedule

| Time | Topic | Duration |
|------|-------|----------|
| 0:00 | Setup & Introduction | 15 min |
| 0:15 | Motivation: Why LLMs for Survey Coding? | 10 min |
| 0:25 | Your First API Call | 20 min |
| 0:45 | Structured Coding with a Codebook | 30 min |
| 1:15 | Efficient Batch Processing | 25 min |
| 1:40 | Break | 10 min |
| 1:50 | Building a Production System | 30 min |
| 2:20 | Validation & Comparison with Human Coders | 15 min |
| 2:35 | Next Steps & Discussion | 10 min |
| 2:45 | Q&A Buffer | 15 min |

## Learning Objectives

By the end of this workshop, you will be able to:

1. **Evaluate** when LLM-based coding is appropriate for your research
2. **Design** effective prompts for structured survey coding
3. **Implement** cost-efficient batch processing with error handling
4. **Validate** AI codes against human benchmarks
5. **Adapt** this approach to your own survey data and codebooks

## Getting Started

Ready to begin? Head to [Setup](setup.html) to install the necessary tools and get your API key.

---

## About This Workshop

This workshop was developed as a practical guide for social scientists who want to leverage modern AI for survey research. The approach emphasizes:

- **Reproducibility:** Temperature=0 for consistent results
- **Validation:** Always compare with human coding
- **Cost-efficiency:** Optimize batch sizes and model choice
- **Transparency:** Understand what the model is doing

The methods taught here can be adapted to many text classification tasks beyond survey coding.
