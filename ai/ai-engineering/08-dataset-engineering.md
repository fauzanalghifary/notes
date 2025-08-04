# Chapter 8 - Dataset Engineering

- The quality of a model depends on the quality of its training data. The best ML team in the world with infinite compute can’t help you finetune a good model if you don’t have data
- The goal of dataset engineering is to create a dataset that allows you to train the best model, ideally within your allocated budget.
- As fewer companies can afford to develop models from scratch, more are turning to data to differentiate their AI performance. As models demand more data, data handling becomes more challenging and demands more investments in talent and infrastructure.
- Many AI companies now employ data labelers, dataset creators, and data quality engineers, either integrated into or working alongside their core engineering teams.
- If the model landscape is confusing enough with numerous offerings, the data landscape is even more complex, with an ever-growing array of datasets and techniques being introduced. This chapter gives you an overview of the data landscape and considerations to take into account when building your own dataset.
- It begins with data curation, addressing questions like What data do you need? How much? What does it mean for data to be of high quality? It then discusses techniques for data synthesis and processing. Data curation, generation, and processing don’t follow a linear path. You’ll likely have to go back and forth between different steps.
- For the same model, different training phases aim to teach the model different capabilities, and, therefore, require datasets with different attributes
- This chapter focuses on post-training data because that’s more relevant to application developers. However, I’ll also include lessons from pre-training data when these lessons are insightful for post-training.
- The increasing focus on data during AI development has given rise to data-centric AI, as opposed to model-centric AI:
  - Model-centric AI tries to improve AI performance by enhancing the models themselves. This involves designing new architectures, increasing the sizes of the models, or developing new training techniques. 
  - Data-centric AI tries to improve AI performance by enhancing the data. This involves developing new data processing techniques and creating high-quality datasets that allow better models to be trained with fewer resources.
  - In the early days of deep learning, many AI benchmarks were model-centric. Given a dataset like ImageNet, people try to train the best possible model using the same dataset. In recent years, more benchmarks have become data-centric. Given the same model, people try to develop a dataset that gives this model the best performance.
  - The model-centric and data-centric division helps guide research. In reality, however, meaningful technological progress often requires investment in both model and data improvements.


### Data Curation


### Data Augmentation and Synthesis


### Data Processing


### Summary