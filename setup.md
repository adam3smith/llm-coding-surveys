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



## Step 1: Set up an R environment and Install R Packages

I always recommend that you create a new R Project in an empty folder for any R coding. Go to File --> New Project and create a project; you can name it however you want.
Then open a new, empty R-Scipt (File --> New File --> RScript). This is where you'll build your code gradually.

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

## Step 2: Get an Anthropic API Key and Set it in R

### Create an Account

**You don't have to do this for the in-person workshop. I'll provide you with an API key**

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

_We'll use this option for the workshop_

```r
# Set for this session only
Sys.setenv(ANTHROPIC_API_KEY = "sk-ant-api-your-key-here")
```

**Verify it's set:**

```r
# Should print your key
Sys.getenv("ANTHROPIC_API_KEY")
```


## Step 3: Test Your Setup

An API is an "Application Programming Interface" -- it's how you can talk to an application (typically, but not always, on the web) from your own program -- here, our RScript. 

Let's start with a simple "Hello World" API call:

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
      model = "claude-haiku-4-5",
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
  
  result
}

# Test it!
output = call_claude_simple("Say 'Setup complete!' in a friendly way.")
```
Let's first look at the function. A lot of the structure here is provided by R's `httr2` package. It contains 4 parts:
1. The URL for the API (`https://api.anthropic.com/v1/messages`)
2. The "Header" -- here's where you tell the API who you are (via the API key) and what type of service you'd like (i.e. a specific version of the API and the format for the response)
3. The "Body" -- this is where all the substance happens:
  - You specify the model
  - The prompt
  - You specify maximum tokens to use (more on this later)
  - You can specify other things, some of which we'll touch on.


Let's look at the output:
![API Output Structure](/images/output_structure.png)

We want the text in the `content` section, so let's try `output$content$text`. You should get something like

> "Setup complete! 🎉 You're all good to go!"

We can now update the last line of the function to directly give us the output, relod the function, and try again.

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

### Rate Limit or Server Overloaded Warning

If you see `429` or `529` errors during testing:
- **Don't worry!** We'll handle this properly in later sections.
- For now, just wait 10 seconds and try again.



## What's Next?

Now that your environment is ready, let's [**meet the data**](the-data.html) we'll be working with.

