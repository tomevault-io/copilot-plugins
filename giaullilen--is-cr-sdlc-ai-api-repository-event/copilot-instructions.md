## is-cr-sdlc-ai-api-repository-event

> Es un API que debe trabajar bajo una arquitectura en capas, la estructura que se debe respetar es la siguiente:


# Proyecto is-cr-sdlc-ai-api-repository-event

Es un API que debe trabajar bajo una arquitectura en capas, la estructura que se debe respetar es la siguiente:

```
La estructura es la siguiente:
is-cr-sdlc-ai-api-repository-event/
├── src/                         # Directorio de código fuente
│   ├── entities/                # Entidades/modelos de datos para la aplicación, son DTOs que representan las entidades que vienen como request y se envían como response
│   ├── mappers/                 # Lógica de transformación de datos entre diferentes formatos
│   ├── middleware/              # Funciones de middleware para procesamiento de solicitudes/mensajes
│   │   └── context.ts           # Gestión de contexto para trazabilidad
│   ├── repositories/            # Capa de acceso a datos para operaciones de base de datos
│   │   └── models/              # Modelos de base de datos que representan tablas
│   ├── integrations/            # Capa para integrarse con APIs externas
│   │   └── models/              # Modelos de datos que representan los request y response de las APIs a consumir
│   ├── publisher/               # Capa para enviar mensajes a topicos pub/sub
│   │   └── models/              # Modelos de los mensajes que se van a enviar
│   ├── services/                # Capa de lógica de negocio
│   ├── controllers/             # Capa de Controladores HTTP
│   ├── routes/                  # Definición de rutas
│   └── utils/                   # Funciones de utilidad y ayudantes
│       ├── environment.ts       # Aqui estan las variables de entorno que se usa en el sistema
│       ├── logger.ts            # Componente para usar un objeto de registro de logs
│       └── database.ts          # Utilidad de conexión a base de datos PostgreSQL
├── tests                        # Pruebas unitarias del proyecto
├── index.ts                     # Punto de entrada principal
├── Dockerfile                   # Dockerfile para  generación de la imagen docker
└── package.json                 # Dependencias de Node y configuración del proyecto
```

# Lineamientos

## Trazabilidad

- En el "controller" se debe crear un objeto de contexto y este objeto se deberá pasar a todas las capas que se ejecutan en el servicio, este objeto contiene un "applicationId" y "transactionId", y estos valores viajan para ser usados en el registro de logs.

Ejemplo de controller:
```typescript
import { Request, ResponseToolkit } from '@hapi/hapi';
import * as Boom from '@hapi/boom';
import * as cumuloService from '../services/cumulo.service';
import { getLogger } from '../utils/logger';
import { Context, ContextRequestApplicationState } from '../middleware/context';

export const getCumuloByDoc = async (request: Request, h: ResponseToolkit) => {
  const doc = request.params.doc;
  const ctx = getContext(request);
  getLogger(ctx).debug(`getCumuloByDoc - Documento a buscar: ${doc}`);

  try {
    const cumulo = await cumuloService.getCumuloByDoc(ctx, doc);
    if (!cumulo) {
      getLogger(ctx).warn(`Registro con numero de documento ${doc} no encontrado`);
      return h.response({
        tipoDocumento: "DOCUMENTO DE IDENTIDAD",
        numeroDocumento: doc,
        cumuloDolar: null,
        cumuloSoles: null
      });
      //return Boom.notFound('No se encontro el registro');
    }
    return h.response(cumulo);
  } catch (error: any) {
    getLogger(ctx).error(error);
    const e = Boom.internal("Error interno");
    e.output.payload.message = 'Error en la consulta';
    return e;
  }
}

const getContext = (request: Request): Context => {
  return (request.app as ContextRequestApplicationState).context;
}
```

## Acceso a Base de datos
- Se usa TypeORM para el acceso a base de datos, el acceso se da dentro de la carpeta src/repositories.
- Se debe usar el "model" que representa la tabla u objeto de base de datos que se requiere.
- Se debe usar un objeto "repository" que se obtiene desde el objeto "datasource" de "/src/utils/database.ts".

Ejemplo de repository:
```typescript
import { Cumulo } from "../models/cumulo.model";
import { dataSource } from "../utils/database";
import { getLogger } from '../utils/logger';
import { Context } from '../middleware/context';

const repository = dataSource.getRepository(Cumulo);

export class CumuloRepository {

    async findByDoc(ctx: Context, numerodocumento: string): Promise<Cumulo | null> {
        getLogger(ctx).debug('Buscando por doc:', numerodocumento);
        try {
            const result = await repository.findOne({
                where: { numerodocumento }
            });

            getLogger(ctx).debug(`Resultado de la consulta: ${JSON.stringify(result)}`);
            return result as Cumulo;
        } catch (error: any) {
            getLogger(ctx).error('Error al buscar por documento:', error);
            return null;
        }
    }
}
```

- Se tiene que mapear cada tabla a una clase de typescript dentro de la carpeta src/repositories/models
Ejemplo de model:
```typescript
import { Column, Entity, PrimaryColumn } from 'typeorm';

@Entity({ name: 'vw_cumulo', synchronize: false })
export class Cumulo {
  @PrimaryColumn()
  tpt_id: number;

  @Column({ nullable: false })
  tipodocumento: string;

  @Column({ nullable: false })
  numerodocumento: string;

  @Column({ nullable: true })
  cumulo_dolar: number;

  @Column({ nullable: true })
  cumulo_soles: number;
}
```

- Para la conexión a las bases de datos se tiene que configurar estos datos en "/src/utils/database.ts". Se debe validar si existe la dependencia en el proyecto.

Ejemplo de conexión para postgres:
```typescript
import { DataSource } from 'typeorm';
import { Cumulo } from '../models/cumulo.model';

const { LOG_LEVEL, DATABASE_HOST, DATABASE_NAME, DATABASE_USER, DATABASE_PASSWORD } = require('../utils/environments');

if (!DATABASE_HOST || !DATABASE_NAME || !DATABASE_USER || !DATABASE_PASSWORD) {
  throw new Error('Environment variables for database are not set properly');
}
export const dataSource = new DataSource({
  type: "postgres",
  host: DATABASE_HOST,
  username: DATABASE_USER,
  password: DATABASE_PASSWORD,
  database: DATABASE_NAME,
  entities: [Cumulo],
  synchronize: false,
  logging: LOG_LEVEL,
});

export const connectToDatabase = async (): Promise<void> => {
  console.log(`Conectando a ${DATABASE_HOST}...`);
  dataSource
    .initialize()
    .then(() => {
      console.log('Conectado a la base de datos');
    })
    .catch((error) => {
      console.error('Error al conectarse a la base de datos:', error);
      process.exit(1);
    });
};

export const checkConnection = (): string => {
  const isInitialized = dataSource.isInitialized;
  if (isInitialized) {
    return "OK";
  }
  throw new Error("ERROR");
}
```

- Por cada model creado se debe registrar en "database.ts" dentro de la funcion 'datasource' en el parámetro 'entities'.

## Registro de logs

- Se debe registrar logs por cada evento relevante para la ejecución del proceso.
- Estos logs no aplican para los archivos "index.ts", los routes, "context.ts" y "database.ts" debido a que cuando se ejecutan estos codigos aún no se ha creado el context, entonces para estos casos se debe usar el "console.log".
- Para el registro de un log se debe importar la función `getLogger` de `utils/logger` y usar el objeto `context` que se propaga por todas las funciones para pintar un mensaje de log.
Ejemplo:
```typescript
import { getLogger } from '../utils/logger';
...
getLogger(context).debug(`Inicio del metodo getPostAndUser, idPost: ${idPost}`);
```

- Para mostrar variables en los mensajes del log debes usar template literals como el siguente ejemplo:
```typescript
getLogger(ctx).debug(`registerIngress - Campo a registrar: ${campo}`);
```

## Configuración de parámetros
- Los parámetros de configuración se deben mantener en un archivo llamado utils/environment el cual contentrá todas las variables que usa el sistema, de esta manera se evita confusiones y variables faltantes cuando se despliegue o ejecute el servicio.

Ejemplo de environment.ts:
```typescript
export const LOG_LEVEL = process.env.LOG_LEVEL || 'info';

// Variables de entorno
export const APIPOST_URL = process.env.APIPOST_URL;
```

- Para usar estas variables se debe importar desde el archivo `environment.ts`. Ejemplo de uso:
```typescript
import { API_PLACEHOLDER_URL } from '../utils/environment';
...
apiPostClient = new ApiPostPlaceHolder(API_PLACEHOLDER_URL);
```

## Integraciones con APIs externas
- Se debe usar la biblioteca `axios`, si no le tienen en su "package.json", agregarla.
- Estas integraciones se deben hacer en la carpeta "integrations" como una clase que abstraiga estas invocaciones a un API, la URL deberá estar configurada en "environment.ts".
- Dentro de "integrations" debe haber una carpeta llamada "models" con las clases que representen a las entidades de request y response de las APIs que se vayan a consumir.

## Capa de routes

En esta capa se registrarán los endpoints que tienen el API, 
Ejemplo de definición de los routes:

```typescript
export const exampleRoutes = (server: Server) => {
  // Definicion de los routes
  server.route({
    method: 'POST',
    path: '/api/example/register',
    options: {
      description: 'Registrar un nuevo elemento',
      notes: 'Registra un nuevo elemento al sistema X',
      tags: ['api'],
      validate: {
        payload: recordRequestSchema,
        failAction: validationFailAction
      }
    },
    handler: recordWorkerController.registerEntry
  });
```

Y en el index.ts se tiene que usar esta función de routes, ejemplo:
```typescript
exampleRoutes(server);
```

## Pruebas unitarias
Genera pruebas unitarias en la carpeta "tests" de lo que generes y que sea reproducible en un flujo de devops.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/giaUllilen)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/giaUllilen)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
