# The Cooja Network Simulator

Cooja is an open source network simulator.

## MSPSim support for the Cooja Simulator

MSPSim is a Java-based instruction level emulator of the MSP430 series
microprocessor and emulation of some sensor networking platforms. It is used
by Cooja to emulate MSP430 based platforms and is part of the Cooja
source code.

## Building

To build Cooja, you can run `./gradlew build`. Cooja is then provided in `build/libs/cooja.jar` as a JAR file. The dependencies are located in `build/libs/lib`.

To build Cooja easily executable and with all dependencies, you can use the following command:
```
./gradlew distTar
```
or
```
./gradlew distZip
```
This command creates a compressed folder in `build/distributions/` which contains both the JAR file and a platform-independent script for execution.

### Building in Network-Isolated Environments

If you need to build Cooja in an environment without network access, you must first pre-populate the Gradle cache from a machine with network connectivity.

**Required artifacts that need to be cached:**
- Gradle wrapper distribution (8.14.2)
- Gradle plugins (spotless, errorprone, foojay-resolver-convention)
- Maven dependencies (~15+ libraries from Maven Central)

**Steps to prepare for offline builds:**

1. On a machine **with network access**, run:
   ```bash
   ./gradlew build --refresh-dependencies
   ```

2. Package the Gradle cache:
   ```bash
   cd ~
   tar -czf gradle-cache.tar.gz .gradle/
   ```

3. Transfer `gradle-cache.tar.gz` to the isolated environment and extract:
   ```bash
   cd ~
   tar -xzf gradle-cache.tar.gz
   ```

4. Build offline:
   ```bash
   ./gradlew build --offline
   ```

**Alternative:** Use the provided helper script to cache only dependencies:
```bash
gradle -b cache-dependencies.gradle cacheDeps
```

Without network access or a pre-populated cache, the build will fail with DNS resolution errors when attempting to download dependencies from:
- `services.gradle.org` (Gradle wrapper)
- `plugins.gradle.org` (Gradle plugins)
- `repo.maven.apache.org` (Maven Central dependencies)
