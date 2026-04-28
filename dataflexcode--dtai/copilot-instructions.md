## dtai

> 1. Create a "root" object for the document:

# cJSONObject Rules

## How to create a JSON object in Dataflex using the cJSONObject class

1. Create a "root" object for the document:
```
Get Create (RefClass(cJsonObject)) to hoRoot
```

2. To add a simple member to a JSON object, use the SetMemberValue message, which takes the member name, type and value as parameters.
```
Send SetMemberValue of hoRoot "model" jsonTypeString "claude-sonnet-4-20250514"
```

Simple types include jsonTypeString, jsonTypeInteger, jsonTypeBoolean, jsonTypeDouble, and jsonTypeNull

3. To add an array to a JSON object, create a new JSON object, initialize it to an array type, and use SetMember to add the array:
```
Get Create (RefClass(cJsonObject)) to hoMessages
Send InitializeJsonType of hoMessages jsonTypeArray
Send SetMember of hoRoot "messages" hoMessages      
```

4. To add an complex item to an array i.e. an object with multiple values,  create a new object, initialize it to the JSON object type, and use AddMember to add it to the array:
```
Get Create (RefClass(cJsonObject)) to hoMessage
Send InitializeJsonType of hoMessage jsonTypeObject
Send AddMember of hoMessages hoMessage
```

5. To add a simple value to an array, use the AddMemberValue method:
```
Send AddMemberValue of hoArrayObject jsonTypeString "new value"
```

6. Once handles to objects are no longer needed, they must be destroyed:
```
send destroy of hoText
```

7. Create a JSON object from a DataFlex Struct

Use procedure DataTypeToJson to create a JSON object with the data in a struct
```
Procedure DataTypeToJson Struct data
```

## Reading/parsing JSON objects with cJSONObject
 
1. To get the number of members in a JSON object, use MemberCount:
```
Function MemberCount Returns Integer
```

2. To get a handle to a member of a JSON object, use MemberByIndex:
```
Function MemberByIndex Integer iIndex Returns Handle
```

3. To get the value of a simple member of a JSON object, use MemberValue:
```
Function MemberValue String sMember Returns String
```

4. To enumerate the names of the members of an object, use MemberNameByIndex:
```
Function MemberNameByIndex Integer iIndex Returns String
```

5. To get a handle to a JSON member use the Member function:
```
Function Member String sMember Returns Handle
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DataFlexCode) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
