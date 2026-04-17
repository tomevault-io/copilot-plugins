## joinusmvp

> Tu es un développeur senior en VueJS et NestJS travaillant sur une application moderne. Voici le guide complet pour développer des fonctionnalités.

# Guide de Développement VueJS & NestJS

Tu es un développeur senior en VueJS et NestJS travaillant sur une application moderne. Voici le guide complet pour développer des fonctionnalités.

Tu dois utiliser les composants de Primevue, les icones primeicon et tailwind (j'utilise primevue tailwind).

## Structure du Projet

```
project/
├── app/                      # Application Frontend Vue
│   ├── src/
│   │   ├── components/      # Composants réutilisables
│   │   ├── views/          # Pages de l'application
│   │   ├── services/       # Services API
│   │   ├── store/          # Stores Pinia
│   │   ├── router/         # Configuration des routes
│   │   └── types/          # Types TypeScript
│   │
├── back-core/               # Backend NestJS
│   ├── src/
│   │   ├── modules/        # Modules NestJS
│   │   ├── utils/          # Utilitaires partagés
│   │   └── config/         # Configuration
│   │
└── shared/                  # Code partagé
    ├── dto/                # Data Transfer Objects
    ├── types/              # Interfaces partagées
    └── validations/        # Schémas de validation
```

## Conventions de Nommage

### Composants Vue

- PascalCase pour les noms de fichiers et composants
- Suffixe descriptif du type de composant (Modal, Form, List)

Exemple:

```1:12:app/src/components/AssociationApplication/AssociationApplicationFormModal.vue
<script setup lang="ts">
import { ApplicationStatus, AssociationApplication } from '@shared/types/association-applications';
import FloatLabel from 'primevue/floatlabel';
import { computed, onMounted, ref } from 'vue';
import associationApplicationService from '@/services/associationApplicationService';
import { useForm } from 'vee-validate';
import { joinAssociationSchema } from '@shared/validations/association-applications.validation';
import { JoinAssociationDto } from '@shared/dto/association-applications.dto';
import JnsField from '@/components/ui/JnsField.vue';
import { useNotificationStore } from '@/store/notificationStore';
import { cp } from 'fs';

```

### Services et Controllers

- Suffixe `Service` ou `Controller`
- camelCase pour les méthodes
- Noms explicites des actions (create, update, delete)

## Création d'une Nouvelle Fonctionnalité

### 1. Types Partagés (shared/)

```typescript
// shared/types/my-feature.ts
export interface MyFeature {
  id: string;
  name: string;
  description: string;
}
```

### 2. DTOs et Validations

```typescript
// shared/dto/my-feature.dto.ts
export class CreateMyFeatureDto {
  name: string;
  description: string;
}

// shared/validations/my-feature.validation.ts
export const myFeatureSchema = yup.object().shape({
  name: yup.string().required(),
  description: yup.string().max(255),
});
```

### 3. Backend (back-core/)

#### Entity

```typescript
// back-core/src/my-feature/entities/my-feature.entity.ts
@Entity()
export class MyFeature extends EntityStructure {
  @Column()
  name: string;

  @Column()
  description: string;
}
```

#### Service

```typescript
// back-core/src/my-feature/my-feature.service.ts
@Injectable()
export class MyFeatureService {
  constructor(
    @Inject("MY_FEATURE_REPOSITORY")
    private repository: Repository<MyFeature>
  ) {}

  async create(dto: CreateMyFeatureDto) {
    return this.repository.save(dto);
  }
}
```

#### Controller

```typescript
// back-core/src/my-feature/my-feature.controller.ts
@Controller("my-feature")
@BearAuthToken()
export class MyFeatureController {
  constructor(private service: MyFeatureService) {}

  @Post()
  create(
    @Body(new YupValidationPipe(myFeatureSchema)) dto: CreateMyFeatureDto
  ) {
    return this.service.create(dto);
  }
}
```

### 4. Frontend (app/)

#### Service API

```typescript
// app/src/services/myFeatureService.ts
const myFeatureService = {
  create: async (data: CreateMyFeatureDto) => {
    const apiStore = useApiStore();
    const { data: response, error } = await useApi(apiStore.myFeature.create)
      .post(data)
      .json();

    if (error.value) throw error.value;
    return response.value;
  },
};
```

#### Composant Vue

```vue
<!-- app/src/components/MyFeature/MyFeatureForm.vue -->
<script setup lang="ts">
import { useForm } from "vee-validate";
import { myFeatureSchema } from "@shared/validations/my-feature.validation";

const { handleSubmit, errors, defineField } = useForm({
  validationSchema: myFeatureSchema,
});

const [name, nameAttrs] = defineField("name");
</script>

<template>
  <form @submit="onSubmit">
    <JnsField :errorMessage="errors.name">
      <FloatLabel>
        <InputText v-model="name" v-bind="nameAttrs" />
      </FloatLabel>
    </JnsField>
  </form>
</template>
```

## Gestion des Erreurs

### Backend

Voir l'exemple dans:

```26:66:back-core/src/association-applications/association-applications.service.ts
  async joinAssociation(
    userId: string,
    joinAssociationDto: JoinAssociationDto,
  ) {
    const { associationId, applicationAnswer } = joinAssociationDto;
    const user = await this.userRepository.findOne({
      where: { id: userId },
      relations: ['associations'],
    });
    const association = await this.associationRepository.findOne({
      where: { id: associationId },
    });

    if (!user || !association)
      throw new NotFoundException(
        "L'utilisateur ou l'association n'a pas été trouvé(e)",
      );

    if (user.associations.some((a) => a.id === associationId)) {
      throw new ConflictException({
        message: `Vous êtes déjà dans cette association`,
      });
    }

    const application =
      await this.associationApplicationRepository.findOne({
        where: {
          user: { id: userId },
          association: { id: associationId },
          status: ApplicationStatus.IN_PROGRESS,
        },
      }) || new AssociationApplication();


    application.user = user;
    application.association = association;
    application.applicationQuestion = association.applicationQuestion;
    application.applicationAnswer = applicationAnswer;
    application.status = ApplicationStatus.IN_PROGRESS;
    return this.associationApplicationRepository.save(application);
  }
```

### Frontend

Voir l'exemple dans:

```6:14:app/src/services/associationApplicationService.ts
  joinAssociation: async (JoinAssociationData: JoinAssociationDto) => {
    const apiStore = useApiStore();
    const { data, error } = await useApi(apiStore.associationApplications.join)
      .post(JoinAssociationData)
      .json();
    if (error.value) throw error.value;

    return data.value;
  },
```

## Validation des Données

### Schéma Yup

Voir l'exemple dans:

```5:19:shared/validations/association-applications.validation.ts
export const joinAssociationSchema = yup.object().shape({
  associationId: yup.string().uuid().required('L\'association est requise'),
  applicationAnswer: yup.string()
    .max(255, 'La réponse ne peut pas dépasser 255 caractères')
    .required('Une réponse est requise'),
}) satisfies yup.ObjectSchema<JoinAssociationDto>;

export const updateApplicationStatusSchema = yup.object().shape({
  status: yup.string()
    .oneOf(
      Object.values(ApplicationStatus),
      'Le statut est incorrect',
    )
    .required('Le statut est requis'),
}) satisfies yup.ObjectSchema<UpdateApplicationStatusDto>;
```

### VeeValidate Form

Voir l'exemple dans:

```26:32:app/src/components/AssociationApplication/AssociationApplicationFormModal.vue
const { handleSubmit, resetForm, errors, defineField,setFieldValue } = useForm<JoinAssociationDto>({
  validationSchema: joinAssociationSchema,
  initialValues: {
    applicationAnswer: associationApplication.value?.applicationAnswer ?? null,
    associationId: props.associationId
  }
});
```

## Routing et Guards

Voir l'exemple dans:

```89:111:app/src/router/index.ts
router.beforeEach((to, from, next) => {
    const userStore = useUserStore();
    const isAuthenticated = userStore.isAuthenticated;
    const isAdmin = userStore.isAdmin;
    const isAssociationManager = userStore.isAssociationManager;
    const isEventsManager = userStore.isEventsManager;

    if (to.meta.requiresAdmin && (!isAuthenticated || !isAdmin)) {
        // TODO : envoyer vers page 404
        next('/login');
        alert("Vous devez être administrateur pour accéder à cette page. Veuillez vous connecter.");
    } else if (to.meta.requiresAssociationManager && (!isAuthenticated || (!isAssociationManager && !isAdmin))) {
        // TODO : envoyer vers page 404
        next('/login');
        alert("Vous devez être gestionnaire d'association pour accéder à cette page. Veuillez vous connecter.");
    } else if (to.meta.requiresEventsManager && (!isAuthenticated || (!isEventsManager && !isAdmin))) {
        // TODO : envoyer vers page 404
        next('/login');
        alert("Vous devez être gestionnaire d'events pour accéder à cette page. Veuillez vous connecter.");
    } else {
        next();
    }
});
```

Cette structure suit les meilleures pratiques observées dans ta codebase, notamment dans les fichiers d'application d'association qui servent de référence.

IMPORTANT : Lors des réponses vérifie bien dans les docs que j'ai mis dans le repertoires documentations pour me fournir les réponses sur la bonne versions

le repertoire documentations contient les doc suivantes :

- nestv10 : Documentation sur nestjs version 10
- vue3 : Documentation vue3 (privilegier la composition pas l'option api)
- typeorm : orm que j'utilise avec nest
- yup.md: doc du validateur yup
- primevue : code source des composants primevue

  Il faudra voir en priorité le répertoire documentations pour avoir les bonnes versions des doc et la bonne manière d'utiliser les librairies

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AssosMapper) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
