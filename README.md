# EKG DOCS

"Money can not buy knowledge, as can not buy love, but wasted knowledge is not wasted love on-someone",

Thank you Astah 🐈‍⬛,  
Thank you God.

## Preface

EKG docs contains tutorials about the EKG standard way to creating GUIs and the fundamentals of EKG model.

## Programming GUIs with EKG

* What is EKG?
* Setup
* Hello-World
* EKG-standard

## EKG Technical Details

Complete details about importants EKG technologies, this section includes: the root problem of legacy EKG, memory-handling model architecture, safety-proofs, package pattern, descriptors, standard format for input-binding(s) and more.

### Memory-Safe Model

EKG is a C++ secure library, these two topics details and give the necessary proofs for this affirmation.

* [Design](./model/design.md)
* [Memory-Model](./model/memory.md)

**note: the memory-handling model article is not complete yet (as EKG code is being implemented, this article will be rewrited to sync with the EKG origin)**

Soon a paper should be done in `./publish/` with objective-proofs.

### Architecture, Rendering and Formatting

* [Architecture](./model/architecture.md)
* [Rendering](./model/rendering.md)
* [Formatting](./model/formatting.md)

These topics are important for contributing to EKG, so, should be caeful read before sending ~useless~ pull requests. As everything here on Vokegpu, take a read on [Vokegpu standard](https://github.com/vokegpu/standard).
