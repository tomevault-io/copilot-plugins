## kinobi-vercel

> Defines standards for creating AI-optimized implementation plans, ensuring clear context, unambiguous directives, and traceable dependencies.

## Implementation Planning Framework

1. Baseline Knowledge
   - Existing Service Interfaces
     ```typescript
     // Required existing interfaces with types
     interface ExistingService {
       method(param: Type): ReturnType;
     }
     ```
   - Implementation Dependencies
     ```
     src/path/to/file.ts        // Purpose/modification
     src/path/to/another.ts     // Purpose/modification
     ```
   - Required File Modifications
     - Clear file paths
     - Modification purpose
     - Integration points

2. Type Definitions
   - New Types
     ```typescript
     interface NewType {
       field: Type;  // Purpose
     }
     ```
   - Extended Types
     ```typescript
     interface ExtendedType extends BaseType {
       newField: Type;  // Purpose
     }
     ```
   - Validation Types
     ```typescript
     interface ValidationResult {
       isValid: boolean;
       errors: Error[];
     }
     ```

3. Implementation Order
   - Dependency Chain
     ```
     Core Service A
     └─> Dependent Service B
         └─> Feature Implementation
     ```
   - Critical Path
     - Required predecessor tasks
     - Blocking dependencies
     - Integration requirements

4. Integration Points
   - Service Integration
     ```typescript
     class ServiceA {
       // Integration method
       methodB(): void;
     }
     ```
   - Event Flow
     ```
     ServiceA -> EventSystem -> ServiceB
     ```
   - Error Handling
     ```typescript
     try {
       // Critical operation
     } catch (error) {
       // Recovery procedure
     }
     ```

5. Success Criteria
   - Functionality Requirements
     - Core features
     - Edge cases
     - Error scenarios
   - Performance Metrics
     - Response times
     - Resource usage
     - Scalability
   - Reliability Standards
     - Error rates
     - Recovery time
     - Data consistency

6. Best Practices
   - Clear Interface Definitions
   - Explicit Dependencies
   - Implementation Sequence
   - Error Recovery Paths
   - Performance Constraints
   - Integration Tests

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/omniharmonic) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
