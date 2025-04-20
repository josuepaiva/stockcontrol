<script setup lang="ts">
defineProps<{
  msg: string
}>()

import { ref, computed } from 'vue'

interface Product {
  id: number
  name: string
  amount: number
  price: number
}

const products = ref<Product[]>([
  {
    id: 1,
    name: 'Base lanfy',
    amount: 1,
    price: 20
  },
  {
    id: 2,
    name: 'po compacto',
    amount: 2,
    price: 20
  }
])

const newProduct = ref({
  name: '',
  amount: 0,
  price: 0
})

const searchQuery = ref('')
const editingProduct = ref<Product | null>(null)

const errors = ref({
  name: '',
  price: '',
  amount: ''
})

const filteredProducts = computed(() => {
  return products.value.filter(product => 
    product.name.toLowerCase().includes(searchQuery.value.toLowerCase())
  )
})

const validateName = () => {
  if (!newProduct.value.name.trim()) {
    errors.value.name = 'O nome do produto é obrigatório'
  } else {
    errors.value.name = ''
  }
}

const validatePrice = () => {
  if (newProduct.value.price <= 0 || isNaN(newProduct.value.price)) {
    errors.value.price = 'O preço deve ser maior que 0'
  } else {
    errors.value.price = ''
  }
}

const validateAmount = () => {
  if (newProduct.value.amount <= 0 || isNaN(newProduct.value.amount)) {
    errors.value.amount = 'A quantidade deve ser maior que 0'
  } else {
    errors.value.amount = ''
  }
}

const validateAndSubmit = () => {
  validateName()
  validatePrice()
  validateAmount()

  if (!errors.value.name && !errors.value.price && !errors.value.amount) {
    if (editingProduct.value) {
      updateProduct()
    } else {
      addProduct()
    }
  }
}

const showForm = ref(false)

function addProduct() {
  const productToAdd = { ...newProduct.value, id: Date.now() }
  products.value.push(productToAdd)
  resetForm()
}

function updateProduct() {
  if (editingProduct.value) {
    const index = products.value.findIndex(p => p.id === editingProduct.value!.id)
    if (index !== -1) {
      products.value[index] = { ...newProduct.value, id: editingProduct.value!.id }
    }
    resetForm()
    editingProduct.value = null
  }
}

function deleteProduct(id: number) {
  products.value = products.value.filter(product => product.id !== id)
}

function editProduct(product: Product) {
  editingProduct.value = product
  newProduct.value = { ...product }
  showForm.value = true
}

function resetForm() {
  newProduct.value = {
    name: '',
    amount: 0,
    price: 0
  }
  showForm.value = false
  errors.value = {
    name: '',
    price: '',
    amount: ''
  }
}

function cancelEdit() {
  resetForm()
  editingProduct.value = null
}
</script>

<template>
  <div class="greetings">
    <h1 class="green">{{ msg }}</h1>
    
    <div class="search-container">
      <input 
        v-model="searchQuery" 
        type="text" 
        placeholder="Buscar produtos..."
        class="search-input"
      />
    </div>

    <div class="button-container">
      <button @click="showForm = !showForm; if(showForm) editingProduct = null">
        {{ showForm ? 'Cancelar' : 'Adicionar Produto' }}
      </button>
    </div>

    <div v-if="showForm" class="form-container">
      <form @submit.prevent="validateAndSubmit">
        <div class="form-group">
          <label for="name">Nome:</label>
          <input
            v-model="newProduct.name"
            type="text"
            id="name"
            name="name"
            required
          />
          <span v-if="errors.name" class="error">{{ errors.name }}</span>
        </div>

        <div class="form-group">
          <label for="amount">Quantidade:</label>
          <input
            v-model.number="newProduct.amount"
            type="number"
            id="amount"
            name="amount"
            required
          />
          <span v-if="errors.amount" class="error">{{ errors.amount }}</span>
        </div>

        <div class="form-group">
          <label for="price">Preço:</label>
          <input
            v-model.number="newProduct.price"
            type="number"
            id="price"
            name="price"
            required
          />
          <span v-if="errors.price" class="error">{{ errors.price }}</span>
        </div>

        <div class="form-actions">
          <button type="submit">{{ editingProduct ? 'Atualizar' : 'Salvar' }}</button>
          <button type="button" @click="cancelEdit" v-if="editingProduct">Cancelar</button>
        </div>
      </form>
    </div>

    <div class="table-container">
      <table>
        <thead>
          <tr>
            <th>Nome</th>
            <th>Quantidade</th>
            <th>Preço</th>
            <th>Ações</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="product in filteredProducts" :key="product.id">
            <td>{{ product.name }}</td>
            <td>{{ product.amount }}</td>
            <td>R$ {{ product.price.toFixed(2) }}</td>
            <td>
              <button @click="editProduct(product)" class="action-button edit">Editar</button>
              <button @click="deleteProduct(product.id)" class="action-button delete">Remover</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<style scoped>
.error {
  color: red;
  font-size: 0.9em;
}

h1 {
  font-weight: 500;
  font-size: 2.6rem;
  position: relative;
  top: -10px;
}

.greetings {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

.search-container {
  margin: 20px 0;
}

.search-input {
  width: 100%;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.button-container {
  margin: 20px 0;
}

.form-container {
  background: #f5f5f5;
  padding: 20px;
  border-radius: 8px;
  margin: 20px 0;
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
}

.form-group input {
  width: 100%;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.form-actions {
  display: flex;
  gap: 10px;
  margin-top: 20px;
}

.table-container {
  margin-top: 20px;
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
}

th, td {
  padding: 12px;
  text-align: left;
  border-bottom: 1px solid #ddd;
}

th {
  background-color: #f5f5f5;
}

.action-button {
  padding: 6px 12px;
  margin: 0 5px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.edit {
  background-color: #4CAF50;
  color: white;
}

.delete {
  background-color: #f44336;
  color: white;
}

button {
  padding: 8px 16px;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:hover {
  opacity: 0.9;
}

@media (min-width: 1024px) {
  .greetings h1 {
    text-align: left;
  }
}
</style>
