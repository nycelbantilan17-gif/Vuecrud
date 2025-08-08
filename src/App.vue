<script setup>
import { ref, onMounted, computed, watch } from 'vue'

const todos = ref([])
const name = ref('')
const input_content = ref('')

const editingTodo = ref(null)
const editedContent = ref('')

const todos_asc = computed(() => {
  return [...todos.value].sort((a, b) => b.createdAt - a.createdAt)
})

const activeTodos = computed(() => todos_asc.value.filter(t => !t.done))
const doneTodos = computed(() => todos_asc.value.filter(t => t.done))

const addTodo = () => {
  if (input_content.value.trim() === '') return
  todos.value.push({
    content: input_content.value,
    done: false,
    createdAt: new Date().getTime(),
  })
  input_content.value = ''
  saveToLocalStorage()
}

const removeTodo = (todo) => {
  todos.value = todos.value.filter(t => t !== todo)
  saveToLocalStorage()
}

const toggleDone = (todo) => {
  todo.done = !todo.done
  saveToLocalStorage()
}

const startEditing = (todo) => {
  editingTodo.value = todo
  editedContent.value = todo.content
}

const cancelEdit = () => {
  editingTodo.value = null
  editedContent.value = ''
}

const saveEdit = () => {
  if (!editedContent.value.trim()) return
  editingTodo.value.content = editedContent.value.trim()
  editingTodo.value = null
  editedContent.value = ''
  saveToLocalStorage()
}

const saveToLocalStorage = () => {
  localStorage.setItem('todos', JSON.stringify(todos.value))
}

watch(name, (newVal) => {
  localStorage.setItem('name', newVal)
})

watch(todos, () => {
  saveToLocalStorage()
}, { deep: true })

onMounted(() => {
  name.value = localStorage.getItem('name') || ''
  const saved = localStorage.getItem('todos')
  if (saved) todos.value = JSON.parse(saved)
})
</script>

<template>
  <div class="container">
    <main class="app">
      <!-- Greeting -->
      <section class="greeting">
        <h2 class="title">
          What's up,
          <input type="text" placeholder="Name here" v-model="name" />
        </h2>
      </section>

      <!-- Create Todo -->
      <section class="create-todo">
        <form @submit.prevent="addTodo">
          <h3>What's on your todo list?</h3>
          <input 
            type="text" 
            placeholder="Enter your task"
            v-model="input_content" />
          <input type="submit" value="Add todo" />
        </form>
      </section>

      <!-- Active Todos -->
      <section class="todo-list">
        <h3>Todo List</h3>
        <div v-if="activeTodos.length === 0">No active todos.</div>

        <div 
          v-for="todo in activeTodos" 
          :key="todo.createdAt" 
          class="todo-item"
        >
          <button class="check-btn" @click="toggleDone(todo)" :aria-pressed="todo.done.toString()">
            <span v-if="todo.done">✔</span>
            <span v-else>⬜</span>
          </button>

          <div v-if="editingTodo === todo" class="todo-content editing">
            <input type="text" v-model="editedContent" />
            <button @click="saveEdit">Save</button>
            <button @click="cancelEdit">Cancel</button>
          </div>

          <div v-else class="todo-content">
            <p>{{ todo.content }}</p>
          </div>

          <button @click="startEditing(todo)">Edit</button>
          <button @click="removeTodo(todo)">Delete</button>
        </div>
      </section>

      <!-- Done Todos -->
      <section class="todo-list">
        <h3>Task Done</h3>
        <div v-if="doneTodos.length === 0">No completed tasks.</div>

        <div 
          v-for="todo in doneTodos" 
          :key="todo.createdAt" 
          class="todo-item"
        >
          <button class="check-btn" @click="toggleDone(todo)" :aria-pressed="todo.done.toString()">
            <span v-if="todo.done">✔</span>
            <span v-else>⬜</span>
          </button>
          <div class="todo-content done">
            <p>{{ todo.content }}</p>
          </div>
          <button @click="removeTodo(todo)">Delete</button>
        </div>
      </section>
    </main>
  </div>
</template>

<style scoped>
/* Light green background */
.container {
  background-color: #d4edda;
  min-height: 100vh;
  padding: 2rem;
  box-sizing: border-box;
}

.app {
  max-width: 600px;
  margin: 0 auto;
  padding: 1rem;
  font-family: sans-serif;
  background-color: #fff;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.greeting .title input {
  margin-left: 0.5rem;
}

.create-todo,
.todo-list {
  margin-top: 2rem;
}

.todo-item {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-bottom: 1rem;
}

.check-btn {
  background: none;
  border: none;
  font-size: 1.5rem;
  cursor: pointer;
  color: green;
  padding: 0;
  width: 2rem;
  height: 2rem;
  display: flex;
  align-items: center;
  justify-content: center;
}

.check-btn span {
  display: inline-block;
  line-height: 1;
  user-select: none;
}

.todo-content p {
  margin: 0;
}

.todo-content.done p {
  text-decoration: line-through;
  color: gray;
}

.todo-content.editing input {
  padding: 0.3rem;
  font-size: 1rem;
  margin-right: 0.5rem;
}
</style>