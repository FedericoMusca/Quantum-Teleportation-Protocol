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
   

7.   Alice waits for a response from Bob to learn if he completed the teleportation. She will then end the protocol.



Bob's protocol performs classical corrections. The control flow is:

1.   Initialize state variables.

2.   Wait for classical measurements from Alice and Bell-state from *bp_source* to arrive. If Alice experienced too many timeouts, she will send an error code instead of the measurements, indicating to end the protocol.

3.   When both arrive, perform corrections based on Alice's measurements.

5.   Measure fidelity and store in *self.fidelity*.

6. Send Alice a success message, then end protocol.

### Simulation and results

Analyzed parameters: overall (Bob's) fidelity, Alice's initialized qubit memory idle time, timeout counts from Alice's side.

### No loss scenario:
All the analyzed parameters are plotted over link lenghts of the experimental network infrastructure. No memory noise or fiber loss was modeled.

1. Fidelity: exponential decrease with no variance for link or memory noise.

2. Alice's qubit memory idle time: linear increase, no variance.

3. Alice's timeouts. 0 timeouts recoarded. No variance

### Alice loss scenario:
Memory noise and fiber loss within the Bell-pair source - Alice link are modeled.

1. Fidelity: We see variance due to
probabilistic time in memory due to probabilstic loss and retransmission.
Alice's qubit to send (qubit_x) will experience a random time in memory
dependent only on the probability of loss, therefore memory noise has
variance

2. Alice's qubit memory idle time: a linear increase trend can stll be observed, with variance due to simulated memory noise
  
3. Alice's timeouts: timeouts increase semi-linearly to a threshold with variance due to loss.

## Part2
In this second part of the project both Alice and Bob could experience photon losses during the distribution of Bell pairs from the Bell Pair Source. The RequestAndMeasure Protocol and the Receive Protocol will be updated to manage such scenarios

### Alice and Bob updated Protocols

*Alice's protocol* is pretty similar to the one presented in the first part of the project. Instead, Bob's one needed some changes in order to manage the photon transmission from Bell Pair Source.

* First, Alice initializes the y0 qubit in her memory via InitQubitXProgram.
Then, she requests the entangled pair distribution to the Bell Pair Source.
Actually, to perform such operation, she must be synchronized with Bob protocol: at the time the request is sent, both the nodes must be ready to activate simultaneously the "timeout window". In the first iteration, Alice will wait for an "initialization" message by Bob, as a sort of handshake before starting the teleportation scheme. After she understood Bob is ready, she can notify him that she's requesting the qubits, so that Bob knows which is the moment to "activate" the timeout window

* Different scenarios can then occur.

  1. If both Alice and Bob receives the corresponding qubit, Alice will not request the distribution again, and the flow of operations follows exactly the one reported in the first part of the project (bell state measurement and photonic state retrieved by Bob with the proper correction applied to his qubit).

  2. If one between Bob and Alice receives the qubit while the other party experiences photon loss, Alice asks again the Bell Pair Source for retransmission and the protocol can restart. This is guaranteed by the exchange of messages between the nodes: "TIMEOUT" if one experiences a timoeut, "SUCCESS" if receives correctly the qubit or "(-99, -99)" if experienced 5 timeouts.
  
    In this case, the initial handshake is performed again, with Bob sending a message saying he's ready and Alice answering by requesting the entangled pair of qubits. This was made because I noticed that in such scenarios Alice is the party that always arrives first at the head of the loop and cannot start the bell pair request if Bob is not ready. She must wait for him and perform again the "initialization" handshake.

  3. If both Alice and Bob experienced a timeout, they will communicate "TIMEOUT" to the other peer. Alice's protocol keeps two boolean flags, to understand when Bob and Alice herself had a timeout. This was crucial to guarantee a correct initial synchronization at the beginning of the scheme when both the nodes experienced a timeout: I noticed that in this scenario the one who always arrived first in the head of the loop was not Alice but Bob, therefore there was not the need to re-perform the initialization hanshake again: Bob in this case is always ready waiting for Alice to send the request! (otherwise the scheme would have been stalled).  


  * Every time a timeout occurs the "timeout_count" is increased: reaching 5 timeouts is the condition that terminates the teleportation scheme.
 

### Results

The analyzed parameters now are: overall (Bob's) fidelity, Alice's initialized qubit memory idle time, timeout counts for bot Alice and Bob nodes. The "no loss scenario" is exactly the same as *Part1*

### Alice and Bob loss scenario

1. Fidelity: increased exponential decrease rate for fidelity with variance due to memory noise due
to probabilistic increased time in memory. This beacuse of the increased loss across both links with
respect to the "Alice only" solutions.

2. Alice's qubit memory idle time: in the end the trend of the curve is less predicatble. This is due to the implementation that I propose, where at the beginning of the scheme the "initialization handshake" (necessary for the correctness of the protocol) introduces a non negligible time interval spent to synchronize the parties. Therefore, for long distances, it's natural to see a great variation of such idle time with a relevant variance in the plot, still guaranteeing the logic correctness of the same. For shorter distances instead, up to the second half of the plot, the memory idle time follows a very appreciable "linear" trend.

3. Alice's and Bob's timeout: timeouts increase to a threshold more quickly and with more variance due to increased
loss with respect to the Alice only solutions.
