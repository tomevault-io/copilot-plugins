## com-fs-email-template-model

> To regenerate the MODEL.md file, provide this instruction to Cursor:

# Project Rules for com-fs-email-template-model

## Regenerating MODEL.md Documentation

To regenerate the MODEL.md file, provide this instruction to Cursor:

**Instruction:**
Generate comprehensive documentation for all Protocol Buffer (`.proto`) files in this repository and write it to MODEL.md. The documentation should:

1. **Structure:**
   - Start with a title: "# Email Template Documentation"
   - Include a table of contents listing all proto files
   - Provide an overview section describing the email template model's purpose and key features
   - Document each proto file in separate sections

2. **For each proto file section, include:**
   - Package information (package name and Go package path)
   - Overview of the file's purpose
   - Complete documentation of all messages with:
     - Field tables showing: Field name, Type, Required/Optional, Description
     - Use cases for each message
     - Important notes and constraints
   - Complete documentation of all enums with:
     - Value tables showing: Value name, Number, Description
   - Complete documentation of all services (for gRPC files) with:
     - Method signatures
     - Request/Response parameters
     - Behavior descriptions
     - Use cases
     - Error conditions

3. **Proto files to document:**
   - emailtemplate.proto - Emailtemplate model and definitions
   
4. **Documentation style:**
   - Use clear, professional language
   - Include field descriptions that explain business logic and usage
   - Reference related messages and enums
   - Note any important constraints or requirements
   - Include use case examples where helpful
   - Extract information from proto file comments where available

5. **End the document with:**
   - A note about version correspondence
   - Support section with references to README.md and other resources

**Quick command to regenerate:**
Ask Cursor: "Generate MODEL.md documentation following the instructions in .cursorrules"

**Note:** The existing MODEL.md file can be used as a style reference, but the documentation should be regenerated from the current proto files to ensure accuracy.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sologenic) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
