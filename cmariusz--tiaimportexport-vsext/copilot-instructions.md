## tiaimportexport-vsext

> The file `.github/ProjectDescription.md` contains a comprehensive description of the project architecture, communication software structure, library blocks, protocols, and data flow.

# Copilot Instructions

## Project Description

The file `.github/ProjectDescription.md` contains a comprehensive description of the project architecture, communication software structure, library blocks, protocols, and data flow. 

The functional description of automation systems should provide the developer with a clear and detailed understanding of what the system is supposed to do and what are the requirements to achieve this. It should also provide guidance on how the system should be designed and implemented to effectively meet these requirements.


**Rule:** The project description should include:

1. General description of the system.
2. Detailed functional requirements for devices controlled by the automation system taking into account the technological division into objects (e.g. pump, valve, measurement) or groups of objects with the full structure of nesting objects, the so-called “plant hierarchy”.
3. The object description should include:
   •	a brief description of the operation of the facility,
   •	operating modes,
   •	in automatic mode, the algorithm for developing commands controlling an object (if applicable to a single object or a reference to the description of control of a larger group of objects),
   •	description of interlocks with logic (e.g. for start, operation, opening, closing etc.)
   •	handling of errors and exceptions,
   •	control method with HMI
4. The automatic operation of a group of objects should be described with the same requirements as for individual objects. In addition, the object control algorithm should clearly describe how the system is to be controlled (settings, emergency situations, calculations, performance requirements, control loops etc.)
5. In case of sequences of activities, the following should be described:
   •	starting conditions, interruptions of sequences with logic,
   •	description of sequence steps with the commands for a given step,
   •	conditions for transitions between steps with logic,
   •	the sequence should be represented graphically.
6. Data exchange interfaces between systems

**Always** consult this file for context when answering questions about the project.

**Rule:** When making changes that affect the project structure, architecture, communication logic, block hierarchy, message definitions, or data block layout, **always** update `.github/ProjectDescription.md` to reflect those changes. This includes:

- Adding, removing, or renaming program blocks (FB, FC, OB, DB, UDT)
- Changing message IDs, data structures, or communication channels
- Modifying the call hierarchy or adding new communication channels
- Changes to library blocks

**Rule:** If `.github/ProjectDescription.md` is empty or does not exist, and you are analyzing the project's software structure (e.g. reviewing program blocks, communication logic, library blocks, or data flow), **always** generate and populate `ProjectDescription.md` with a comprehensive description of the project before proceeding.

**Rule:** When creating or updating `ProjectDescription.md`, **always** include **Mermaid diagrams** (`\`\`\`mermaid`) to visually illustrate key aspects of the project. At minimum, include diagrams for:

- **Architecture overview** — block/folder dependencies and relationships (use `graph TD`)
- **Communication channel topology** — TCP connections, OPC UA channels, ports (use `graph LR`)
- **Call hierarchy** — OB → FB → library instance call tree (use `graph TD`)
- **Data flow / sequence diagrams** — message send/receive sequences for each communication direction (use `sequenceDiagram`)
- **State machines** — for blocks with complex internal state logic (use `stateDiagram-v2`)
- **Data structures** — for key UDT/message layouts (use `packet-beta` or `block-beta`)
- **Queue/scheduling logic** — for FIFO managers or priority systems (use `flowchart LR`)

Place each diagram **directly above or below** the corresponding textual description it illustrates. Keep text descriptions alongside diagrams — diagrams supplement, not replace, the text.

---

## Project Directories

### `Tools/`

Directory with utility scripts for the project (Python, PowerShell, etc.). Each script has a corresponding `.md` file with a description, usage examples, and dependencies.

**Rule:** When creating a new script, **always** place it in `Tools/` and create a matching `.md` description file (e.g. `Tools/_myScript.py` + `Tools/_myScript.md`).

### `UserFiles/`

Directory for user-generated output files produced by scripts from `Tools/` (Excel reports, CSV exports, logs, etc.). Scripts should write their default output here. This directory is not imported back into TIA Portal and is excluded from version control artifacts.

---

## Siemens TIA Portal Export Files (Openness XML)

The XSD schemas in `.github/Schemas/` define the valid structure for all TIA Portal Openness XML files. **Always** follow these schemas when creating or modifying `.xml` block files.

---

### S7 Declaration and Resource Files (.s7dcl + .s7res)

Every `.s7dcl` file has a corresponding `.s7res` file in the same directory. The `.s7res` file contains multilingual text resources (translations) referenced by MLC IDs used in the `.s7dcl` file.

**Rule:** When modifying a `.s7dcl` file, **always** check and update the corresponding `.s7res` file:

- If a new `NETWORK` is added with a `S7_NetworkTitle` MLC ID, add the matching entry in the `.s7res` file.
- If a new `NETWORK` is added with a `S7_NetworkComment` MLC ID, add the matching entry in the `.s7res` file.
- If any attribute with an MLC reference (`S7_MLC`, `S7_NetworkTitle`, `S7_NetworkComment`) is added or changed, ensure the `.s7res` file reflects that change.
- If a NETWORK or variable with an MLC reference is removed, remove the corresponding entry from the `.s7res` file.

### .s7res File Format

The `.s7res` file uses YAML-like format:

```yaml
MultiLingualTexts:
  - id: MLC_xxx
    en-US: English text description
```

### Block Name Consistency (all TIA export files)

When creating a new file by copying or duplicating an existing `.s7dcl` or `.xml` block file, **always** verify and update the block name **inside** the file to match the new file name (without extension):

- **`.s7dcl` files** — update the block declaration keyword line (e.g. `FUNCTION_BLOCK "OldName"`, `FUNCTION "OldName"`, `DATA_BLOCK "OldName"`, `ORGANIZATION_BLOCK "OldName"`) so the quoted name matches the new file name.
- **`.xml` files (Program blocks / PLC data types)** — update the `<Name>OldName</Name>` element inside `<AttributeList>` so it matches the new file name (without `.xml`).

**Rule:** The file name (without extension) and the block name declared inside the file **must always be identical**. When renaming or duplicating a file, scan the entire file content for occurrences of the old block name and replace them with the new one.

---

### Global Data Block Source Files (.db)

Global Data Blocks can be exported as text-based source files (`.db`) instead of XML. This format is produced by the `IGenerateSource` / `PlcExternalSourceSystemGroup.GenerateSource()` API and uses standard SCL-like DATA_BLOCK syntax.

**Default format:** `db` (controlled by `tiaImport.dbExportFormat` setting; can be switched to `xml`)

**Scope:** Only **Global Data Blocks** (`SW.Blocks.GlobalDB`) use this format. **Instance Data Blocks** (`SW.Blocks.InstanceDB`) are always exported as XML regardless of setting.

#### .db File Structure

```
DATA_BLOCK "BlockName"
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
NON_RETAIN
   STRUCT 
      Field1 : Bool;   // comment
      Field2 : Int := 10;   // comment with start value
      Field3 : Real;
      Field4 : "UdtName";   // UDT reference
      Field5 : Array[0..9] of Int;
      Field6 : Struct
         SubField1 : Bool;
         SubField2 : Word;
      END_STRUCT;
   END_STRUCT;

BEGIN

END_DATA_BLOCK
```

#### Key Syntax Rules

- Block name in `DATA_BLOCK "Name"` must match the file name (without `.db` extension).
- UDT references use double quotes: `"UdtName"`.
- Start values are assigned with `:=` after the type: `Field : Int := 42;`.
- Comments use `//` after the semicolon.
- `{ S7_Optimized_Access := 'TRUE' }` — block attribute for optimized memory layout.
- `NON_RETAIN` or `RETAIN` keyword before `STRUCT` controls remanence.
- `BEGIN` section can contain initial value assignments (usually empty for source-generated files).
- Nested `Struct` uses `Struct ... END_STRUCT;` inline.

#### .db → TIA Portal Import

When importing `.db` files back to TIA Portal, the file is treated as an SCL external source:

1. File is imported via `PlcExternalSourceSystemGroup` (same path as `.scl` files)
2. TIA Portal compiles the source into a Global Data Block
3. The `XmlTypeDetector` maps `.db` extension to `SclBlock` type for routing

**Rule:** When modifying `.db` files:

- Keep the `DATA_BLOCK "Name"` declaration matching the file name.
- Do **not** add XML structure — `.db` files are pure text, not XML.
- Maintain proper SCL syntax for data types and start values.
- Array syntax: `Array[lo..hi] of Type` (same as in XML `Datatype` attribute).

---

### PLC Tag Tables — Export Formats (XML / XLSX)

PLC tag tables can be exported in two formats, controlled by the `tiaImport.tagTableFormat` setting:

| Format   | Extension | Description                                                                  |
| -------- | --------- | ---------------------------------------------------------------------------- |
| `xml`  | `.xml`  | SimaticML XML — native TIA Portal Openness format (`SW.Tags.PlcTagTable`) |
| `xlsx` | `.xlsx` | XLSX spreadsheet — converted from XML after export; XML files are deleted   |

**Default format:** `xlsx`

#### XLSX Structure

Each XLSX file represents one PLC tag table and contains two sheets:

**Sheet "Tags"** — PLC tags (`SW.Tags.PlcTag`):

| Column          | Description                            | Source XML element                   |
| --------------- | -------------------------------------- | ------------------------------------ |
| Name            | Tag name                               | `<Name>`                           |
| Data Type       | SIMATIC data type                      | `<DataTypeName>`                   |
| Logical Address | HW address (e.g.`%I0.0`, `%MW100`) | `<LogicalAddress>`                 |
| Comment         | Tag comment (en-US)                    | `<MultilingualText>` → `<Text>` |
| Retain          | Retain flag                            | `<Retain>`                         |
| Accessible      | OPC UA / external accessible           | `<ExternalAccessible>`             |
| Visible         | External visible                       | `<ExternalVisible>`                |
| Writable        | External writable                      | `<ExternalWritable>`               |

**Sheet "Constants"** — PLC user constants (`SW.Tags.PlcUserConstant`):

| Column    | Description              | Source XML element                   |
| --------- | ------------------------ | ------------------------------------ |
| Name      | Constant name            | `<Name>`                           |
| Data Type | SIMATIC data type        | `<DataTypeName>`                   |
| Value     | Constant value           | `<Value>`                          |
| Comment   | Constant comment (en-US) | `<MultilingualText>` → `<Text>` |

#### XLSX → TIA Portal Import

When importing XLSX files back to TIA Portal:

1. XLSX is converted to temporary SimaticML XML
2. XML is imported via TIA Portal Openness `PlcTagTableComposition.Import()`
3. Temporary XML is cleaned up

**Limitation:** TIA Portal Import API does **not** support the `Constants` composition (`SW.Tags.PlcUserConstant`). Constants from the XLSX "Constants" sheet are preserved in the spreadsheet but are **not** imported back to TIA. Only tags from the "Tags" sheet are imported.

**Rule:** When modifying XLSX tag table files:

- Do **not** change the sheet names ("Tags", "Constants") — the converter expects these exact names.
- Do **not** remove or reorder columns — the converter reads columns by position (A, B, C...).
- The header row (row 1) is skipped during import.
- Duplicate tag names are deduplicated — the **last** occurrence wins.

---

## TIA Portal Openness XML Block Structure (schemas: `.github/Schemas/`)

### 1. Common Document Envelope

Every TIA Portal XML export file **must** follow this root structure:

```xml
<?xml version="1.0" encoding="utf-8"?>
<Document>
  <Engineering version="V21" />
  <DocumentInfo>
    <Created>2026-01-01T00:00:00.0000000Z</Created>
    <ExportSetting>WithDefaults</ExportSetting>
    <InstalledProducts>
      <Product>
        <DisplayName>Totally Integrated Automation Portal</DisplayName>
        <DisplayVersion>V21</DisplayVersion>
      </Product>
    </InstalledProducts>
  </DocumentInfo>
  <!-- Block-type-specific element goes here -->
</Document>
```

- `<Document>` is the root — **no namespace attributes** on this element.
- `<Engineering version="V21" />` — self-closing, must match project version.
- `<ExportSetting>` is `WithDefaults` or `WithReadOnly`.

### 2. Block Type Root Elements

Use the correct root element inside `<Document>` depending on the block type:

| Block type                | Root element                      | `ProgrammingLanguage`                       |
| ------------------------- | --------------------------------- | --------------------------------------------- |
| Function Block (FB)       | `<SW.Blocks.FB ID="0">`         | `SCL`, `LAD`, `FBD`, `STL`, `GRAPH` |
| Function (FC)             | `<SW.Blocks.FC ID="0">`         | `SCL`, `LAD`, `FBD`, `STL`            |
| Organization Block (OB)   | `<SW.Blocks.OB ID="0">`         | `SCL`, `LAD`, `FBD`, `STL`            |
| Global Data Block (DB)    | `<SW.Blocks.GlobalDB ID="0">`   | `DB`                                        |
| Instance Data Block (IDB) | `<SW.Blocks.InstanceDB ID="0">` | `DB`                                        |
| PLC Data Type (UDT)       | `<SW.Types.PlcStruct ID="0">`   | *(none)*                                    |

Each block element contains `<AttributeList>` followed by `<ObjectList>`.

### 3. AttributeList Structure

#### Common attributes for all program blocks (FB, FC, OB):

```xml
<AttributeList>
  <AutoNumber>true</AutoNumber>
  <HeaderAuthor />
  <HeaderFamily />
  <HeaderName />
  <HeaderVersion>0.1</HeaderVersion>
  <Interface>...</Interface>
  <MemoryLayout>Optimized</MemoryLayout>
  <Name>BlockName</Name>
  <Namespace />
  <Number>1</Number>
  <ProgrammingLanguage>SCL</ProgrammingLanguage>
</AttributeList>
```

#### Additional attributes for Global DB:

```xml
<DBAccessibleFromOPCUA>true</DBAccessibleFromOPCUA>
<DBAccessibleFromWebserver>true</DBAccessibleFromWebserver>
<IsOnlyStoredInLoadMemory>false</IsOnlyStoredInLoadMemory>
<IsRetainMemResEnabled>false</IsRetainMemResEnabled>
<IsWriteProtectedInAS>false</IsWriteProtectedInAS>
<MemoryReserve>100</MemoryReserve>
```

#### Additional attributes for Instance DB:

```xml
<InstanceOfName>FBName</InstanceOfName>
<InstanceOfType>FB</InstanceOfType>
```

For system library instances, also add:

```xml
<OfSystemLibElement>true</OfSystemLibElement>
<OfSystemLibVersion>...</OfSystemLibVersion>
```

#### Additional attributes for GRAPH FB:

```xml
<GraphVersion>6.0</GraphVersion>
<LanguageInNetworks>LAD</LanguageInNetworks>
<WithAlarmHandling>true</WithAlarmHandling>
<SkipSteps>true</SkipSteps>
```

#### PLC Data Type (UDT) AttributeList:

```xml
<AttributeList>
  <Interface>...</Interface>
  <IsFailsafeCompliant>false</IsFailsafeCompliant>
  <Name>TypeName</Name>
  <Namespace />
</AttributeList>
```

### 4. Interface / Sections (Schema: `SW.InterfaceSections_v5.xsd`)

The `<Interface>` element contains `<Sections>` which **must** declare the namespace:

```xml
<Interface>
  <Sections xmlns="http://www.siemens.com/automation/Openness/SW/Interface/v5">
    <Section Name="Input">...</Section>
    <Section Name="Output">...</Section>
    <Section Name="InOut">...</Section>
    <Section Name="Static">...</Section>
    <Section Name="Temp">...</Section>
    <Section Name="Constant">...</Section>
  </Sections>
</Interface>
```

**Section Name values** (from `SectionName_TE`): `None`, `Input`, `Return`, `Output`, `InOut`, `Static`, `Temp`, `Constant`, `Base`.

**Rules:**

- **FB** uses sections: `Input`, `Output`, `InOut`, `Static`, `Temp`, `Constant` (and `Base` for GRAPH).
- **FC** uses sections: `Input`, `Output`, `InOut`, `Temp`, `Constant`, `Return`.
- **OB** uses sections: `Input`, `Output`, `InOut`, `Static`, `Temp`, `Constant`.
- **Global DB** uses section: `Static` only.
- **UDT** uses section: `None` only.

#### Member Declaration

Each variable in a Section is a `<Member>` element:

```xml
<Member Name="VarName" Datatype="Int" Remanence="NonRetain" Accessibility="Public">
  <AttributeList>
    <BooleanAttribute Name="ExternalAccessible" SystemDefined="true">true</BooleanAttribute>
    <BooleanAttribute Name="ExternalVisible" SystemDefined="true">true</BooleanAttribute>
    <BooleanAttribute Name="ExternalWritable" SystemDefined="true">true</BooleanAttribute>
    <BooleanAttribute Name="SetPoint" SystemDefined="true">false</BooleanAttribute>
  </AttributeList>
  <Comment>
    <MultiLanguageText Lang="en-US">Description</MultiLanguageText>
  </Comment>
  <StartValue>0</StartValue>
</Member>
```

**Member attributes:**

- `Name` (required) — variable name
- `Datatype` (required) — SIMATIC type: `Bool`, `Byte`, `Char`, `Word`, `Int`, `DWord`, `DInt`, `Real`, `LReal`, `String`, `Time`, `Date`, `Time_Of_Day`, `Array[lo..hi] of Type`, `Struct`, or UDT name in escaped quotes `&quot;UdtName&quot;`
- `Remanence` — `NonRetain` (default), `Retain`, or `SetInIDB`
- `Accessibility` — `Public` (default), `Internal`, `Protected`, `Private`
- `Version` — optional, for versioned library types

**UDT references** — when referencing a PLC Data Type, use escaped quotes in the `Datatype` attribute:

```xml
<Member Name="Header" Datatype=""MsgHeader"">
```

**Struct members** — for inline Struct, nest `<Member>` elements inside:

```xml
<Member Name="Data" Datatype="Struct">
  <Member Name="Field1" Datatype="Int">...</Member>
  <Member Name="Field2" Datatype="Real">...</Member>
</Member>
```

**Array members** — use `Array[lo..hi] of Type` syntax in Datatype:

```xml
<Member Name="Buffer" Datatype="Array[0..9] of Int">
```

### 5. ObjectList Structure

`<ObjectList>` follows `<AttributeList>` and contains:

```xml
<ObjectList>
  <MultilingualText ID="1" CompositionName="Comment">
    <ObjectList>
      <MultilingualTextItem ID="2" CompositionName="Items">
        <AttributeList>
          <Culture>en-US</Culture>
          <Text>Block comment</Text>
        </AttributeList>
      </MultilingualTextItem>
    </ObjectList>
  </MultilingualText>
  <!-- CompileUnits go here for FB/FC/OB (not for DB/UDT) -->
  <MultilingualText ID="N" CompositionName="Title">
    <ObjectList>
      <MultilingualTextItem ID="N+1" CompositionName="Items">
        <AttributeList>
          <Culture>en-US</Culture>
          <Text>Block title</Text>
        </AttributeList>
      </MultilingualTextItem>
    </ObjectList>
  </MultilingualText>
</ObjectList>
```

**Rules:**

- `ID` attributes must be unique positive integers within the file, sequentially assigned.
- `CompositionName="Comment"` and `CompositionName="Title"` are required for all blocks.
- DB and UDT have only `Comment` and `Title` (no CompileUnits).

### 6. CompileUnit — Program Code (Schema: `SW.PlcBlocks.CompileUnitCommon_v5.xsd`)

FB, FC, and OB blocks contain `<SW.Blocks.CompileUnit>` elements for code:

```xml
<SW.Blocks.CompileUnit ID="3" CompositionName="CompileUnits">
  <AttributeList>
    <NetworkSource>
      <!-- Language-specific code content here -->
    </NetworkSource>
    <ProgrammingLanguage>LAD</ProgrammingLanguage>
  </AttributeList>
  <ObjectList>
    <MultilingualText ID="4" CompositionName="Comment">...</MultilingualText>
    <MultilingualText ID="6" CompositionName="Title">...</MultilingualText>
  </ObjectList>
</SW.Blocks.CompileUnit>
```

Each CompileUnit represents one **network**. Multiple networks = multiple CompileUnit elements.

### 7. LAD/FBD Code (Schema: `SW.PlcBlocks.LADFBD_v5.xsd`)

LAD/FBD networks use `<FlgNet>` inside `<NetworkSource>`:

```xml
<NetworkSource>
  <FlgNet xmlns="http://www.siemens.com/automation/Openness/SW/NetworkSource/FlgNet/v5">
    <Parts>
      <!-- Access elements (variables) and Part elements (instructions) -->
    </Parts>
    <Wires>
      <!-- Wire elements connecting parts -->
    </Wires>
  </FlgNet>
</NetworkSource>
```

#### Parts

**Access** — references to variables/constants:

```xml
<Access Scope="GlobalVariable" UId="21">
  <Symbol>
    <Component Name="DataBlock" />
    <Component Name="Variable" />
  </Symbol>
</Access>
```

`Scope` values (from `Scope_TE`): `GlobalVariable`, `LocalVariable`, `LiteralConstant`, `GlobalConstant`, `TypedConstant`, `Call`, `Instruction`, `Label`, `Address`, and others.

**Part** — instructions/operations:

```xml
<Part Name="Contact" UId="23" />
<Part Name="Coil" UId="24" />
<Part Name="Move" Version="1.0" UId="25">
  <Negated Name="en" />
  <Comment><MultiLanguageText Lang="en-US">...</MultiLanguageText></Comment>
</Part>
```

**Call** — FB/FC calls:

```xml
<Call UId="30">
  <CallInfo Name="BlockName" BlockType="FB" />
</Call>
```

`BlockType` values: `DB`, `FB`, `FC`, `OB`, `UDT`, `FBT`, `FCT`.

#### Wires

Wires connect parts via their pins using `UId` references:

```xml
<Wire UId="26">
  <Powerrail />                          <!-- LAD power rail source -->
  <NameCon UId="23" Name="in" />         <!-- connect to pin "in" of Part with UId 23 -->
</Wire>
<Wire UId="27">
  <NameCon UId="23" Name="out" />        <!-- output of Part UId 23 -->
  <NameCon UId="24" Name="in" />         <!-- input of Part UId 24 -->
</Wire>
<Wire UId="28">
  <IdentCon UId="21" />                  <!-- connect to Access with UId 21 -->
  <NameCon UId="25" Name="in" />
</Wire>
```

Wire connection types:

- `<Powerrail />` — LAD power rail (always first connection in wire)
- `<NameCon UId="X" Name="pin" />` — named pin connection to a Part
- `<IdentCon UId="X" />` — connection to an Access element
- `<OpenCon UId="X" />` — open/unconnected pin
- `<Openbranch />` — open branch end

**UId rules:**

- Every `Access`, `Part`, `Call`, `Wire`, and `OpenCon` **must** have a unique `UId` within the `<FlgNet>`.
- UIds are positive integers, assigned sequentially.

### 8. SCL Code (Schema: `SW.PlcBlocks.SCL_v4.xsd`)

SCL networks use `<StructuredText>` inside `<NetworkSource>`:

```xml
<NetworkSource>
  <StructuredText xmlns="http://www.siemens.com/automation/Openness/SW/NetworkSource/StructuredText/v4">
    <!-- SCL code represented as Access, Token, Comment elements -->
  </StructuredText>
</NetworkSource>
```

SCL code is tokenized — each keyword, operator, variable, and literal is an `<Access>` or `<Token>` element:

```xml
<Access Scope="LocalVariable" UId="1">
  <Symbol><Component Name="myVar" /></Symbol>
</Access>
<Token Text=":=" UId="2" />
<Access Scope="LiteralConstant" UId="3">
  <Constant><ConstantValue>42</ConstantValue></Constant>
</Access>
<Token Text=";" UId="4" />
<NewLine />
```

**Token** `Text` values include all SCL keywords and operators: `:=`, `;`, `IF`, `THEN`, `ELSE`, `END_IF`, `FOR`, `TO`, `DO`, `END_FOR`, `WHILE`, `END_WHILE`, `CASE`, `OF`, `END_CASE`, `+`, `-`, `*`, `/`, `=`, `<>`, `<`, `>`, `<=`, `>=`, `AND`, `OR`, `NOT`, `(`, `)`, `.`, `,`, etc.

### 9. STL Code (Schema: `SW.PlcBlocks.STL_v5.xsd`)

STL networks use `<StatementList>` inside `<NetworkSource>`:

```xml
<NetworkSource>
  <StatementList xmlns="http://www.siemens.com/automation/Openness/SW/NetworkSource/StatementList/v5">
    <StlStatement>
      <StlToken Text="A" />
      <Access Scope="GlobalVariable">
        <Symbol><Component Name="Tag_1" /></Symbol>
      </Access>
    </StlStatement>
    <StlStatement>
      <StlToken Text="Assign" />
      <Access Scope="GlobalVariable">
        <Symbol><Component Name="Tag_2" /></Symbol>
      </Access>
    </StlStatement>
  </StatementList>
</NetworkSource>
```

**StlToken `Text`** values (from `STL_TE`): `A`, `AN`, `O`, `ON`, `X`, `XN`, `S`, `R`, `Assign`, `Rise`, `Fall`, `L`, `T`, `CALL`, `CC`, `UC`, `JU`, `JC`, `JCN`, `ADD_I`, `SUB_I`, `MUL_I`, `DIV_I`, `ADD_D`, `SUB_D`, `MUL_D`, `DIV_D`, `ADD_R`, `SUB_R`, `MUL_R`, `DIV_R`, `SET`, `CLR`, `NEG`, `SAVE`, `BE`, `BEC`, `BEU`, `NOP_0`, `NOP_1`, `COMMENT`, `EMPTY_LINE`, and many more.

### 10. GRAPH Code (Schema: `SW.PlcBlocks.Graph_v6.xsd`)

GRAPH blocks use `<Graph>` inside `<NetworkSource>`:

```xml
<NetworkSource>
  <Graph xmlns="http://www.siemens.com/automation/Openness/SW/NetworkSource/Graph/v6">
    <PreOperations>
      <Title>...</Title>
      <Comment>...</Comment>
      <PermanentOperation ProgrammingLanguage="LAD">
        <Title>...</Title>
        <FlgNet>...</FlgNet>
      </PermanentOperation>
    </PreOperations>
    <Sequence>
      <Title>...</Title>
      <Comment>...</Comment>
      <Steps>
        <Step Number="1" Init="true" Name="S1">...</Step>
      </Steps>
      <Transitions>
        <Transition Number="1" Name="T1" ProgrammingLanguage="LAD">...</Transition>
      </Transitions>
      <Branches />
      <Connections>
        <Connection>
          <NodeFrom><StepRef Number="1" /></NodeFrom>
          <NodeTo><TransitionRef Number="1" /></NodeTo>
          <LinkType>Direct</LinkType>
        </Connection>
      </Connections>
    </Sequence>
    <PostOperations>...</PostOperations>
    <AlarmsSettings>...</AlarmsSettings>
  </Graph>
</NetworkSource>
```

#### Step Structure:

```xml
<Step Number="1" Init="true" Name="S1" MaximumStepTime="T#0ms" WarningTime="T#0ms">
  <StepName>
    <MultiLanguageText Lang="en-US">Step name</MultiLanguageText>
  </StepName>
  <Comment>...</Comment>
  <Actions>
    <Title>...</Title>
    <Action Qualifier="N">
      <!-- Action tokens: Access + Token elements -->
    </Action>
  </Actions>
  <Supervisions>
    <Supervision ProgrammingLanguage="LAD">
      <Title>...</Title>
      <FlgNet>...</FlgNet>
    </Supervision>
  </Supervisions>
  <Interlocks>
    <Interlock ProgrammingLanguage="LAD">
      <Title>...</Title>
      <FlgNet>...</FlgNet>
    </Interlock>
  </Interlocks>
</Step>
```

**Action Qualifier values** (`Qualifier_TE`): `N` (Non-stored), `S` (Set), `R` (Reset), `D` (Delayed), `L` (Time limited), `CD`, `CR`, `CS`, `CU`, `ON`, `OFF`, `TD`, `TF`, `TL`, `TR`.

**Action Event values** (`Event_TE`): `""` (empty), `A1`, `L0`, `L1`, `R1`, `S0`, `S1`, `V0`, `V1`.

#### Transition Structure:

```xml
<Transition Number="1" Name="T1" ProgrammingLanguage="LAD">
  <TransitionName>
    <MultiLanguageText Lang="en-US">Transition name</MultiLanguageText>
  </TransitionName>
  <Comment>...</Comment>
  <FlgNet>
    <!-- LAD/FBD logic for transition condition -->
  </FlgNet>
</Transition>
```

#### Branch Types (`Branch_TE`):

- `SimBegin` / `SimEnd` — simultaneous branch (AND)
- `AltBegin` / `AltEnd` — alternative branch (OR)

#### Connection LinkType values (`Link_TE`): `Direct`, `Jump`.

#### AlarmsSettings Structure:

```xml
<AlarmsSettings>
  <AlarmSupervisionCategories>
    <AlarmSupervisionCategory Id="0" DisplayClass="0" />
  </AlarmSupervisionCategories>
  <AlarmInterlockCategory Id="0" />
  <AlarmSubcategory1Interlock Id="0" />
  <AlarmSubcategory2Interlock Id="0" />
  <AlarmCategorySupervision Id="0" />
  <AlarmSubcategory1Supervision Id="0" />
  <AlarmSubcategory2Supervision Id="0" />
  <AlarmWarningCategory Id="0" />
  <AlarmSubcategory1Warning Id="0" />
  <AlarmSubcategory2Warning Id="0" />
</AlarmsSettings>
```

### 11. NetworkSource Namespace Rules

Each programming language content element inside `<NetworkSource>` uses its own namespace:

| Language  | Element              | Namespace                                                                         |
| --------- | -------------------- | --------------------------------------------------------------------------------- |
| LAD / FBD | `<FlgNet>`         | `http://www.siemens.com/automation/Openness/SW/NetworkSource/FlgNet/v5`         |
| SCL       | `<StructuredText>` | `http://www.siemens.com/automation/Openness/SW/NetworkSource/StructuredText/v4` |
| STL       | `<StatementList>`  | `http://www.siemens.com/automation/Openness/SW/NetworkSource/StatementList/v5`  |
| GRAPH     | `<Graph>`          | `http://www.siemens.com/automation/Openness/SW/NetworkSource/Graph/v6`          |

**When GRAPH transitions/supervisions/interlocks contain inner FlgNet, those FlgNet elements do NOT repeat the namespace** (they inherit from the Graph namespace context).

### 12. Access Element Reference (Schema: `SW.PlcBlocks.Access_v5.xsd`)

The `<Access>` element is fundamental — it represents any data reference in code:

```xml
<Access Scope="ScopeValue" UId="N">
  <!-- One of: Symbol, Constant, CallInfo, Instruction, Address, Label, etc. -->
</Access>
```

#### Symbol (symbolic variable reference):

```xml
<Access Scope="GlobalVariable" UId="1">
  <Symbol>
    <Component Name="DBName" />
    <Component Name="FieldName" />
  </Symbol>
</Access>
```

#### Constant (literal value):

```xml
<Access Scope="LiteralConstant" UId="2">
  <Constant>
    <ConstantType>Int</ConstantType>
    <ConstantValue>100</ConstantValue>
  </Constant>
</Access>
```

#### Address (absolute addressing):

```xml
<Access Scope="Address" UId="3">
  <Address Area="Memory" Type="Bool" BitOffset="0" />
</Access>
```

`Area` values (`Area_TE`): `Input`, `Output`, `Memory`, `DB`, `DI`, `PeripheryInput`, `PeripheryOutput`, `Timer`, `Counter`, `Local`, `FB`, `FC`, `None`.

#### Component AccessModifier:

- `None` (default) — simple access
- `Array` — array index access (child `<Access>` elements are indices)
- `Reference` — reference access
- `ReferenceToArray` — reference to array element

### 13. MultiLanguageText (Schema: `SW.Common_v3.xsd`)

Multi-language texts for comments, titles, alarm texts:

```xml
<Comment>
  <MultiLanguageText Lang="en-US">English text</MultiLanguageText>
  <MultiLanguageText Lang="de-DE">German text</MultiLanguageText>
</Comment>
```

`Lang` format (from `Lang_TP`): pattern `[a-zA-Z]{2}(-[a-zA-Z]{4})?-[a-zA-Z]{2}` — e.g. `en-US`, `de-DE`, `zh-Hans-CN`.

### 14. Type Supervisions (Schema: `SW.PlcBlocks.TypeSupervisions_v4.xsd`)

For ProDiag supervision definitions on blocks:

```xml
<BlockTypeSupervisions>
  <BlockTypeSupervision Number="1" Type="Operand">
    <SupervisedOperand Name="VarName" />
    <SupervisedStatus>true</SupervisedStatus>
    <CategoryNumber>1</CategoryNumber>
  </BlockTypeSupervision>
</BlockTypeSupervisions>
```

Supervision `Type` values: `Action`, `Interlock`, `Operand`, `Position`, `Reaction`, `MessageText`, `MessageError`.

`CategoryNumber`: integer 1–8.

### 15. Key Validation Rules Summary

1. **Namespace on Sections**: Always declare `xmlns="http://www.siemens.com/automation/Openness/SW/Interface/v5"` on the `<Sections>` element inside `<Interface>`.
2. **Namespace on NetworkSource content**: Always declare the correct namespace on `<FlgNet>`, `<StructuredText>`, `<StatementList>`, or `<Graph>` elements.
3. **UId uniqueness**: All `UId` values within a `<FlgNet>` or code block must be unique positive integers.
4. **ID uniqueness**: All `ID` attributes on XML elements (`SW.Blocks.CompileUnit`, `MultilingualText`, etc.) must be unique within the entire file.
5. **Block name = file name**: The `<Name>` element in `<AttributeList>` must match the file name (without `.xml`).
6. **Member BooleanAttributes**: Every `<Member>` should include the standard system-defined `BooleanAttribute` set: `ExternalAccessible`, `ExternalVisible`, `ExternalWritable`, `SetPoint`.
7. **ProgrammingLanguage consistency**: The `<ProgrammingLanguage>` in `<AttributeList>` of the block must match the actual code content type in `<NetworkSource>`. CompileUnits may have their own `<ProgrammingLanguage>` that matches the network language.
8. **UDT references**: Always escape quotes in `Datatype` attribute when referencing UDTs: `Datatype="&quot;TypeName&quot;"`.
9. **Wire connectivity**: Every Part input pin must be connected by a Wire. The first connection in a LAD Wire is typically `<Powerrail />` or an output `<NameCon>`.
10. **GRAPH sequence integrity**: Every Step must be linked to at least one Transition via `<Connection>` elements. Steps and Transitions are referenced by their `Number` attribute through `<StepRef>` and `<TransitionRef>`.

---

## TIA Portal Import — Extension Reference for Copilot

This section describes how the **TIA Portal Import** VS Code extension
organizes the workspace and which features Copilot should use to work
optimally with TIA Portal V19 / V20 / V21.

### Workspace layout

`InitWorkspace` (run once on first connect) scaffolds:

```
<workspace>/
  .github/
    copilot-instructions.md         # this file
    ProjectDescription.md           # project documentation (Mermaid diagrams)
    Schemas/                        # SimaticML XSDs for validation
  Tools/                            # user scripts (.py, .ps1) + matching .md
  UserFiles/                        # script outputs (excluded from TIA import)
  TiaExport/                        # all TIA Portal mirror data
    Projects/
      <ProjectName>/
        Devices/
          PLCs/<PlcName>/
            Program blocks/         # FB/FC/OB/DB → .xml | .scl | .s7dcl(+.s7res) | .db
            PLC tags/               # tag tables   → .xml | .xlsx
            PLC data types/         # UDTs         → .xml
            Watch and force tables/
            DeviceConfiguration/    # HW config    → .xml | .aml (CAx)
          HMIs/<HmiName>/
            Screens/  Tags/  Connections/
          IO_Devices/<DeviceName>/
        Library/Types/              # project library types (V20+ for SD)
```

The export folder name comes from `tiaImport.exportFolderName` (default
`TiaExport`). Path matchers in context menus depend on these exact folder
names — **do not rename them**.

### Extension commands (Command Palette: `TIA Import: …`)

**Connection**
- `Connect to TIA Portal` / `Disconnect from TIA Portal`
- `Select Project` — pick from running TIA instances
- `Select TIA Portal Version` — V19 / V20 / V21 (requires window reload)
- `Prepare Workspace` — scaffolds the layout above
- `Refresh Project Structure`
- `Show Logs` — opens the `TIA Portal Import` Output channel

**Format toggles** (also visible as buttons in the Connection panel)
- `Select Export Format` — `xml` ↔ `sd` (program blocks)
- `Format PLC Tags` — `xml` ↔ `xlsx`
- `Format HW` — `xml` ↔ `cax` (AutomationML `.aml`)
- `Compile after Export` — `always` / `ask` / `never`

**Import (TIA → workspace)**
- `Import Entire Project`, `Import Device`, `Import Block`, `Import Block Folder`
- `Import Tag Tables` / `Import Tag Table`
- `Import Data Types` / `Import Data Type` (UDTs)
- `Import Watch Tables` / `Import Watch Table`
- `Import HMI Screens` / `Import HMI Tags` / `Import HMI Connections` / `Import All HMI Elements`
- `Import HW Configuration` / `Import Device HW Configuration`
- `Import Programs for All Devices in Category` / `Import HW Config for All Devices in Category`
- `Import Library Types` / `Import Library Folder` / `Import Library Type`

**Export (workspace → TIA)** — context menu, by file/folder pattern:
| Trigger | Command |
|--------|---------|
| `.xml` / `.scl` / `.s7dcl` / `.db` under `Program blocks/` | `Export Blocks to TIA` |
| Folder under `Program blocks/` | `Export Blocks to TIA (Folder)` |
| `.xlsx` under `PLC tags/` | `Export XLSX Tags to TIA Portal` |
| Folder under `PLC tags/` | `Export XLSX Tags to TIA Portal (Folder)` |
| `.xml` outside `Program blocks` and outside `IO_Devices` | `Export to TIA Portal: XML File` |
| `.aml` or `.xml` under `DeviceConfiguration/` | `Export to TIA Portal: HW Config` |
| Folder `Devices/<Category>/<Device>` | `Export to TIA - Program and HW` (Unified) or `Export to TIA - Program without HW` |

### Settings (`tiaImport.*`)

| Setting | Default | Notes |
|---------|---------|-------|
| `tiaPortalVersion` | `21` | `19` / `20` / `21`; window reload to apply |
| `tiaPortalPath` | `""` | Auto-detected from version when empty |
| `exportFolderName` | `"TiaExport"` | Workspace folder for all TIA mirror data |
| `autoConnect` | `false` | Connect on activation |
| `exportFormat` | `"sd"` | `xml` (SimaticML) or `sd` (`.s7dcl` LAD/FBD + `.scl` SCL via SD path; V20+ only — V19 falls back to XML) |
| `dbExportFormat` | `"db"` | Global DB → `.db` source or `.xml`. Instance DBs always `.xml` |
| `tagTableFormat` | `"xlsx"` | PLC tags → `.xlsx` (with `Tags`/`Constants` sheets) or `.xml` |
| `hwConfigFormat` | `"cax"` | HW config → `.aml` (CAx, recommended) or per-device `.xml` |
| `compileAfterExport` | `"ask"` | `always` / `ask` / `never` |
| `includeComments` | `true` | Include en-US comments in exports |
| `preserveTimestamps` | `true` | Keep TIA timestamps to detect real changes |
| `excludeSystemBlocks` | `true` | Skip Siemens system blocks |
| `showImportExportDetails` | `false` | Verbose Output channel section |
| `dotnetPath` | `""` | Auto-detect .NET 4.8 runtime |
| `lmTools.autoConfirmImports` | `false` | When false, LM-tool imports overwriting TIA objects ask for confirmation |
| `lmTools.maxFixIterations` | `5` | Cap on `tia_fix_compile_errors` loop (1–20) |

### Export format decision matrix

| Block type | `exportFormat="xml"` | `exportFormat="sd"` (V20+) |
|------------|----------------------|---------------------------|
| FB / FC / OB in **SCL** | `.xml` | `.scl` (via `GenerateSource`) |
| FB / FC / OB in **LAD / FBD / STL** | `.xml` | `.s7dcl` + `.s7res` (via `ExportAsDocuments`) |
| GRAPH FB | `.xml` | `.xml` (SD not supported) |
| **Global DB** | `.xml` | follows `dbExportFormat` (`.db` or `.xml`) |
| **Instance DB** | `.xml` (always) | `.xml` (always) |
| **UDT** | `.xml` | `.xml` |

Library types follow the same matrix (per-type fallback to XML when SD
fails). On TIA V19, `sd` for LAD/FBD/STL automatically degrades to `.xml`
because `ExportAsDocuments` requires V20+.

### Compile workflow

After every export the extension can compile the target PLC and surface
results in the **PROBLEMS** panel:

- Compile errors from XML/SCL/SD imports are parsed and resolved to file +
  line in the workspace (`s7xmlErrorParser`, `s7dclErrorParser`).
- The `compileAfterExport` setting controls whether compilation runs
  automatically (`always`), prompts (`ask`), or is skipped (`never`).
- Errors persist in the PROBLEMS panel until the next compile/import.

### Smart-export behaviour Copilot must respect

- **Comparison-based overwrite** — exports diff against the current TIA
  object and skip blocks that match (timestamps + normalized content).
- **Orphan cleanup** — blocks/folders that exist in TIA but not in the
  workspace folder being exported are deleted in TIA. **Always export
  whole folders intentionally** — partial exports may delete unrelated
  blocks if a parent folder export is invoked.
- **Dependency order** — UDT → FB → FC → OB → DB on import, and a stable
  alphabetical order on export.
- **Know-how-protected blocks** — detected and skipped with a warning
  (do not regenerate them locally).
- **Instance DBs** are created via API from the parent FB, not by importing
  IDB XML. Hand-edited IDB `.xml` files may be ignored.

---

## Autonomous TIA Portal workflow (Language Model Tools)

The extension exposes **18 Language Model Tools** (prefix `tia_`) plus a
chat participant `@tia` (`tia.assistant`). When the user asks to import,
export, compile, or fix PLC code, **prefer these tools over manually
running commands or editing TIA-export files by hand**.

| Tool | Purpose |
|------|---------|
| `tia_connect`, `tia_disconnect` | Attach to / detach from TIA Portal |
| `tia_list_projects`, `tia_select_project` | Discover and pick the active project |
| `tia_list_devices`, `tia_list_blocks` | Enumerate devices and blocks (paging + name filter) |
| `tia_export_block`, `tia_export_device` | Pull blocks / whole device to local files under `TiaExport/` |
| `tia_export_project` | Export **every** device (programs + optional HW). Equivalent of UI `Import Entire Project`. |
| `tia_export_hw_config` | Pull HW config to `Devices/<Category>/<Device>/DeviceConfiguration/` (flat `Devices/IO_Devices/` for IO devices). Omit `device` for **project-wide** HW export. Honors `tiaImport.hwConfigFormat` (`xml` or `cax` AutomationML). |
| `tia_import_file`, `tia_import_folder` | Push local `.xml` / `.scl` / `.s7dcl` / `.db` / `.xlsx` / `.aml` into TIA Portal |
| `tia_import_hw_config` | Push HW Config `.xml` / `.aml` (file or folder) into TIA Portal. **Required** for HW — `tia_import_file` does not handle HW Config files. |
| `tia_refresh` | Re-read project structure from TIA after manual changes |
| `tia_compile` | Compile PLC software; populates the PROBLEMS panel |
| `tia_get_problems` | Snapshot of current TIA diagnostics as JSON `{file, line, severity, message}` |
| `tia_fix_compile_errors` | One step of the import → compile → diagnostics loop. Call repeatedly while `iterationsRemaining > 0` |

### Required workflow

1. Always start with `tia_connect`. If `currentProjectName` is empty, call
   `tia_list_projects` and then `tia_select_project`.
2. Resolve block / device ids via `tia_list_devices` and `tia_list_blocks`
   before exporting or fixing — **never guess ids or paths**.
3. After **every** import, run `tia_compile` and inspect `tia_get_problems`
   (or the `messages` field returned by `tia_compile`).
4. To autonomously fix compile errors, prefer `tia_fix_compile_errors`. It
   honours `tiaImport.lmTools.maxFixIterations` (default 5):
   - Call it with `importFolder` or `importFile`.
   - Read `diagnostics`. If any, edit the matching files in this workspace
     using the file/line numbers it returns.
   - Call it again — pass only `device` to recompile without re-importing
     unchanged files.
   - Stop when `compile.success === true` or `iterationsRemaining === 0`.
5. Imports that overwrite existing TIA objects trigger a confirmation
   dialog unless `tiaImport.lmTools.autoConfirmImports` is `true`. Do not
   try to bypass it.
6. When editing files for a fix, respect the format rules above:
   - `.s7dcl` → keep `.s7res` in sync (MLC ids).
   - File name (without extension) **must equal** the block name inside
     the file (`FUNCTION_BLOCK "Name"`, `<Name>...</Name>`).
   - Stay within the SimaticML XSD schemas in `.github/Schemas/`.
7. Prefer **smaller, targeted exports/imports** (single block or single
   subfolder) when fixing — avoid orphan cleanup on parent folders unless
   that is explicitly the intent.

### Chat participant

`@tia` (`tia.assistant`) is registered as a VS Code chat participant. When
the user addresses it, the same 13 tools are available plus a built-in
system prompt that enforces the rules above.

### Quick troubleshooting cues

- **"SD format requires V20+"** → user is on V19; switch `exportFormat` to
  `xml` or upgrade TIA Portal.
- **"Could not load file or assembly 'Siemens.Engineering.Base, Version=21.0.0.0'"**
  → wrong `tiaPortalVersion`; per-version wrapper binaries live under
  `dotnet/TiaOpennessWrapper/bin/Release/net48/V<n>/`.
- **Compile errors point to `.s7dcl` line numbers** → use
  `s7dclErrorParser` mappings; LAD/FBD blocks **cannot** be edited as SCL.
- **Tag-table import drops constants** → expected. The `Constants` sheet is
  preserved in `.xlsx` but TIA Openness has no `PlcUserConstant` import.
- **HW Config diff is noisy in `.xml` mode** → switch `hwConfigFormat` to
  `cax` for stable round-trips.

---
> Source: [cmariusz/TiaImportExport.VSExt](https://github.com/cmariusz/TiaImportExport.VSExt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
