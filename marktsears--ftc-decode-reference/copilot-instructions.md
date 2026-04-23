## ftc-decode-reference

> Code conventions and naming standards for Java files


# Code Conventions

## Package Structure
```
teamcode/
├── common/           # Robot class, Subsystem base, shared utilities
├── opmodes/          # All OpModes (auto/, teleop/, test/)
├── pedroPathing/     # Pedro Pathing configuration
├── subsystems/       # All subsystems (drive, turret, shooter, vision, etc.)
└── utils/            # Utility classes
```

## Naming Conventions
- OpModes: `Auto_[Alliance]_[Name].java` or `[Name]TeleOp.java`
- Constants: `[Subsystem]Constants.java` with `@Config` annotation for FTC Dashboard
- Subsystems: Extend `Subsystem` base class, implement `periodic()` and `writeTelemetry()`

## Hardware Names (Robot Configuration)
- Motors: `leftFront`, `leftRear`, `rightFront`, `rightRear`, `turret`, `flywheel`, `intakeMotor`
- Servos: `hood`, `gate`, `indexer`
- Sensors: `limelight`, `pinpoint`

## Subsystem Pattern
All subsystems should:
1. Extend the `Subsystem` base class
2. Implement `periodic()` for state updates
3. Implement `writeTelemetry()` for debugging
4. Use constants from a companion `*Constants.java` file
5. Annotate constants with `@Config` for FTC Dashboard tuning

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/marktsears) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
