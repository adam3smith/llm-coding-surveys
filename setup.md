---
layout: default
title: "Setup"
nav_order: 2
---

# Setup
{: .no_toc }

## Getting Your Environment Ready
{: .no_toc }

<div class="code-example" markdown="1">
**Estimated time:** 15 minutes
</div>

---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Step 1: Install R Packages

We'll use three key packages:

```r
# Install required packages
install.packages(c("tidyverse", "httr2", "jsonlite"))

# Load them
library(tidyverse)
library(httr2)
library(jsonlite)
```

**What each package does:**
- `tidyverse`: Data manipulation and visualization
- `httr2`: Modern HTTP client for making API calls
- `jsonlite`: Parsing JSON responses from the API

## Step 2: Get an Anthropic API Key

### Create an Account

1. Go to [console.anthropic.com](https://console.anthropic.com)
2. Sign up with your email or Google account
3. Verify your email address
4. You'll receive **$5 in free credits** (enough for this entire workshop!)

### Generate an API Key

1. Navigate to **Settings** → **API Keys**
2. Click **"Create Key"**
3. Give it a name (e.g., "Survey Coding Workshop")
4. Copy the key (starts with `sk-ant-api...`)

{: .warning }
> **Keep your API key secret!** Never commit it to GitHub or share it publicly. Anyone with your key can use your credits.

### Set Up Your API Key

**Option 1: Environment Variable (Recommended)**

Add to your `.Renviron` file:

```r
# Edit your .Renviron file
usethis::edit_r_environ()

# Add this line (replace with your actual key):
ANTHROPIC_API_KEY=sk-ant-api-your-key-here

# Save and restart R
```

**Option 2: Session Variable (Quick & Temporary)**

```r
# Set for this session only
Sys.setenv(ANTHROPIC_API_KEY = "sk-ant-api-your-key-here")
```

**Verify it's set:**

```r
# Should print your key
Sys.getenv("ANTHROPIC_API_KEY")
```

## Step 3: Download Workshop Materials

### Sample Data

Download the survey data: [Kam_Burge.csv](../data/Kam_Burge.csv)

This contains 1,944 survey responses with:
- **Q43, Q45, Q47, Q49:** Open-ended responses
- **C1Q1*, C2Q1*:** Human coder 1 & 2 codes (for validation)

```r
# Create a data directory
dir.create("data", showWarnings = FALSE)

# Read the data
survey_data <- read_csv("data/Kam_Burge.csv")

# Quick look
glimpse(survey_data)
```

### Codebook

Download: [codebook.pdf](../resources/codebook.pdf)

This 9-variable coding scheme will guide our structured coding.

## Step 4: Test Your Setup

Let's make sure everything works with a simple "Hello World" API call:

```r
# Function to call Claude
call_claude_simple <- function(prompt, api_key = Sys.getenv("ANTHROPIC_API_KEY")) {
  
  response <- request("https://api.anthropic.com/v1/messages") |>
    req_headers(
      `x-api-key` = api_key,
      `anthropic-version` = "2023-06-01",
      `content-type` = "application/json"
    ) |>
    req_body_json(list(
      model = "claude-haiku-4-5-20251001",
      max_tokens = 100,
      messages = list(
        list(
          role = "user",
          content = prompt
        )
      )
    )) |>
    req_perform()
  
  result <- resp_body_string(response) |>
    fromJSON()
  
  result$content[[1]]$text
}

# Test it!
call_claude_simple("Say 'Setup complete!' in a friendly way.")
```

**Expected output:** Something like `"Setup complete! 🎉 You're ready to go!"`

## Troubleshooting

### API Key Issues

**Error:** `"x-api-key header is required"`
- **Solution:** Your API key isn't set. Go back to Step 2.

**Error:** `"Invalid API key"`
- **Solution:** Double-check you copied the entire key, including `sk-ant-api` prefix.

### Package Issues

**Error:** `there is no package called 'httr2'`
- **Solution:** Run `install.packages("httr2")`

**Error:** `could not find function "request"`
- **Solution:** Load the package: `library(httr2)`

### Network Issues

**Error:** `Failed to connect to api.anthropic.com`
- **Solution:** Check your internet connection or firewall settings.

### Rate Limit Warning

If you see `429` errors during testing:
- **Don't worry!** We'll handle this properly in later sections.
- For now, just wait 10 seconds and try again.

## Verify Your Setup

Run this checklist:

```r
# ✅ Packages loaded
library(tidyverse)
library(httr2)
library(jsonlite)

# ✅ API key set
stopifnot(Sys.getenv("ANTHROPIC_API_KEY") != "")

# ✅ Data loaded
survey_data <- read_csv("data/Kam_Burge.csv")
stopifnot(nrow(survey_data) == 1944)

# ✅ API call works
test_response <- call_claude_simple("Say hello!")
print(test_response)

cat("✅ All checks passed! You're ready to start.\n")
```

## What's Next?

Now that your environment is ready, let's understand [**why we're using LLMs**](motivation.html) for survey coding and how they differ from traditional approaches.

---

## Quick Reference

**Key Files:**
- `data/Kam_Burge.csv` - Survey responses
- `codebook.pdf` - Coding scheme

**Essential Commands:**
```r
# Load packages
library(tidyverse); library(httr2); library(jsonlite)

# Check API key
Sys.getenv("ANTHROPIC_API_KEY")

# Load data
survey_data <- read_csv("data/Kam_Burge.csv")
```
