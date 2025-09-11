# Opening

# 0. AI Engineering

## 1st Page

- Thank you
- AI Engineering

## Intro Page

- Loosely based on AI Engineering book (Chip Huyen)
- Integrating AI
- Consider when integrating AI
- Popular book

## Sharing Session

- 5 Sections
- Reverse order
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
- Some of you know
  - Learner
- Simple
  - Spawn => Guess

## Let's Play

- 3 Game Mode
  - No Move
- More context
- Let's play!
- 5-7 minutes
- Still learning
  - Easy round
  - Decent guess
- RUN STOPWATCH!

## While Playing

- Collecting the clues
- Sun, road lines, poles, house, infra, vibes, etc
- 50% of my power = Sun
- 5 to 7 minutes later …

## How to Get Better

- 2 ways to getting better at the game:
  - plays a lot => took a lot of time
  - plonkit => tedious and less fun
- Combine both!
- OPEN THE EXTENSION!

## Possible Country and Possible Region

- READ
- See Maps

## Visual Clue

- What this visual clues is doing:
  - look image,
  - relevant clues,
- READ
- In tool calling
  - why not 100% accurate
- Useful

## Trivia

- For fun

## Verified Metas

- First 4 sections => skimming
- Pay most attention: Verified Metas
  - What I did:
    - Scrape plonkit
    - Among many clues => LLM will filter
  - Instead of tediously read, just play
  - Contextual Learning
- Play some rounds
  - false positive

## Demo Session Epilog

- Combine playing and learning
  - Keep playing, but also learning
- That's the demo session

# 2. Eval and Observability

## Title Page

- Chapter 3 and 4
- Characteristic of AI
  - same input, different output,
  - non-deterministic or probabilistic nature,
  - vs traditional software (same input, same output)
- Because of that non-deterministic nature 
  - Challenges: How we can make sure that the AI output in our application is working as expected?

## Evals Definition

- That's what evals is
- Evals in geowizr

## Evals Hot Topic

- To tweet page
- To the research page
- To the AI bottleneck page
- Evals is hot topic
  - We still figure out
  - Tools and practice will keep changing

## How To Do Evals

- Human evals (SKIP)
- Code-based evals (SKIP)
- LLM-based Evals (GO)
  - getting more popular
  - READ
  - GO TO GEOWIZR

## LLM-as-a-Judge

- Next 2 pages
  - Common practices
  - Reliable

## LANGFUSE Trace

- Langfuse
  - Observability
  - Evals
- Trace
  - List all rounds
- CLICK ONE
  - 2 Parallel LLM Request
  - input-output, latency, token usage, cost, model
- Helpful

## Langfuse Evals

- 1 Evals on Visual Clues
  - Comparison check evals
  - 1, 0, 0.5
- GO TO llm-as-a-judge

## Langfuse LLM-as-a-judge

- READ PROMPT
- Important Skill
  - Thailand vs Cambodia
  - North Brazil vs South Brazil
- The idea:
  - put evals on the place that we care about

## Langfuse Dataset

- Run evals on each request
  - not ideal in prod
  - costly
- Prepare the datasets
- DATASETS MENU
  - READ

## Langfuse Dataset Geowizr

- 100-200 locations
- "Add to datasets" button
- Run every week, or every change
  - Is the AI Output still work as we want?
- Depth and broad topic

# 3. Choosing A Model

## Amplify Common Models

- Amplify
- TOP 3
- Personal
  - Anthropic
- Geowizr
  - OPENAI

## Capabilities, Speed or Latency, Costs

- Anthropic, similar
- Capabilities, Latency, Costs

## Costs

- Not really care
- Focus on capabilities and latency
  - find balance

## Capabilities and Latency (OpenAI Docs)

- Reasoning model is better
- GO TO OPENAI DOCS
  - CHECK REASONING MODELS PAGE
  - READ 1ST PARAGRAPH
- Higher latency

## Geowizr Latency Issue

- Need instant response
- Using reasoning models on competitive duels
- Find the one that best for your use case

## On Choosing the Model 

- Quote
- Why evals
  - compare evals score, latency, costs

# 4. Prompt Engineering

## Title Page

- Most familiar with

## Prompt Engineering Definitions

- Can skip

## Prompt Engineering Claude

- Anthropic docs
- GO TO: CLAUDE 4 BEST PRACTICES IN PROMPT ENGINEERING

## Prompt Engineering Tips

- Be explicit
  - READ
  - GO TO: BE CLEAR AND DIRECT PAGE
- Add context
  - READ
- Examples
  - GO TO => USE EXAMPLES (MULTI-SHOT PROMPTING)
  - CHECK THE EXAMPLE
  - GO TO: OPENAI DOCS (PROMPTING => PROMPT ENGINEERING => FEW-SHOT LEARNING)
- XML Tags
  - READ
  - BACK TO: OPENAI DOCS (PROMPTING => PROMPT ENGINEERING => FEW-SHOT LEARNING) 
- Epilog
  - BACK TO SLIDE

## Prompt Tips from The Book

- Tips from the books
  - skimming

# 5. Tool Calling

## CLAUDE.AI TOOL CALLING

- Screenshot
  - Why tool calling powerful
- Point: tool calling is powerful
  - reduce hallucinations
  - increasing the accuracy

## KARPATHY

- Why wrong answer?
- Karpathy
- Analogy with people reading book

## Tools Calling Definition

- Skip.

## Tools Calling (OPENAI DOCS)

- OpenAI
- GeoWizr
  - Function Calling
  - Web Search

## Function Calling

- READ
- Latitude, Longitude
- Still inaccurate
- GoogleMaps API
  - Exact city/province
- Remove non-deterministic part

## Web Search

- READ
- Verify factual correctness
- Cat Story
- That's tool calling
  - Reduce hallucinations or inaccuracy

# 6. Beyond This Session

- I can't discuss
  - time
  - limited understanding
- Scratching the surface

# Closing

- Going deeper
  - Read the book!
- Curious about the world
  - Try the game!
- Thank you!

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