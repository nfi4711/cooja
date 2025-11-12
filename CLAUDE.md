# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Cooja is an open source network simulator for the Contiki-NG operating system. It's a Java-based application that provides instruction-level emulation of MSP430-based sensor network platforms and includes MSPSim for hardware emulation.

## Essential Commands

### Building
- `./gradlew build` - Build the project, creates `build/libs/cooja.jar`
- `./gradlew distTar` or `./gradlew distZip` - Create distributable package in `build/distributions/` with all dependencies
- `./gradlew fullJar` - Create fat JAR with all dependencies embedded
- `./gradlew copyDependencies` - Copy runtime dependencies to `build/libs/lib/`

### Running
- `./gradlew run` - Run Cooja directly via Gradle
- `java -jar build/libs/cooja.jar` - Run from built JAR (requires dependencies in `build/libs/lib/`)

### Code Quality
- `./gradlew spotlessApply` - Apply code formatting (uses Spotless with clang-format for C code)
- `./gradlew test` - Run JUnit tests
- ErrorProne static analysis available with `-Perrorprone` flag

### Development Options
- Use `-Passertions` to enable Java assertions
- Use `-PclangFormat=clang-format-15` to specify clang-format binary
- System properties prefixed with `cooja.` are passed to the application as `-D` flags

## Architecture

### Core Components
- **Main class**: `org.contikios.cooja.Main` - Application entry point
- **GUI**: `org.contikios.cooja.GUI` - Main GUI framework
- **Simulation**: `org.contikios.cooja.Simulation` - Core simulation engine
- **Motes**: `org.contikios.cooja.Mote` - Individual simulated nodes
- **Radio Medium**: `org.contikios.cooja.RadioMedium` - Wireless communication simulation

### Key Packages
- `org.contikios.cooja.contikimote` - Contiki mote implementations
- `org.contikios.cooja.mspmote` - MSP430 hardware emulation (Sky, Z1, Exp5438)
- `org.contikios.cooja.plugins` - GUI plugins and visualization tools
- `org.contikios.cooja.radiomediums` - Different radio propagation models
- `se.sics.mspsim` - MSPSim emulator integration

### Plugin System
Cooja uses an extensive plugin architecture. Plugins are located in:
- `org.contikios.cooja.plugins` - Core simulation plugins
- `org.contikios.cooja.mspmote.plugins` - MSP430-specific plugins

### Configuration
- `config/` - Contains configuration files, images, and default scripts
- `gradle.properties` - Gradle configuration
- Java toolchain version: 21
- JVM args: `-Xms400M -Xmx2048M -XX:-UseCompressedOops -XX:-UseCompressedClassPointers --enable-preview --enable-native-access ALL-UNNAMED`

## Testing
- Tests located in `src/test/java/`
- Uses JUnit Jupiter (JUnit 5)
- Integration test suite configured
- Test reports generated with per-test-case XML output

## MSP430 Platform Support
Supported hardware platforms:
- **Sky/TelosB**: `SkyMote`, `SkyMoteType`
- **Z1**: `Z1Mote`, `Z1MoteType`
- **Exp5438**: `Exp5438Mote`, `Exp5438MoteType`
- **CC430**: `CC430Node`

Each platform has specific radio implementations (CC2420, CC2520, CC1101, etc.) in the `interfaces` packages.
