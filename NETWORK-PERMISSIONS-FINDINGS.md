# Network Permissions Analysis for Cooja Project Build

## Executive Summary

The Cooja project build requires network access but **Java does not automatically use the HTTP_PROXY/HTTPS_PROXY environment variables**. While network connectivity exists and the proxy is properly configured (as evidenced by successful `curl` commands), Java applications require explicit proxy configuration via system properties to access external resources.

## Problem Statement

When attempting to build the Cooja project using `./gradlew build`, the following error occurs:

```
Exception in thread "main" java.net.UnknownHostException: services.gradle.org
```

This happens because:
1. Java's networking stack does not honor HTTP_PROXY/HTTPS_PROXY environment variables by default
2. The build tool (Gradle) is a Java application and inherits this limitation
3. Without proxy configuration, Java cannot resolve DNS or connect to external hosts

## Network Requirements

### Required Endpoints for Building

The build process requires access to the following hosts:

#### Gradle Infrastructure
- `services.gradle.org:443` - Gradle wrapper distribution
- `plugins.gradle.org:443` - Gradle plugins

#### Maven Repositories
- `repo.maven.apache.org:443` (or `repo1.maven.org`) - Maven Central repository

#### Specific Dependencies
The project requires downloading the following dependencies from Maven Central:
- ch.qos.logback:logback-classic:1.5.20
- com.github.cliftonlabs:json-simple:4.0.1
- com.formdev:flatlaf:3.6.2
- de.sciss:syntaxpane:1.3.0
- info.picocli:picocli:4.7.7
- org.jdom:jdom2:2.0.6.1
- org.jfree:jfreechart:1.5.6
- org.openjdk.nashorn:nashorn-core:15.7
- org.slf4j:slf4j-api:2.0.17
- org.swinglabs.swingx:swingx-autocomplete:1.6.5-1
- com.google.errorprone:error_prone_core:2.40.0

## Current Environment Status

### Environment Configuration
- **Proxy**: Configured and functional (`HTTP_PROXY`, `HTTPS_PROXY` environment variables are set)
- **DNS Resolution**: Working at OS level
- **Network Connectivity**: Confirmed working (curl successfully connects to required hosts)
- **Proxy Allowlist**: All required hosts (services.gradle.org, repo.maven.apache.org, etc.) are included

### Test Results

| Tool | DNS Resolution | HTTP Connection | Status |
|------|----------------|-----------------|--------|
| `curl` | ✅ Success | ✅ Success | Working |
| Java | ❌ Failure | ❌ Failure | **Not Working** |
| Gradle | ❌ Failure | ❌ Failure | **Not Working** |

### Java Configuration
- **Java Version**: OpenJDK 21.0.8
- **Security Manager**: Disabled (not restricting network access)
- **Java Security Policies**: Checked, no restrictive rules found
- **Issue**: Java not using proxy from environment variables

## Root Cause

Java requires explicit configuration to use HTTP/HTTPS proxies. The standard environment variables (`HTTP_PROXY`, `HTTPS_PROXY`) work for tools like `curl`, `wget`, and many others, but Java ignores them unless configured through:

1. **System Properties**:
   - `-Dhttp.proxyHost=<host>`
   - `-Dhttp.proxyPort=<port>`
   - `-Dhttps.proxyHost=<host>`
   - `-Dhttps.proxyPort=<port>`

2. **Gradle Properties** (in `gradle.properties`):
   - `systemProp.http.proxyHost=<host>`
   - `systemProp.http.proxyPort=<port>`
   - `systemProp.https.proxyHost=<host>`
   - `systemProp.https.proxyPort=<port>`

3. **GRADLE_OPTS Environment Variable**:
   ```bash
   export GRADLE_OPTS="-Dhttp.proxyHost=<host> -Dhttp.proxyPort=<port> ..."
   ```

## Attempted Solutions

### Solution 1: gradle.properties Configuration
Created `/home/user/cooja/gradle.properties` with proxy settings extracted from environment variables.

**Result**: Java successfully connected to the proxy but received `401 Unauthorized` error. This is because the proxy uses JWT-based authentication embedded in the username, which Java's HTTP client doesn't handle in the same way as curl.

### Solution 2: System Proxy Detection
Java's proxy authentication mechanism differs from curl's URL-based authentication. The proxy credentials are embedded in the proxy URL in the environment variables, but Java's `systemProp.http.proxyUser` and `systemProp.http.proxyPassword` use a different authentication mechanism.

**Result**: Authentication failure - the JWT token format in the proxy credentials is not compatible with Java's Basic/Digest proxy authentication.

## Conclusion

### Missing Network Permissions

**Yes, there are network permissions missing**:

1. **Java cannot access the proxy without explicit configuration** - While the proxy is configured at the environment level, Java needs system properties to be set

2. **Proxy authentication incompatibility** - The proxy uses JWT-based authentication embedded in the URL (`http://user:jwt_token@host:port`), but Java's proxy configuration expects separate host/port and user/password fields with Basic/Digest authentication

3. **No automatic proxy detection** - Java 21 does not automatically detect and use system-level proxy settings from environment variables

### Required Permissions/Configuration

To enable the build, one of the following is needed:

1. **Configure Java to use system proxy with proper authentication** - This may require a custom proxy authenticator or different authentication method

2. **Allowlist the required domains** - If there's a firewall or network policy, add:
   - services.gradle.org
   - plugins.gradle.org
   - repo.maven.apache.org (or repo1.maven.org)

3. **Use a proxy that supports Java's authentication methods** - The current proxy uses JWT tokens which aren't directly compatible with Java's built-in proxy authentication

4. **Direct network access** - If possible, bypass the proxy for these specific Maven/Gradle domains

## Recommendations

1. **For Infrastructure Teams**: Consider adding support for Java applications to your proxy, either by:
   - Providing an alternative authentication method (Basic/Digest)
   - Configuring IP-based allowlisting for trusted hosts
   - Implementing a transparent proxy that doesn't require authentication

2. **For Developers**: Until infrastructure changes are made:
   - Document this limitation for other developers
   - Consider creating a wrapper script that configures Java proxy settings
   - Use pre-downloaded dependencies if available

## Technical Details

- **Environment**: Ubuntu on Linux 4.4.0
- **Java**: OpenJDK 21.0.8
- **Gradle**: 8.14.2 (via wrapper)
- **Build Tool**: Gradle with Gradle Wrapper
- **Proxy Type**: HTTP proxy with JWT authentication
- **Proxy Host**: 21.0.0.23:15004

## Files Modified

- `/home/user/cooja/gradle.properties` - Created to document the issue
- `/tmp/network-permissions-analysis.md` - Detailed technical analysis
- `/home/user/cooja/NETWORK-PERMISSIONS-FINDINGS.md` - This document
