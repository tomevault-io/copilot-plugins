## typesafe-ai-php

> This is a community-maintained PHP client for the TypeSafe AI evaluation API.

# AI Agent Guidelines

This is a community-maintained PHP client for the TypeSafe AI evaluation API.

It is designed to be type-safe and easy to use: requests are built with a fluent interface, and each answer type has a typed accessor.

- **PHP Version:** 8.2 or newer.
- **API reference:** https://docs.typesafe.ai (`POST /v1/systemone`, and `GET /v1/models`, which the published reference does not document).
- **Core Architecture:**
    - **Client:** `TypeSafeClient` sends requests and deserializes responses.
    - **Request:** `SystemOneRequest` retains the state and a map of `Question` DTOs (`Noul`, `Choice`, `Score`). The client serializes it with JMS.
    - **Response/DTOs:** `SystemOneResult` retains a map of `Answer` DTOs. JMS selects the subclass using the `type` field.
    - **Models:** `TypeSafeClient::models()` returns a `ModelsResponse` of `DTO\ModelCard`.
    - **Attributes:** `Noul`, `Choice`, and `Score` are PHP attributes too, so a result class can declare its own questions. `TypeSafeClient::evaluate()` is the only entry point. It builds a single `AttributeReader` for the class, sends the questions that the reader parses from the constructor, and then hydrates the class with `hydrate()`. The request and the result know nothing about attributes: keep the attribute logic in the reader and its one caller.

End-user documentation:

@README.md

## Project Navigation

**Key locations:**
- **Core Logic:** @src/TypeSafeClient.php (the main entry point for all API calls).
- **Request:** `src/SystemOneRequest.php` and `src/Question/`.
- **Data Models:** `src/SystemOneResult.php`, `src/ModelsResponse.php`, and `src/DTO/` (all response objects).
- **Tests:** `tests/`, with response fixtures in `tests/data/`.

## Coding Standards

- **Type Hinting:** Use precise type hints for parameters and return types. Use generics (`@template`) where appropriate.
- **DTOs:** Data Transfer Objects are simple classes with public properties.
- **Little logic in DTOs:** Requests and responses are mostly pure data: public properties. Put serialization rules in JMS attributes where possible. When a rule applies to one DTO only, a small serialization hook on that DTO (see `NoulCriteria::descriptions()`) is better than a global strategy in the serialization context of the client. The request builder methods forward their arguments unchanged.
- **No validation:** Question types do not validate their contents; the API does, so the SDK continues to operate when the API relaxes a rule.
- **Naming:** Follow PER-CS coding standards (extended PSR-12). Run `make cs` to validate.

## Implementation Details

- **JSON maps**: The API uses maps keyed by ids and options that you choose. Declare them with a key type, such as `#[Type('array<string, string>')]`: JMS then writes a JSON object, also when the map is empty or has keys such as `"0"`. Declare lists as `array<string>`; JMS re-indexes them.
- **Free-form JSON values**: Instructions and criteria can be text, a JSON object, an array, or null. Declare such a value as `union`, as in `#[Type('array<string, union>')]` for a map and `#[Type('array<union>')]` for a list: JMS dispatches on the runtime type and does not modify the value. A value type of `string` stringifies nested data, `mixed` is not a JMS type and throws, and omitting the type re-indexes numeric keys. `union` works only when serializing; use a bare `#[Type('array')]` for a free-form value that is also deserialized, as `ScoreAnswer::$legend` does.
    - `union` is the name JMS gives a PHP union property itself (`TypedPropertiesDriver`), and `UnionHandler` is registered for that name. Writing the name by hand is not a documented feature, so treat it as load-bearing on JMS internals: `serializeUnion()` ignores the type parameters and dispatches on the runtime type, which is what makes it a pass-through, while `deserializeUnion()` reads them and throws when they are absent.
    - The tripwire is `SystemOneRequestTest::provideRequests()`. Its structured cases assert the exact serialized JSON, so a JMS upgrade that changes this fails the build rather than quietly sending `"Array"` in place of a nested description. Keep those cases when you modify the provider.
- **Nulls**: The client serializes with `serializeNull` enabled, so all null values are sent: in arrays (a choice option without a description) and in the user state. If a DTO has optional fields that the API must not receive as null, the DTO omits them itself: exclude the properties and add an inline virtual property that returns only the values that are set (see `NoulCriteria::descriptions()`). To omit a nested object that yields no values, add `#[SkipWhenEmpty]` (see `Noul::$criteria`).
- **Question type field**: Each question class has a `public string $type` property with a default value. There is no discriminator on requests, so a custom question type does not need a change in the SDK.
- **Attribute reading**: `AttributeReader` defers execution until a caller consumes it. `questions()` parses the question attributes, `hydrate()` inspects the parameter types, and neither duplicates the other. Each therefore rejects only the defects that it evaluates: a parameter without a single question attribute stops `evaluate()` before the request, and a parameter that is not typed as an `Answer` stops it after the response. Do not add a pass that validates both in advance. The tests enforce this: `testConstructorReadsNothing`, `testQuestionsDoNotReadTheAnswerType`, and `testHydrationDoesNotReadTheQuestions` inject a mocked `ReflectionClass` and fail when the reader evaluates more than it requires, and `testInvalidClass` drives both reads and fails when either stops rejecting its own case.
- **Answer types**: `DTO\Answer` has a JMS `#[Discriminator]` on the `type` field. To add an answer type, add a subclass, a map entry, and an accessor on `SystemOneResult`.
- **JMS attributes**: Use PHP attributes such as `#[Type(...)]` for JMS serializer metadata. Keep PHPDoc like `@var` where it provides static-analysis detail.
- **Serializer property names**: The JSON serializer uses JMS' `IdenticalPropertyNamingStrategy`, so DTO property names must match API field names unless a `#[SerializedName(...)]` override is added.
- **Retries**: `408`, `429`, and every `5xx` response, plus connection timeouts, are retried twice by `GuzzleRetryMiddleware`. Other HTTP errors throw Guzzle exceptions.
- **Environment**: `createInstance()` defaults to `TYPESAFE_API_KEY` and `TYPESAFE_BASE_URL`, the names the other SDKs use. There is no default-model variable: the model is a property of the request.

## Development Workflow

1. **Code Standardization**: Always run `make cs` before submitting changes for review. This ensures style compliance (PER-CS), applies modern PHP standards, removes unused imports, and maintains project-wide structural consistency.
2. **Full Verification**: Run `make -j -k` to execute the complete validation pipeline in parallel and identify all failures at once. This typically includes:
    - Coding style and linting.
    - Static analysis.
    - Unit and functional tests, with 100% code coverage.
    - Mutation testing.
    - Package and configuration validation.
    *Refer to the output of `make -j -k` for the exact tools and current configurations.*
3. **Testing Requirement**: Every new feature or bug fix must be accompanied by a corresponding test in the `tests/` directory.
    - To run a single test file while iterating: `vendor/bin/phpunit tests/SpecificTest.php`.
    - Data providers run before coverage is collected. If a provider builds the object under test, yield a closure and call it in the test.
4. **Mocking**: Client tests use the real `createInstance()` and replace the handler of its stack with a Guzzle `MockHandler`; see `tests/TypeSafeClientTest.php`.
5. **Fixtures**: Put response JSON in `tests/data/`. `SerializationTest` checks that each file deserializes and then serializes to identical JSON. It selects the class by filename prefix from `PREFIX_CLASS_MAP`; a file with no matching prefix is reported as incomplete rather than failing, so add the prefix with the fixture.

The build system uses `chronic` to suppress output for successful commands; if a command produces no output, it has succeeded.

## Documentation Style

- **No Hard-Wrapped Lines:** Write each paragraph as a single long line in Markdown files. Let the editor apply soft-wrapping.
- **Clarity:** Keep documentation concise and focused on usage examples, following ASD-STE100 guidelines.

---
> Source: [sanmai/typesafe-ai-php](https://github.com/sanmai/typesafe-ai-php) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
