# m17-cxx-demod Demodulator Design Notes
Paul Williamson KB5MU and Michelle Thompson W5NYV

The purpose of this document is to provide context for the study of the `m17-demod` application as implemented in C++ by Rob Riggs (mobilinkd).

It is based on study of the C++ source code in https://github.com/mobilinkd/m17-cxx-demod on the `master` branch as of commit `7e30f0601a33da423cb416fec5890c67af5b870a`, March 27, 2022, and informed by the documentation on https://m17-protocol-specification.readthedocs.io as of Build #16289613, March 7, 2022. I'll assume some level of familiarity with the M17 protocol.

## Overview

The m17-cxx-demod repository contains both a modulator application `m17-mod` and a demodulator application `m17-demod` for the M17 narrowband VHF/UHF mode under development by the M17 Project. This document focuses on the implementation of the demodulator application.

Strictly speaking, these programs don't actually do modulation or demodulation. Those functions are left to hardware components. Instead, m17-mod creates and m17-demod accepts a stream of samples of the frequency offset from channel center. That is, the samples from m17-mod are the input to an FM transmitter, and the samples going in to m17-demod are the discriminator output of an FM receiver. These are scalar values, not I/Q quadrature samples as typically found in an SDR-based system. They are sampled at 48,000 samples per second (10 samples per symbol), and to some extent this sample rate is baked into the code.

Likewise, the audio input to m17-mod and audio output from m17-demod in voice mode are sampled at a fixed rate ot 8000 samples per second, and each sample is a 16-bit signed integer. These values are also baked in.

The M17 protocol specification has a fixed modulation (4FSK) at a fixed symbol rate (4800 symbols per second) with a fixed frame size (40ms or 192 symbols). There are several frame formats. All of this is extensively baked in to the code as well.

The code does not actually deal with real time at all, so the rates and durations mentioned above are purely notional. m17-demod receives its input from stdin, from which it is processed through a queue, and generates its output directly to stdout, without regard for the passage of real time. This makes it ideal for offline processing of files of samples, which it does as fast as possible. It can also be used together with programs that do handle the realtime needs of the input and output. The README contains sample command lines to demonstrate this on a command line. Integrating this code into an actual realtime system is left as an exercise for the reader.

At the time of review, m17-demod implements only full-rate 3200bps voice streaming mode and bit error rate testing (BERT) mode. It includes a partial implementation of data-only mode, assuming AX.25 in unencapsulated ("raw") data frames, which it just dumps in a human-readable trace mode to stderr. Since m17-mod doesn't implement any data mode functions at all, it seems likely that m17-demod's data functions haven't seen a lot of testing.

The main program file for each application is found in the `apps` directory, but in common C++ style much of its functionality is implemented in external classes. Each of these is found in its own `.h` include file, found in the `include/m17cxx` directory. In addition, the program relies on several external libraries, including the codec2 library that implements the voice encoding and decoding.

Conceptually, the m17-demod program can be thought of as having these main components:

* A supervisory state machine that monitors the receiving process and arranges for received frames of various types to be processed correctly.

* A set of tracking functions that continually monitors signal presence ("DCD") and physical layer parameters such as frequency error and deviation.

* A stream of processing that works at the baseband sample level. This stream runs the tracking functions, and also handles searching for preambles and sync words, under the supervision of the state machine. Processing at this level culminates in a fancy modulation decoder that fills up a buffer with a frame's worth of decoded symbols.

* A stream of processing that works at the frame level, and is customized for each of the frame types.

* Output processing for received data. For BERT and data modes, the output goes to a text console (there is currently no provision for interfacing to an external data consumer, such as a network stack). For voice mode, the voice payload frames (two per M17 frame in the supported voice-only mode) are processed through the codec2 decoder (an external library function) and output to stdout as raw audio samples.

## Breakdown by Files (Classes)

### m17-demod.cpp
The main program for receive. Includes application-layer handlers for all the supported frame types, and the dispatcher that chooses which handler to call. Handles command-line arguments. Reads the baseband samples from stdin and sends them to the M17Demodulator object.
### m17-demod.h
Convenience include file, just includes several other include files.
### ax25_frame.h
AX.25 frame decoder and monitor output. No actual protocol.
### ClockRecovery.h
Processing at the sample level, tries to find the peak energy in the center of each symbol to recover the symbol clock.
### Convolution.h
Provides the convolve_bit() and update_memory() primitives that make up a convolutional encoder. Used directly in m17-mod, not used in m17-demod.
### Correlator.h
Detects matching preambles and sync words in the symbol stream.
### CRC16.h
Computes CRC values.
### DataCarrierDetect.h
Decides whether a signal is present, by comparing the power in-band to the power out of band. (This is probably not a practical method for on-the-air use.)
### Filter.h
Provides virtual base class for filters, FilterBase.
### FirFilter.h
Provides finite impulse response (FIR) filter, BaseFirFilter.
### FreqDevEstimator.h
Processing at the sample level, estimates the deviation and frequency offset of the received signal.
### Golay24.h
Implements Golay encoder and decoder.
### IirFilter.h
Provides infinite impulse response (IIR) filter, BaseIIRFilter, used by Correlator, FreqDevEstimator, SymbolEvm.
### LinkSetupFrame.h
Provides encoder and decoder for callsigns in the LSF. Aspires to help with other parts of the LSF.
### M17Demodulator.h
Main processing for the demodulator. Orchestrates all aspects of operation, including the sample-level processing, state machine, and frame-level processing.
### M1FrameDecoder.h
Frame-level processing for all supported frame types.
### M17Framer.h
Constructs frames from decoded symbols.
### M17Randomizer.h
Implements the data decorrelator, XOR with a known sequence. Used in both directions.
### PolynomialInterleaver.h
Implements the Quadratic Permutation Polynomial (QPP) interleaver. Used in both directions.
### queue.h
Thread-safe queue funtions. Used in m17-mod to handle incoming PCM audio.
### SlidingDFT.h
Implements a sliding DFT. Used by DataCarrierDetect.
### Trellis.h
Defines puncture matrixes. Implements trellis primitives for the Viterbi decoder.
### Util.h
LLR primitives; puncturing primitives; bit and byte manipulation;
PRBS9 pseudorandom bit sequence generator, synchronizer, and validator; etc.
### Viterbi.h
Viterbi decoder for the convolutional code.

## Unused Files
These files are in the source base but not currently part of the build. Some of them may be obsolete. Some of them are part of work in progress to modularize the modulator.
### CarrierDetect.h
Not used.
### DeviationError.h
Not used.
### FrequencyError.h
Not used.
### Fsk4Demod.h
Not used.
### M17Modulator.h
Not used. Aspires to take over most functions from the main module of m17-mod.
### M17Synchronizer.h
Not used.
### PhaseEstimator.h
Not used.
### SymbolEvm.h
Not used.