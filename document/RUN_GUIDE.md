# P2P Chat App – Run Guide

##  System Requirements

* Java 11 or newer
* A JavaFX-capable environment (for GUI mode)

##  Ways to Run

### Method 1: GUI Mode (Recommended)

**Windows:**

```bash
# Double-click
start-gui.bat

# Or run from terminal
java -jar ./target/p2p-chat-1.0-SNAPSHOT.jar
```

**Linux/Mac:**

```bash
# Use the launcher script
./start-gui.sh

# Or run directly
java -jar ./target/p2p-chat-1.0-SNAPSHOT.jar
```

### Method 2: Command-Line Mode

**Windows:**

```bash
# Double-click
start-cli.bat

# Or run from terminal
java -cp ./target/classes com.group7.chat.Main
```

**Linux/Mac:**

```bash
# Use the launcher script
./start-cli.sh

# Or run directly
java -cp ./target/classes com.group7.chat.Main
```

### Method 3: Run with Maven

```bash
# Compile the project
mvn clean compile

# Run CLI version
mvn exec:java -Dexec.mainClass="com.group7.chat.Main"

# Run GUI version (requires JavaFX)
mvn javafx:run
```

### Method 4: Run Class Files Directly

```bash
# Compile the project
mvn clean compile

# Run CLI version
java -cp target/classes com.group7.chat.Main

# Run GUI version (needs JavaFX module path)
java --module-path /path/to/javafx/lib --add-modules javafx.controls,javafx.fxml -cp target/classes com.group7.chat.gui.ChatApplication
```

##  Files Overview

* `target/p2p-chat-1.0-SNAPSHOT.jar` — Executable fat JAR with all dependencies (~8.8 MB) — **launches GUI**
* `target/decentralized-chat-1.0-SNAPSHOT.jar` — Thin JAR with project classes only (~115 KB)
* `start-gui.bat` / `start-gui.sh` — GUI mode launch scripts
* `start-cli.bat` / `start-cli.sh` — CLI mode launch scripts

##  Usage

### Command-Line Mode

After starting, you can use:

* `connect <host:port>` — connect to a node
* `send <message>` — broadcast a message to connected nodes
* `status` — show current status
* `quit` — exit

### GUI Mode

In the GUI you can:

1. View the online members list
2. Send group chat messages
3. Start private chats
4. Transfer files
5. View connection status

##  Troubleshooting

### “Unable to access jarfile”

1. Ensure you’re in the correct directory (the one with the `target` folder).
2. Verify the JAR exists: `ls -la target/*.jar`
3. Use the correct file name: `p2p-chat-1.0-SNAPSHOT.jar`

### JavaFX-related errors

1. Make sure your Java setup includes JavaFX.
2. Or use the CLI version: `java -cp target/classes com.group7.chat.Main`

### Port already in use

1. Change the default port: `java -jar target/p2p-chat-1.0-SNAPSHOT.jar 8081`
2. Or stop the process occupying the port.

##  Network Configuration

* Default listen port: **8080**
* File transfer port: **9080**
* Secure file transfer port: **10080**

Ensure your firewall allows traffic on these ports.

##  Notes

1. An RSA key pair is generated automatically on first run.
2. The project supports end-to-end encrypted communication.
3. Includes a distributed overlay networking capability.
4. Intentionally includes security vulnerabilities for learning/research.
