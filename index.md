---
layout: default
title: "Workshop: Introduction to Survey Coding with LLMs"
nav_order: 1
---

# Introduction to Survey Coding with LLMs
{: .no_toc }

## A  Workshop for Social Scientists
{: .no_toc }


---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## What You'll Build

In this workshop, you'll learn to use Large Language Models (LLMs) to code open-ended survey.

**Real-world example:** We'll work with survey responses about racial attitudes from an article published in 2018 in the _Journal of Politics_, using Claude to apply a detailed 9-variable codebook. 


## What You'll Learn

- **When to use LLMs** for survey coding (and when not to)
- **How modern LLMs work** differently from previous classification models
- **API fundamentals** with Claude using R
- **Prompt engineering** for structured, reproducible coding
- **Cost optimization** through batching and model selection
- **Validation techniques** comparing AI with human coders

## Prerequisites

### Required Setup
- R (version 4.0+) with RStudio
- R packages: `tidyverse`, `httr2`, `jsonlite`
- Anthropic API key (will be provided)
- Survey data (provided)

### Recommended Reading
- The article we're going to "replicate":
  Kam, Cindy D., and Camille D. Burge. 2018. "Uncovering Reactions to the Racial Resentment Scale across the Racial Divide." The _Journal of Politics_ 80 (1): 314–20. https://doi.org/10.1086/693907
- Some background reading on the (technical) history of LLMs -- The "History" section of [the Wikipedia article](https://en.wikipedia.org/wiki/Large_language_model) is as good a start as .

## Workshop Materials

All materials are available for download:

- 📊 **Sample data:** 1,944 survey responses on racial attitudes
- 📋 **Codebook:** 9-variable coding scheme
- 💻 **Complete R scripts:** From basic to production code


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

1. **Understand** how LLMs 'code' textual data.
2. **Evaluate** when LLM-based coding is appropriate for your research
3. **Design** effective prompts for structured survey coding
4. **Implement** cost-efficient batch processing with error handling
5. **Validate** AI codes against human benchmarks
6. **Adapt** this approach to your own survey data and codebooks and beyond

## Getting Started

Ready to begin? Head to [Setup](setup.html) to install the necessary tools and get your API key.

