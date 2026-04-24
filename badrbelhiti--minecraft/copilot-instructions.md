## minecraft

> A Kotlin-based Minecraft server network with AWS CDK infrastructure management.

# Nerds @ ATX Minecraft Network

A Kotlin-based Minecraft server network with AWS CDK infrastructure management.

> **Note to Claude Code**: This file should be continuously updated with best practices, patterns, and architectural decisions established during development. When new patterns emerge or practices are refined, update this document to reflect the current state of the project.

## Project Structure

This is a Gradle monorepo with a flat directory structure:

```
Minecraft/
├── settings.gradle.kts           # Defines all modules
├── build.gradle.kts              # Root build config with common settings
├── gradle.properties             # Gradle configuration
├── gradle/
│   └── libs.versions.toml        # Centralized dependency versions
├── MinecraftCdk/                 # AWS CDK infrastructure as code
│   ├── build.gradle.kts
│   └── src/main/kotlin/dev/nerdsatx/cdk/
├── BungeeCordProxy/              # BungeeCord proxy plugin
│   ├── build.gradle.kts
│   └── src/main/kotlin/dev/nerdsatx/proxy/
├── SurvivalServer/               # Survival server configuration
│   ├── build.gradle.kts
│   └── server/
├── SkyBlockServer/               # SkyBlock server configuration
│   ├── build.gradle.kts
│   └── server/
├── SurvivalPlugin/               # Survival game plugin
│   ├── build.gradle.kts
│   └── src/main/kotlin/dev/nerdsatx/survival/
├── SkyBlockPlugin/               # SkyBlock game plugin
│   ├── build.gradle.kts
│   └── src/main/kotlin/dev/nerdsatx/skyblock/
├── SharedCore/                   # Shared utilities and models
│   ├── build.gradle.kts
│   └── src/main/kotlin/dev/nerdsatx/shared/
└── Documentation/                # Documentation and diagrams
    ├── build.gradle.kts
    ├── diagrams/                 # Generated PNG diagrams
    └── src/main/plantuml/        # PlantUML source files
```

## Package Structure

All code uses the base package: `dev.nerdsatx`

- `dev.nerdsatx.shared` - Shared utilities, models, database code
- `dev.nerdsatx.survival` - Survival plugin code
- `dev.nerdsatx.skyblock` - SkyBlock plugin code
- `dev.nerdsatx.proxy` - BungeeCord proxy plugin
- `dev.nerdsatx.cdk` - AWS infrastructure code

## Module Dependencies

```
SurvivalPlugin  ──→  SharedCore
SkyBlockPlugin  ──→  SharedCore
BungeeCordProxy ──→  SharedCore
MinecraftCdk    (independent)
SurvivalServer  (independent)
SkyBlockServer  (independent)
Documentation   (independent)
```

## Technology Stack

- **Language**: Kotlin 1.9.22
- **Build Tool**: Gradle with Kotlin DSL
- **Minecraft Platform**: Paper API (1.20.4) for game servers
- **Proxy**: BungeeCord API
- **Infrastructure**: AWS CDK
- **Plugin Packaging**: Shadow plugin for fat JARs

## Key Gradle Patterns

### Version Catalog (`gradle/libs.versions.toml`)

We use Gradle's version catalog feature for centralized dependency management:

```toml
[versions]
kotlin = "1.9.22"
paper = "1.20.4-R0.1-SNAPSHOT"

[libraries]
paper-api = { module = "io.papermc.paper:paper-api", version.ref = "paper" }

[plugins]
kotlin-jvm = { id = "org.jetbrains.kotlin.jvm", version.ref = "kotlin" }
```

Access in build files: `libs.paper.api` or `alias(libs.plugins.kotlin.jvm)`

### Common Build Patterns

**Plugin Module Pattern** (SurvivalPlugin, SkyBlockPlugin):
```kotlin
plugins {
    alias(libs.plugins.kotlin.jvm)
    alias(libs.plugins.shadow)
}

dependencies {
    implementation(project(":SharedCore"))
    compileOnly(libs.paper.api)
    implementation(libs.kotlin.stdlib)
}

tasks {
    shadowJar {
        archiveFileName.set("PluginName.jar")
        relocate("kotlin", "dev.nerdsatx.pluginname.shaded.kotlin")
    }
    build {
        dependsOn(shadowJar)
    }
}
```

**Proxy Module Pattern** (BungeeCordProxy):
```kotlin
plugins {
    alias(libs.plugins.kotlin.jvm)
    alias(libs.plugins.shadow)
}

dependencies {
    implementation(project(":SharedCore"))
    compileOnly(libs.bungeecord.api)
    implementation(libs.kotlin.stdlib)
}
```

**CDK Module Pattern** (MinecraftCdk):
```kotlin
plugins {
    alias(libs.plugins.kotlin.jvm)
    application
}

application {
    mainClass.set("dev.nerdsatx.cdk.MinecraftCdkAppKt")
}

dependencies {
    implementation(libs.aws.cdk.lib)
    implementation(libs.aws.constructs)
}
```

**Documentation Module Pattern** (Documentation):
```kotlin
plugins {
    base  // Required for IntelliJ to recognize as a Gradle module
}

// Configuration for PlantUML dependency
val plantuml by configurations.creating

dependencies {
    plantuml("net.sourceforge.plantuml:plantuml:1.2024.3")
}

val plantumlSourceDir = file("src/main/plantuml")
val plantumlOutputDir = file("diagrams")

tasks {
    // Generate PlantUML diagrams
    val generateDiagrams by registering(JavaExec::class) {
        description = "Generate PNG diagrams from PlantUML files"
        group = "documentation"

        mainClass.set("net.sourceforge.plantuml.Run")
        classpath = plantuml
        jvmArgs("-DPLANTUML_LIMIT_SIZE=8192")
        args(
            "-v",
            "-tpng",
            "-SdefaultRenderer=smetana",  // Use built-in Smetana renderer
            "-o", plantumlOutputDir.absolutePath,
            plantumlSourceDir.absolutePath
        )

        inputs.dir(plantumlSourceDir)
        outputs.dir(plantumlOutputDir)

        doFirst {
            plantumlOutputDir.mkdirs()
        }
    }

    build {
        dependsOn(generateDiagrams)
    }
}
```

**Key Points**:
- PlantUML diagrams are stored in `src/main/plantuml/`
- Generated PNGs are output to `diagrams/` within the Documentation module
- Diagrams are automatically regenerated on `./gradlew build`
- Uses Smetana renderer (built-in) - no Graphviz installation required
- Add `!pragma layout smetana` at the top of .puml files for best results
- The module is independent and has no dependencies on other modules
- Diagrams can be referenced in README.md via `Documentation/diagrams/filename.png`

## Common Gradle Commands

```bash
# Build entire project
./gradlew build

# Build specific plugin
./gradlew :SurvivalPlugin:build
./gradlew :SurvivalPlugin:shadowJar

# Build all plugins
./gradlew :SurvivalPlugin:shadowJar :SkyBlockPlugin:shadowJar :BungeeCordProxy:shadowJar

# Deploy infrastructure
./gradlew :MinecraftCdk:run

# Generate documentation diagrams
./gradlew :Documentation:build

# Clean and rebuild
./gradlew clean build
```

## IntelliJ IDEA Setup

1. **Import Project**: File → Open → Select root `build.gradle.kts`
2. **Auto-import**: Settings → Build, Execution, Deployment → Build Tools → Gradle
    - Enable "Reload changes automatically"
3. **Build Delegation**: Build and run using "Gradle" (not IntelliJ)
4. **JVM Version**: Ensure Gradle JVM is set to Java 17+

### Module Registration

Modules are automatically registered in IntelliJ via `.idea/gradle.xml`. When adding a new module:

1. Add the module to `settings.gradle.kts` (required for Gradle)
2. Add the module to `.idea/gradle.xml` in the `<modules>` section (for IntelliJ recognition)

Example entry in `.idea/gradle.xml`:
```xml
<option name="modules">
  <set>
    <option value="$PROJECT_DIR$" />
    <option value="$PROJECT_DIR$/Documentation" />
    <option value="$PROJECT_DIR$/SharedCore" />
    <!-- ... other modules ... -->
  </set>
</option>
```

**Current registered modules**:
- Root project
- BungeeCordServer
- Documentation
- MinecraftCdk
- SharedCore
- SurvivalPlugin
- SurvivalServer

After updating `.idea/gradle.xml`, IntelliJ will automatically recognize the modules on project reload.

## Plugin Development Workflow (Docker-Integrated)

1. Make changes to plugin code in `src/main/kotlin/`
2. Update `plugin.yml` or `bungee.yml` in `src/main/resources/` if needed
3. Build with `./gradlew build` (rebuilds plugin JAR AND Docker image automatically)
4. Restart containers with `docker-compose up -d`
5. Test your changes

**The Gradle build automatically**:
- Compiles Kotlin code
- Creates shadow JAR with relocated dependencies
- Rebuilds the Docker image with the new plugin
- No manual file copying needed!

## Shared Code Guidelines

- Put common utilities in `SharedCore`
- Database models and DAOs go in `SharedCore/src/main/kotlin/dev/nerdsatx/shared/database/`
- Shared data classes go in `SharedCore/src/main/kotlin/dev/nerdsatx/shared/models/`
- Use `compileOnly` for Paper/Bungee APIs in SharedCore to avoid conflicts

## Shadow JAR & Relocation

Plugins shade and relocate dependencies to avoid conflicts:

```kotlin
shadowJar {
    relocate("kotlin", "dev.nerdsatx.pluginname.shaded.kotlin")
    relocate("org.jetbrains.exposed", "dev.nerdsatx.pluginname.shaded.exposed")
    relocate("com.zaxxer.hikari", "dev.nerdsatx.pluginname.shaded.hikari")
}
```

## Docker Integration

### Gradle + Docker Build Integration

**Critical Pattern**: Docker images are built automatically as part of the Gradle build process. This ensures that plugin code changes are always included in the Docker images without manual intervention.

**Plugin Module Docker Build Pattern**:
```kotlin
tasks {
    shadowJar {
        archiveFileName.set("PluginName.jar")
        relocate("kotlin", "dev.nerdsatx.pluginname.shaded.kotlin")
        minimize()
    }

    // Auto-build Docker image after building plugin
    val buildDockerImage by registering(Exec::class) {
        description = "Build the Docker image with updated plugin"
        group = "docker"

        workingDir = rootProject.projectDir
        commandLine("docker-compose", "build", "service-name")

        dependsOn(shadowJar)
    }

    build {
        dependsOn(shadowJar)
        finalizedBy(buildDockerImage)  // Auto-rebuild Docker image
    }
}
```

This pattern is implemented in:
- `SurvivalPlugin/build.gradle.kts` - builds `survival` Docker image
- `BungeeCordServer/build.gradle.kts` - builds `bungeecord` Docker image

### Docker Best Practices

**1. Bake Everything into Images**
- All plugins, configurations, and static assets are copied into Docker images at build time
- NO volume mounts for plugins or configs (only for persistent data)
- Images are self-contained and production-ready

**2. Dockerfile Patterns**

**Server Module Dockerfile** (e.g., SurvivalServer):
```dockerfile
FROM itzg/minecraft-server:latest

# All configuration via ENV in Dockerfile, not docker-compose
ENV EULA=TRUE \
    TYPE=PAPER \
    VERSION=1.20.4 \
    MEMORY=2G \
    SERVER_NAME="Survival Server" \
    ONLINE_MODE=FALSE \
    SPIGOT_BUNGEECORD=true

# Copy plugin into image
COPY SurvivalPlugin/build/libs/SurvivalPlugin.jar /plugins/

EXPOSE 25565
```

**Proxy Module Dockerfile** (BungeeCordServer):
```dockerfile
FROM itzg/bungeecord:latest

ENV TYPE=BUNGEECORD \
    MEMORY=1G

# Copy plugin
COPY BungeeCordServer/build/libs/BungeeCordServer.jar /server/plugins/

# CRITICAL: itzg/bungeecord uses /config/config.yml as source
# It copies /config/config.yml to /server/config.yml on startup
COPY BungeeCordServer/config.yml /config/config.yml

EXPOSE 25577
```

**3. docker-compose.yml Minimalism**

Keep docker-compose.yml minimal - it's for orchestration only, not configuration:

```yaml
version: '3.8'

services:
  # Use simple service names (not survival-server)
  survival:
    build:
      context: .
      dockerfile: SurvivalServer/Dockerfile
    container_name: minecraft-survival
    volumes:
      # ONLY persist data that needs persistence
      - survival-data:/data
    networks:
      - minecraft-network
    restart: unless-stopped

  bungeecord:
    build:
      context: .
      dockerfile: BungeeCordServer/Dockerfile
    container_name: minecraft-bungeecord
    ports:
      - "25577:25577"  # Only expose proxy to host
    networks:
      - minecraft-network
    depends_on:
      - survival
    restart: unless-stopped

networks:
  minecraft-network:
    driver: bridge

volumes:
  survival-data:  # Persists world data
```

**Key Principles**:
- NO environment variables in docker-compose (they're in Dockerfiles)
- NO plugin volume mounts (plugins are baked into images)
- Service names are simple and match DNS hostnames used in configs
- Only expose ports that need external access (BungeeCord proxy)

**4. Docker Networking & DNS**

**Service Name = DNS Hostname**:
- Service name `survival` creates DNS alias `survival`
- BungeeCord config uses: `address: survival:25565`
- Docker's built-in DNS resolves service names automatically

**Network Aliases**:
```bash
# Service "survival" gets these aliases:
- survival              # Service name (PRIMARY - use this)
- minecraft-survival    # Container name
- <container-id>        # Container ID prefix
```

**5. itzg Image Configuration**

**itzg/minecraft-server** (Paper servers):
- Configure via ENV variables in Dockerfile
- Plugin directory: `/plugins/`
- Data directory: `/data/` (mount as volume for persistence)

**itzg/bungeecord** (Proxy):
- Configure via ENV variables in Dockerfile
- Plugin directory: `/server/plugins/`
- Config source: `/config/config.yml` (copied to `/server/config.yml` on startup)
- **CRITICAL**: Always copy config to `/config/config.yml`, NOT `/server/config.yml`

## Development Workflow

### Standard Development Cycle

```bash
# 1. Make code changes to plugins
# 2. Build everything (plugins AND Docker images)
./gradlew build

# 3. Start/restart containers
docker-compose up -d

# Or use the convenience script:
./start-local.sh
```

### start-local.sh Pattern

```bash
#!/bin/bash
set -e

echo "🎮 Starting Nerds @ ATX Minecraft Network..."

# Check Docker is running
if ! docker info > /dev/null 2>&1; then
    echo "❌ Error: Docker is not running."
    exit 1
fi

# Build plugins (this also builds Docker images via Gradle tasks)
echo "📦 Building plugins..."
./gradlew build -q

echo "✅ Plugins built and Docker images rebuilt"

# Start containers
echo "🚀 Starting Docker containers..."
docker-compose up -d

echo "✅ Servers started!"
echo "🎮 Connect to: localhost:25577"
```

### When Plugin Code Changes

**The integrated build handles everything**:
1. `./gradlew build` compiles plugin
2. Gradle's `buildDockerImage` task rebuilds Docker image
3. `docker-compose up -d` recreates containers with new image

**No manual steps needed** - the Docker image is always up-to-date with your code.

### Local vs Production Similarity

The setup is designed for **maximum similarity** between local dev and production (ECS Fargate):
- Same Docker images used locally and in production
- No environment-specific scripts or configurations
- All config baked into images
- To override config in production: use ECS task definition environment variables

### Common Issues & Solutions

**Issue**: Plugin changes not appearing
**Solution**: Run `./gradlew build` (not just `docker-compose up -d`) - Gradle rebuilds images

**Issue**: "network has active endpoints" error
**Solution**:
```bash
docker stop minecraft-bungeecord minecraft-survival
docker rm minecraft-bungeecord minecraft-survival
docker network rm minecraft_minecraft-network
docker-compose up -d
```

**Issue**: BungeeCord can't find survival server (UnknownHostException)
**Solution**: Check that docker-compose service names match hostnames in configs

**Issue**: BungeeCord using default config instead of custom config
**Solution**: Ensure config is copied to `/config/config.yml` in Dockerfile, NOT `/server/config.yml`

## AWS Infrastructure

The `MinecraftCdk` module contains CDK for:
- ECS Fargate for game servers and bungee cord proxy
- VPC and networking configuration
- Security groups and IAM roles
- S3 buckets for backups
- CloudWatch monitoring

Minimize cross-stack dependencies as much as possible. All
game server and bungee cord servers should be in one stack.
There can be a separate database stack once we add one. There
can be a separate website stack once we add one.

## Development Guidelines

- **Kotlin Style**: Follow official Kotlin coding conventions
- **JVM Target**: Java 17
- **Null Safety**: Leverage Kotlin's null safety features
- **Coroutines**: Use for async operations where appropriate
- **API Compatibility**: Target Paper API, not Bukkit/Spigot directly
- **Testing**: Add tests in `src/test/kotlin/` (JUnit 5)

## Repositories

Maven repositories are configured in root `build.gradle.kts`:
- Maven Central (default dependencies)
- Paper Maven (Paper API)
- Sonatype OSS (BungeeCord API)

## Important Notes

- Each module is independent and can be built separately
- SharedCore must be built before dependent plugins
- Use `:ModuleName` syntax for inter-module dependencies
- Shadow plugin creates fat JARs with all dependencies included
- Always relocate shaded dependencies to avoid classpath conflicts

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BadrBelhiti) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
