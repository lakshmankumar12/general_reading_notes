
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
-binary     :   provide op in binary form (used in dgst)
-text       :   provide op in text form
-pubout     :   give public key as output
-noout      :   omits the output of the encoded version of whatever
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
openssl dsaparam -genkey 2048 | openssl dsa -out dsa.key

#ecdsa -- you give the name of the curve
openssl ecparam -genkey -name secp256r1 | openssl ec -out ec.key

# add a paraphrase to your key.. Use this option
-aes128

```

## CSR


```sh

#create a certificate signing request
openssl req -new -key fd.key -out fd.csr
### .. and fill fields manually

## extendedKeyUsage -- clientAuth / serverAuth

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

#a CA signing a host
openssl x509 -req -days 365 -in host.csr \
     -CA site-root.crt -CAkey site-root.key -CAcreateserial \
     -out host.crt -extensions user_crt -extfile host.cnf


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

## generate random bytes of a given length
openssl rand 32
## generate random bytes of a length and then convert to base64
openssl rand -base64 32

```

## encrypt and decrypt

```sh
# mere conversion from b64
openssl base64 -in file.orig -out file.b64
openssl base64 -d -in file.b64 -out file.orig

#des3 or blowfish
openssl des3
openssl bf

#simple shared key
#ask key from stdin
openssl enc -in foo.bar -aes-256-cbc -pass stdin > foo.bar.enc
### TO check if this works:
### openssl enc -in foo.bar -aes-256-cbc -pass file:file_with_pass -out foo.bar.enc -outform DER
# more fancy options
-salt   : add a salt. Actually default.
-pbkdf2 : Use PBKDF2 algorithm with default iteration count unless otherwise specified.
    -iter <count>

#decode back
openssl enc -d -in foo.bar.enc -aes-256-cbc -pass stdin > foo.bar

#with a pub-priv pair... Create a self-signed cert and then:
openssl smime -encrypt -aes256 -in clear_text.txt -binary -outform DER -out encrypted.file certfile.pem
#and decrypt with priv-key
openssl smime -decrypt -in encrypted.file -inform DER -inkey privkey.pem -out clear_back.txt


```

## digests

```sh
# just get sha256sum of a message
openssl dgst -sha256 input_file
# gives the actual 32 byte in binary
openssl dgst -sha256 -binary input_file

# sign some file with a private key. output is binary(~512 bytes)
# you can choose to convert that to base64 if you want later.
openssl dgst -sha256 -sign privkey.pem -out sign.sha256 file_to_be_signed

# verify the signature
openssl dgst -sha256 -verify pubkey.pem -signature sign.sha256 file_to_be_signed


```




## extract

Tip: Use sha256sum to compare the big outputs quickly

* Getting pub key from cert and priv-key is a way to match
  cert to its priv-key.
```sh

#extract pubkey from cert
openssl x509 -in Org1-cert.pem -noout -pubkey

#extract pubkey from any type of key
openssl pkey -pubout -in Org1-key.pem

```

* Extracting modulus for rsa keys is another way. You can compare this.
```
openssl x509 -modulus -noout -in cert.pem

openssl req -noout -modulus -in example.csr
```



## openssl testing

```sh

openssl verify -CAfile site-root.crt server.crt

openssl s_client -connect localhost:1443 -CAfile site-root.crt -servername server.gxc.io

curl --cert client.crt --key client.key --cacert site-root.crt --resolve server.gxc.io:1443:127.0.0.1 https://server.gxc.io:1443

```

# study links

https://www.freecodecamp.org/news/openssl-command-cheatsheet-b441be1e8c4a/
https://www.electricmonk.nl/log/2018/06/02/ssl-tls-client-certificate-verification-with-python-v3-4-sslcontext/
https://stackoverflow.com/questions/22429648/ssl-in-python3-with-httpserver
