<template>
  <div
    class="flex flex-col h-screen bg-background transition-colors duration-200 overflow-hidden"
  >
    <div class="container mx-auto max-w-2xl p-6 pb-3">
      <!-- Header -->
      <div class="flex items-center justify-between mb-4">
        <div>
          <h1 class="text-4xl font-bold text-foreground">ToDo App</h1>
          <p class="text-lg mt-2 text-muted-foreground">
            Organize your daily tasks
          </p>
        </div>

        <!-- Theme Toggle -->
        <Button
          @click="toggleTheme"
          variant="outline"
          size="icon"
          class="p-3 cursor-pointer"
        >
          <Sun v-if="isDark" class="h-5 w-5" />
          <Moon v-else class="h-5 w-5" />
        </Button>
      </div>

      <Card class="p-6 mb-6">
        <div class="flex gap-3">
          <Input
            v-model="newTodo"
            @keyup.enter="addTodo"
            placeholder="What do you need to do?"
            class="flex-1"
          />
          <Button @click="addTodo" class="flex items-center gap-2">
            <Plus class="h-4 w-4" />
            Add
          </Button>
        </div>
      </Card>
    </div>
    <div class="container mx-auto max-w-2xl px-6">
      <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
        <div class="flex items-center justify-center sm:justify-start gap-6">
          <span class="text-sm text-muted-foreground">
            <Badge
              variant="secondary"
              class="bg-blue-100 text-blue-800 dark:bg-blue-900 dark:text-blue-200"
            >
              {{ activeTodosCount }}
            </Badge>
            earrings
          </span>
          <span class="text-sm text-muted-foreground">
            <Badge
              variant="secondary"
              class="bg-green-100 text-green-800 dark:bg-green-900 dark:text-green-200"
            >
              {{ completedTodosCount }}
            </Badge>
            completed
          </span>
        </div>
        <div class="flex items-center sm:justify-end gap-1">
          <Button
            v-for="filterType in filters"
            :key="filterType.value"
            @click="filter = filterType.value"
            :variant="filter === filterType.value ? 'default' : 'ghost'"
            size="sm"
          >
            {{ filterType.label }}
          </Button>
        </div>
      </div>
    </div>
    <ScrollArea class="container mx-auto max-w-2xl p-6 flex-1 overflow-auto">
      <!-- Todo List -->
      <div class="space-y-3">
        <!-- Empty State -->
        <Card v-if="filteredTodos.length === 0" class="p-12 text-center">
          <div class="text-6xl mb-4 opacity-20">📝</div>
          <p class="text-lg text-muted-foreground">
            {{ getEmptyMessage() }}
          </p>
          <p
            v-if="filter === 'all'"
            class="text-sm mt-2 text-muted-foreground/70"
          >
            Add your first task above!
          </p>
        </Card>

        <TodoItem
          v-for="todo in filteredTodos"
          :key="todo.id"
          :todo="todo"
          @delete-todo="deleteTodo"
        />
      </div>
    </ScrollArea>
  </div>
</template>

<script setup>
import { ref, computed, watch } from "vue";
import { Badge } from "@/components/ui/badge";
import { Button } from "@/components/ui/button";
import { Card } from "@/components/ui/card";
import { Input } from "@/components/ui/input";
import { ScrollArea } from "@/components/ui/scroll-area";
import TodoItem from "./TodoItem.vue";
import { Moon, Sun, Plus } from "lucide-vue-next";

// Theme management
const isDark = ref(false);
// Todo storage
const todos = ref([]);
// Filter options
const filter = ref("all");
// New Todo input
const newTodo = ref("");

// Filter options
const filters = [
  { value: "all", label: "All" },
  { value: "active", label: "Earrings" },
  { value: "completed", label: "Completed" },
];

const activeTodosCount = computed(
  () => todos.value.filter((todo) => !todo.completed).length
);

const completedTodosCount = computed(
  () => todos.value.filter((todo) => todo.completed).length
);

// Computed properties
const filteredTodos = computed(() => {
  switch (filter.value) {
    case "active":
      return todos.value.filter((todo) => !todo.completed);
    case "completed":
      return todos.value.filter((todo) => todo.completed);
    default:
      return todos.value;
  }
});

// Function to toggle theme
const toggleTheme = () => {
  isDark.value = !isDark.value;
  document.documentElement.classList.toggle("dark", isDark.value);
  localStorage.setItem("theme", isDark.value ? "dark" : "light");
};

// Initialize theme from localStorage
const initTheme = () => {
  const savedTheme = localStorage.getItem("theme");
  const prefersDark = window.matchMedia("(prefers-color-scheme: dark)").matches;

  isDark.value = savedTheme === "dark" || (!savedTheme && prefersDark);
  document.documentElement.classList.toggle("dark", isDark.value);
};

// Save todos to localStorage
const saveTodos = () => {
  localStorage.setItem("todos", JSON.stringify(todos.value));
};

const loadTodos = () => {
  const saved = localStorage.getItem("todos");
  if (saved) {
    todos.value = JSON.parse(saved);
  }
};

// Delete Todo by ID
const deleteTodo = (id) => {
  todos.value = todos.value.filter((todo) => todo.id !== id);
};

// Add new Todo
const addTodo = () => {
  if (newTodo.value.trim()) {
    todos.value.push({
      id: Date.now(),
      text: newTodo.value.trim(),
      completed: false,
      createdAt: new Date(),
    });
    newTodo.value = "";
  }
};

const getEmptyMessage = () => {
  if (filter.value === "active") {
    return "You have no earrings!";
  } else if (filter.value === "completed") {
    return "You have no completed tasks!";
  } else {
    return "You have no tasks!";
  }
};

watch(todos, saveTodos, { deep: true });

initTheme();
loadTodos();
</script>
