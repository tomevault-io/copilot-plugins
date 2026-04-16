## iadevst2

> Utilize Express para mapear os endpoints.


# REST/HTTP

## Framework

Utilize Express para mapear os endpoints.

**Exemplo:**
```typescript
import express from 'express';

const app = express();
app.use(express.json());

app.get('/users', (req, res) => {
  // implementação
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

## Padrão REST

Utilize o padrão REST para consultas, mantendo o nome dos recursos em inglês e no plural, permitindo a navegabilidade em recursos alinhados.

**Exemplo:**
```typescript
// ✅ Prefira
GET /users
GET /users/:userId
GET /playlists/:playlistId/videos
GET /customers/:customerId/invoices

// ❌ Evite
GET /getUsers
GET /user/:userId (singular)
GET /usuario/:usuarioId (português)
```

## Nomenclatura de Recursos

Recursos e verbos compostos devem usar kebab-case.

**Exemplo:**
```typescript
// ✅ Prefira
GET /scheduled-events
POST /users/:userId/change-password
GET /payment-methods
POST /orders/:orderId/process-payment

// ❌ Evite
GET /scheduledEvents (camelCase)
GET /scheduled_events (snake_case)
```

## Profundidade de Recursos

Evite criar endpoints com mais de 3 recursos.

**Exemplo:**
```typescript
// ❌ Evite - muito profundo
GET /channels/:channelId/playlists/:playlistId/videos/:videoId/comments

// ✅ Prefira - mais direto
GET /videos/:videoId/comments
GET /comments?videoId=:videoId

// ✅ Ou organize de forma mais plana
GET /channels/:channelId/playlists
GET /playlists/:playlistId/videos
GET /videos/:videoId/comments
```

## Mutações e Ações

Para mutações, não siga REST à risca. Utilize uma combinação de REST para navegar nos recursos e verbos para representar ações que estão sendo executadas, sempre com POST.

**Exemplo:**
```typescript
// ✅ Prefira - verbos para ações específicas
POST /users/:userId/change-password
POST /orders/:orderId/cancel
POST /invoices/:invoiceId/send-reminder
POST /accounts/:accountId/activate

// ❌ Evite - PUT genérico para ações específicas
PUT /users/:userId
PUT /orders/:orderId

// ✅ PUT é apropriado para substituição completa do recurso
PUT /users/:userId
{
  "name": "John Doe",
  "email": "john@example.com",
  "age": 30
}
```

## Formato de Dados

O formato do payload de requisição e resposta deve ser sempre JSON, salvo que especificado algo diferente.

**Exemplo:**
```typescript
app.use(express.json());

app.post('/users', (req, res) => {
  const { name, email } = req.body;
  const user = createUser({ name, email });
  res.json({ id: user.id, name: user.name, email: user.email });
});

// Request
// Content-Type: application/json
// { "name": "John Doe", "email": "john@example.com" }

// Response
// Content-Type: application/json
// { "id": "123", "name": "John Doe", "email": "john@example.com" }
```

## Códigos de Status HTTP

### 200 - OK
Retorne quando a requisição for bem-sucedida.

**Exemplo:**
```typescript
app.get('/users/:userId', (req, res) => {
  const user = getUser(req.params.userId);
  res.status(200).json(user);
});

app.post('/users', (req, res) => {
  const user = createUser(req.body);
  res.status(200).json(user);
});
```

### 404 - Not Found
Retorne se um recurso não for encontrado.

**Exemplo:**
```typescript
app.get('/users/:userId', (req, res) => {
  const user = getUser(req.params.userId);
  if (!user) {
    return res.status(404).json({ 
      error: 'User not found',
      userId: req.params.userId 
    });
  }
  res.json(user);
});
```

### 500 - Internal Server Error
Retorne se for um erro inesperado.

**Exemplo:**
```typescript
app.get('/users', (req, res) => {
  try {
    const users = getUsers();
    res.json(users);
  } catch (error) {
    console.error('Unexpected error fetching users', { error });
    res.status(500).json({ 
      error: 'Internal server error',
      message: 'An unexpected error occurred'
    });
  }
});
```

### 422 - Unprocessable Entity
Retorne se for um erro de negócio.

**Exemplo:**
```typescript
app.post('/orders/:orderId/cancel', (req, res) => {
  const order = getOrder(req.params.orderId);
  
  if (order.status === 'shipped') {
    return res.status(422).json({ 
      error: 'Cannot cancel shipped order',
      orderId: order.id,
      currentStatus: order.status
    });
  }
  
  cancelOrder(order.id);
  res.json({ message: 'Order cancelled successfully' });
});
```

### 400 - Bad Request
Retorne se a requisição não estiver bem formatada.

**Exemplo:**
```typescript
app.post('/users', (req, res) => {
  const { name, email } = req.body;
  
  if (!name || !email) {
    return res.status(400).json({ 
      error: 'Missing required fields',
      required: ['name', 'email']
    });
  }
  
  if (!isValidEmail(email)) {
    return res.status(400).json({ 
      error: 'Invalid email format',
      field: 'email'
    });
  }
  
  const user = createUser({ name, email });
  res.json(user);
});
```

### 401 - Unauthorized
Retorne se o usuário não estiver autenticado.

**Exemplo:**
```typescript
app.get('/profile', (req, res) => {
  const token = req.headers.authorization?.replace('Bearer ', '');
  
  if (!token) {
    return res.status(401).json({ 
      error: 'Authentication required',
      message: 'Please provide a valid token'
    });
  }
  
  const user = verifyToken(token);
  if (!user) {
    return res.status(401).json({ 
      error: 'Invalid token',
      message: 'Token is expired or invalid'
    });
  }
  
  res.json(user);
});
```

### 403 - Forbidden
Retorne se o usuário não estiver autorizado.

**Exemplo:**
```typescript
app.delete('/users/:userId', (req, res) => {
  const currentUser = getCurrentUser(req);
  const targetUserId = req.params.userId;
  
  // Usuário está autenticado (401), mas não tem permissão
  if (currentUser.role !== 'admin' && currentUser.id !== targetUserId) {
    return res.status(403).json({ 
      error: 'Insufficient permissions',
      message: 'You are not allowed to delete this user'
    });
  }
  
  deleteUser(targetUserId);
  res.json({ message: 'User deleted successfully' });
});
```

## Cliente HTTP

Utilize Axios para fazer chamadas para API externa.

**Exemplo:**
```typescript
import axios from 'axios';

// GET request
async function getUser(userId: string) {
  try {
    const response = await axios.get(`https://api.example.com/users/${userId}`);
    return response.data;
  } catch (error) {
    if (axios.isAxiosError(error)) {
      console.error('API request failed', { 
        status: error.response?.status,
        message: error.message 
      });
    }
    throw error;
  }
}

// POST request
async function createUser(userData: CreateUserData) {
  const response = await axios.post('https://api.example.com/users', userData, {
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${token}`
    }
  });
  return response.data;
}

// Configurar instância com defaults
const api = axios.create({
  baseURL: 'https://api.example.com',
  timeout: 5000,
  headers: {
    'Content-Type': 'application/json'
  }
});

api.interceptors.request.use(config => {
  const token = getAuthToken();
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});
```

## Middlewares

Utilize middlewares para funcionalidades transversais.

**Exemplo:**
```typescript
// Middleware de autenticação
function authenticate(req: Request, res: Response, next: NextFunction) {
  const token = req.headers.authorization?.replace('Bearer ', '');
  
  if (!token) {
    return res.status(401).json({ error: 'Authentication required' });
  }
  
  const user = verifyToken(token);
  if (!user) {
    return res.status(401).json({ error: 'Invalid token' });
  }
  
  req.user = user;
  next();
}

// Middleware de validação
function validateUserInput(req: Request, res: Response, next: NextFunction) {
  const { name, email } = req.body;
  
  if (!name || !email) {
    return res.status(400).json({ error: 'Missing required fields' });
  }
  
  next();
}

// Uso
app.post('/users', authenticate, validateUserInput, (req, res) => {
  const user = createUser(req.body);
  res.json(user);
});
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rodrigobranas) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
