# Chapter 3 - Evaluation Methodology

- As teams rush to adopt AI, many quickly realize that the biggest hurdle to bringing AI applications to reality is evaluation. For some applications, figuring out evaluation can take up the majority of the development effort
- evaluation has to be considered in the context of a whole system, not in isolation. Evaluation aims to mitigate risks and uncover opportunities. To mitigate risks, you first need to identify the places where your system is likely to fail and design your evaluation around them. Often, this may require redesigning your system to enhance visibility into its failures. Without a clear understanding of where your system fails, no amount of evaluation metrics or tools can make the system robust.
- Evaluating foundation models is especially challenging because they are open-ended, and I’ll cover best practices for how to tackle these. Using human evaluators remains a necessary option for many applications. However, given how slow and expensive human annotations can be, the goal is to automate the process. This book focuses on automatic evaluation, which includes both exact and subjective evaluation.
- The rising star of subjective evaluation is AI as a judge—the approach of using AI to evaluate AI responses. It’s subjective because the score depends on what model and prompt the AI judge uses. While this approach is gaining rapid traction in the industry, it also invites intense opposition from those who believe that AI isn’t trustworthy enough for this important task.

### Challenges of Evaluating Foundation Models

- There are multiple reasons why evaluating foundation models is more challenging than evaluating traditional ML models.
- evaluation can be so much more time-consuming for sophisticated tasks. You can no longer evaluate a response based on how it sounds. You’ll also need to fact-check, reason, and even incorporate domain expertise.
- With traditional ML, most tasks are close-ended. For example, a classification model can only output among the expected categories. To evaluate a classification model, you can evaluate its outputs against the expected outputs. If the expected output is category X but the model’s output is category Y, the model is wrong. However, for an open-ended task, for a given input, there are so many possible correct responses. It’s impossible to curate a comprehensive list of correct outputs to compare against.
- Details such as the model architecture, training data, and the training process can reveal a lot about a model’s strengths and weaknesses. Without those details, you can evaluate only a model by observing its outputs.
- At the same time, publicly available evaluation benchmarks have proven to be inadequate for evaluating foundation models
- A benchmark becomes saturated for a model once the model achieves the perfect score. With foundation models, benchmarks are becoming saturated fast
- When asked how they are evaluating their AI applications, many people told me that they just eyeballed the results. Many have a small set of go-to prompts that they use to evaluate models. The process of curating these prompts is ad hoc, usually based on the curator’s personal experience instead of based on the application’s needs. You might be able to get away with this ad hoc approach when getting a project off the ground, but it won’t be sufficient for application iteration. This book focuses on a systematic approach to evaluation.

### Understanding Language Modeling Metrics

- Therefore, a rough understanding of language modeling metrics can be quite helpful in understanding downstream performance
- In ML lingo, a language model learns the distribution of its training data. The better this model learns, the better it is at predicting what comes next in the training data, and the lower its training cross entropy. As with any ML model, you care about its performance not just on the training data but also on your production data. In general, the closer your data is to a model’s training data, the better the model can perform on your data.

Entropy

- Entropy measures how much information, on average, a token carries. The higher the entropy, the more information each token carries, and the more bits are needed to represent a token
- Intuitively, entropy measures how difficult it is to predict what comes next in a language. The lower a language’s entropy (the less information a token of a language carries), the more predictable that language. In our previous example, the language with only two tokens is easier to predict than the language with four (you have to predict among only two possible tokens compared to four). This is similar to how, if you can perfectly predict what I will say next, what I say carries no new information.

Cross Entropy

- When you train a language model on a dataset, your goal is to get the model to learn the distribution of this training data. In other words, your goal is to get the model to predict what comes next in the training data. A language model’s cross entropy on a dataset measures how difficult it is for the language model to predict what comes next in this dataset.
- A model’s cross entropy on the training data depends on two qualities:
  - The training data’s predictability, measured by the training data’s entropy 
  - How the distribution captured by the language model diverges from the true distribution of the training data
- A language model is trained to minimize its cross entropy with respect to the training data. If the language model learns perfectly from its training data, the model’s cross entropy will be exactly the same as the entropy of the training data. The KL divergence of Q with respect to P will then be 0. You can think of a model’s cross entropy as its approximation of the entropy of its training data.

Bits-per-Character and Bits-per-Byte

- Cross entropy tells us how efficient a language model will be at compressing text. If the BPB of a language model is 3.43, meaning it can represent each original byte (8 bits) using 3.43 bits, this language model can compress the original training text to less than half the text’s original size.

Perplexity

- Perplexity is the exponential of entropy and cross entropy
- If cross entropy measures how difficult it is for a model to predict the next token, perplexity measures the amount of uncertainty it has when predicting the next token. Higher uncertainty means there are more possible options for the next token.
- Popular ML frameworks, including TensorFlow and PyTorch, use nat (natural log) as the unit for entropy and cross entropy. 

Perplexity Interpretation and Use Cases

- As discussed, cross entropy, perplexity, BPC, and BPB are variations of language models’ predictive accuracy measurements. The more accurately a model can predict a text, the lower these metrics are.
- Remember that the more uncertainty the model has in predicting what comes next in a given dataset, the higher the perplexity.
- What’s considered a good value for perplexity depends on the data itself and how exactly perplexity is computed, such as how many previous tokens a model has access to
- More structured data gives lower expected perplexity
  - Therefore, the expected perplexity of a model on HTML code should be lower than the expected perplexity of a model on everyday text.
- The bigger the vocabulary, the higher the perplexity
  - Intuitively, the more possible tokens there are, the harder it is for the model to predict the next token.
- The longer the context length, the lower the perplexity
  - The more context a model has, the less uncertainty it will have in predicting the next token
- a perplexity of 3 means that this model has a 1 in 3 chance of predicting the next token correctly
- perplexity is useful in many parts of an AI engineering workflow
- First, perplexity is a good proxy for a model’s capabilities. If a model’s bad at predicting the next token, its performance on downstream tasks will also likely be bad
- Perplexity might not be a great proxy to evaluate models that have been post-trained using techniques like SFT and RLHF. Post-training is about teaching models how to complete tasks. As a model gets better at completing tasks, it might get worse at predicting the next tokens. A language model’s perplexity typically increases after post-training. Some people say that post-training collapses entropy. Similarly, quantization—a technique that reduces a model’s numerical precision and, with it, its memory footprint—can also change a model’s perplexity in unexpected ways.
- or a given model, perplexity is the lowest for texts that the model has seen and memorized during training. Therefore, perplexity can be used to detect whether a text was in a model’s training data. This is useful for detecting data contamination—if a model’s perplexity on a benchmark’s data is low, this benchmark was likely included in the model’s training data, making the model’s performance on this benchmark less trustworthy. This can also be used for deduplication of training data: e.g., add new data to the existing training dataset only if the perplexity of the new data is high.
- Perplexity and its related metrics help us understand the performance of the underlying language model, which is a proxy for understanding the model’s performance on downstream tasks. The rest of the chapter discusses how to measure a model’s performance on downstream tasks directly.

### Exact Evaluation

- When evaluating models’ performance, it’s important to differentiate between exact and subjective evaluation
- Exact evaluation produces judgment without ambiguity. For example, if the answer to a multiple-choice question is A and you pick B, your answer is wrong. There’s no ambiguity around that
- As you’ll see in the next section, AI as a judge is subjective. The evaluation result can change based on the judge model and the prompt.
- I’ll cover two evaluation approaches that produce exact scores: functional correctness and similarity measurements against reference data. Note that this section focuses on evaluating open-ended responses (arbitrary text generation) as opposed to close-ended responses (such as classification).
- This section focuses on open-ended evaluation because close-ended evaluation is already well understood.

Functional Correctness

- Functional correctness evaluation means evaluating a system based on whether it performs the intended functionality. For example, if you ask a model to create a website, does the generated website meet your requirements? If you ask a model to make a reservation at a certain restaurant, does the model succeed?
- Functional correctness is the ultimate metric for evaluating the performance of any application, as it measures whether your application does what it’s intended to do. However, functional correctness isn’t always straightforward to measure, and its measurement can’t be easily automated.
- Long before AI was used for writing code, automatically verifying code’s functional correctness was standard practice in software engineering. Code is typically validated with unit tests where code is executed in different scenarios to ensure that it generates the expected outputs. Functional correctness evaluation is how coding platforms like LeetCode and HackerRank validate the submitted solutions.

Similarity Measurements Against Reference Data

- If the task you care about can’t be automatically evaluated using functional correctness, one common approach is to evaluate AI’s outputs against reference data.
- Reference responses are also called ground truths or canonical responses. Metrics that require references are reference-based, and metrics that don’t are reference-free.
- Since this evaluation approach requires reference data, it’s bottlenecked by how much and how fast reference data can be generated. Reference data is generated typically by humans and increasingly by AIs. 
- Human-generated data can be expensive and time-consuming to generate, leading many to use AI to generate reference data instead. AI-generated data might still need human reviews, but the labor needed to review it is much less than the labor needed to generate reference data from scratch.
- There are four ways to measure the similarity between two open-ended texts:
  - Asking an evaluator to make the judgment whether two texts are the same 
  - Exact match: whether the generated response matches one of the reference responses exactly 
  - Lexical similarity: how similar the generated response looks to the reference responses 
  - Semantic similarity: how close the generated response is to the reference responses in meaning (semantics)
- Two responses can be compared by human evaluators or AI evaluators. AI evaluators are increasingly common and will be the focus of the next section.
- Despite the ease of use and flexibility of the AI as a judge approach, hand-designed similarity measurements are still widely used in the industry for their exact nature.

Exact match

- It’s considered an exact match if the generated response matches one of the reference responses exactly. Exact matching works for tasks that expect short, exact responses such as simple math problems, common knowledge queries, and trivia-style questions.
- Beyond simple tasks, exact match rarely works.
- It’s impossible to create an exhaustive set of possible responses for an input. For complex tasks, lexical similarity and semantic similarity work better.

Lexical similarity

- Lexical similarity measures how much two texts overlap. You can do this by first breaking each text into smaller tokens.
- In its simplest form, lexical similarity can be measured by counting how many tokens two texts have in common
- One way to measure lexical similarity is approximate string matching, known colloquially as fuzzy matching. It measures the similarity between two texts by counting how many edits it’d need to convert from one text to another, a number called edit distance
- Since the rise of foundation models, fewer benchmarks use lexical similarity. 
- A drawback of this method is that it requires curating a comprehensive set of reference responses. A good response can get a low similarity score if the reference set doesn’t contain any response that looks like it
- Not only that, but references can be wrong
- Low-quality reference data is one of the reasons that reference-free metrics were strong contenders for reference-based metrics in terms of correlation to human judgment
- Another drawback of this measurement is that higher lexical similarity scores don’t always mean better responses. For example, on HumanEval, a code generation benchmark, OpenAI found that BLEU scores for incorrect and correct solutions were similar

Semantic similarity

- Lexical similarity measures whether two texts look similar, not whether they have the same meaning.
- Semantic similarity aims to compute the similarity in semantics. This first requires transforming a text into a numerical representation, which is called an embedding
- For now, let’s assume that you have a way to transform texts into embeddings. The similarity between two embeddings can be computed using metrics such as cosine similarity. Two embeddings that are exactly the same have a similarity score of 1. Two opposite embeddings have a similarity score of –1.
- While I put semantic similarity in the exact evaluation category, it can be considered subjective, as different embedding algorithms can produce different embeddings. However, given two embeddings, the similarity score between them is computed exactly.
- Semantic textual similarity doesn’t require a set of reference responses as comprehensive as lexical similarity does. However, the reliability of semantic similarity depends on the quality of the underlying embedding algorithm. Two texts with the same meaning can still have a low semantic similarity score if their embeddings are bad. Another drawback of this measurement is that the underlying embedding algorithm might require nontrivial compute and time to run.

Introduction to Embedding

- Since computers work with numbers, a model needs to convert its input into numerical representations that computers can process. An embedding is a numerical representation that aims to capture the meaning of the original data.
- Here, I use a small vector as an example. In reality, the size of an embedding vector (the number of elements in the embedding vector) is typically between 100 and 10,000.
- The goal of the embedding algorithm is to produce embeddings that capture the essence of the original data.
- At a high level, an embedding algorithm is considered good if more-similar texts have closer embeddings, measured by cosine similarity or related metrics. The embedding of the sentence “the cat sits on a mat” should be closer to the embedding of “the dog plays on the grass” than the embedding of “AI research is super fun”.
- You can also evaluate the quality of embeddings based on their utility for your task. Embeddings are used in many tasks, including classification, topic modeling, recommender systems, and RAG.
- CLIP is trained using (image, text) pairs. The text corresponding to an image can be the caption or a comment associated with this image. For each (image, text) pair, CLIP uses a text encoder to convert the text to a text embedding, and an image encoder to convert the image to an image embedding. It then projects both these embeddings into a joint embedding space. The training goal is to get the embedding of an image close to the embedding of the corresponding text in this joint space.

### AI as A Judge

- The challenges of evaluating open-ended responses have led many teams to fall back on human evaluation. As AI has successfully been used to automate many challenging tasks, can AI automate evaluation as well? The approach of using AI to evaluate AI is called AI as a judge or LLM as a judge. An AI model that is used to evaluate other AI models is called an AI judge
- While the idea of using AI to automate evaluation has been around for a long time,16 it only became practical when AI models became capable of doing so, which was around 2020 with the release of GPT-3
- AI as a judge has become one of the most, if not the most, common methods for evaluating AI models in production. Most demos of AI evaluation startups I saw in 2023 and 2024 leveraged AI as a judge in one way or another. LangChain’s State of AI report in 2023 noted that 58% of evaluations on their platform were done by AI judges. AI as a judge is also an active area of research.

Why AI as a Judge?

- AI judges are fast, easy to use, and relatively cheap compared to human evaluators. They can also work without reference data, which means they can be used in production environments where there is no reference data.
- You can ask AI models to judge an output based on any criteria: correctness, repetitiveness, toxicity, wholesomeness, hallucinations, and more
- With the right prompt for the right model, you can get reasonably good judgments on a wide range of topics.
- Studies have shown that certain AI judges are strongly correlated to human evaluators. In 2023, Zheng et al. found that on their evaluation benchmark, MT-Bench, the agreement between GPT-4 and humans reached 85%, which is even higher than the agreement among humans (81%). AlpacaEval authors (Dubois et al., 2023) also found that their AI judges have a near perfect (0.98) correlation with LMSYS’s Chat Arena leaderboard, which is evaluated by humans.
- Not only can AI evaluate a response, but it can also explain its decision, which can be especially useful when you want to audit your evaluation results
- Its flexibility makes AI as a judge useful for a wide range of applications, and for some applications, it’s the only automatic evaluation option. Even when AI judgments aren’t as good as human judgments, they might still be good enough to guide an application’s development and provide sufficient confidence to get a project off the ground.

How to Use AI as a Judge

- There are many ways you can use AI to make judgments. For example, you can use AI to evaluate the quality of a response by itself, compare that response to reference data, or compare that response to another response
  - Evaluate the quality of a response by itself, given the original question,
  - Compare a generated response to a reference response to evaluate whether the generated response is the same as the reference response. This can be an alternative approach to human-designed similarity measurements,
  - Compare two generated responses and determine which one is better or predict which one users will likely prefer. This is helpful for generating preference data for post-training alignment (discussed in Chapter 2), test-time compute (discussed in Chapter2), and ranking models using comparative evaluation (discussed in the next section):
- A general-purpose AI judge can be asked to evaluate a response based on any criteria.
- It’s essential to remember that AI as a judge criteria aren’t standardized. Azure AI Studio’s relevance scores might be very different from MLflow’s relevance scores. These scores depend on the judge’s underlying model and prompt.
- How to prompt an AI judge is similar to how to prompt any AI application. In general, a judge’s prompt should clearly explain the following:
  - The task the model is to perform, such as to evaluate the relevance between a generated answer and the question. 
  - The criteria the model should follow to evaluate, such as “Your primary focus should be on determining whether the generated answer contains sufficient information to address the given question according to the ground truth answer”. The more detailed the instruction, the better. 
  - The scoring system, which can be one of these:
    - Classification, such as good/bad or relevant/irrelevant/neutral. 
    - Discrete numerical values, such as 1 to 5. Discrete numerical values can be considered a special case of classification, where each class has a numerical interpretation instead of a semantic interpretation. 
    - Continuous numerical values, such as between 0 and 1, e.g., when you want to evaluate the degree of similarity.
- Language models are generally better with text than with numbers. It’s been reported that AI judges work better with classification than with numerical scoring systems. 
- For numerical scoring systems, discrete scoring seems to work better than continuous scoring. Empirically, the wider the range for discrete scoring, the worse the model seems to get. Typical discrete scoring systems are between 1 and 5.
- Prompts with examples have been shown to perform better. If you use a scoring system between 1 and 5, include examples of what a response with a score of 1, 2, 3, 4, or 5 looks like, and if possible, why a response receives a certain score. Best practices for prompting are discussed in Chapter 5.
- Here’s part of the prompt used for the criteria relevance by Azure AI Studio. It explains the task, the criteria, the scoring system, an example of an input with a low score, and a justification for why this input has a low score. Part of the prompt was removed for brevity.
```markdown
Your task is to score the relevance between a generated answer and the question
based on the ground truth answer in the range between 1 and 5, and please also 
provide the scoring reason.

Your primary focus should be on determining whether the generated answer
contains sufficient information to address the given question according to the 
ground truth answer. …

If the generated answer contradicts the ground truth answer, it will receive a 
low score of 1-2.

For example, for the question "Is the sky blue?" the ground truth answer is "Yes, 
the sky is blue." and the generated answer is "No, the sky is not blue."

In this example, the generated answer contradicts the ground truth answer by 
stating that the sky is not blue, when in fact it is blue.

This inconsistency would result in a low score of 1–2, and the reason for the 
low score would reflect the contradiction between the generated answer and the 
ground truth answer.
```
- An AI judge is not just a model—it’s a system that includes both a model and a prompt. Altering the model, the prompt, or the model’s sampling parameters results in a different judge.

Limitations of AI as a Judge

- Despite the many advantages of AI as a judge, many teams are hesitant to adopt this approach. Using AI to evaluate AI seems tautological. The probabilistic nature of AI makes it seem too unreliable to act as an evaluator. AI judges can potentially introduce nontrivial costs and latency to an application. Given these limitations, some teams see AI as a judge as a fallback option when they don’t have any other way of evaluating their systems, especially in production.
  - Inconsistency
    - For an evaluation method to be trustworthy, its results should be consistent. Yet AI judges, like all AI applications, are probabilistic. The same judge, on the same input, can output different scores if prompted differently. Even the same judge, prompted with the same instruction, can output different scores if run twice. This inconsistency makes it hard to reproduce or trust evaluation results.
    - including evaluation examples in the prompt can increase the consistency of GPT-4 from 65% to 77.5%
    - However, they acknowledged that high consistency may not imply high accuracy—the judge might consistently make the same mistakes. On top of that, including more examples makes prompts longer, and longer prompts mean higher inference costs. In Zheng et al.’s experiment, including more examples in their prompts caused their GPT-4 spending to quadruple.
  - Criteria ambiguity
    - Unlike many human-designed metrics, AI as a judge metrics aren’t standardized, making it easy to misinterpret and misuse them
    - The faithfulness scores outputted by these three tools won’t be comparable. If, given a (context, answer) pair, MLflow gives a faithfulness score of 3, Ragas outputs 1, and LlamaIndex outputs NO, which score would you use?
    - An application evolves over time, but the way it’s evaluated ideally should be fixed. This way, evaluation metrics can be used to monitor the application’s changes. However, AI judges are also AI applications, which means that they also can change over time.
    - Imagine that last month, your application’s coherence score was 90%, and this month, this score is 92%. Does this mean that your application’s coherence has improved? It’s hard to answer this question unless you know for sure that the AI judges used in both cases are exactly the same. What if the judge’s prompt this month is different from the one last month? Maybe you switched to a slightly better-performing prompt or a coworker fixed a typo in last month’s prompt, and the judge this month is more lenient.
    - As a result, the application team might mistakenly attribute the changes in the evaluation results to changes in the application, rather than the changes in the judges.
    - Do not trust any AI judge if you can’t see the model and the prompt used for the judge.
    - Evaluation methods take time to standardize. As the field evolves and more guardrails are introduced, I hope that future AI judges will become a lot more standardized and reliable.
  - Increased costs and latency
    - You can use AI judges to evaluate applications both during experimentation and in production. Many teams use AI judges as guardrails in production to reduce risks, showing users only generated responses deemed good by the AI judge.
    - Using powerful models to evaluate responses can be expensive. If you use GPT-4 to both generate and evaluate responses, you’ll do twice as many GPT-4 calls, approximately doubling your API costs. If you have three evaluation prompts because you want to evaluate three criteria—say, overall response quality, factual consistency, and toxicity—you’ll increase your number of API calls four times.
    - You can reduce costs by using weaker models as the judges (see “What Models Can Act as Judges?”.) You can also reduce costs with spot-checking: evaluating only a subset of responses. Spot-checking means you might fail to catch some failures. The larger the percentage of samples you evaluate, the more confidence you will have in your evaluation results, but also the higher the costs. Finding the right balance between cost and confidence might take trial and error
    - Implementing AI judges in your production pipeline can add latency. If you evaluate responses before returning them to users, you face a trade-off: reduced risk but increased latency. The added latency might make this option a nonstarter for applications with strict latency requirements.
  - Biases of AI as a judge
    - AI judges tend to have self-bias, where a model favors its own responses over the responses generated by other models
    - GPT-4 favors itself with a 10% higher win rate, while Claude-v1 favors itself with a 25% higher win rate.
    - Many AI models have first-position bias. An AI judge may favor the first answer in a pairwise comparison or the first in a list of options. This can be mitigated by repeating the same test multiple times with different orderings or with carefully crafted prompts. The position bias of AI is the opposite of that of humans. Humans tend to favor the answer they see last, which is called recency bias.
    - Some AI judges have verbosity bias, favoring lengthier answers, regardless of their quality
    - Both Zheng et al. (2023) and Saito et al. (2023), however, discovered that GPT-4 is less prone to this bias than GPT-3.5, suggesting that this bias might go away as models become stronger.
    - Both Zheng et al. (2023) and Saito et al. (2023), however, discovered that GPT-4 is less prone to this bias than GPT-3.5, suggesting that this bias might go away as models become stronger.

What Models Can Act as Judges?

- One open question is whether the judge can be weaker than the model being judged. Some argue that judging is an easier task than generating. Anyone can have an opinion about whether a song is good, but not everyone can write a song. Weaker models should be able to judge the outputs of stronger models.
- At first glance, a stronger judge makes sense. Shouldn’t the exam grader be more knowledgeable than the exam taker? Not only can stronger models make better judgments, but they can also help improve weaker models by guiding them to generate better responses.
- if you already have access to the stronger model, why bother using a weaker model to generate responses? The answer is cost and latency. You might not have the budget to use the stronger model to generate all responses, so you use it to evaluate a subset of responses.
- Using the stronger model as a judge leaves us with two challenges. First, the strongest model will be left with no eligible judge. Second, we need an alternative evaluation method to determine which model is the strongest.
- Using a model to judge itself, self-evaluation or self-critique, sounds like cheating, especially because of self-bias. However, self-evaluation can be great for sanity checks. If a model thinks its own response is incorrect, the model might not be that reliable. Beyond sanity checks, asking a model to evaluate itself can nudge a model to revise and improve its responses
- One open question is whether the judge can be weaker than the model being judged. Some argue that judging is an easier task than generating. Anyone can have an opinion about whether a song is good, but not everyone can write a song. Weaker models should be able to judge the outputs of stronger models.
- Zheng et al. (2023) found that stronger models are better correlated to human preference, which makes people opt for the strongest models they can afford.
- Because there are many possible ways to use AI judges, there are many possible specialized AI judges. Here, I’ll go over examples of three specialized judges: reward models, reference-based judges, and preference models.
- Reward model
  - A reward model takes in a (prompt, response) pair and scores how good the response is given the prompt. Reward models have been successfully used in RLHF for many years.
- Reference-based judge
  - A reference-based judge evaluates the generated response with respect to one or more reference responses. This judge can output a similarity score or a quality score (how good the generated response is compared to the reference responses).
- Preference model
  - A preference model takes in (prompt, response 1, response 2) as input and outputs which of the two responses is better (preferred by users) for the given prompt. This is perhaps one of the more exciting directions for specialized judges. Being able to predict human preference opens up many possibilities. 
  - preference data is essential for aligning AI models to human preference, and it’s challenging and expensive to obtain. Having a good human preference predictor can generally make evaluation easier and models safer to use. 
  - Despite its limitations, the AI as a judge approach is versatile and powerful. Using cheaper models as judges makes it even more useful. Many of my colleagues, who were initially skeptical, have started to rely on it more in production. 
- AI as a judge is exciting, and the next approach we’ll discuss is just as intriguing. It’s inspired by game design, a fascinating field..


### Ranking Models with Comparative Evaluation

- Often, you evaluate models not because you care about their scores, but because you want to know which model is the best for you. What you want is a ranking of these models. You can rank models using either pointwise evaluation or comparative evaluation.
- With pointwise evaluation, you evaluate each model independently,22 then rank them by their scores. For example, if you want to find out which dancer is the best, you evaluate each dancer individually, give them a score, then pick the dancer with the highest score.
- With comparative evaluation, you evaluate models against each other and compute a ranking from comparison results. For the same dancing contest, you can ask all candidates to dance side-by-side and ask the judges which candidate’s dancing they like the most, and pick the dancer preferred by most judges.
- For responses whose quality is subjective, comparative evaluation is typically easier to do than pointwise evaluation. For example, it’s easier to tell which song of the two songs is better than to give each song a concrete score.
- In AI, comparative evaluation was first used in 2021 by Anthropic to rank different models. It also powers the popular LMSYS’s Chatbot Arena leaderboard that ranks models using scores computed from pairwise model comparisons from the community.
- A very important thing to keep in mind is that not all questions should be answered by preference. Many questions should be answered by correctness instead. Imagine asking the model “Is there a link between cell phone radiation and brain tumors?” and the model presents two options, “Yes” and “No”, for you to choose from. Preference-based voting can lead to wrong signals that, if used to train your model, can result in misaligned behaviors.
- Asking users to pick can also cause user frustration. Imagine asking the model a math question because you don’t know the answer, and the model gives you two different answers and asks you to pick the one you prefer. If you had known the right answer, you wouldn’t have asked the model in the first place.
- When collecting comparative feedback from users, one challenge is to determine what questions can be determined by preference voting and what shouldn’t be. Preference-based voting only works if the voters are knowledgeable in the subject
- Comparative evaluation shouldn’t be confused with A/B testing. In A/B testing, a user sees the output from one candidate model at a time. In comparative evaluation, a user sees outputs from multiple models at the same time.
- Given comparative signals, a rating algorithm is then used to compute a ranking of models. Typically, this algorithm first computes a score for each model from the comparative signals and then ranks models by their scores.
- Comparative evaluation is new in AI but has been around for almost a century in other industries. It’s especially popular in sports and video games. Many rating algorithms developed for these other domains can be adapted to evaluating AI models, such as Elo, Bradley–Terry, and TrueSkill
- A ranking is correct if, for any model pair, the higher-ranked model is more likely to win in a match against the lower-ranked model. If model A ranks higher than model B, users should prefer model A to model B more than half the time.
- Through this lens, model ranking is a predictive problem. We compute a ranking from historical match outcomes and use it to predict future match outcomes. Different ranking algorithms can produce different rankings, and there’s no ground truth for what the correct ranking is. The quality of a ranking is determined by how good it is in predicting future match outcomes.

Challenges of Comparative Evaluation

- With pointwise evaluation, the heavy-lifting part of the process is in designing the benchmark and metrics to gather the right signals. Computing scores to rank models is easy. With comparative evaluation, both signal gathering and model ranking are challenging
- Scalability bottlenecks
  - Comparative evaluation is data-intensive.
  - They argued that human preference is not necessarily transitive. In addition, non-transitivity can happen because different model pairs are evaluated by different evaluators and on different prompts.
  - There’s also the challenge of evaluating new models. With independent evaluation, only the new model needs to be evaluated. With comparative evaluation, the new model has to be evaluated against existing models, which can change the ranking of existing models.
  - The scaling bottleneck can be mitigated with better matching algorithms
- Lack of standardization and quality control
  - One way to collect comparative signals is to crowdsource comparisons to the community the way LMSYS Chatbot Arena does. Anyone can go to the website, enter a prompt, get back two responses from two anonymous models, and vote for the better one. Only after voting is done are the model names revealed.
  - If a public leaderboard doesn’t support sophisticated context construction, such as augmenting the context with relevant documents retrieved from your internal databases, its ranking won’t reflect how well a model might work for your RAG system
  - The ability to generate good responses is different from the ability to retrieve the most relevant documents.
  - Another option is to incorporate comparative evaluation into your products and let users evaluate models during their workflows. For example, for the code generation task, you can suggest users two code snippets inside the user’s code editor and let them pick the better one. Many chat applications are already doing this. However, as mentioned previously, the user might not know which code snippet is better, since they’re not the expert.
  - Some teams prefer AI to human evaluators. AI might not be as good as trained human experts but it might be more reliable than random internet users.
- From comparative performance to absolute performance
  - For many applications, we don’t necessarily need the best possible models. We need a model that is good enough. Comparative evaluation tells us which model is better. It doesn’t tell us how good a model is or whether this model is good enough for our use case
  - Several people have told me that in their experience, a 1% change in the win rate can induce a huge performance boost in some applications but just a minimal boost in other applications.
  - When deciding to swap out A for B, human preference isn’t everything. We also care about other factors like cost. Not knowing what performance boost to expect makes it hard to do the cost–benefit analysis. If model B costs twice as much as A, comparative evaluation isn’t sufficient to help us determine if the performance boost from B will be worth the added cost.

The Future of Comparative Evaluation

- Given so many limitations of comparative evaluation, you might wonder if there’s a future to it. There are many benefits to comparative evaluation. First, as discussed in “Post-Training”, people have found that it’s easier to compare two outputs than to give each output a concrete score. As models become stronger, surpassing human performance, it might become impossible for human evaluators to give model responses concrete scores
- Second, comparative evaluation aims to capture the quality we care about: human preference. It reduces the pressure to have to constantly create more benchmarks to catch up with AI’s ever-expanding capabilities. Unlike benchmarks that become useless when model performance achieves perfect scores, comparative evaluations will never get saturated as long as newer, stronger models are introduced.
- Comparative evaluation is relatively hard to game, as there’s no easy way to cheat, like training your model on reference data. For this reason, many trust the results of public comparative leaderboards more than any other public leaderboards.

### Summary

- The stronger AI models become, the higher the potential for catastrophic failures, which makes evaluation even more important. At the same time, evaluating open-ended, powerful models is challenging. These challenges make many teams turn toward human evaluation. Having humans in the loop for sanity checks is always helpful, and in many cases, human evaluation is essential. However, this chapter focused on different approaches to automatic evaluation.
- This chapter then shifted the focus to the different approaches to evaluate open-ended responses, including functional correctness, similarity scores, and AI as a judge. The first two evaluation approaches are exact, while AI as a judge evaluation is subjective.
- Unlike exact evaluation, subjective metrics are highly dependent on the judge. Their scores need to be interpreted in the context of what judges are being used. Scores aimed to measure the same quality by different AI judges might not be comparable. AI judges, like all AI applications, should be iterated upon, meaning their judgments change. This makes them unreliable as benchmarks to track an application’s changes over time. While promising, AI judges should be supplemented with exact evaluation, human evaluation, or both.
- 