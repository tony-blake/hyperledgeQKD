The submission is organized by folders: each folder called exercise-[N] implements completely the Nth exercise defined in the project specification. Each exercise can be run in its folder, by executing sh run_example.sh. All exercises but the first require an input, the number of qubits to be sent. So they should be run with: sh run_example <n> where <n> is an integer, the number of qubits sent.
All folders have a communication.py file to handle classical communication, which uses python sockets (instead of the service provided in cqc) created in the defined host and port in the appNodes.cfg file (same address and port that would be used by cqc)

- QuaRT:
- sender.py and receiver.py are the main files of this project.
    sender receives the path of the file to send and calls the send function of the Sender class which does:
       - calculate the number of qubits to be sent given the file length and custom parameters (defined in qkd.conf with the specification in qkd.spec)
       - sends the file length, filename and qkd parameters to the receiver
       - performs the BB84 protocol with the receiver: sends qubits, performs basis sift, error estimation, error correction and privacy amplfication
       - uses one time pad to encrypt the file and send to receiver

     receiver:
       - receives the parameters from sender and calculates the number of qubits that will receive
       - performs the BB84 protocol with the sender: receives qubits, performs basis sift, error estimation, error correction and privacy amplification
       - receives the encrypted message from the sender, uses one time pad to decrypt it, print it in the console and write it to a file called receiver_name+_+filename
    The error estimation step stops the protocol if the channel is too noisy or if the protocol simply didn't produce enough bits of sifted key to generate a key of desired length with security of the security_parameter, printing "Not enough min-entropy :("
    Even if there are no errors (or the error estimation is 0.0), the protocol will run error correction with estimate 0.01
    This is because we are using small keys (too many qubit exchanges are slow) and it's harder to estimate the error with small keys. It's a "Better safe than sorry" approach.
    The same reasoning applies to the number of qubits being sent on the protocol.
    This was tested with the noisy qubits option set to True and works (has some chance of error, bigger the smaller the key is) 

- eavesdropper.py redirects all qubits from sender to the receiver and quits after not receiving qubits for a certain time (the defined timeout for cqc)
   While it's not performing any attack, it's interesting to keep this class to allow experimenting attacks and see how easily they are noticed by the protocol

- authentication.py provides utility functions to deal with the authenticated channel. This includes:
    - generating EC-DSA private and public keys for sender and receiver
    - publishing the public keys and retrieving the other party public key
    - sign and verify messages

- communication.py provides utility functions to communicate classically. This includes:
    - send and receive regular messages
    - send and receive lists or binary lists
    - (de)serialize lists of bits to bytes to improve classical communication efficiency
  this module makes use of the authentication module to sign and verify every message.

- error_correction.py provides two classes for the cascade error correction algorithm.
    - These classes are called by sender and receiver and are responsible for running the algorithm
    - They are separated by behaviour, one implements the behaviour of the sender and the other of the receiver (CascadeSender, CascadeReceiver)

- progress_bar.py provides the function to draw the progress bar in the send qubits phase

- utils.py provides utility functions

- config.py interacts with the configuration file to define the QKD parameters
   by default it uses the parameters in qkd.spec if exists a file qkd.conf with the specification in qkd.spec it will use that file.

- run_example.sh runs one file transfer being called: sh run_example.sh <filename>
