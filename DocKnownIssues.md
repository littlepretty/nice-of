# Known issues #

## Graph generation ##

The model checker is capable of generating a DOT graph of the state space. Unfortunately it becomes very slow and memory hungry for all graphs with more than a few hundreds of nodes. While it makes for beautiful posters, we suggest using this feature only for very small state spaces.

## Symbolic execution ##

The symbolic engine currently generates only Ethernet packets, additional symbolic packets types have to be added to the Nox API reimplementation. Also the symbolic engine currently executes only the statistics and `packet_in` callbacks, passing symbolic packets.