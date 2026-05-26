<template>
  <v-container fluid class="d-flex justify-center">
    <v-container style="width: 100%; max-width: 1920px" class="pa-0">
      <v-container fluid class="pt-2 pr-0 pl-0 pb-2">
        <v-row>
          <v-col cols="auto" class="d-flex align-center justify-center" style="height: 40px">
            <v-icon @click="$router.back()" class="mr-2" style="vertical-align: middle">mdi-arrow-left</v-icon>
          </v-col>
          <div class="d-flex align-center ga-3 mb-4" style="height: 40px">
            <div style="width: 4px; height: 32px; border-radius: 2px; background: rgb(var(--v-theme-primary))"></div>
            <h2 class="font-weight-medium ma-0" style="font-weight: 600; line-height: 1.1">{{ $t('iscsi') }}</h2>
          </div>
        </v-row>
      </v-container>

      <v-container fluid class="pa-0">
        <v-card class="pa-0" style="margin-bottom: 80px">
          <v-card-text class="pt-0">
            <v-tabs v-model="activeTab" bg-color="transparent" color="primary" class="mb-3">
              <v-tab value="server">
                <div class="d-flex align-center ga-2">
                  <v-icon size="small">mdi-server</v-icon>
                  <span>{{ $t('iscsi server') }}</span>
                  <v-chip size="x-small" color="primary" variant="tonal">
                    {{ targets.length }}
                  </v-chip>
                </div>
              </v-tab>
              <v-tab value="client">
                <div class="d-flex align-center ga-2">
                  <v-icon size="small">mdi-connection</v-icon>
                  <span>{{ $t('iscsi client') }}</span>
                  <v-chip size="x-small" color="primary" variant="tonal">
                    {{ initiatorTargets.length }}
                  </v-chip>
                </div>
              </v-tab>
            </v-tabs>

            <v-window v-model="activeTab">

              <!-- Server -->              
              <v-window-item value="server">
                <v-col cols="12" class="d-flex align-center justify-space-between">
                  <span class="text-title-medium font-weight-medium">{{ $t('targets') }}</span>
                  <v-btn size="small" variant="text" color="primary" prepend-icon="mdi-plus" @click="addTarget()">
                    {{ $t('add') }}
                  </v-btn>
                </v-col>

                <v-expansion-panels v-if="targets.length > 0" variant="accordion">
                  <v-expansion-panel v-for="(target, i) in targets" :key="i">
                    <v-expansion-panel-title>
                      <div class="d-flex align-center ga-3 w-100 pr-2">
                        <div class="flex-grow-1">
                          <div class="font-weight-medium">{{ target.name || $t('new target') }}</div>
                          <div class="text-caption text-medium-emphasis">{{ target.iqn }}</div>
                        </div>
                        <div class="d-flex align-center ga-2 flex-shrink-0">
                          <span class="text-caption text-medium-emphasis">{{ target.portal }}</span>
                          <v-chip size="x-small" :color="target.isActive ? 'green' : 'default'" variant="tonal">
                            {{ target.isActive ? $t('active') : $t('inactive') }}
                          </v-chip>
                        </div>
                      </div>
                    </v-expansion-panel-title>
                    <v-expansion-panel-text>
                      <v-row class="mt-1">
                        <v-col cols="12" sm="4">
                          <v-text-field v-model="target.name" :label="$t('name')" density="compact" hide-details="auto"></v-text-field>
                        </v-col>
                        <v-col cols="12" sm="4">
                          <v-text-field v-model="target.portal" :label="$t('portal')" placeholder="0.0.0.0:3260" density="compact" hide-details="auto"></v-text-field>
                        </v-col>
                        <v-col cols="12" sm="4">
                          <v-text-field v-model="target.iqn" :label="$t('target iqn')" placeholder="iqn.2024-01.com.example:storage" density="compact" hide-details="auto"></v-text-field>
                        </v-col>
                      </v-row>
                      <v-divider class="my-4"></v-divider>
                      <v-col cols="12" class="d-flex align-center justify-space-between">
                        <span class="text-title-medium font-weight-medium">{{ $t('chap authentication') }}</span>
                      </v-col>
                      <v-select
                        v-model="target.authentication.method"
                        :label="$t('authentication method')"
                        :items="authMethods"
                        item-title="label"
                        item-value="value"
                        density="compact"
                        hide-details="auto"
                        class="mt-3"
                      ></v-select>
                      <v-expand-transition>
                        <div v-if="target.authentication.method === 'chap'">
                          <v-text-field v-model="target.authentication.username" :label="$t('chap username')" class="mt-3" density="compact" hide-details="auto"></v-text-field>
                          <v-text-field v-model="target.authentication.password" :label="$t('chap password')" type="password" class="mt-3" density="compact" hide-details="auto"></v-text-field>
                        </div>
                      </v-expand-transition>
                      <v-divider class="my-4"></v-divider>
                      <v-col cols="12" class="d-flex align-center justify-space-between">
                        <span class="text-title-medium font-weight-medium">{{ $t('luns') }}</span>
                        <v-btn size="small" variant="text" color="green" prepend-icon="mdi-plus" @click="addLun(target)">
                          {{ $t('add lun') }}
                        </v-btn>
                      </v-col>
                      <v-table v-if="target.luns.length > 0" density="compact" class="rounded border">
                        <thead>
                          <tr>
                            <th>{{ $t('lun id') }}</th>
                            <th>{{ $t('path') }}</th>
                            <th>{{ $t('backing store') }}</th>
                            <th>{{ $t('mode') }}</th>
                            <th>{{ $t('size') }}</th>
                            <th style="width: 48px"></th>
                          </tr>
                        </thead>
                        <tbody>
                          <tr v-for="(lun, li) in target.luns" :key="li">
                            <td>
                              <v-text-field v-model.number="lun.id" density="compact" variant="plain" type="number" hide-details style="min-width: 60px; max-width: 80px"></v-text-field>
                            </td>
                            <td>
                              <v-text-field v-model="lun.path" density="compact" variant="plain" placeholder="/dev/sda" hide-details></v-text-field>
                            </td>
                            <td>
                              <v-select v-model="lun.backing_store" density="compact" variant="plain" :items="backingStores" hide-details style="min-width: 90px"></v-select>
                            </td>
                            <td>
                              <v-text-field v-model="lun.mode" density="compact" variant="plain" hide-details style="min-width: 90px"></v-text-field>
                            </td>
                            <td>
                              <v-text-field v-model="lun.size" density="compact" variant="plain" placeholder="1G" hide-details style="min-width: 60px"></v-text-field>
                            </td>
                            <td>
                              <v-btn icon size="x-small" variant="text" color="red" @click="removeLun(target, li)">
                                <v-icon>mdi-delete</v-icon>
                              </v-btn>
                            </td>
                          </tr>
                        </tbody>
                      </v-table>
                      <v-alert v-else type="info" variant="tonal" density="comfortable" class="mt-2">
                        {{ $t('no luns configured') }}
                      </v-alert>
                      <v-divider class="my-4"></v-divider>
                      <v-col cols="12" class="d-flex align-center justify-space-between">
                        <span class="text-title-medium font-weight-medium">{{ $t('allowed initiators') }}</span>
                        <v-btn size="small" variant="text" color="green" prepend-icon="mdi-plus" @click="addInitiator(target)">
                          {{ $t('add') }}
                        </v-btn>
                      </v-col>
                      <v-table v-if="target.initiators.length > 0" density="compact" class="rounded border">
                        <thead>
                          <tr>
                            <th>{{ $t('initiator iqn') }}</th>
                            <th>{{ $t('authentication method') }}</th>
                            <th style="width: 48px"></th>
                          </tr>
                        </thead>
                        <tbody>
                          <tr v-for="(initiator, ii) in target.initiators" :key="ii">
                            <td>
                              <v-text-field v-model="initiator.iqn" density="compact" variant="plain" placeholder="iqn.2024-01.com.client:host" hide-details></v-text-field>
                            </td>
                            <td>
                              <v-select
                                v-model="initiator.authentication.method"
                                density="compact"
                                variant="plain"
                                :items="authMethods"
                                item-title="label"
                                item-value="value"
                                hide-details
                                style="min-width: 100px"
                              ></v-select>
                            </td>
                            <td>
                              <v-btn icon size="x-small" variant="text" color="red" @click="removeInitiator(target, ii)">
                                <v-icon>mdi-delete</v-icon>
                              </v-btn>
                            </td>
                          </tr>
                        </tbody>
                      </v-table>
                      <v-alert v-else type="info" variant="tonal" density="comfortable" class="mt-2">
                        {{ $t('no initiators configured') }}
                      </v-alert>

                      <v-divider class="my-4"></v-divider>
                      <div class="d-flex justify-end">
                        <v-btn variant="text" color="red" prepend-icon="mdi-delete" @click="removeTarget(i)">
                          {{ $t('delete target') }}
                        </v-btn>
                      </div>
                    </v-expansion-panel-text>
                  </v-expansion-panel>
                </v-expansion-panels>
                <v-alert v-else type="info" variant="tonal" density="comfortable">
                  {{ $t('no targets configured') }}
                </v-alert>
              </v-window-item>

              <!-- Client -->
              <v-window-item value="client">
                <v-text-field v-model="initiatorConfig.name" :label="$t('initiator iqn')" placeholder="iqn.2024-01.com.example:client" hide-details="auto" class="mt-2"></v-text-field>
                <v-divider class="mt-6 mb-4"></v-divider>
                <v-col cols="12" class="d-flex align-center justify-space-between">
                  <span class="text-title-medium font-weight-medium">{{ $t('targets') }}</span>
                  <v-btn size="small" variant="text" color="primary" prepend-icon="mdi-plus" @click="addInitiatorTarget()">
                    {{ $t('add') }}
                  </v-btn>
                </v-col>

                <v-expansion-panels v-if="initiatorTargets.length > 0" variant="accordion">
                  <v-expansion-panel v-for="(target, i) in initiatorTargets" :key="i">
                    <v-expansion-panel-title>
                      <div class="d-flex align-center ga-3 w-100 pr-2">
                        <div class="flex-grow-1">
                          <div class="font-weight-medium text-caption">{{ target.name || $t('new target') }}</div>
                        </div>
                        <span class="text-caption text-medium-emphasis flex-shrink-0">{{ target.portal.address }}:{{ target.portal.port }}</span>
                      </div>
                    </v-expansion-panel-title>
                    <v-expansion-panel-text>
                      <v-text-field v-model="target.name" :label="$t('target iqn')" placeholder="iqn.2024-01.com.example:storage" density="compact" hide-details="auto" class="mb-3"></v-text-field>
                      <v-row>
                        <v-col cols="12" sm="8">
                          <v-text-field v-model="target.portal.address" :label="$t('address')" placeholder="10.0.0.1" density="compact" hide-details="auto"></v-text-field>
                        </v-col>
                        <v-col cols="12" sm="4">
                          <v-text-field v-model="target.portal.port" :label="$t('portal port')" placeholder="3260" density="compact" hide-details="auto"></v-text-field>
                        </v-col>
                      </v-row>
                      <v-switch v-model="target.connection.automount" :label="$t('automount')" color="green" density="compact" inset hide-details="auto" class="mt-3"></v-switch>
                      <v-divider class="my-4"></v-divider>
                      <div class="d-flex justify-end">
                        <v-btn variant="text" color="red" prepend-icon="mdi-delete" @click="removeInitiatorTarget(i)">
                          {{ $t('delete target') }}
                        </v-btn>
                      </div>
                    </v-expansion-panel-text>
                  </v-expansion-panel>
                </v-expansion-panels>
                <v-alert v-else type="info" variant="tonal" density="comfortable">
                  {{ $t('no targets configured') }}
                </v-alert>
              </v-window-item>
            </v-window>
          </v-card-text>
        </v-card>
      </v-container>
    </v-container>
  </v-container>

  <v-fab @click="saveConfig()" color="primary" style="position: fixed; bottom: 32px; right: 32px; z-index: 1000" size="large" icon>
    <v-icon>mdi-content-save</v-icon>
  </v-fab>

  <v-overlay :model-value="overlay" class="align-center justify-center">
    <v-progress-circular color="onPrimary" size="64" indeterminate></v-progress-circular>
  </v-overlay>
</template>

<script setup>
import { onMounted, ref } from 'vue';
import { useI18n } from 'vue-i18n';
import { showSnackbarError, showSnackbarSuccess } from '@/composables/snackbar';

const { t } = useI18n();
const overlay = ref(false);
const activeTab = ref('server');

const targets = ref([]);

const authMethods = [
  { label: 'None', value: 'none' },
  { label: 'CHAP', value: 'chap' },
];

const backingStores = ['rdwr', 'rdonly'];

const initiatorConfig = ref({
  name: '',
});

const initiatorTargets = ref([]);

const authHeaders = () => ({
  Authorization: 'Bearer ' + localStorage.getItem('authToken'),
  'Content-Type': 'application/json',
});

function emptyTarget() {
  return {
    id: null,
    name: '',
    iqn: '',
    portal: '0.0.0.0:3260',
    authentication: { method: 'none' },
    luns: [],
    initiators: [],
    isActive: false,
  };
}

function addTarget() {
  targets.value.push(emptyTarget());
}

function removeTarget(index) {
  targets.value.splice(index, 1);
}

function addLun(target) {
  target.luns.push({
    id: target.luns.length + 1,
    path: '',
    backing_store: 'rdwr',
    mode: 'logicalunit',
    size: '',
  });
}

function removeLun(target, index) {
  target.luns.splice(index, 1);
}

function addInitiator(target) {
  target.initiators.push({ iqn: '', authentication: { method: 'none' } });
}

function removeInitiator(target, index) {
  target.initiators.splice(index, 1);
}

function addInitiatorTarget() {
  initiatorTargets.value.push({
    id: null,
    name: '',
    portal: { address: '', port: '3260' },
    connection: { automount: false },
  });
}

function removeInitiatorTarget(index) {
  initiatorTargets.value.splice(index, 1);
}

async function loadConfig() {
  overlay.value = true;
  try {
    const [targetsRes, clientRes] = await Promise.all([
      fetch('/api/v1/iscsi/targets', {
        method: 'GET',
        headers: authHeaders(),
      }),
      fetch('/api/v1/iscsi/initiator', {
        method: 'GET',
        headers: authHeaders(),
      }),
    ]);

    if (!targetsRes.ok) {
      const error = await targetsRes.json();
      throw new Error(error.error || t('unknown error'));
    }
    if (!clientRes.ok) {
      const error = await clientRes.json();
      throw new Error(error.error || t('unknown error'));
    }

    const targetsData = await targetsRes.json();
    const initiatorData = await clientRes.json();

    targets.value = (targetsData ?? []).map((t) => ({
      id: t.id ?? null,
      name: t.name ?? '',
      iqn: t.iqn ?? '',
      portal: t.portal ?? '0.0.0.0:3260',
      authentication: {
        method: t.authentication?.method ?? 'none',
        username: t.authentication?.username ?? '',
        password: t.authentication?.password ?? '',
      },
      luns: (t.luns ?? []).map((l) => ({
        id: l.id,
        path: l.path ?? '',
        backing_store: l.backing_store ?? 'rdwr',
        mode: l.mode ?? 'logicalunit',
        size: l.size ?? '',
      })),
      initiators: (t.initiators ?? []).map((i) => ({
        iqn: i.iqn ?? '',
        authentication: { method: i.authentication?.method ?? 'none' },
      })),
      isActive: t.isActive ?? false,
    }));

    initiatorConfig.value = {
      name: initiatorData.initiator?.name ?? '',
    };

    initiatorTargets.value = (initiatorData.targets ?? []).map((t) => ({
      id: t.id ?? null,
      name: t.name ?? '',
      portal: {
        address: t.portal?.address ?? '',
        port: t.portal?.port ?? '3260',
      },
      connection: {
        automount: t.connection?.automount ?? false,
      },
    }));
  } catch (e) {
    showSnackbarError(t('iscsi config could not be loaded'), e.message);
  } finally {
    overlay.value = false;
  }
}

async function saveConfig() {
  overlay.value = true;
  try {
    await Promise.all([

      ...targets.value
        .filter((target) => target.id === null)
        .map(async (target) => {
          target.id = Date.now();
          const auth =
            target.authentication.method === 'chap'
              ? { method: 'chap', username: target.authentication.username, password: target.authentication.password }
              : { method: 'none' };
          const res = await fetch('/api/v1/iscsi/targets', {
            method: 'POST',
            headers: authHeaders(),
            body: JSON.stringify({
              id: target.id,
              name: target.name,
              iqn: target.iqn,
              portal: target.portal,
              authentication: auth,
              luns: target.luns.map((l) => ({
                id: l.id,
                path: l.path,
                backing_store: l.backing_store,
                mode: l.mode,
              })),
              initiators: target.initiators.map((i) => ({
                iqn: i.iqn,
                authentication: { method: i.authentication.method },
              })),
            }),
          });
          if (!res.ok) {
            const error = await res.json();
            target.id = null;
            throw new Error(error.error || t('unknown error'));
          }
          const data = await res.json();
          if (data.id != null) {
            target.id = data.id;
          }
        }),
      // Client-side initiator targets
      ...initiatorTargets.value
        .filter((target) => target.id === null)
        .map(async (target) => {
          const res = await fetch('/api/v1/iscsi/initiator/targets', {
            method: 'POST',
            headers: authHeaders(),
            body: JSON.stringify({
              name: target.name,
              portal: {
                address: target.portal.address,
                port: target.portal.port,
              },
              connection: {
                automount: target.connection.automount,
              },
            }),
          });
          if (!res.ok) {
            const error = await res.json();
            throw new Error(error.error || t('unknown error'));
          }
          const data = await res.json();
          if (data.id != null) {
            target.id = data.id;
          }
        }),
    ]);
    showSnackbarSuccess(t('iscsi config saved successfully'));
  } catch (e) {
    showSnackbarError(t('iscsi config could not be saved'), e.message);
  } finally {
    overlay.value = false;
  }
}

onMounted(loadConfig);
</script>
