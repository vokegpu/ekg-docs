# EKG DOCS

## Preface

EKG docs contains tutorial about the EKG standard way to creating GUIs and the fundamentals of EKG model.

EKG is a significant project of my life, I wasted lot of time coding EKG, but it is what I love to do, graphics, GUIs, and GPU-accelerated softwares.

Thank you Astah 🐈‍⬛,  
Thank you God.

## Programming GUIs with EKG

* What is EKG?
* Setup
* Hello-World
* EKG-standard

## EKG Technical Details

Complete details about importants EKG technologies, this section includes: the root problem of legacy EKG, memory-handling model architecture, safety-proofs, package pattern, descriptors, standard format for input-binding(s) and more.

### The Memory-Handling Model

EKG is a C++ secure library, these two topics details and give the necessary proofs for this affirmation.

**note: the memory-handling model article is not complete yet (as EKG code is being implemented, this article will be rewrited to sync with the EKG origin)**

* [Design Decisions](./model/design.md)
* [Memory-Model](./model/architecture.md)

Soon a paper should be done in `./publish/` with objective-proofs.

### Architecture, Rendering and Formatting

* [Architecture](./model/architecture.md)
* [Rendering](./model/rendering.md)
* [Formatting](./model/formatting.md)

This topic is important for contributing to EKG, so, should be caeful read before sending ~useless~ pull requests. As everything here on Vokegpu, take a read on [Vokegpu standard](https://github.com/vokegpu/standard).
