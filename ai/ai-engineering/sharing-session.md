# Opening

# 0. AI Engineering

## 1st Page

- Thank you. 
- AI Engineering.

## Intro Page

- Loosely based on AI Engineering book (Chip Huyen)
- Integrating AI
- Consider when integrating AI
- Popular book

## Sharing Session

- 5 Sections
- Reverse order
- long and short
- Disclaimer: not expert

# 1. Demo Session

## TITLE PAGE

- Apply knowledge
- Prototype
- Extension
- The idea

## What's Geoguessr

- Geoguessr!
- Web
- Some of you know. 
  - Learner.
- Simple. 
  - Spawn => Guess.

## Let's Play

- 3 Game Mode.
  - No Move.
- More context.
- Let's play!
- 5-7 minutes.
- Still learning. 
  - Easy round.
  - Decent guess

## While Playing

- Collecting the clues.
- Sun, road lines, poles, house, infra, vibes, etc.
- 50% of my power = Sun
- 5 to 7 minutes later …

## How to Get Better

- 2 ways to getting better at the game:
  - plays a lot => took a lot of time
  - plonkit => tedious and less fun
- Combine both!
- OPEN THE EXTENSION!
- Competitive duel vs Solo session
  - NMPZ!

## Possible Country and Possible Region

- READ.
- See Maps.

## Visual Clue

- What this visual clues is doing:
  - It will look at the image,
  - and tell me the relevant clues that I need to pay attention
- READ
- and based on my experience in the last few weeks, by reading this, I feel that I'm getting better at recognizing different region in the country.
- Discuss in tool calling => we'll see why we can't 100% believe all the sentences here.
- Useful

## Trivia

- For fun

## Verified Metas

- First 4 sections => skimming
- The one that I pay most attention: Verified Metas.
  - What I did here,
  - Scrape plonkit
  - It contains a lot of clues in the country
  - LLM will filter among a lot of clues here => which one is relevant to the location
  - Instead of just tediously reading the page.
  - I can just be doing some practice here, and then just read this sections here.
  - Now the learning is more is contextual.
- Play some rounds
  - false positive => better models that has lower amount of false-positives, but there is a reason why I don't use that model => we'll see why when we talk about Choosing a Model.

## Demo Session Epilog

- So What I'm trying here is combining the playing and learning part of it,
  - so I can keep playing but also at the same time learning something about this location.
- That's the demo session.

# 2. Eval and Observability

## Title Page

- Chapter 3 and 4
- Characteristic of AI => same input, diff output.
  - non-deterministic or probabilistic nature.
  - different with traditional software => the same input will reliably produce the same output.
- Because of that non-deterministic nature 
  - we have a challenges  
  - How we can make sure that the AI output in our application is working as expected?
- That what we'll discuss in this section.
  - Evals => Evaluation.

## Evals Definition

- That's what evals is.
- We'll see how I implement evals in my Chrome extension.

## Evals Hot Topic

- To tweet page.
- To the research page.
- To the AI bottleneck page.
- The last 3 pages just shows that Eval is hot topic in AI Engineering.
- This is something that we, as an industry, still figuring out how to do it correctly.
- and it's expected, in the next few months or next few years, the tools and the practice on how to do it will keep changing.

## How To Do Evals

- As for now, generally, there are 3 approach of doing evals.
- Human evals (SKIP)
- Code-based evals (SKIP)
- LLM-based Evals (GO)
  - This is one approach that getting more popular recently.
  - READ.
  - Basically, we use LLM to judge the output of another LLM.
  - So, for example, in my app here, this is the output of the LLM call, right.
    - So, as we'll see shortly, I have another LLM that judges this output. 
    - This another LLM will judge if this output is what I want or not.
  - scalable, and also well-suited for open-ended tasks.

## LLM-as-a-Judge

- Next 2 pages.
- These last 2 pages just illustrate that using LLM as a judge is the common practices, a lot of startups do that.
- And these 2 researches show that LLM-as-a-judge is reliable tools.

## Langfuse Trace

- Langfuse
  - I use Langfuse here as an observability tools.
  - And I also use this to do the evals.
- Before we see the evals
- Trace
  - This lists all the rounds that have been analyzed by my extension.
- CLICK ONE.
  - 2 Parallel LLM Request
    - One for 4 sections
    - One for verified metas
  - Input-Output, Latency, Token usage, cost, model
- I found having trace like this really helpful when I try to develop and improve this application

## Langfuse Evals

- So, yeah, let's talk about the evals that I have in place here.
  - To be fair, this is quite a simple app, so for now I only 1 simple evals.
- I have 1 evals that evaluate the result of this section (Visual Clues)
  - We can see on the traces here, I have an eval that running on each request.
  - Comparison Check Evals
  - This one, the score can be 1, 0.5, 0.
  - I use LLM-as-a-judge here.
- Let's see how it calculates this score.

## Langfuse LLM-as-a-judge

- Check LLM-as-a-judge menu
- READ
- So, in this section, I want to make sure that the list here is contain geographical comparison.
- Because that's the skill that important.
  - How to differentiate between Thailand and Cambodia.
  - Or how to differentiate between North of Brazil and South Brazil.
- That's why in the prompt I emphasize about including comparison.

## Goals of Evals

- So, yeah, that's the evals that I implement on this app.
- But I can imagine another scenario, for example, I can have an evals to evaluate the factual correctness of each sentence here.
  - We'll talk about tool calling later, but it's possible for example, if judges here utilize web search to verify the correctness of each sentence.
- So, there are a lot of possibility here.
- But the idea is: we put evals on the place that we care about.
  - If I care about the existence of comparison here, I can put an eval on that.
  - If I care about the factual correctness of each sentence here, I can put an eval on that.
  - There are a lot of possibility here.

## Langfuse Dataset

- But of course, it's not ideal, to run an eval on each request like what I do here. 
  - because, for example, if this is on production, with a lot of users, it will be costly to run the evals on each request.
  - because it's, at the end of the day, just another LLM call.
- So, from what I heard, the approach that commonly use in a lot of AI startup: instead of running evals on every request, they prepare the datasets.
- DATASETS MENU. 
  - READ.

## Langfuse Dataset Geowizr

- For example, in my app, I can collect like 100-200 locations and then insert it to the datasets.
- Actually langfuse also have the button here: Add to datasets.
- So I can just add this location in the datasets.
- So yeah, I can collect like 200 locations in my datasets. 
  - And then, I can run it every week, or every time I change the models, or every time I changed the prompt.
  - I can just run the evals on the dataset, and then check the average of the evals score.
- So, yeah, from what I heard, that's what they usually do:
  - Everytime they make a changes, either changes a prompt, a model, or make a new release.
  - They can run the evals on the dataset to check if the AI output still work as expected or not.

## Evals ending

- That's all about evals.
- Of course this is depth and broad topic, so I'm only able to scratching the surface here.
- So, let's move on to the next section.
- The rest of the sections should be much shorter than this section.

# 3. Choosing A Model

## Amplify Common Models

- This is research from Amplify.
- Which LLM are you using for customer facing products.
- Of course, the 3 common name here is: OpenAI, Anthropic, and Google Gemini.
  - There is also deepseek here.
  - For my app here, I actually tried 3 different providers: Gemini, Claude, and OpenAI.
  - For this use case, I found that the models from OpenAI is performed better than the others.

## Capabilities, Speed or Latency, Costs

- This is from Anthropic docs, but the principle generally similar.
- This doc said that when choosing the models, there are 3 factors that we need to consider: Capabilities, Speed or Latency, and Costs.

## Costs

- For my app, I'm not really care about costs, because this app is only for personal use, and I also don't play that much.
- So what I really care here are: capabilities and latency.
  - And how to find balance between those 2

## Capabilities and Latency (OpenAI Docs)

- About the capabilities, I found that Reasoning Model produce better output for my app.
- GO TO OPENAI DOCS
- CHECK REASONING MODELS PAGE.
- READ 1ST PARAGRAPH.
- But the issue with reasoning model is: they have higher latency than non-reasoning model.

## Geowizr Latency Issue

- And for my use case here: I need an instant response. So if I use reasoning model, the output is better, but I need to wait longer.
  - And for this specific case, it's not the trade-off that I want to make.
  - Okay with false-positive
  - that's why I go to with GPT4.1, which is non-reasoning model.
- But I can imagine, maybe I can use the reasoning model on competitive duels,
  - not use that for cheating.
  - but maybe analyze my gameplay while I'm playing.
  - and after the game is finished, I can see the summary, the feedback, the clues that I miss, etc.
  - for that case, I think reasoning models could be good, because latency is something that I can tolerate there, since I'll see the result after the game is finished, so the model will have time to do the reasoning.
- So at the end of the day, we need to decide which model that better for our use case.

## On Coosing the Model 

- There is also this quote from the book: ...
- And this is also shows why it's important to have evals.
  - to know if this model is suitable or not for our use case, we can just run the evals and do the comparison between models there.
  - evals score, latency, cost.

# 4. Prompt Engineering

## Title Page

- Now we move on to prompt engineering.
- I think this is the one that we are most familiar with.

## Prompt Engineering Definitions

- This is the definition.
- I think we can skip that.

## Prompt Engineering Claude

- This is some tips from Anthropics docs.
  - I think we can go there.
- GO TO CLAUDE 4 BEST PRACTICES IN PROMPT ENGINEERING.
- This is specifically for Anthropic model, but I think it generally useful also for the others models.

## Prompt Engineering Tips

- Be explicit.
  - READ.
  - This is also the tips that I consistently found in the docs from the others models.
  - I think here they give use some examples and comparison (GO TO: BE CLEAR AND DIRECT PAGE)
- Add context
  - READ
- Examples
  - GO TO => USE EXAMPLES (MULTI-SHOT PROMPTING)
  - This is also discussed quite heavily in the book.
  - It's good to give an example in our prompt.
  - this is the comparison here.
  - GO TO: OPENAI DOCS (PROMPTING => PROMPT ENGINEERING => FEW-SHOT LEARNING)
- XML Tags
  - READ
  - BACK TO: OPENAI DOCS (PROMPTING => PROMPT ENGINEERING => FEW-SHOT LEARNING) => it's also use XML TAGS
- Epilog
  - There are a lot of others tips here related to prompt engineering.
  - BACK TO SLIDE.

## Prompt Tips from The Book

- The book itself mention some tips that I include here:
  - Not all parts of prompt are equal
  - why examples
  - number of examples
  - json format
- JSON FORMAT
  - This book discuss Structured Output.
  - How if we want the output of the LLM follow certain format.
  - But for this case, the library that I use is solved that issue.
- GO TO AI SDK
  - I use this AI SDK from Vercel.
  - This is really streamlining my process of building AI application.
  - One of the benefit is I can change the model provider, like OpenAI, Anthropic, Google Gemini, without changing my code too much.
  - The other benefit is I can just specify the schema here, so the LLM output will always follow this structured.
  - READ EXAMPLE.
  - I found that in 99% of the case, it works as expected.
  - So yeah, I think this is really helpful.

# 5. Tool Calling


## CLAUDE.AI TOOL CALLING

- Here I have a screenshot from my chat with Claude.
- I think is conversations shows why tool calling is really helpful.
- here I explicitly tell Claude to not use any tool calling on doing this multiplication.
  - And it gives me the wrong answers
- And when I tell Claude to use tool calling, it will use this Code Execution tool.
  - It will use JS code here.
  - And as we can see, the result is now correct.
- I explicitly said not to use tool calling, because the model quite good enough to figure out that this task need tool calling
  - So I explicitly said don't, to make this comparison.

## KARPATHY

- Why it gives me the wrong answer here.
- I think the explanation from Andrej Karpathy, the cofounder of OpenAI, is really good.
  - We can think LLM as something that consume the whole internet.
  - So, when it tries to answer this question, without using tool calling.
  - It's similar with: When I read a book, and then someone ask me, about the content of the book.
  - If I don't use any tool calling, I will answer the question based on what is in my head.
  - But I can use tool calling, I can, wait a minute, let me check the book again, or let me check the internet about that, etc.
  - I think that analogy quite useful to understand why tool calling is really important
- So this illustrates why tool calling is powerful on increasing the accuracy of LLM output.

## Tools Calling Definition

- I think we can skip this.

## Tools Calling (OPENAI DOCS)

- This is the list of all tool calling that OpenAI model can do.
  - READ
- For my app, I can imagine Function Calling and Web Search will be useful.

## Function Calling

- READ
- I can imagine, if you see here, I actually have the latitude and longitude here.
  - And LLM use this data to infer the country and the region inside that country.
  - But now it still inaccurate sometimes.
- So maybe I can create a custom function in my code, that call GoogleMaps API, so I can get the exact City or the exact Province of this location. 
  - So it's no longer prediction.
  - So we can remove the non-deterministic part in this section.

## Web Search

- READ
- I can imagine, the LLM can use the web search to verify the correctness of all of this statement.
  - Because sometimes, here, the AI give me something that just didn't make sense.
- CAT story
  - So one day, I got Sri Lanka => look MAP => south of India.
  - In the image, there is a road, and on the side of the road, there is a CAT.
  - And in this section here, the AI said that "Cat in Sri Lanka usually look healthier than the cat in India"
  - And I'm not really sure about that.
  - I google it, and didn't find any information about that.
  - So yeah, maybe, I can use tool calling to increase the factual correctness of the statements in this section.
- So that's about tool calling
  - One of the way to reduce the hallucinations or inaccuracy in the AI output.

# 6. Beyond This Session

- So yeah, there are a lot of things that I can't discuss in this sharing session.
  - Partially because I don't have much time.
  - But mostly because my understanding of these topic still limited.
- So yeah, basically, this sharing session only scratching the surface about AI Engineering.

# Closing

- So if you are interested on going deeper on AI Engineering, I found that this book is quite helpful.
- And lastly, if any of you have interest in geography or just curious about the world. Please try the game, it's really fun, at least for me.
- That's all from me.
  - Thank you.

# Reference

- https://blog.duolingo.com/how-duolingo-experts-work-with-ai/
- https://www.amplifypartners.com/blog-posts/the-2025-ai-engineering-report
- https://www.aihero.dev/how-to-improve-your-llm-powered-app

- EVALS

- https://vercel.com/blog/eval-driven-development-build-better-ai-faster
- https://www.lennysnewsletter.com/p/an-ai-glossary
- https://www.lennysnewsletter.com/p/beyond-vibe-checks-a-pms-complete

# Docs

- https://platform.openai.com/docs
- https://cookbook.openai.com/
- https://ai.google.dev/gemini-api/docs
- https://docs.anthropic.com/en/docs
- https://ai-sdk.dev/docs