<script setup lang="ts">
import { onMounted, ref, watchEffect, type Ref, watch } from 'vue'
import type { Todo } from './types/Todo'

const STORAGE_KEYS = 'local_todos'

const todos: Ref<Todo[]> = ref([])

// const todos: Ref<Todo[]> = ref([
//   { id: 1, text: 'Learn Vue Js', completed: false },
//   { id: 2, text: 'Build a to-do application', completed: false },
//   { id: 3, text: 'Push it onto github', completed: false },
// ])

const newTodoText: Ref<string> = ref('')

function addTodo(): void {
  if (newTodoText.value.trim()) {
    todos.value.push({
      id: Date.now(),
      text: newTodoText.value.trim(),
      completed: false,
    })

    newTodoText.value = ''
  }
}

function toggleTodo(todo: Todo): void {
  todo.completed = !todo.completed
}

function removeTodo(todoId: number) {
  todos.value = todos.value.filter((todo) => todo.id != todoId)
}
const loaded = ref(false)

onMounted(() => {
  const storedTodos = localStorage.getItem(STORAGE_KEYS)
  if (storedTodos) {
    try {
      todos.value = JSON.parse(storedTodos)
    } catch (e) {
      console.error(`Failed to parse todos: ${e}`)
    }
  }
  loaded.value = true
})

watch(
  todos,
  (newTodos) => {
    if (loaded.value) {
      localStorage.setItem(STORAGE_KEYS, JSON.stringify(newTodos))
    }
  },
  { deep: true },
)

watchEffect(() => {
  console.log('add todos to local storage')
  const todoJson = JSON.stringify(todos.value)
  localStorage.setItem(STORAGE_KEYS, todoJson)
})
</script>

<template>
  <main>
    <h1>To-Do list using Vue 3</h1>

    <div class="input-section">
      <input type="text" v-model="newTodoText" placeholder="Add a todo" @keypress.enter="addTodo" />
      <button @click="addTodo">Add todo</button>
    </div>

    <ul>
      <li
        v-for="todo in todos"
        :key="todo.id"
        @click="toggleTodo(todo)"
        :class="{ completed: todo.completed }"
      >
        {{ todo.text }}
        <button @click.stop="removeTodo(todo.id)" class="remove-button">X</button>
      </li>
    </ul>
  </main>
</template>

<style scoped>
main {
  max-width: 500px;
  margin: 40px auto;
  padding: 20px;
  border: 1px solid black;
  border-radius: 8px;
  font-family: sans-serif;
}

h1 {
  text-align: center;
  color: #333;
}

input[type='text'] {
  padding: 8px;
  margin-right: 5px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

button {
  padding: 8px 12px;
  background-color: #4caf50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:hover {
  background-color: #45a049;
}

.input-section {
  margin-bottom: 20px;
}

.completed {
  text-decoration: line-through;
  color: #888;
}

li {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 8px;
  padding: 8px;
  border-bottom: 1px solid #eee;
}

li span {
  flex-grow: 1;
  margin-right: 10px;
  cursor: pointer;
}

.remove-button {
  background-color: #f44336;
  padding: 4px 8px;
  font-size: 0.8em;
}

.remove-button:hover {
  background-color: #d32f2f;
}
</style>
