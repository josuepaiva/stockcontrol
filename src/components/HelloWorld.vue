<script setup lang="ts">
import { ref, computed } from 'vue'
import Sidebar from './Sidebar.vue'

interface Product {
  id: number
  name: string
  amount: number
  price: number
}

type SortKey = 'name' | 'amount' | 'price'

defineProps<{
  msg: string
}>()

// Navigation
const currentRoute = ref('products')

// Search and filters
const searchQuery = ref('')
const minAmount = ref('')
const maxAmount = ref('')
const minPrice = ref('')
const maxPrice = ref('')

// Products data
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
  price: 0.00
})

const editingProduct = ref<Product | null>(null)
const showForm = ref(false)

const errors = ref({
  name: '',
  price: '',
  amount: ''
})

// Sorting
const sortConfig = ref({
  key: 'name' as SortKey,
  direction: 'asc' as 'asc' | 'desc'
})

// Computed properties
const filteredProducts = computed(() => {
  return products.value.filter(product => {
    const nameMatch = product.name.toLowerCase().includes(searchQuery.value.toLowerCase())
    const amountMatch = (!minAmount.value || product.amount >= Number(minAmount.value)) &&
                       (!maxAmount.value || product.amount <= Number(maxAmount.value))
    const priceMatch = (!minPrice.value || product.price >= Number(minPrice.value)) &&
                      (!maxPrice.value || product.price <= Number(maxPrice.value))
    
    return nameMatch && amountMatch && priceMatch
  })
})

const sortedProducts = computed(() => {
  const sorted = [...filteredProducts.value]
  sorted.sort((a, b) => {
    const aValue = a[sortConfig.value.key]
    const bValue = b[sortConfig.value.key]
    
    if (sortConfig.value.direction === 'asc') {
      return aValue > bValue ? 1 : -1
    } else {
      return aValue < bValue ? 1 : -1
    }
  })
  return sorted
})

// Functions
function handleNavigation(route: string) {
  currentRoute.value = route
}

function validateName() {
  if (!newProduct.value.name.trim()) {
    errors.value.name = 'O nome do produto é obrigatório'
  } else {
    errors.value.name = ''
  }
}

function validatePrice() {
  const price = parseFloat(newProduct.value.price.toString())
  if (isNaN(price) || price <= 0) {
    errors.value.price = 'O preço deve ser maior que 0'
  } else {
    errors.value.price = ''
  }
}

function validateAmount() {
  if (newProduct.value.amount <= 0 || isNaN(newProduct.value.amount)) {
    errors.value.amount = 'A quantidade deve ser maior que 0'
  } else {
    errors.value.amount = ''
  }
}

function validateAndSubmit() {
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
  const product = products.value.find(p => p.id === id)
  if (product && confirm(`Tem certeza que deseja remover o produto "${product.name}"?`)) {
    products.value = products.value.filter(product => product.id !== id)
  }
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
    price: 0.00
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

function sortTable(key: SortKey) {
  if (sortConfig.value.key === key) {
    sortConfig.value.direction = sortConfig.value.direction === 'asc' ? 'desc' : 'asc'
  } else {
    sortConfig.value.key = key
    sortConfig.value.direction = 'asc'
  }
}
</script>

<template>
  <div class="app-container">
    <Sidebar @navigate="handleNavigation" />
    <div class="main-content">
      <template v-if="currentRoute === 'products'">
        <h1 class="green">{{ msg }}</h1>
        
        <div class="filters-container">
          <div class="search-container">
            <input 
              v-model="searchQuery" 
              type="text" 
              placeholder="Buscar por nome..."
              class="search-input"
            />
          </div>
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
                step="0.01"
                min="0"
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
                <th @click="sortTable('name')" class="sortable">
                  Nome
                  <span v-if="sortConfig.key === 'name'" class="sort-indicator">
                    {{ sortConfig.direction === 'asc' ? '↑' : '↓' }}
                  </span>
                </th>
                <th @click="sortTable('amount')" class="sortable">
                  Quantidade
                  <span v-if="sortConfig.key === 'amount'" class="sort-indicator">
                    {{ sortConfig.direction === 'asc' ? '↑' : '↓' }}
                  </span>
                </th>
                <th @click="sortTable('price')" class="sortable">
                  Preço
                  <span v-if="sortConfig.key === 'price'" class="sort-indicator">
                    {{ sortConfig.direction === 'asc' ? '↑' : '↓' }}
                  </span>
                </th>
                <th>Ações</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="product in sortedProducts" :key="product.id">
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
      </template>
    </div>
  </div>
</template>

<style scoped>
.app-container {
  display: flex;
  min-height: 100vh;
}

.main-content {
  flex: 1;
  margin-left: 200px;
  padding: 20px;
}

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

.filters-container {
  margin: 20px 0;
  background: #f5f5f5;
  padding: 15px;
  border-radius: 8px;
}

.search-container {
  margin-bottom: 15px;
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

.sortable {
  cursor: pointer;
  user-select: none;
}

.sortable:hover {
  background-color: #e0e0e0;
}

.sort-indicator {
  margin-left: 5px;
  font-size: 0.8em;
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

.filter-group {
  display: flex;
  gap: 15px;
  flex-wrap: wrap;
}

.filter-item {
  flex: 1;
  min-width: 200px;
}

.filter-item label {
  display: block;
  margin-bottom: 5px;
  font-weight: 500;
}

.filter-input {
  width: 100%;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
}
</style>
