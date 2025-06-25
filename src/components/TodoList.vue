<!-- src/components/TodoList.vue -->
<template>
  <div class="max-w-xl mx-auto">
    <!-- ─── Form tambah todo ──────────────────────────────────────────────── -->
    <div class="flex gap-2 mb-4">
      <input
        v-model="newTodo"
        @keyup.enter="addTodo"
        type="text"
        placeholder="Tambah todo baru..."
        class="flex-1 rounded border px-3 py-2 focus:outline-none focus:ring"
      />
      <button
        @click="addTodo"
        class="rounded bg-blue-500 px-4 py-2 font-semibold text-white transition hover:bg-blue-600"
      >
        Tambah
      </button>
    </div>

    <!-- ─── Daftar todo ───────────────────────────────────────────────────── -->
    <ul>
      <li
        v-for="todo in todos"
        :key="todo.id"
        class="mb-2 flex items-center gap-2"
      >
        <input
          type="checkbox"
          v-model="todo.completed"
          @change="toggleComplete(todo)"
          class="accent-blue-500"
        />

        <span
  :class="[
    'flex-1 text-lg',  // <-- tambah ukuran teks
    todo.completed ? 'line-through text-gray-500' : 'text-gray-200'
  ]"
>
  {{ todo.text }}
</span>


        <button
          @click="startEdit(todo)"
          class="rounded bg-gray-100 px-3 py-1 text-sm font-medium transition hover:bg-gray-200"
        >
          Edit
        </button>
        <button
          @click="deleteTodo(todo.id)"
          class="rounded bg-gray-100 px-3 py-1 text-sm font-medium transition hover:bg-gray-200"
        >
          Hapus
        </button>
      </li>
    </ul>

    <!-- ─── Mode edit (inline) ────────────────────────────────────────────── -->
    <div v-if="editing" class="mt-4 space-y-2">
      <input
        v-model="editedText"
        type="text"
        class="w-full rounded border px-3 py-2 focus:outline-none focus:ring"
      />
      <div class="flex gap-2">
        <button
          @click="saveEdit"
          class="rounded bg-blue-500 px-4 py-2 font-semibold text-white transition hover:bg-blue-600"
        >
          Simpan
        </button>
        <button
          @click="cancelEdit"
          class="rounded bg-gray-100 px-4 py-2 font-medium transition hover:bg-gray-200"
        >
          Batal
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

/* ── reactive state ──────────────────────────────────────────────────────── */
const todos = ref([])
const newTodo = ref('')

/* mode edit */
const editing = ref(false)
const editedText = ref('')
let editedTodo = null

/* ── helper ──────────────────────────────────────────────────────────────── */
const API = 'http://localhost:3000/todos'

const fetchTodos = async () => {
  const { data } = await axios.get(API)
  todos.value = data
}

const addTodo = async () => {
  if (!newTodo.value.trim()) return
  await axios.post(API, { text: newTodo.value, completed: false })
  newTodo.value = ''
  fetchTodos()
}

const deleteTodo = async id => {
  await axios.delete(`${API}/${id}`)
  fetchTodos()
}

const toggleComplete = async todo => {
  await axios.patch(`${API}/${todo.id}`, { completed: todo.completed })
}

const startEdit = todo => {
  editedTodo = todo
  editedText.value = todo.text
  editing.value = true
}

const saveEdit = async () => {
  if (!editedText.value.trim()) return
  await axios.patch(`${API}/${editedTodo.id}`, { text: editedText.value })
  cancelEdit()
  fetchTodos()
}

const cancelEdit = () => {
  editing.value = false
  editedTodo = null
  editedText.value = ''
}

/* ── lifecycle ───────────────────────────────────────────────────────────── */
onMounted(fetchTodos)
</script>
