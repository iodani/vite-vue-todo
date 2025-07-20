<template>
  <Card
    :class="[
      'p-4 transition-all duration-200 select-none hover:shadow-md',
      todo.completed ? 'opacity-75 bg-muted/20' : 'hover:bg-accent/5',
    ]"
  >
    <div class="flex items-center gap-3">
      <Checkbox
        :id="`todo-${todo.id}`"
        v-model="todo.completed"
        class="cursor-pointer shrink-0"
        :aria-label="
          todo.completed ? 'Marcar como pendiente' : 'Marcar como completada'
        "
      />
      <div class="flex-1">
        <Input
          v-if="editing"
          v-model="todo.text"
          @keyup.esc="editing = false"
          @blur="editing = false"
          class="w-full"
          ref="inputRef"
        />
        <span
          v-else
          :class="[
            'text-foreground',
            todo.completed ? 'line-through opacity-60' : '',
          ]"
        >
          {{ todo.text }}
        </span>
      </div>
      <div class="flex gap-1">
        <template v-if="editing">
          <Button
            @click="editing = false"
            variant="ghost"
            size="icon"
            class="h-8 w-8 text-green-600 hover:text-green-700 cursor-pointer"
          >
            <Check class="h-4 w-4" />
          </Button>
          <Button
            @click="editing = false"
            variant="ghost"
            size="icon"
            class="h-8 w-8 text-gray-500 hover:text-gray-600 cursor-pointer"
          >
            <X class="h-4 w-4" />
          </Button>
        </template>
        <template v-else>
          <Button
            @click="editing = true"
            variant="ghost"
            size="icon"
            class="h-8 w-8 text-muted-foreground hover:text-foreground cursor-pointer"
          >
            <Edit3 class="h-4 w-4" />
          </Button>
          <Button
            @click="handlerTrash"
            variant="ghost"
            size="icon"
            class="h-8 w-8 text-muted-foreground hover:text-destructive cursor-pointer"
          >
            <Trash2 class="h-4 w-4" />
          </Button>
        </template>
      </div>
    </div>
  </Card>
</template>

<script setup>
import { defineProps, defineEmits, ref, watch, nextTick } from "vue";
import { Button } from "@/components/ui/button";
import { Card } from "@/components/ui/card";
import { Checkbox } from "@/components/ui/checkbox";
import Input from "@/components/ui/input/Input.vue";
import { Check, X, Edit3, Trash2 } from "lucide-vue-next";

const props = defineProps({
  todo: {
    type: Object,
    required: true,
  },
});

const emit = defineEmits(["delete-todo"]);

const editing = ref(false);
const inputRef = ref(null);

watch(editing, (newValue) => {
  if (newValue) {
    nextTick(() => {
      if (inputRef.value) {
        inputRef.value.$el.focus();
      }
    });
  }
});

const handlerTrash = () => {
  if (props.todo?.id) {
    editing.value = false;
    emit("delete-todo", props.todo.id);
  }
};
</script>
