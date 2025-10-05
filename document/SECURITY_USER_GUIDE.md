#  P2P Chat App — Secure Features User Guide

##  Overview

This P2P chat app implements enterprise-grade security with end-to-end encryption, authentication, and digital signatures.

###  Security Features

* **End-to-end encryption:** AES-256 for all message content
* **RSA key exchange:** 2048-bit RSA secures key exchange
* **Digital signatures:** Prevent tampering and forgery
* **Identity verification:** Public-key–based node auth
* **Secure file transfer:** Fully encrypted file sending
* **Key management:** Automated key generation, storage, rotation

---

##  Quick Start

### Enable Security

Security initializes automatically when a node starts:

```java
Node node = new Node(8080);
node.start();

// Check security status
if (node.isSecurityEnabled()) {
    System.out.println("Security enabled");
    System.out.println(node.getSecurityManager().getSecurityStatus());
}
```

### Basic Security Settings

```java
SecurityManager securityManager = node.getSecurityManager();

// Strict mode (encrypted-only traffic)
securityManager.setStrictMode(true);

// View status
String status = securityManager.getSecurityStatus();
System.out.println(status);
```

---

##  Key Management

### Auto Key Generation

On startup a node keypair is created:

```
=== Security Manager Init ===
[KeyManager] Generating new node key pair
[KeyManager] Saved to: keys/
[KeyManager] Public key fingerprint: ABC123...
```

### Key Exchange

A key exchange runs automatically when connecting to a new node:

```java
// Manually trigger key exchange
boolean success = securityManager.handleKeyExchange("target-node-id");
if (success) {
    System.out.println("Key exchange succeeded");
}
```

### Inspect Key State

```java
KeyManager keyManager = securityManager.getKeyManager();

boolean hasSessionKey = keyManager.hasSessionKey("node-id");
boolean hasPublicKey  = keyManager.hasPublicKey("node-id");

int sessionKeyCount = keyManager.getSessionKeyCount();
int publicKeyCount  = keyManager.getPublicKeyCount();
```

---

##  Secure Messaging

### Automatic Encryption

Messages are encrypted automatically when keys exist:

```java
// Send (auto-encrypts if keys available)
Message msg = new Message(Message.Type.CHAT, nodeId, "Hello World!");
String processed = securityManager.processOutgoingMessage(msg, targetNodeId);

// Receive (auto-decrypts)
Message received = securityManager.processIncomingMessage(rawMessage, senderNodeId);
```

### Enforce Encryption

```java
securityManager.setStrictMode(true);  // Rejects any unencrypted traffic
```

### Integrity Verification

All messages carry signatures and are verified automatically:

```
[Security] Integrity OK: node-123
[Security] Signature verified: node-456
```

---

##  Secure File Transfer

### Send Encrypted Files

```java
SecurityManager securityManager = node.getSecurityManager();

SecureFileTransferService.TransferResult result =
    securityManager.sendSecureFile("target-node-id",
                                   "/path/to/file.txt",
                                   "/save/path/file.txt");

if (result.isSuccess()) {
    System.out.println("File sent: " + result.getBytesTransferred() + " bytes");
} else {
    System.err.println("File transfer failed: " + result.getErrorMessage());
}
```

### Receive Encrypted Files

```
[SecureFile] Incoming request: document.pdf (1.2MB)
[SecureFile] Decrypting...
[SecureFile] Saved to: /downloads/document.pdf
[SecureFile] Integrity verified
```

---

##  Authentication

### Node Authentication

```java
AuthenticationService authService = securityManager.getAuthenticationService();

AuthenticationService.AuthenticationResult result =
    securityManager.authenticateNode("node-id", "public-key-string");

switch (result) {
    case SUCCESS: System.out.println("Authentication succeeded"); break;
    case FAILED:  System.out.println("Authentication failed");    break;
    case PENDING: System.out.println("Authenticating...");        break;
}
```

### Trust Levels

```java
int trust = authService.getTrustLevel("node-id");
System.out.println("Trust: " + trust + "/100");

authService.updateTrustLevel("node-id", 85);
```

### Identity Certificates

```java
String cert = securityManager.exportNodeCertificate();
System.out.println("Certificate: " + cert);

boolean valid = securityManager.importNodeCertificate(cert);
if (valid) System.out.println("Certificate verified");
```

---

##  Security Monitoring

### Real-Time Status

```java
String securityStatus = securityManager.getSecurityStatus();
System.out.println(securityStatus);
```

Example:

```
=== Security Summary ===
Security: Enabled
Strict Mode: On
Node ID: Node-1234567890
Session Keys: 3
Known Public Keys: 5
Nodes: total=5, verified=3, trusted=2, active challenges=1
Active File Transfers: 0
```

### Encryption Status per Peer

```java
SecureMessageHandler h = securityManager.getSecureMessageHandler();
String state = h.getEncryptionStatus("node-id");
System.out.println("Encryption: " + state);
// "Fully Encrypted" / "Key Exchange Required" / "Unencrypted"
```

---

##  Advanced Configuration

### Toggle Security

```java
securityManager.setSecurityEnabled(false); // testing only
securityManager.setSecurityEnabled(true);
boolean isEnabled = securityManager.isSecurityEnabled();
```

### Key Rotation

```java
KeyManager km = securityManager.getKeyManager();

km.removeSessionKey("old-node-id");
SecretKey newKey = km.generateSessionKey();
km.storeSessionKey("node-id", newKey);
```

### Housekeeping

* **Every 5 min:** purge expired auth challenges
* **Every 30 min:** print security stats
* **On startup:** verify key file integrity

---

##  Troubleshooting

**1. Key exchange failed**

```
[Security] Key exchange failed: target-node-id
```

Fix:

* Ensure the target node is online
* Confirm the target public key is correct
* Retry the connection

**2. Message decryption failed**

```
[Security] Incoming processing failed: decryption error
```

Fix:

* Check that a session key exists
* Validate the message format
* Re-run key exchange

**3. File encryption/transfer failed**

```
[SecureFile] Encryption failed: missing session key
```

Fix:

* Establish a session key with the target
* Trigger key exchange manually
* Check file permissions

### Debug Mode

Enable verbose logs:

```java
-Djava.util.logging.level=FINE
```

---

##  Best Practices

### 1. Key Management

* Back up key files regularly
* Never transmit private keys over insecure channels
* Rotate session keys periodically

### 2. Network Security

* Keep **strict mode** enabled in production
* Re-verify peer identities periodically
* Monitor anomalous connection behavior

### 3. File Transfer

* Verify peer identity before large/sensitive transfers
* Use extra access controls for sensitive files
* Clean up temporary files regularly

### 4. Monitoring & Audit

* Review security status routinely
* Log all security-relevant events
* Track trust level changes

---

##  API Reference

### `SecurityManager` (main)

| Method                      | Description                    |
| --------------------------- | ------------------------------ |
| `isSecurityEnabled()`       | Check if security is enabled   |
| `setStrictMode(boolean)`    | Enforce encrypted-only traffic |
| `getSecurityStatus()`       | Summary of security state      |
| `handleKeyExchange(String)` | Run a key exchange with a peer |
| `sendSecureFile(...)`       | Send an encrypted file         |
| `exportNodeCertificate()`   | Export the node’s certificate  |

### `KeyManager`

| Method                  | Description                  |
| ----------------------- | ---------------------------- |
| `hasSessionKey(String)` | Check for a session key      |
| `hasPublicKey(String)`  | Check for a known public key |
| `getSessionKeyCount()`  | Number of session keys       |
| `getPublicKeyCount()`   | Number of known public keys  |

### `AuthenticationService`

| Method                     | Description              |
| -------------------------- | ------------------------ |
| `authenticateNode(...)`    | Authenticate a node      |
| `getTrustLevel(String)`    | Get a node’s trust score |
| `updateTrustLevel(...)`    | Set/update trust score   |
| `getAuthenticationStats()` | View auth statistics     |

---

##  Support

If you encounter issues:

1. Check security logs in the console
2. Verify key files are intact
3. Confirm network connectivity
4. See the Troubleshooting section above

These security features provide enterprise-grade protection for your P2P chat—ensuring confidentiality, integrity, and availability.
