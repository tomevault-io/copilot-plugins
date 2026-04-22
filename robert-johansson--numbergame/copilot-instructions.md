## numbergame

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Run the Number Game demo
clj -M -m numbergame.core

# Run Gen.clj learning examples
clj -M -m ex01-distributions        # Probability distributions
clj -M -m ex02-generative-functions # Generative functions and traces
clj -M -m ex03-choicemaps           # Choice maps and addressing
clj -M -m ex04-inference            # Importance sampling inference
clj -M -m ex05-trace-updates        # Trace updates and MH
clj -M -m ex06-advanced             # Advanced features (assess, untraced, etc.)

# Start REPL
clj

# Run tests (when added)
clj -X:test
```

## Project Structure

```
.
├── Gen.clj/              # Gen.clj probabilistic programming library (git submodule)
├── src/numbergame/       # Main source code
│   ├── hypotheses.clj    # Hypothesis space (rules + intervals)
│   ├── inference.clj     # Analytic Bayesian inference (size principle)
│   ├── generalization.clj# Generalization predictions
│   ├── model.clj         # Gen.clj generative model
│   └── core.clj          # Main entry point with demo
├── examples/             # Gen.clj learning examples (ex01-ex06)
├── docs/                 # Comprehensive Gen.clj documentation
└── deps.edn              # Clojure deps configuration
```

## Documentation

See `docs/` for detailed Gen.clj documentation:
- `gen-clj.md` - Overview and getting started
- `distributions.md` - Probability distributions
- `generative-functions.md` - The `gen` macro and composition
- `traces-and-choicemaps.md` - Working with traces
- `inference.md` - Inference algorithms
- `api-reference.md` - Quick API reference

## Project Overview

This is a Clojure project implementing Tenenbaum's "Number Game" (1999) - a Bayesian model of concept learning. It demonstrates how humans learn abstract concepts from positive examples only.

### The Number Game Task

- A hidden concept is a subset of integers 1-100
- Participants see positive examples (e.g., [16, 8, 2, 64])
- They judge probability that other numbers belong to the concept

### Key Findings the Model Captures

- **Rule-like behavior**: [16, 8, 2, 64] → strongly predicts "powers of 2"
- **Similarity-based behavior**: [16, 23, 19, 20] → smooth generalization in that range
- Both emerge from the **same** Bayesian machinery

## Architecture

The project has two complementary approaches:

### 1. Analytic Bayesian Layer

Pure Clojure functions for exact inference:

```clojure
(require '[numbergame.inference :as infer]
         '[numbergame.generalization :as gen])

;; Compute posterior over 5,078 hypotheses
(infer/posterior [16 8 2 64])

;; Get top hypotheses
(infer/top-hypotheses [16 8 2 64] 5)
;; => [[:powers-of-2 0.9945] ...]

;; Generalization predictions
(gen/p-in-concept [16 8 2 64] 32)  ;; => ~1.0
(gen/p-in-concept [16 8 2 64] 20)  ;; => ~0.006
```

### 2. Gen.clj Probabilistic Programming Layer

For simulation and potential extensions:

```clojure
(require '[numbergame.model :as model])

;; Simulate concept + examples
(model/simulate-with-hypothesis 4)
;; => {:hypothesis {:id :powers-of-2 :members #{...}}
;;     :examples [8 16 32 4]}

;; Access full trace
(let [tr (model/simulate 3)]
  (trace/get-choices tr)
  (trace/get-score tr))
```

## Key Implementation Details

### Hypothesis Space

- **Rule-based** (28 hypotheses): powers of 2, primes, multiples of k, odd/even, ends-in-d
- **Interval-based** (5,050 hypotheses): all [a, b] where 1 ≤ a ≤ b ≤ 100

### The Size Principle

Likelihood: `P(examples | h) = |h|^(-n)` if all examples in h, else 0

Smaller consistent hypotheses get exponentially more support as examples accumulate.

### Priors Available

```clojure
numbergame.hypotheses/uniform-prior      ; Equal weight to all
numbergame.hypotheses/rule-biased-prior  ; More weight to rules
numbergame.hypotheses/size-biased-prior  ; Prefer medium-sized concepts
```

## Gen.clj Usage Patterns

```clojure
(require '[gen.dynamic :as dynamic :refer [gen]]
         '[gen.distribution.commons-math :as dist]
         '[gen.generative-function :as gf]
         '[gen.trace :as trace]
         '[gen.choicemap :as choicemap])

;; Define generative function
(def my-model
  (gen [args]
    (let [x (dynamic/trace! :x dist/normal 0.0 1.0)]
      x)))

;; Simulate (unconstrained)
(gf/simulate my-model [args])

;; Generate with constraints
(gf/generate my-model [args] {:x 0.5})

;; Access trace data
(trace/get-choices tr)                    ; choice map
(trace/get-retval tr)                     ; return value
(trace/get-score tr)                      ; log probability
(choicemap/get-value choices :x)          ; unwrap Choice → raw value
```

## Design Principles

- State passed explicitly as arguments and returned as values
- Beliefs (posteriors) are vectors of probabilities over hypotheses
- Hypotheses are EDN maps: `{:id :keyword :members #{...}}`
- Log-space computation for numerical stability
- Analytic inference for speed, Gen.clj for extensibility

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/robert-johansson) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
