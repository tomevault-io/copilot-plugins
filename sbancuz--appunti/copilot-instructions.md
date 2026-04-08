## appunti

> The is no unique solution but there are some approaches to build agents of increasing complexity. You could try to encode your knowledge, but that could be either too complex or you could be just not skilled enough in the topic your training your agent.

The is no unique solution but there are some approaches to build agents of increasing complexity. You could try to encode your knowledge, but that could be either too complex or you could be just not skilled enough in the topic your training your agent.  

![[Agent coplexity.png]]
### Simple reflex agent

This is the simplest kind of agent. These agents select actions on the basis of the current percept, ignoring the rest of the percept history. This kind of agent will work only if the correct decision can be made on the basis of just the current percept -- that is, only if the environment is fully observable. Even a little bit of unobservability can cause serious trouble.
```pseudo
   \begin{algorithm}
    \begin{algorithmic}
      \PROCEDURE{Simple-reflex-agent}{$percept$} 
	    \State $\textbf{persistent}:$ rules, a set of condition-action rules
		\STATE $state \gets$  \CALL{Interpret-input}{$percept$}
        \STATE $rule \gets$  \CALL{Rule-match}{$state$}
        \STATE $state \gets rule.ACTION$
	    \return{action}
      \ENDPROCEDURE
      \end{algorithmic}
    \end{algorithm}
```
>[!example]
>The first Roomba was a simple reflex agent that detected whenever it needed to change direction if there was an obstacle in the way. 

Simple reflex agents can easily be caught in infinite loops unless the agent can **randomize** its actions. 
### Model-based reflex agents

The most effective way to handle partial observability is for the agent to keep track of the part of the world it can't see now, that is, the agent should maintain some sort of **internal state** that depends on the percept history. 

Updating this internal state information as time goes by requires two kinds of knowledge:
- Some information on how the world changes over time called the **transition model** of the world
- Some information about how the state of the world is reflected in the agent percepts. This is called the **sensor model**
```pseudo
   \begin{algorithm}
    \begin{algorithmic}
      \PROCEDURE{Model-based-reflex-agent}{$percept$} 
	    \State $\textbf{persistent}:$ \\
			transition-model, a description transitions between states and action\\ 
			sensor-model, how the environment is perceived by the actor\\ 
			rules, a set of condition-action rules\\
			action, the most recent action\\
		\STATE $state \gets$  \CALL{Update-state}{$state, percept, action, transition-model, sensor-model$}
        \STATE $rule \gets$  \CALL{Rule-match}{$state, rules$}
        \STATE $state \gets rule.ACTION$
	    \return{action}
      \ENDPROCEDURE
      \end{algorithmic}
    \end{algorithm}
```
 The next Roomba can create a model of the environment.
### Goal-based agents

Knowing something about the current state of the environment is not always enough to decide what to do. The correct decision depends on its **goal** that has to describe what situations are desirable. This agent also has to use [[Search problem]] to plan it's actions while considering their consequence upon the environment. 

This could be achieved with the [[Minimax algorithm]] 

>[!example]-
 ![[Boids]]
### Utility-based agents

Goals alone are not enough to generate high-quality behavior in most environments. In this type of agent goal are still present, but they only provide a binary distinction of good and bad states. A more general performance measure should allow a comparison of different world states according to the **utility**.

An agent's **utility function** is an internalization of the performance measure. Provided that the internal utility function and the external performance measure are in agreement, an agent chooses actions to maximize its utility will be rational according to the external performance measure.
### Learning agents

This is the preferred method of building AI, and proposes their construction by simply teaching machines how to do something. Other than the ease of creation, the learning agent has another advantage: it allow the agent to operate in initially unknown environments.

Learning allows the agent to operate in initially unknown environments and to become more competent than its initial knowledge alone might allow.

![[learning agents.png]]

When an agent is a computer we call this [[Machine learning]].

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sbancuz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sbancuz)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
