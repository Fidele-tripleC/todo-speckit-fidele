<script setup>
import { onMounted, ref } from "vue";
import listServices from "../services/listServices.js";
import todoServices from "../services/todoServices.js";
import { formatDueDate, isTodoOverdue, optionalDueDateRules, toDateInputValue } from "../config/validation.js";

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

const itemsDialogOpen = ref(false);
const addItemDialogOpen = ref(false);
const editItemDialogOpen = ref(false);
const deleteItemDialogOpen = ref(false);
const activeList = ref(null);
const todos = ref([]);
const todosLoading = ref(false);
const todosError = ref("");
const addItemForm = ref(null);
const editItemForm = ref(null);
const newTodoTitle = ref("");
const newTodoDueDate = ref("");
const editTodoTitle = ref("");
const editTodoDueDate = ref("");
const todoToEdit = ref(null);
const todoToDelete = ref(null);
const addItemLoading = ref(false);
const editItemLoading = ref(false);
const deleteItemLoading = ref(false);
const itemDialogError = ref("");

const todoTitleRules = [
  (value) => !!value?.trim() || "Todo title is required.",
  (value) => !value || value.trim().length <= 255 || "Todo title must be 255 characters or fewer.",
];

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

const sortTodos = (items) =>
  [...items].sort((a, b) => {
    if (a.completed !== b.completed) {
      return a.completed ? 1 : -1;
    }

    return new Date(a.createdAt) - new Date(b.createdAt);
  });

const openItemsDialog = async (list) => {
  activeList.value = list;
  todos.value = [];
  todosError.value = "";
  itemsDialogOpen.value = true;
  todosLoading.value = true;

  try {
    const response = await todoServices.getTodos(list.id);
    todos.value = response.data;
  } catch (error) {
    todosError.value = error.response?.data?.message || "Failed to load todos.";
  } finally {
    todosLoading.value = false;
  }
};

const openAddItemDialog = () => {
  newTodoTitle.value = "";
  newTodoDueDate.value = "";
  itemDialogError.value = "";
  addItemDialogOpen.value = true;
};

const handleAddItem = async () => {
  itemDialogError.value = "";
  const { valid } = await addItemForm.value.validate();

  if (!valid || !activeList.value) {
    return;
  }

  addItemLoading.value = true;

  try {
    const response = newTodoDueDate.value
      ? await todoServices.createTodo(activeList.value.id, newTodoTitle.value.trim(), newTodoDueDate.value)
      : await todoServices.createTodo(activeList.value.id, newTodoTitle.value.trim());
    todos.value = sortTodos([...todos.value, response.data]);
    addItemDialogOpen.value = false;
    newTodoTitle.value = "";
    newTodoDueDate.value = "";
  } catch (error) {
    itemDialogError.value = error.response?.data?.message || "Failed to add todo.";
  } finally {
    addItemLoading.value = false;
  }
};

const toggleCompleted = async (todo, completed) => {
  todosError.value = "";

  try {
    const response = await todoServices.updateTodo(todo.id, { completed });
    todos.value = sortTodos(
      todos.value.map((item) => (item.id === response.data.id ? response.data : item))
    );
  } catch (error) {
    todosError.value = error.response?.data?.message || "Failed to update todo.";
  }
};

const openEditItemDialog = (todo) => {
  todoToEdit.value = todo;
  editTodoTitle.value = todo.title;
  editTodoDueDate.value = toDateInputValue(todo.dueDate);
  itemDialogError.value = "";
  editItemDialogOpen.value = true;
};

const handleEditItem = async () => {
  itemDialogError.value = "";
  const { valid } = await editItemForm.value.validate();

  if (!valid || !todoToEdit.value) {
    return;
  }

  editItemLoading.value = true;

  try {
    const response = await todoServices.updateTodo(todoToEdit.value.id, {
      title: editTodoTitle.value.trim(),
      dueDate: editTodoDueDate.value || null,
    });
    todos.value = sortTodos(
      todos.value.map((item) => (item.id === response.data.id ? response.data : item))
    );
    editItemDialogOpen.value = false;
  } catch (error) {
    itemDialogError.value = error.response?.data?.message || "Failed to update todo.";
  } finally {
    editItemLoading.value = false;
  }
};

const openDeleteItemDialog = (todo) => {
  todoToDelete.value = todo;
  itemDialogError.value = "";
  deleteItemDialogOpen.value = true;
};

const handleDeleteItem = async () => {
  if (!todoToDelete.value) {
    return;
  }

  deleteItemLoading.value = true;
  itemDialogError.value = "";

  try {
    await todoServices.deleteTodo(todoToDelete.value.id);
    todos.value = todos.value.filter((item) => item.id !== todoToDelete.value.id);
    deleteItemDialogOpen.value = false;
  } catch (error) {
    itemDialogError.value = error.response?.data?.message || "Failed to delete todo.";
  } finally {
    deleteItemLoading.value = false;
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
                icon="mdi-format-list-checks"
                size="small"
                variant="text"
                :aria-label="`View items for ${list.name}`"
                @click="openItemsDialog(list)"
              />
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

    <v-dialog v-model="itemsDialogOpen" max-width="640">
      <v-card v-if="itemsDialogOpen">
        <v-card-item>
          <v-card-title>{{ activeList?.name }} — Items</v-card-title>
          <template #append>
            <v-btn color="primary" variant="elevated" class="oc-cta" @click="openAddItemDialog">
              + Add Item
            </v-btn>
          </template>
        </v-card-item>
        <v-card-text>
          <v-progress-linear v-if="todosLoading" indeterminate color="primary" class="mb-4" />
          <v-alert v-if="todosError" type="error" class="mb-4">
            {{ todosError }}
          </v-alert>
          <p v-if="!todosLoading && todos.length === 0">No todos in this list yet.</p>
          <v-list v-else-if="!todosLoading">
            <v-list-item v-for="todo in todos" :key="todo.id">
              <template #prepend>
                <v-checkbox
                  :model-value="todo.completed"
                  hide-details
                  density="compact"
                  @update:model-value="(value) => toggleCompleted(todo, value)"
                />
              </template>
              <v-list-item-title :class="{ 'text-decoration-line-through text-medium-emphasis': todo.completed }">
                {{ todo.title }}
              </v-list-item-title>
              <v-list-item-subtitle v-if="todo.dueDate">
                <span :class="{ 'text-error': isTodoOverdue(todo) }">
                  Due {{ formatDueDate(todo.dueDate) }}
                </span>
              </v-list-item-subtitle>
              <template #append>
                <v-btn
                  icon="mdi-pencil"
                  size="small"
                  variant="text"
                  aria-label="Edit todo"
                  @click="openEditItemDialog(todo)"
                />
                <v-btn
                  icon="mdi-delete"
                  size="small"
                  variant="text"
                  aria-label="Delete todo"
                  @click="openDeleteItemDialog(todo)"
                />
              </template>
            </v-list-item>
          </v-list>
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn variant="text" @click="itemsDialogOpen = false">Close</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="addItemDialogOpen" max-width="480">
      <v-card v-if="addItemDialogOpen">
        <v-card-title>Add item</v-card-title>
        <v-card-text>
          <v-form ref="addItemForm" @submit.prevent="handleAddItem">
            <v-text-field v-model="newTodoTitle" label="Todo title" :rules="todoTitleRules" />
            <v-text-field
              v-model="newTodoDueDate"
              label="Due date"
              type="date"
              :rules="optionalDueDateRules"
            />
            <v-alert v-if="itemDialogError && addItemDialogOpen" type="error" class="mt-2">
              {{ itemDialogError }}
            </v-alert>
          </v-form>
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn variant="text" @click="addItemDialogOpen = false">Cancel</v-btn>
          <v-btn
            color="primary"
            variant="elevated"
            class="oc-cta"
            :loading="addItemLoading"
            @click="handleAddItem"
          >
            Add
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="editItemDialogOpen" max-width="480">
      <v-card v-if="editItemDialogOpen">
        <v-card-title>Edit item</v-card-title>
        <v-card-text>
          <v-form ref="editItemForm" @submit.prevent="handleEditItem">
            <v-text-field v-model="editTodoTitle" label="Todo title" :rules="todoTitleRules" />
            <v-text-field
              v-model="editTodoDueDate"
              label="Due date"
              type="date"
              :rules="optionalDueDateRules"
              clearable
            />
            <v-alert v-if="itemDialogError && editItemDialogOpen" type="error" class="mt-2">
              {{ itemDialogError }}
            </v-alert>
          </v-form>
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn variant="text" @click="editItemDialogOpen = false">Cancel</v-btn>
          <v-btn
            color="primary"
            variant="elevated"
            class="oc-cta"
            :loading="editItemLoading"
            @click="handleEditItem"
          >
            Save
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="deleteItemDialogOpen" max-width="480">
      <v-card v-if="deleteItemDialogOpen">
        <v-card-title>Delete item</v-card-title>
        <v-card-text>
          Delete {{ todoToDelete?.title }}? This cannot be undone.
          <v-alert v-if="itemDialogError && deleteItemDialogOpen" type="error" class="mt-2">
            {{ itemDialogError }}
          </v-alert>
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn variant="text" @click="deleteItemDialogOpen = false">Cancel</v-btn>
          <v-btn color="primary" variant="elevated" :loading="deleteItemLoading" @click="handleDeleteItem">
            Delete
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
