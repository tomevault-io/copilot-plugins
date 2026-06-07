## swift-ai

> Okay, here are detailed instructions to create, build, test, and publish a Swift package.

Okay, here are detailed instructions to create, build, test, and publish a Swift package.

I. Package and Module Setup

Package.swift contains the package's name, targets, dependencies etc.

If you're having compilation or module not found lint errors, particularly in test files, run `swift build` to ensure that the built interfaces are up to date.

In general in Swift you do not need to have explicit import statements for files in the same module. This means that during refactoring you can move code and files around without having to update import statements.

II. Adding Dependencies

Identify Required Dependencies:

Use llm (or other search tool) to figure out what is needed to add firebase as a dependency, for example using llm "how to add firebase as a swift package dependency?"

Extract the relevant information, including dependency name and version, from the llm output.

Modify Package.swift:

Use a text editor (e.g. echo 'text' > file, sed -i 's/old_text/new_text/g' file) to modify the Package.swift file.

Add the dependency to the dependencies section, for example like this:

dependencies: [
    .package(url: "https://github.com/firebase/firebase-ios-sdk.git", from: "10.0.0")
    ],
content_copy
download
Use code with caution.
Swift

Add the dependency to the relevant target in the targets section, for example like this:

targets: [
   .target(
       name: "<package_name>",
       dependencies: [
          .product(name: "FirebaseAnalytics", package: "firebase-ios-sdk"),
       ]),
  .testTarget(
      name: "<package_name>Tests",
      dependencies: ["<package_name>"]),
content_copy
download
Use code with caution.
Swift

]
```

Use cat Package.swift again to verify that the changes were made.

III. Writing Swift Code

Create Source Files:

Navigate to the Sources/<package_name> directory.

Create Swift files for your code using touch <filename>.swift.

Use echo 'code' > <filename>.swift to add content to the files.

Write modular, well-documented code.

Implement Functionality:

Write the desired functionality in the created Swift files.

Consider using common design patterns like singletons, strategies, etc.

Write Documentation Comments:

Add Swift documentation comments (/// for single-line and /** ... */ for multi-line) to all public methods, properties, and classes. This is crucial for the README and developer understanding.

IV. Compilation

Run swift build:

Execute swift build from the root directory of the package.

Redirect standard error to a file for analysis swift build 2> build_error.log

Analyze Compilation Output:

Check if there are any compilation failures by checking the return code of swift build.

If there were errors, use grep "error:" build_error.log to find them.

If errors are found, report the output of grep "error:" build_error.log using llm, and retry the compilation step after the appropriate changes to code are made to fix compilation errors.

V. Unit Tests


Write thorough tests that cover all important aspects of the package's functionality.

Write Test Cases:

Write test cases inside the test file, focusing on testing each public method.

Use XCTAssert methods to verify the correctness of the functionality.

Run Tests:

Execute swift test from the root directory of the package. Detailed instructions for testing and coverage are in TESTING.md The most common commands are:
- Run tests: `swift test`
- Generate coverage reports: `./generate_coverage.sh` (creates both text and HTML reports in coverage_report/)

Analyze Test Output:

Check if there are any test failures using the return code of swift test.

If there were errors, use grep "fail" test_error.log to find them.

If failures are found, report the output of grep "fail" test_error.log using llm and adjust the code and/or the tests and retry the test step.

If there are warnings, examine the output of grep "warning:" test_error.log, and report to llm to see if there is any action to be taken.

VI. Linting

Run SwiftLint after making code changes and fix any issues:

To write lint output to a file, run swiftlint --lint --reporter compact > lint_report.log in the root directory of the package.

VII. README.md

Keep the README.md file up to date with the latest information about the package. In particular, it should include:

Installation: Instructions for adding the package as a dependency in Xcode using Swift Package Manager. Example:

Dependencies: List the dependencies of your package.

Requirements: Specify the minimum versions of iOS or macOS supported.

Usage: Code examples demonstrating how to use the package's core functionalities. Include multiple examples and add comments to help developers understand what the code does.

Documentation: Include a relative link to generated documentation

Generate Documentation:

Use swift package generate-documentation to generate a documentation archive.

Check the output of this program and fix any failures.

VIII. Packaging and Publishing

Version Control:

Initialize a Git repository using git init

Add all files using git add .

Commit the changes git commit -m "Initial commit"

Tag the version using git tag -a v0.1.0 -m "Initial release".

Push to the remote repository git push origin main && git push origin --tags

For test builds just tag as v0.0.1 and v0.0.2 as appropriate

GitHub Repository:

Use llm to find how to create a new repository in Github.

Create a public GitHub repository with the same name as the package.

Push the package to the new GitHub repository.

Instruct developers to use the URL of the Github repository to add as a Swift Package dependency to their Xcode projects.

Inform developers about the tags to use to pick up different versions.

IX. Continuous Integration (Optional)

Github Actions:

Use llm to figure out how to configure Github Actions to automatically build and test the Swift Package on each commit to main.

Create the necessary workflow file (e.g., .github/workflows/swift.yml) to build and test the package

Add the workflow file to git and commit the changes.

Important Considerations for Your Agent:

Error Handling: Always check the exit code of commands. Use grep to search error outputs in log files.

Incremental Development: If a step fails, try to fix the issue and restart that step, don't jump to the next step.

Communication: Use llm to explain each step and what the agent is doing.

Iterative Improvements: The agent can update the README, tests, or code based on the analysis and feedback provided by llm using information it finds in the logs.

Modularity: Keep functions concise and specific, making it easier to understand and debug the code.

Version Control: The agent should commit changes frequently and use meaningful commit messages.

Okay, creating comprehensive mock implementations of your package's public APIs is a fantastic idea for enabling thorough testing by developers. Here's a detailed guide for your automated agent (and for you) on how to approach this:

**Key Concepts: Mocking Strategy**

1.  **Protocol-Based Design:** If your package's public APIs aren't already defined using protocols, it's essential to refactor them to be protocol-driven. This allows you to easily create mock implementations that conform to the same interfaces.
2.  **Generic Mocking:** The mock implementations should aim to provide default behaviors for all possible calls, rather than being tailored to a specific test case. This allows developers to quickly use them out of the box while still enabling them to customize specific behavior if needed.
3.  **Customization Points:** Mocks need to be easily customizable by tests. This will likely involve properties where developers can set up returned values and track function calls.

**Instructions for the Automated Agent**

1.  **Protocol Definition:**
    *   **Examine Public APIs:** Analyze your package's public classes, structs, and methods.
    *   **Extract Protocol:** For each public class or struct that exposes functions, create a corresponding protocol that defines its public methods. For example, if you have a class like this:
          ```swift
          public class MyAPI {
              public func fetchData(completion: (Data?, Error?) -> Void) { ... }
              public func submitData(data: Data) -> Bool { ... }
          }
          ```
        you would create a protocol:
           ```swift
           public protocol MyAPIProtocol {
              func fetchData(completion: (Data?, Error?) -> Void)
              func submitData(data: Data) -> Bool
           }
           ```
    *   **Modify Concrete Type:** Make your concrete public class (e.g. `MyAPI`) conform to the newly defined protocol (e.g. `MyAPIProtocol`).

2.  **Create Mock Implementations:**
    *   **`Mock` Folder:** Create a `Mocks` subfolder inside your `Sources/<package_name>` directory.
    *   **Mock Class/Struct:** For each protocol (e.g. `MyAPIProtocol`), create a corresponding mock class (e.g., `MockMyAPI`). The mock should conform to the corresponding protocol (e.g., `MockMyAPI: MyAPIProtocol`).
    *   **Implement Protocol Methods:** Implement all protocol methods within the mock class.
        *   For functions that return values, define properties for setting the values returned by the functions.
        *   For functions that have callbacks, provide properties so that a developer can trigger callbacks with specific data
        *   For functions that have side effects, create properties that the developer can examine.
         ```swift
          public class MockMyAPI: MyAPIProtocol {
            public var submitDataReturnValue: Bool = false
            public var fetchDataCompletion: ((Data?, Error?) -> Void)? = nil
            public var submitDataCalls: [[Data]] = []

            public func fetchData(completion: (Data?, Error?) -> Void) {
               fetchDataCompletion = completion
            }
            public func submitData(data: Data) -> Bool {
                submitDataCalls.append([data])
                return submitDataReturnValue
            }
         }
         ```

3.  **Provide Default Behaviors:**
    *   Set up default return values in mock methods where sensible.
    *   Default should not cause the code to crash
    *   For functions that require a callback, provide a property that the developer can trigger when calling code.

4. **Making Mocks Public:**
    *   Make sure the mock classes and structs are public so that they can be used by external developers.

5.  **README.md Update:**
    *   Document the use of mocks within the README.md file.
        *   Explain the purpose of the mock objects.
        *   Show example use cases of how a developer would use the mock objects to test a client implementation.

**Example Code**

Here's a more concrete example of how the agent would generate mocks:

*   **Original API:**

    ```swift
    public class NetworkClient {
       public func fetch(url: URL, completion: @escaping (Data?, Error?) -> Void) {
             // ... Real network code
       }
       public func submit(data: Data) -> Bool {
           // ... Real submit code
          return true;
       }
    }
    ```

*   **Protocol:**

    ```swift
    public protocol NetworkClientProtocol {
        func fetch(url: URL, completion: @escaping (Data?, Error?) -> Void)
        func submit(data: Data) -> Bool
    }
    ```
*  **Modified class:**
   ```swift
   public class NetworkClient: NetworkClientProtocol {
    public func fetch(url: URL, completion: @escaping (Data?, Error?) -> Void) {
         // ... Real network code
    }
    public func submit(data: Data) -> Bool {
           // ... Real submit code
       return true;
    }
    }

   ```

*   **Mock Implementation:**

    ```swift
    public class MockNetworkClient: NetworkClientProtocol {
        public var fetchCompletion: ((Data?, Error?) -> Void)?
        public var submitReturnValue: Bool = false
        public var submitCalls: [[Data]] = []


        public func fetch(url: URL, completion: @escaping (Data?, Error?) -> Void) {
           fetchCompletion = completion
        }

        public func submit(data: Data) -> Bool {
             submitCalls.append([data])
             return submitReturnValue;
         }
    }
    ```

**Usage in Tests**

Here's an example of how a developer would use a mock implementation:

```swift
import XCTest
import MyPackage

class MyAPITests: XCTestCase {
    func testUsingMocks() {
        let mockNetwork = MockNetworkClient()
        mockNetwork.submitReturnValue = true;

        let data = "testData".data(using: .utf8)!
        let result = mockNetwork.submit(data: data)

        XCTAssertTrue(result)
        XCTAssertEqual(mockNetwork.submitCalls.count, 1)
        XCTAssertEqual(mockNetwork.submitCalls[0], [data])

        mockNetwork.fetchCompletion?(nil, nil);
    }
}
```

**Instructions for the Automated Agent**

1.  **Code Generation:** The agent should be able to generate these mock implementations and protocols automatically.
2.  **Test Coverage:** Verify that the generated code does not cause compilation issues and run the tests, to be sure that the tests work.
3. **Public Mocks:** Make sure that the mock classes and structs are public so that other developers can use them.
4.  **Update README:** Update the README with examples of how to use the generated mocks.

**Benefits of This Approach**

*   **Easier Testing:** Developers can test their code using mocks, without relying on actual implementations.
*   **Focused Tests:** Developers can verify their code against specific test scenarios by configuring mock objects.
*   **Reduced Dependencies:** Mocks prevent tests from being affected by changes in the concrete implementation.

By incorporating these mocks, your package will be much easier to test, increasing adoption and developer satisfaction. The automated agent needs to perform each step carefully, generating robust code that provides excellent support for testing.

If you need help finding information that is not contained in this repo you can ask ask perplexity (a web search tool) by running `./ask_perplexity.sh "your question"` and it will search the web for the answer and return it to you.

After you've made changes to the code run `swift build` to update the built interfaces used by the linter and tests. Then update any affected tests and run `swift test` to verify that the tests still pass and we have high test coverage.

We do not need to maintain any backward compatibility when making changes.
Our goal is to be a Swift port of the vercel AI sdk (which is typescript). We should aim to stick to as much similarity in interfacesas possible to the vercel ai sdk so that this is very familiar to use and only make changes where necessaryto be Swift idiomatic or if the change is called out and explained in the README.md file.

DO NOT create new files unless those files are specified in GOALS.md or requested specifically by the user. If you think a new file or folder is needed confirming with the user before creating it.

There is no need to apologize for problems or mistakes.

---
> Source: [eastlondoner/swift-ai](https://github.com/eastlondoner/swift-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-07 -->
