# Running an application in NICE #

The script `nice/run_demo.sh` should be used to test if the NICE distribution is installed correctly.
```
$ cd nice
$ ./run_demo.sh
```

Should output something like:
```
NICE (No Bugs In controller Execution): systematic testing of OpenFlow controller programsInstrumenting se_dict.py
Instrumenting nox/lib/core.py
Instrumenting nox/lib/openflow.py
Instrumenting nox/lib/util.py
MC init complete
Instrumenting nox/lib/packet/packet_utils.py
Instrumenting nox/lib/packet/ethernet.py
Instrumenting nox/lib/packet/__init__.py
Instrumenting nox/lib/__init__.py
Instrumenting nox/__init__.py
Instrumenting pyswitch.py
Instrumenting custom_constraints.py
nice.inv: Invariant violation: NoDrop: Dropped packets: pkt_reply, pkt_reply (1.44s, 34 trans, 34 states, path len 34)
nice.inv: Invariant violation: StrictDirectRoute: Strict: Packet should have gone on a direct route! (1.68s, 124 trans, 85 states, path len 29)
nice.inv: Invariant violation: NoDrop: Dropped packets: pkt_reply, pkt_reply (2.08s, 291 trans, 184 states, path len 34)
nice.inv: Invariant violation: StrictDirectRoute: Strict: Packet should have gone on a direct route! (2.23s, 345 trans, 216 states, path len 29)
nice.inv: Invariant violation: NoDrop: Dropped packets: pkt_reply, pkt_reply (2.55s, 478 trans, 295 states, path len 34)
nice.inv: Invariant violation: StrictDirectRoute: Strict: Packet should have gone on a direct route! (3.80s, 944 trans, 521 states, path len 31)
nice.inv: Invariant violation: NoDrop: Dropped packets: pkt_reply, pkt_reply (3.80s, 950 trans, 527 states, path len 36)
nice.inv: Invariant violation: StrictDirectRoute: Strict: Packet should have gone on a direct route! (4.05s, 1025 trans, 576 states, path len 31)
nice.inv: Invariant violation: NoDrop: Dropped packets: pkt_reply, pkt_reply (4.05s, 1031 trans, 582 states, path len 36)
Total: 1225, unique: 685, revisited: 540, max path len: 36, violations: 13, SE calls: 1385 (302 tr/sec)
```

To run one of the examples, or your own configuration file, you need to execute the following command:

```
$ cd nice
$ ./nice.py config/pyswitch.conf (or any other config file)
```

The directory `nice/config/paper` contains the configuration files used to reproduce the bugs reported in the paper.

Nice generates a big amount of output that can significantly slow down its execution, moreover if one is using a remote terminal. To reduce and regulate the output generated the options `–p` and `–l` are available.
  * `-p` will start Nice in “demo” mode, printing a progress line and the invariant violations in red
  * `-l <filename>` will save the output in the specified filename, printing only errors and invariant violations
  * `--help` to see all other options.

In the configuration file, under the runtime section, there is a log\_level option that can be used to fine-tune the amount of output.