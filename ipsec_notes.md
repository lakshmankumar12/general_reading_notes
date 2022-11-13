# IPSec Architecture

* AH is for authentication
   -> Sender is the one who sent it, pkt is received-as-is.
* ESP is for encryption + optionally authentication
* Both AH and ESP have headers that follow the IP header.
  That is, they are the next-protocol in IP.
* IKE is the way to negotiate a key to use dynamically.


## AH versus ESP

> "Authentication Header" (AH) and "Encapsulating Security Payload" (ESP) are the
> two main wire-level protocols used by IPsec, and they authenticate (AH) and
> encrypt+authenticate (ESP) the data flowing over that connection. They are
> typically used independently, though it's possible (but uncommon) to use them
> both together.

## Tunnel mode versus Transport mode

Transport Mode provides a secure connection between two endpoints as it
encapsulates IP's payload, while Tunnel Mode encapsulates the entire IP packet
to provide a virtual "secure hop" between two gateways. The latter is used to
form a traditional VPN, where the tunnel generally creates a secure tunnel
across an untrusted Internet.


## Crypto choices

* MD5 versus SHA-1 versus DES versus 3DES versus AES versus blah blah blah

Setting up an IPsec connection involves all kinds of crypto choices, but this
is simplified substantially by the fact that any given connection can use at
most two or (rarely) three at a time.  Authentication calculates an Integrity
Check Value (ICV) over the packet's contents, and it's usually built on top of
a cryptographic hash such as MD5 or SHA-1. It incorporates a secret key known
to both ends, and this allows the recipient to compute the ICV in the same way.
If the recipient gets the same value, the sender has effectively authenticated
itself (relying on the property that cryptographic hashes can't practically be
reversed). AH always provides authentication, and ESP does so optionally.
Encryption uses a secret key to encrypt the data before transmission, and this
hides the actual contents of the packet from eavesdroppers. There are quite a
few choices for algorithms here, with DES, 3DES, Blowfish and AES being common.
Others are possible too.

## IKE versus manual keys

Since both sides of the conversation need to know the secret values used in
hashing or encryption, there is the question of just how this data is
exchanged. Manual keys require manual entry of the secret values on both ends,
presumably conveyed by some out-of-band mechanism, and IKE (Internet Key
Exchange) is a sophisticated mechanism for doing this online.

## Main mode versus aggressive mode

These modes control an efficiency-versus-security tradeoff during initial IKE
key exchange. "Main mode" requires six packets back and forth, but affords
complete security during the establishment of an IPsec connection, while
Aggressive mode uses half the exchanges providing a bit less security because
some information is transmitted in cleartext.



# Implementation of IPSEC

* SAD - Security association database: Contains info on existing assocations
  (outer-source-ip, outer-dest-ip, SPI, keys to use for encryption)
* SPD - Security policy database: Contains the traffic patterns to match
  and which SPI to use.


# Strongswan

* https://docs.strongswan.org/docs/5.9/howtos/introduction.html
* https://wiki.strongswan.org/projects/strongswan/wiki/Fromipsecconf

# swanctl

```sh
swanctl --load-creds
swanctl --load-pools
swanctl --load-conns

swanctl --initiate --child connname
swanctl --list-pools --leases
swanctl --terminate --ike connname
swanctl --list-sas

swanctl --log
# or grep charon in /var/log/messages or syslog


```



#Sources

* http://www.unixwiz.net/techtips/iguide-ipsec.html
* https://superuser.com/a/790598/544330
* http://www.brocade.com/content/html/en/vrouter5600/40r1/vrouter-40r1-ipsecvpn/GUID-8BC898A5-9906-485E-BE44-7C8D38E8EB05.html

# Book

* IPSEC - New security Standard, Intranets, VPN by Naganand Doraswamy, Dan Harkins
