## imlaw

> This document captures the systematic analysis and field handling rules for Form I-485 (Application to Adjust Status to Permanent Resident in the United States).

# I-485 Form Analysis Rules

This document captures the systematic analysis and field handling rules for Form I-485 (Application to Adjust Status to Permanent Resident in the United States).

## Form Structure Overview
- **Total Parts**: 14 numbered parts
- **Pre-Part 1**: Office/Attorney section
- **Analysis Approach**: Interactive part-by-part with domain expert annotations

---

## Office Section (Pre-Part 1)

**Description**: Attorney and office administrative fields that appear before Part 1

**Fields Included**:
- Checkboxes (various)
- Volag Number
- "Select this box if form G-28 attached" checkbox
- Attorney State Bar
- Attorney or Accredited Representative 
- USCIS Online Account Number (if any)

**Rules**:
- **Persona**: `attorney`
- **Domain**: `office`
- **Uniqueness**: Single occurrence per form (not repeating within this form)
- **Cross-form Usage**: Same data types collected across all forms
- **Collection Mapping**: 1:1 relationship (one collection field maps to one form field)
- **Screen Labels**: Use tooltip text as provided
- **Optional Fields**: USCIS Online Account Number marked as optional due to "(if any)" notation

**Implementation Notes**:
- These fields should create individual collection fields for reuse across forms
- No grouping or repeating field logic needed for this section
- Standard attorney persona validation applies

---

## Part 1: Information About You

**Description**: Applicant's current and historical personal information

**General Rules**:
- **Persona**: `applicant` (all fields)
- **Domain**: `personal` (except immigration history items 18-24 which are `immigration`)

**Field Patterns & Collection Mapping**:

1. **Current Names (1.a-c)**: Family/Given/Middle name
   - **Collection Mapping**: One-to-many (reused across forms)
   - **Fields**: Family Name, Given Name, Middle Name

2. **US Mailing Address**: Complete address information  
   - **Collection Mapping**: One-to-many (reused across forms)
   - **Domain**: `personal`

3. **Previous Names (2-4.a-c)**: Other names used since birth
   - **Collection Mapping**: Repeating fields (0-3 occurrences)
   - **Logic**: If 1 collected → maps to 2.a-c, if 2 collected → maps to 2.a-c + 3.a-c, etc.

4. **Date of Birth**: DOB field
   - **Type**: Date
   - **Collection Mapping**: One-to-many

5. **Dual Checkbox Pattern (Q6 Sex)**: Male/Female selection
   - **Structure**: Separate Male checkbox + Female checkbox  
   - **Logic**: Value determined by which checkbox = true
   - **Pattern**: Recurring across forms for either/or questions

6. **Yes/No Button Pattern (Q14 SSA)**: Has SSA issued card?
   - **Structure**: Yes button + No button
   - **Logic**: Standard selection pattern

7. **Character-by-character Fields**:
   - **Q15 SSN**: 9 individual character boxes
   - **Alien Number**: Similar individual character pattern
   - **Logic**: Order-dependent collection, combine into single field value

8. **Recent Immigration History (Items 18-24)**:
   - **Domain**: `immigration` 
   - **Persona**: `applicant`
   - **Fields**: Date of last arrival, passport info, etc.

9. **Simple Checkbox + Text Combo (25.a)**:
   - **Structure**: Checkbox (does it apply) + Text area (additional info)
   - **Logic**: Linked fields that go together

10. **Complex Option Pattern (12.c)**: Address unit type
    - **Structure**: Multiple checkboxes (Apt, Ste, Flr) + conditional text field
    - **Grouping**: All components grouped as single question
    - **Logic**: Text field activated based on checkbox selection

11. **I-94 Number (26.a)**: Character-by-character like SSN
    - **Logic**: Same pattern as SSN collection

12. **Context-specific Name Reuse (28.a-c)**: Name in native alphabet
    - **Collection Mapping**: One-to-many (same as 1.a-c fields)
    - **Example**: Demonstrates field reuse in different contexts

**Implementation Notes**:
- Previous names require dynamic mapping based on quantity collected
- Character-by-character fields need position-aware collection logic
- Grouped option fields (12.c) must maintain component relationships
- Immigration history section uses different domain but same persona

## Part 2: Application Type or Filing Category

**Description**: Immigration category selection and derivative applicant information

**Section 2A - Immigration Category Selection**:
- **Persona**: `applicant`
- **Domain**: `immigration`

**Field Pattern - Category Selection (1.a - 1.g)**:
- **Structure**: Question + multiple checkbox options grouped under each sub-question
- **Sub-questions**: 1.a Family-based, 1.b Employment-based, 1.c Special Immigrant, 1.d Asylee/Refugee, 1.e Human Trafficking/Crime Victim, 1.f Special Programs, 1.g Additional Options
- **Logic**: Each sub-question (1.a, 1.b, etc.) groups its checkboxes as options for that field
- **Collection Mapping**: Grouped checkboxes per sub-question, tooltip should list all options

**Section 2B - Information About Your Immigrant Category**:
- **Description**: For derivative applicants (beneficiaries) to provide principal applicant details
- **Persona**: `beneficiary` 
- **Domain**: `immigration`
- **Collection Mapping**: One-to-one collection fields for principal applicant data
- **Fields**: Name, alien number, address, etc. stored as "Principal Applicant" data

## Part 3: Information About the Applicant

**Description**: Applicant's address and employment history

**General Rules**:
- **Persona**: `applicant`
- **Domain**: `personal`

**Field Patterns**:

1. **Current Address Information**:
   - **Collection Mapping**: One-to-many (reused across forms)

2. **Previous Addresses**:
   - **Structure**: Two previous addresses requested
   - **Collection Options**: 
     - Option A: Separate collection fields (Address1, Address2)
     - Option B: Previous address collection with flags (addr1, addr2)
   - **Logic**: Must map to both address slots

3. **Current Employment History**:
   - **Collection Mapping**: One-to-many employment data

4. **Previous Employers**:
   - **Structure**: Two previous employers requested  
   - **Collection Logic**: Same pattern as addresses - either separate fields or flagged collection
   - **Fields**: Employer1, Employer2 or Previous Employer with position flags

## Part 4: Information About Your Parents

**Description**: Parent identification and name history

**General Rules**:
- **Persona**: `family_member` (specifically parents)
- **Domain**: `personal`

**Field Patterns**:

1. **Parent Structure**:
   - **Count**: Two parents (Parent 1, Parent 2)
   - **Required**: At least one parent must be provided

2. **Parent Name Pattern**:
   - **Current Name**: Legal name (required)
   - **Birth Name**: Name if different at time of birth (conditional)
   - **Collection Logic**: Parent name collection with flag (legal/previous)
   - **Mapping**: Must map to both name fields per parent

3. **Implementation Notes**:
   - Each parent can have up to 2 names (current + birth if different)
   - Collection field should handle name change history
   - Flag system: "legal" (current) vs "previous" (at birth)

## Part 5: [TO BE DOCUMENTED]

## Part 6: [TO BE DOCUMENTED]

## Part 7: [TO BE DOCUMENTED]

## Part 8: [TO BE DOCUMENTED]

## Part 9: [TO BE DOCUMENTED]

## Part 10: [TO BE DOCUMENTED]

## Part 11: [TO BE DOCUMENTED]

## Part 12: [TO BE DOCUMENTED]

## Part 13: [TO BE DOCUMENTED]

## Part 14: [TO BE DOCUMENTED]

---

## Processing Guidelines

**Field Extraction Priority**:
1. Apply part-specific persona and domain rules
2. Extract screen labels from tooltips
3. Determine collection field mapping (1:1 vs grouped vs repeating)
4. Validate against cross-form consistency requirements

**Domain Validation**:
- `office`: Attorney/administrative fields
- `personal`: Applicant personal information  
- `family`: Family member information
- `medical`: Health and medical examination data
- `criminal`: Criminal history and inadmissibility
- `immigration`: Immigration history and status

**Persona Validation**:
- `attorney`: Legal representative fields
- `applicant`: Primary filer information
- `family_member`: Spouse, children, parents
- `preparer`: Form preparation assistance
- `interpreter`: Translation services
- `physician`: Medical examination

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/amyamyloyd) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-15 -->
