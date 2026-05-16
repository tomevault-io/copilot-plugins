## rebuilt-2026

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FRC Team 1350 (La Salle Robotics) 2026 robot code. Command-based Java robot with swerve drive, shooter, turret auto-aiming, intake, climber, and vision-based localization.

## Competition Rules

Competition rules are available at: https://firstfrc.blob.core.windows.net/frc2026/Manual/2026GameManual.pdf

## Build & Deploy Commands

```bash
./gradlew build                  # Build the project
./gradlew deploy                 # Deploy to RoboRIO
./gradlew test                   # Run JUnit 5 tests
./gradlew simulateJava           # Run robot simulation
```

Gradle wrapper (v8.11) with GradleRIO 2026.2.1. Java 17.

## Architecture

**Command-Based Robot** using WPILib's command framework:
- `Robot.java` — TimedRobot entry point, schedules auto commands, configures Limelight
- `RobotContainer.java` — Instantiates all subsystems, binds Xbox controller buttons to commands
- `Telemetry.java` — Publishes pose/velocity data to NetworkTables

**Subsystems** (in `frc.robot.subsystems`):
- `CommandSwerveDrivetrain` — Phoenix6 swerve drive (4 modules), Pigeon2 IMU, Limelight pose estimation (MegaTag2)
- `ShooterPowerSubsystem` — Dual REV SparkFlex motors (IDs 16, 17)
- `ShooterAimSubsystem` — TalonFX (ID 14), calculates angle to hub using pose estimator
- `TurretSubsystem` — TalonFX (ID 13), auto-aims toward hub coordinates (4.625594, 4.03479)m
- `IntakeLevelSubsystem` — TalonFX (ID 17), MotionMagic position control
- `IntakeWheelSubsystem` — TalonFX (ID 18), velocity control
- `ThroatAndIndexerSubsystem` — TalonFX (ID 32), ball feed to shooter
- `ClimberSubsystem` — TalonFX (ID 15), MotionMagic up/down

**Commands** (in `frc.robot.commands`):
- `AlignToHub` — Vision-based alignment using Limelight PID control
- `GetTurretToHub` — Utility for turret-to-hub bearing calculation

**Autonomous:** PathPlanner (v2026.1.2) with paths in `src/main/deploy/pathplanner/`. Auto selection via SendableChooser in RobotContainer.

## Key Vendor Libraries

- **Phoenix6 (CTRE)** — TalonFX motors, Pigeon2 gyro, swerve drive
- **REVLib** — SparkFlex motors (shooter)
- **PathPlannerLib** — Autonomous path following

## Important Conventions

- Swerve constants are generated in `frc.robot.generated.TunerConstants` — supports dual bot configs (ThorBot/LokiBot)
- TalonFX subsystems follow a consistent MotionMagic tuning pattern: kP=4.8, kI=0, kD=0.1, kV=0.12, kA=0.01, kS=0.25
- Limelight camera name: `"limelight-fifteen"`
- Hub target height: 2.64m, bot height: 0.4495m
- Most simple commands use `Commands.runOnce()` pattern

---
> Source: [RambotsTeam1350/REBUILT-2026](https://github.com/RambotsTeam1350/REBUILT-2026) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-16 -->
