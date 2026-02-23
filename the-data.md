---
layout: default
title: "The Data"
nav_order: 3
---

# Meet the Data

We're replicating [a research note](https://doi.org/10.1086/693907) by Cindy Kam and  Camille Burge. 2018. ‘Uncovering Reactions to the Racial Resentment Scale across the Racial Divide’. The idea of the paper is quite straightforward. They ask participants -- half of them Black, half white -- a battery of questions referred to as the "Racial Resentment Scale" and then ask them follow-ups to figure out what they're thinking about. Here's how they describe this part in the article:

>  After responding to each of the racial resentment items, each respondent received a “stop and reflect” cognitive response task (Ericsson and Simon 1980): “Thinking about the question you just answered, exactly what things went through your mind? Please type your response below.” The method provides a glimpse into “the way in which a person views the world” (Cacioppo et al. 1997, 928) but should not be assumed to be a veritable explanation of attitudes or behaviors.2 Two coders, blind to identifying characteristics of the respondents, coded each open-ended response.3 We analyze the extent to which respondents engage the three pillars of racial resentment: negative traits of blacks, the principle of individualism, and recognition (or denial) of discrimination.


## Download the Data and Codebook

It takes some time to figure out where their data, code, and codebook are. 

- The data and code are on Dataverse. Download from [the Harvard Dataverse](https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/UV7ZPZ)
- The coding instructions for the coders are actually not in the Codebook but in the [article's appendix](https://www.journals.uchicago.edu/doi/suppl/10.1086/693907/suppl_file/170173Appendix.pdf) \[PDF\].

## Meet the data

We'll put the CSV file into a data subdirectory in our project folder. We can then load it using

```r
survey_data <- read_csv("data/Kam_Burge.csv")
```

You can see the basic structure here: a number reflecting the answer to a Likert-type question from the racial resentment scale followed by the respondents' explanation.

![Survey structure](/images/data_structure.png)

Here's the same section from [their codebook](/resources/Kam%20and%20Burge%20Codebook.pdf)

![Codebook excerpt from Kam and Burge](/images/codebook_excerpt.png)


### Coding Details

Here is an excerpt from the coding instructions

![Coding instructions excerpt](/images/coding_instructions.png)

The full instructions, which we'll use later, are [in their appendix](/resources/Appendix.pdf).

If you look at the codebook and the data, you'll see that each of the "reflection" questions was coded for these 9 variables and the data reflects which of the coders (or both) has done so, so we end up with a _lot_ of variables. You can look at the authors' [Stata code](/resources/Kam%20and%20Burge%20JOP%20replication%20code.do) to see how they end up aggregrating these variables, but we'll not get to that in this workshop.

## What's Next?

Now that your environment is ready and we've met the data, let's understand [**why we're using LLMs**](motivation.html) for survey coding and how they differ from traditional approaches.
