---
layout: default
title: "Your First API Call"
nav_order: 4
---

# Your First API Call
{: .no_toc }

## Coding Survey Responses with Claude
{: .no_toc }

<div class="code-example" markdown="1">
**Estimated time:** 20 minutes
</div>

---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Understanding the API

The Claude API follows a simple request-response pattern:

```
You → [Prompt] → Claude API → [Response] → You
```

**Key components:**
- **Model:** Which version of Claude to use
- **Messages:** Your prompt(s) to Claude
- **Max tokens:** Maximum length of response
- **Temperature:** Randomness level (more on this below)

## Choosing Your Model

Anthropic offers several Claude models. Here's how to choose:

| Model | Speed | Cost (per 1M tokens) | Best For |
|-------|-------|---------------------|----------|
| **Haiku 4.5** | Fastest | $0.25 in / $1.25 out | **← Use this one** |
| Sonnet 4.5 | Medium | $3 in / $15 out | Complex reasoning |
| Opus 4.5 | Slowest | $15 in / $75 out | Highest quality |

### Why We Use Haiku 4.5

For survey coding, Haiku 4.5 is ideal because:

✅ **Fast:** ~2 seconds per batch  
✅ **Cheap:** $2-3 for 1,944 responses  
✅ **Accurate enough:** 90-95% agreement with humans  
✅ **Large context:** 200K tokens (fits codebook + batches)

**When to consider alternatives:**
- **Sonnet 4.5:** If Haiku's accuracy isn't sufficient (test first!)
- **Opus 4.5:** Rarely needed for coding tasks
- **GPT-4o-mini:** Comparable alternative from OpenAI

### Model Identifiers

```r
# Model strings for API calls
"claude-haiku-4-5-20251001"    # ← We'll use this
"claude-sonnet-4-5-20250929"
"claude-opus-4-5-20251101"
```

The date suffix indicates the release version.

## Temperature: The Consistency Knob

Temperature controls randomness in model outputs:

```
Temperature 0.0 ━━━●━━━━━━━━━━━━━━━━━ 2.0
              Deterministic      Creative
```

**For research coding, always use `temperature = 0`:**

```r
# ✅ Correct: Same response → same code
temperature = 0

# ❌ Wrong: Same response → different codes
temperature = 1.0
```

**Why temperature=0?**
- 🎯 **Reproducibility:** Essential for research
- 📊 **Consistency:** Same input always gives same output
- 🔬 **Comparable:** Can meaningfully compare with human coders

**Other uses of temperature:**
- `0.3-0.7`: Slight variation in outputs
- `0.7-1.0`: Creative writing, brainstorming
- `1.0+`: Maximum diversity

{: .warning }
> **Critical:** Without `temperature=0`, your codes won't be reproducible. A response coded today might be coded differently tomorrow.

We tested this empirically: temperature 0 vs 1.0 gave ~94% vs ~91% consistency across runs. Small difference, but **always use 0 for research**.

## Building Our First Function

Let's create a simple function to call Claude:

```r
library(tidyverse)
library(httr2)
library(jsonlite)

call_claude <- function(prompt, 
                        model = "claude-haiku-4-5-20251001",
                        max_tokens = 1024,
                        temperature = 0,  # Always 0 for coding!
                        api_key = Sys.getenv("ANTHROPIC_API_KEY")) {
  
  response <- request("https://api.anthropic.com/v1/messages") |>
    req_headers(
      `x-api-key` = api_key,
      `anthropic-version` = "2023-06-01",
      `content-type` = "application/json"
    ) |>
    req_body_json(list(
      model = model,
      max_tokens = max_tokens,
      temperature = temperature,
      messages = list(
        list(
          role = "user",
          content = prompt
        )
      )
    )) |>
    req_perform()
  
  # Parse the JSON response
  result <- resp_body_string(response) |>
    fromJSON()
  
  # Extract just the text
  result$content[[1]]$text
}
```

**Test it:**

```r
response <- call_claude("What is 2 + 2?")
print(response)
```

## Exploratory Coding: First Survey Response

Let's code a real survey response! First, load the data:

```r
survey_data <- read_csv("data/Kam_Burge.csv")

# Get first non-empty Q43 response
first_response <- survey_data |>
  filter(!is.na(Q43), Q43 != "") |>
  slice(1) |>
  pull(Q43)

print(first_response)
```

**Output:** 
> "they are"

### Ask Claude to Analyze It

```r
prompt <- sprintf('
Analyze this survey response about racial attitudes:

"%s"

What themes do you see? Consider:
- Mentions of racial groups
- Attitudes about equality or discrimination
- References to individualism or hard work
- Emotional reactions

Provide a brief analysis.
', first_response)

analysis <- call_claude(prompt)
cat(analysis)
```

**Claude's response:**
> This response is extremely brief and lacks context... appears to be an incomplete thought... 
> Cannot be meaningfully coded without more information.

### Try a More Substantial Response

```r
# Get a longer response
substantial_response <- survey_data |>
  filter(!is.na(Q43), Q43 != "", nchar(Q43) > 50) |>
  slice(1) |>
  pull(Q43)

print(substantial_response)
```

**Example response:**
> "we are all struggling to be considered equal, so we should all expect equality in the chances we are given, with no 'special favors'"

### Analyze This One

```r
prompt <- sprintf('
Analyze this survey response about racial attitudes:

"%s"

Identify:
1. Are blacks explicitly mentioned?
2. Are whites explicitly mentioned?
3. What is the attitude toward individualism/equal treatment?
4. What is the attitude toward discrimination or special treatment?
5. Overall theme?

Be specific and quote from the text.
', substantial_response)

analysis <- call_claude(prompt, max_tokens = 500)
cat(analysis)
```

**Claude's analysis:**
> 1. Blacks mentioned: No, not explicitly
> 2. Whites mentioned: No, not explicitly
> 3. Individualism: Strong affirmation - "we should all expect equality in the chances we are given"
> 4. Special treatment: Opposed - explicit rejection of "special favors"
> 5. Theme: Colorblind individualism - emphasizes equal treatment without acknowledging racial groups

## Batch Analysis

Try analyzing multiple responses at once:

```r
# Get 5 responses
five_responses <- survey_data |>
  filter(!is.na(Q43), Q43 != "", nchar(Q43) > 30) |>
  slice(1:5) |>
  pull(Q43)

# Number them
numbered_responses <- paste0(
  seq_along(five_responses), ". ", five_responses,
  collapse = "\n\n"
)

prompt <- sprintf('
Analyze these survey responses about racial attitudes.
For each, identify the main theme in 1-2 sentences:

%s

Format: 
Response 1: [theme]
Response 2: [theme]
etc.
', numbered_responses)

analysis <- call_claude(prompt, max_tokens = 800)
cat(analysis)
```

## Understanding the Response Structure

The API returns JSON with this structure:

```json
{
  "content": [
    {
      "type": "text",
      "text": "Claude's response here..."
    }
  ],
  "usage": {
    "input_tokens": 145,
    "output_tokens": 87
  }
}
```

**Key fields:**
- `content[0].text`: The actual response
- `usage.input_tokens`: How many tokens you sent
- `usage.output_tokens`: How many tokens Claude generated

**Why this matters:**
- You pay per token: ~$0.25 per 1M input, $1.25 per 1M output
- Track usage to estimate costs

### Adding Usage Tracking

```r
call_claude_with_usage <- function(prompt, ...) {
  
  response <- request("https://api.anthropic.com/v1/messages") |>
    req_headers(
      `x-api-key` = Sys.getenv("ANTHROPIC_API_KEY"),
      `anthropic-version` = "2023-06-01",
      `content-type` = "application/json"
    ) |>
    req_body_json(list(
      model = "claude-haiku-4-5-20251001",
      max_tokens = 1024,
      temperature = 0,
      messages = list(list(role = "user", content = prompt))
    )) |>
    req_perform()
  
  result <- resp_body_string(response) |> fromJSON()
  
  # Return both text and usage
  list(
    text = result$content[[1]]$text,
    input_tokens = result$usage$input_tokens,
    output_tokens = result$usage$output_tokens,
    cost = (result$usage$input_tokens / 1e6 * 0.25) +
           (result$usage$output_tokens / 1e6 * 1.25)
  )
}

# Test it
result <- call_claude_with_usage("Analyze: 'they need to be just like every one else'")
cat(sprintf("Response: %s\n\nUsage: %d in, %d out\nCost: $%.4f\n",
            result$text, result$input_tokens, result$output_tokens, result$cost))
```

## Exercise: Explore Different Responses

Try these on your own:

**1. Find the longest response:**
```r
longest <- survey_data |>
  filter(!is.na(Q43), Q43 != "") |>
  arrange(desc(nchar(Q43))) |>
  slice(1) |>
  pull(Q43)

# Ask Claude to analyze it
```

**2. Find responses that mention "Obama":**
```r
obama_responses <- survey_data |>
  filter(str_detect(Q43, regex("obama", ignore_case = TRUE)))

# How does Claude interpret these?
```

**3. Compare very short vs. very long responses:**
```r
# Can Claude do anything useful with short responses?
# Do long responses provide more nuance?
```

## Limitations of Exploratory Coding

What we've done so far is **exploratory**—useful for understanding the data, but:

❌ Not systematic  
❌ Not reproducible  
❌ Can't compare across coders  
❌ No structured output for analysis

**Next step:** Use a formal codebook to get structured, consistent codes!

## What's Next?

In the [next section](codebook.html), we'll use a detailed codebook to get structured JSON output that can be merged back into our dataset for analysis.

---

## Key Takeaways

- 🤖 Use **Claude Haiku 4.5** for survey coding (fast + cheap)
- 🎯 Always set **temperature = 0** for reproducibility
- 📊 Track **token usage** to estimate costs
- 🔍 Exploratory analysis reveals themes
- ➡️ But we need **structured coding** for systematic analysis

## Quick Reference

```r
# Basic API call
call_claude <- function(prompt, temperature = 0) {
  # ... (function from above)
}

# With one response
response <- call_claude("Analyze: 'text here'")

# With multiple responses (batch)
responses <- paste(survey_responses, collapse = "\n\n")
analysis <- call_claude(sprintf("Analyze:\n%s", responses))
```
