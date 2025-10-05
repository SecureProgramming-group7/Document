# Quick Fix Guide

##  Common Issues

### 1. Seeing mojibake/garbled text

Your batch script may have an encoding problem.

### 2. “Module javafx.controls not found”

Your Java environment is missing JavaFX modules.

##  Immediate Solutions

### If JavaFX is missing (recommended easiest):

1. Download a JDK that includes JavaFX: [https://www.azul.com/downloads/?package=jdk-fx](https://www.azul.com/downloads/?package=jdk-fx)
2. Re-run `start.bat`

**Quick test (no JavaFX needed):**

```cmd
start-cli.bat
```

### If JavaFX is installed but you still have issues:

**Method 1: Run via command line**
Open Command Prompt, cd into the project directory, then:

```cmd
java --module-path . --add-modules javafx.controls,javafx.fxml -jar target\p2p-chat-1.0-SNAPSHOT.jar
```

**Method 2: Use the simple script**
Double-click: `start.bat`

**Method 3: Run CLI version**

```cmd
:: compile first (if needed)
mvn clean compile

:: then run
java -cp target\classes com.group7.chat.Main
```

Or double-click `start-cli.bat`.

##  If the `java` command doesn’t work

1. **Check Java is installed:**

   ```cmd
   java -version
   ```

2. **If not installed:**

   * Install Java 11+
   * Recommended: Azul Zulu JDK (with JavaFX): [https://www.azul.com/downloads/?package=jdk-fx](https://www.azul.com/downloads/?package=jdk-fx)

3. **If installed but command fails:**

   * Ensure Java’s `bin` is in your PATH
   * Restart Command Prompt

##  Full Steps

1. Download the project
2. Extract to a folder
3. Open Command Prompt
4. Go to the project folder: `cd C:\path\to\P2pChat`
5. Run:

   ```cmd
   java --module-path . --add-modules javafx.controls,javafx.fxml -jar target\p2p-chat-1.0-SNAPSHOT.jar
   ```

##  Still stuck?

See the detailed guide: `JAVAFX_RUNTIME_SOLUTIONS.md`

