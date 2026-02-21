---
layout: default
title: "Resources & Downloads"
nav_order: 11
---

# Resources & Downloads
{: .no_toc }

## Everything You Need
{: .no_toc }

---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Workshop Materials

### Data Files

**Survey Data**
- [Kam_Burge.csv](../data/Kam_Burge.csv) - 1,944 survey responses with human codes
- Format: CSV with Q43, Q45, Q47, Q49 (open-ended) and C1Q1*, C2Q1* (human codes)
- Size: ~500 KB

**Codebook**
- [Codebook PDF](../resources/codebook.pdf) - Full 9-variable coding scheme
- [Codebook TXT](../resources/codebook.txt) - Plain text version for copy-paste

### R Scripts

**Complete Working Scripts**
- [01_setup.R](../scripts/01_setup.R) - Initial setup and API testing
- [02_codebook_coding.R](../scripts/02_codebook_coding.R) - Structured coding functions
- [03_production_system.R](../scripts/03_production_system.R) - Full production code with progress saving
- [04_validation.R](../scripts/04_validation.R) - Compare with human coders
- [ALL_SCRIPTS.zip](../scripts/ALL_SCRIPTS.zip) - Download everything

**Helper Functions**
- [call_claude.R](../scripts/call_claude.R) - API call function with retry logic
- [parse_json.R](../scripts/parse_json.R) - JSON parsing and cleaning
- [merge_codes.R](../scripts/merge_codes.R) - Merge codes back to data

### Slides

- [Workshop Slides (PDF)](../slides/LLM_Survey_Coding_Workshop.pdf)
- [Workshop Slides (PowerPoint)](../slides/LLM_Survey_Coding_Workshop.pptx)

## Code Examples

### Basic API Call

```r
library(tidyverse)
library(httr2)
library(jsonlite)

call_claude <- function(prompt, 
                        model = "claude-haiku-4-5-20251001",
                        max_tokens = 1024,
                        temperature = 0,
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
        list(role = "user", content = prompt)
      )
    )) |>
    req_perform()
  
  result <- resp_body_string(response) |> fromJSON()
  result$content[[1]]$text
}
```

### Codebook Template

```r
codebook <- '
CODEBOOK FOR [YOUR QUESTION]:

For each response, assign codes for the following variables:

1. [variable_name] ([description])
   0 = [category 0 description]
   1 = [category 1 description]
   2 = [category 2 description]

2. [another_variable] ([description])
   0 = [category description]
   1 = [category description]

[Add more variables as needed]
'
```

### Complete Workflow

```r
# 1. Load packages
library(tidyverse)
library(httr2)
library(jsonlite)

# 2. Load data
survey_data <- read_csv("data/Kam_Burge.csv")

# 3. Source functions
source("scripts/03_production_system.R")

# 4. Code responses
q43_codes <- code_Q43(survey_data)

# 5. Merge
survey_data <- merge_codes_into_survey(survey_data, q43_codes)

# 6. Validate
agreement <- compare_all_variables(survey_data)

# 7. Save
write_csv(survey_data, "survey_with_codes.csv")
```

## External Resources

### Anthropic Documentation

- **Main Docs:** [docs.anthropic.com](https://docs.anthropic.com)
- **API Reference:** [docs.anthropic.com/en/api](https://docs.anthropic.com/en/api)
- **Prompt Engineering:** [docs.anthropic.com/en/docs/build-with-claude/prompt-engineering](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering)
- **Rate Limits:** [docs.anthropic.com/en/api/rate-limits](https://docs.anthropic.com/en/api/rate-limits)

### R Packages

**Required:**
- `tidyverse` - [tidyverse.org](https://www.tidyverse.org/)
- `httr2` - [httr2.r-lib.org](https://httr2.r-lib.org/)
- `jsonlite` - [CRAN](https://cran.r-project.org/package=jsonlite)

**Optional:**
- `psych` - For Cohen's kappa
- `caret` - For confusion matrices
- `furrr` - For parallel processing

### Learning Resources

**Prompt Engineering:**
- [Prompt Engineering Guide](https://www.promptingguide.ai/)
- [OpenAI Prompt Engineering](https://platform.openai.com/docs/guides/prompt-engineering)
- [Anthropic Claude Prompting](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering)

**LLMs for Research:**
- [Using LLMs for Qualitative Analysis](https://arxiv.org/abs/2304.08109)
- [Reliability of LLM Annotations](https://arxiv.org/abs/2306.00176)

**Survey Research:**
- [Inter-rater Reliability Primer](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3900052/)
- [Coding Open-Ended Responses](https://journals.sagepub.com/doi/10.1177/0049124118799377)

## Community & Support

### Getting Help

**Technical Issues:**
- **R questions:** [StackOverflow](https://stackoverflow.com/) (tags: `r`, `httr2`, `claude-api`)
- **API issues:** [Anthropic Support](https://support.anthropic.com)
- **Anthropic Discord:** [Community link](https://www.anthropic.com/discord) (if available)

**Methodological Questions:**
- **Computational Social Science:** [CSS community forums](https://computationalsocialscience.org/)
- **Text-as-Data:** [Text as Data Association](https://www.textasdata.org/)

### Workshop Contact

- **Instructor:** [Your name/email]
- **GitHub Repository:** [github.com/your-repo/llm-survey-coding](https://github.com/)
- **Questions/Issues:** [GitHub Issues](https://github.com/your-repo/issues)

## Citation

If you use this workshop material in your research, please cite:

```bibtex
@misc{llm_survey_coding_workshop,
  title = {LLM-Powered Survey Coding with Claude: A Hands-On Workshop},
  author = {[Your Name]},
  year = {2026},
  howpublished = {\url{[workshop-url]}},
  note = {Workshop materials for using large language models in survey research}
}
```

## Related Publications

Papers using similar methods:

1. Gilardi, F., Alizadeh, M., & Kubli, M. (2023). "ChatGPT Outperforms Crowd-Workers for Text-Annotation Tasks." *arXiv preprint arXiv:2303.15056*.

2. Törnberg, P. (2023). "ChatGPT-4 Outperforms Experts and Crowd Workers in Annotating Political Twitter Messages with Zero-Shot Learning." *arXiv preprint arXiv:2304.06588*.

3. Ziems, C., et al. (2023). "Can Large Language Models Transform Computational Social Science?" *arXiv preprint arXiv:2305.03514*.

## Model Information

### Claude Models (January 2026)

| Model | Context | Input Cost | Output Cost | Best For |
|-------|---------|------------|-------------|----------|
| Haiku 4.5 | 200K | $0.25/1M | $1.25/1M | High-volume coding |
| Sonnet 4.5 | 200K | $3/1M | $15/1M | Complex reasoning |
| Opus 4.5 | 200K | $15/1M | $75/1M | Highest quality |

**Model IDs:**
- `claude-haiku-4-5-20251001`
- `claude-sonnet-4-5-20250929`
- `claude-opus-4-5-20251101`

### Rate Limits (Tier 1)

- **Requests:** 50 per minute
- **Input tokens:** 50,000 per minute
- **Output tokens:** 50,000 per minute
- **Daily limit:** Based on tier

Check current limits: [console.anthropic.com/settings/limits](https://console.anthropic.com/settings/limits)

## Changelog

### Version 1.0 (January 2026)
- Initial workshop release
- Supports Claude Haiku 4.5
- 9-variable codebook for racial attitudes survey
- Production-ready code with progress saving

### Planned Updates
- Support for Claude Sonnet 4.5
- Additional codebook examples
- Multi-language survey support
- Integration with Qualtrics API

## License

Workshop materials are released under [MIT License / CC-BY-4.0].

**You are free to:**
- Use for teaching
- Adapt for your research
- Share with attribution

## Acknowledgments

This workshop was developed with support from:
- [Your institution]
- [Funding source]
- [Collaborators]

Special thanks to participants who provided feedback!

---

## Quick Links

**Essential Links:**
- 📊 [Survey Data](../data/Kam_Burge.csv)
- 💻 [All Scripts](../scripts/ALL_SCRIPTS.zip)
- 📚 [Anthropic Docs](https://docs.anthropic.com)
- 🆘 [Troubleshooting](troubleshooting.html)

**Start Here:**
1. [Setup](setup.html) - Get your environment ready
2. [Motivation](motivation.html) - Understand the approach
3. [First API Call](first-call.html) - Make your first request

**Need Help?**
- Check [Troubleshooting](troubleshooting.html) first
- Ask on StackOverflow with tags `r` + `claude-api`
- Contact workshop instructor: [email]
