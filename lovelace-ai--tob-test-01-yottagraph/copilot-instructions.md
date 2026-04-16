## tob-test-01-yottagraph

> Copy-paste UI patterns for common pages: entity search, data table, form, chart, master-detail. Read when building new pages or features.


# UI Pattern Cookbook

Copy-paste patterns using the project's actual composables and Vuetify components. Adapt to your needs.

## 1. Entity Search Page

Search for entities by name and display results.

```vue
<template>
    <div class="d-flex flex-column fill-height pa-4">
        <h1 class="text-h5 mb-4">Entity Search</h1>
        <v-text-field
            v-model="query"
            label="Search entities"
            prepend-inner-icon="mdi-magnify"
            variant="outlined"
            @keyup.enter="search"
            :loading="loading"
        />
        <v-alert v-if="error" type="error" variant="tonal" class="mt-2" closable>
            {{ error }}
        </v-alert>
        <v-list v-if="results.length" class="mt-4">
            <v-list-item
                v-for="(neid, i) in results"
                :key="neid"
                :title="names[i] || neid"
                :subtitle="neid"
            />
        </v-list>
        <v-empty-state
            v-else-if="searched && !loading"
            headline="No results"
            icon="mdi-magnify-remove-outline"
        />
    </div>
</template>

<script setup lang="ts">
    import { useElementalClient } from '@yottagraph-app/elemental-api/client';

    const client = useElementalClient();
    const query = ref('');
    const results = ref<string[]>([]);
    const names = ref<string[]>([]);
    const loading = ref(false);
    const error = ref<string | null>(null);
    const searched = ref(false);

    async function search() {
        if (!query.value.trim()) return;
        loading.value = true;
        error.value = null;
        searched.value = true;
        try {
            const res = await client.getNEID({
                entityName: query.value.trim(),
                maxResults: 10,
                includeNames: true,
            });
            results.value = res.neids || [];
            names.value = res.names || [];
        } catch (e: any) {
            error.value = e.message || 'Search failed';
            results.value = [];
        } finally {
            loading.value = false;
        }
    }
</script>
```

## 2. Data Table Page

Sortable table with loading, empty, and error states.

```vue
<template>
    <div class="d-flex flex-column fill-height pa-4">
        <h1 class="text-h5 mb-4">Data Table</h1>
        <v-alert v-if="error" type="error" variant="tonal" class="mb-4" closable>
            {{ error }}
        </v-alert>
        <v-data-table
            :headers="headers"
            :items="items"
            :loading="loading"
            density="comfortable"
            hover
        >
            <template v-slot:item.actions="{ item }">
                <v-btn icon size="small" variant="text" @click="onView(item)">
                    <v-icon>mdi-eye</v-icon>
                </v-btn>
            </template>
            <template v-slot:no-data>
                <v-empty-state headline="No data" icon="mdi-database-off" />
            </template>
        </v-data-table>
    </div>
</template>

<script setup lang="ts">
    const headers = [
        { title: 'Name', key: 'name', sortable: true },
        { title: 'Type', key: 'type', sortable: true },
        { title: 'Updated', key: 'updatedAt', sortable: true },
        { title: '', key: 'actions', sortable: false, width: 60 },
    ];

    const items = ref<any[]>([]);
    const loading = ref(false);
    const error = ref<string | null>(null);

    function onView(item: any) {
        // Handle row action
    }

    onMounted(async () => {
        loading.value = true;
        try {
            // items.value = await fetchData();
        } catch (e: any) {
            error.value = e.message || 'Failed to load';
        } finally {
            loading.value = false;
        }
    });
</script>
```

## 3. Form with Validation

Form with field validation, submit handler, and user feedback.

```vue
<template>
    <v-container class="py-6" style="max-width: 600px">
        <h1 class="text-h5 mb-4">Settings</h1>
        <v-form ref="formRef" v-model="valid" @submit.prevent="submit">
            <v-text-field
                v-model="form.name"
                label="Name"
                :rules="[rules.required]"
                variant="outlined"
                class="mb-3"
            />
            <v-text-field
                v-model="form.email"
                label="Email"
                :rules="[rules.required, rules.email]"
                variant="outlined"
                class="mb-3"
            />
            <v-select
                v-model="form.role"
                :items="['Admin', 'Editor', 'Viewer']"
                label="Role"
                variant="outlined"
                class="mb-3"
            />
            <v-btn type="submit" color="primary" :loading="saving" :disabled="!valid">
                Save
            </v-btn>
        </v-form>
    </v-container>
</template>

<script setup lang="ts">
    import { useNotification } from '~/composables/useNotification';

    const { showSuccess, showError } = useNotification();
    const formRef = ref();
    const valid = ref(false);
    const saving = ref(false);

    const form = reactive({ name: '', email: '', role: 'Viewer' });

    const rules = {
        required: (v: string) => !!v || 'Required',
        email: (v: string) => /.+@.+\..+/.test(v) || 'Invalid email',
    };

    async function submit() {
        const { valid: isValid } = await formRef.value.validate();
        if (!isValid) return;
        saving.value = true;
        try {
            // await saveSettings(form);
            showSuccess('Settings saved');
        } catch (e: any) {
            showError(e.message || 'Failed to save');
        } finally {
            saving.value = false;
        }
    }
</script>
```

## 4. Chart Page

Chart.js chart with dark theme colors.

```vue
<template>
    <div class="d-flex flex-column fill-height pa-4">
        <h1 class="text-h5 mb-4">Analytics</h1>
        <v-card class="flex-grow-1 pa-4">
            <canvas ref="chartCanvas" />
        </v-card>
    </div>
</template>

<script setup lang="ts">
    import { Chart, registerables } from 'chart.js';
    Chart.register(...registerables);

    const chartCanvas = ref<HTMLCanvasElement | null>(null);
    let chart: Chart | null = null;

    onMounted(() => {
        if (!chartCanvas.value) return;
        chart = new Chart(chartCanvas.value, {
            type: 'line',
            data: {
                labels: ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun'],
                datasets: [
                    {
                        label: 'Mentions',
                        data: [12, 19, 3, 5, 2, 15],
                        borderColor: '#3fea00',
                        backgroundColor: 'rgba(63, 234, 0, 0.1)',
                        fill: true,
                        tension: 0.3,
                    },
                ],
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                scales: {
                    x: { ticks: { color: '#999' }, grid: { color: '#222' } },
                    y: { ticks: { color: '#999' }, grid: { color: '#222' } },
                },
                plugins: {
                    legend: { labels: { color: '#e5e5e5' } },
                },
            },
        });
    });

    onUnmounted(() => chart?.destroy());
</script>
```

Note: `chart.js` must be installed (`npm install chart.js`). Use the brand colors: `#3fea00` (green), `#003bff` (blue), `#ff5c00` (orange).

## 5. Dialog

Confirmation or form dialog. The global `VCard` default is `variant: 'outlined'` (transparent background), but `nuxt.config.ts` sets a nested `VDialog > VCard` default of `variant: 'flat'` so dialog cards get a solid background automatically. No manual override needed.

```vue
<template>
    <v-dialog v-model="open" max-width="500" persistent>
        <v-card>
            <v-card-title class="d-flex align-center">
                <span>Confirm Action</span>
                <v-spacer />
                <v-btn icon variant="text" @click="open = false">
                    <v-icon>mdi-close</v-icon>
                </v-btn>
            </v-card-title>
            <v-divider />
            <v-card-text>
                Are you sure you want to proceed? This action cannot be undone.
            </v-card-text>
            <v-divider />
            <v-card-actions>
                <v-spacer />
                <v-btn variant="text" @click="open = false">Cancel</v-btn>
                <v-btn color="primary" :loading="loading" @click="confirm">Confirm</v-btn>
            </v-card-actions>
        </v-card>
    </v-dialog>
</template>

<script setup lang="ts">
    const open = defineModel<boolean>({ default: false });
    const emit = defineEmits<{ confirmed: [] }>();
    const loading = ref(false);

    async function confirm() {
        loading.value = true;
        try {
            emit('confirmed');
            open.value = false;
        } finally {
            loading.value = false;
        }
    }
</script>
```

## 6. Master-Detail View

Two-column layout with selectable list and detail panel.

```vue
<template>
    <div class="d-flex fill-height">
        <!-- List panel -->
        <v-card class="flex-shrink-0" style="width: 320px; overflow-y: auto" flat>
            <v-list density="compact" nav>
                <v-list-item
                    v-for="item in items"
                    :key="item.id"
                    :title="item.name"
                    :subtitle="item.type"
                    :active="selected?.id === item.id"
                    @click="selected = item"
                />
            </v-list>
            <v-empty-state
                v-if="!items.length"
                headline="No items"
                icon="mdi-playlist-remove"
                density="compact"
            />
        </v-card>

        <v-divider vertical />

        <!-- Detail panel -->
        <div class="flex-grow-1 overflow-y-auto pa-4">
            <template v-if="selected">
                <h2 class="text-h5 mb-2">{{ selected.name }}</h2>
                <v-chip class="mb-4">{{ selected.type }}</v-chip>
                <p>{{ selected.description }}</p>
            </template>
            <v-empty-state
                v-else
                headline="Select an item"
                text="Choose from the list to see details"
                icon="mdi-arrow-left"
            />
        </div>
    </div>
</template>

<script setup lang="ts">
    interface Item {
        id: string;
        name: string;
        type: string;
        description: string;
    }

    const items = ref<Item[]>([]);
    const selected = ref<Item | null>(null);

    onMounted(async () => {
        // items.value = await fetchItems();
    });
</script>
```

## 7. Get Filings for a Company

Fetch Edgar filings (or any relationship-linked documents) for an organization.

**Important:** `getLinkedEntities` only supports graph node types (person,
organization, location). Documents, filings, articles, and other types are
NOT supported — use `getPropertyValues` with the relationship PID instead.

```vue
<template>
    <div class="d-flex flex-column fill-height pa-4">
        <h1 class="text-h5 mb-4">Company Filings</h1>
        <v-text-field
            v-model="query"
            label="Company name"
            prepend-inner-icon="mdi-magnify"
            @keyup.enter="search"
            :loading="loading"
        />
        <v-alert v-if="error" type="error" variant="tonal" class="mt-2" closable>
            {{ error }}
        </v-alert>
        <v-data-table
            v-if="filings.length"
            :headers="headers"
            :items="filings"
            :loading="loading"
            density="comfortable"
            hover
            class="mt-4"
        />
        <v-empty-state
            v-else-if="searched && !loading"
            headline="No filings found"
            icon="mdi-file-document-off"
        />
    </div>
</template>

<script setup lang="ts">
    import { useElementalClient } from '@yottagraph-app/elemental-api/client';

    const client = useElementalClient();
    const query = ref('');
    const filings = ref<{ neid: string; name: string }[]>([]);
    const loading = ref(false);
    const error = ref<string | null>(null);
    const searched = ref(false);

    const headers = [
        { title: 'NEID', key: 'neid', sortable: true },
        { title: 'Name', key: 'name', sortable: true },
    ];

    async function getPropertyPidMap(client: ReturnType<typeof useElementalClient>) {
        const schemaRes = await client.getSchema();
        const properties = schemaRes.schema?.properties ?? (schemaRes as any).properties ?? [];
        return new Map(properties.map((p: any) => [p.name, p.pid]));
    }

    async function search() {
        if (!query.value.trim()) return;
        loading.value = true;
        error.value = null;
        searched.value = true;
        try {
            const lookup = await client.getNEID({
                entityName: query.value.trim(),
                maxResults: 1,
                includeNames: true,
            });
            if (!lookup.neids?.length) {
                filings.value = [];
                return;
            }
            const orgNeid = lookup.neids[0];

            const pidMap = await getPropertyPidMap(client);
            const filedPid = pidMap.get('filed');
            if (!filedPid) {
                error.value = '"filed" relationship not found in schema';
                return;
            }

            const res = await client.getPropertyValues({
                eids: JSON.stringify([orgNeid]),
                pids: JSON.stringify([filedPid]),
            });

            const docNeids = res.values.map((v: any) =>
                String(v.value).padStart(20, '0'),
            );

            const names = await Promise.all(
                docNeids.map(async (neid: string) => {
                    try {
                        const r = await client.getNamedEntityReport(neid);
                        return r.report?.name ?? (r as any).name ?? neid;
                    } catch {
                        return neid;
                    }
                }),
            );

            filings.value = docNeids.map((neid: string, i: number) => ({
                neid,
                name: names[i],
            }));
        } catch (e: any) {
            error.value = e.message || 'Failed to load filings';
            filings.value = [];
        } finally {
            loading.value = false;
        }
    }
</script>
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Lovelace-AI) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
