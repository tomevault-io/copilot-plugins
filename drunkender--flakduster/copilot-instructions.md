## flakduster

> A comprehensive guide to XML Patch Operations in RimWorld modding.

# RimWorld Patch Operations

A comprehensive guide to XML Patch Operations in RimWorld modding.

## Table of Contents
- [Basics](#basics)
- [XPath](#xpath)
- [PatchOperation Types](#patchoperation-types)
- [Advanced Operations](#advanced-operations)
- [Tips and Tricks](#tips-and-tricks)
- [Common Issues](#common-issues--troubleshooting)
- [References](#references)

## Basics

PatchOperations are written as XML nodes that go into your mod's `Patches` folder:

```
MyModFolder/
├─ About/
├─ Defs/
└─ Patches/
   └─ MyPatchFile.xml
```

Just as with XML Defs, folder and file names do not matter and you can freely name and organize your patch files in whatever manner you wish. Individual patches themselves are a standard XML file with `<Patch>` as the root tag:

```xml
<?xml version="1.0" encoding="utf-8"?>
<Patch>
  <!-- PatchOperations go here -->
</Patch>
```

## XPath

Most PatchOperations must be targeted at one or more XML nodes inside the master XML document. This is done via [XML Path Language](https://developer.mozilla.org/en-US/docs/Web/XPath) or XPath.

Note that an XPath is targeting the structure of the XML document after it's been parsed by RimWorld's parser, thus it has nothing to do with the actual file or folder paths.

### Basic XPath Structure

For example, if you wanted to add a stat value to the vanilla Wall building, you might use an XPath like so:

```
Defs/ThingDef[defName="Wall"]/statBases
```

- The first segment of any XPath targeting an XML Def will be `Defs/` as all XML Defs use the `<Defs>` root tag.
- The square brackets denote a predicate, or conditional match. In this case, we are looking for ThingDefs that have a child tag `defName` equal to "Wall".

### Targeting Attributes

You can target Defs that do not have a `defName` (such as abstract bases) by targeting their identifying attributes. For example, if you wanted to add another Stuff category to the abstract base Def for all shelves:

```
Defs/ThingDef[@Name="ShelfBase"]/stuffCategories
```

You can use the same technique to target all Defs that inherit from a common base. For example, you could use the following xpath to target all ThingDefs that inherit from ApparelBase:

```
Defs/ThingDef[@ParentName="ApparelBase"]
```

### Additional XPath Resources

- [Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/XPath)
- [W3Schools XPath Tutorial](https://www.w3schools.com/xml/xml_xpath.asp)
- [StackOverflow XPath Tag](https://stackoverflow.com/questions/tagged/xpath)

## PatchOperation Types

### PatchOperationAdd

Inserts the specified values as a child node of the XML nodes targeted by the operation's xpath. By default, the new nodes will be inserted after any existing child nodes (Append). You can use `<order>Prepend</order>` in the patch operation to insert them before existing child nodes instead.

**Note:** PatchOperationAdd will not overwrite any existing tags. If one of your values overlaps with an existing node and you are not targeting a list node, then it will cause an error on game load.

#### Example

**Before:**
```xml
<ExampleDef>
  <defName>SampleDef</defName>
  <aaa>Some text</aaa>
</ExampleDef>
```

**Patch:**
```xml
<Operation Class="PatchOperationAdd">
  <xpath>Defs/ExampleDef[defName="SampleDef"]</xpath>
  <value>
    <bbb>New text</bbb>
  </value>
</Operation>
```

**After:**
```xml
<ExampleDef>
  <defName>SampleDef</defName>
  <aaa>Some text</aaa>
  <bbb>New text</bbb>
</ExampleDef>
```

#### List Example with Prepend

**Before:**
```xml
<ExampleDef>
  <defName>SampleDef</defName>
  <exampleList>
    <li>Bar</li>
  </exampleList>
</ExampleDef>
```

**Patch:**
```xml
<Operation Class="PatchOperationAdd">
  <xpath>Defs/ExampleDef[defName="SampleDef"]/exampleList</xpath>
  <order>Prepend</order>
  <value>
    <li>Foo</li>
  </value>
</Operation>
```

**After:**
```xml
<ExampleDef>
  <defName>SampleDef</defName>
  <exampleList>
    <li>Foo</li>
    <li>Bar</li>
  </exampleList>
</ExampleDef>
```

### PatchOperationInsert

Inserts the specified values as a sibling above the selected node(s). You can specify `<order>Append</order>` to insert it after the targeted node(s) instead (default is Prepend).

#### Example

**Before:**
```xml
<ExampleDef>
  <defName>Rainbow</defName>
  <colors>
    <li>Red</li>
    <li>Yellow</li>
    <li>Green</li>
    <li>Blue</li>
    <li>Violet</li>
  </colors>
</ExampleDef>
```

**Patch:**
```xml
<Operation Class="PatchOperationInsert">
  <xpath>Defs/ExampleDef[defName="Rainbow"]/colors/li[text()="Yellow"]</xpath>
  <value>
    <li>Orange</li>
  </value>
</Operation>
```

**After:**
```xml
<ExampleDef>
  <defName>Rainbow</defName>
  <colors>
    <li>Red</li>
    <li>Orange</li>
    <li>Yellow</li>
    <li>Green</li>
    <li>Blue</li>
    <li>Violet</li>
  </colors>
</ExampleDef>
```

### PatchOperationRemove

Removes the targeted node.

#### Example

**Before:**
```xml
<ExampleDef>
  <defName>Sample</defName>
  <foo>Uno</foo>
  <bar>Dos</bar>
  <baz>Tres</baz>
</ExampleDef>
```

**Patch:**
```xml
<Operation Class="PatchOperationRemove">
  <xpath>Defs/ExampleDef[defName="Sample"]/bar</xpath>
</Operation>
```

**After:**
```xml
<ExampleDef>
  <defName>Sample</defName>
  <foo>Uno</foo>
  <baz>Tres</baz>
</ExampleDef>
```

### PatchOperationReplace

Replaces the selected node(s) with your values.

#### Example

**Before:**
```xml
<ExampleDef>
  <defName>Sample</defName>
  <foo>Uno</foo>
  <bar>Dos</bar>
  <baz>Tres</baz>
</ExampleDef>
```

**Patch:**
```xml
<Operation Class="PatchOperationReplace">
  <xpath>Defs/ExampleDef[defName="Sample"]/baz</xpath>
  <value>
    <baz>Drei</baz>
  </value>
</Operation>
```

**After:**
```xml
<ExampleDef>
  <defName>Sample</defName>
  <foo>Uno</foo>
  <bar>Dos</bar>
  <baz>Drei</baz>
</ExampleDef>
```

### PatchOperationAttributeAdd

Adds the specified attribute to the targeted node(s), but only if that attribute is not yet present.

#### Example

**Before:**
```xml
<ExampleDef>
  <defName>Sample</defName>
  <foo>Uno</foo>
  <bar>Dos</bar>
  <baz>Tres</baz>
</ExampleDef>
```

**Patch:**
```xml
<Operation Class="PatchOperationAttributeAdd">
  <xpath>Defs/ExampleDef[defName="Sample"]</xpath>
  <attribute>Name</attribute>
  <value>SampleBase</value>
</Operation>
```

**After:**
```xml
<ExampleDef Name="SampleBase">
  <defName>Sample</defName>
  <foo>Uno</foo>
  <bar>Dos</bar>
  <baz>Tres</baz>
</ExampleDef>
```

### PatchOperationAttributeSet

Adds the specified attribute to the targeted node(s), or overwrites the existing value if the specified attribute already exists.

#### Example

**Before:**
```xml
<ExampleDef Name="SampleSource">
  <defName>Sample</defName>
  <foo>Uno</foo>
  <bar>Dos</bar>
  <baz>Tres</baz>
</ExampleDef>
```

**Patch:**
```xml
<Operation Class="PatchOperationAttributeSet">
  <xpath>Defs/ExampleDef[defName="Sample"]</xpath>
  <attribute>Name</attribute>
  <value>SampleBase</value>
</Operation>
```

**After:**
```xml
<ExampleDef Name="SampleBase">
  <defName>Sample</defName>
  <foo>Uno</foo>
  <bar>Dos</bar>
  <baz>Tres</baz>
</ExampleDef>
```

### PatchOperationAttributeRemove

Removes the specified attribute from the targeted node(s).

#### Example

**Before:**
```xml
<ExampleDef Name="SampleBase">
  <defName>Sample</defName>
  <foo>Uno</foo>
  <bar>Dos</bar>
  <baz>Tres</baz>
</ExampleDef>
```

**Patch:**
```xml
<Operation Class="PatchOperationAttributeRemove">
  <xpath>Defs/ExampleDef[defName="Sample"]</xpath>
  <attribute>Name</attribute>
</Operation>
```

**After:**
```xml
<ExampleDef>
  <defName>Sample</defName>
  <foo>Uno</foo>
  <bar>Dos</bar>
  <baz>Tres</baz>
</ExampleDef>
```

### PatchOperationAddModExtension

Adds the specified [DefModExtension](https://rimworldwiki.com/wiki/Modding_Tutorials/DefModExtension) to the targeted Def. Automatically creates a `<modExtensions>` node if one doesn't already exist.

#### Example

**Before:**
```xml
<ExampleDef Name="SampleBase">
  <defName>Sample</defName>
  <foo>Uno</foo>
  <bar>Dos</bar>
  <baz>Tres</baz>
</ExampleDef>
```

**Patch:**
```xml
<Operation Class="PatchOperationAddModExtension">
  <xpath>Defs/ExampleDef[defName="Sample"]</xpath>
  <value>
    <li Class="MyNamespace.MyModExtension">
      <key>Value</key>
    </li>
  </value>
</Operation>
```

**After:**
```xml
<ExampleDef Name="SampleBase">
  <defName>Sample</defName>
  <foo>Uno</foo>
  <bar>Dos</bar>
  <baz>Tres</baz>
  <modExtensions>
    <li Class="MyNamespace.MyModExtension">
      <key>Value</key>
    </li>
  </modExtensions>
</ExampleDef>
```

### PatchOperationSetName

Changes the name of a node. Most useful for changing the names of the nodes in "dictionary"-style nodes without changing their contents, such as for stat blocks or recipe products.

#### Example

**Before:**
```xml
<ThingDef>
  <defName>ExampleThing</defName>
  <!-- many nodes omitted -->
  <statBases>
    <Insulation_Cold>10</Insulation_Cold>
  </statBases>
</ThingDef>
```

**Patch:**
```xml
<Operation Class="PatchOperationSetName">
  <xpath>Defs/ThingDef[defName="ExampleThing"]/statBases/Insulation_Cold</xpath>
  <name>Insulation_Heat</name>
</Operation>
```

**After:**
```xml
<ThingDef>
  <defName>ExampleThing</defName>
  <!-- many nodes omitted -->
  <statBases>
    <Insulation_Heat>10</Insulation_Heat>
  </statBases>
</ThingDef>
```

## Advanced Operations

### PatchOperationSequence

Contains one or more additional PatchOperations, which are executed in order. If any of them fail, then the Sequence stops and will not run any additional PatchOperations.

**WARNING:** PatchOperations within an XML file will run in order even without a PatchOperationSequence and using a PatchOperationSequence can obfuscate or hide errors, making it difficult to debug if your child PatchOperations have errors. While PatchOperationSequence can be used to sequence multiple PatchOperations with a single PatchOperationConditional or PatchOperationFindMod, it is generally recommended to use [LoadFolders.xml](https://rimworldwiki.com/wiki/Modding_Tutorials/Mod_Folder_Structure) instead.

#### Example

```xml
<Operation Class="PatchOperationSequence">
  <operations>
    <li Class="PatchOperationAdd">
      <xpath>Defs/ExampleDef[defName="Sample"]/statBases</xpath>
      <value>
        <Mass>10</Mass>
      </value>
    </li>
    <li Class="PatchOperationSetName">
      <xpath>Defs/ExampleDef[defName="Sample"]/statBases/Flammability</xpath>
      <name>ToxicEnvironmentResistance</name>
    </li>
    <!-- etc -->
  </operations>
</Operation>
```

### PatchOperationFindMod

Checks whether any of the specified mods or DLCs are loaded and allows you to run an additional PatchOperation for match or nomatch results.

**WARNING:** Unlike all other mod-compatibility features in RimWorld, PatchOperationFindMod uses the mod name and not its packageId.

**NOTE:** For expansions, PatchOperationFindMod uses the label value defined in the ExpansionDef for the DLC. At present, this means they are identified by just their single word names: Royalty, Ideology, Biotech, Anomaly.

**NOTE:** PatchOperationFindMod should only be used if you are implementing optional compatibility for the target mod, i.e. if your mod can work with or without the mod in question. If you are creating a compatibility mod specifically for a target mod that would be meaningless without that mod present, then it's better to simply specify that mod in your About.xml as a dependency and forgo the use of PatchOperationFindMod.

#### Example

The following example adds a mod extension to the targeted Defs if RimQuest is loaded:

```xml
<Operation Class="PatchOperationFindMod">
  <mods>
    <li>RimQuest</li>
  </mods>
  <match Class="PatchOperationAddModExtension">
    <xpath>Defs/IncidentDef[defName="MFI_DiplomaticMarriage" or defName="MFI_HuntersLodge" or defName="MFI_Quest_PeaceTalks"]</xpath>
    <value>
      <li Class="RimQuest.RimQuest_ModExtension">
        <canBeARimQuest>false</canBeARimQuest>
      </li>
    </value>
  </match>
</Operation>
```

The following example replaces the tabWindowClass of the Factions button when Relations Tab is not loaded:

```xml
<Operation Class="PatchOperationFindMod">
  <mods>
    <li>Relations Tab</li>
  </mods>
  <nomatch Class="PatchOperationReplace">
    <xpath>/Defs/MainButtonDef[defName="Factions"]/tabWindowClass</xpath>
    <value>
      <tabWindowClass>MyNameSpace.MyTabWindowClass</tabWindowClass>
    </value>
  </nomatch>
</Operation>
```

### PatchOperationConditional

Tests for the existence/validity of the specified node(s) and allows you to run optional match or nomatch PatchOperations in response.

#### Example

The following example adds a `<comps>` node to the Caravan Def if it does not exist already, then adds a custom comp to said list:

```xml
<?xml version="1.0" encoding="utf-8"?>
<Patch>
  <!-- add comps field to Caravan WorldObjectDef if it doesn't exist -->
  <Operation Class="PatchOperationConditional">
    <xpath>Defs/WorldObjectDef[defName="Caravan"]/comps</xpath>
    <nomatch Class="PatchOperationAdd">
      <xpath>Defs/WorldObjectDef[defName="Caravan"]</xpath>
      <value>
        <comps />
      </value>
    </nomatch>
  </Operation>
  
  <!-- add pyromaniac caravan handler comp to Caravan WorldObjectDef -->
  <Operation Class="PatchOperationAdd">
    <xpath>Defs/WorldObjectDef[defName="Caravan"]/comps</xpath>
    <value>
      <li Class="BetterPyromania.WorldObjectCompProperties_Pyromania">
        <fuelCount>20</fuelCount>
        <cooldown>30000</cooldown>
        <needThreshold>0.5</needThreshold>
      </li>
    </value>
  </Operation>
</Patch>
```

### Custom PatchOperations

Custom PatchOperation types can be created in C# by subclassing `Verse.PatchOperation`, which is useful for performing a patch based on ModSettings values or other custom behavior.

A tutorial for this process does not exist yet, but you can check out the following references of custom PatchOperations:

- **XML Extensions** - An entire framework mod containing many useful custom PatchOperations. Please note that this framework uses the viral [GVPLv3 License](https://www.gnu.org/licenses/gpl-3.0.en.html), meaning that if your mod makes use of the framework it must also be licensed under GPLv3.
- [PatchOperationMakeGunCECompatible](https://github.com/CombatExtended-Continued/CombatExtended/blob/Development/Source/CombatExtended/CombatExtended/PatchOperationMakeGunCECompatible.cs) - Used by Combat Extended to automatically apply multiple changes to a single firearm for compatibility.
- [PatchOperationAddOrReplace](https://gist.github.com/Lanilor/e326e33e268e78f68a2f5cd3cdbbe8c0) - An example of a custom variant of PatchOperationAdd that replaces an existing value.

### "Success" Option

The `<success>...</success>` node determines how errors are handled, usually in a PatchOperationSequence. Note that use of this tag should be considered obsolete; its use was common before PatchOperationConditional was implemented, but it should no longer be required and can cause confusion as it can suppress legitimate errors from showing up.

Options available are:
- **Always** - This patch operation always succeeds, i.e. it suppresses all errors that might have occurred. This used to be used in PatchOperationSequence along with PatchOperationTest in order to conditionally run patches, but is now obsolete.
- **Normal** - Errors are handled normally
- **Invert** - Errors are considered a success and success is considered a failure. This used to be used in PatchOperationSequences to test the negative of a PatchOperationTest; you should now use `<nomatch>` on PatchOperationConditional.
- **Never** - This patch operation always fails. Generally only used in testing to see if a Sequence is working correctly, should never be used in published mods.

## Tips and Tricks

- **Load Order:** Patching is run after all XML Defs are loaded into memory and in mod list order. If you are encountering compatibility issues with another mod's PatchOperations, you can use `loadBefore` and `loadAfter` in your About.xml to help players resolve these issues.

- **Inheritance Timing:** Patching is done before Def inheritance takes place. This means that you cannot target tags that are inherited from a parent, but also that if you alter a parent, all of its children will inherit the patched value.

- **Override Inherited Values:** You can override a value inherited from a parent Def without changing that value for all Defs by using `Inherit="False"`:

```xml
<Operation Class="PatchOperationAdd">
  <xpath>/Defs/ThingDef[defName = "Apparel_KidPants"]</xpath>
  <value>
    <thingCategories Inherit="False">
      <li>NewValue</li>
    </thingCategories>
  </value>
</Operation>
```

- **Multiple Targets:** You can use `or` in predicates to target multiple nodes: `Defs/ThingDef[defName="Cassowary" or defName = "Emu" or defName = "Ostrich" or defName = "Turkey"]` This is generally better than using multiple PatchOperations so long as you are making the same changes to each target.

- **Target Text Content:** You can use `/text()` to target the text contents of a tag instead of the whole tag for operations like PatchOperationReplace. This is especially useful if you do not want to accidentally remove attributes.

## Common Issues / Troubleshooting

- **Case Sensitivity:** XPath and XML nodes in general is case-sensitive and must be exact. Copying values such as `defName` fields is strongly recommended to avoid errors.

- **Malformed XML:** Malformed XML such as unclosed or mismatched tags can cause the XML parser to crash completely, which can result in a blank screen on startup or a "Recovered from a fatal error" screen and the removal of all active mods. If this occurs, run all of your XML through an XML validator to ensure that it is structurally correct. Checking your Player.log file can help in diagnosing the exact cause as well.

- **Insert vs Add Confusion:** It's very easy to confuse Insert vs Add; Insert will "add" the value as a sibling of the target node(s), while Add will "insert" the value as a child of the target node(s).

- **XPath vs File Path:** Remember that XPath targets the XML data structure, not the file path.

- **Duplicate Nodes:** Nodes in XML other than `li` list items must be unique at each level. If you Add or Insert a duplicate node, that will generate a red error on load and cause that Def to fail to load.

- **Getting Help:** If you're still stumped, feel free to join the [RimWorld Discord server](https://discord.gg/RimWorld) and ask questions in the #mod-development channel.

## References

- Original tutorial on PatchOperations created by Zhentar: [Introduction to PatchOperation](https://gist.github.com/Zhentar/4a1b71cea45b9337f70b30a21d868782)
- RimWorld Wiki: [Modding Tutorials/PatchOperations](https://rimworldwiki.com/wiki/Modding_Tutorials/PatchOperations)
- RimWorld Discord: [#mod-development](https://discord.gg/RimWorld)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Drunkender) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
