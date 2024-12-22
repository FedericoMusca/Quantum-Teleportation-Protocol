# Quantum-Teleportation-Protocol

## Part1

In this first part of the project I designed a quantum teleportation protocol between 3 nodes: Alice, Bob, and a Bell-Pair distribution source. Upon request, the Bell-pair source will distribute one Bell-state to Alice and Bob each. Alice will then perform a Bell-State Measurement (BSM) on her qubit X and her Bell-state and send the classical results to Bob, who will perform corrections to convert his Bell-state into qubit X.

### Network Entities
Each *Node* will share the same *QuantumProcessor*, qprocessor.

*bp_source:* *bp_source* is a *Node* that has a *QSource* subcomponent. In response to classical distribution requests from Alice, the *Node's* *QSource* will distribute Bell-pairs on-demand to Alice and Bob.

*AliceNode*: Alice is a *Node* running the *RequestAndMeasureProtocol*. The simulation will start by Alice requesting BP distribution from *bp_source* (for simplicity we will assume Bob is always ready when Alice requests). At this time, the protocol will also run a *QuantumProgram* to initialize qubit X in memory. When the qubit is initialized and the Bell-state is received, the protocol will run a *QuantumProgram* to perform a BSM on qubit X and the Bell-state. The protocol then sends the classical results to Bob.

Finally, the protocol must be repeated if a Bell-state is lost within the fiber due to attenuation. In reality, this could occur on either fiber from *bp_source* to Alice or Bob, but for simplicity,  **we will only assume a Bell-state can be lost from *bp_source* --> Alice.** If Alice does not receive a Bell-state within a certain timeout window, she will request distribution again.

*BobNode*: Bob is a *Node* running the *ReceiveProtocol*. The protocol will wait for both the Bell-state from *bp_source* and the classical measurements from Alice. When both arrive, Bob may perform corrections on his qubit to create the teleported state, qubit X.

### Protocols

Alice's *RequestAndMeasureProtocol* will

1.   Initialize state variables and *QuantumPrograms*.


2.   Initialize qubit X in Alice's memory using *InitQubitXProgram*.


3.   We track the time the qubit idles in memory before the Bell-state measurement.


4.   Request BP distribution from *bp_source* using a classical message. Any input on *bp_source*'s 'trigger' port will cause distribution.


5.   Alice will wait for the Bell-state to arrive, or her timeout to trigger, which indicates the photon was lost. If 5 timeouts occur, she will end the protocol. She needs to inform Bob that we are ending the protocol, so she sends him the failure message (-99, -99).


6.   When the Bell-state arrives and qubit X is initialized, Alice runs the *BellMeasurementProgram* qprogram and sends the classical results to Bob. We store the total time qubit X was idling in qmemory before the BSM.
7.   

8.   Alice waits for a response from Bob to learn if he completed the teleportation. She will then end the protocol.



Bob's protocol performs classical corrections. The control flow is:

1.   Initialize state variables.

2.   Wait for classical measurements from Alice and Bell-state from *bp_source* to arrive. If Alice experienced too many timeouts, she will send an error code instead of the measurements, indicating to end the protocol.

3.   When both arrive, perform corrections based on Alice's measurements.

5.   Measure fidelity and store in *self.fidelity*.

6. Send Alice a success message, then end protocol.

### Simulation and results



