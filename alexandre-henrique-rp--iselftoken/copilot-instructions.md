## iselftoken

> - **Linguagem Ubíqua**: Use sempre os termos do domínio do negócio


# Rules para Projeto Next.js com TypeScript, Clean Code e DDD

## Princípios Fundamentais

### Domain Driven Design (DDD)
- **Linguagem Ubíqua**: Use sempre os termos do domínio do negócio
- **Bounded Contexts**: Mantenha contextos bem delimitados
- **Agregados**: Agrupe entidades relacionadas logicamente
- **Value Objects**: Use para dados imutáveis e validações
- **Domain Services**: Para lógicas que não pertencem a entidades específicas

### Clean Code
- **Funções pequenas**: Máximo 20 linhas por função
- **Responsabilidade única**: Uma função, uma responsabilidade
- **Nomes descritivos**: Sem abreviações, sem comentários desnecessários
- **Sem efeitos colaterais**: Funções puras sempre que possível
- **DRY**: Don't Repeat Yourself - componentize e reutilize

## Estrutura de Pastas

### Organização Arquitetural
```
src/
├── app/                    # Next.js App Router
│   ├── (protected)/       # Rotas autenticadas
│   ├── (public)/          # Rotas públicas
│   └── api/               # API Routes do Next.js
├── components/             # Componentes reutilizáveis
│   ├── ui/                # Componentes base (Button, Input)
│   ├── forms/             # Componentes de formulário
│   ├── layout/            # Header, Footer, Sidebar
│   └── business/          # Componentes específicos do domínio
├── models/                # Modelos de domínio e Value Objects
│   ├── entities/          # Entidades do domínio
│   ├── value-objects/     # Objetos de valor
│   └── types/             # Tipos TypeScript
├── modules/               # Serviços e casos de uso reutilizáveis
│   ├── auth/              # Autenticação
│   ├── email/             # Disparador de email
│   ├── validation/        # Validações
│   └── utils/             # Utilitários gerais
├── infrastructure/        # Camada de infraestrutura
│   ├── database/          # Configurações de BD
│   ├── external/          # Serviços externos
│   └── http/              # Clientes HTTP
└── hooks/                 # Custom hooks React
```

## Rules para Models

### Entidades (models/entities/)
```typescript
// ✅ Correto
export class Startup {
  constructor(
    private readonly id: StartupId,
    private name: StartupName,
    private valuation: Money,
    private category: StartupCategory
  ) {}

  public changeName(newName: StartupName): void {
    this.name = newName;
  }

  public calculateInvestmentPercentage(investment: Money): Percentage {
    return this.valuation.calculatePercentage(investment);
  }
}

// ❌ Evitar
export class Startup {
  id: number;
  name: string;
  valuation: number; // Sem encapsulamento
}
```

### Value Objects (models/value-objects/)
```typescript
// ✅ Correto - Imutável e com validação
export class Email {
  private constructor(private readonly value: string) {
    this.validate();
  }

  static create(email: string): Either<Error, Email> {
    try {
      return Right.create(new Email(email));
    } catch (error) {
      return Left.create(error);
    }
  }

  private validate(): void {
    if (!this.isValidEmail(this.value)) {
      throw new Error('Invalid email format');
    }
  }

  toString(): string {
    return this.value;
  }
}
```

### Types (models/types/)
```typescript
// ✅ Correto - Tipos específicos do domínio
export type StartupStatus = 'ACTIVE' | 'INACTIVE' | 'PENDING';
export type InvestmentType = 'SEED' | 'SERIES_A' | 'SERIES_B';

export interface StartupCreationData {
  name: string;
  description: string;
  category: string;
  valuation: number;
}
```

## Rules para Modules

### Estrutura de Módulos
```typescript
// app/api/startups/route.ts - API Route do Next.js
export async function GET(request: Request) {
  try {
    const { searchParams } = new URL(request.url);
    const page = searchParams.get('page') || '1';
    
    const getStartupsUseCase = new GetStartupsUseCase(
      startupRepository,
      authService
    );
    
    const result = await getStartupsUseCase.execute({ page: Number(page) });
    
    if (result.isFailure()) {
      return NextResponse.json(
        { error: result.error.message },
        { status: 400 }
      );
    }
    
    return NextResponse.json({
      status: 'success',
      data: result.value
    });
  } catch (error) {
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}

export async function POST(request: Request) {
  try {
    const body = await request.json();
    const createStartupCommand = CreateStartupCommand.create(body);
    
    const createStartupUseCase = new CreateStartupUseCase(
      startupRepository,
      emailService
    );
    
    const result = await createStartupUseCase.execute(createStartupCommand);
    
    return NextResponse.json({
      status: 'success',
      data: result.value
    }, { status: 201 });
  } catch (error) {
    return NextResponse.json(
      { error: error.message },
      { status: 400 }
    );
  }
}
```

```typescript
// modules/email/email.service.ts - Serviço reutilizável
export interface EmailService {
  sendWelcomeEmail(to: Email, data: WelcomeEmailData): Promise<Result<void>>;
  sendInvestmentConfirmation(to: Email, data: InvestmentData): Promise<Result<void>>;
}

export class NodemailerEmailService implements EmailService {
  constructor(private readonly config: EmailConfig) {}

  async sendWelcomeEmail(to: Email, data: WelcomeEmailData): Promise<Result<void>> {
    try {
      const template = WelcomeEmailTemplate.create(data);
      await this.send(to, template);
      return Result.success();
    } catch (error) {
      return Result.failure(error);
    }
  }
}
```
### Casos de Uso (modules/{domain}/use-cases/)
```typescript
// ✅ Correto - Um caso de uso, uma responsabilidade
  constructor(
    private readonly startupRepository: StartupRepository,
    private readonly emailService: EmailService
  ) {}

  async execute(data: CreateStartupCommand): Promise<Result<Startup>> {
    const startup = await Startup.create(data);
    if (startup.isFailure()) {
      return Result.failure(startup.error);
    }

    const savedStartup = await this.startupRepository.save(startup.value);
    await this.emailService.sendWelcomeEmail(
      startup.value.getOwnerEmail(),
      { startupName: startup.value.getName() }
    );

    return Result.success(savedStartup);
  }
}
```

## Rules para Componentes

### Componentes Base (components/ui/)
```typescript
// ✅ Correto - Componente reutilizável e tipado
interface ButtonProps {
  variant: 'primary' | 'secondary' | 'danger';
  size: 'sm' | 'md' | 'lg';
  loading?: boolean;
  disabled?: boolean;
  onClick?: () => void;
  children: React.ReactNode;
}

export const Button: React.FC<ButtonProps> = ({
  variant,
  size,
  loading = false,
  disabled = false,
  onClick,
  children
}) => {
  const className = getButtonClassName(variant, size, loading, disabled);
  
  return (
    <button
      className={className}
      disabled={disabled || loading}
      onClick={onClick}
    >
      {loading ? <Spinner size={size} /> : children}
    </button>
  );
};
```

### Componentes de Negócio (components/business/)
```typescript
// ✅ Correto - Específico do domínio
interface StartupCardProps {
  startup: StartupViewModel;
  onInvest?: (startupId: string) => void;
  onViewDetails?: (startupId: string) => void;
}

export const StartupCard: React.FC<StartupCardProps> = ({
  startup,
  onInvest,
  onViewDetails
}) => {
  const progressPercentage = startup.calculateProgressPercentage();
  
  return (
    <Card className="startup-card">
      <StartupLogo src={startup.logoUrl} alt={startup.name} />
      <StartupInfo startup={startup} />
      <ProgressBar percentage={progressPercentage} />
      <StartupActions
        startupId={startup.id}
        onInvest={onInvest}
        onViewDetails={onViewDetails}
      />
    </Card>
  );
};
```

## Rules para Hooks Customizados

### Hooks de Domínio (hooks/)
```typescript
// ✅ Correto - Hook específico e reutilizável
export const useStartupInvestment = (startupId: string) => {
  const [investment, setInvestment] = useState<Investment | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);

  const investInStartup = useCallback(async (amount: number) => {
    setLoading(true);
    setError(null);

    try {
      const investmentCommand = InvestmentCommand.create({
        startupId,
        amount,
        investorId: getCurrentUserId()
      });

      const result = await investmentService.createInvestment(investmentCommand);
      
      if (result.isSuccess()) {
        setInvestment(result.value);
      } else {
        setError(result.error);
      }
    } catch (err) {
      setError(err as Error);
    } finally {
      setLoading(false);
    }
  }, [startupId]);

  return { investment, loading, error, investInStartup };
};
```

## Rules de Validação e Error Handling

### Result Pattern
```typescript
// models/common/result.ts
export class Result<T> {
  private constructor(
    private readonly success: boolean,
    private readonly _value?: T,
    private readonly _error?: Error
  ) {}

  static success<T>(value: T): Result<T> {
    return new Result(true, value);
  }

  static failure<T>(error: Error): Result<T> {
    return new Result(false, undefined, error);
  }

  isSuccess(): boolean {
    return this.success;
  }

  get value(): T {
    if (!this.success) {
      throw new Error('Cannot get value from failed result');
    }
    return this._value!;
  }

  get error(): Error {
    if (this.success) {
      throw new Error('Cannot get error from successful result');
    }
    return this._error!;
  }
}
```

### Validações (modules/validation/)
```typescript
// ✅ Correto - Validação específica do domínio
export class StartupValidator {
  static validateCreationData(data: StartupCreationData): ValidationResult {
    const errors: ValidationError[] = [];

    if (!this.isValidName(data.name)) {
      errors.push(new ValidationError('name', 'Nome deve ter entre 2 e 100 caracteres'));
    }

    if (!this.isValidValuation(data.valuation)) {
      errors.push(new ValidationError('valuation', 'Valuation deve ser maior que zero'));
    }

    return errors.length > 0 
      ? ValidationResult.withErrors(errors)
      : ValidationResult.success();
  }

  private static isValidName(name: string): boolean {
    return name.length >= 2 && name.length <= 100;
  }

  private static isValidValuation(valuation: number): boolean {
    return valuation > 0;
  }
}
```
## Rules de Performance e Otimização

### Memoização e Lazy Loading
```typescript
// ✅ Correto - Componente otimizado
export const StartupList = memo<StartupListProps>(({ startups, filters }) => {
  const filteredStartups = useMemo(
    () => startups.filter(startup => matchesFilters(startup, filters)),
    [startups, filters]
  );

  const handleStartupSelect = useCallback((startupId: string) => {
    router.push(`/startups/${startupId}`);
  }, [router]);

  return (
    <VirtualizedList
      items={filteredStartups}
      renderItem={({ item }) => (
        <StartupCard
          key={item.id}
          startup={item}
          onSelect={handleStartupSelect}
        />
      )}
    />
  );
});
```

## Rules de Testing

### Testes de Unidade
```typescript
// ✅ Correto - Teste focado na lógica de domínio
describe('StartupInvestmentService', () => {
  it('should calculate correct investment percentage', () => {
    // Arrange
    const startup = StartupBuilder.create()
      .withValuation(Money.create(1000000))
      .build();
    
    const investment = Money.create(100000);

    // Act
    const percentage = startup.calculateInvestmentPercentage(investment);

    // Assert
    expect(percentage.value).toBe(10);
  });
});
```
## Convenções de Nomenclatura

### Arquivos e Pastas
- **PascalCase**: Componentes React (`StartupCard.tsx`)
- **kebab-case**: Páginas e rotas (`startup-details.tsx`)
- **camelCase**: Funções e variáveis (`calculatePercent

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alexandre-henrique-rp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
