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

# Decoding in wireshark

* Enable charon log:
    ```sh
    sudo ipsec stroke loglevel ike 4
    ```
* Let IKE-INIT happen and collect keys
    ```log
    2023-04-06T16:15:39.277175+00:00 lakshman-dagw charon: 11[IKE] Sk_ai secret => 32 bytes @ 0x7f53fc013b10
    2023-04-06T16:15:39.277189+00:00 lakshman-dagw charon: 11[IKE]    0: 36 94 20 17 41 C0 35 D9 69 6A D9 41 FA F8 CB 2D  6. .A.5.ij.A...-
    2023-04-06T16:15:39.277201+00:00 lakshman-dagw charon: 11[IKE]   16: A4 33 56 1D 7F 03 66 00 63 AE 62 9F 18 EC 95 09  .3V...f.c.b.....
    2023-04-06T16:15:39.277262+00:00 lakshman-dagw charon: 11[IKE] Sk_ar secret => 32 bytes @ 0x7f53fc007180
    2023-04-06T16:15:39.277273+00:00 lakshman-dagw charon: 11[IKE]    0: A7 BC 6F FE 8E 79 D3 F4 A6 F1 A1 D3 C3 2F F7 F4  ..o..y......./..
    2023-04-06T16:15:39.277288+00:00 lakshman-dagw charon: 11[IKE]   16: B4 EA 5C D6 AC 67 2D 29 59 C6 7E 48 12 C2 CD 5A  ..\\..g-)Y.~H...Z
    2023-04-06T16:15:39.277685+00:00 lakshman-dagw charon: 11[IKE] Sk_ei secret => 16 bytes @ 0x7f53fc0132f0
    2023-04-06T16:15:39.277698+00:00 lakshman-dagw charon: 11[IKE]    0: 60 64 42 D3 AF DB 3D 3B E1 CF 0D 82 BF 9B E5 91  `dB...=;........
    2023-04-06T16:15:39.277742+00:00 lakshman-dagw charon: 11[IKE] Sk_er secret => 16 bytes @ 0x7f53fc012de0
    2023-04-06T16:15:39.277754+00:00 lakshman-dagw charon: 11[IKE]    0: 8B 2E AB FE EC C5 E3 C3 55 D9 68 C0 86 94 6E B4  ........U.h...n.
    ```
* Convenient awk
    ```sh
    file=/var/log/syslog
    awk 'function getkey(line){match(line,"[[:digit:]]: ([0-9A-Fa-f ]+)  .",result);gsub(" ","",result[1]);return result[1]} /Sk_ai secret/ || /Sk_ar secret/ { print $0 ; getline key1 ; k=getkey(key1) ; getline key2 ; k2=getkey(key2) ; print k k2 } /Sk_ei secret / || /Sk_er secret/ { print $0 ; getline key ; k=getkey(key) ; print k } ' $file
    2023-04-06T16:15:39.277175+00:00 lakshman-dagw charon: 11[IKE] Sk_ai secret => 32 bytes @ 0x7f53fc013b10
    3694201741C035D9696AD941FAF8CB2DA433561D7F03660063AE629F18EC9509
    2023-04-06T16:15:39.277262+00:00 lakshman-dagw charon: 11[IKE] Sk_ar secret => 32 bytes @ 0x7f53fc007180
    A7BC6FFE8E79D3F4A6F1A1D3C32FF7F4B4EA5CD6AC672D2959C67E4812C2CD5A
    2023-04-06T16:15:39.277685+00:00 lakshman-dagw charon: 11[IKE] Sk_ei secret => 16 bytes @ 0x7f53fc0132f0
    606442D3AFDB3D3BE1CF0D82BF9BE591
    2023-04-06T16:15:39.277742+00:00 lakshman-dagw charon: 11[IKE] Sk_er secret => 16 bytes @ 0x7f53fc012de0
    8B2EABFEECC5E3C355D968C086946EB4
    ```
* Get the following from `swanctl --list-sas`
    ```
    onyxedge@lakshman-dagw:~$ ipsec_sas
    enodeb_1202000038269KP0037: #1, ESTABLISHED, IKEv2, 4de28a796cecfab2_i 3c87f93c76a316cb_r*
                                                        ^^^^^^^^^^^^^^^^   ^^^^^^^^^^^^^^^^           <---
      local  'CN=79c8533e-2b99-4ebe-a92e-39c46611885c' @ 192.168.123.68[4500]
      remote 'C=US, O=Gxc, OU=GxcEnodeb, CN=1202000038269KP0037' @ 192.168.123.55[4500] [10.0.30.1]
      AES_CBC-128/HMAC_SHA2_256_128/PRF_AES128_XCBC/ECP_256
      ^^^^^^^^^^^ ^^^^^^^^^^^^^^^^^                                                                   <---
      established 843s ago, rekeying in 12797s
      enodeb_1202000038269KP0037: #1, reqid 1, INSTALLED, TUNNEL, ESP:AES_CBC-128/HMAC_SHA2_256_128
        installed 843s ago, rekeying in 2464s, expires in 3117s
        in  c400e8c7,      0 bytes,     0 packets
        out cb0b8432,      0 bytes,     0 packets
        local  10.0.10.1/32
        remote 10.0.30.1/32
    onyxedge@lakshman-dagw:~$
    ```
* Get the following from `ip xfrm state`. For both dirs
    ```
    onyxedge@lakshman-dagw:~$ sudo ip xfrm state
    src 192.168.123.68 dst 192.168.123.55
        ^^^^^^^^^^^^^^     ^^^^^^^^^^^^^^                                                                 <---
            proto esp spi 0xcb0b8432 reqid 1 mode tunnel
                          ^^^^^^^^^^                                                                      <---
            replay-window 0 flag af-unspec
            auth-trunc hmac(sha256) 0x39421a4abc17949d3cd739a84f13e760d4f25d4da11fad831de5e3356f620153 128
                       ^^^^^^^^^^^  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^    <---
            enc cbc(aes) 0xb0f513708e8f98c4cfff27183ccea221
                ^^^^^^^^ ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^                                               <---
            anti-replay context: seq 0x0, oseq 0x0, bitmap 0x00000000
    src 192.168.123.55 dst 192.168.123.68
            proto esp spi 0xc400e8c7 reqid 1 mode tunnel
            replay-window 32 flag af-unspec
            auth-trunc hmac(sha256) 0xbf4c0657a58c31228a68da2ddfc92572586fdb4124d46bfd8b11cb36583a771b 128
            enc cbc(aes) 0x3763b36c3d8d86f17328cd0c56445794
            anti-replay context: seq 0x0, oseq 0x0, bitmap 0x00000000
    onyxedge@lakshman-dagw:~$
    ```
* Open wireshar. Click one IKE packet
* Goto `Edit->Preferences->Protocol->ISAKMP->Ikev2 decryption table->Edit`. You should see this box.
    * Click '+' and then enter the following
        Initiator SPI, from ipsec_sas output
        Responder SPI, from ipsec_sas output
        SK_ei, from charon log
        SK_er, from charon log
        Encryption algorithm, from ipsec_sas output. This is mostly AES-CBC-128 for AGW-AP ipsec.
        SK_ai, from charon log
        SK_ar, from charon log
        Integrity algorithm, from ipsec_sas output. This is mostly HMAC-SHA2-256-128 for AGW-AP ipsec
* Click one ESP packet. Goto `Edit->Preferences->Protocol->ESP`.
* Check-box all 4 items.
* Click Edit against ESP SAs. Add 2 entries from that of xfrm state output
    Protocol: IPv4
    SrcIP, Dst IP
    SPI
    Encryption algo. This is typically AES-CBC for AGW-AP ipsec.
    Encryption key.
    Authentication algo. This is typically HMAC-SHA2-256-128 for AGW-AP ipsec
    Authentication key.
* Editing via file
    ```
    C:\Users\<username>\AppData\Roaming\Wireshark\ikev2_decryption_table
    C:\Users\<username>\AppData\Roaming\Wireshark\esp_sa
    ```

#Sources

* http://www.unixwiz.net/techtips/iguide-ipsec.html
* https://superuser.com/a/790598/544330
* http://www.brocade.com/content/html/en/vrouter5600/40r1/vrouter-40r1-ipsecvpn/GUID-8BC898A5-9906-485E-BE44-7C8D38E8EB05.html

# Book

* IPSEC - New security Standard, Intranets, VPN by Naganand Doraswamy, Dan Harkins
