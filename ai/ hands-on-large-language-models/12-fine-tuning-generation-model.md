# Chapter 12. Fine-Tuning Generation Models

- In this chapter, we will take a pretrained text generation model and go over the process of fine-tuning it. This fine-tuning step is key in producing high-quality models and an important tool in our toolbox to adapt a model to a specific desired behavior. Fine-tuning allows us to adapt a model to a specific dataset or domain.
- Throughout this chapter, we will guide you among the two most common methods for fine-tuning text generation models, supervised fine-tuning and preference tuning.

## The Three LLM Training Steps: Pretraining, Supervised Fine-Tuning, and Preference Tuning

- There are three common steps that lead to creating a high-quality LLM:
  - Language modeling
    - The first step in creating a high-quality LLM is to pretrain it on one or more massive text datasets (Figure12-1). During training, it attempts to predict the next token to accurately learn linguistic and semantic representations found in the text
    - This produces a base model, also commonly referred to as a pretrained or foundation model. Base models are a key artifact of the training process but are harder for the end user to deal with. This is why the next step is important.
  - Fine-tuning 1 (supervised fine-tuning)
    - With supervised fine-tuning (SFT), we can adapt the base model to follow instructions. During this fine-tuning process, the parameters of the base model are updated to be more in line with our target task, like following instructions. Like a pretrained model, it is trained using next-token prediction but instead of only predicting the next token, it does so based on a user input
  - Fine-tuning 2 (preference tuning)
    - The final step further improves the quality of the model and makes it more aligned with the expected behavior of AI safety or human preferences. This is called preference tuning. Preference tuning is a form of fine-tuning and, as the name implies, aligns the output of the model to our preferences, which are defined by the data that we give it
- In this chapter, we use a base model that was already trained on massive datasets and explore how we can fine-tune it using both fine-tuning strategies

## Supervised Fine-Tuning (SFT)

## Full Fine-Tuning

- The most common fine-tuning process is full fine-tuning. Like pretraining an LLM, this process involves updating all parameters of a model to be in line with your target task. The main difference is that we now use a smaller but labeled dataset whereas the pretraining process was done on a large dataset without any labels
- During full fine-tuning, the model takes the input (instructions) and applies next-token prediction on the output (response). In turn, instead of generating new questions, it will follow instructions.

## Parameter-Efficient Fine-Tuning (PEFT)

- Updating all parameters of a model has a large potential of increasing its performance but comes with several disadvantages. It is costly to train, has slow training times, and requires significant storage. To resolve these issues, attention has been given to parameter-efficient fine-tuning (PEFT) alternatives that focus on fine-tuning pretrained models at higher computational efficiency.
- To have the LLM follow instructions, we will need to prepare instruction data that follows a chat template

## Evaluating Generative Models

- Evaluating generative models poses a significant challenge. Generative models are used across many diverse use cases, making it a challenge to rely on a singular metric for judgment. Unlike more specialized models, a generative model’s ability to solve mathematical questions does not guarantee success in solving coding questions.
- Given their probabilistic nature, generative models do not necessarily generate consistent outputs; there is a need for robust evaluation.
  - Word-Level Metrics
    - One common metrics category for comparing generative models is word-level evaluation. These classic techniques compare a reference dataset with the generated tokens on a token(set) level. Common word-level metrics include perplexity,8 ROUGE,9 BLEU,10 and BERTScore.11
    - Although perplexity, and other word-level metrics, are useful metrics to understand the confidence of the model, they are not a perfect measure. They do not account for consistency, fluency, creativity, or even correctness of the generated text.
  - Benchmarks
    - A common method for evaluating generative models on language generation and understanding tasks is on well-known and public benchmarks, such as MMLU,12 GLUE,13 TruthfulQA,14 GSM8k,15 and HellaSwag.16 These benchmarks give us information about basic language understanding but also complex analytical answering, like math problems.
    - Benchmarks are a great way to get a basic understanding on how well a model performs on a wide variety of tasks. A downside to public benchmarks is that models can be overfitted to these benchmarks to generate the best responses. Moreover, these are still broad benchmarks and might not cover very specific use cases. Lastly, another downside is that some benchmarks require strong GPUs with a long running time (over hours) to compute, which makes iteration difficult.
  - Leaderboards
    - As such, leaderboards were developed containing multiple benchmarks. A common leaderboard is the Open LLM Leaderboard, which, at the time of writing, includes six benchmarks, including HellaSwag, MMLU, TruthfulQA, and GSM8k. Models that top the leaderboard, assuming they were not overfitted on the data, are generally regarded as the “best” model. However, since these leaderboards often contain publicly available benchmarks, there is a risk of overfitting on the leaderboard
  - Automated Evaluation
  - Human Evaluation
    - Although benchmarks are important, the gold standard of evaluation is generally considered to be human evaluation. Even if an LLM scores well on broad benchmarks, it still might not score well on domain-specific tasks. Moreover, benchmarks do not fully capture human preference and all methods discussed before are merely proxies for that.
- As a result, there is no one perfect method of evaluating LLMs. All mentioned methodologies and benchmarks provide an important, although limited evaluation perspective. Our advice is to evaluate your LLM based on the intended use case. For coding, HumanEval would be more logical than GSM8k.