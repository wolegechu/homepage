+++
date = "2018-02-21T00:38:00"
draft = false
tags = ["Today I Learned", "Deep Learning", "star"]
title = "From Deep Learning of Disentangled Representations to High-level Cognition"
math = true
outurl = ""
summary = """
A Note from Yoshua Bengio's Talk.
"""

[header]
image = ""
caption = ""

+++
{{% toc %}}

## Still Far from Human-level AI
- Industrial successes mostly based on supervised learning
- Learning superficial clues, not generalizing well enough outside of training contexts, easy to fool trained networks:
    - Current models cheat by picking on surface regularities

### Paper: Measuring the tendency of CNNs to Learn Surface Statistical Regularities
- **Hypothesis**: *Deep CNNs have a tendency to learn superficial statistical regularities in the dataset rather than high level abstract concepts.*
- From the perspective of learning high level abstractions, Fourier image statistics can be *superficial* regularities, not changing object category.
- Different Fourier filters, same high level abstractions (objects) but different surface statistical regularities (Fourier image statistics).
- **Experiments**: Train on one training set and evaluate the test sets.
    - A generalization gap: max difference in test accuracies
    - Large generalization gap: CNN exploits too much of low level regularities, as opposed to learning the abstract high level concepts.

'''
Intuition: we have a mental model that captures the explanatory factors of our world to some extent. It's not perfect. And we can generalize to new configurations of the existing factor. When in new situation, involve concepts that we already know, thaen just combine  in very new ways.
'''

## Learning  <<How the world ticks>>
- So long as our machine learning models << cheat >> by relying only on superficial statistical regularities, they remain vulnerable to out-of-distribution examples
- Humans generalize better than other animals thanks to a more accurate internal model of the **underlying causal relationships**
- To predict future situations (e.g., the effect of planned actions) far from anything seen before while involving known concepts, an essential component of reasoning, intelligence and science

### Paper: Learning Multiple Levels of Abstraction
- The big payoff of deep learning is to allow learning higher levels of abstraction
- High-level abstractions **disentangle the factors of variation**, which allows much easier generalization and transfer
- with Data -> Information -> Knowledge -> Wisdom
    - Organizational Maturity gain
    - Abstraction Level gain

#### Invariance and Disentangling
The notion of disentangling is related but different from the notion of invariance, as being a very important notion in computer visions, speech recognition and so on, where we'd like to build detectors and features that are invariant to the things we don't care about, but sensitive to the things we do care about.

- Invariant features
- Alternative: Learning to disentangle factors
- Good disentangling -> avoid the curse of dimensionality

#### Latent Variables and abstract Representations
![](/img/post/QQ20180219-231015.png)

- Encoder/decoder view: maps between low & high-levels
- Encoder doses inference: interpret the data at the abstract level
- Decoder can generate new configurations
- Encoder flattens and disentangles the data manifold
- Marginal independence in h-space (Can assemble from each of the factors if dimension is independently)

### Paper: Manifold Flattening
- Deeper representations -> abstractions -> disentangling
- Manifolds are expanded and flattened

## What's Missing and What's Needed with Deep Learning? - Deep Understanding and Abstract Representations

### Current ML theory Beyond i.i.d. Data
- Real-life applications often require generalizations in regimes not seen during training
- Humans can project themselves in situations they have never been (e.g. imagine being on another planet, or going through exceptional events like in many movies)
- **Key: understanging explanatory/causal factors and mechanisms**

### How to Discover Good Disentangled Representations
- How to discover abstractions?
- What is a good representation? *(Bengio et al 2013)*
- Need clues (= priors) to help *disentangle* the underlying factors, such as
    - Spatial & temporal scales
    - Marginal independence
    - Simple dependencies between factors
        - Consciousness prior
    - Causal / mechanism independence
        - Controllable factors

### Acting to Guide Representation Learning & Disentangling
- **Some factors (e.g. objects) correspond to 'independently controllable' aspects of the world**
- *Can only be discovered by acting in the world*
    - *Control linked to notion of objects & agents*
    - *Causal but agent-specific & subjective*

(Acting to acquire information)

### Paper: Independently Controllable Factors
- Jointly train for each aspect (factor)
    - A policy $\pi_k$ (which tries to selectively change just that factor)
        - Kth policy is going to control the Kth factor
    - A representation (which maps state to value of factor) $f_k$
    - Discrete case, $\phi \in {1,..,N}$, define *selectivity$:
        - $$\sum_{k=1}^N{\mathbb{E}_{(s_t, a_t, s_{t+1})} [\pi_k{a_t|s_t} \cfrac{f_k(s_{t+1}) - f_k(s_t)}{\sum_{k'}{|f_{k'}(s_{t+1})f_{k'}(s_t)|}}]}$$
- Optimize both policy $\pi_k$ and representation $f_k$ to minimize
    - $$\mathbb{E}_s[\frac{1}{2} \lVert s - g(f(s)) \rVert^2_2] - \lambda \sum_k{\mathbb{E}_s[ \sum_a{\pi_k(a|s)\log{sel(s,a,k)}}]}$$
    - Namely, (reconstruction error) - $\lambda$ (disentanglement objective)
- Example 1.Predict the effect of actions in attribute space
- Example 2.Given two states, recover the casual actions leading from one to the other
Given initial state and set of actions, predict new attribute values and the corresponding reconstructed images.

![](/img/post/QQ20180220-175520.png)

- Multi-step policies $\epsilon$ Embedding Space for Naming Factors and Policies
    ![](/img/post/QQ20180220-180229.png)

#### Abstraction Challenge for Unsupervised Learning
- Why is modeling P(acoustics) so much worse than modeling P(acoustics|phonemes) P(phonemes)?
- Wrong level of abstraction?
    - many more entropy bits in acoustic details then linguistic content -> predict the future in abstract space instead

### Paper: The Consciousness Prior (inspired by cognitive psychology)
- Conscious thoughts are very low-dimensional objects compared to the full state of the (unconscious) brain
- Yet they have unexpected predictive value of usefulness -> strong constraint or prior on the underlying representation
- Other
    - **Thought*: composition of few selected factors / concepts (key/value) at the highest level of abstraction of our brain
    - Richer than but closely associated with short verbal expression such as a **sentence** or phrase, a **rule** or **fact** (link to classical symbolic AI & knowledge representation)
- **How to select a few relevant abstract concepts making a thought?**
    - Content-based  Attention

#### On the relation between Abstraction and Attention
- Attention allows to focus on a few elements out of a large set
- Soft-attention allows this process to be trainable with gradient-based optimization and backprop
- Attention focuses on a few appropriate abstract or concret elements of mental representation
- Access consciousness is one aspect of consciousness, what is accessed and comes to mind (Dehaene's C1 or global availability), See *Dehaene et al, Science, 2017.*
    - ![](/img/post/QQ20180221-044638.png)
- 2 levels of representation
    - High-dimensional abstract representation space (all known concepts and factors) *h*
    - Low-dimensional conscious thought *c*, extracted from *h*
        - ![](/img/post/QQ20180221-045704.png)
    - *c* includes names (keys) and values of factors
- Conscious prediction over attended variables A (soft-attention)
    - $$V = - \sum_A{w_a \log{P(h_{t,A} = a | c_{t-1})}}$$
        - $w_A$: Attention weights
        - $h_{t,A}$: Factor **name**
        - $a$: Predicted **value**
        - $c_{t-1}$: Earlier conscious state
    
#### What Training Objective?
- How to train the attention mechanism which selects which variables to predict?
    - Representation learning without reconstruction:
        - Maximize entropy of code (preserve a lot of information about the data)
        - Maximize mutual information between past and future representations (see also *Becker & Hinton 1992*)
    - *Objective function completely in abstract space, higher-level parameters model dependencies in abstract space*
    - **Usefulness of thoughts: as conditioning information for action, i.e., a particular form of planning for RL, i.e., the estimated gradient of rewards could also be used to drive learning of abstract representations**

#### Consiciousness Prior and Classical AI
- *Conscious thought is closedly related to a linguistic utterance*
- (conditioning variables -> predicted variables) = a **rule**
- Conscious thoughts & short-term memory: good level of representation for memorization and memories
- Need to refer to variables in conscious states facilitates recursive compositional computation
- Consciousness makes it easier to associate preception, action and **natural language** and use *natural language data to guide the creation of high-level abstraction expressed linguistically*

## Ongoing Research

### Deep Unsupervised RL for AI neural nets -> cognition
- Learn more abstract representations which capture causality
- Disentangle controllable factors
- Naturally gives rise to the notion of **objects, attributes, agents & goals**
- Natural language & consciousness prior: other clue about abstract representations (see also *Andreas, Klein & Levine 2017*)
- Unsupervised RL research, performed in gradually more challenging sumulated environments

### AI for Good: take action!
- Beyond developing the next gadget
- Actionalbe items:
    - Favor ML applications which help the poorest countries, may help with fighting climate change, etc.
    - A role for the Partnership on AI: fund an organization which will coordinate, prioritize and channel funding for such applications, as well as facilitate internships for students from poor conuntries in top ML labs
    - Accept such interns even if they are below our bar, they will return home with knowledge of the state-of-the-art in ML