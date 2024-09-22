<script setup lang="ts">
defineProps<{
  msg: string
}>()

import { ref } from 'vue'

const products = ref([
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

const errors = ref({
  name: '',
  price: '',
  amount: ''
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
    addProduct()
  }
}

const showForm = ref(false)

function addProduct() {
  const productToAdd = { ...newProduct.value, id: Date.now() }
  products.value.push(productToAdd)
  newProduct.value.name = ''
  newProduct.value.amount = 0
  newProduct.value.price = 0
  showForm.value = false
}
</script>

<template>
  <div class="greetings">
    <h1 class="green">{{ msg }}</h1>
    <div>
      <button @click="showForm = !showForm">
        {{ showForm ? 'Cancelar' : 'Adicionar Produto' }}
      </button>
      <div v-if="showForm">
        <form @submit.prevent="validateAndSubmit">
          <label for="name">Nome:</label><br />
          <input
            v-model="newProduct.name"
            type="text"
            id="name"
            name="name"
            value="gloss bocão"
            required
          /><br />
          <span v-if="errors.name" class="error">{{ errors.name }}</span>
          <br />
          <label for="amount">Quantidade:</label><br />
          <input
            v-model.number="newProduct.amount"
            type="text"
            id="amount"
            name="amount"
            value="maior que zero"
            required
          /><br />
          <span v-if="errors.amount" class="error">{{ errors.amount }}</span>
          <br />
          <label for="amount">Preço:</label><br />
          <input
            v-model.number="newProduct.price"
            type="text"
            id="amount"
            name="amount"
            value="maior que zero"
            required
          /><br />
          <span v-if="errors.price" class="error">{{ errors.price }}</span>
          <br />
          <input type="submit" value="Salvar" />
        </form>
      </div>
    </div>
    <div>
      <ul>
        <li v-for="product of products" :key="product.id">{{ product.name }}</li>
      </ul>
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

h3 {
  font-size: 1.2rem;
}

.greetings h1,
.greetings h3 {
  text-align: center;
}

@media (min-width: 1024px) {
  .greetings h1,
  .greetings h3 {
    text-align: left;
  }
}
</style>
