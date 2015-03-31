# Introduction #

NICE is built around two major components, the symbolic engine and the model checker.

The symbolic engine is called by the model checker when the network model requires the generation of new packets to inject. The symbolic engine uses several advanced techniques to trace the execution of the Python interpreter. For this reason it is heavily dependent upon the Python interpreter version in use and the code being executed.

Currently most of the more common Python opcodes are implemented, but new code could require implementation to support new constructs.

Also the symbolic engine requires a C library (the STP solver). A pre-compiled object is provided in the distribution package for Linux in both 64bit and 32bit flavours.

The model checker is written in standard Python code. It is based on the concept of a model that describes the network topology in terms of clients, switches, controller and links between them. Currently the topology is encoded in Python, but support for the Mininet format is in the working.

While there is only one type of switch, there are several types of clients to choose from, with and without TCP support. A particular client type uses the symbolic engine to create new packets to inject in the network.

While some invariant checks are hardcoded for obvious bugs, common to all possible Openflow implementations, the model specifies also other invariants that should be checked during each transition of the model checker. Invariants use a system of events and callbacks to hook into the model checker execution and inspect the system state.

In the rest of the documentation the following conventions apply:
  * All file paths described assume as root, the root of the NICE tarball
  * The term application always refers to the Nox Component subclass that implements the application to be tested