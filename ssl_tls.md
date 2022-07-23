
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


## Key Exchange

* In TLS, the security of the session depends on a 48-byte shared key called
  the master secret.
* The goal of key exchange is to generate another value, the premaster secret,
  which is the value from which the master secret is constructed.
* No matter which key exchange is used, the server has the opportunity to speak
  first by sending its ServerKeyExchange message

## Interesting Extension fields

* SNI
    * Server name Information
    * Wireshark filter - `tls.handshake.extensions_server_name == 'gxc.io'`

# PKI

## Entities

* subscriber who provide secure services
    * wants to share his information using his certificate
    * Eg: a domain-owner like https://reachmesecurely.com,
         ipsec-server, individual wanting to sign documents
* RA (Registration authority)
    * validates subscriber's request for a certificate
    * carries mgmt functions
* CA (certificate authority)
    * issues those root certificates and revocation lists
* relying party
    * the consumer of the certifcate (web-browser)

## Revocation

CRL - Certificate revocation list
OCSP - Online Certificate Status Protocol

## Encoding

* Certificates are in ASN.
* ASN has BER(Basic encoding rules) and DER(distinguished ER) is a stricter way of using BER
* PEM (privacy enhanced mail) is a base64 encoded DER
* PKCS#7 certificate(s)
    * typically with `.p7b` or `.p7c` extention.
    * rfc3215
    * can embed entire cert-chains
    * supported by java tools
* PKCS#12
    * typically with `.p12` or `.pfx` extention.
    * A complex format that can store and protect a server key
      along with an entire certificate chain

## Certificate fields

* Version
* Serial - uniquely idenfies a certificate issued by CA
* Algo/Issuer/Validity
* Subject
    * DN - distinguished name. Has components below
        * CN - common name
    * Deprecated. Alt Subject name is used now
* Pub Key
* Extensions
    * Subjet Alt Name
        * supports multiple identities - IP, DNS-name, URI
    * Name contraints
        * limits to what a CA (the interim CA that is to which this cert belongs) can sign

## Certifiate lifecycle

* CSR - cert signing request
    * pub-key to sign.
    * proof of posession of private key (using a sign)
        * By principle a csr can be verified
* Domain Validation
    * Eg. send a email to an approved email and following the link givein.
* Organization Validation
    * Lots of inconsitencies here
* Extended Validation
    * covers gaps in Organization Validation

    * proof of posession of the domain/id

# OpenSSL

General openssl arg meaning
```sh
#get to know
openssl version

#list all commands
openssl help

-nodes      :   no encrpyption of private key if created.
```

## Key generations

```sh

# rsa key of a given strength
openssl genrsa  -out site-root.key 4096

#generate a rsa key - interactive - asks for paraphrase
openssl genrsa -aes128 -out fd.key 2048

#show the key
openssl rsa -in fd.key -text -noout

#get the pub key
openssl rsa -in fd.key -pubout -out fd-public.key

#dsa is a 2 step process
openssl dsaparam -genkey 2048 | openssl dsa -out dsa.key -aes128

#ecdsa -- you give the name of the curve
openssl ecparam -genkey -name secp256r1 | openssl ec -out ec.key -aes128

```

## CSR


```sh

#create a certificate signing request
openssl req -new -key fd.key -out fd.csr
### .. and fill fields manually

# from a config file
cat <<EOF > host.cnf
[ req ]
prompt              = no
distinguished_name  = req_distinguished_name
policy              = policy_match
x509_extensions     = user_crt
req_extensions      = v3_req

[ req_distinguished_name ]
countryName                     = IN
stateOrProvinceName             = KARNATAKA
localityName                    = BENGALURU
0.organizationName              = gxc
organizationalUnitName          = gxc
commonName                      = host.gxc.io

[ user_crt ]
basicConstraints        = critical, CA:FALSE
subjectKeyIdentifier    = hash
authorityKeyIdentifier  = keyid:always, issuer:always
keyUsage                = critical, nonRepudiation, digitalSignature, keyEncipherment, keyAgreement
extendedKeyUsage        = critical, clientAuth
subjectAltName          = @alt_names

[ v3_req ]
basicConstraints        = critical, CA:FALSE
subjectKeyIdentifier    = hash
keyUsage                = critical, nonRepudiation, digitalSignature, keyEncipherment, keyAgreement
extendedKeyUsage        = critical, clientAuth
subjectAltName          = @alt_names

[alt_names]
DNS.1   = host.gxc.io
DNS.2   = host1.gxc.io
EOF
openssl req -key host.key -config host.cnf -new -out host.csr

#dump the csr
openssl req -in fd.csr -text -noout
#you can also very
openssl req -in fd.csr -text -noout -verify

#already have a cert.. just get a csr to extend it.
openssl x509 -x509toreq -in fd.crt -out fd.csr -signkey fd.key

```

## Issue Cert


```sh

# issue a self-signed cert in one-shot from a config file (No csr)
cat <<EOF > root.cnf
[ req ]
prompt             = no
distinguished_name = req_distinguished_name
x509_extensions     = v3_ca

[ req_distinguished_name ]
countryName                     = IN
stateOrProvinceName             = KARNATAKA
localityName                    = BENGALURU
0.organizationName              = GXC
organizationalUnitName          = GXC
commonName                      = root.gxc.io

[ v3_ca ]
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical,CA:true
keyUsage  = critical, cRLSign, digitalSignature, keyCertSign
nsComment = "OpenSSL Generated Certificate"
EOF
openssl req -new -x509 -days 365 -config root.cnf  -key site-root.key -out site-root.crt

#non-interfactive - just with subj and no extensions
openssl req -new -x509 -days 365 -key fd.key -out fd.crt \
        -subj "/C=GB/L=London/O=Feisty Duck Ltd/CN=www.feistyduck.com"

#signing our own certificate from a csr if present
openssl x509 -req -days 365 -in fd.csr -signkey fd.key -out fd.crt


#read a certificate
openssl x509 -in fd.crt -noout -text

```

## Conversions


```sh


#convert PEM to DER
openssl x509 -inform PEM -in fd.pem -outform DER -out fd.der
#convert DER to PEM
openssl x509 -inform DER -in fd.der -outform PEM -out fd.pem

#convert key, pem-cert, cert-chain into pfx
openssl pkcs12 -export -name "My Certificate" -out fd.p12 \
      -inkey fd.key -in fd.crt -certfile fd-chain.crt

#pfx to pem
## you will have to manually split the parts
### Note add -legacy if required.
openssl pkcs12 -in fd.p12 -out fd.pem -nodes

```


openssl genrsa -aes128 -out fd.key 2048
