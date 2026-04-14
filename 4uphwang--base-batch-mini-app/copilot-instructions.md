## base-batch-mini-app

> TypeScript and React specific coding standards


# TypeScript & React Coding Standards

## TypeScript Rules

### Type Safety

```typescript
// ✅ Good - Explicit types
interface Card {
  id: number;
  nickname: string;
  bio: string;
  address: string;
  skills: string[];
}

// ✅ Good - Generic constraints
function useApi<T>(url: string): UseQueryResult<T, Error> {
  return useQuery({
    queryKey: [url],
    queryFn: () => fetch(url).then((res) => res.json()),
  });
}

// ❌ Bad - Any types
function processData(data: any) {}
const result: any = await api.getData();
```

### Interface Design

```typescript
// ✅ Good - Specific interfaces
interface CardCreateRequest {
  nickname: string;
  bio: string;
  imageURI: string;
  address: string;
  skills: string[];
}

interface CardUpdateRequest {
  nickname?: string;
  bio?: string;
  skills?: string[];
}

// ✅ Good - Discriminated unions
type ApiResponse<T> =
  | { success: true; data: T }
  | { success: false; error: string };
```

### Error Handling

```typescript
// ✅ Good - Proper error types
class CardError extends Error {
  constructor(
    message: string,
    public code: "VALIDATION_ERROR" | "NETWORK_ERROR" | "UNKNOWN_ERROR"
  ) {
    super(message);
    this.name = "CardError";
  }
}

// ✅ Good - Error boundaries
interface ErrorBoundaryState {
  hasError: boolean;
  error?: Error;
}
```

## React Component Rules

### Component Structure

```typescript
// ✅ Good - Proper component structure
interface CardProps {
  card: Card;
  onEdit?: (id: number) => void;
  onDelete?: (id: number) => void;
}

export function CardComponent({ card, onEdit, onDelete }: CardProps) {
  // Hooks at the top
  const [isEditing, setIsEditing] = useState(false);
  const { mutate: updateCard } = useUpdateCard();

  // Event handlers
  const handleEdit = useCallback(() => {
    setIsEditing(true);
    onEdit?.(card.id);
  }, [card.id, onEdit]);

  // Render
  return (
    <div className="card">
      <CardHeader card={card} />
      <CardContent card={card} />
      <CardActions onEdit={handleEdit} onDelete={onDelete} />
    </div>
  );
}
```

### Hook Rules

```typescript
// ✅ Good - Custom hook with proper dependencies
export function useCardData(cardId: number) {
  const [card, setCard] = useState<Card | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    let cancelled = false;

    async function fetchCard() {
      try {
        setLoading(true);
        const data = await fetchCardById(cardId);
        if (!cancelled) {
          setCard(data);
          setError(null);
        }
      } catch (err) {
        if (!cancelled) {
          setError(err instanceof Error ? err.message : "Unknown error");
        }
      } finally {
        if (!cancelled) {
          setLoading(false);
        }
      }
    }

    fetchCard();

    return () => {
      cancelled = true;
    };
  }, [cardId]);

  return { card, loading, error };
}
```

### State Management

```typescript
// ✅ Good - Focused state atoms
export const cardState = atom({
  selectedCard: null as Card | null,
  isEditing: false,
  isLoading: false,
});

export const useCardState = () => {
  const [state, setState] = useAtom(cardState);

  const selectCard = useCallback(
    (card: Card) => {
      setState((prev) => ({ ...prev, selectedCard: card }));
    },
    [setState]
  );

  const startEditing = useCallback(() => {
    setState((prev) => ({ ...prev, isEditing: true }));
  }, [setState]);

  return {
    ...state,
    selectCard,
    startEditing,
  };
};
```

## Performance Rules

### Memoization

```typescript
// ✅ Good - Memoized components
const CardItem = memo(({ card, onSelect }: CardItemProps) => {
  const handleClick = useCallback(() => {
    onSelect(card.id);
  }, [card.id, onSelect]);

  return (
    <div onClick={handleClick} className="card-item">
      <CardImage src={card.imageURI} alt={card.nickname} />
      <CardInfo card={card} />
    </div>
  );
});

// ✅ Good - Memoized callbacks
function CardList({ cards, onCardSelect }: CardListProps) {
  const handleCardSelect = useCallback(
    (cardId: number) => {
      onCardSelect(cardId);
    },
    [onCardSelect]
  );

  return (
    <div className="card-list">
      {cards.map((card) => (
        <CardItem key={card.id} card={card} onSelect={handleCardSelect} />
      ))}
    </div>
  );
}
```

### Lazy Loading

```typescript
// ✅ Good - Lazy loading heavy components
const CardSharePopup = lazy(() => import("./CardSharePopup"));
const CardEditModal = lazy(() => import("./CardEditModal"));

function CardActions({ cardId }: { cardId: number }) {
  const [showShare, setShowShare] = useState(false);

  return (
    <div>
      <button onClick={() => setShowShare(true)}>Share Card</button>
      {showShare && (
        <Suspense fallback={<div>Loading...</div>}>
          <CardSharePopup cardId={cardId} />
        </Suspense>
      )}
    </div>
  );
}
```

## Web3 Integration Rules

### Contract Interaction

```typescript
// ✅ Good - Type-safe contract calls
export function useCardMint() {
  const { writeContract } = useWriteContract();
  const { address } = useAccount();

  const mintCard = useCallback(
    async (cardData: CardMintData) => {
      if (!address) {
        throw new Error("Wallet not connected");
      }

      try {
        const hash = await writeContract({
          address: BASECARD_CONTRACT_ADDRESS,
          abi: BaseCardABI,
          functionName: "mint",
          args: [
            cardData.nickname,
            cardData.bio,
            cardData.imageURI,
            cardData.skills,
          ],
        });

        return hash;
      } catch (error) {
        console.error("Mint failed:", error);
        throw new Error("Failed to mint card");
      }
    },
    [writeContract, address]
  );

  return { mintCard };
}
```

### Error Handling

```typescript
// ✅ Good - Web3 error handling
export function useWeb3ErrorHandler() {
  const handleError = useCallback((error: unknown) => {
    if (error instanceof UserRejectedRequestError) {
      toast.error("Transaction rejected by user");
    } else if (error instanceof InsufficientFundsError) {
      toast.error("Insufficient funds for transaction");
    } else if (error instanceof ContractFunctionExecutionError) {
      toast.error("Contract execution failed");
    } else {
      toast.error("Transaction failed");
      console.error("Web3 error:", error);
    }
  }, []);

  return { handleError };
}
```

## Testing Rules

### Component Testing

```typescript
// ✅ Good - Testable component design
interface CardProps {
  card: Card;
  onEdit?: (id: number) => void;
  "data-testid"?: string;
}

export function Card({ card, onEdit, ...props }: CardProps) {
  return (
    <div data-testid={props["data-testid"] || "card"}>
      <h3>{card.nickname}</h3>
      <p>{card.bio}</p>
      {onEdit && (
        <button onClick={() => onEdit(card.id)} data-testid="edit-button">
          Edit
        </button>
      )}
    </div>
  );
}
```

### Hook Testing

```typescript
// ✅ Good - Testable hook
export function useCardValidation() {
  const validateCard = useCallback((card: Partial<Card>) => {
    const errors: string[] = [];

    if (!card.nickname?.trim()) {
      errors.push("Nickname is required");
    }

    if (card.skills && card.skills.length > 5) {
      errors.push("Maximum 5 skills allowed");
    }

    return errors;
  }, []);

  return { validateCard };
}
```

## File Organization

### Import Order

```typescript
// ✅ Good - Import order
// 1. React imports
import React, { useState, useEffect, useCallback } from "react";

// 2. Third-party imports
import { useQuery, useMutation } from "@tanstack/react-query";
import { useAtom } from "jotai";

// 3. Internal imports
import { Card } from "@/types/card";
import { useCardValidation } from "@/hooks/useCardValidation";
import { cardState } from "@/store/cardState";

// 4. Relative imports
import "./Card.css";
```

### Export Patterns

```typescript
// ✅ Good - Named exports
export { CardComponent } from "./CardComponent";
export { useCardData } from "./useCardData";
export type { Card, CardProps } from "./types";

// ✅ Good - Default export for main component
export default function CardPage() {
  return <div>Card Page</div>;
}
```

## Common Patterns

### Loading States

```typescript
// ✅ Good - Loading state pattern
function CardList() {
  const { data: cards, isLoading, error } = useCards();

  if (isLoading) return <CardListSkeleton />;
  if (error) return <CardListError error={error} />;
  if (!cards?.length) return <EmptyCardList />;

  return (
    <div className="card-list">
      {cards.map((card) => (
        <CardItem key={card.id} card={card} />
      ))}
    </div>
  );
}
```

### Form Handling

```typescript
// ✅ Good - Form handling pattern
function CardForm({ onSubmit }: { onSubmit: (data: CardData) => void }) {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<CardData>();

  const handleFormSubmit = (data: CardData) => {
    onSubmit(data);
  };

  return (
    <form onSubmit={handleSubmit(handleFormSubmit)}>
      <input
        {...register("nickname", { required: "Nickname is required" })}
        placeholder="Enter nickname"
      />
      {errors.nickname && <span>{errors.nickname.message}</span>}

      <button type="submit">Create Card</button>
    </form>
  );
}
```

---

**Remember**: These TypeScript and React rules ensure type safety, performance, and maintainability. Always prioritize readability and testability in your code.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/4uphwang)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/4uphwang)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
