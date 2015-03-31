# Invariants #

Invariants are used to check correctness properties during NICE execution.
They collect data during the transitions explored by the  model checker and check for violations.

The following invariants represent well-behaved networks and correct use of the NOX API. They are enabled by default for all applications:
  * NoDrop: packets should never get dropped (aka no black holes)
  * NoForgottenPackets: no packets should be left in the switch buffers by the controller
  * ReturnContinueStop: the packet\_in handler should always return either `CONTINUE` or `STOP` (NOX API check)

Applications can add their own invariants to check properties specific to their behavior.

All invariants reside in `model_checker/invariants/`.  They are classes that derive from `Invariant`:
```
class NoDropInvariant(Invariant):
    def __init__(self):
        Invariant.__init__(self, "NoDrop")
```

A number of test points have been added throughout the NICE code. The invariants can hook up to one or more of these test points and have access to the state of the model during the execution.

An invariant can declare interest in a test point by declaring a method with a special name in its class:
```
def <test point name>_cb(self, <list of arguments>):
```

For example, the callback for the `packet_sent` test point is called `packet_sent_cb`.

Each test point passes to the invariant parameters that are specific to the event that is happening.

The test points currently defined are (see also `model_checker/invariants/invariant.py`):
  * **transition\_start**: just before the model checker starts to execute the transition
    * model: reference to the model
  * **transition\_end**: just after the transition has been completed
    * model: reference to the model
  * **path\_start**: a new exploration is going to start
    * model: reference to the model in the initial state
  * **path\_end**: the model checker reached a state where no more transitions can be executed
    * model: reference to the model
    * cached\_state: boolean, if True means that the execution has finished early because DPOR or state caching reported the state as not worth exploring
  * **packet\_received**: whenever a packet is received by any node, except the controller
    * receiver: reference to the receiving node
    * packet: the packet being received
    * port: the port on which it is being received
  * **packet\_sent**: whenever a packet is sent by an end-node (those in `model_checker/clients/`)
    * sender: reference to the sender node
    * receiver: reference to the receiving node
    * packet: the packet being sent
  * **before\_cnt\_packet\_in**: the controller's packet\_in callback is going to be called
    * same parameters as passed to the controller's packet\_in callback
  * **after\_cnt\_packet\_in**: the controller packet\_in callback has finished execution
    * controller: reference to the controller node
    * packet: the packet passed to the callback
    * return\_value: the return value of the callback
  * **switch\_flood\_packet\_start**: a switch is going to start a flood
    * switch: reference to the switch
    * packet: the packet being flooded
  * **switch\_sent\_packet\_on\_port**: a switch is sending a packet
    * switch: reference to the switch
    * packet: the packet being sent
    * port: the port used for sending
  * **switch\_enqueue\_command**: the controller queued a command in the switch
    * switch: reference to the switch
    * command: the queued command
  * **switch\_process\_packet**: a switch is processing a packet
    * switch: reference to the switch
    * packet: the packet being processed
    * port: the port where the packet was queued/received
  * **syn\_packet\_received**: a SYN packet was received by a TCP server
    * receiver: reference to the receiving node
    * packet: the received packet
  * **ack\_packet\_received**: an ACK packet was received by a TCP server (handshake completed)
    * receiver: reference to the receiving node
    * packet: the received packet
  * **tcp\_connection\_start**: a TCP client is initiating a connection
    * client: reference to the TCP client
    * packet: the received packet

Whenever a violation is found, the invariant must inform NICE by instantiating a Violation object:
```
v = Violation(self, "Dropped packets")
self.reportViolation(v)
```

A human-readable description of the violation must be passed to the constructor. The string is used in the reporting and is free-form.