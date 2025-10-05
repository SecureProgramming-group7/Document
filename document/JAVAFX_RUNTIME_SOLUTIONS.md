# JavaFX Runtime Issue — Solutions

##  Error

```
Error: JavaFX runtime components are missing, and are required to run this application
```

##  Cause

Since Java 11, JavaFX is no longer bundled with the JDK. Even if your JAR includes JavaFX classes, some Java versions still require specific launch flags.

##  Solutions

### Option 1: Use module-path flags (Recommended)

**Windows:**

```cmd
java --module-path . --add-modules javafx.controls,javafx.fxml -jar p2p-chat-1.0-SNAPSHOT.jar
```

**Linux/Mac:**

```bash
java --module-path . --add-modules javafx.controls,javafx.fxml -jar p2p-chat-1.0-SNAPSHOT.jar
```

### Option 2: Install the JavaFX SDK

1. **Download JavaFX SDK:**

   * Visit [https://openjfx.io/](https://openjfx.io/)
   * Download the SDK for your OS
   * Extract, e.g. `C:\javafx-sdk-17.0.2`

2. **Run with the SDK:**

```cmd
java --module-path "C:\javafx-sdk-17.0.2\lib" --add-modules javafx.controls,javafx.fxml -jar p2p-chat-1.0-SNAPSHOT.jar
```

### Option 3: Use the CLI build (no JavaFX)

If the GUI won’t start:

1. **Build the project:**

```cmd
mvn clean compile
```

2. **Run the CLI app:**

```cmd
java -cp target\classes com.group7.chat.Main
```

### Option 4: Install a JDK that includes JavaFX

* **Azul Zulu FX:** [https://www.azul.com/downloads/?package=jdk-fx](https://www.azul.com/downloads/?package=jdk-fx)
* **Liberica JDK Full:** [https://bell-sw.com/pages/downloads/](https://bell-sw.com/pages/downloads/)

##  Quick-Run Scripts

### Windows (run-with-javafx.bat)

```batch
@echo off
echo Trying to run P2P Chat with JavaFX support...

REM Method 1: module path
java --module-path . --add-modules javafx.controls,javafx.fxml -jar p2p-chat-1.0-SNAPSHOT.jar
if %ERRORLEVEL% EQU 0 goto :ok

REM Method 2: direct
java -jar p2p-chat-1.0-SNAPSHOT.jar
if %ERRORLEVEL% EQU 0 goto :ok

REM Method 3: CLI
java -cp target/classes com.group7.chat.Main
if %ERRORLEVEL% EQU 0 goto :ok

echo All methods failed. Please check your JavaFX setup.
goto :end

:ok
echo Application started successfully!
:end
pause
```

### Linux/Mac (run-with-javafx.sh)

```bash
#!/bin/bash
echo "Trying to run P2P Chat with JavaFX support..."

# Method 1: module path
if java --module-path . --add-modules javafx.controls,javafx.fxml -jar p2p-chat-1.0-SNAPSHOT.jar; then
  echo "Application started successfully!"
  exit 0
fi

# Method 2: direct
if java -jar p2p-chat-1.0-SNAPSHOT.jar; then
  echo "Application started successfully!"
  exit 0
fi

# Method 3: CLI
if java -cp target/classes com.group7.chat.Main; then
  echo "Application started successfully!"
  exit 0
fi

echo "All methods failed. Please check your JavaFX installation."
exit 1
```

##  Check Your Java

```cmd
java -version
```

**Recommended setups:**

* Java 11 + JavaFX SDK
* Java 17 + JavaFX SDK
* Or a JDK distribution that bundles JavaFX

##  Troubleshooting Steps

1. Check Java version: `java -version`
2. Try **Option 1** (module-path flags)
3. If it fails, install the JavaFX SDK (Option 2)
4. As a last resort, run the CLI build (Option 3)

##  Why this happens

1. **Java Module System:** introduced in Java 9.
2. **JavaFX unbundled:** since Java 11, it’s separate from the JDK.
3. **Module path:** you must explicitly point to JavaFX modules at runtime.

##  Recommended

**Easiest:** install **Azul Zulu FX** (includes JavaFX) or use the provided launch scripts.
**Most reliable:** install the **JavaFX SDK** and launch with the full `--module-path ... --add-modules ...` command.
