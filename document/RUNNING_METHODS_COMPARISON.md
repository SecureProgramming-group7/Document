# Run Modes Explained

##  Why did `mvn javafx:run` work before?

Great question! Here’s how each run mode works and why results can differ.

##  Three Run Modes Compared

### 1. `mvn javafx:run` (JavaFX Maven Plugin)

**How it works:**

```bash
mvn clean compile
mvn javafx:run
```

**What happens under the hood:**

1. Maven compiles sources to `target/classes`.
2. The JavaFX plugin fetches JavaFX runtime automatically.
3. The plugin sets module path and JVM args.
4. It launches your main class.

**Pros:**

*  Handles JavaFX deps automatically
*  Sets module path for you
*  Convenient during development
*  No JAR packaging required

**Cons:**

*  Requires Maven
*  Needs network for deps on first run
*  May fail on servers (no GUI)
*  Not for end users

---

### 2. `java -jar p2p-chat-1.0-SNAPSHOT.jar` (Fat JAR)

**How it works:**

```bash
mvn clean package
java -jar target/p2p-chat-1.0-SNAPSHOT.jar
```

**What happens under the hood:**

1. Maven Shade plugin bundles all dependencies into one JAR.
2. MANIFEST.MF has the correct Main-Class.
3. Runs standalone—no extra deps.

**Pros:**

*  Self-contained; no Maven needed
*  All deps included
*  Ideal for distribution
*  Single-file delivery

**Cons:**

*  Larger file (~8.4 MB)
*  Requires packaging step

---

### 3. `java -cp target/classes` (Run compiled classes directly)

**How it works:**

```bash
mvn clean compile
java -cp target/classes com.group7.chat.Main
```

**What happens under the hood:**

1. Runs compiled classes directly.
2. Relies on system-installed JavaFX (if needed).
3. No external deps bundled.

**Pros:**

*  Fast startup
*  Good for quick dev/debug
*  Tiny footprint

**Cons:**

*  You manage deps yourself
*  JavaFX may be missing
*  Not for distribution

---

##  Comparison Table

| Feature               | `mvn javafx:run` | `java -jar` (Fat JAR) | `java -cp` (classes) |
| --------------------- | ---------------- | --------------------- | -------------------- |
| Dependency mgmt       | Automatic        | Bundled               | Manual               |
| JavaFX support        | Automatic        | Included              | System-provided      |
| Size                  | N/A              | ~8.4 MB               | ~113 KB              |
| Network needed        | First run        | No                    | No                   |
| Needs Maven           | Yes              | No                    | Build only           |
| Distribution-friendly | Bad                | Good                     | Bad                    |
| Dev-friendly          | Good                | Normal                    | Good                    |
| End-user-friendly     | Bad                | Good                     | Bad                    |

---

##  When to use which?

### Development

```bash
# Quick CLI test
mvn clean compile
java -cp target/classes com.group7.chat.Main

# GUI test (if JavaFX available)
mvn javafx:run
```

### Testing

```bash
# Test the distributable
mvn clean package
java -jar target/p2p-chat-1.0-SNAPSHOT.jar
```

### Distribution

```bash
# Provide just this file
target/p2p-chat-1.0-SNAPSHOT.jar

# Users run:
java -jar p2p-chat-1.0-SNAPSHOT.jar
```

---

##  Why might `mvn javafx:run` fail now?

Typical server/CI issues:

1. **No GUI**: Headless environments lack display.
2. **JavaFX modules**: Module path complexity.
3. **Permissions**: Window creation blocked.
4. **Dependency conflicts**: Mixed JavaFX versions.

---

##  Best Practices

**For developers**

* Use `mvn javafx:run` or direct class runs while coding.
* Validate the final user experience with the Fat JAR.
* Use IDE/CLI for targeted debugging.

**For end users**

* Ship only the Fat JAR.
* Simple command: `java -jar xxx.jar`
* No Maven or dependency juggling.

---

##  Recommended Workflow

```bash
# 1) Dev – fast iteration
mvn clean compile
java -cp target/classes com.group7.chat.Main

# 2) Feature test – GUI
mvn javafx:run  # if environment supports JavaFX

# 3) Release prep – build artifact
mvn clean package

# 4) Final UX test
java -jar target/p2p-chat-1.0-SNAPSHOT.jar

# 5) Distribute
# Provide p2p-chat-1.0-SNAPSHOT.jar only
```

That’s why `mvn javafx:run` worked previously, but the Fat JAR is now the preferred, user-friendly way to run it.

