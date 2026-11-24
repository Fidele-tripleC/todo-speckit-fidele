<script setup>
import { onMounted, ref } from "vue";
import listServices from "../services/listServices.js";

const lists = ref([]);
const loading = ref(false);
const errorMessage = ref("");

const createDialogOpen = ref(false);
const renameDialogOpen = ref(false);
const deleteDialogOpen = ref(false);

const createForm = ref(null);
const renameForm = ref(null);

const newListName = ref("");
const renameListName = ref("");
const listToRename = ref(null);
const listToDelete = ref(null);

const createLoading = ref(false);
const renameLoading = ref(false);
const deleteLoading = ref(false);
const dialogError = ref("");

const listNameRules = [
  (value) => !!value?.trim() || "List name is required.",
  (value) => !value || value.trim().length <= 100 || "List name must be 100 characters or fewer.",
];

const sortLists = (items) => [...items].sort((a, b) => a.name.localeCompare(b.name));

const loadLists = async () => {
  loading.value = true;
  errorMessage.value = "";

  try {
    const response = await listServices.getLists();
    lists.value = response.data;
  } catch (error) {
    errorMessage.value = error.response?.data?.message || "Failed to load lists.";
  } finally {
    loading.value = false;
  }
};

const openCreateDialog = () => {
  newListName.value = "";
  dialogError.value = "";
  createDialogOpen.value = true;
};

const handleCreate = async () => {
  dialogError.value = "";
  const { valid } = await createForm.value.validate();

  if (!valid) {
    return;
  }

  createLoading.value = true;

  try {
    const response = await listServices.createList(newListName.value.trim());
    lists.value = sortLists([...lists.value, response.data]);
    createDialogOpen.value = false;
    newListName.value = "";
  } catch (error) {
    dialogError.value = error.response?.data?.message || "Failed to create list.";
  } finally {
    createLoading.value = false;
  }
};

const openRenameDialog = (list) => {
  listToRename.value = list;
  renameListName.value = list.name;
  dialogError.value = "";
  renameDialogOpen.value = true;
};

const handleRename = async () => {
  dialogError.value = "";
  const { valid } = await renameForm.value.validate();

  if (!valid || !listToRename.value) {
    return;
  }

  renameLoading.value = true;

  try {
    const response = await listServices.updateList(listToRename.value.id, renameListName.value.trim());
    lists.value = sortLists(
      lists.value.map((list) => (list.id === response.data.id ? response.data : list))
    );
    renameDialogOpen.value = false;
  } catch (error) {
    dialogError.value = error.response?.data?.message || "Failed to rename list.";
  } finally {
    renameLoading.value = false;
  }
};

const openDeleteDialog = (list) => {
  listToDelete.value = list;
  dialogError.value = "";
  deleteDialogOpen.value = true;
};

const handleDelete = async () => {
  if (!listToDelete.value) {
    return;
  }

  deleteLoading.value = true;
  dialogError.value = "";

  try {
    await listServices.deleteList(listToDelete.value.id);
    lists.value = lists.value.filter((list) => list.id !== listToDelete.value.id);
    deleteDialogOpen.value = false;
  } catch (error) {
    dialogError.value = error.response?.data?.message || "Failed to delete list.";
  } finally {
    deleteLoading.value = false;
  }
};

onMounted(loadLists);
</script>

<template>
  <v-container class="py-8">
    <v-card elevation="2">
      <v-card-item>
        <v-card-title>My Lists</v-card-title>
        <template #append>
          <v-btn color="primary" variant="elevated" class="oc-cta" @click="openCreateDialog">
            + New List
          </v-btn>
        </template>
      </v-card-item>

      <v-card-text>
        <v-progress-linear v-if="loading" indeterminate color="primary" class="mb-4" />

        <v-alert v-if="errorMessage" type="error" class="mb-4">
          {{ errorMessage }}
        </v-alert>

        <p v-if="!loading && lists.length === 0">No lists yet. Create your first list.</p>

        <v-list v-else-if="!loading">
          <v-list-item v-for="list in lists" :key="list.id" :title="list.name">
            <template #append>
              <v-btn
                icon="mdi-pencil"
                size="small"
                variant="text"
                aria-label="Edit list"
                @click="openRenameDialog(list)"
              />
              <v-btn
                icon="mdi-delete"
                size="small"
                variant="text"
                aria-label="Delete list"
                @click="openDeleteDialog(list)"
              />
            </template>
          </v-list-item>
        </v-list>
      </v-card-text>
    </v-card>

    <v-dialog v-model="createDialogOpen" max-width="480">
      <v-card>
        <v-card-title>New list</v-card-title>
        <v-card-text>
          <v-form ref="createForm" @submit.prevent="handleCreate">
            <v-text-field v-model="newListName" label="List name" :rules="listNameRules" />
            <v-alert v-if="dialogError && createDialogOpen" type="error" class="mt-2">
              {{ dialogError }}
            </v-alert>
          </v-form>
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn variant="text" @click="createDialogOpen = false">Cancel</v-btn>
          <v-btn
            color="primary"
            variant="elevated"
            class="oc-cta"
            :loading="createLoading"
            @click="handleCreate"
          >
            Create
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="renameDialogOpen" max-width="480">
      <v-card>
        <v-card-title>Rename list</v-card-title>
        <v-card-text>
          <v-form ref="renameForm" @submit.prevent="handleRename">
            <v-text-field v-model="renameListName" label="List name" :rules="listNameRules" />
            <v-alert v-if="dialogError && renameDialogOpen" type="error" class="mt-2">
              {{ dialogError }}
            </v-alert>
          </v-form>
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn variant="text" @click="renameDialogOpen = false">Cancel</v-btn>
          <v-btn
            color="primary"
            variant="elevated"
            class="oc-cta"
            :loading="renameLoading"
            @click="handleRename"
          >
            Save
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="deleteDialogOpen" max-width="480">
      <v-card>
        <v-card-title>Delete list</v-card-title>
        <v-card-text>
          Delete {{ listToDelete?.name }}? This cannot be undone.
          <v-alert v-if="dialogError && deleteDialogOpen" type="error" class="mt-2">
            {{ dialogError }}
          </v-alert>
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn variant="text" @click="deleteDialogOpen = false">Cancel</v-btn>
          <v-btn color="primary" variant="elevated" :loading="deleteLoading" @click="handleDelete">
            Delete
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>
