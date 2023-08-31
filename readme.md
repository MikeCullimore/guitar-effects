# Guitar effects

How close can we get our guitar tone to a given recording?

* Possible to avoid having to model specific guitars and amps? (Might not know what was used anyway; prohibitively expensive and time-consuming.)
* NeuralPi example below works in real-time but that requires _a lot_ of work!
* Start with offline, any language and hardware.
* Use dynamic time warping to align guitar input to target recording?
    * First do coarse alignment: which section of recording corresponds to input?
* Use source separation to isolate the guitar track in the recording.
* There will be limits imposed by the guitar and amp e.g. no amount of audio processing could make an SSS strat sound like a guitar with humbuckers.
* Port to Rust?! (NeuralPi requires SIMD: only experimental support in Rust.)

## Useful links

* [RTNeural](https://github.com/jatinchowdhury18/RTNeural): A lightweight neural network inferencing engine written in C++. This library was designed with the intention of being used in real-time systems, specifically real-time audio processing.

### NeuralPi

NeuralPi is a guitar pedal using neural networks to emulate real amps and pedals on a Raspberry Pi 4.

* [GitHub repo](https://github.com/GuitarML/NeuralPi)
* [Blog post](https://towardsdatascience.com/neural-networks-for-real-time-audio-raspberry-pi-guitar-pedal-bded4b6b7f31)
* [YouTube demo](https://www.youtube.com/watch?v=_3zFD6h6Wrc)

### PedalNet

* [Original repo](https://github.com/teddykoker/pedalnet)
* [GuitarML fork](https://github.com/GuitarML/PedalNetRT)

## todo

* Choose example song and isolate guitar part.
    * Record yourself trying to play that same part (short subsection).
* Read links above and see what can be adapted:
    * How do they define loss function?
    * What are their inputs and outputs?
* Gather more links, summarise here.
* Diagrams
    * Data preparation
    * Loss function
    * Model inference
