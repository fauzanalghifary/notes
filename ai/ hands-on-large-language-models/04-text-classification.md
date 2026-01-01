# Chapter 4. Text Classification

- A common task in natural language processing is classification. The goal of the task is to train a model to assign a label or class to some input text 

## Text Classification with Representation Models

- Classification with pretrained representation models generally comes in two flavors, either using:
  - a task-specific model, or
  - an embedding model
- A task-specific model is a representation model, such as BERT, trained for a specific task, like sentiment analysis
- an embedding model generates general-purpose embeddings that can be used for a variety of tasks not limited to classification, like semantic search
- We will leverage pretrained models that others have already fine-tuned for us and explore how they can be used to classify our selected movie reviews.

## Model Selection

- Moreover, it’s crucial to select a model that fits your use case and consider its language compatibility, the underlying architecture, size, and performance.
- Selecting the right model for the job can be a form of art in itself. Trying thousands of pretrained models that can be found on Hugging Face’s Hub is not feasible so we need to be efficient with the models that we choose.

## Using a Task-Specific Model

- As we load our model, we also load the tokenizer, which is responsible for converting input text into individual tokens
- These tokens are at the core of most language models, as explored in depth in Chapter2. A major benefit of these tokens is that they can be combined to generate representations even if they were not in the training data
- four such methods, namely precision, recall, accuracy, and the F1 score:
  - Precision measures how many of the items found are relevant, which indicates the accuracy of the relevant results.
  - Recall refers to how many relevant classes were found, which indicates its ability to find all relevant results.
  - Accuracy refers to how many correct predictions the model makes out of all predictions, which indicates the overall correctness of the model.
  - The F1 score balances both precision and recall to create a model’s overall performance.

## Classification Tasks That Leverage Embeddings

- There might be times when you want to fine-tune the model yourself if you have sufficient computing available (see Chapter11). 
  - However, not everyone has access to extensive computing. 
  - This is where general-purpose embedding models come in.
- Instead of directly using the representation model for classification, we will use an embedding model for generating features. 
  - Those features can then be fed into a classifier, thereby creating a two-step approach
    - Feature extractor
    - Classifier
- In the first step, we convert our textual input to embeddings using the embedding model
- The classifier is trainable and not limited to logistic regression and can take on any form as long as it performs classification.

## What If We Do Not Have Labeled Data?

- 