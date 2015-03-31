# Strategies #

The exploration strategies are described in detail in the NICE paper. Currently the following are available and can be set in the configuration file:
  * CompleteCoverage: cover all the state space, using only state caching as optimization
  * HeuristicConstCtrlDelay: heuristic that assumes a constant delay (4 transitions by default) for all communications with the controller
  * HeuristicMicroFlowIndependence: heuristic that does not try to reorder actions belonging to different (independent) micro flows. Needs support from the controller class.
  * HeuristicNoCtrlDelay: heuristic that assumes no delay in all communications with the controller

DPOR can be combined with CompleteCoverage or any heuristic, excluding the MicroFlowIndipendence. DPOR is in development and currently at an experimental stage.