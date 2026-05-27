## 000-xml-cursor-rules

> Use ALWAYS when asked to CREATE A RULE or UPDATE A RULE or taught a lesson from the user that should be retained as a new rule for Cursor


<?xml version="1.0" encoding="UTF-8"?>
<cursor-rule>
  <title>Cursor Rules Format</title>
  <file-organization>
    <location>
      <path>.cursor/rules/</path>
      <extension>.mdc</extension>
    </location>
    
    <naming-convention>
      <pattern>PREFIX-name.mdc</pattern>
      <prefixes>
        <prefix code="0XX">Core standards</prefix>
        <prefix code="1XX">Tool configs</prefix>
        <prefix code="3XX">Testing standards</prefix>
        <prefix code="4XX">Documentation standards</prefix>
        <prefix code="1XXX">Language rules</prefix>
        <prefix code="2XXX">Framework rules</prefix>
        <prefix code="8XX">Workflows</prefix>
        <prefix code="9XX">Templates</prefix>
        <prefix code="__XXX_name.mdc">Private rules</prefix>
      </prefixes>
    </naming-convention>

    <glob-patterns>
      <pattern type="core-standards">.cursor/rules/*.mdc</pattern>
      <pattern type="language-rules">src/**/*.{js,ts}</pattern>
      <pattern type="testing-standards">**/*.test.{js,ts}</pattern>
      <pattern type="react-components">src/components/**/*.tsx</pattern>
      <pattern type="documentation">docs/**/*.md</pattern>
      <pattern type="configuration">*.config.{js,json}</pattern>
      <pattern type="build-artifacts">dist/**/*</pattern>
      <pattern type="multiple-extensions">src/**/*.{js,jsx,ts,tsx}</pattern>
      <pattern type="multiple-files">dist/**/*, docs/**/*.md</pattern>
    </glob-patterns>
  </file-organization>

  <required-fields>
    <frontmatter>
      <field name="description">ACTION TRIGGER OUTCOME format</field>
      <field name="globs">glob pattern for files and folders</field>
    </frontmatter>
    
    <body-requirements>
      <requirement>version tag in format X.Y.Z</requirement>
      <requirement>context: Usage conditions</requirement>
      <requirement>requirements: Actionable items</requirement>
      <requirement>examples: Both valid and invalid</requirement>
    </body-requirements>
  </required-fields>

  <formatting-guidelines>
    <guideline>Always indent content within XML or nested XML tags by 2 spaces</guideline>
    <guideline>Keep rules as short as possible</guideline>
    <guideline>Use Mermaid syntax if it will be shorter or clearer than describing a complex rule</guideline>
    <guideline>Use Emojis where appropriate to convey meaning that will improve rule understanding by the AI Agent</guideline>
    <guideline>Keep examples as short as possible to clearly convey the positive or negative example</guideline>
  </formatting-guidelines>

  <ai-optimization>
    <tips>
      <tip>Use precise, deterministic ACTION TRIGGER OUTCOME format in description</tip>
      <tip>Provide concise positive and negative example of rule application in practice</tip>
      <tip>Optimize for AI context window efficiency</tip>
      <tip>Remove any non-essential or redundant information</tip>
      <tip>Use standard glob patterns without quotes (e.g., *.js, src/**/*.ts)</tip>
    </tips>

    <context-efficiency>
      <guideline>Keep frontmatter description under 120 characters while maintaining clear intent</guideline>
      <guideline>Limit examples to essential patterns only</guideline>
      <guideline>Use hierarchical structure for quick parsing</guideline>
      <guideline>Remove redundant information across sections</guideline>
      <guideline>Maintain high information density with minimal tokens</guideline>
      <guideline>Focus on machine-actionable instructions over human explanations</guideline>
    </context-efficiency>
  </ai-optimization>

  <critical-rules>
    <rule>The front matter can ONLY have the fields description and globs</rule>
    <rule>The file MUST start with the this exactly:
      ---
      description: front matter description
      globs: front matter glob pattern or no value
      ---
    </rule>
    <rule>All contents after the frontmatter will be styled as XML
    <rule>NEVER include verbose explanations or redundant context that increases AI token overhead</rule>
    <rule>Keep file as short and to the point as possible BUT NEVER at the expense of sacrificing rule impact and usefulness for the AI Agent</rule>
  </critical-rules>
</cursor-rule>

---
> Source: [bmadcode/cursor-xml-rules-trial](https://github.com/bmadcode/cursor-xml-rules-trial) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
