# P2P Chat App — Testing Guide

##  Quick Start

### Environment Requirements

* Java 11 or later
* Maven 3.6 or later
* A system with JavaFX support

### Build the Project

```bash
cd P2pChat
mvn clean compile
```

##  Run the App

### Method 1: JavaFX Maven Plugin (Recommended)

```bash
# Start first node (default port 8080)
mvn javafx:run

# Start second node (port 8081, connect to first)
mvn javafx:run -Djavafx.args="8081 localhost:8080"

# Start third node (port 8082, connect to second)
mvn javafx:run -Djavafx.args="8082 localhost:8081"
```

### Method 2: Command-Line Mode

```bash
# Start CLI build (for debugging network features)
mvn exec:java -Dexec.mainClass="com.yourgroup.chat.Main" -Dexec.args="8080"
mvn exec:java -Dexec.mainClass="com.yourgroup.chat.Main" -Dexec.args="8081 localhost:8080"
```

##  Feature Tests

### 1. Basic Network Connectivity

**Steps**

1. Start the first node: `mvn javafx:run`
2. Start the second node: `mvn javafx:run -Djavafx.args="8081 localhost:8080"`
3. Observe connection status in both windows

**Expected**

* Both windows show “Connections: 1”
* The online members list shows the peer node
* System messages indicate connection success

---

### 2. Group Chat

**Steps**

1. Ensure at least two nodes are connected
2. Type a message in any node’s input box
3. Click **Send** or press **Enter**

**Expected**

* Sent message appears locally as a right-aligned blue bubble
* Other nodes see it as a left-aligned white bubble
* Messages include sender ID and timestamp

---

### 3. Private Chat

**Steps**

1. Double-click any online member in the member list
2. A private chat window pops up
3. Send a message in that window

**Expected**

* A separate private chat window opens
* Private messages appear only in that private window
* Main window shows messages marked “[Private]”
* Multiple private windows can be open simultaneously

---

### 4. Emoji Feature

**Steps**

1. Click the 😊 button next to the input box
2. Emoji picker pops up
3. Click any emoji
4. Send a message containing the emoji

**Expected**

* Emoji picker shows 4 categories correctly
* Clicking inserts at the cursor position
* Emoji renders properly in messages

> *Note: If your build removed emoji support, skip this test.*

---

### 5. File Transfer

**Steps**

1. Click the 📁 file button
2. Choose a file
3. Observe the receiver’s behavior

**Expected**

* Sender shows “Sending file…” status
* Receiver sees a confirmation dialog
* On accept, both sides display transfer messages

---

### 6. Multi-Node Network

**Steps**

1. Launch 3+ nodes
2. Form topology A–B–C
3. Send a message from node A

**Expected**

* Message floods to all nodes
* Every node receives it
* No duplicate displays

##  Troubleshooting

### Common Issues

**1. JavaFX runtime error**

```
Error: JavaFX runtime components are missing
```

**Fix**

* Ensure Java 11+ and JavaFX are available
* Prefer `mvn javafx:run` over running the JAR directly

**2. Port already in use**

```
java.net.BindException: Address already in use
```

**Fix**

* Use a different port
* Find and stop the process using the port

**3. Connection failed**

```
Unable to connect to node
```

**Fix**

* Ensure the target node is running
* Check firewall rules
* Verify IP/hostname and port

### Debug Mode

Enable verbose logging:

```bash
mvn javafx:run -Djavafx.options="-Djava.util.logging.level=ALL"
```

##  Performance Tests

### Network Latency

1. Start multiple nodes
2. Send a large volume of messages
3. Observe propagation latency

### Memory Usage

1. Run the app for an extended period
2. Send many messages and files
3. Monitor memory consumption

### Concurrent Connections

1. Start 10+ nodes
2. Run multiple private chats simultaneously
3. Check overall stability

##  UI Tests

### Responsive Layout

* Resize the window
* Try various screen resolutions
* Verify adaptive layout

### Emoji Rendering

* Test diverse Unicode emoji
* Check rendering across OSes
* Measure picker performance

##  Test Report Template

```
Test Date:
Environment:
Java Version:
Operating System:

Feature Results:
- [ ] Basic Connectivity
- [ ] Group Chat
- [ ] Private Chat
- [ ] Emoji Feature
- [ ] File Transfer
- [ ] Multi-Node Network

Issues Found:
1.
2.
3.

Recommendations:
1.
2.
3.
```

##  Advanced Scenarios

### Network Partition

1. Run 5 nodes to form a network
2. Disconnect a central node
3. Observe self-healing behavior

### Large File Transfer

1. Try transferring files >100 MB
2. Check stability during transfer
3. Verify file integrity

### Long-Running Stability

1. Run for 24+ hours
2. Send periodic messages/files
3. Watch for memory leaks and performance degradation

---

**Note:** This is an educational project. Please test in a safe network environment.
