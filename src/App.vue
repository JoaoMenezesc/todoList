<script setup>
import { ref, onMounted, computed, watch } from 'vue'
import axios from 'axios'

const apiUrl = 'http://localhost:3000'
const todoList = ref([])
const name = ref('')
const nameList = ref([])
let inputContent = ref('')

const ascTodos = computed(() => todoList.value.sort((a, b) => {
  return b.createdAt - a.createdAt
}))

const fetchTodos = async () => {
  try {
    const responseTodos = await axios.get(`${apiUrl}/todos`)
    todoList.value = responseTodos.data
  } catch (error) {
    console.error('Error fetching todos:', error)
  }
}

const fetchName = async () => {
  try {
    const responseName = await axios.get(`${apiUrl}/name/`)
    nameList.value = responseName.data
  } catch (error) {
    console.error('Error fetching name:', error)
  }
}

const addName = async () => {
  const newNameObj = {
    value: name.value,
    createdAt: new Date().getTime()
  }

  if (nameList.value.find(n => n.value === name.value)) {
    return 
  }
  else {
  try {
    const responseAddName = await axios.post(`${apiUrl}/name`, newNameObj)
    nameList.value.push(responseAddName.data)
  } catch (error) {
    console.error('Error adding name:', error)
  }
}}

const addTodo = async () => {
  if (inputContent.value.trim() === '') {
    return
  }

  const newTodo = {
    content: inputContent.value,
    nameTodo: name.value,
    done: false,
    createdAt: new Date().getTime()
  }
  try {
    const response = await axios.post(`${apiUrl}/todos`, newTodo)
    todoList.value.push(response.data)
    inputContent.value = ''
    addName()
  } catch (error) {
    console.error('Error adding todo:', error)
  }
}

const removeTodo = async (todo) => {
  try {
    await axios.delete(`${apiUrl}/todos/${todo.id}`)
    todoList.value = todoList.value.filter(t => t.id !== todo.id)
    removeName()
  } catch (error) {
    console.error('Error removing todo:', error)
  }
}

const updateTodo = async (todo) => {
  try {
    await axios.put(`${apiUrl}/todos/${todo.id}`, todo)
    await axios.put(`${apiUrl}/name/${todo.id}`, todo)
  } catch (error) {
    console.error('Error updating todo:', error)
  }
}

watch(todoList, (newVal) => {
  newVal.forEach(todo => updateTodo(todo))
}, { deep: true })


onMounted(() => {
  fetchName()
  fetchTodos()
})
</script>

<template>
<main class="app">
  <section class="greeting">
    <h1 class="list">
      Oi <input type="text" placeholder="nome aqui" v-model="name" />
    </h1>
  </section>

  <section class="create-todo">
    <form @submit.prevent="addTodo()">
      <h3>Adicione itens na lista!</h3>
      <input 
      type="text" 
      placeholder="ex: estudar" 
      v-model="inputContent">

      <input type="submit" value="Adicionar tarefa">
    </form>
  </section>

  <section class="todo-list">
    <h3>TODO LIST</h3>
    <div class="list">
        <div v-for="todo in ascTodos" :key="todo.id" :class="`todo-item ${todo.done && 'done'}`">
          <label>
            <input type="checkbox" v-model="todo.done">
            <span :class="`bubble ${todo.category}`"></span>
          </label>
          <div class="todo-content">
            <input type="text" v-model="todo.content">
          </div>
          <div class="actions">
            <button class="delete" @click="removeTodo(todo)">Apagar</button>
          </div>
        </div>
    </div>
  </section>
</main>
</template>
