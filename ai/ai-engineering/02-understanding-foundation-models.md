# Chapter 2 - Understanding Foundation Models

- In general, however, differences in foundation models can be traced back to decisions about training data, model architecture and size, and how they are post-trained to align with human preferences.
- Since models learn from data, their training data reveals a great deal about their capabilities and limitations
- Sampling is how a model chooses an output from all possible options. It is perhaps one of the most underrated concepts in AI. Not only does sampling explain many seemingly baffling AI behaviors, including hallucinations and inconsistencies, but choosing the right sampling strategy can also significantly boost a model’s performance with relatively little effort.
- Concepts covered in this chapter are fundamental for understanding the rest of the book.

### Training Data

- An AI model is only as good as the data it was trained on.
- If you want a model to improve on a certain task, you might want to include more data for that task in the training data.
- To address this issue, it’s crucial to curate datasets that align with your specific needs.
- While language- and domain-specific foundation models can be trained from scratch, it’s also common to finetune them on top of general-purpose models.
- a model trained with a smaller amount of high-quality data might outperform a model trained with a large amount of low-quality data

Domain-Specific Models

- Even though general-purpose foundation models can answer everyday questions about different domains, they are unlikely to perform well on domain-specific tasks, especially if they never saw these tasks during training.
- To train a model to perform well on these domain-specific tasks, you might need to curate very specific datasets.

- This section gave a high-level overview of how training data impacts a model’s performance. Next, let’s explore the impact of how a model is designed on its performance.

### Modeling

- Before training a model, developers need to decide what the model should look like. What architecture should it follow? How many parameters should it have?

Model Architecture

- As of this writing, the most dominant architecture for language-based foundation models is the transformer architecture
- attention mechanism
- seq2seq
  - In its most basic form, the encoder processes the input tokens sequentially, outputting the final hidden state that represents the input. The decoder then generates output tokens sequentially, conditioned on both the final hidden state of the input and the previously generated token
  - 2 problems:
    - First, the vanilla seq2seq decoder generates output tokens using only the final hidden state of the input. Intuitively, this is like generating answers about a book using the book summary. This limits the quality of the generated outputs.
    - Second, the RNN encoder and decoder mean that both input processing and output generation are done sequentially, making it slow for long sequences. If an input is 200 tokens long, seq2seq has to wait for each input token to finish processing before moving on to the next
- Transformer architecture
  - address seq2seq problems with the attention mechanism.
  - The attention mechanism allows the model to weigh the importance of different input tokens when generating each output token. This is like generating answers by referencing any page in the book.
  - With transformers, the input tokens can be processed in parallel, significantly speeding up input processing
  - While the transformer removes the sequential input bottleneck, transformer-based autoregressive language models still have the sequential output bottleneck.
- Inference for transformer-based language models, therefore, consists of two steps:
  - Prefill
    - The model processes the input tokens in parallel.
  - Decode
    - The model generates one output token at a time.
- As explored later in Chapter 9, the parallelizable nature of prefilling and the sequential aspect of decoding both motivate many optimization techniques to make language model inference cheaper and faster.
- Attention mechanism
  - At the heart of the transformer architecture is the attention mechanism
- Under the hood, the attention mechanism leverages key, value, and query vectors:
  - The query vector (Q): 
    - the current state of the decoder at each decoding step
    -  the person looking for information to create a summary.
  - Each key vector (K): 
    - previous token
    - If each previous token is a page in the book, each key vector is like the page number
    - at a given decoding step, previous tokens include both input tokens and previously generated tokens.
  - Each value vector (V):
    - the actual value of a previous token, as learned by the model.
    - Each value vector is like the page’s content.
- The attention mechanism computes how much attention to give an input token by performing a dot product between the query vector and its key vector. A high score means that the model will use more of that page’s content (its value vector) when generating the book’s summary. 
  - Because each previous token has a corresponding key and value vector, the longer the sequence, the more key and value vectors need to be computed and stored. This is one reason why it’s so hard to extend context length for transformer models.
- in general, each transformer block contains the attention module and the MLP (multi-layer perceptron) module:
  -Attention module: Each attention module consists of four weight matrices: query, key, value, and output projection.
  - MLP module
- Note that while the increased context length impacts the model’s memory footprint, it doesn’t impact the model’s total number of parameters.
- Modeling long sequences remains a core challenge in developing LLMs.

Model Size

- In general, increasing a model’s parameters increases its capacity to learn, resulting in better models.
- The number of parameters helps us estimate the compute resources needed to train and run this model. 
- A larger model can also underperform a smaller model if it’s not trained on enough data.
- When discussing model size, it’s important to consider the size of the data it was trained on. For most models, dataset sizes are measured by the number of training samples
- A better measurement is the number of tokens in the dataset.
- As of this writing, LLMs are trained using datasets in the order of trillions of tokens.
- The number of tokens in a model’s dataset isn’t the same as its number of training tokens. The number of training tokens measures the tokens that the model is trained on. If a dataset contains 1 trillion tokens and a model is trained on that dataset for two epochs—an epoch is a pass through the dataset—the number of training tokens is 2 trillion.
- While this section focuses on the scale of data, quantity isn’t the only thing that matters. Data quality and data diversity matter, too. Quantity, quality, and diversity are the three golden goals for training data
- What’s considered good utilization depends on the model, the workload, and the hardware. Generally, if you can get half the advertised performance, 50% utilization, you’re doing okay. Anything above 70% utilization is considered great.
- In summary, three numbers signal a model’s scale:
  - Number of parameters, which is a proxy for the model’s learning capacity.
  - Number of tokens a model was trained on, which is a proxy for how much a model learned.
  - Number of FLOPs, which is a proxy for the training cost.
- They found that larger language models are sometimes (only sometimes) worse on tasks that require memorization and tasks with strong priors.
- ...
  - Model performance depends on the model size and the dataset size.
  - Bigger models and bigger datasets require more compute.
  - Compute costs money.
- Given a compute budget, the rule that helps calculate the optimal model size and dataset size is called the Chinchilla scaling law
- for compute-optimal training, you need the number of training tokens to be approximately 20 times the model size.
- The model size and the number of training tokens should be scaled equally: for every doubling of the model size, the number of training tokens should also be doubled.
- Smaller models are easier to work with and cheaper to run inference on, which helped their models gain wider adoption
- The performance of a model depends heavily on the values of its hyperparameters
- A parameter can be learned by the model during the training process. A hyperparameter is set by users to configure the model and control how the model learns. Hyperparameters to configure the model include the number of layers, the model dimension, and vocabulary size. Hyperparameters to control how a model learns include batch size, number of epochs, learning rate, per-layer initial variance, and more.
- While it’s hard to answer these questions, there are already two visible bottlenecks for scaling: training data and electricity.
- Once the publicly available data is exhausted, the most feasible paths for more human-generated training data is proprietary data.
- the next critical set of design choices: how to align models with human preferences.

### Post-Training

- in general, post-training consists of two steps:
  - Supervised finetuning (SFT): Finetune the pre-trained model on high-quality instruction data to optimize models for conversations instead of completion.
  - Preference finetuning: Further finetune the model to output responses that align with human preference. Preference finetuning is typically done with reinforcement learning (RL)
- For language-based foundation models, pre-training optimizes token-level quality, where the model is trained to predict the next token accurately. However, users don’t care about token-level quality—they care about the quality of the entire response. Post-training, in general, optimizes the model to generate responses that users prefer. Some people compare pre-training to reading to acquire knowledge, while post-training is like learning how to use that knowledge.
- As post-training consumes a small portion of resources compared to pre-training (InstructGPT used only 2% of compute for post-training and 98% for pre-training), you can think of post-training as unlocking the capabilities that the pre-trained model already has but are hard for users to access via prompting alone.
- Note that a combination of pre-training, SFT, and preference finetuning is the popular solution for building foundation models today, but it’s not the only solution

Supervised Finetuning

- pre-trained model is likely optimized for completion rather than conversing
- demonstration data.
- behavior cloning: you demonstrate how the model should behave, and the model clones this behavior.
- good labelers are important for AIs to learn how to conduct intelligent conversations
- Companies, therefore, often use highly educated labelers to generate demonstration data
- Technically, you can train a model from scratch on the demonstration data instead of finetuning a pre-trained model, effectively eliminating the self-supervised pre-training step. However, the pre-training approach often has returned superior results.

Preference Finetuning

- Demonstration data teaches the model to have a conversation but doesn’t teach the model what kind of conversations it should have. For example, if a user asks the model to write an essay about why one race is inferior or how to hijack a plane, should the model comply?
- The goal of preference finetuning is to get AI models to behave according to human preference.
- given the ambitious nature of the goal, the solution we have today is complicated. The earliest successful preference finetuning algorithm, which is still popular today, is RLHF
- RLHF:
  - Train a reward model that scores the foundation model’s outputs.
  - Optimize the foundation model to generate responses for which the reward model will give maximal scores.
- While RLHF is still used today, newer approaches like DPO
- RLHF, while more complex than DPO, provides more flexibility to tweak the model
- RLHF relies on a reward model. Given a pair of (prompt, response), the reward model outputs a score for how good the response is. Training a model to score a given input is a common ML task. The challenge, similar to that of SFT, is to obtain reliable data.
- Evaluating each sample independently is also called pointwise evaluation.
- An easier task is to ask labelers to compare two responses and decide which one is better.
- The reward model can be trained from scratch or finetuned on top of another model, such as the pre-trained or SFT model. Finetuning on top of the strongest foundation model seems to give the best performance. Some people believe that the reward model should be at least as powerful as the foundation model to be able to score the foundation model’s responses. However, as we’ll see in the Chapter 3 on evaluation, a weak model can judge a stronger model, as judging is believed to be easier than generation.
- Empirically, RLHF and DPO both improve performance compared to SFT alone. However, as of this writing, there are debates on why they work. As the field evolves, I suspect that preference finetuning will change significantly in the future
- Both SFT and preference finetuning are steps taken to address the problem created by the low quality of data used for pre-training. If one day we have better pre-training data or better ways to train foundation models, we might not need SFT and preference at all.
- Some companies find it okay to skip reinforcement learning altogether. For example, Stitch Fix and Grab find that having the reward model alone is good enough for their applications. They get their models to generate multiple outputs and pick the ones given high scores by their reward models. This approach, often referred to as the best of N strategy, leverages how a model samples outputs to improve its performance

### Sampling

- A model constructs its outputs through a process known as sampling.
- Sampling makes AI’s outputs probabilistic. Understanding this probabilistic nature is important for handling AI’s behaviors, such as inconsistency and hallucination. This section ends with a deep dive into what this probabilistic nature means and how to work with it.

Sampling Fundamentals

- Given an input, a neural network produces an output by first computing the probabilities of possible outcomes
- For a language model, to generate the next token, the model first computes the probability distribution over all tokens in the vocabulary
- When working with possible outcomes of different probabilities, a common strategy is to pick the outcome with the highest probability. Always picking the most likely outcome = is called greedy sampling. This often works for classification tasks. For example, if the model thinks that an email is more likely to be spam than not spam, it makes sense to mark it as spam. However, for a language model, greedy sampling creates boring outputs. Imagine a model that, for whatever question you ask, always responds with the most common words.
- Instead of always picking the next most likely token, the model can sample the next token according to the probability distribution over all possible values. Given the context of “My favorite color is …” as shown in Figure 2-14, if “red” has a 30% chance of being the next token and “green” has a 50% chance, “red” will be picked 30% of the time, and “green” 50% of the time.

Sampling Strategies

- The right sampling strategy can make a model generate responses more suitable for your application
- For example, one sampling strategy can make the model generate more creative responses, whereas another strategy can make its generations more predictable
- Temperature
  - One problem with sampling the next token according to the probability distribution is that the model can be less creative.
  - To redistribute the probabilities of the possible values, you can sample with a temperature. Intuitively, a higher temperature reduces the probabilities of common tokens, and as a result, increases the probabilities of rarer tokens. This enables models to create more creative responses.
  - The higher the temperature, the less likely it is that the model is going to pick the most obvious value (the value with the highest logit), making the model’s outputs more creative but potentially less coherent. The lower the temperature, the more likely it is that the model is going to pick the most obvious value, making the model’s output more consistent but potentially more boring.
  - A temperature of 0.7 is often recommended for creative use cases, as it balances creativity and predictability, but you should experiment and find the temperature that works best for you.
  - It’s common practice to set the temperature to 0 for the model’s outputs to be more consistent. Technically, temperature can never be 0—logits can’t be divided by 0. In practice, when we set the temperature to 0, the model just picks the token with the largest logit,25 without doing logit adjustment and softmax calculation.
  - A common debugging technique when working with an AI model is to look at the probabilities this model computes for given inputs. For example, if the probabilities look random, the model hasn’t learned much.
  - Logprobs, short for log probabilities, are probabilities in the log scale. Log scale is preferred when working with a neural network’s probabilities because it helps reduce the underflow problem
  - logprobs are useful for building applications (especially for classification), evaluating applications, and understanding how models work under the hood. However, as of this writing, many model providers don’t expose their models’ logprobs, or if they do, the logprobs API is limited.27 The limited logprobs API is likely due to security reasons as a model’s exposed logprobs make it easier for others to replicate the model.
- Top-k
  - Top-k is a sampling strategy to reduce the computation workload without sacrificing too much of the model’s response diversity.
  - A smaller k value makes the text more predictable but less interesting, as the model is limited to a smaller set of likely words.
- Top-p
  - In top-k sampling, the number of values considered is fixed to k. However, this number should change depending on the situation.
  - Top-p, also known as nucleus sampling, allows for a more dynamic selection of values to be sampled from. In top-p sampling, the model sums the probabilities of the most likely next values in descending order and stops when the sum reaches p. Only the values within this cumulative probability are considered.
  - Common values for top-p (nucleus) sampling in language models typically range from 0.9 to 0.95. A top-p value of 0.9, for example, means that the model will consider the smallest set of values whose cumulative probability exceeds 90%.
  - In theory, there don’t seem to be a lot of benefits to top-p sampling. However, in practice, top-p sampling has proven to work well, causing its popularity to rise.
- Stopping condition

Test Time Compute

- One simple way to improve a model’s response quality is test time compute: instead of generating only one response per query, you generate multiple responses to increase the chance of good responses.
- you randomly generate multiple outputs and pick one that works best
- However, you can also be more strategic about how to generate multiple outputs. For example, instead of generating all outputs independently, which might include many less promising candidates, you can use beam search to generate a fixed number of most promising candidates (the beam) at each step of sequence generation.
- To pick the best output, you can either show users multiple outputs and let them choose the one that works best for them, or you can devise a method to select the best one.
- They found that using a verifier significantly boosted the model performance. In fact, the use of verifiers resulted in approximately the same performance boost as a 30× model size increase
- DeepMind further proves the value of test time compute, arguing that scaling test time compute (e.g., allocating more compute to generate more outputs during inference) can be more efficient than scaling model parameters
- You can also use application-specific heuristics to select the best response. For example, if your application benefits from shorter responses, you can pick the shortest candidate. If your application converts natural language to SQL queries, you can get the model to keep on generating outputs until it generates a valid SQL query.
- A model is considered robust if it doesn’t dramatically change its outputs with small variations in the input. The less robust a model is, the more you can benefit from sampling multiple outputs
- For one project, we used AI to extract certain information from an image of the product. We found that for the same image, our model could read the information only half of the time. For the other half, the model said that the image was too blurry or the text was too small to read. However, by trying three times with each image, the model was able to extract the correct information for most images.


## Structured Outpus

- Structured outputs are crucial for the following two scenarios:
  - Tasks requiring structured outputs. The most common category of tasks in this scenario is semantic parsing
  - Tasks whose outputs are used by downstream applications. In this scenario, the task itself doesn’t need the outputs to be structured, but because the outputs are used by other applications, they need to be parsable by these applications.
- Note that an API’s JSON mode typically guarantees only that the outputs are valid JSON—not the content of the JSON objects
- You can guide a model to generate structured outputs at different layers of the AI stack: prompting, post-processing, test time compute, constrained sampling, and finetuning. The first three are more like bandages. They work best if the model is already pretty good at generating structured outputs and just needs a little nudge. For intensive treatment, you need constrained sampling and finetuning.
- Prompting
  - Prompting is the first line of action for structured outputs. You can instruct a model to generate outputs in any format. However, whether a model can follow this instruction depends on the model’s instruction-following capability (discussed in Chapter4), and the clarity of the instruction (discussed in Chapter5).
  - To increase the percentage of valid outputs, some people use AI to validate and/or correct the output of the original prompt. This is an example of the AI as a judge approach discussed in Chapter 3. This means that for each output, there will be at least two model queries: one to generate the output and one to validate it. While the added validation layer can significantly improve the validity of the outputs, the extra cost and latency incurred by the extra validation queries can make this approach too expensive for some.
- Post-processing
  - Post-processing is simple and cheap but can work surprisingly well.
  - When I started working with foundation models, I noticed the same thing. A model tends to repeat similar mistakes across queries. This means if you find the common mistakes a model makes, you can potentially write a script to correct them.
  - LinkedIn’s defensive YAML parser increased the percentage of correct YAML outputs from 90% to 99.99%
  - JSON and YAML are common text formats. LinkedIn found that their underlying model, GPT-4, worked with both, but they chose YAML as their output format because it is less verbose, and hence requires fewer output tokens than JSON
  - Post-processing works only if the mistakes are easy to fix. This usually happens if a model’s outputs are already mostly correctly formatted, with occasional small errors.
- Constrained sampling
  - Constraint sampling is a technique for guiding the generation of text toward certain constraints. It is typically followed by structured output tools.
  - At a high level, to generate a token, the model samples among values that meet the constraints.
  - Building out that grammar and incorporating it into the sampling process is nontrivial. Because each output format—JSON, YAML, regex, CSV, and so on—needs its own grammar, constraint sampling is less generalizable. Its use is limited to the formats whose grammars are supported by external tools or by your team. Grammar verification can also increase generation latency
  - Some are against constrained sampling because they believe the resources needed for constrained sampling are better invested in training models to become better at following instructions.
- Finetuning
  - Finetuning a model on examples following your desirable format is the most effective and general approach to get models to generate outputs in this format.34 It can work with any expected format. While simple finetuning doesn’t guarantee that the model will always output the expected format, it is much more reliable than prompting.
  - End-to-end training requires more resources, but promises better performance.
  - We need techniques for structured outputs because of the assumption that the model, by itself, isn’t capable of generating structured outputs. However, as models become more powerful, we can expect them to get better at following instructions. I suspect that in the future, it’ll be easier to get models to output exactly what we need with minimal prompting, and these techniques will become less important.

The Probabilistic Nature of AI

- The way AI models sample their responses makes them probabilistic.
- The opposite of probabilistic is deterministic, when the outcome can be determined without any random variation.
- This probabilistic nature can cause inconsistency and hallucinations. Inconsistency is when a model generates very different responses for the same or slightly different prompts. Hallucination is when a model gives a response that isn’t grounded in facts.
- Anything with a non-zero probability, no matter how far-fetched or wrong, can be generated by AI
- This probabilistic nature makes AI great for creative tasks. What is creativity but the ability to explore beyond the common paths—to think outside the box? AI is a great sidekick for creative professionals. It can brainstorm limitless ideas and generate never-before-seen designs. However, this same probabilistic nature can be a pain for everything else.
- Inconsistency
  - Model inconsistency manifests in two scenarios:
    - Same input, different outputs: Giving the model the same prompt twice leads to two very different responses.
    - Slightly different input, drastically different outputs: Giving the model a slightly different prompt, such as accidentally capitalizing a letter, can lead to a very different output.
  - Inconsistency can create a jarring user experience. In human-to-human communication, we expect a certain level of consistency.
  - users expect a certain level of consistency when communicating with AI.
  - For the same input, different outputs scenario, there are multiple approaches to mitigate inconsistency. You can cache the answer so that the next time the same question is asked, the same answer is returned. You can fix the model’s sampling variables, such as temperature, top-p, and top-k values, as discussed earlier. You can also fix the seed variable, which you can think of as the starting point for the random number generator used for sampling the next token.
  - Even if you fix all these variables, however, there’s no guarantee that your model will be consistent 100% of the time
  - Fixing the output generation settings is a good practice, but it doesn’t inspire trust in the system. Imagine a teacher who gives you consistent scores only if that teacher sits in one particular room. If that teacher sits in a different room, that teacher’s scores for you will be wild.
  - The second scenario—slightly different input, drastically different outputs—is more challenging. Fixing the model’s output generation variables is still a good practice, but it won’t force the model to generate the same outputs for different inputs. It is, however, possible to get models to generate responses closer to what you want with carefully crafted prompts (discussed in Chapter5) and a memory system (discussed in Chapter6).
- Hallucination
  - Hallucinations are fatal for tasks that depend on factuality.
  - The first hypothesis, originally expressed by Ortega et al. at DeepMind in 2021, is that a language model hallucinates because it can’t differentiate between the data it’s given and the data it generates.
  - The DeepMind paper showed that hallucinations can be mitigated by two techniques. The first technique comes from reinforcement learning, in which the model is made to differentiate between user-provided prompts (called observations about the world in reinforcement learning) and tokens generated by the model (called the model’s actions). The second technique leans on supervised learning, in which factual and counterfactual signals are included in the training data.
  - The second hypothesis is that hallucination is caused by the mismatch between the model’s internal knowledge and the labeler’s internal knowledge. This view was first argued by Leo Gao, an OpenAI researcher.
  - Based on the assumption that a foundation model knows what it knows, some people try to reduce hallucination with prompts, such as adding “Answer as truthfully as possible, and if you’re unsure of the answer, say, ‘Sorry, I don’t know.’” Asking models for concise responses also seems to help with hallucinations—the fewer tokens a model has to generate, the less chance it has to make things up. Prompting and context construction techniques in Chapters 5 and 6 can also help mitigate hallucinations.
  - If we can’t stop hallucinations altogether, can we at least detect when a model hallucinates so that we won’t serve those hallucinated responses to users? Well, detecting hallucinations isn’t that straightforward either—think about how hard it is for us to detect when another human is lying or making things up. But people have tried. We discuss how to detect and measure hallucinations in Chapter 4.

### Summary

- A crucial factor affecting a model’s performance is its training data.
- After sourcing the data, model development can begin. While model training often dominates the headlines, an important step prior to that is architecting the model.
- The scale of a model can be measured by three key numbers: the number of parameters, the number of training tokens, and the number of FLOPs needed for training. Two aspects that influence the amount of compute needed to train a model are the model size and the data size.
- Due to the low quality of training data and self-supervision during pre-training, the resulting model might produce outputs that don’t align with what users want. This is addressed by post-training, which consists of two steps: supervised finetuning and preference finetuning.
- Sampling makes AI models probabilistic. This probabilistic nature is what makes models like ChatGPT and Gemini great for creative tasks and fun to talk to. However, this probabilistic nature also causes inconsistency and hallucinations.
- Working with AI models requires building your workflows around their probabilistic nature. The rest of this book will explore how to make AI engineering, if not deterministic, at least systematic. The first step toward systematic AI engineering is to establish a solid evaluation pipeline to help detect failures and unexpected changes. Evaluation for foundation models is so crucial that I dedicated two chapters to it, starting with the next chapter.