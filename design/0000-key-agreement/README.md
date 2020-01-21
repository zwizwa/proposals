```yaml
Status: Draft
```

# Motivation

# Description


Breaking down exactly as per the noise paper:
```
XX:
  -> e
  <- e, ee, s, es
  -> s, se
```

Both parties maintain the following state variables:

* `s, e`: The local party’s static and ephemeral key pairs (which may be empty).

* `rs, re`: The remote party’s static and ephemeral public keys (which may be empty).

* `h`: A handshake hash value that hashes all the handshake data that’s been sent and received

* `ck`: A chaining key that hashes all previous DH outputs. Once the handshake completes,
the chaining key will be used to derive the encryption keys for transport messages

* `k, n`: An encryption key k (which may be empty) and a counter-based nonce n.
Whenever a new DH output causes a new ck to be calculated, a new k is also calculated.
The key k and nonce n are used to encrypt static public keys and handshake payloads

Expands to:

```
Initiator                                 |   Responder
------------------------------------------|----------------------------------------


M1 (Send)
-> e

1. Pick a static 25519 keypair for
this handshake and set it to s

2. Generate an ephemeral 25519
keypair for this handshake and set
it to e

3. Set k to empty, Set n to 0

4. Set h and ck to
'Noise_XX_25519_AESGCM_SHA256'

5. h = SHA256(h || prologue),
prologue is empty

6. h = SHA256(h || e.PublicKey),
Write e.PublicKey to outgoing message
buffer, BigEndian

7. h = SHA256(h || payload),
payload is empty


------------------------------------------|----------------------------------------


                                                M1 (Receive)
                                                -> e

                                                1. Pick a static 25519 keypair for
                                                this handshake and set it to s

                                                2. Generate an ephemeral 25519
                                                keypair for this handshake and set
                                                it to e

                                                3. Set k to empty, Set n to 0

                                                4. Set h and ck to
                                                'Noise_XX_25519_AESGCM_SHA256'

                                                5. h = SHA256(h || prologue),
                                                prologue is empty

                                                6. Read 32 bytes from the incoming
                                                message buffer, parse it as a public
                                                key, set it to re
                                                h = SHA256(h || re)

                                                7. read remaining message as payload
                                                h = SHA256(h || payload),
                                                payload should be empty


------------------------------------------|----------------------------------------


                                                M2 (Send)
                                                <- e, ee, s, es

                                                1. h = SHA256(h || e.PublicKey),
                                                Write e.PublicKey to outgoing message
                                                buffer, BigEndian

                                                2. ck, k = HKDF(ck, DH(e, re), 2)
                                                n = 0

                                                3. c = ENCRYPT(k, n++, h, s.PublicKey)
                                                h =  SHA256(h || c),
                                                Write c to outgoing message
                                                buffer, BigEndian

                                                4. ck, k = HKDF(ck, DH(s, re), 2)
                                                n = 0

                                                5. c = ENCRYPT(k, n++, h, payload)
                                                h = SHA256(h || c),
                                                payload is empty

------------------------------------------|----------------------------------------


M2 (Receive)
<- e, ee, s, es

1. Read 32 bytes from the incoming
message buffer, parse it as a public
key, set it to re
h = SHA256(h || re)

2. ck, k = HKDF(ck, DH(e, re), 2)
n = 0

3. Read 48 bytes the incoming
message buffer as c
p = DECRYPT(k, n++, h, c)
h = SHA256(h || c),
parse p as a public key,
set it to rs

4. ck, k = HKDF(ck, DH(e, rs), 2)
n = 0

5. Read remaining bytes of incoming
message buffer as c
p = DECRYPT(k, n++, h, c)
h = SHA256(h || c),
parse p as a payload,
payload should be empty


------------------------------------------|----------------------------------------


M3 (Send)
-> s, se

1. c = ENCRYPT(k, n++, h, s.PublicKey)
h =  SHA256(h || c),
Write c to outgoing message
buffer, BigEndian

2. ck, k = HKDF(ck, DH(s, re), 2)
n = 0

3. c = ENCRYPT(k, n++, h, payload)
h = SHA256(h || c),
payload is empty


------------------------------------------|----------------------------------------


                                                M3 (Receive)
                                                -> s, se

                                                1. Read 48 bytes the incoming
                                                message buffer as c
                                                p = DECRYPT(k, n++, h, c)
                                                h = SHA256(h || c),
                                                parse p as a public key,
                                                set it to rs

                                                2.ck, k = HKDF(ck, DH(e, rs), 2)
                                                n = 0

                                                3. Read remaining bytes of incoming
                                                message buffer as c
                                                p = DECRYPT(k, n++, h, c)
                                                h = SHA256(h || c),
                                                parse p as a payload,
                                                payload should be empty


------------------------------------------|----------------------------------------


                                                1. k1, k2 = HKDF(ck, zerolen, 2)
                                                n1 = 0, n2 = 0
                                                Use (k1, n1) to encrypt outgoing
                                                Use (k2, n2) to decrypt incoming

1. k1, k2 = HKDF(ck, zerolen, 2)
n1 = 0, n2 = 0
Use (k1, n1) to decrypt incoming
Use (k2, n2) to encrypt outgoing

```

# Security Consideration

# References


[<span id="ref1">1</span>]: Perrin, Trevor. "The noise protocol framework", (revision 34)\
https://github.com/noiseprotocol/noise_spec/tree/ecdf084ece2bf92b16b1201b6ae5c99d23fb4151.
.