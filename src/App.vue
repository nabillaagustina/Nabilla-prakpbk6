// File: src/App.vue
<template>
  <div class="container">
    <h1>📝 Todo List dengan Axios & JSON Server<br /><small>(CRUD Lengkap)</small></h1>

    <div class="form">
      <input v-model="newTodo" placeholder="Tambah todo baru..." @keyup.enter="addTodo" />
      <button @click="addTodo">➕ Tambah</button>
    </div>

    <ul class="todo-list">
      <li v-for="todo in todos" :key="todo.id">
        <input type="checkbox" v-model="todo.completed" @change="toggleComplete(todo)" />

        <template v-if="isEditing(todo.id)">
          <input v-model="editTodoText" />
          <button @click="saveEdit(todo)">💾 Simpan</button>
          <button class="cancel" @click="cancelEdit">❌ Batal</button>
        </template>

        <template v-else>
          <span :class="{ done: todo.completed }">{{ todo.text }}</span>
          <button class="edit" @click="startEdit(todo)">✏ Edit</button>
          <button class="delete" @click="deleteTodo(todo.id)">🗑 Hapus</button>
        </template>
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

const todos = ref([])
const newTodo = ref('')
const editId = ref(null)
const editTodoText = ref('')

const API = 'http://localhost:3000/todos'

const fetchTodos = async () => {
  try {
    const res = await axios.get(API)
    todos.value = res.data
  } catch (err) {
    console.error('Gagal mengambil data:', err)
  }
}

const addTodo = async () => {
  const text = newTodo.value.trim()
  if (!text) return

  try {
    const res = await axios.post(API, { text, completed: false })
    todos.value.push(res.data)
    newTodo.value = ''
  } catch (err) {
    console.error('Gagal menambah todo:', err)
  }
}

const deleteTodo = async (id) => {
  try {
    await axios.delete(`${API}/${id}`)
    todos.value = todos.value.filter(todo => todo.id !== id)
  } catch (err) {
    console.error('Gagal menghapus todo:', err)
  }
}

const toggleComplete = async (todo) => {
  try {
    await axios.put(`${API}/${todo.id}`, { ...todo, completed: todo.completed })
  } catch (err) {
    console.error('Gagal update status todo:', err)
  }
}

const startEdit = (todo) => {
  editId.value = todo.id
  editTodoText.value = todo.text
}

const isEditing = (id) => editId.value === id

const saveEdit = async (todo) => {
  const updatedText = editTodoText.value.trim()
  if (!updatedText) return

  try {
    await axios.put(`${API}/${todo.id}`, { ...todo, text: updatedText })
    todo.text = updatedText
    editId.value = null
    editTodoText.value = ''
  } catch (err) {
    console.error('Gagal menyimpan edit:', err)
  }
}

const cancelEdit = () => {
  editId.value = null
  editTodoText.value = ''
}

onMounted(fetchTodos)
</script>

<style scoped>
body {
  background: #1e1e1e;
  color: #f0f0f0;
  font-family: 'Segoe UI', sans-serif;
}

.container {
  max-width: 700px;
  margin: auto;
  padding: 2rem;
  border-radius: 8px;
  background: #2c2c2c;
  box-shadow: 0 0 15px rgba(0, 0, 0, 0.5);
}
h1 {
  text-align: center;
  font-size: 2.4rem;
  margin-bottom: 1rem;
}
h1 small {
  font-size: 1rem;
  color: #bbb;
}
.form {
  display: flex;
  margin-bottom: 1.5rem;
  gap: 0.5rem;
}
input[type="text"] {
  flex: 1;
  padding: 0.6rem;
  font-size: 1rem;
  background: #333;
  border: 1px solid #555;
  border-radius: 4px;
  color: #fff;
}
button {
  padding: 0.6rem 1rem;
  border: none;
  background: #3498db;
  color: white;
  cursor: pointer;
  border-radius: 5px;
  font-size: 0.9rem;
  transition: 0.2s;
}
button:hover {
  background: #2980b9;
}
button.delete {
  background: #e74c3c;
}
button.delete:hover {
  background: #c0392b;
}
button.cancel {
  background: #95a5a6;
}
button.cancel:hover {
  background: #7f8c8d;
}
ul.todo-list {
  list-style: none;
  padding: 0;
}
ul.todo-list li {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  margin-bottom: 0.7rem;
  background: #333;
  padding: 0.5rem;
  border-radius: 4px;
}
.done {
  text-decoration: line-through;
  color: gray;
}
</style>
