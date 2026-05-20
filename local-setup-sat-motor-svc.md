# Local SSH Tunnel Setup & Build Guide for `sat-motor-svc`

This document outlines the detailed configuration changes and steps required to run the `sat-motor-svc` service locally and connect it to the remote AWS RDS Development database via a secure SSH Tunnel (Jump Host / Bastion).

---

## 🛠️ Detailed Configuration Changes Made

### 1. Update `application-local.properties`
Configured the local properties inside `src/main/resources/application-local.properties` to route database traffic through the SSH Tunnel local port `15432`:

```properties
# AWS Cloud Configuration
spring.cloud.aws.region.static=ap-southeast-1
spring.cloud.aws.stack.auto=false

# Primary Datasource via SSH Tunnel Port 15432
spring.datasource.primary.url=jdbc:postgresql://localhost:15432/sat?currentSchema=sat
spring.datasource.primary.username=DBBEND_SAT
spring.datasource.primary.password=4nV3y!kCJBYgA3KyeJ7G
spring.datasource.primary.driver-class-name=org.postgresql.Driver

# Secondary Datasource via SSH Tunnel Port 15432
spring.datasource.secondary.url=jdbc:postgresql://localhost:15432/sat?currentSchema=sat
spring.datasource.secondary.username=DBBEND_SAT
spring.datasource.secondary.password=4nV3y!kCJBYgA3KyeJ7G
spring.datasource.secondary.driver-class-name=org.postgresql.Driver
```

---

### 2. Add `@DependsOn("sshTunnelConfig")` to `DataSourceConfig`
Modified `src/main/java/com/allianz/sat/motor/config/DataSourceConfig.java` to make sure the SSH Tunnel bean is fully initialized before Spring Boot attempts to configure the DataSources:

```java
package com.allianz.sat.motor.config;

// ... other imports
import org.springframework.context.annotation.DependsOn;

@Configuration
@DependsOn("sshTunnelConfig") // Ensure SSH Tunnel starts first
public class DataSourceConfig {
    // ... bean declarations
}
```

---

### 3. Add `jsch` dependency to `pom.xml`
Added the JSch library dependency (version `0.1.55`) inside the `pom.xml` of the `sat-motor-svc` module to programmatically establish the SSH tunnel:

```xml
<dependency>
    <groupId>com.jcraft</groupId>
    <artifactId>jsch</artifactId>
    <version>0.1.55</version>
</dependency>
```

---

### 4. Create new class `SshTunnelConfig`
Created `SshTunnelConfig.java` inside package `com.allianz.sat.motor.config` to automatically start and stop the SSH tunnel when the application starts or stops:

```java
package com.allianz.sat.motor.config;

import com.jcraft.jsch.JSch;
import com.jcraft.jsch.Session;
import jakarta.annotation.PostConstruct;
import jakarta.annotation.PreDestroy;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SshTunnelConfig {

    private Session session;

    @PostConstruct
    public void startTunnel() {
        try {
            String sshUser = "swo_alphasat";
            String sshHost = "44.113.141.105";
            String sshPassword = "Allianz@2026";

            String remoteHost = "sat-dev-pgdb-002.cluster-ch2s4y4socw2.ap-southeast-1.rds.amazonaws.com";

            int localPort = 15432; // tránh conflict

            JSch jsch = new JSch();
            session = jsch.getSession(sshUser, sshHost, 22);
            session.setPassword(sshPassword);

            session.setConfig("StrictHostKeyChecking", "no");

            session.connect(10_000);

            session.setPortForwardingL(localPort, remoteHost, 5432);

            System.out.println("✅ SSH tunnel opened at localhost:" + localPort);

        } catch (Exception e) {
            throw new RuntimeException("❌ SSH tunnel failed", e);
        }
    }

    @PreDestroy
    public void stopTunnel() {
        if (session != null && session.isConnected()) {
            session.disconnect();
        }
    }
}
```

---

## 🚀 How to Run the Application Locally

### Step 1: Build the project and fetch new dependencies
Open your terminal in the `sat-motor-svc` directory and run:
```bash
mvn clean install -DskipTests
```

### Step 2: Run the application with the `local` profile active
* **Using Command Line / Terminal:**
  ```bash
  mvn spring-boot:run -Dspring-boot.run.profiles=local
  ```
* **Using your IDE (IntelliJ IDEA / Eclipse / VS Code):**
  1. Open your application's **Run/Debug Configuration**.
  2. Add the following JVM Option: `-Dspring.profiles.active=local`
  3. Or add the environment variable: `SPRING_PROFILES_ACTIVE=local`
  4. Start the application in **Run** or **Debug** mode.

### Step 3: Verify the connection
Monitor the console log output on startup. You should see the successful connection log:
```text
✅ SSH tunnel opened at localhost:15432
```
Followed by standard HikariCP connection pool initialized logs pointing to `jdbc:postgresql://localhost:15432/sat`.

---

## ⚠️ Troubleshooting

| Issue | Potential Cause | Resolution |
| :--- | :--- | :--- |
| **Address already in use / Port conflict** | Local port `15432` is already being used by another process or another SSH tunnel connection. | 1. Close any other running terminal tab or process using this port.<br>2. Or change the port (e.g., to `15433`) in both `SshTunnelConfig.java` and `application-local.properties`. |
| **Connection Timeout** | Network firewall or corporate VPN blocking outgoing port `22` traffic to the SSH Host `44.113.141.105`. | Verify your network connection or contact network administrators to ensure outbound connection to `44.113.141.105:22` is permitted. |
