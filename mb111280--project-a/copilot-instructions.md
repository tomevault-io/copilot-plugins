## project-a

> Rule to help create product breifs

# Product Brief Rule

This rule helps generate and validate product briefs according to Fourth's standard template.

## Required Information
Before proceeding with document generation, the following information is required:

### Minimum Required Information
- Product name
- Basic product description

### Additional Helpful Information
- Target market or user base
- Key problems the product aims to solve
- Specific requirements or constraints
- Timeline or launch expectations
- Existing documentation (if available)

## Rule Type: Document Generation & Validation

## Purpose
- Generate new product briefs following the standard template
- Validate existing product briefs against the template structure
- Highlight missing sections for consideration

## Template Structure
The following sections are recommended for a complete product brief. While not all sections are mandatory, missing sections will be highlighted for review:

```markdown
**Product Brief: [Product/Feature Name]**

### Product Overview
- Product Name: [Name]
- Product Category: [Category]
- Target Launch Date: [Date]
- Product Owner: [Name]
- Team: [Team Members]

### Executive Summary
[2-3 paragraphs summarizing the product, its value proposition, target market, and key objectives]

### Problem Statement
- What problem does this product solve?
- Who experiences this problem?
- How significant is this problem?
- What are the current alternatives/workarounds?

### Target Market
- Primary User Persona:
  - Demographics
  - Behaviors
  - Pain Points
  - Goals
- Market Size (TAM/SAM/SOM)
- Market Trends
- Competitive Landscape

### Product Vision
- Core Value Proposition
- Key Differentiators
- Strategic Alignment
- Long-term Vision

### Success Metrics
- Key Performance Indicators (KPIs)
- Success Criteria
- Measurement Methods
- Target Metrics

### Features & Requirements
#### Must Have (P0)
- Feature 1
  - Description
  - User Story
  - Acceptance Criteria
- Feature 2...

#### Should Have (P1)
- Feature 1...
- Feature 2...

#### Nice to Have (P2)
- Feature 1...
- Feature 2...

### Technical Requirements
- Platform/Technology Stack
- Integration Requirements
- Security Requirements
- Performance Requirements
- Scalability Requirements

### Dependencies
- Internal Dependencies
- External Dependencies
- Technical Dependencies
- Resource Dependencies

### Timeline & Milestones
- Phase 1: [Description & Date]
- Phase 2: [Description & Date]
- Phase 3: [Description & Date]
- Launch: [Date]

### Resources & Budget
- Team Resources Required
- Budget Breakdown
- Additional Resources Needed

### Risks & Mitigation
- Risk 1
  - Impact
  - Probability
  - Mitigation Strategy
- Risk 2...

### Go-to-Market Strategy
- Launch Strategy
- Marketing Plan
- Distribution Channels
- Pricing Strategy

### Future Considerations
- Product Roadmap
- Future Features
- Scalability Plans
- Maintenance Plans

### Approvals
- Product Owner: [Name & Date]
- Engineering Lead: [Name & Date]
- Design Lead: [Name & Date]
- Other Stakeholders: [Names & Dates]

### Open Questions & Assumptions
- List any critical questions that still need answers
- List any key assumptions made while drafting this brief
```

## Generation Process
1. The AI will first request the minimum required information
2. If minimum information is not provided, the AI will not proceed with generation
3. Once minimum information is provided, the AI will:
   - Request any additional helpful information
   - Make reasonable assumptions for missing information
   - Clearly mark any assumptions made
   - Generate the document following the template structure

## Generation Prompt
When generating a new product brief, the AI must follow this process:

1. Initial Prompt:
```
You are an expert Product Owner Assistant AI. Your goal is to help create a comprehensive product brief following Fourth's standard template.

To proceed, I need the following minimum information:
- Product name
- Basic product description

Please provide this information before we continue.
```

2. After receiving minimum information:
```
Thank you for providing the basic information. To create a more comprehensive brief, please provide any of the following additional information (if available):
- Target market or user base
- Key problems the product aims to solve
- Specific requirements or constraints
- Timeline or launch expectations
- Any existing documentation

If you don't have this information, I'll proceed with reasonable assumptions, which I'll clearly mark in the document.
```

3. After receiving additional information (or if none is provided):
```
Based on the information provided, I'll now generate a complete product brief following the template structure. I will:
1. Use the provided information
2. Make reasonable assumptions for missing information
3. Clearly mark any assumptions made
4. Highlight any critical information that still needs to be determined
5. Focus on core sections first (Product Overview, Problem Statement, Target Market, Features & Requirements, Technical Requirements)
```

## Validation Rules
When validating an existing product brief:
1. Check for the presence of each recommended section
2. Highlight any missing sections with a warning (not an error)
3. Verify that each section contains appropriate content
4. Suggest improvements for sections that appear incomplete
5. Ensure the document maintains an appropriate level of detail
6. Validate that assumptions are clearly marked

## Guidelines
1. Minimize questions - only ask for essential missing information
2. Focus on creating a solid draft rather than perfection
3. Be concise in interactions and output
4. Use Markdown formatting for readability
5. Make reasonable assumptions where information can be inferred
6. Prioritize core sections (Product Overview, Problem Statement, Target Market, Features & Requirements, Technical Requirements)

## Examples

### Example Generation
```markdown
# Product Brief: [Product Name]

## Product Overview
- Product Name: [Name]
- Product Category: [Category]
...

[Continue with all sections]
```

### Example Validation
```markdown
⚠️ Missing Sections:
- Technical Requirements
- Resources & Budget
- Future Considerations

✅ Present Sections:
- Product Overview
- Executive Summary
...
```

## Best Practices
1. Start with the most critical sections first (Product Overview, Problem Statement, Goals)
2. Use bullet points for clarity and readability
3. Include specific metrics where possible
4. Document assumptions clearly
5. Review and update regularly as the product evolves

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mb111280) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
