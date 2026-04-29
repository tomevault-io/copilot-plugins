## digital-waste-tracking-perf-test

> Payload Management Rules


# Payload Management Rules

## Payload Generation Structure
- Use JSR223PreProcessor with external Groovy scripts for dynamic payload generation
- Store Groovy scripts in `scripts/` directory at repository root
- Use `testType` parameter for test-specific customization
- Generate JSON payloads programmatically with variable substitution

## Payload Content Rules
- Include all required API fields
- Use JMeter variables for dynamic data: `${__threadNum}`, `${__Random()}`, etc.
- Use `${testType}` parameter for test-specific descriptions
- Include realistic but performance-focused test data
- Avoid hardcoded values that don't support load testing

## Variable Usage
- `${organisationApiId}` - Organization API ID
- `${testType}` - Test type (Baseline, Load, Stress, Spike, Basic)
- `${__time(yyyy-MM-dd'T'HH:mm:ss.SSS'Z')}` - Current timestamp
- `${__threadNum}` - Thread number for unique identification
- `${__Random(min,max)}` - Random number generation
- `${wasteTrackingId}` - Waste tracking ID (for updates)
- `${postPayload}` - JSON payload for POST/create operations
- `${putPayload}` - JSON payload for PUT/update operations
- `${__P(global_access_token)}` - OAuth2 access token from shared authentication
- `${__P(environment)}` - Environment-specific configuration

## JMX Integration
- Use JSR223PreProcessor with external Groovy scripts for dynamic payload generation
- Use HTTP Arguments (not postBodyRaw) for complex JSON payloads with variable substitution
- Set `testType` parameter in User Defined Variables section
- Use `${postPayload}` for POST/create operations and `${putPayload}` for PUT/update operations in HTTP Arguments
- Ensure Groovy scripts are accessible from JMeter execution context
- Include `get_accessToken.groovy` JSR223PreProcessor for shared authentication
- Use HeaderManager for Content-Type and Authorization headers
- Use `${__P(global_access_token)}` for authorization header value

## Maintenance
- Update Groovy scripts when API changes
- Test payload changes with all test types
- Keep payload generation simple and focused on performance testing
- Avoid complex business logic in payload generation
- Use `vars.put("postPayload", postPayload)` for create operations and `vars.put("putPayload", putPayload)` for update operations
- Ensure proper error handling in Groovy scripts

## Groovy Script Patterns
- `create_waste_movement_payload.groovy` - Sets `postPayload` variable for create POST operations
- `update_waste_movement_payload.groovy` - Sets `putPayload` variable for update PUT operations
- `payloads/bulk_create_waste_movement_payload.groovy` - Sets `bulkPostPayload` variable for bulk create POST operations
- `payloads/bulk_update_waste_movement_payload.groovy` - Sets `bulkPutPayload` variable for bulk update PUT operations
- Scripts should use `vars.put("variableName", value)` to make variables available to JMeter
- Scripts should access test properties via `props.get("propertyName")` for environment configuration
- Scripts should access test variables via `vars.get("variableName")` for test-specific data

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DEFRA) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
