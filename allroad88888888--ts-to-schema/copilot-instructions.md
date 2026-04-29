## ts-to-schema

> Transforms to:

# TS to JSON Schema Conversion Rules

These rules define how to transform TypeScript types into JSON Schema dynamically.
They will be used by cursor to generate JSON Schema from provided TypeScript code.

---

## Rule 1: Interface Declarations

- **When:** The node represents an `InterfaceDeclaration`.
- **Action:** 
  - Generate an object schema.
  - Use the interface name as the schema title.
  - For each property:
    - Recursively transform its type.
    - Include a description if available.
    - If not optional, add the property name to the `required` array.
- **Example:**  
  For an interface like:
  ```tsx
  export interface VDom extends HTMLElement {
    custom: string
  }
  ```
  The transformation should produce an object schema that extends the `HTMLElement` definition and adds a `custom` property.

---

## Rule 2: Union Types

- **When:** The node represents a union type (using the `|` operator).
- **Action:** 
  - Create an `anyOf` array.
  - Each member of the union is recursively transformed.
- **Example:**  
  For:
  ```tsx
  export type SimpleChartMark = SimpleIntervalMark | SimplePointMark;
  ```
  The resulting schema should have an `anyOf` with two schemas: one for `SimpleIntervalMark` and one for `SimplePointMark`.

---

## Rule 3: Literal String Types

- **When:** The node is a literal type with a string value (e.g. `'interval'` or `'point'`).
- **Action:** 
  - Generate a schema with `type: "string"` and use `const` to lock the value.
  - Include any available description.
- **Example:**  
  ```tsx
  type: 'interval';
  ```
  Transforms to:
  ```json
  {
    "type": "string",
    "const": "interval",
    "description": "标记类型标识符"
  }
  ```

---

## Rule 4: Primitive Types

- **When:** Encountering TypeScript primitive types.
- **Action:**  
  - `number` → `{ "type": "number" }`
  - `string` → `{ "type": "string" }`

---

## Rule 5: Date Type

- **When:** The type is a reference to `Date`.
- **Action:**  
  - Transform to a JSON string with ISO date-time format:
    ```json
    {
      "type": "string",
      "format": "date-time",
      "description": "Date 对象，在 JSON 中序列化为 ISO 日期字符串"
    }
    ```

---

## Rule 6: Type References

- **When:** A node is a type reference (除了 Date 外)。
- **Action:**  
  - Generate a `$ref` pointing to the corresponding definition in the `#/definitions` section.
- **Example:**  
  ```tsx
  vDom: VDom;
  ```
  Transforms to:
  ```json
  { "$ref": "#/definitions/VDom" }
  ```

---

## Rule 7: Object Literals / Inline Objects

- **When:** The node represents an inline object type.
- **Action:**  
  - Recursively parse its properties, similar to interface declarations.
  - Build an object schema，包含 `properties` 和 `required` 字段。

---

## Definitions Handling

- 所有通过 TypeReference 引用的类型（如 `HTMLElement`、`CSSProperties` 等）都应在最终 schema 的 `definitions` 中定义。
- 定义的内容可以进一步扩展，支持更多属性。
- 示例中：
  - `HTMLElement` 包含 `id`、`className`、`tagName`、`innerHTML` 和 `style` 属性。
  - `CSSProperties` 定义常见的 CSS 属性及允许的额外属性。

---

## Example Transformation

假设有如下 TypeScript 定义：

```tsx
export interface VDom extends HTMLElement{
  custom:string
}

export type SimpleIntervalMark = {
  type: 'interval';
  x: number;
  y: number;
  color: string;
  vDom: VDom;
};

export type SimplePointMark = {
  type: 'point';
  x: number;
  y: number;
  size: number;
  date: Date;
  style: CssProperties;
};

export type SimpleChartMark = SimpleIntervalMark | SimplePointMark;
```

根据以上规则，cursor 生成的 JSON Schema 应该类似于：

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "SimpleChartMark",
  "description": "图表标记的联合类型，支持区间和点两种标记类型",
  "anyOf": [
    {
      "title": "SimpleIntervalMark",
      "type": "object",
      "properties": {
        "type": {
          "type": "string",
          "const": "interval",
          "description": "标记类型标识符"
        },
        "x": { "type": "number", "description": "X轴坐标" },
        "y": { "type": "number", "description": "Y轴坐标" },
        "color": { "type": "string", "description": "标记颜色" },
        "vDom": {
          "type": "object",
          "description": "虚拟DOM元素，继承自HTMLElement",
          "allOf": [
            { "$ref": "#/definitions/HTMLElement" },
            {
              "properties": {
                "custom": {
                  "type": "string",
                  "description": "自定义属性"
                }
              },
              "required": ["custom"]
            }
          ]
        }
      },
      "required": ["type", "x", "y", "color", "vDom"]
    },
    {
      "title": "SimplePointMark",
      "type": "object",
      "properties": {
        "type": {
          "type": "string",
          "const": "point",
          "description": "标记类型标识符"
        },
        "x": { "type": "number", "description": "X轴坐标" },
        "y": { "type": "number", "description": "Y轴坐标" },
        "size": { "type": "number", "description": "点的大小" },
        "date": {
          "type": "string",
          "format": "date-time",
          "description": "Date 对象，在 JSON 中序列化为 ISO 日期字符串"
        },
        "style": {
          "type": "object",
          "$ref": "#/definitions/CSSProperties",
          "description": "CSS样式属性集合，类型为 CssProperties"
        }
      },
      "required": ["type", "x", "y", "size", "date", "style"]
    }
  ],
  "definitions": {
    "HTMLElement": {
      "type": "object",
      "description": "HTML元素基础接口",
      "properties": {
        "id": { "type": "string", "description": "元素ID" },
        "className": { "type": "string", "description": "元素的 class 属性" },
        "tagName": { "type": "string", "description": "元素的标签名称" },
        "innerHTML": { "type": "string", "description": "元素的 HTML 内容" },
        "style": {
          "$ref": "#/definitions/CSSProperties",
          "description": "元素的内联样式"
        }
      }
    },
    "CSSProperties": {
      "type": "object",
      "description": "CSS 样式属性集合",
      "properties": {
        "color": { "type": "string", "description": "文本颜色" },
        "backgroundColor": { "type": "string", "description": "背景颜色" },
        "fontSize": { "type": "string", "description": "字体大小" },
        "fontWeight": { "type": "string", "description": "字体粗细" },
        "margin": { "type": "string", "description": "外边距" },
        "padding": { "type": "string", "description": "内边距" },
        "display": { "type": "string", "description": "显示方式" },
        "position": { "type": "string", "description": "定位方式" }
      },
      "additionalProperties": {
        "type": "string",
        "description": "其他 CSS 属性"
      }
    }
  }
}
```

---

## Usage Instructions

1. **保存**：将本文件内容保存为 `.mdc` 文件，例如 `ts-to-json-schema.mdc`。
2. **加载到 cursor**：在 cursor 中加载此规则文件。
3. **生成代码**：输入你的 TypeScript 类型代码，cursor 将依据上述规则动态生成对应的 JSON Schema。
4. **扩展调整**：如有更多类型需求或复杂情况，请按实际情况扩展规则。

---

以上即为示例规则，供你参考使用，确保 cursor 在动态生成代码时能够深入解析嵌套对象及联合类型，而不是停留在 object 层级。  
根据具体项目需求，你可以进一步细化每条规则的判断和转换逻辑。
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/allroad88888888) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
