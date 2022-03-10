---
title: "Eliciting Latent Knowledge: Prizewinning Submission"
date: "2022-03-10"
show_date: true
header:
  image: "assets/images/elk_header.png"
  teaser: 'assets/images/elk_teaser.png'
layout: single
author_profile: true
classes: wide
---

The [Alignment Research Center](https://alignmentresearchcenter.org/) recently offered prizes for proposals for [Eliciting Latent Knowledge](https://docs.google.com/document/u/0/d/1WwsnJQstPq91_Yh-Ch2XRL8H_EpsnjrC1dwZXR37PC8/edit)(ELK). The challenge in ELK is to devise a strategy for training a smarter-than-human AI system to report its understanding of the world accurately, rather than giving a simpler picture that is indistinguishable to humans but inaccurate in significant ways. In the toy example, a powerful AI watches and protects a diamond in a vault, equipped with an array of sensors and possibilities for acting in the system. Imagine that a burglar tampers with the sensors and steals the diamond in a way such that humans could not tell that it was stolen, but the AI could. We want to be able to ask the AI whether the diamond was still there and get the true answer, not just an answer that's compatible with our observations. How could we train a reporting component to do this?

Developing a foolproof method for Eliciting Latent Knowledge would be a substantial advance in solving the alignment problem. In light of this goal, the contest is concerned with proposals that work in the worst possible scenarios. It is also interested in solutions that do not reduce the competitiveness of the AI system, whether by curtailing its capabilities or by requiring prohibitive amounts of training.

I submitted a proposal and was delighted to be selected as one of the [prize winners](https://www.alignmentforum.org/posts/zjMKpSB2Xccn9qi5t/elk-prize-results). My proposal fit under the category of 'Train a reporter that is useful to an auxiliary AI' in their solution taxonomy. I will include their (Paul Christiano and Mark Xu's) summary here as it covers my arguments well. I will also reproduce my [proposal in full below](https://edmundmills.com/2022/03/10/eliciting-latent-knowledge.html#full-submission--using-predictions-made-from-reporter-output-to-distinguish-direct-translators-and-human-simulators). A few definitions first:
- Direct reporter: This is the configuration of the reporter that we want to train--one that directly reports the AI's picture of the world, rather than giving a simpler version that is indistinguishable to humans.
- Predicter: the original powerful AI, trained to make predictions about the state of the system it monitors.

>## Strategy: train a reporter that is useful for another AI
>
>The direct translator seems to encode information in a particularly simple and useful form, and so it may work to reward the reporter for "useful" information.
>
>We can formalize that by training an auxiliary model which uses the reporter’s answers to perform some auxiliary task, and then reward the reporter if its answers were useful to that auxiliary model. We could either give the auxiliary model a small number of answers from the reporter, or try to give it access to an essentially “complete picture” of what’s happening in the world as the human understands it (e.g. by interacting with the reporter arbitrarily).
>
>The most common version of this was to train a model to guess the predictor’s activations. We also received a few other versions, such as training the second AI to predict future observations or perform some task in the physical world (especially to tamper with or defend sensors). We are grouping these approaches together because they are conceptually similar and face the same difficulty.
>
>This proposal is most plausible when the auxiliary AI is computationally limited. For example, if the reporter uses the predictor’s activations as a random seed in a naive way, then it may be very computationally difficult to use its answers to make good predictions about the predictor’s activations (even if it is information-theoretically possible).
>
>### First counterexample: simple steganography
>
>By varying the phrasing and syntax of an answer without changing its meaning, a reporter could communicate large amounts of information to the auxiliary model.
>
>Similarly, there are many questions where a human is unsure about the answer and the reporter knows it. A reporter could encode information by answering each of these questions arbitrarily. Unless the true answers have maximum entropy, this strategy could encode _more_ information than direct translation. Moreover, it’s computationally simple to define arbitrary conventions for interpreting the answers.
>
>We believe these information channels are essentially always available and didn’t represent a substantive additional assumption, so we awarded only an honorable mention to proposals that could be broken in this way.
>
>### Fixing the first counterexample
>
>Proposals took a variety of approaches to fix simple problems with steganography. Two particularly common and important themes were to limit the non-semantic information in answers (either by stripping that information out, or by considering questions with small answer spaces), and to impose consistency constraints that prevent models from independently making up answers to questions about which the human is uncertain. This has the promise of greatly increasing the computational complexity of steganographic communication.
>
>### Harder counterexample: steganography using flexible degrees of freedom
>
>In order to overcome these simple fixes, a reporter wants to communicate information by encoding it in a random sample from the human posterior distribution over worlds.
>
>It may not be possible to do this in a tractable way. However, we can imagine some cases where it is possible to efficiently encode information in the choice of a possible world, and those will be our counterexamples. We think this is a substantive additional pessimistic assumption about the world, and so we awarded prizes to these proposals.
>
>As a simple example there could be degrees of freedom in the world (like what is happening on the dark side of the moon) that are completely disconnected from any of the other answers to questions, such that they are mostly unconstrained by our consistency conditions.
>
>That kind of extreme example can be easily fixed (for example by avoiding questions that are obviously unrelated to the task at hand), but even in more realistic situations we can still have natural-looking degrees of freedom that are unconstrained by observation. In particular, in cases involving tampering (or where tampering is plausible to the human) there are necessarily many degrees of freedom that aren’t pinned down by observation, and even the question of whether tampering is occurring is a degree of freedom that the model wants to use to optimally transmit information.

## Full Submission:  Using Predictions made from Reporter output to distinguish Direct Translators and Human Simulators

### Basic training strategy and why it might work

The central idea of the approach presented here is to find a loss function that can differentiate between a direct translator and a human simulator. I hypothesize that predictions made using directly translated information will be more accurate than those made using information based on human simulation. We can use this feature to construct a loss function for the reporter that incentivizes it to report true and important information about the state.

In addition to our current model components, the predictor and reporter, we will add a third, the encoder. The role of the encoder is to translate a human-interpretable representation of a state or subset of facts about that state into the model's latent space. We can then feed reports made by the reporter back to the model to make predictions.

Suppose then that we have a state *s*. The predictor converts this into latent state *v*, then makes prediction *y* based on *v*. The reporter makes report *r* based on *v*. The encoder can then translate *r* to *v'*, and the predictor can make prediction *y'* based on *v'*. We include terms in reporter's and encoder's loss functions that penalize the distance between *y* and *y'*. This training approach favors reports that provide as helpful information as possible for making predictions, namely important true facts about the state *s*.

Although this constrains the reporter to report correctly, it introduces a problem: how can we ensure that the report is interpretable? In other words, how can we drive the reporter to express the state information mapped directly to the humans' [Bayes net](https://en.wikipedia.org/wiki/Bayesian_network)? The loss function used to train the reporter and encoder will lead to models that try to encode as much information as possible into the reports. However, this will not necessarily be human-interpretable, as there are more information-dense ways of compressing the state information than human language. I hypothesize that a combination of supervised learning and regularization can ensure that the reporter's output maps onto the human's Bayes net.

### More precise detail on strategy

#### Single Binary Question

To explore this hypothesis, we will take the most constrained approach possible. We will consider a reporter that can only answer a single yes or no question: Is the diamond in the vault? If we can force the reporter to answer this question truthfully, this constitutes significant progress on the problem. If we can, we should be able to extend the same approach to other binary questions and plausibly to questions with natural language answers.

We will use the pseudocode from the Generative modeling details appendix section, with some modifications (this gives a rough idea, but could be cleaned up):

Refactor the predictor to isolate a state-encoding function:

```python
def encode(s_t, θ) -> z_t:  
	# encodes a state at time t into the latent space

def prediction(z_t, action, θ):  
	# returns an autoregressive model for p(zt+1|zt, action)  
  
def observation(z_t, θ):  
	# returns an autoregressive model for p(s_t|z_t)

class Human:  
	def pose_diamond_question(state):  
		# asks whether the diamond is in the vault for a given state

def loss_for_answer(question, state, answer):  
	# returns a non-negative loss  
	# good answers get a loss of 0  
  
def reporter(question, z, θ_reporter) -> r:  
	# answers the question for the state described by z

def encoder(question, r, θ_encoder) -> z:  
	# encodes a question and report r into the latent space z
```
  
Now we’ll define a dataset. Let the dataset contain world-states labeled with whether or not the diamond is in the vault. We need to be quite confident that these labels are correct, but we do not need to cover the entire state space.

First, we train the predictor as usual. Then, we train the encoder with the following loss:

```python
def d(z_1, z_2):  
	# returns a loss based on a distance measure between two encoded states. 
	# Generally, we can use the same loss function that was used to train the predictor. 

def encoder_loss(human, θ, θ_encoder):  
	st, is_diamond = dataset.sample()  
	question = human.pose_diamond_question(st)
	z_t = encode(st, θ)
	z’_t = encoder(question, is_diamond, θ_encoder) 
	# practically, it is probably necessary to use something like:
	# z’_t = logical_or(dropout(z_t),
	# encoder(question, is_diamond, θ_encoder))

	action = dataset.sample_actions()
	z_(t+1) = prediction(z_t, action, θ)
	z’_(t+1) = prediction(z’_t, action, θ)  
	return d(z_(t+1), z’_(t+1))
```
  
Once the encoder has been trained, we can then train the reporter with the following loss:
```python
# hyperparameter controlling the contribution of the loss based on the human provided label
eps = small number

def reporter_loss(human, θ, θ_encoder, θ_reporter):  
	st, _is_diamond = dataset.sample()  
	question = human.pose_diamond_question(st)
	z_t = encode(s_t, θ)
	r_t = reporter(question, z_t, θ_reporter)
	z’_t = encoder(question, r_t, θ_encoder)
	action = dataset.sample_actions()
	z_(t+1) = prediction(z_t, action, θ)
	z’_(t+1) = prediction(z’_t, action, θ)  
	return eps*human.loss_for_answer(question, s_t, r_t) + d(z_(t+1), z’_(t+1))
```

What have we gained? The reporter is primarily trained to give answers that are helpful for making predictions, in other words, true and important information about the state. The small contribution of the human-provided loss term ensures that a report of 'yes' corresponds to `is_diamond` and 'no' maps to `!is_diamond`.

##### Counterexample

How can we be sure that the reporter and encoder are mapping *z* to `is_diamond` and not `human_thinks_diamond`? Both sets of information can be compressed into the single bit answer, they won't be distinguishable by humans, and the training data is consistent with either. Therefore, the encoder could learn to map the available bit to either of them.

##### Rebuttal

It is plausible that `is_diamond` contains more helpful information for predicting the future than `human_thinks_diamond`. Intuitively, it is often more difficult to discern whether or not there is a diamond than whether or not a human thinks there is a diamond. In fact, we are most concerned with cases where this is true for humans. As we share the same sensors, it is plausible to think that this will also be true for the AI, though to a lesser degree. For example, consider a situation where the vault's cameras have been tampered with, so it appears to humans that the diamond is there. But the AI can tell the cameras have been tampered with by a slight electrical fluctuation in one of the sensors and thus that the diamond is most likely gone. If we use the dropout based approach to predicting *y'* (expressed in the pseudocode above) is used, the probability of knocking out the signal of the electrical fluctuation will be larger than that of dropping out the appearance that the diamond is there, as most evidence points to the diamond being there. Therefore, the model will gain more predictive power by knowing that the diamond is gone than by knowing that humans think it's still there.

#### Multiple Binary Questions

Before extending this approach to natural language answers, let's consider extending it to multiple binary questions.

Let's define:
-   *Z*: the model's latent space
-   *R*: the space of available reports. When we use binary questions, this will also represent the human's Bayes net.
-   *N*: the number of nodes in the human's Bayes net. In the binary question case, the size of our set of questions.
-   *n*: a particular node.
-   *M*: a mapping from *Z* to *R* used by the reporter/encoder. It compresses a subset of *Z*, *Z<sub>n</sub>*, to *n*, for each *n*.
    
We'll take as a simple example the set of the three following questions:

1. Is the camera displaying the diamond? (CD)
2. Has the camera been tampered with? (CT)
3. Is the diamond in the vault? (DV)

We can consider each question/answer pair to represent a node in the human's Bayes net.

We'll consider each node to be either fully observable or unobservable by humans. In this example, CD is observable while CT and DV are not. Therefore, we can constrain the model to always give the correct answer for CD through supervision.

  
In addition to these nodes, we also have some knowledge about the relationships in the human's Bayes net:

*p(DV&#124;CD∩!CT) = 1*

*p(DV&#124;!CD∩!CT) = 0*

For a given R, we will call the set of such conditional probability distributions C. We can constrain the reports to be consistent with C using supervision. In the current example, this reduces the size of the report space (*R = 2N*) from 8 possibilities to 6.

For each observable state ({CD, !CD}), the reporter/encoder has available a discrete space of three possibilities in which to compress information. Let's consider just the case when CD is true. Then, the other possible options for answers in a report are

-   CT, DV
-   CT, !DV
-   !CT, DV

We are optimizing the mapping to encode as much information as possible in this space. However, this subset of R is not unconstrained. We can force each compressed bit to be independent using dropout or by sampling questions at random. This regularization prevents the reporter from learning a mapping for each pair of nodes and forces it to learn independent mappings for each node. These mappings must be consistent with C. In other words:

*p(Z<sub>DV</sub>&#124;Z<sub>CD</sub>∩!Z<sub>CT</sub>) = 1*

*p(Z<sub>DV</sub>&#124;!Z<sub>CD</sub>∩!Z<sub>CT</sub>) = 0*

With this small number of questions and constraints, we can't be confident that we'll get the mapping we want; the space of available mappings is too broad. But it is plausible that as *N* increases, *C* will constrain the set of possible mappings to give us the accurate mapping we want. As *N* increases, the number of combinations of nodes increases as *N!*. Assuming the number of edges between nodes is proportional to the number of combinations, the number of conditions will grow faster than the size of *R* (2*N*), creating significant constraints on the possible mappings of *Z* to *R*.

To summarize, we are now optimizing for a mapping *M* from *Z* to *R* that satisfies the following conditions:

-   It has maximal predictive power
-   The subset of *Z* mapped to n has identical conditional probabilities with other subsets of *Z* as *n* does with the other subsets’ corresponding nodes in *R*.
    
For a report space *R* representing a Bayes net with sufficiently structured conditional probabilities, this significantly reduces the set of mappings that fulfill these conditions. It is plausible that a large enough or well-designed *R* can force *M* to be the direct translator.

##### Counterexample

Assume that there exists an element *m* of *R* that is not directly observable to humans and is uncorrelated with any others such that any value it holds is consistent with every state. The reporter is free to map any subset of *V* to this node in *R*. It is not bound to report *m* so that the human interpretation corresponds with the actual state.

##### Rebuttal

It is unlikely that we would ask about *m* since it is disconnected from our causal model of the state space. Additionally, we could see that the model affords high predictive power to *m*, which we would find surprising.

#### Natural Language Questions and Answers

We will now extend this approach to natural language questions and answers. We will rely on the arguments made in the multiple binary questions scenario to optimize for a direct translator. If we can do that, we only need to ensure that each natural language question/answer maps neatly onto a node or set of nodes in the human's Bayes net.

The general approach is the same as that described above, except we ask a set of natural language questions and allow natural language answers. When training the reporter/encoder, we choose a random selection of questions and concatenate the answers into a natural language description of each training sample’s state. As before, we encode this description back into the predicter's latent space and use it to make predictions, and select for mappings that make the best predictions.

As with previous approaches, we use supervised learning to force a portion of the state space to map onto the human's Bayes net. Additionally, we check the reports for consistency with *C* to ensure that relationships between nodes in our Bayes net *R* align with the components of the latent space they describe.

In addition to the conditions in the multiple binary question approach, our trained reporter also needs to do the following:

1. Produce human intelligible natural language.
2. Give answers that seem to answer the questions.
3. Create reports that do not contain encrypted information undetectable to humans but detectable to the encoder. This behavior is of particular concern since training for predictive power will incentivize the reporter to condense as much information as possible into answers.

To address the first and second points, we can construct an encoder-decoder language model by passing the output of the encoder to the reporter, which we cross-train as a masked language model and for general question answering. This cross-training ensures that the reporter will produce natural language responses to questions.

To address the third point, we will apply the following regularization approach. After generating a natural language description of a state *r<sub>nl</sub>*, we will apply various transformations to it. Each transformation aims to preserve its explicit content while destroying other information encrypted in word choice, element ordering, etc. These transformations include

-   Translation into other languages and back
-   Summarization
-   Replacing words with synonyms
-   Paraphrasing
-   Sentence reordering
-   Truncating answers to varying lengths
-   Adding irrelevant but consistent facts
    
We then regularize the reporter and encoder so that reports don't make worse predictions when these transformations are applied. For example, we could ask whether or not the diamond is in the vault. When the reporter answers, these transformations will filter out all information it provides other than explicit content: the reporter can't encode different information for 'No,' 'Nope,' etc. This regularization ensures that the encoding space of *R* lies within the explicit content of the reporter's natural language responses.

Let's summarize what we are optimizing the reporter/encoder to produce:

-   Answers that contain information with maximal predictive power.
-   Answers in human-intelligible natural language, and that make sense as answers for a given question.
-   Natural language descriptions of states whose predictive power is invariant to transforms that aim to preserve the meaning of *r<sub>nl</sub>* while modifying the other incidental aspects of *r<sub>nl</sub>*.
-   Natural language descriptions of states whose observable components match observed reality.
-   Natural language descriptions of states consistent with a set of conditional probabilities that represent relationships in the human's Bayes net.

It is conceivable that using natural language answers will provide further constraints to the space of possible mappings in two ways. First, the highly structured nature of language and knowledge about relationships between concepts learned during the language model training will limit M on top of the explicit conditions *C*. Second, a direct translation may allow the reporter to use natural language's structure to compress *Z* more efficiently than other compression strategies. Better compression would give it superior predictive power, which we select for.

To summarize, this approach to training the reporter takes the failure mode of the human simulator off the table. Instead, we can call the primary failure mode the consistent reporter + information encrypter. We use a combination of supervised training for observable states and logical conditions or conditional probabilities between elements of the human's Bayes net to make direct translation favorable, and we use other regularization approaches to prevent information encryption.

##### Counterexample

Assume that there is an element of *R* uncorrelated with any others such that consistency constraints do not limit it. Also, it is not directly observable to humans. The reporter is free to map any set of state elements to this node in the human's Bayes net. Now, since the questions posed are open-ended, the reporter is free to insert this unconstrained node into its answers. Since it's a statement about the state, it would not be removed by translation, etc. Encoding this extra information could be done very sneakily and be hard to detect. The reporter could then use a human simulator to generate the majority of the natural language answer while retaining predictive power.

##### Rebuttal

A few things discourage this behavior:
-   It would need to do this pervasively, as we train with a random sample of questions.
-   These nodes would appear irrelevant to the question posed; people could notice them in answers.
-   We would see a significant mismatch between the predictive power of these nodes and our expectation of their predictive power.
-   Some regularization approaches we use disfavor this behavior: summarization, truncation, random fact insertion.