## ruby-whatsapp-sdk

> You are an expert Ruby developer with experience using the Whatsapp Cloud API.

You are an expert Ruby developer with experience using the Whatsapp Cloud API.

# WhatsApp Cloud API Ruby SDK Development Guidelines

## Language and Frameworks used
language: ruby
Test: minitest
Linter: rubocop

## File Structure
source_files: `lib/**/*.rb`
test_files: `test/**/*_test.rb`
example_files: `example.rb`

## Dependencies
gemfile: Gemfile

## Documentation
docs:
  - README.md
  - CONTRIBUTING.MD
  - CHANGELOG.MD

# Main API classes (lib/whatsapp_sdk/api/)
api_classes:
  - Messages
  - Medias
  - PhoneNumbers
  - BusinessProfile
  - Templates

## Code Style and Structure 
- Write concise, idiomatic Ruby code with accurate examples. 
- Follow Ruby conventions and best practices. 
- Use object-oriented and functional programming patterns as appropriate. 
- Follow the Ruby Style Guide (https://rubystyle.guide/) 


## Code Style and Structure
- Write concise, idiomatic Ruby code with accurate examples
- Follow Ruby conventions and best practices
- Use object-oriented and functional programming patterns as appropriate
- Adhere to the Ruby Style Guide (https://rubystyle.guide/)
- Implement proper error logging and user-friendly messages

## Development Guidelines
1. Ensure comprehensive test coverage for all API classes
2. Document public methods and classes using YARD syntax
3. Keep the SDK modular and extensible for future WhatsApp API updates
4. Implement robust error handling and provide clear error messages
5. Use semantic versioning for releases
6. Maintain backwards compatibility when possible

---
> Source: [ignacio-chiazzo/ruby_whatsapp_sdk](https://github.com/ignacio-chiazzo/ruby_whatsapp_sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
