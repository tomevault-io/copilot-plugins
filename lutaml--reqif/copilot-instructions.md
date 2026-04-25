## reqif

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a Ruby gem (`reqif`) that parses and generates ReqIF (Requirements Interchange Format) XML documents. ReqIF is an OMG standard (v1.2) for exchanging requirements between different tools. The gem is part of the `lutaml` family.

## Commands

```bash
bundle install          # Install dependencies
bundle exec rake        # Run specs + rubocop (default)
bundle exec rake spec   # Run specs only
bundle exec rake rubocop  # Run linter only
bundle exec rspec spec/reqif_spec.rb  # Run a single spec file
bundle exec bin/console # Start an IRB console with the gem loaded
```

## Architecture

### Framework: lutaml/model

All model classes inherit from `Lutaml::Model::Serializable` and use an `xml` DSL block to define XML serialization. The gem uses `Lutaml::Model::XmlAdapter::NokogiriAdapter` for XML parsing/generation.

### Class-per-file pattern

Each ReqIF concept is a separate class in `lib/reqif/`. Classes follow a consistent structure:

```ruby
module Reqif
  class Foo < Lutaml::Model::Serializable
    attribute :bar, :string
    attribute :baz, SomeOtherClass

    xml do
      element "FOO"
      namespace "http://www.omg.org/spec/ReqIF/20110401/reqif.xsd"
      map_attribute "BAR", to: :bar
      map_element "BAZ", to: :baz
    end
  end
end
```

### Key model classes

- `ReqIf` — Root element, contains `THE-HEADER`, `CORE-CONTENT`, `TOOL-EXTENSIONS`
- `ReqIfContent` — Contains `DATATYPES`, `SPEC-TYPES`, `SPEC-OBJECTS`, `SPEC-RELATIONS`, `SPECIFICATIONS`, `SPEC-RELATION-GROUPS`
- `SpecObject` — A single requirement with attributes (identifier, long_name, desc, etc.) and `VALUES`
- `Specification` — A container/hierarchy that has `CHILDREN` referencing other specs
- `SpecRelation` — A relation between two specs with `SOURCE` and `TARGET` references
- `Values` — Container for `AttributeValue` elements (boolean, date, enumeration, integer, real, string, xhtml)
- `Type` / `SpecObjectType` / `SpecificationType` — Type definitions for spec objects/specifications
- `Datatypes` — Container for datatype definitions (boolean, date, enumeration, integer, real, string, xhtml)

### Attribute naming conventions

XML element names use uppercase-kebab-case (e.g., `SPEC-OBJECT`, `LAST-CHANGE`) while Ruby attributes use snake_case (e.g., `spec_object`, `last_change`). The `map_element` and `map_attribute` DSL methods handle the translation.

### Autoloading

Zeitwerk is used for autoloading. `lib/reqif.rb` sets up the loader and eager loads all classes.

### XML round-trip testing

Round-trip tests in `spec/reqif_spec.rb` parse fixture XML files, regenerate XML via `to_xml`, and compare canonicalization using `xml-c14n`. All fixture files under `spec/fixtures/` (`.xml`, `.reqif`) are automatically included.

### Style

- String literals: double quotes enforced by RuboCop
- Ruby version: 3.0+
- Frozen string literals: required at the top of every file

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lutaml) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
