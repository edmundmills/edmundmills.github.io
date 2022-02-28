---
title: "2021 MineRL BASALT Competition (2nd Place)"
date: "2022-02-05"
show_date: true
header:
  image: "assets/images/2021-minerl-basalt.png"
  teaser: 'assets/images/minerl-basalt-preview.png'
layout: single
author_profile: true
classes: wide
---
In the fall of 2021, I participated in the [2021 MineRL BASALT competition](https://www.aicrowd.com/challenges/neurips-2021-minerl-basalt-competition) ([Submission Code](https://github.com/edmundmills/basalt-competition)). This competition, organized by the [Center for Human-Compatible AI (CHAI)](https://humancompatible.ai/), was intended to promote research into AI systems that can learn from human feedback. 

When I started the competition, I had only a few months of experience with Machine Learning and practically no experience with Reinforcement Learning or Imitation Learning. But I thought it would be an excellent way to learn, and indeed it was. It encouraged me to read many papers, implement various algorithms, and stretch far past what I thought I'd be able to do. In the end, my team took second place in the competition for our approach.

This competition allowed me to learn and practice many parts of research engineering, including:
- Implementing online imitation learning algorithms, where the agent learns from both demonstrations and from interacting with the environment
- Implementing training of recurrent neural networks
- Working with the competition dataset: exploring, preprocessing, and augmenting the demonstrations
- Working with GPUs, profiling and optimizing our training algorithm for efficient use of our computational resources
- Hyperparameter search 
- Experimenting with a variety of algorithms and approaches

## Competition Motivation and Overview
As we develop more powerful AI systems, we want to train them to have goals that are difficult to specify precisely. A simple example of this kind of task is creating a beautiful flower arrangement. Our idea of what defines beautiful is too nuanced to translate directly into a defined numeric reward system. Demonstrations and human feedback provide a potential way to communicate these types of rich concepts, from which an AI agent could learn what it is we'd like it to do. On a larger scale, for example, in city planning, the hard to describe nuances of complex goals could significantly influence human well-being.

CHAI organized The MineRL Benchmark for Agents that Solve Almost-Lifelike Tasks (BASALT) competition to promote this research. The tasks were defined by short natural language descriptions and were set in MineCraft, a simulated open world, chosen for its wide range of possible actions and world-states. They were:
- Find a cave.
- Make a waterfall in a beautiful location, return to the bottom and take a picture.
- Create a pen enclosing two animals.
- Build a house matching the style of the local village, and give a tour of it.

The organizers provided a dataset of human demonstrations of the tasks we could use to teach our agents about them. Additionally, we were allowed to incorporate human feedback into our training strategy. Rather than evaluating submissions by a numeric metric, contractors rated them subjectively. 

Here's a cherry-picked video of our agent on the make waterfall task:

<iframe width="560" height="315" src="https://www.youtube.com/embed/gPY5_Ai9pYk" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
<br>

## Approach
During my literature review in the early stages of the competition, I found a paper that seemed quite promising for the competition, [IQ-Learn: Inverse Soft-Q Learning for Imitation](https://arxiv.org/abs/2106.12142). I reached out to the primary author, [Divyansh Garg](https://divyanshgarg.com/), with some questions about it, and we ended up working together on the competition. It was a fruitful collaboration: I did most of the engineering and implementation while he provided ideas and guidance. Our approach was centered around using the IQ-Learn algorithm to train an agent for each competition task.

### IQ-Learn
One commonly used approach to imitation learning is behavioral cloning, which learns a policy that tries to match an expert's actions for any given state. Policies learned from behavioral cloning run into problems when the agent encounters a state the expert hadn't, leading to out-of-distribution errors. The agent hasn't built an implicit model of what it is trying to accomplish or a sense of the values of different states, so it cannot generalize to these new states. The canonical example of this is a car going off course: the expert demonstration does not provide information about what to do in these states, so the agent won't know what to do and can get farther and farther off course.

 IQ-Learn learns a Q-function using human demonstrations, which implicitly represents both a reward function and a policy (for guiding the agent's behavior). Q-learning is an approach to reinforcement learning in which an algorithm optimizes a Q-function. The Q-function represents the expected value of each action an agent could take from a particular state (the Q-value). It does this by considering the immediate reward of taking that action, along with the Q-values of the actions it can take from the expected next state that it ends up in (see [Bellman Equation](https://en.wikipedia.org/wiki/Bellman_equation)).

IQ-Learn allows for direct optimization of a Q-function from expert demonstrations. Methods that learn Q-function-based policies can mitigate the out-of-distribution problems that behavioral cloning faces. They can model the value of particular states and assign a higher value to those that expert visits than those they don't. They can then learn to return to states the expert visits, course correcting and returning to states where it has learned how to behave. Policies learned this way can also learn about environmental dynamics by learning an implicit model of the action-mediated transitions between states.

Since IQ-Learn learns a Q-function by optimizing over a single training loop, it is almost as simple to implement as behavioral cloning while implicitly modeling environmental dynamics. In contrast, many other approaches to imitation learning (like Inverse Reinforcement Learning and adversarial methods) that have these properties involve optimizing over nested training loops and are significantly more challenging to implement.

### Recurrent Neural Networks
Another core component of our approach was using Recurrent Neural Networks (RNNS). In the reinforcement learning context, recurrent neural nets allow an agent to carry information along as it interacts with its environment, rather than learning to act based on its immediate observations alone. Here's a schematic showing the general idea:

![Unrolled Recurrent Neural Network](/assets/images/Computational-graph-of-a-recurrent-neural-network-with-one-hidden-layer-29.png)
<figcaption align = "center">(M. Zakeri-Nasrabadi, Saeed Parsa, 2021)</figcaption>

When we included an RNN as part of the agent's policy, it learned more complex behaviors than similarly trained non-recurrent policies. These behaviors included swimming, which requires performing the same action repeatedly over many timesteps. Additionally, in the MakeWaterfall task, the agent learned to stay nearby and end the episode after placing the waterfall rather than moving away and continuing to act. This behavior was a significant differentiator of success and failure on the task.

## Conclusion
I highly recommend participating in this sort of competition as a way to learn. The BASALT competition is aimed at a fascinating area of research and was well designed and run by the organizers. I appreciate the competition organizers for all their work, my teammate Div for his support, and the other competitors for the sense of community.

To conclude, here are a few more videos of our agents:

<iframe width="560" height="315" src="https://www.youtube.com/embed/EMVb5LDQDaA" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
<br>

<iframe width="560" height="315" src="https://www.youtube.com/embed/WaxwEwn1TmQ" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
<br>