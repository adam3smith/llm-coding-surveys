---
layout: default
title: "Structured Coding with a Codebook"
nav_order: 5
---

# Structured Coding with a Codebook
{: .no_toc }

## From Exploration to Systematic Analysis
{: .no_toc }

<div class="code-example" markdown="1">
**Estimated time:** 30 minutes
</div>

---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## The Problem with Free-Form Analysis

Our exploratory approach was useful, but had major limitations:

```r
# What we got:
"This response shows opposition to special treatment..."

# What we need:
{
  "discrim": 2,      # Against discrimination but not special treatment
  "individ": 1,      # Affirmation of individualism
  "bexpment": 0,     # Blacks not explicitly mentioned
  ...
}
```

For systematic analysis, we need:
- ✅ **Consistent format** across all responses
- ✅ **Numeric codes** for statistical analysis
- ✅ **Multiple variables** coded simultaneously
- ✅ **Merge-able** back into our dataset

## Introducing the Codebook

Our codebook has **9 variables** for coding each response:

### 1. **code** (Codeable content?)
- `0` = No (blank, "don't know", one-word answers)
- `1` = Yes (provides understandable idea or reaction)

### 2. **bexpment** (Blacks explicitly mentioned)
- `0` = No
- `1` = Yes

### 3. **wexpment** (Whites explicitly mentioned)
- `0` = No
- `1` = Yes (including Irish, Italians, Jews)

### 4. **tblk** (Traits of Blacks mentioned)
- `0` = No mention
- `1` = Positive (hardworking, self-reliant, smart)
- `2` = Positive & negative
- `3` = Negative (lazy, dependent, making excuses)
- `21` = Blacks are like other groups

### 5. **individ** (Individualism/American Dream)
- `0` = Not mentioned
- `1` = Affirmation (work hard, earn your way)
- `2` = "Not enough" (trying isn't enough for blacks)
- `3` = People flout individualism (get something for nothing)

### 6. **discrim** (Discrimination)
- `0` = Not mentioned
- `1` = Against discrimination; for equality
- `2` = Against discrimination but not special treatment
- `11` = Exists/is problem for blacks
- `21` = Exists/is problem for whites (reverse racism)
- `31` = Not as big a problem anymore
- `51` = "This is not a race thing"

### 7. **welf** (Welfare mentioned)
- `0` = Not mentioned
- `1` = Welfare/government programs mentioned

### 8. **obama** (Obama mentioned)
- `0` = Not mentioned
- `1` = Mentioned

### 9. **rxn** (Reaction to question itself)
- `0` = No reaction
- `1` = Racist/biased question
- `2` = Emotionally upset
- `3` = Dispute accuracy
- `5` = Don't understand question

## Creating the Codebook Prompt

Here's how we embed the codebook in our prompt:

```r
create_codebook_prompt <- function(responses, response_ids) {
  
  codebook <- '
CODEBOOK FOR Q43 (Special Favors Question):

For each response, assign codes for the following variables:

1. code (Codeable content?)
   0 = no (blank/don\'t know/one-word answers)
   1 = yes (provides understandable idea or reaction)

2. bexpment (Blacks explicitly mentioned)
   0 = no
   1 = yes

3. wexpment (Whites explicitly mentioned)
   0 = no  
   1 = yes (including Irish, Italians, Jews)

4. tblk (Traits of Blacks)
   0 = no mention
   1 = positive (hardworking, self-reliant, overcoming adversity)
   3 = negative (lazy, dependent, making excuses)
   21 = blacks are like other groups

5. individ (Individualism/American Dream/work your way up)
   0 = no individualism mentioned
   1 = affirmation of individualism
   2 = "not enough" (trying is not enough for blacks)
   3 = people flout individualism

6. discrim (Discrimination)
   0 = no mention
   1 = against discrimination; should not exist
   2 = against discrimination but not special treatment
   11 = exists/is problem for blacks
   21 = exists/is problem for whites (reverse racism)
   31 = not as big a problem anymore
   51 = "this is not a race thing"

7. welf (Welfare explicitly mentioned)
   0 = no mention
   1 = welfare/government programs mentioned

8. obama (Obama mentioned)
   0 = no
   1 = yes

9. rxn (Reaction to question itself)
   0 = no reaction
   1 = racist/biased/prejudice
   3 = dispute accuracy of statement
'
  
  # Number each response
  numbered_responses <- paste0(
    response_ids, ". ", responses,
    collapse = "\n\n"
  )
  
  # Build complete prompt
  prompt <- sprintf(
    '%s

RESPONSES TO CODE:

%s

Please code each response according to the codebook above. 
Return your results as valid JSON in exactly this format:

{
  "codes": [
    {
      "id": 1,
      "code": 1,
      "bexpment": 0,
      "wexpment": 1,
      "tblk": 0,
      "individ": 1,
      "discrim": 2,
      "welf": 0,
      "obama": 0,
      "rxn": 0
    },
    {
      "id": 2,
      ...
    }
  ]
}

Return ONLY the JSON, no other text.',
    codebook,
    numbered_responses
  )
  
  return(prompt)
}
```

## Requesting Structured Output

The key is being explicit about format:

```r
# ✅ Good: Specific format request
"Return your results as valid JSON in exactly this format: {...}"

# ❌ Vague: Will get free-form text
"Please code these responses."
```

**Critical elements:**
1. Show the exact JSON structure
2. Say "Return ONLY the JSON"
3. Include an example with the right fields

## Coding Your First Batch

Let's code 10 responses:

```r
library(tidyverse)
library(httr2)
library(jsonlite)

# Load data
survey_data <- read_csv("data/Kam_Burge.csv")

# Get 10 non-empty responses
test_responses <- survey_data |>
  filter(!is.na(Q43), Q43 != "") |>
  slice(1:10)

# Create prompt
prompt <- create_codebook_prompt(
  responses = test_responses$Q43,
  response_ids = 1:10
)

# Call Claude
result <- call_claude(prompt, max_tokens = 2000)

# See what we got
cat(result)
```

**Expected output:**
```json
{
  "codes": [
    {
      "id": 1,
      "code": 0,
      "bexpment": 0,
      ...
    },
    ...
  ]
}
```

## Parsing the JSON Response

Claude might wrap JSON in markdown code blocks. Clean it first:

```r
parse_coding_response <- function(claude_response) {
  
  # Remove markdown code fences if present
  cleaned <- claude_response |>
    str_replace_all("```json\\s*", "") |>
    str_replace_all("```\\s*$", "") |>
    str_trim()
  
  # Parse JSON
  parsed <- fromJSON(cleaned, simplifyVector = FALSE)
  
  # Convert to dataframe
  codes_df <- map_dfr(parsed$codes, function(item) {
    tibble(
      response_id = item$id,
      AI_Q1code = item$code,
      AI_Q1bexpment = item$bexpment,
      AI_Q1wexpment = item$wexpment,
      AI_Q1tblk = item$tblk,
      AI_Q1individ = item$individ,
      AI_Q1discrim = item$discrim,
      AI_Q1welf = item$welf,
      AI_Q1obama = item$obama,
      AI_Q1rxn = item$rxn
    )
  })
  
  return(codes_df)
}

# Parse our results
coded_data <- parse_coding_response(result)
print(coded_data)
```

**Output:**
```
# A tibble: 10 × 10
   response_id AI_Q1code AI_Q1bexpment AI_Q1wexpment AI_Q1tblk AI_Q1individ ...
         <int>     <int>         <int>         <int>     <int>        <int>
 1           1         0             0             0         0            0
 2           2         1             0             1         0            1
 3           3         1             1             0         3            1
...
```

## Viewing Codes Alongside Text

```r
# Merge codes back with original text
coded_with_text <- test_responses |>
  mutate(response_id = row_number()) |>
  select(response_id, Q43) |>
  left_join(coded_data, by = "response_id")

# View first few
coded_with_text |>
  select(response_id, Q43, AI_Q1individ, AI_Q1discrim, AI_Q1bexpment) |>
  head(3)
```

**Example output:**
```
  response_id Q43                                    AI_Q1individ AI_Q1discrim ...
1           1 they are                                          0            0
2           2 we are all struggling to be consider...           1            2
3           3 alot of black people have worked th...            1            0
```

## Complete Workflow Function

Wrap it all together:

```r
code_batch <- function(responses, response_ids, temperature = 0) {
  
  cat(sprintf("Coding %d responses...\n", length(responses)))
  
  # Create prompt
  prompt <- create_codebook_prompt(responses, response_ids)
  
  # Call Claude (with enough max_tokens for batch)
  estimated_output <- length(responses) * 120 + 500
  result <- call_claude(prompt, 
                       max_tokens = estimated_output,
                       temperature = temperature)
  
  # Parse
  codes_df <- parse_coding_response(result)
  
  cat(sprintf("✓ Successfully coded %d responses\n", nrow(codes_df)))
  
  return(codes_df)
}

# Use it
codes <- code_batch(
  responses = test_responses$Q43,
  response_ids = 1:10
)
```

## Understanding the Results

Let's examine what Claude coded:

```r
# Distribution of codes
cat("Codeable responses:\n")
table(codes$AI_Q1code)

cat("\nBlacks mentioned:\n")
table(codes$AI_Q1bexpment)

cat("\nIndividualism codes:\n")
table(codes$AI_Q1individ)

cat("\nDiscrimination codes:\n")
table(codes$AI_Q1discrim)
```

## Spot-Checking Quality

Always verify a sample manually:

```r
# Look at a few coded responses
for (i in 1:3) {
  cat(strrep("=", 60), "\n")
  cat(sprintf("Response %d:\n", i))
  cat(sprintf('"%s"\n\n', test_responses$Q43[i]))
  
  cat("Codes:\n")
  cat(sprintf("  individ: %d\n", codes$AI_Q1individ[i]))
  cat(sprintf("  discrim: %d\n", codes$AI_Q1discrim[i]))
  cat(sprintf("  bexpment: %d\n", codes$AI_Q1bexpment[i]))
  cat("\n")
}
```

**Ask yourself:**
- Does the `individ` code make sense?
- Did it correctly identify racial group mentions?
- Are the codes consistent with the codebook?

## Exercise: Code 50 Responses

Try coding a larger batch:

```r
# Get 50 responses
test_50 <- survey_data |>
  filter(!is.na(Q43), Q43 != "") |>
  slice(1:50)

# Code them
codes_50 <- code_batch(
  responses = test_50$Q43,
  response_ids = 1:50
)

# Examine distributions
summary(codes_50)

# Look for patterns
codes_50 |>
  count(AI_Q1individ, AI_Q1discrim) |>
  arrange(desc(n))
```

**Questions to explore:**
1. What's the most common combination of codes?
2. How many responses explicitly mention both blacks and whites?
3. Which responses did Claude mark as not codeable (code=0)?

## Common Issues

### Issue 1: Truncated JSON

**Error:** `"premature EOF"`

**Cause:** `max_tokens` too small for batch size

**Solution:** 
```r
# Calculate appropriate max_tokens
max_tokens <- length(responses) * 120 + 500
```

### Issue 2: Markdown Wrappers

**Response:**
````
```json
{
  "codes": [...]
}
```
````

**Solution:** Already handled in `parse_coding_response()`

### Issue 3: Inconsistent Field Names

Claude might use slightly different names. Verify the structure:

```r
# Check what we got
names(fromJSON(result)$codes[[1]])

# Should be: id, code, bexpment, wexpment, etc.
```

## Comparing with Human Codes

Our data includes human coders (C1Q1*, C2Q1*). Let's compare:

```r
# Merge with survey data
comparison <- survey_data |>
  filter(!is.na(Q43), Q43 != "") |>
  slice(1:10) |>
  mutate(response_id = row_number()) |>
  left_join(codes, by = "response_id")

# Calculate agreement
agreement_individ <- mean(
  comparison$AI_Q1individ == comparison$C1Q1individ,
  na.rm = TRUE
)

cat(sprintf("Agreement on 'individ': %.1f%%\n", agreement_individ * 100))
```

We'll do comprehensive validation in the [Validation section](validation.html).

## What's Next?

We can now code responses systematically! But doing all 1,944 one-by-one would be slow and expensive.

In the [next section](batching.html), we'll learn how to batch multiple responses per API call to maximize efficiency.

---

## Key Takeaways

- 📋 **Codebook in prompt** ensures consistent interpretation
- 📊 **JSON output** enables structured analysis
- 🔢 **Numeric codes** are ready for statistical tests
- ✅ **Parse carefully** to handle markdown wrappers
- 🎯 **Spot-check** to verify quality
- 📏 Use `length(responses) * 120` to estimate `max_tokens`

## Complete Code

```r
# Complete workflow
library(tidyverse)
library(httr2)
library(jsonlite)

# 1. Create codebook prompt
prompt <- create_codebook_prompt(responses, ids)

# 2. Call Claude
result <- call_claude(prompt, max_tokens = 2000, temperature = 0)

# 3. Parse JSON
codes <- parse_coding_response(result)

# 4. Merge with data
survey_data <- survey_data |>
  left_join(codes, by = "response_id")
```
