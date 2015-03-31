## NICE ##

NICE is a tool to test [OpenFlow](http://www.openflow.org/) controller application for the [NOX](http://noxrepo.org/) controller platform through a combination of model checking and symbolic execution.

The emergence of OpenFlow-capable switches enables exciting new network functionality, at the risk of programming errors that make communication less reliable. The centralized programming model, where a single controller program manages the network, seems to reduce the likelihood of bugs. However, the system is inherently distributed and asynchronous, with events happening at different switches and end hosts, and inevitable delays affecting communication with the controller. In this paper, we present efficient, systematic techniques for testing unmodified controller programs. Our NICE tool applies model checking to explore the state space of the entire system—the controller, the switches, and the hosts. Scalability is the main challenge, given the diversity of data packets, the large system state, and the many possible event orderings. To address this, we propose a novel way to augment model checking with symbolic execution of event handlers (to identify representative packets that exercise code paths on the controller). Our prototype tests Python applications on the popular NOX platform.



## Documentation ##

NICE documentation index:
  * [Introduction](DocIntroduction.md)
  * [Requirements](DocRequirements.md)
  * [Running an application in NICE](DocRunning.md)
  * [Adding a new application in NICE](DocNewApplication.md)
  * [Invariants](DocInvariants.md)
  * [Strategies](DocStrategies.md)
  * [Known Issues](DocKnownIssues.md)

## Publications ##

  * [A NICE Way to Test OpenFlow Applications](http://infoscience.epfl.ch/record/170618), Marco Canini, Daniele Venzano, Peter Perešíni, Dejan Kostić, and Jennifer Rexford, Proceedings of the 9th USENIX Symposium on Networked Systems Design and Implementation (NSDI '12), April 2012.
  * [A NICE Way to Test OpenFlow Applications](http://infoscience.epfl.ch/record/169211), Marco Canini, Daniele Venzano, Peter Perešíni, Dejan Kostić, and Jennifer Rexford, EPFL Technical Report EPFL-REPORT- 169211, October 2011.
  * [Automating the Testing of OpenFlow Applications](http://infoscience.epfl.ch/record/167777), Marco Canini, Dejan Kostić, Jennifer Rexford, and Daniele Venzano, Proceedings of the 1st International Workshop on Rigorous Protocol Engineering (WRiPE), October 2011.