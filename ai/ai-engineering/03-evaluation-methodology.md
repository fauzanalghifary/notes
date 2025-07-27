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

-