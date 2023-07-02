---
layout: post
title:  "Managing (dialogues with) unreliable things"
date: 2023-07-02
categories: ChatGPT chatbots
---
# Response to a question from Andras Kornai

I said something elliptical and ill-advised about the "leakiness" of LLMs.


Andras said:
> Please explain this further. What leakage is there in LLMs? 
> LLMs are essentially captured by the 150 or so lines of code Andrei Karpathy used to explain them. "Grammars leak" for sure. But LLMs have a leak-proof definition.

I agree that LLMs and their behavior admit of crisp, clean definitions. I am not denying that. The leakage I am talking about is the breakdown of abstractions that are
typically found in traditional information systems. They come under some degree of stress when one tries to move from a one-shot task-satisfaction scenario to a richer conversational interaction.
The fundamental problem is that the system's users are erratic and unpredictable. Conversational systems have always had a hard time with this. It's hard to make a person seem enough like a software
component. My point is that when we are trying to do this, and we want to mirror ChatGPT's relative naturalness, by using an LLM, you introduce another erratic component into the system.

Here's why I say that LLMs are erratic. Autoregressive text transformers are stateless stimulus-response engines that map from a text input to a text output. In that sense they are all equivalent to large look-up tables. The engine behind ChatGPT is no exception.
But, feeding a lightly-curated version of the conversational history to such an engine, and returning a lightly-curated version of the output as next contribution made by the simulated agent.  This works really well for the
particular use-case of a semi-informative chat engine. Extra training and prompting can even make systems of this type seem helpful, cooperative, responsive to interlocutor goals, and all that. They do much of what a designed dialogue system does, 
but unreliably. You can ask them to do what you want, and they often will, but sometimes won't. And when they don't, you will be none the wiser.

Systems like Staffan Larsson's Issue-Based Dialogue Managers have a lot of useful ideas for how to mamage a task-oriented 
conversation about an an information system made up of components that have sensible and readily enumerable failure modes. Sadly, viewed as software components, LLMs are almost as erratic and unpredictable as people. How do you give 
reliable conversational feedback when the "subsystems" on which you rely might not even try to do what they are supposed to. The good news is that common-sense methods of dealing with unreliable people apply. The bad news is that this 
is what automation was supposed to get us away from.
