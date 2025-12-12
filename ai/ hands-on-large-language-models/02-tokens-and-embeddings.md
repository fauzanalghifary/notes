# Chapter 2. Tokens and Embeddings

## How Tokenizers Prepare the Inputs to the Language Model

- This is how the tokenizer broke down our input prompt. Notice the following:
  - The first token is ID 1 (<s>), a special token indicating the beginning of the text. 
  - Some tokens are complete words (e.g., Write, an, email).
  - Some tokens are parts of words (e.g., apolog, izing, trag, ic).
  - Punctuation characters are their own token.
- Just like on the input side, we need the tokenizer on the output side to translate the token ID into the actual text. We do that using the tokenizer’s decode method

## How Does the Tokenizer Break Down Text?

- There are three major factors that dictate how a tokenizer breaks down an input prompt.
  - First, at model design time, the creator of the model chooses a tokenization method. Popular methods include byte pair encoding (BPE) (widely used by GPT models) and WordPiece (used by BERT). These methods are similar in that they aim to optimize an efficient set of tokens to represent a text dataset, but they arrive at it in different ways.
  - Second, after choosing the method, we need to make a number of tokenizer design choices like vocabulary size and what special tokens to use.
  - Third, the tokenizer needs to be trained on a specific dataset to establish the best vocabulary it can use to represent that dataset. Even if we set the same methods and parameters, a tokenizer trained on an English text dataset will be different from another trained on a code dataset or a multilingual text dataset.
- In addition to being used to process the input text into a language model, tokenizers are used on the output of the language model to turn the resulting token ID into the output word or token associated with it

## Word Versus Subword Versus Character Versus Byte Tokens

- The tokenization scheme we just discussed is called subword tokenization. It’s the most commonly used tokenization scheme but not the only one. 
- The four notable ways to tokenize:
  - Word tokens
    - used less and less in NLP
    - may be unable to deal with new words that enter the dataset after the tokenizer was trained. 
    - This also results in a vocabulary that has a lot of tokens with minimal differences between them (e.g., apology, apologize, apologetic, apologist)
  - Subword tokens
    - contains full and partial words
    - ability to represent new words by breaking down the new token into smaller characters, which tend to be a part of the vocabulary.
  - Character tokens
    - Where a model with subword tokenization can represent “play” as one token, a model using character-level tokens needs to model the information to spell out “p-l-a-y” in addition to modeling the rest of the sequence.
    - Subword tokens present an advantage over character tokens in the ability to fit more text within the limited context length of a Transformer model.
  - Byte tokens
    - breaks down tokens into the individual bytes that are used to represent unicode characters. 
    - some subword tokenizers also include bytes as tokens in their vocabulary as the final building block to fall back to when they encounter characters they can’t otherwise represent.

## Comparing Trained LLM Tokenizers

- three major factors that dictate the tokens that appear within a tokenizer: 
  - the tokenization method, 
    - Each of these methods outlines an algorithm for how to choose an appropriate set of tokens to represent a dataset. 
    - You can find a great overview of all these methods on the Hugging Face page that summarizes tokenizers.
  - the parameters and special tokens we use to initialize the tokenizer, and
    - Vocabulary size
    - Special tokens
    - Capitalization
  - the dataset the tokenizer is trained on
    - Even if we select the same method and parameters, tokenizer behavior will be different based on the dataset it was trained on (before we even start model training)

## Token Embeddings

- The next piece of the puzzle is finding the best numerical representation for these tokens that the model can use to calculate and properly model the patterns in the text.
- what embeddings are. They are the numeric representation space utilized to capture the meanings and patterns in language.

## A Language Model Holds Embeddings for the Vocabulary of Its Tokenizer

- After a tokenizer is initialized and trained, it is then used in the training process of its associated language model. This is why a pretrained language model is linked with its tokenizer and can’t use a different tokenizer without training.
- The language model holds an embedding vector for each token in the tokenizer’s vocabulary, as we can see in Figure 2-7. When we download a pretrained language model, a portion of the model is this embeddings matrix holding all of these vectors.

## Creating Contextualized Word Embeddings with Language Models

- Instead of representing each token or word with a static vector, language models create contextualized word embeddings (shown in Figure2-8) that represent a word with a different token based on its context
- We recap the input tokenization and resulting outputs of a language model in Figure 2-9. Technically, the switch from token IDs into raw embeddings is the first step that occurs inside a language model.

## Text Embeddings (for Sentences and Whole Documents)

- While token embeddings are key to how LLMs operate, a number of LLM applications require operating on entire sentences, paragraphs, or even text documents. This has led to special language models that produce text embeddings—a single vector that represents a piece of text longer than just one token.
- There are multiple ways of producing a text embedding vector. One of the most common ways is to average the values of all the token embeddings produced by the model. Yet high-quality text embedding models tend to be trained specifically for text embedding tasks.

## Word Embeddings Beyond LLMs

- Embeddings are useful even outside of text and language generation. Embeddings, or assigning meaningful vector representations to objects, turns out to be useful in many domains, including recommender engines and robotics
- This idea of a model that takes two vectors and predicts if they have a certain relation is one of the most powerful ideas in machine learning, and time after time has proven to work very well with language models. This is why we’re dedicating Chapter10 to this concept and how it optimizes language models for specific tasks (like sentence embeddings and retrieval).

## Summary

- We explored how tokenizers are the first step in processing input to an LLM, transforming raw textual input into token IDs. Common tokenization schemes include breaking text down into words, subword tokens, characters, or bytes, depending on the specific requirements of a given application.
- Three of the major tokenizer design decisions are the tokenizer algorithm (e.g., BPE, WordPiece, SentencePiece), tokenization parameters (including vocabulary size, special tokens, capitalization, treatment of capitalization and different languages), and the dataset the tokenizer is trained on.
- Language models are also creators of high-quality contextualized token embeddings that improve on raw static embeddings
- In addition to producing token embeddings, language models can produce text embeddings that cover entire sentences or even documents
- Before LLMs, word embedding methods like word2vec, GloVe, and fastText were popular. In language processing, this has largely been replaced with contextualized word embeddings produced by language models.
- In the next chapter, we will take a deep dive into the process after tokenization: how does an LLM process these tokens and generate text? We will look at some of the main intuitions of how LLMs that use the Transformer architecture work.