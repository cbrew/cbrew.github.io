---
layout: post
title: "Problem-First Research: A Practitioner's Method for NLP"
date: 2026-04-03
categories: science nlp compling methodology
---

## The method nobody writes down

Since around 1994 I have been advising graduate students on how to
tackle NLP research problems. Over the years I developed an implicit
method --- a sequence of moves that I found myself recommending
again and again, across wildly different projects. I never wrote it
down as a recipe. It lived in whiteboard conversations, in the moment
when a student came in stuck and left with a direction. Dennis Mehay,
one of my former PhD students, could probably reconstruct some of the
weirder things I said to him during those conversations. But a method
that only exists in oral tradition is a method that might get lost, so
here is my attempt to make it explicit.

If you read one thing to prepare for this kind of work, read George
Polya's [*How to Solve
It*](https://en.wikipedia.org/wiki/How_to_Solve_It). The book is
apparently about mathematics, but it is really about how to get an
actionable grasp on whatever problem is in front of you. Polya's key
insight is that the hard part is not execution --- it is formulating the
problem clearly enough that execution becomes possible.

## The seven moves

### 1. What kind of problem do we have?

Classification? Sequence modeling? Text generation? Dialogue?
Retrieval? Something else entirely? I don't always know the full
taxonomy, and the taxonomy itself keeps evolving. I tend to look at
places where someone has already done the work of organizing the space
--- [Hugging Face](https://huggingface.co/tasks),
[scikit-learn](https://scikit-learn.org/stable/tutorial/machine_learning_map/index.html),
or textbook chapters that lay out families of tasks. The goal is not to
commit to a formulation prematurely, but to know what formulations are
available.

### 2. Where is the data?

What does it look like? How much is there? What are its biases and
gaps? Is there a standard split? Has anyone released annotations? The
answers to these questions constrain everything downstream.

### 3. Supervised or unsupervised?

This is a coarser version of Move 1, but it deserves its own step
because the answer reshapes the entire research plan. Sometimes the
answer is "supervised, but we don't have labels yet," which is its own
interesting problem.

### 4. What is the dumbest possible baseline?

Majority class? Bag of words? Longest common substring? A random
baseline? The dumbest baseline you can build is the one that tells you
whether you even have a real problem. If a trivial approach gets 95%
accuracy, either the task is easy or the evaluation is broken. Either
way, you need to know before you invest effort.

### 5. Find a similar problem

No matter how strange the analogy. Can we borrow that approach? What
would have to change? Why won't it work directly? This is the most
creative step, and the one where reading widely pays the biggest
dividends. The analogy does not have to come from NLP. It does not even
have to come from computer science.

### 6. If an obstacle blocks Move 5, cheat

If you can't directly apply the borrowed approach, find a shortcut.
Simplify the problem. Use a proxy task. Generate synthetic data. Remove
a constraint and see what happens. The obstacle is usually informative
--- it tells you what is actually hard about your problem.

### 7. Loop back to Move 5 with a better analogy

The first analogy is rarely the best one. Each failed or partial
attempt teaches you something about the structure of the problem,
which lets you search for a better match.

## The non-negotiable

The one thing I do not do is move forward without an analysis of the
problem. Throughout all seven moves, I am doing exploratory reading,
talking to colleagues, and trying to make connections to things I
already know or half-know. The method is not a pipeline --- it is a
set of orienteering moves that keep you from wandering into a
solution before you understand the terrain.

## How this played out: spectral clustering for German verbs

My favorite example of this method in action produced one of my more
unexpected papers: ["Spectral Clustering for German
Verbs"](https://aclanthology.org/W02-1016/) (Brew and Schulte im
Walde, 2002).

The starting point was Sabine Schulte im Walde's work on inducing
semantic verb classes for German from subcategorization frames ---
a classification problem (**Move 1**) with rich syntactic distributional
data (**Move 2**) in an unsupervised setting (**Move 3**). Standard
k-means clustering was our first baseline (**Move 4**), and it worked,
but not as well as we wanted. The high-dimensional space of
subcategorization frame distributions was causing trouble: noisy
features, curse of dimensionality, the usual problems.

We had a natural analogy (**Move 5**): other people had clustered
distributional data before, using methods from information retrieval
and computational psycholinguistics. But our initial clustering
approach was hitting a wall --- the direct application of standard
methods to our high-dimensional feature space was not giving us clean
enough clusters.

Here is where the method earned its keep. Rather than pushing harder
on the same approach, I went looking for a better analogy (**Move 7**).
Through exploratory reading I found Andrew Ng's work on spectral
clustering, which had been developed in the context of computer vision
problems like image segmentation. The paper was clear, the math was
elegant, and the core idea --- use the eigenvectors of a similarity
matrix to project data into a lower-dimensional space before
clustering --- was a natural fit for our problem, even though it came
from a completely different field.

But you only find that kind of cross-disciplinary connection if you
have done the prior work of understanding your problem abstractly
enough to recognize it when it appears in a different guise. We knew
we needed dimensionality reduction on a similarity structure. That
abstract framing is what let us see that a technique from computer
vision was exactly what we needed for computational linguistics.

The resulting paper applied spectral clustering to the verb
classification task, showed that the eigendecomposition of the
similarity matrix gave us a better low-dimensional representation
than direct clustering, and produced cleaner semantic verb classes.
It appeared at EMNLP 2002 alongside our companion paper ["Inducing
German Semantic Verb Classes from Purely Syntactic Subcategorisation
Information"](https://aclanthology.org/P02-1029/) (Schulte im Walde
and Brew, 2002) at ACL.

## The method across projects

This same pattern of problem analysis, analogy-seeking, and
cross-disciplinary borrowing shows up across many of the projects I
have supervised or collaborated on:

- **Verb class disambiguation** (Lapata and Brew, 2004; Li and Brew,
  2007, 2008, 2010): Starting from the question "can we disambiguate
  Levin verb classes automatically?", we explored informative priors,
  distributional features, and subcategorization data --- each iteration
  sharpening the problem formulation.

- **CCG-based semantic role labeling** (Boxwell, Mehay, and Brew,
  2009, 2010, 2011): Dennis Mehay and Stephen Boxwell brought CCG
  parsing to bear on semantic role labeling, a move that required
  seeing SRL as a structured prediction problem where syntactic
  representations from a different parsing tradition could provide
  better features.

- **Resource-light morphological annotation** (Feldman, Hana, and
  Brew, 2004, 2005, 2006): The problem was "how do you build NLP tools
  for a language when you have almost no annotated data?" The cheat
  (**Move 6**) was to borrow resources from a related language ---
  tagging Russian using Czech resources, transferring morphological
  annotation cross-linguistically.

- **Clinical NLP** (Raghavan, Fosler-Lussier, Brew, and Lai, 2012):
  Medical event coreference resolution required recognizing that
  clinical narratives have their own structure, and that tools from
  the UMLS Metathesaurus could serve as a bridge between NLP methods
  and medical knowledge.

- **Abusive language detection** (Narang and Brew, 2020): Using
  syntactic dependency graphs for abuse detection was another instance
  of asking "what representation makes this problem tractable?" and
  finding the answer in an unexpected place.

## Why Polya matters

Polya's great contribution was to observe that problem-solving is
itself a skill, not a talent, and that it can be taught by making
heuristic strategies explicit. His four phases --- understand the
problem, devise a plan, carry out the plan, look back --- map
naturally onto the seven moves above. Moves 1--3 are understanding the
problem. Moves 4--7 are devising and revising the plan. The
non-negotiable --- never move forward without analysis --- is Polya's
insistence that understanding must precede action.

What I would add to Polya, from three decades of doing this in NLP, is
that **the most productive analogies often come from outside your
field**. The spectral clustering example came from computer vision. The
cross-lingual annotation transfer came from thinking about cognate
relationships. If you understand your problem abstractly enough, you
can recognize solutions when they appear in unfamiliar clothing. That
recognition is the core skill, and it only develops through broad
reading, broad conversation, and a willingness to follow your
curiosity into territory that looks, at first, like a detour.
