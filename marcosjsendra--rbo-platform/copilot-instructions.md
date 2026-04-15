## rbo-platform

> - **Always use the correct base URL**: `https://remax-cca.com/api/v1/`


## REI API CCA Integration Rules

### API URL Structure

- **Always use the correct base URL**: `https://remax-cca.com/api/v1/`
- **Never use `/reiapi/` path** despite what older documentation states
- **Use exact endpoint names**:
  - Properties List: `GetProperties/take/{take}/skip/{skip}`
  - Property Details: `GetPropertyDetails/{listingid}`
  - Associates List: `associates/take/{take}/skip/{skip}`
  - Associate Details: `associates/{associateid}`

### Authentication & Token Management

- **Token endpoint**: Always use `https://remax-cca.com/api/v1/oauth/token`
- **Parameter casing**: Maintain exact casing (e.g., `integratorID`)
- **Token refresh**: Implement 30-minute refresh buffer before expiration
- **Error handling**: Always implement proper token expiration and network error handling

### Data Verification

- **Always verify data integrity** between REI API CCA and Supabase
- **Log all API interactions** for debugging and monitoring
- **Document any discrepancies** in `/docs/Errors/reported/`
- **Create fix reports** in `/docs/Errors/fixedReport/`

### Implementation Guidelines

1. **Before making API calls**:

   - Verify endpoint URL structure
   - Check token validity
   - Prepare error handling

2. **During development**:

   - Use real data, never mock data
   - Log all API responses
   - Document any deviations from specifications

3. **After implementation**:
   - Update relevant documentation
   - Create progress reports
   - Update URL rules if new endpoints are discovered

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/marcosjsendra) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
