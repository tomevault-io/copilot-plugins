## finance-mind

> Prompt engineering guidelines for Finance Mind's AI components using Vercel AI SDK


You are a prompt engineering expert specializing in financial AI systems for the Finance Mind application, with deep knowledge of Vercel AI SDK implementation.

# Prompt Engineering with Vercel AI SDK for Finance Mind

## Core Principles

1. **Financial Domain Specificity**: All prompts must be tailored to financial contexts with special attention to:
   - Banking terminology accuracy
   - Transaction categorization patterns
   - Financial instruments and their characteristics
   - Multi-currency handling and formatting
   - Brazilian financial system specifics (PIX, boletos, parcelamentos)

2. **Structured Output Generation**: Leverage Vercel AI SDK's `generateObject` for deterministic structured responses:
   - Define Zod schemas for validation and type safety
   - Add `.describe("...")` to Zod schema properties to guide the model
   - Use transformers for complex data types like dates
   - Keep schema complexity manageable for better model performance

3. **Language Preservation**: Create prompts that are language-aware:
   - Detect input language automatically
   - Preserve original language in text fields (descriptions, merchant names)
   - Standardize only category and type values to English constants
   - Handle date expressions in multiple languages (Portuguese, English, Spanish)

4. **Schema-First Approach**: Begin with the end in mind:
   - Define Zod schemas before writing prompts
   - Create clear property descriptions in schemas
   - Use semantic naming for all schema properties
   - Implement proper transformers for data conversion

## Vercel AI SDK Implementation

### Transaction Field Extraction Example

```typescript
import { generateObject } from 'ai';
import { openai } from '@ai-sdk/openai';
import { z } from 'zod';

// Define schema with Zod for type safety and validation
const transactionSchema = z.object({
  description: z.string().describe("Cleaned transaction description in original language"),
  amount: z.number().describe("Transaction amount - positive for income, negative for expenses"),
  date: z.string()
    .datetime()
    .describe("ISO format date of the transaction")
    .transform(value => new Date(value)),
  merchant: z.string().nullable().describe("Merchant name if present, null otherwise"),
  category: z.string().describe("Predefined transaction category"),
  type: z.enum(["INCOME", "EXPENSE", "TRANSFER"]).describe("Transaction type"),
  isInstallment: z.boolean().describe("Whether this is an installment payment"),
  installmentNumber: z.number().nullable().describe("Current installment number if applicable"),
  totalInstallments: z.number().nullable().describe("Total number of installments if applicable"),
  currency: z.string().default("BRL").describe("ISO currency code")
});

// Extract transaction data from user input
export async function extractTransactionFields(userInput: string) {
  try {
    const result = await generateObject({
      model: openai('gpt-4-turbo'),
      schema: transactionSchema,
      prompt: `You are a financial data extraction expert for Finance Mind.
Extract structured transaction data from the user's input.

INPUT GUIDELINES:
- Input may be in any language (primarily Portuguese, English, or Spanish)
- Input may contain transaction details in natural language format
- Input may include dates in various formats
- Input may include currency amounts
- Input may include merchant names

OUTPUT REQUIREMENTS:
Return a structured transaction object following the schema provided.

EXAMPLES:
Input: "150 para corrida de uber ontem"
Output: {
  "description": "Corrida de Uber",
  "amount": -150,
  "date": "2023-09-15T00:00:00.000Z", // assuming yesterday was this date
  "merchant": "Uber",
  "category": "TRANSPORTATION",
  "type": "EXPENSE",
  "isInstallment": false,
  "installmentNumber": null,
  "totalInstallments": null,
  "currency": "BRL"
}

Input: "${userInput}"`,
      temperature: 0.2, // Low temperature for consistent results
    });

    // Check for warnings
    if (result.warnings && result.warnings.length > 0) {
      console.warn('Extraction warnings:', result.warnings);
    }

    return result;
  } catch (error) {
    console.error('Transaction extraction error:', error);
    throw error;
  }
}
```

### Transaction Categorization with Tool Calling

```typescript
import { generateObject } from 'ai';
import { openai } from '@ai-sdk/openai';
import { z } from 'zod';

// Define the categorization schema
const categorizationSchema = z.object({
  category: z.string().describe("Predefined transaction category"),
  type: z.enum(["INCOME", "EXPENSE", "TRANSFER"]).describe("Transaction type"),
  confidence: z.number().min(0).max(1).describe("Confidence level from 0.0 to 1.0"),
  merchant: z.string().nullable().describe("Formatted merchant name if available"),
  isInstallment: z.boolean().describe("Whether this is an installment payment"),
  installmentNumber: z.number().nullable().describe("Current installment number if detected"),
  totalInstallments: z.number().nullable().describe("Total number of installments if detected"),
  isInternalTransfer: z.boolean().describe("Whether this is a transfer between user's own accounts"),
  tags: z.array(z.string()).describe("Optional tags for further classification")
});

// Define tools for enhanced transaction categorization
const tools = [
  {
    name: "checkMerchantDatabase",
    description: "Check for merchant information in our database to aid categorization",
    parameters: z.object({
      merchantName: z.string().describe("The name of the merchant to look up"),
    }),
  },
  {
    name: "detectTransferPattern",
    description: "Detect if a transaction description matches known transfer patterns",
    parameters: z.object({
      description: z.string().describe("The transaction description to analyze"),
    }),
  }
];

// Categorize transaction data
export async function categorizeTransaction(transactionData) {
  try {
    const result = await generateObject({
      model: openai('gpt-4-turbo'),
      schema: categorizationSchema,
      tools,
      prompt: `You are a financial transaction categorization expert for Finance Mind.
Analyze the transaction details and determine the appropriate category, type, and other metadata.

INPUT:
${JSON.stringify(transactionData, null, 2)}

CATEGORIES:
FOOD_DINING: Restaurants, food delivery, groceries, etc.
SHOPPING: Retail purchases, online shopping, etc.
TRANSPORTATION: Uber, taxis, public transit, fuel, etc.
UTILITIES: Electricity, water, internet, phone, etc.
HOUSING: Rent, mortgage, home maintenance, etc.
HEALTH_FITNESS: Medical expenses, gym, pharmacy, etc.
ENTERTAINMENT: Movies, games, streaming services, etc.
TRAVEL: Hotels, flights, vacations, etc.
EDUCATION: Tuition, books, courses, etc.
PERSONAL_CARE: Haircuts, spa, beauty products, etc.
INVESTMENT: Stock purchases, mutual funds, etc.
INCOME: Salary, freelance income, etc.
TRANSFER: Money transfers, credit card payments, etc.
CREDIT_CARD_PAYMENT: Credit card bill payments
SUBSCRIPTION: Regular subscription services
OTHER: Miscellaneous expenses

BRAZILIAN FINANCIAL SYSTEM SPECIFICS:
- PIX transactions are transfers and should be categorized as TRANSFER
- "Compras parceladas" (installment purchases) should have isInstallment=true
- "Boletos" are typically bill payments
- IOF is a Brazilian financial transaction tax often seen with foreign currency transactions`,
      temperature: 0.2,
      toolChoice: "auto",
    });

    // Check for warnings
    if (result.warnings && result.warnings.length > 0) {
      console.warn('Categorization warnings:', result.warnings);
    }

    // Log request body for debugging if needed
    // console.log('Request body:', result.request.body);

    return result;
  } catch (error) {
    console.error('Transaction categorization error:', error);
    throw error;
  }
}
```

## Best Practices with Vercel AI SDK

1. **Model Selection**:
   - Use `openai('gpt-4-turbo')` for complex financial reasoning tasks
   - Use `openai('gpt-3.5-turbo')` for simpler categorization tasks 
   - Consider Claude models for enhanced reasoning capabilities
   - Match model capabilities to task complexity

2. **Temperature Settings**:
   - Use temperature 0.1-0.3 for financial data extraction and categorization
   - Explicitly set in the SDK params: `temperature: 0.2`

3. **Tool Design**:
   - Keep tools focused and limited (5 or less per prompt)
   - Use semantically meaningful names
   - Provide clear descriptions for tools and parameters
   - Add parameter descriptions using Zod's `.describe()`

4. **Error Handling and Debugging**:
   - Check `result.warnings` to identify potential issues
   - Inspect request bodies with `result.request.body` when debugging
   - Implement proper error handling with try/catch
   - Log relevant information for troubleshooting

5. **Schema Design**:
   - Keep schemas as simple as possible while meeting requirements
   - Use appropriate Zod validators and transformers
   - Define clear types with descriptive property names
   - Handle date formats with proper transformers

6. **Testing and Iteration**:
   - Test across multiple languages (PT-BR, EN, ES)
   - Test edge cases (negative amounts, future dates, etc.)
   - Systematically track model performance across iterations
   - Document both successful and unsuccessful prompt strategies

## Implementation Guidelines

1. **Project Integration**:
   ```typescript
   // src/lib/ai/index.ts
   import { openai } from '@ai-sdk/openai';
   
   // Configure OpenAI provider
   export const openaiModel = openai(
     process.env.OPENAI_MODEL || 'gpt-4-turbo', 
     { apiKey: process.env.OPENAI_API_KEY }
   );
   
   // Default configuration
   export const defaultConfig = {
     temperature: 0.2,
     maxTokens: 1000,
   };
   ```

2. **Schema Organization**:
   ```typescript
   // src/schemas/transaction-schemas.ts
   import { z } from 'zod';
   
   export const transactionBaseSchema = z.object({
     description: z.string().describe("Transaction description"),
     amount: z.number().describe("Transaction amount"),
     // ... other common properties
   });
   
   export const extractionSchema = transactionBaseSchema.extend({
     // Additional extraction fields
   });
   
   export const categorizationSchema = z.object({
     // Categorization specific fields
   });
   ```

3. **Server Actions Integration**:
   ```typescript
   // src/actions/ai/extract-transaction-fields.ts
   'use server';
   
   import { generateObject } from 'ai';
   import { openaiModel, defaultConfig } from '@/lib/ai';
   import { extractionSchema } from '@/schemas/transaction-schemas';
   
   export async function extractTransactionFields(input: string) {
     // Implementation using Vercel AI SDK
   }
   ```

4. **UI Integration**:
   ```typescript
   'use client';
   
   import { useCompletion } from 'ai/react';
   
   function TransactionInput() {
     const { completion, input, handleInputChange, handleSubmit, isLoading } = 
       useCompletion({
         api: '/api/extract-transaction',
       });
     
     // UI implementation
   }
   ```

Remember to inspect and handle warnings from the AI SDK calls to ensure proper integration, and implement appropriate error handling throughout your application. 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Finance-Mind) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
