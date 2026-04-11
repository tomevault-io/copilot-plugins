## sunub-spaceship

> Naming should be as descriptive as possible. The only exception is the indexing variable in a loop. That can be shortened to a single letter starting from i.

## Coding Style

### Naming

Naming should be as descriptive as possible. The only exception is the indexing variable in a loop. That can be shortened to a single letter starting from i.

- variableNamesLikeThis
- functionNamesLikeThis
- ClassNamesLikeThis
- methodNamesLikeThis
- ConstantsLikeThis or CONSTANTS_LIKE_THIS (depending on use case)

Private properties and methods of objects begin with an underscore _.

### Literals

#### Objects

- Object should always be created by `{}` and not `new Object()`.
- When creating an empty Object, use `{}` without spaces.
- When creating an Object with keys, 
   - There should be a single space after the opening bracket.
   - There should be a single space before the closing bracket.
   - Keys should be written without quotes. The exception is when it is necessary.
   - There should be no space between the key word and the colon.
   - There should be a space between the colon and the value.
   - The comma should have no space before and 1 space behind itself.
   - The last value shouldn't be trailed with a comma.
   - The comma shouldn't start a new line. It should always trail the previous value.

good:

    var obj = { A: 1, b: 2, C: 3 };

    var obj = {
        A: 1, 
        b: 2, 
        C: 3 
    };

poor:

    var obj = {A:1,b:2,C:3};

    var obj = {A:1, b:2, C:3};

    var obj = {A : 1, b : 2, C : 3};

    var obj = { "A" : 1, "b" : 2, "C" : 3 };

    var obj = { A : 1, b : 2, C : 3 };

    var obj = { A :1, b :2, C :3 };

    var obj = { A : 1 , b : 2 , C : 3 };

    var obj = {
        A : 1, 
        b : 2, 
        C : 3, 
    };


    var obj = {
        A : 1 
      , b : 2
      , C : 3 
    };

#### Classes

- Private properties should start with an underscore.

#### Arrays

- Arrays should always be created by `[]` and not `new Array()`.
- When creating an empty Array, use `[]` without spaces.
- When creating an Array with values, 
   - There should be a single space after the opening bracket.
   - There should be a single space before the closing bracket.
   - The comma should have no space before and 1 space behind itself.
   - The last value shouldn't be trailed with a comma.
   - The comma shouldn't start a new line. It should always trail the previous value.

good:

    var arr = [ 1, 2, 3 ];

    var arr = [
        1, 
        2, 
        3 
    ];

poor:

    var arr = [1,2,3];

    var arr = [1, 2, 3];

    var arr = [ 1 , 2 , 3 ];

    var arr = [
        1, 
        2, 
        3,
    ];

    var arr = [
        1
      , 2
      , 3 
    ];

#### Strings

Strings are written using single quotes:

good:

    var  lyrics  =  'Never gonna Give you up, Never gonna Let you down' ;

#### Semicolon

Semicolons are always placed.

### Blocks

- The opening brackets should be followed by 1 empty line.
- The closing brackets should be behind 1 empty line.
- The opening brackets should always follow a space and not start at a new line

good:

    if ( a === 0 ) {

       // this is good
       return true;

    }

poor:

    if ( a === 0 ) {
        // this is bad: missing empty line after '{' 
        return true;

    }

    if ( a === 0 ) {

        // this is bad: missing empty line before '}'
        return true;
    }

    if ( a === 0 ) { // this is bad:  stuff after '{'

        return true;

    }

    if ( a === 0 ){

       // this is bad: no space before '{'
       return true;

    }


### Conditional instructions

#### if statement

- The if keyword should always be followed with a space, an opening parenthesis and another space.
- The test should end with a space, a closing parenthesis and another space.
- The if statement should always contain a block.
- If there is an else statement, it should be on the same line as the closing bracket of the block.
- The else statement is followed by another block and should be separated from both blocks with a single space on both sides.
- Assignment should not be used in a test.

good:

    if ( test ) { 

        // ... 

    } else { 

        // ... 

    }

poor:

    if (test) { 

        // ... 

    }

    if( test ) { 

        // ... 

    }

    if (test ) { 

        // ... 

    }

    if ( test) { 

        // ... 

    }

    if ( test ){ 

        // ... 

    }

    if ( test ) { 

        // ... 

    }else { 

        // ... 

    }

    if ( test ) { 

        // ... 

    } else{ 

        // ... 

    }


#### switch

- The switch keyword should always be followed with a space, an opening parenthesis and another space.
- The test value should end with a space, a closing parenthesis and another space.
- Every case should have a break statement, except for the default case or when returning a value.
- Every case should have a space between the colon and the test.
- Before and after each case, there should be a new line.
- Assignment should not be used in a test.

good:

    switch ( value ) { 

        case 1 : 

            // ... 
            break ;

        case 2 : 

            // ... 
            break ;

        default : 

            // ... 
            // no break keyword on the last case 

    }

### Cycles

#### for

- Be aware that `foreach` and `for in` are much slower than `for`.
- The for keyword should always be followed with a space, an opening parenthesis and another space.
- The assignment part should start with a space.
- The assignment part should end with a semicolon and a space.
- The test part should start with a space.
- The test part should end with a semicolon and a space.
- The update part should start with a space.
- The update part should end with a space, a closing parenthesis and another space 

### Operators

#### with

Operator `with` not used.

#### The equality operator

Always use strict equality `===` (inequality `!==` ).

#### Ternary operator

Always use spaces around the colon and question mark.

#### Unary

All unary operators are written together with the following operands:

var  foo  =  ! bar;

#### eval

Avoid using eval. To parse json, use JSON.parse.

#### undefined

Check the value through a strict comparison.

Good:

    x === undefined;
Poor:

    / / In modern browsers already defined immutable undefined. 
    var undefined; 
    x === undefined;

    typeof x === 'undefined'

    x === void 0

##  TSL Generation Guidelines (Local Context)
You are required to write Three.js Shading Language (TSL) code based on the following local documents. When generating code, prioritize the guidelines and API designs found in the folders listed below.

### 1. Local Knowledge Base
Root Path: `./three.js.wiki`

Core Reference Files:

`./three.js.wiki/Three.js-Shading-Language-Examples.md`: Basic structure and syntax of TSL.

`./three.js.wiki/Three.js-WebGPU.md`: Technical white paper on next-gen web 3D graphics implementation strategies using Three.js and WebGPU.

### 2. Instructions
All shader logic must adhere to the `three/webgpu` and `three/tsl` standards.

Prioritize the creation of functional nodes according to the latest specifications in the `./three.js.wiki` folder.

If the implementation of a specific feature is ambiguous, locate and emulate the style of similar example files (.md) within the mentioned folder.

Before writing any code, thoroughly review the contents of the necessary documents and ensure compliance with the established rules.

## Three.js Development Guidelines

### 1. Primary Documentation Reference

- Local Documentation Path: `three.js.wiki/three.js-docs/`
- Context: This directory contains the cloned API reference for Three.js, including detailed specifications for classes, methods, and properties.

### 2. Mandatory Instructions for Three.js Coding

- Priority Access: When generating, refactoring, or debugging Three.js related code, you must prioritize the documentation located at three.js.wiki/three.js-docs/ over your general pre-trained knowledge.

- Strict Method Validation: Always verify method signatures, parameter types, and return values against the local .md files in the specified path before providing code examples.

- Compliance with Standards: Ensure that all Three.js implementations (e.g., Geometry, Material, WebGLRenderer settings) strictly follow the conventions and latest standards defined in the local documentation.

### 3. Response Requirements

- If a Three.js feature is requested, analyze the relevant files in the three.js-docs folder first.

- In case of any conflict between general web data and the provided local documentation, the local documentation must be treated as the source of truth.

- When providing Three.js code, you may implicitly or explicitly reference which parts of the local docs were used to ensure factual accuracy.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sunub)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sunub)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
