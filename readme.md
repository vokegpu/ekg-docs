# EKG DOCS

## Preface

EKG docs contains tutorial about the EKG standard way to creating GUIs and the fundamentals of EKG model-architecture.

## Programming GUIs with EKG

* What is EKG?
* Setup
* Hello-World
* EKG-standard

## EKG Technical Details

Complete details about importants EKG technologies, this section includes: the root problem of legacy EKG, memory-handling model architecture, safety-proofs, package pattern, descriptors-model, standard format for input-binding(s) and more.

### The Memory-Handling Model

EKG is a C++ secure library, these two topics details and give the necessary proofs for this affirmation.

**note: the memory-handling model article is not complete yet (as EKG code is being implemented, this article will be rewrited to sync with the EKG origin)**

* [The problem of legacy EKG memory-handling model](./model/the-problem.md)
* [Memory-handling model](./model/architecture-model.md)

### Descriptors, Performance and Patterns

* [Packages pattern model](./model/packages-pattern-model.md)
* [UI descriptor-based model](./model/ui-descriptor-based-model.md)
* [Value low-frequency-based model](./model/value-low-frequency-model.md)
* [CPU-side and GPU-side performance-model](./model/performance-model.md)
* [Input-binding tag-style](./model/input-binding-tag-style.md)

This topic is important for contributing to EKG, so, should be caeful read before sending ~useless~ pull requests. As everything here on Vokegpu, take a read on [Vokegpu standard](https://github.com/vokegpu/standard).
