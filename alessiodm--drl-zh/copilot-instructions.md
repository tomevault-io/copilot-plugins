## drl-zh

> The “Deep Reinforcement Learning - Zero to Hero” project is an educational resource about deep

# Deep Reinforcement Learning: Zero to Hero! (drl-zh)

The “Deep Reinforcement Learning - Zero to Hero” project is an educational resource about deep
reinforcement learning that guides the user from the foundation to advanced topics with a very
hands-on approach. In particular, the “solution” folder contains all the notebooks with all the code
that executes and trains the various algorithms correctly. The style of the notebooks is very
educational but at the same time produces code that works: there are paragraphs explaining the
theory, when simplifications are necessary they are highlighted, but the entirety of the code
represents how the RL algorithms work with lots of fidelity. An example is the RLHF notebook, where
we take the shortcut of not training a reward model but using a “reward function heuristic” in its
place. That helps also to connect with the GRPO implementation, making the notebook building on
itself and teaching the core concepts of various RLHF techniques to the user.

The "root" folder contains the exact same notebooks, but with lines of code emptied out and/or
stripped out with corresponding “# TODO: …” comments for the user to write the code themselves and
learn along the way. The TODO comments are generally complete and helpful, and if the user is stuck
they can always get back and check the "solution" folder. As some more details about the "root"
notebooks with TODOs, you'll see that variable names and skeleton of the algorithm are left there,
with more commentary around the code and the respective TODO. For example a typical section looks
like:

```
# <comment about what the section does>
# TODO: <what to do in the next line to replace the None>. Hint: <hints>.
my_variable = None
```

Other examples are for-loops where they may be there but just have a `pass` under them (if simple),
or the values with `None` like above. We want this so that the empty notebook really guides the user
along.

Summarizing: the notebooks are educational, contain enough theory for the savvy user, have clean and
simple code focused on learning, and setup the most minimal environment to learn the various aspects
and the RL technique in question (for each notebook).

---
> Source: [alessiodm/drl-zh](https://github.com/alessiodm/drl-zh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
