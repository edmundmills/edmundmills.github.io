---
title: "Training Transformers using Reinforcement Learning"
date: "2022-02-28"
show_date: true
header:
  image: "assets/images/essay_feedback_header.png"
  teaser: 'assets/images/essay_feedback_teaser.png'
layout: single
author_profile: true
classes: wide
---

The capabilities of Large Language Models have increased dramatically in the past few years, particularly with GPT-3, and they are frankly amazing. Watching GPT-3 compose original poems, complete analogies, and write coherent essays is fascinating. It's easy to imagine that these technologies will significantly change how we interact with computers and the web. These models also create an opportunity for applied alignment research as they are the largest we've trained so far. I recently participated in [Kaggle's Feedback Prize - Evaluating Student Writing](https://www.kaggle.com/c/feedback-prize-2021/) to learn skills relevant to this area of research--natural language processing, using transformers, and training transformers with reinforcement learning.

## Large Language Models and Alignment Research
The Large Language Models(LLMs), with [parameters now in the trillions](https://arxiv.org/pdf/2101.03961.pdf), are by far the largest models researchers and practitioners are creating. Their size allows them to have the complex behavior needed to understand language and makes their behavior hard to predict and understand. Since they are trained on large amounts of text scraped from the internet, they learn unsavory behavior: biases, racism, sexism, offensive language, etc. They also have issues with truthfulness since they are trained to imitate people, not answer questions correctly. They may answer a question incorrectly even when they 'know' the correct answer--that is, they'll give the right answer if you phrase the question differently.

By studying LLMs, we may develop insights and methodologies to understand and control their behavior that can scale to even larger and more complex systems. For example, [Redwood Research's current project](https://www.alignmentforum.org/posts/k7oxdbNaGATZbtEg3/redwood-research-s-current-project) is to develop a training methodology to ensure that a large language model does not exhibit a particular behavior, without otherwise degrading performance and remaining competitive in training time. The example they've chosen is an LLM trained to generate fanfic-like text, and they are trying to prevent it from creating text where any of the characters are harmed.

## Using Reinforcement Learning to Train Large Language Models
Research in this area often uses the technique of training language models with reinforcement learning, which allows them to optimize language models for purposes where a loss function can't be otherwise defined. For example, in [Learning to summarize from human feedback](https://arxiv.org/abs/2009.01325), the authors use human feedback on generated text summaries to improve summarization performance. A model learns to model subjective human feedback to evaluate summarization quality. They then use this as a reward function to train the summarizing language model with reinforcement learning. This approach led to significant improvements in summarization quality over language models trained only by example.

This approach seems broadly applicable in applied alignment research on LLMs. For example, one direction Redwood Research is exploring is training a model to understand when a character is harmed and then using this as a reward function to discourage a language model from generating such text.

## Essay Competition
To learn more about this approach, I worked on ([Code](https://github.com/edmundmills/writing-feedback)) a Kaggle competition for evaluating student writing, aimed at developing a teaching tool to provide real-time feedback on student essays. In this competition, the organizers provided student essays with annotations indicating which text sections correspond to various discourse elements: claims, evidence, positions, concluding statements, etc. The goal was to use these essays to develop a system to automate labeling essays this way.

Most teams in the competition appear to be using Named Entity Recognition (NER). In this approach, a model classifies each word as belonging to one of the types of discourse elements and whether or not it begins a new discourse element. Heuristics can then be used to select the spans and types of discourse elements based on the NER predictions for each word.

NER optimizes the model for different criteria than used to evaluate the submissions. The NER optimizes for token-wise identification of a class label using cross-entropy loss. In contrast, the competition considers F-score on predicting the presence of discourse elements, with some leeway for imperfect overlap. This gap creates an opportunity to improve performance. With this in mind, I used reinforcement learning to train a model directly on the competition's evaluation criteria.

### Reinforcement Learning for Essay Feedback
Through experimentation, I found that the reinforcement formulation that worked best for this problem had the following elements:
- The state-space was composed of:
	- Classification probabilities for segments of an essay found by splitting it at possible discourse element boundaries identified from the NER output and combining the NER classification probabilities for the words in that segment. [(1)](#Notes)
	- A one-hot vector indicating which segment the agent should predict for at the current time-step.
- The action-space was a single prediction for a segment, predicting its type and whether it begins a new discourse element.
- In an episode, the agent makes predictions sequentially for each segment then receives a reward for the F-score of the predictions for the essay.

The agent's model used attention layers to understand the relationships between the segments. It then gives an action based on the output for the channel indicated by the observation.

Behavioral Cloning (BC) was used to train the agent on the correct predictions. Then, the [stablebaselines3](https://stable-baselines3.readthedocs.io/en/master/) implementation of [PPO](https://openai.com/blog/openai-baselines-ppo/) was used to fine-tune the agent directly on the competition's metric. As in *Learning to summarize from human feedback*, the KL divergence between the policy and the initial BC policy was penalized to prevent catastrophic forgetting.

These three approaches gave the following approximate average F-scores across multiple training runs:
- NER using naive prediction: 0.42
- BC for classification of identified segments: 0.39
- RL fine-tuning: 0.51

 While these results are not competitive, they show that reinforcement learning led to significant improvements over a simple heuristic-based approach. There are a few promising ways to improve their performance:
 - Hyperparameter optimization on the NER task
 - Ensembling models to make predictions on the test set (these results are from cross-validation scores on k-fold validation sets)
 - Using more complicated heuristics for making predictions based on the NER probabilities
 
## Conclusion & Future Work
Rather than trying to refine and optimize further, I'm leaving this project here as I think that I learned what I set out to learn, and further work would yield diminishing returns. I'm glad to have learned:

- NLP basics--stemming, data cleaning and augmentation, vectorization, etc.
- Training transformer-based language models
- K-fold cross-validation
- Gradient accumulation
- Familiarity with Stable Baselines
- Using [sentence transformers](https://www.sbert.net/) for sentence embeddings

Possible future work could include using an auto-regressive approach for the agent. Another possibility is training an agent directly on the NER task using reinforcement learning, although this is significantly more computationally demanding.

### Notes
**[1]** I identified likely places that discourse elements would split from a combination of start token probability and changes in class probability, prioritizing recall. There is a tradeoff between expressiveness and task difficulty in choosing the threshold for identifying segments, as more segments lead to more possibilities for combining and classifying them incorrectly. This procedure generated about twice as many segments per essay as discourse elements with the chosen balance. If these segments were combined and classified correctly, the average score would be approximately .98. Averaging the 'continue' probabilities across the segments' tokens and the 'start' probability across the first three tokens of that segment generated a new set of features. I also included the segment length as a feature. I also tried using [sentence embeddings](https://arxiv.org/abs/1908.10084) of each segment as additional features, but this had almost no effect on performance.