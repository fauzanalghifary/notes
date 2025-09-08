# Opening

# 0. AI Engineering

- Thank you. 
- AI Engineering.

- Loosely based on AI Engineering book (Chip Huyen)
- Integrating AI
- Consider when integrating AI
- Popular Book

- 5 Sections
- Reverse order
- Disclaimer: not expert

# 1. Demo Session

## TITLE PAGE

- Apply Knowledge
- Prototype
- Extension
- The idea

## GEOGUESSR SESSION

## what's geoguessr

- Geoguessr!
- Web
- Some of you know. Learner.
- Simple. Spawn => Guess.

## let's play

- More context.
- Let's play!
- 5-7 minutes.
- 3 Game Mode.
  - No Move.
- Still learning. Easy round.

## playing

- Collecting the clues.
- Road lines, Poles, Sun, House, Infra, etc.
- 5 to 7 minutes later …

## how to get better

- 2 ways to getting better at the game
  - plays a lot => took a lot of time
  - plonkit => tedious and less fun
- Combine both!

## open extension!

- Competitive duel vs Solo session
- NMPZ!

## explain the extension

- 5 Sections

## Possible Country and Possible Region

- 99% correct.
- Not because the AI is that good => because of the input
- Langfuse
  - Will talk more when we discuss Eval and Observability.
  - 6 data
  - intercepting network request.
  - also image.

## Visual Clue

- Look at the image => tell me the important part in the image => how it relate to the country and/or region.
- READ
- discuss in tool calling => we'll see why we can't 100% believe the fact that present here.
- based on experience => help me label things.
  - Tree name
  - Pampas Region
  - Mountain Range in East USA
- This visual clues here is giving me labels to things that I otherwise will never known.
- Comparison.
  - Poland.

## Trivia

- for fun

## Verified Metas

- First 4 sections => skimming
- The one that I pay most attention: Verified Metas.
  - Once i got the country code
  - I have map this to the country name
  - My app will scrap this plonkit page
  - LLM will filter among a lot of clues/meta here => which one is relevant to the location
  - Instead of just tediously reading the page.
  - I can just doing some practice here, and then just read this sections here.
  - Now the learning is more is contextual.

## ending

- That's the demo session.
- The intention is I want to keep playing, but also learning something about the game, so I can getting better faster at this game.

# 2. Eval and Observability

## Title Page

- Chapter 3 and 4
- Characteristic of AI => same input, diff output.
- non-deterministic or probabilistic nature.
- different with traditional software => the same input will reliably produce the same output.
- Because of that non-deterministic nature => we have a challenges => How we can make sure that the AI output in our application is working as expected?
- That what we'll discuss in this section.
- Evals => Evaluation.

## Evals Definition

- To page lenny, to vercel page, back to lenny.
- That's what evals is.
- We'll see how I implement evals in my chrome extension.

## Evals Hot Topic

- to tweet page.
- to the research page.
- to the AI bottleneck page.
- The last 3 pages just shows that Eval is hot topic in AI Engineering.
- This is something that we, as an industry, still figuring out how to do it correctly.
- and it's expected, in the next few months or next few years, the tools and the practice on how to do it will keep changing.

## How To Do Evals

- As for now, generally, there are 3 approach of doing evals.
- Human evals
  - READ
  - Me.
  - Evaluate the output here. 
  - Give the thumbs up, thumbs down, or give the rating to the output here. and send it to the system.
  - And the owner of the app, can collect the data, and make improvements based on those feedback.
  - But the minus here:
    - Costly and time-consuming.
- Code-based evals
  - READ
  - This is quite similar with traditional unit test that we usually write.
  - Given this input, for example, the output should contain this keyword.
  - But the difference is, as we discuss previously, AI is non-deterministic, so the same input can produce different output.
  - It's more challenging if the tasks is open ended.
- LLM-based Evals
  - The approach that getting more popular recently, is LLM-based Evals.
  - So, for example, in my app here, this is the output of the LLM call, right.
  - So, as we'll see shortly, I have another LLM that judges this output. 
  - This another LLM will judge if this output is what we want or not.
  - scalable, and also well-suited for open-ended tasks.

## LLM-as-a-Judge (SKIP)

- Next 2 pages.
- These last 2 pages just illustrate that using LLM as a judge is the common practices, a lot of startups do that.
- And these 2 researches show that LLM-as-a-judge is reliable tools.

## What's Inside Langfuse

- Langfuse
  - As we see previously, I use Langfuse here as an observability tools.
  - And I also use this to do the evals.
- Before we see the evals
- Trace
  - This lists all the rounds that have been analyzed by my extension.
  - CLICK ONE.
  - 2 Parallel LLM Request
  - Latency, Token usage, cost, model
- I found having trace like this really helpful when I try to develop the application.

--- optional (SKIP)

- For example, here I have a reason field, this is not shown in the UI.
  - Sometimes when I check this section and found something that is a bit off, I check this, and maybe adjust my prompt so it produce better result next time.
- I also can compare latency and cost when I experimenting with different models.
- So, yeah, I found that having trace like this really helpful on improving the application.

## Langfuse Evals

- So, yeah, let's talk about the evals that I have in place here.
- To be fair, this is quite a simple app, so for now I only 1 simple evals.
- I have 1 evals that evaluate the result of this section (Visual Clues)
- We can see on the traces here, I have an eval that running on each request.
- Comparison Check Evals
- This one, the score can be 1, 0.5, 0.
- I use LLM-as-a-judge here.
- Let's see how it calculates this score.

## Langfuse Evals Part 2

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
- If I care about the present of comparison here, I can put an eval on that.
- If I care about the factual correctness of each sentence here, I can put an eval on that.
- There are a lot possibility here.

## Langfuse Dataset

- But of course, it's not ideal, to run an eval on each request, because it will be costly.
- So, from what I heard, the approach that commonly use in a lot of AI startup: instead of running evals on every request, they prepare the datasets.
- DATASETS MENU. READ.
- For example, in my app, I can collect like 100-200 locations and then insert it to the datasets.
- Actually langfuse also have the button here: Add to datasets.
- So I can just add this location in the datasets.
- So yeah, I can collect like 200 locations in my datasets. 
- And then, I can run it every week, or every time I changes the models, or every times I changed the prompt.
- I can just run the evals on the dataset, and then check the average of the evals score.
- So, yeah, from what I heard, that's what they usually do:
  - Everytime they make a changes, either changes a prompt, a model, or the app flow.
  - They can run the evals on the dataset to check if the AI output still work as expected or not.

## Evals ending

- That's all about evals.
- Of course this is depth and broad topic, so I only able to scratching the surface here.
- Okay, let's move on to the next section.
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

## Costs (Claude Docs)

- About cost, this is quite straightforward.
- One of the nuance about cost is, it's easy to assume we can just see the pricing table and then compare the number here.
- For example, by looking at this table, we can conclude that Opus 4 will cost 5x times than Sonnet 4 here.
- the issue is: different model will use different number of token.
- So that's why the better way to compare the cost is to collect the dataset, like we mention previously.
- So we can just run the dataset for each model that we want to test, and then we can compare the cost.
- For my app, I'm not really care about costs, because this app is only for personal use, and I also don't play that much.
- So what I really care here are: capabilities and latency.

## Capabilities and Latency (OpenAI Docs)

- About the capabilities, I found that Reasoning Model is produce better output for my app.
- GO TO OPENAI DOCS
- CHECK REASONING MODELS PAGE ON OPENAI DOCS.
- READ 1ST PARAGRAPH.
- But the issue with reasoning model is: they have high latency than reasoning model.
- And for my use case here: I need an instant response. So if I use reasoning model, the output is better, but I need to wait longer.
- And for this specific case, it's not the trade-off that I want to make.
- MOVE TO BEST PRACTICE.
- CHECK WHEN TO USE REASONING MODELS.
- and here is the one that related to my use case: visual reasoning.
- I got better results using reasoning models.
- but for my case here, I need instant response, and I also don't mind if the AI give me false-positive.
- that's why i go to with 4.1
- but I can imagine, maybe I can use this tools on competitive duels,
  - not use that for cheating.
  - but maybe analyze my gameplay while I'm playing.
  - and after the game is finished, I can see the summary, the feedback, the clues that I miss, etc.
  - for that case, I think reasoning models could be good, because latency is something that I can tolerate there, since I'll see the result after the game is finished, so the model will have time to do the reasoning.

## On Coosing the Model 

- So, yeah, when choosing the model, it really depends on the use case in our application.
- There is also this quote from the book: ...
- And this is also shows why it's important to have evals in place.
- to know if this model is suitable or not for our app, we can just run the evals and do the comparison between models there.

## Update models (SKIP)

- lastly, there is research here from Amplify.
  - How often do you update / change your models.   
  - The majority here is Monthly and Quarterly.
- This is, once again, shows why evals is important, because every time we changes the models, we can just try to run the dataset that we have.
- We can see and compare the capability, the latency, and the cost there.

# 4. Prompt Engineering

- Now we move on to prompt engineering.
- I think this is the one that we are most familiar with.
- This is the definition.
- This is also the quote from the book.
- This is some tips from Anthropics docs.
  - I think we can go there.

## Anthropic Docs

- GO TO CLAUDE 4 BEST PRACTICES IN PROMPT ENGINEERING.
- This is specifically for Anthropic model, but I think it generally useful also for the others models.
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
  - BACK TO: OPENAI DOCS (PROMPTING => PROMPT ENGINEERING => FEW-SHOT LEARNING) => it's also use XML TAGS
- Epilog
  - There are a lot of others tips here related to prompt engineering.
  - BACK TO SLIDE.
- The book itself mention some tips that I include here:
  - Not all parts of prompt are equal
  - why examples
  - number of examples
  - json format
- JSON FORMAT
  - This book discuss about Structured Output.
  - How if we want the output of the LLM follow certain format.
  - But for this case, the library that I use is solved that issue.
- GO TO AI SDK
  - I use this AI SDK from Vercel.
  - This is really streamlining my process of building AI application.
  - One of the benefit is I can changing the model provider, like OpenAI, Anthropic, Google Gemini, without changing my code too much.
  - The other benefit is I can just specify the schema here, so the LLM ouput will always follow this structured.
  - For example.
  - I found that in 99% of the case, it works as expected.
  - So yeah, I think this is really helpful.

# 5. Tool Calling

- This is short section.

## CLAUDE.AI TOOL CALLING

- Here I have a screenshot from my chat with Claude.
- I think is conversations shows why tool calling is really helpful.
- here I explicitly tell Claude to not use any tool calling on doing this multiplication.
  - And it give me wrong answers
- And when I tell Claude to use tool calling, it will use this Code Execution tool.
  - It will use JS code here.
  - And as we can see, the result is now correct.
- I explicitly said not to use tool calling, because the model quite good enough to figure out that this task need tool calling
  - So i explicitly said don't, to make this comparison.

## KARPATHY

- Why it give the wrong answer here.
- I think the explanation from Andrej Karpathy, the cofounder of OpenAI, is really good.
- We can think LLM as something that consume the whole internet.
- So, when it tries to answer this question, without using tool calling.
- It's similar with: When I read a book, and then someone ask me, about the content of the book.
- If I don't use any tool calling, I will answer the question based on what is in my head.
- But I can use tool calling, I can, wait a minute, let me check the book again, or let me check the internet about that, etc.
- So this illustrate why tool calling is powerful on increasing the accuracy of LLM ouput.

## Tools Calling Definition

- I think we can skip this.

## Tools Calling (OPENAI DOCS)

- This is the list of all tool calling that OpenAI model can do.
  - READ
- For my app, I can imagine Function Calling and Web Search will be useful.
- Function Calling
  - READ
  - I can imagine, because as we see previously (LANGFUSE), i got this latitude and langitude right.
  - So maybe I can create a custom function in my code, that call GoogleMaps API, so I can get the exact City or the exact Province of this location. 
    - So it's no longer prediction.
    - So we can remove the non-deterministic part in this section.
- Web Search
  - READ
  - I can imagine, the LLM can use the web search to verify the correctness of all of this statement.
  - Because sometimes, here, the AI give me something that just didn't make sense.
- CAT story
  - So one day, I got Sri Lanka => look MAP => south of India.
  - In the image, there is a road, and on the side of the road.
  - And in this section here, it said that " Cat in Sri Lanka usually look healthier than the cat in India"
  - And I'm not really sure about that.
  - I google it, and didn't find any information about that.
  - So yeah, maybe, I can use tool calling to increase the factual correctness of the statements in this section.
- So that's about tool calling

# 6. Beyond This Session

- So yeah, there are a lot of things that I can't discuss in this sharing session.
- Partially because I don't have much time.
- But mostly because my understanding of these topic still limited.
- So yeah, basically, this sharing session only scratching the surface about AI Engineering.

# Closing

- So if you are interested on going deeper on AI Engineering, I found that this book is quite helpful.
- And lastly, if any of you have interest in geography or just curious about the world. Please try the game, it's really fun, at least for me.

# Reference

- https://blog.duolingo.com/how-duolingo-experts-work-with-ai/
- https://www.amplifypartners.com/blog-posts/the-2025-ai-engineering-report

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