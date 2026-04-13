## series-sample

> You are an expert in GraphQL and Simfinity.js framework. Your role is to help generate complete, production-ready GraphQL type definitions that are fully compatible with Simfinity.js patterns.

# Simfinity GraphQL Type Generation Rules

You are an expert in GraphQL and Simfinity.js framework. Your role is to help generate complete, production-ready GraphQL type definitions that are fully compatible with Simfinity.js patterns.

## Core Principles

1. **Simfinity Compatibility**: All generated code must follow Simfinity.js patterns exactly as shown in the series-sample project
2. **Type Safety**: Use proper GraphQL types with validation and business rules
3. **Relationship Management**: Handle embedded objects, collections, and relations correctly
4. **Validation**: Include comprehensive field and type-level validations
5. **Controllers**: Implement proper lifecycle hooks and business logic
6. **State Machines**: Support state management when needed

## Project Structure

When generating a new Simfinity project, follow this exact structure:

```
project-name/
├── types/
│   ├── index.js                    # Central type loading with dependency order
│   ├── scalars.js                  # Custom validated scalars
│   ├── validators/
│   │   ├── customErrors.js         # Custom error classes
│   │   ├── fieldValidators.js      # Field-level validations
│   │   └── typeValidators.js       # Type-level business rules
│   ├── controllers/
│   │   └── [typeName]Controller.js # Type-specific controllers
│   └── [typeName].js               # Individual type definitions
├── package.json                    # Dependencies and scripts with server options
├── index.js                        # Main application entry (Simfinity server)
├── index.apollo.js                 # Apollo Server implementation
├── index.yoga.js                   # GraphQL Yoga implementation
└── .cursorrules                    # This file
```

## Type Definition Patterns

### Basic Type Structure

```javascript
import * as graphql from 'graphql';
import * as simfinity from '@simtlix/simfinity-js';
import { validateFieldName } from './validators/fieldValidators.js';
import { validateTypeBusinessRules } from './validators/typeValidators.js';
import { CustomScalar } from './scalars.js';

const { GraphQLObjectType, GraphQLString, GraphQLID, GraphQLNonNull, GraphQLList } = graphql;

const typeName = new GraphQLObjectType({
  name: 'typeName',
  extensions: {
    validations: {
      create: [validateTypeBusinessRules],
      update: [validateTypeBusinessRules]
    }
  },
  fields: () => ({
    id: { type: GraphQLID },
    // Field definitions with validations
  })
});

export default typeName;
simfinity.connect(null, typeName, 'typeName', 'collectionName', controller, null, stateMachine);
```

### Field Types and Extensions

#### Basic Fields
```javascript
// Required string field
name: { 
  type: new GraphQLNonNull(GraphQLString),
  extensions: {
    validations: {
      save: [validateName],
      update: [validateName]
    }
  }
}

// Optional string field
description: { type: GraphQLString }

// List field
categories: { 
  type: new GraphQLList(GraphQLString),
  extensions: {
    validations: {
      save: [validateCategories],
      update: [validateCategories]
    }
  }
}
```

#### Custom Scalars
```javascript
// Use custom validated scalars
episodeNumber: { 
  type: EpisodeNumberScalar,
  extensions: {
    validations: {
      save: [validateEpisodeNumber],
      update: [validateEpisodeNumber]
    }
  }
}
```

#### Relations

**Embedded Objects (addNoEndpointType)**
```javascript
director: {
  type: new GraphQLNonNull(simfinity.getType('director')),
  extensions: {
    relation: {
      embedded: true,
      displayField: 'name'
    }
  }
}
```

**Collection Relations**
```javascript
// One-to-many relation
seasons: {
  type: new GraphQLList(simfinity.getType('season')),
  extensions: {
    relation: { 
      connectionField: 'serie' 
    }
  }
}

// Many-to-many through junction table
stars: {
  type: new GraphQLList(simfinity.getType('assignedStarAndSerie')),
  extensions: {
    relation: {
      connectionField: 'serie'
    }
  }
}
```

**Foreign Key Relations**
```javascript
serie: {
  type: simfinity.getType('serie'),
  extensions: {
    relation: {
      connectionField: 'serie',
      displayField: 'name'
    }
  }
}
```

### Enums

```javascript
const stateEnum = new GraphQLEnumType({
  name: 'stateEnum',
  values: {
    SCHEDULED: { value: 'SCHEDULED' },
    ACTIVE: { value: 'ACTIVE' },
    FINISHED: { value: 'FINISHED' }
  }
});

// Usage in type
state: { type: stateEnum }
```

### State Machines

```javascript
const stateMachine = {
  initialState: stateEnum.getValue('SCHEDULED'),
  actions: {
    activate: {
      from: stateEnum.getValue('SCHEDULED'),
      to: stateEnum.getValue('ACTIVE'),
      action: async (params) => {
        console.log('Activating:', JSON.stringify(params));
      }
    },
    finalize: {
      from: stateEnum.getValue('ACTIVE'),
      to: stateEnum.getValue('FINISHED'),
      action: async (params) => {
        console.log('Finalizing:', JSON.stringify(params));
      }
    }
  }
};
```

### Controllers

```javascript
const typeController = {
  onSaving: async (doc, args, session) => {
    console.log(`Creating ${doc.name}`);
    // Pre-save logic
  },

  onUpdating: async (id, doc, session) => {
    console.log(`Updating ${id}`);
    // Pre-update logic
  },

  onDelete: async (doc, session) => {
    console.log(`Deleting ${doc.name}`);
    // Pre-delete validation
  }
};
```

### Custom Scalars

```javascript
export const CustomScalar = simfinity.createValidatedScalar(
  'CustomScalar',
  'Description of the scalar validation',
  GraphQLString, // Base GraphQL type
  (value) => {
    if (value === null || value === undefined) {
      return value;
    }
    
    // Validation logic
    if (value.length < 2) {
      throw new ValidationError('Value must be at least 2 characters');
    }
    
    return value;
  }
);
```

### Validators

#### Field Validators
```javascript
const validateFieldName = async (typeName, fieldName, value, session) => {
  if (!value || value.trim().length === 0) {
    throw new ValidationError(`${fieldName} cannot be empty`);
  }
  if (value.trim().length < 2) {
    throw new ValidationError(`${fieldName} must be at least 2 characters long`);
  }
};
```

#### Type Validators
```javascript
const validateTypeBusinessRules = async (typeName, args, modelArgs, session) => {
  // Business logic validation
  if (!args.id && !modelArgs.requiredField) {
    throw new BusinessError('Required field is missing');
  }
};
```

## Generation Rules

### From Application Description

When given an application description:

1. **Identify Entities**: Extract main entities and their properties
2. **Define Relationships**: Map one-to-one, one-to-many, many-to-many relationships
3. **Identify Embedded Objects**: Objects that don't need their own collections
4. **Create Junction Tables**: For many-to-many relationships
5. **Define Validations**: Business rules and field constraints
6. **Add State Machines**: For entities with state transitions
7. **Create Controllers**: For complex business logic

### From Mermaid Diagrams

When given a Mermaid diagram:

1. **Parse Entities**: Extract class/entity definitions
2. **Map Relationships**: Convert associations to Simfinity relations
3. **Handle Inheritance**: Convert to embedded objects or separate types
4. **Generate Validations**: Based on constraints in the diagram
5. **Create Dependencies**: Order types by dependency relationships

### Type Loading Order

Always load types in dependency order in `index.js`:

```javascript
// 1. Custom scalars first
import './scalars.js';

// 2. Independent types (no dependencies)
import './director.js';
import './star.js';

// 3. Types that depend on the above
import './assignedStarAndSerie.js';
import './season.js';
import './episode.js';

// 4. Main types that reference others
import './serie.js';
```

## Common Patterns

### Junction Tables
For many-to-many relationships, create junction types:
```javascript
const assignedStarAndSerieType = new GraphQLObjectType({
  name: 'assignedStarAndSerie',
  fields: () => ({
    id: { type: GraphQLID },
    serie: {
      type: new GraphQLNonNull(simfinity.getType('serie')),
      extensions: {
        relation: {
          embedded: false,
          connectionField: 'serie',
          displayField: 'name'
        }
      }
    },
    star: {
      type: new GraphQLNonNull(simfinity.getType('star')),
      extensions: {
        relation: {
          embedded: false,
          connectionField: 'star',
          displayField: 'name'
        }
      }
    }
  })
});
```

### Unique Constraints
```javascript
name: {
  type: GraphQLString,
  extensions: { 
    unique: true,
    validations: {
      save: [validateName, validateUniqueName],
      update: [validateName, validateUniqueName]
    }
  }
}
```

### Required vs Optional Fields
- Use `GraphQLNonNull` for required fields
- Use base types for optional fields
- Always include `id` field with `GraphQLID` type

## Error Handling

Use custom error classes:
```javascript
import { ValidationError, BusinessError } from './validators/customErrors.js';

// Field validation errors
throw new ValidationError('Field validation message');

// Business logic errors  
throw new BusinessError('Business rule violation message');
```

## Best Practices

1. **Naming**: Use camelCase for field names, PascalCase for type names
2. **Validation**: Always include appropriate validations for business rules
3. **Relations**: Use `embedded: true` for objects that don't need their own endpoints
4. **Controllers**: Implement lifecycle hooks for complex business logic
5. **State Machines**: Use for entities with clear state transitions
6. **Error Handling**: Use appropriate error types for different scenarios
7. **Dependencies**: Always load types in correct dependency order
8. **Documentation**: Include comments explaining complex business logic

## Project Generation

When generating a complete project:

1. **Copy Structure**: Use series-sample as template
2. **Update Dependencies**: Ensure all required packages are included
3. **Generate Types**: Create all type definitions following patterns
4. **Create Validators**: Implement field and type validators
5. **Add Controllers**: Implement business logic controllers
6. **Update Index**: Configure main application file
7. **Add Documentation**: Include README with setup instructions

## Mermaid Integration

Support generating types from Mermaid class diagrams:

```mermaid
classDiagram
    class Serie {
        +String name
        +String description
        +String[] categories
        +Director director
    }
    
    class Director {
        +String name
        +String country
    }
    
    class Season {
        +Int number
        +Int year
        +SeasonState state
    }
    
    Serie ||--o{ Season : has
    Serie ||--|| Director : directed_by
```

This should generate:
- `serie.js` with embedded director and seasons collection
- `director.js` as embedded type (addNoEndpointType)
- `season.js` with state enum and state machine
- Proper validators and controllers
- Complete project structure

Remember: Always prioritize Simfinity.js compatibility and follow the exact patterns shown in the series-sample project.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/simtlix) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
