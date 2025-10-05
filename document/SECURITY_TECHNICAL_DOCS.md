#  P2P Chat App — Security Mechanisms Technical Document

##  Security Architecture

### Overall Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    P2P Chat Application                     │
├─────────────────────────────────────────────────────────────┤
│                       Security Layer                        │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────────┐ │
│  │ SecurityMgr │ │ KeyManager  │ │ AuthenticationService   │ │
│  └─────────────┘ └─────────────┘ └─────────────────────────┘ │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────────┐ │
│  │CryptoService│ │SecureMsgHdlr│ │ SecureFileTransferSvc   │ │
│  └─────────────┘ └─────────────┘ └─────────────────────────┘ │
├─────────────────────────────────────────────────────────────┤
│                     Application Layer                       │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────────┐ │
│  │    Node     │ │MessageRouter│ │    FileTransferSvc      │ │
│  └─────────────┘ └─────────────┘ └─────────────────────────┘ │
├─────────────────────────────────────────────────────────────┤
│                      Network Layer                          │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────────┐ │
│  │PeerConnection│ │   Socket    │ │        TCP/UDP          │ │
│  └─────────────┘ └─────────────┘ └─────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### Core Components

#### 1. SecurityManager

* **Role:** Orchestrates all security functions
* **Functions:** Policy control, component coordination, status monitoring
* **API:** High-level security interfaces

#### 2. KeyManager

* **Role:** Key generation, storage, and lifecycle
* **Functions:** RSA keypair management, AES session key management, persistence
* **Algorithms:** RSA-2048, AES-256

#### 3. CryptoService

* **Role:** Low-level cryptographic ops
* **Functions:** Symmetric/asymmetric crypto, signatures, hashing
* **Algorithms:** AES/GCM/NoPadding, RSA/ECB/OAEP-SHA256, SHA-256

#### 4. AuthenticationService

* **Role:** Node identity and trust management
* **Functions:** Challenge–response auth, trust scoring, certificate handling
* **Mechanism:** Public-key–based identity verification

#### 5. SecureMessageHandler

* **Role:** Secure message processing
* **Functions:** E2E encryption, signature verification, integrity protection
* **Protocol:** Custom secure message format

#### 6. SecureFileTransferService

* **Role:** Secure file transfer
* **Functions:** Encrypted transfer, integrity checks, progress monitoring
* **Features:** Chunk encryption, streaming I/O

##  Cryptographic Algorithms

### Symmetric Crypto (AES-256-GCM)

```java
// Encryption
Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
cipher.init(Cipher.ENCRYPT_MODE, secretKey);
byte[] iv = cipher.getIV();
byte[] encryptedData = cipher.doFinal(plaintext.getBytes(StandardCharsets.UTF_8));
```

**Notes:**

* **Cipher:** AES-256
* **Mode:** GCM (authenticated encryption)
* **Benefits:** Integrity, anti-tampering, high performance
* **IV length:** 12 bytes (random)
* **Tag length:** 16 bytes (auth tag)

### Asymmetric Crypto (RSA-2048)

```java
// Key generation
KeyPairGenerator keyGen = KeyPairGenerator.getInstance("RSA");
keyGen.initialize(2048);
KeyPair keyPair = keyGen.generateKeyPair();
```

**Notes:**

* **Key size:** 2048 bits
* **Padding:** OAEP with SHA-256
* **Use:** Key transport, digital signatures
* **Security level:** ~112-bit symmetric equivalent

### Digital Signatures (RSA-SHA256)

```java
// Sign
Signature signature = Signature.getInstance("SHA256withRSA");
signature.initSign(privateKey);
signature.update(data);
byte[] signatureBytes = signature.sign();
```

**Notes:**

* **Hash:** SHA-256
* **Purpose:** Integrity + authenticity
* **Collision resistance:** ~2^128 work factor

##  Key Management

### Key Hierarchy

```
Root Key (Node Identity)
├── Node Private Key (RSA-2048)
├── Node Public Key (RSA-2048)
└── Session Keys (AES-256)
    ├── Session Key — Peer A
    ├── Session Key — Peer B
    └── Session Key — Peer C
```

### Key Lifecycle

1. **Generation**

   * RSA pair at node startup
   * AES session keys on connection
   * SecureRandom for all key material

2. **Distribution**

   * Public keys via handshake
   * Session keys transported with RSA
   * Supports key updates/rotation

3. **Use**

   * Session keys for message encryption
   * Private key for signatures
   * Public key for verification

4. **Destruction**

   * Clear session keys on disconnect
   * Zeroize sensitive memory
   * Periodic rotation supported

### Key Storage

```java
// On-disk layout
keys/
├── node_private.key    // PKCS#8
├── node_public.key     // X.509
└── session_keys.dat    // Encrypted cache
```

**Protections:**

* Private key file perms (600)
* In-memory encryption for session keys
* Regular backups and recovery procedures

##  Authentication Protocol

### Challenge–Response

```
Node A                           Node B
  |                               |
  |  1. Authentication Request    |
  |------------------------------>|
  |                               |
  |  2. Challenge (Random Data)   |
  |<------------------------------|
  |                               |
  |  3. Signed Challenge          |
  |------------------------------>|
  |                               |
  |  4. Verification Result       |
  |<------------------------------|
```

#### Flow Details

1. **Request**

```java
AuthenticationChallenge challenge = authService.createChallenge(targetNodeId);
```

2. **Challenge**

```java
byte[] challengeData = secureRandom.nextBytes(32);
String challengeId = UUID.randomUUID().toString();
```

3. **Response**

```java
byte[] signature = cryptoService.sign(challengeData, nodePrivateKey);
```

4. **Verification**

```java
boolean valid = cryptoService.verifySignature(challengeData, signature, nodePublicKey);
```

### Trust Level Evaluation

```java
public class TrustLevel {
    private static final int BASE_TRUST = 30;
    private static final int AUTH_SUCCESS_BONUS = 20;
    private static final int COMMUNICATION_BONUS = 10;
    private static final double TIME_DECAY = 0.95;
}
```

**Levels:**

* **0–30:** Untrusted
* **31–50:** Low trust
* **51–70:** Medium trust
* **71–90:** High trust
* **91–100:** Fully trusted

##  Secure Message Format

### Structure

```json
{
  "messageId": "uuid-string",
  "senderId": "node-id",
  "timestamp": 1234567890,
  "encryptedContent": "base64-encrypted-data",
  "signature": "base64-signature",
  "metadata": {
    "algorithm": "AES-256-GCM",
    "keyId": "session-key-id",
    "iv": "base64-iv"
  }
}
```

### Encryption Flow

1. **Prepare**

```java
String fullContent = timestamp + ":" + messageId + ":" + content;
```

2. **Encrypt**

```java
EncryptionResult result = cryptoService.encryptWithAES(fullContent, sessionKey);
```

3. **Sign**

```java
byte[] signature = cryptoService.sign(fullContent.getBytes(), privateKey);
```

4. **Wrap**

```java
SecureMessage secureMessage = new SecureMessage(senderId, encryptedContent, signature, timestamp, messageId);
```

### Decryption Flow

1. **Verify**

```java
boolean valid = cryptoService.verifySignature(content, signature, senderPublicKey);
```

2. **Decrypt**

```java
String decryptedContent = cryptoService.decryptWithAES(encryptionResult, sessionKey);
```

3. **Integrity Check**

```java
String[] parts = decryptedContent.split(":", 3);
long messageTimestamp = Long.parseLong(parts[0]);
String messageId = parts[1];
String actualContent = parts[2];
```

##  Secure File Transfer Protocol

### Transfer Flow

```
Sender                           Receiver
  |                               |
  |  1. File Transfer Request     |
  |------------------------------>|
  |                               |
  |  2. Accept/Reject Response    |
  |<------------------------------|
  |                               |
  |  3. Encrypted File Header     |
  |------------------------------>|
  |                               |
  |  4. Encrypted File Chunks     |
  |------------------------------>|
  |  ...                          |
  |------------------------------>|
  |                               |
  |  5. Transfer Complete         |
  |------------------------------>|
  |                               |
  |  6. Verification Result       |
  |<------------------------------|
```

### Encrypted Format

```
File Header (Encrypted):
├── Original Filename (UTF-8)
├── File Size (8 bytes)
├── Checksum (SHA-256, 32 bytes)
└── Metadata (JSON)

File Data (Encrypted):
├── Chunk 1 (8 KB, AES-256-GCM)
├── Chunk 2 (8 KB, AES-256-GCM)
├── ...
└── Chunk N (≤ 8 KB, AES-256-GCM)
```

### Integrity Verification

```java
// Sender checksum
MessageDigest digest = MessageDigest.getInstance("SHA-256");
byte[] fileData = Files.readAllBytes(filePath);
byte[] checksum = digest.digest(fileData);

// Receiver verification
byte[] receivedChecksum = digest.digest(receivedData);
boolean valid = Arrays.equals(checksum, receivedChecksum);
```

##  Performance Optimization

### Crypto Performance

**AES-256-GCM:**

* **Throughput:** ~500 MB/s (modern CPUs)
* **Latency:** < 1 ms (small msgs)
* **Memory:** minimal buffers

**RSA-2048:**

* **Sign:** ~1,000 ops/s
* **Verify:** ~30,000 ops/s
* **Keygen:** ~100 ms

### Strategies

1. **Key Caching**

```java
private final Map<String, SecretKey> sessionKeyCache = new ConcurrentHashMap<>();
```

2. **Batch Ops**

```java
public List<SecureMessage> encryptBatch(List<String> messages, String targetNodeId);
```

3. **Async Crypto**

```java
CompletableFuture<EncryptionResult> encryptFileAsync(Path filePath);
```

4. **Memory Hygiene**

```java
Arrays.fill(sensitiveData, (byte) 0);
```

##  Threat Model

### Analysis

1. **Passive**

   * **Eavesdropping**
   * **Mitigation:** E2E encryption (AES-256)

2. **Active**

   * **Tampering**
   * **Mitigation:** Digital signatures (RSA-2048)

3. **Impersonation**

   * **Mitigation:** Public-key identity auth (challenge–response)

4. **Replay**

   * **Mitigation:** Timestamps + uniqueness checks

### Assumptions

1. Private keys remain secret
2. Algorithms are secure as standardized
3. Implementations are bug-free
4. Runtime environment is trustworthy

### Risk Assessment

| Threat        | Likelihood | Impact   | Risk   | Mitigation            |
| ------------- | ---------- | -------- | ------ | --------------------- |
| Eavesdropping | High       | High     | High   | End-to-end encryption |
| Tampering     | Medium     | High     | Medium | Digital signatures    |
| Impersonation | Low        | High     | Medium | Identity auth         |
| Key leakage   | Low        | Critical | Medium | Key management        |
| Replay        | Medium     | Medium   | Low    | Timestamp checks      |

##  Security Auditing

### Audit Log API

```java
public class SecurityAuditLog {
    public void logKeyGeneration(String nodeId);
    public void logKeyExchange(String nodeId, boolean success);
    public void logAuthentication(String nodeId, AuthResult result);
    public void logEncryption(String messageId, String algorithm);
    public void logDecryption(String messageId, boolean success);
    public void logSecurityViolation(String nodeId, String violation);
}
```

### Monitoring Metrics

1. **Encryption coverage** (encrypted vs total msgs)
2. **Auth success rate**
3. **Key exchange frequency**
4. **Security incident counts**

### Compliance Checks

```java
public class SecurityCompliance {
    public boolean checkKeyStrength();
    public boolean checkEncryptionCoverage();
    public boolean checkAuthenticationPolicy();
    public boolean checkAuditLogs();
}
```

##  Future Extensions

### Planned Features

1. **Forward Secrecy**

   * Perfect Forward Secrecy
   * Periodic key rotation
   * Historic message protection

2. **Post-Quantum Readiness**

   * PQC integration
   * Hybrid schemes
   * Smooth migration

3. **Zero-Knowledge Proofs**

   * Stronger auth
   * Privacy protection
   * Verifiable computation

4. **Multi-Party Security**

   * Group key agreement
   * Secure group messaging
   * Privacy-preserving aggregation

### Roadmap

```
Phase 1 (Current): Basic Security
├─ AES-256 Encryption
├─ RSA-2048 Key Exchange
├─ Digital Signatures
└─ Basic Authentication

Phase 2 (Next): Enhanced Security
├─ Perfect Forward Secrecy
├─ Key Rotation
├─ Advanced Authentication
└─ Security Monitoring

Phase 3 (Future): Quantum-Ready
├─ Post-Quantum Crypto
├─ Hybrid Encryption
├─ Zero-Knowledge Proofs
└─ Multi-Party Computation
```

---

##  References

### Standards & Specs

* **RFC 5246** — TLS 1.2
* **RFC 8446** — TLS 1.3
* **NIST SP 800-57** — Key Management
* **FIPS 140-2** — Crypto Module Security

### Algorithms

* **AES** — NIST FIPS 197
* **RSA** — PKCS #1
* **SHA-256** — NIST FIPS 180-4
* **GCM** — NIST SP 800-38D

### Security Frameworks

* **OWASP**
* **NIST Cybersecurity Framework**
* **ISO 27001**
* **Common Criteria**

This document provides end-to-end technical details of the P2P chat app’s security mechanisms, serving as a deep implementation reference for developers and security practitioners.
