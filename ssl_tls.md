
# TLS introduction

## Record protocol

* The basic transport protocol.
* Comprises of TLS Record
    ```
    RecordProt Header     -- Data
    Version/Len/ContType     subtype/ver/length/Mandatory-fields / Extension-fields
    ```
    * ContentType - change_cipher_spec (20), alert (21), handshake (22), application_data (23)
    * Each record is assigned a seq-number(not sent on wire), that helps in preventing replay
* Has encryption and integrity validation
* Compression - never used
* Extensibility - well, does't do anythig beyond. Opaque transported - so basically fits everywhere.

## Handshake protocol

### Full Handshake

* Full-Handshake - first time when no apriori exchange has happened.
    * Exchange capabilities and agree on desired connection parameters.
    * Validate the presented certificate(s) or authenticate using other means.
    * Agree on a shared master secret that will be used to protect the session.
    * Verify that the handshake messages haven’t been modified by a third party.
* Exchanges
  ```
        Client                        Server
                --- Client Hello -->
                <-- Server Hello ---
                <-- Certificate  ---
                <-- ServKeyExchange ---
                <-- Certificate Req ---          (mutual TLS)
                <-- ServHelloDone   ---
                --- Certificate     -->          (mutual TLS)
                --- ClientKeyExchge -->
                --- Certificate Verify -->       (mutual TLS)
                --- [ChangeCipherSpec] -->
                --- Finished           -->
                <-- [ChangeCipherSpec] ---
                <-- Finished           ---
  ```
    * Client begins a new handshake and submits its capabilities to the server.
    * Server selects connection parameters.
    * Server sends its certificate chain (only if server authentication is required).
    * Depending on the selected key exchange, the server sends additional information required to generate the master secret.
    * Server indicates completion of its side of the negotiation.
    * Client sends additional information required to generate the master secret.
    * Client switches to encryption and informs the server.
    * Client sends a MAC of the handshake messages it sent and received.
    * Server switches to encryption and informs the client.
    * Server sends a MAC of the handshake messages it received and sent.
* Clock sync isn't requried.
* Both client and server contribute to random data in exchange -- making each handshake unique
* Client-Hello -- sessionId is empty => its a fresh handshake
* Server-Key-Exchange is optional and gives addition data for the selected key exchange
* Client-Key-Exchange is mandatory and gives client contribution to the key exchange
* The ChangeCipherSpec message is a signal that the sending side obtained
  enough information to manufacture the connection parameters, generated the
  encryption keys, and is switching to encryption. Client and server both send
  this message when the time is right.
  * Note the ChangeCipherSpec is a type in its own right (not handshake type)

#### Mutual TLS 

* server asks for client certificate
* client in the CertificateVerify prooves its possession of private key by
  sending a sign of all handshake messages so far

### Quick Handshake / Session Resumption

* Server gives the client a session-id in its hello. Client replays that in its hello in
  a subsequent quick-handshake

  ```
        Client                        Server
                --- Client Hello -->
                <-- Server Hello ---
                <-- ChangeCipherSpec  ---
                <-- Finished     ---
                --- ChangeCipherSpec -->
                --- Finished     -->
  ```
* Alternative is a session ticket where the server issues a session-ticket that puts the
  burden of storing all details on the client.



# dump notes

openssl genrsa -aes128 -out fd.key 2048
