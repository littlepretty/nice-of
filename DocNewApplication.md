# Adding a new application in NICE #

  * Controller subclass
  * Models
  * NICE configuration
    * Model
    * Model-specific options
    * Strategy
    * Runtime options
  * NOX app state
  * Invariants
  * Symbolic engine configuration


---


Adding support to a new application inside NICE is better done with a two-step approach. First configure the application to run with predetermined fixed inputs in the model checker. When this works, the second step consists in using the symbolic engine to generate inputs dynamically.

To run with the model checker, the following things are needed:
  1. A copy of the application in its own subdirectory under nox\_app
  1. A Controller subclass that refers to the application
  1. A model that describes the network topology and the invariants to be checked
  1. A configuration file containing the application location, runtime, model and strategy options
  1. A getstate() function that returns a checkpoint of the application state

To run with the symbolic engine:
  1. A Python description of the application
  1. Custom constraints to apply to the generated inputs

In the following sections we use the Pyswitch application, to be tested with the following network topology:

![http://wiki.nice-of.googlecode.com/hg/images/sample_network.png](http://wiki.nice-of.googlecode.com/hg/images/sample_network.png)

## Controller subclass ##

Controller subclasses are defined in `model_checker_cmc/of_controller/`. They must contain an `__init__()` method that imports the application module and calls the `__init__()` method of the parent, passing the application module as one of the arguments.
```
class PySwitchController(Controller):
	"""A pyswitch controller"""
	def __init__(self, name, ctxt, version="pyswitch"):
		if version == "pyswitch":
			import pyswitch.pyswitch as pyswitch_mod
			Controller.__init__(self, name, pyswitch_mod.pyswitch, ctxt)
		elif version == "wildswitch":
			import wildswitch.wildswitch as wildswitch_mod
			Controller.__init__(self, name, wildswitch_mod.wildswitch, ctxt)
		else:
			assert False

	def isSameMicroflow(self, packet1, packet2):
		return (packet1.src == packet2.src and packet1.dst == packet2.dst)
```

This example shows how to customize the instantiation by passing a parameter.

The method `isSameMicroflow` is optional and used only by the `HeuristicMicroflowIndependence` strategy. Given two packets, it returns True if the two packets belong to the same flow, False if they don’t.

For timer callbacks (NOX's `post_callback`), see the `LoadBalancerController` implementation for an advanced example on how they can be run inside Nice. In that code, a particular ordering between two callbacks is enforced.

## Models ##

Models reside in the `model_checker_cmc/models` directory. They are classes that derive from `Model` and implement the `initTopology` method. The method takes an argument, topo, that is not currently used.

Here is a commented example of a model of this network:
```
class PyswitchModel(Model):
    def initTopology(self, topo):
        # instantiate the controller application, passing additional options
        self.controller = PySwitchController(name="ctrl", \
                   ctxt=self.of_context, version="pyswitch")
        # give the controller to the OF context
        self.of_context.setController(self.controller)
        # create two switches with 2 ports each
        # of_id is the openflow ID (the dp_id)
        sw1 = OpenflowSwitch(name="s1", port_count=2, of_id=1)
        sw2 = OpenflowSwitch(name="s2", port_count=2, of_id=2)
        mac1 = (0x00, 0x01, 0x02, 0x03, 0x04, 0x00)
        mac2 = (0x00, 0x01, 0x02, 0x03, 0x05, 0x05)
        # instatiate an Arrival client with mac1 that sends pkts ethernet frames to
        # mac2, sequentially (or not)
        cl1 = Arrival(name="h1", mymac=mac1, dstmac=mac2, \
                   pkts=self.config.get("pyswitch_model.pkts"), \
                   sequential=self.config.get("pyswitch_model.sequential"))
        # cl2 is a replier (ping server) that generates a reply whenever a packet
        # is received
        cl2 = Replier(name="h2", mymac=mac2)
        # Topology for sw1
        # port 0 is connected to port 0 on cl1
        # port 1 is connected to port 0 on sw2
        sw1.initTopology({0: (cl1, 0), 1: (sw2, 0)})
        # Topology for sw2
        # port 0 is connected to port 1 on sw1
        # port 1 is connected to port 0 on cl2
        sw2.initTopology({0: (sw1, 1), 1: (cl2, 0)})
        cl1.initTopology({0: (sw1, 0)})
        cl2.initTopology({0: (sw2, 1)})
        # Connect the switches to the controller
        sw1.setController(self.controller)
        sw2.setController(self.controller)
        # Add the clients
        self.clients.append(cl1)
        self.clients.append(cl2)
        # Add the switches
        self.switches.append(sw1)
        self.switches.append(sw2)
        self.switches_idx[sw1.getOpenflowID()] = sw1
        self.switches_idx[sw2.getOpenflowID()] = sw2
        # invariants (see the Invariants section)
        self.invariants = [NoLoopInvariant(), NoDropInvariant(), \
                   DirectRouteInvariant(), StrictDirectRouteInvariant()]
        # callbacks to call whenever a new path exploration starts
        self.controller.start_callbacks.append(lambda: self.controller.install())
        self.controller.start_callbacks.append(lambda: self.controller.addSwitch(sw1))
        self.controller.start_callbacks.append(lambda: self.controller.addSwitch(sw2))
```

## NICE configuration ##

Configuration files reside in `nice/config/`. They are written in a INI file format. Next each section is explained in detail. The code block represent default values that are used if the corresponding entry is not defined in the configuration file.

The default values can be seen also in the `common_modules/config_parser.py`.

### Model ###

  * `name`: the class name of the application (the subclass of Component)
  * `app_dir`: the directory under `nox_apps` that contains the application files. It is expected that the directory and the main module of the application have the same name.
  * `app_descr`: configuration files used by the symbolic engine
  * `cutoff`: maximum path length to explore. -1 means no cutoff
  * `faults`: number of faults to inject in the swiches
  * `flow_entry_expiration`: (true/false) enables or disables the expiration of flow entries. Setting this to true will have a big impact on running time

Example:
```
[model]
name = NiceModel
app_dir = pyswitch
app_descr = apps/pyswitch/se_descr.py
cutoff = -1
faults = 0
flow_entry_expiration = false
```

### Model-specific options ###

```
[nice_model]
max_pkts = 2
max_burst = 1
```

### Strategy ###

  * `name`: name of the strategy class
  * `dpor`: enable Distributed Partial Order Reduction (experimental)

Example:
```
[strategy]
dpor = False
name = CompleteCoverage
```

### Runtime options ###
  * `log_level`: output log level. Possible values are `debug`, `info`, `warning`, `error` and `critical`.
  * `replay_debug`: enable additional checks to debug inconsistent states after replay
  * `test`: do random walks until an exception is thrown
  * `progress`: demo mode (equivalent to –p option)
  * `graph`: filename where to save the dot formatted graph

Example:
```
[runtime]
log_level = info
replay_debug = false
test = false
progress = false
graph = none
```

## NOX app state ##

The model checker needs to be able to serialize the state of the application. This is done to implement state caching and to recognize states where the symbolic execution can find new inputs.

In the `Component` subclass that implements the application, the method `__getstate__(self)` needs to be implemented. It should return a dictionary that represents the current internal state of the application. Python will call automatically `__getstate__` on non-standard objects referenced in the returned dictionay. Python's default is to use the object's dict field.

The returned dictionary will be serialized with cPickle to a hashable string.

The state must not contain any time dependent value (timeouts, timestamps), nor any randomly generated value.
Internally Nice will reinitialize the seed of the random module, so if you do not instantiate a new number generator or reset the seed, random values should not affect the model checker exploration.

Example:
```
def __getstate__(self):
    """ Added to serialize the app with only the necessary state """
    di = Component.__getstate__(self)
    di["st"] = {}
    dpids = self.st.keys()
    dpids.sort() # dictionaries do not guarantee any order
    for d in dpids:
        di["st"][d] = {}
        macs = self.st[d].keys()
        macs.sort()
        for m in macs:
            # set to zero the timestamp field in the tuple
            di["st"][d][m] = (self.st[d][m][0], 0, self.st[d][m][2])
    return di
```

## Invariants ##

See the [Invariants](DocInvariants.md) chapter.

## Symbolic engine configuration ##

The symbolic engine requires a Python description of the application that contain:
  * the modules it is composed of
  * the callbacks that need to be executed and their parameters
  * a reset function
  * custom constraints

The custom constraints are needed to tell the engine which MAC addresses are used in the model.

To run NICE with the symbolic engine, the model must contain one `sym_exec_sender` client. That client will call the symbolic engine whenever new packets need to be injected in the model.

The `run_demo.sh` script executes NICE with the symbolic engine enabled on the `PySwitch` application. The script runs NICE witht the `config/pyswitch.conf` configuration file. Check also the files under `nice/apps/pyswitch` for examples on calling the `packet_in` callback and `nice/apps/eate*` for symbolic statistics.