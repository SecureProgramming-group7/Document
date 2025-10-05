### Secure Communication Protocol (SCP) v1.0

**1. Protocol Objectives**

This Secure Communication Protocol (SCP) provides end-to-end encryption (E2EE) for the P2P chat application, ensuring that only the communicating parties can read message content. It protects **confidentiality**, **integrity**, and **authenticity**, and offers forward secrecy and replay resistance.

**2. Core Components**

* **Long-Term Identity Keys:** Each node holds an RSA-2048 key pair (public/private). The public key is used for identification and signatures; the private key must be securely stored and never leave the node.
* **Session Keys:** For each private or group chat session, participants negotiate an ephemeral symmetric key. The protocol uses **AES-256-GCM** for session encryption. GCM provides both confidentiality and integrity (authentication).
* **Key Exchange Protocol:** **Elliptic Curve Diffie–Hellman (ECDH)** is used to securely agree on session keys, specifically `Curve25519`.

**3. Protocol Flow**

**3.1. 1-on-1 Chat Session Establishment**

When node A initiates a secure session with node B:

1. **Initiation**

   * A generates an ephemeral ECDH key pair (`E_pubA`, `E_privA`).
   * A signs `E_pubA` with its **long-term identity private key**, producing `SigA`.
   * A sends a `KEY_EXCHANGE` message to B containing:

     * A’s long-term identity public key (`RSA_pubA`)
     * A’s ephemeral ECDH public key (`E_pubA`)
     * The signature over `E_pubA` (`SigA`)

2. **Response & Key Generation**

   * Upon receipt, B verifies `SigA` using `RSA_pubA` to confirm the request is from A.
   * If valid, B generates its own ephemeral ECDH key pair (`E_pubB`, `E_privB`).
   * B signs `E_pubB` with its **long-term identity private key**, producing `SigB`.
   * Using `E_privB` and received `E_pubA`, B computes the shared secret `S`.
   * B derives the final AES-256 session key `K_session` from `S` via a KDF such as HKDF.
   * B replies with a `KEY_EXCHANGE` message containing:

     * B’s long-term identity public key (`RSA_pubB`)
     * B’s ephemeral ECDH public key (`E_pubB`)
     * The signature over `E_pubB` (`SigB`)

3. **Session Start**

   * A verifies `SigB` using `RSA_pubB`.
   * If valid, A computes the same shared secret `S` using `E_privA` and `E_pubB`.
   * A derives the same `K_session` using the same KDF.
   * Both sides now share `K_session` and can communicate using AES-256-GCM.

**3.2. Message Encryption & Decryption**

* **Encryption**

  * The sender encrypts plaintext with `K_session` and a unique, never-reused **nonce** (IV) using AES-256-GCM.
  * The transmitted `SECURE_PRIVATE_CHAT` payload includes:

    * The ciphertext
    * The nonce used
    * The GCM authentication tag

* **Decryption**

  * The receiver decrypts using `K_session`, the received nonce, and the tag.
  * If tag verification fails, the message is considered tampered with and must be discarded.

**4. Security Properties**

* **Confidentiality:** Only holders of the session key can decrypt.
* **Integrity:** GCM’s auth tag detects any modification to the ciphertext.
* **Authenticity:** Signatures during key exchange authenticate the parties.
* **Forward Secrecy:** Because session keys derive from ephemeral ECDH keys, past sessions remain secure even if long-term keys are later compromised.
* **Replay Resistance:**

  * The receiver maintains a sliding window of recently seen message IDs or hashes.
  * Any message found in this window is discarded.
  * The uniqueness of the GCM nonce further helps prevent replay.

**5. Planned Vulnerabilities (for coursework) — *For Internal Use Only***

To satisfy the assignment’s requirements, one or more of the following weaknesses may be deliberately introduced in the implementation:

1. **Lax signature verification:** In step 2, B replies with its ECDH public key *before* fully validating A’s signature; an attacker could spoof A to consume B’s resources. A subtler variant: after partial verification, a corner-case branch (e.g., an error handler) skips remaining checks and proceeds with an insufficiently verified key.

2. **Nonce reuse:** In AES-GCM, intentionally use a predictable or reused nonce generator (e.g., a simple counter that resets after node restart). Nonce reuse in GCM is catastrophic and can expose keys.

3. **Weak KDF:** Replace standard HKDF with a simplistic custom KDF, such as deriving the session key by directly hashing the shared secret `S` with SHA-256. This weakens key material and eases cryptanalysis.

4. **Logging leakage:** In debug logs, inadvertently print sensitive artifacts (e.g., ephemeral ECDH private keys or derived session keys). While production may raise the log level, this is still a critical flaw in debug mode.
