# Opening

# 0. AI Engineering

- Thank you. 
- AI Engineering.

- Loosely based on AI Engineering book (Chip Huyen)
- Integrating AI
- Consider when integrating AI
- Popular Book

- 5 Sections
- Disclaimer: not expert

# 1. Demo Session

## TITLE PAGE

- Apply Knowledge
- Prototype
- Extension

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
- Intermediate. Easy round.

## playing

- Collecting the clues.
- Road lines, Poles, Sun, House, Infra, etc.
- 5 to 7 minutes later …

## how to get better

- 2 ways of getting better at the game
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
  - Me.
  - Evaluate the output here. 
  - Give the thumbs up, thumbs down, or give the rating to the output here. and send it to the system.
  - And the owner of the app, can collect the data, and make improvements based on those feedback.
  - Costly and time-consuming.
- Code-based evals
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

## LLM-as-a-Judge

- Next 2 pages.
- These last 2 pages just illustrate that using LLM as a judge is the common practices, a lot of startups do that.
- And these 2 researches show that LLM-as-a-judge is reliable tools.

## What's Inside Langfuse

- Langfuse
  - As we see previously, I use Langfuse here as an observability tools.
  - And I also use this to do the evals.
- Trace
  - This lists all the rounds that have been analyzed by my extension.
  - CLICK ONE.
  - 
- LLM call
  - Input - Output
  - Latency
  - Costs
  - Models

## Langfuse Evals

- Evals
- Dataset

# 3. Choosing A Model



# 4. Prompt Engineering



# 5. Tool Calling

# 6. Beyond This Session

# Closing


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