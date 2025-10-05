# JavaFX Installation Guide

##  Error

```
java.lang.module.FindException: Module javafx.controls not found
```

##  Cause

Your Java environment is missing JavaFX modules. Since Java 11, JavaFX is no longer bundled with the standard JDK.

##  Solutions

### Option 1: Install a JDK that includes JavaFX (Recommended)

**Azul Zulu JDK with JavaFX (easiest):**

1. Go to: [https://www.azul.com/downloads/?package=jdk-fx](https://www.azul.com/downloads/?package=jdk-fx)
2. Select your OS (Windows).
3. Download and install Zulu JDK FX.
4. Run your app again.

**Liberica JDK Full:**

1. Go to: [https://bell-sw.com/pages/downloads/](https://bell-sw.com/pages/downloads/)
2. Choose the **Full** build (includes JavaFX).
3. Download and install.
4. Run your app again.

### Option 2: Download JavaFX SDK separately

1. **Get the JavaFX SDK:**

   * Visit [https://openjfx.io/](https://openjfx.io/)
   * Download the Windows JavaFX SDK.
   * Extract it, e.g. `C:\javafx-sdk-17.0.2`

2. **Run with the SDK:**

```cmd
java --module-path "C:\javafx-sdk-17.0.2\lib" --add-modules javafx.controls,javafx.fxml -jar target\p2p-chat-1.0-SNAPSHOT.jar
```

### Option 3: Use the CLI version (no JavaFX needed)

If you just want to test functionality quickly:

```cmd
REM Run the CLI version
java -cp target\classes com.group7.chat.Main

REM Or double-click
start-cli.bat
```

### Option 4: Run with Maven (if Maven is installed)

```cmd
mvn clean compile
mvn javafx:run
```

##  Recommended Steps

### Easiest path:

1. **Uninstall your current Java.**
2. **Install Azul Zulu JDK FX:** [https://www.azul.com/downloads/?package=jdk-fx](https://www.azul.com/downloads/?package=jdk-fx)
3. **Run:** `start-simple.bat`

### Quick test:

1. **Double-click:** `start-cli.bat`
2. Use the command-line UI to verify the app.

##  Verify Installation

After installation, run:

```cmd
java -version
java --list-modules | findstr javafx
```

If you see JavaFX modules listed, you’re good to go.

##  Create a custom launch script

If your JavaFX SDK is at `C:\javafx-sdk-17.0.2`, create a `.bat` file:

```batch
@echo off
java --module-path "C:\javafx-sdk-17.0.2\lib" --add-modules javafx.controls,javafx.fxml -jar target\p2p-chat-1.0-SNAPSHOT.jar
pause
```

##  Notes

1. Avoid non-ASCII (e.g., Chinese) characters in file paths.
2. Ensure Java 11 or newer.
3. Match JavaFX and Java versions for compatibility.

**Tip:** Option 1 (Azul Zulu JDK FX) is the simplest.
