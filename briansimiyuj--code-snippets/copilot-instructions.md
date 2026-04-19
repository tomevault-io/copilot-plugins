## code-snippets

> You must strictly follow the user's manual coding style. This is a requirement for all generated code blocks, components, and logic.

# TypeScript Coding Style: High-Air & Tight-Syntax (v2)

You must strictly follow the user's manual coding style. This is a requirement for all generated code blocks, components, and logic.

## 1. Vertical Spacing ("The Airy Rule")
- **Mandatory Empty Lines**: Leave exactly one empty line after every single line of code.
- **The Grouping Exception**: Do NOT leave empty lines between variables declared in a single comma-separated `const` block or consecutive `useState` declarations.
- **The Object/Interface Exception**: Inside an object, interface, or switch config, do NOT leave empty lines between properties, but DO leave one empty line after the opening brace `{` and one before the closing brace `}`.

## 2. Horizontal Syntax ("The Tight Rule")
- **No Space at Braces**: There must be NO space between the closing parenthesis/operator and the opening curly brace.
  - Correct: `const name = (props)=>{` | `if(condition){` | `switch(value){`
- **No Space in Assignments**: There must be NO space between the equals sign or colon and the opening brace.
  - Correct: `const object ={` | `property:{` | `interface Name{`
- **Generics**: No spaces inside or around angle brackets: `useState<string>(null)`.
- **Self-Closing Components**: There must be NO space before the closing slash in a self-closing component.
  - Correct: `<Component/>`
  - Incorrect: `<Component />`

## 3. Component & Logic Structure
- **Component Definition**: Use `const Component: React.FC<Props> = (props)=>{`.
- **Logic Placement**: Define helper functions (logic) inside the component body using the `const name = ()=>{` pattern.
- **Inner Block Padding**: The very first line of code inside any function or if-statement must be preceded by an empty line.
- **Return Statement**: Always leave an empty line after `return(` and before the closing `)`.

## 4. TypeScript & Syntax Details
- **No Semicolons**: Never include semicolons at the end of lines.
- **Destructuring**: Prefer the comma-separated multi-line `const` declaration for hooks and context.
- **Environment**: Use `import.meta.env.VITE_VARIABLE`.
- **Default Exports**: Always `export default Name` at the very bottom of the file with no semicolon.

## 5. Visual Reference Example
```typescript
const PatientStatus: React.FC = ()=>{

    const { appointment } = useManageAppointmentContext(),
          patient = appointment?.patient

    if(!appointment){

        return(

            <LoadingSpinner/>

        )

    }

    const statusConfig ={

        "Ready":{

            icon: <FaCheckCircle/>,
            text: "Ready"
            
        }

    }

    return(

        <div className="container">

            <PatientHeader/>

            <h1>{patient.name}</h1>

        </div>

    )

}

export default PatientStatus

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/briansimiyuj) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
