# P2P Chat App — Security Architecture Design

## Security Objectives

### Core Security Requirements

* **Confidentiality:** Only the sender and intended recipient can read message content.
* **Integrity:** Ensure messages aren’t altered in transit.
* **Authentication:** Verify the identities of both parties.
* **Non-repudiation:** Prevent senders from denying messages they sent.
* **Forward secrecy:** Past messages remain secure even if long-term keys are later compromised.

### Threat Model

* **Passive attacks:** Eavesdropping, traffic analysis.
* **Active attacks:** Tampering, replay, man-in-the-middle.
* **Node compromise:** Malicious peers joining the network.
* **Key leakage:** Exposure of long-term or session keys.

## Cryptographic Scheme

### Hybrid Encryption

Use a **RSA + AES** hybrid:

* **RSA-2048:** key exchange and digital signatures.
* **AES-256-GCM:** symmetric encryption for messages and files.
* **ECDH:** optional enhancement for key agreement.

### Key Hierarchy

```
Identity Key Pair (RSA-2048)
├── Long-term public/private keys
├── Used for identity/auth and key exchange
└── Generated or loaded at node startup

Session Key (AES-256)
├── One per conversation
├── Transmitted encrypted with RSA
└── Rotated periodically (optional)

Message Key (AES-256)
├── Derived from the session key
├── Unique IV per message
└── Supports forward secrecy
```

## Security Protocols

### Node Authentication Protocol

1. **Node registration:** generate RSA key pair; public key is the node’s identity.
2. **Identity proof:** sign a challenge with the private key.
3. **Trust establishment:** verify and pin public-key fingerprints.

### Key Exchange Protocol

```
Alice                           Bob
  |                              |
  |--- Hello + PublicKey_A ----->|
  |                              |
  |<-- Hello + PublicKey_B ------|
  |                              |
  |--- Encrypted(SessionKey) --->|
  |                              |
  |<-- ACK + Encrypted(Confirm)--|
  |                              |
```

### Message Encryption Protocol

1. **Prepare message:** plaintext + timestamp + sequence number.
2. **Encrypt:** AES-256-GCM, producing an auth tag.
3. **Transmit:** ciphertext + IV + auth tag.
4. **Decrypt/verify:** validate the tag; only then decrypt.

## Implementation Architecture

### Core Security Components

```
SecurityManager
├── KeyManager
├── CryptoService
├── AuthenticationService
└── SecureChannel

KeyManager
├── Generate/store key pairs
├── Coordinate key exchanges
└── Manage session keys

CryptoService
├── RSA encrypt/decrypt
├── AES encrypt/decrypt
├── Sign/verify
└── Hashing

AuthenticationService
├── Node identity verification
├── Message integrity checks
└── Replay protection

SecureChannel
├── Secure message transport
├── Key negotiation
└── Session lifecycle
```

### Secure Storage

* **Key storage:** protect private keys with Java KeyStore.
* **Encrypted config:** store sensitive settings encrypted.
* **Secure erasure:** zeroize keys in memory after use.

## Performance Considerations

### Optimizations

* **Session key caching:** reduce repeated RSA operations.
* **Batch encryption:** group multiple messages per encrypt call.
* **Async processing:** run crypto on background threads.
* **HW acceleration:** leverage AES-NI when available.

### Targets

* **RSA ops:** key exchange < 100 ms.
* **AES:** per-message encryption < 1 ms.
* **Overall overhead:** added latency from security < 10 ms.

## Security Configuration

### Tunables

* **Cipher strength:** RSA key size, AES key size.
* **Key rotation:** session key update frequency.
* **Auth mode:** strict vs. permissive.
* **Logging level:** verbosity of security events.

### Default Policies

* **Encryption required:** all traffic must be encrypted.
* **Identity verification:** new nodes must authenticate.
* **Key rotation:** rotate session keys hourly.
* **Audit logging:** record all security-relevant events.

## Compliance

### Standards

* **FIPS 140-2:** cryptographic module security.
* **RFC 3447:** RSA (PKCS #1).
* **RFC 3394:** AES key wrap.
* **RFC 5652:** Cryptographic Message Syntax (CMS).

### Export Controls

* Use standard Java crypto providers (meets export requirements).
* Provide configurable cipher strengths.
* Offer a reduced-strength mode for restricted environments.

This security architecture delivers comprehensive protection for the P2P chat application, safeguarding the confidentiality, integrity, and availability of communications.
