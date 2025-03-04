<script setup lang="ts">
import { ref, defineEmits, watchEffect } from 'vue';
import { type Product } from '../script/types';

const props = defineProps<{
  product?: Product | null;
}>();

const newProductTitle = ref('');
const newProductDescription = ref('');
const newProductPrice = ref('');
const newProductStocks = ref('');

const emits = defineEmits(['create-product']);

const errors = ref({
  title: '',
  description: '',
  price: '',
  stock: ''
});

const ERROR_MESSAGES = {
  title: 'Le titre ne doit pas être vide',
  description: 'La description ne doit pas être vide',
  price: 'Le prix doit être un nombre valide et doit respecter le format 10.99',
  stock: 'La quantité en stock doit être un nombre valide',
};

function displayError(field: keyof typeof errors.value, message: string) {
  errors.value[field] = message;
}

function clearError(field: keyof typeof errors.value) {
  errors.value[field] = '';
}

function validateField(field: keyof typeof errors.value, value: string) {
  clearError(field);
  switch (field) {
    case 'title':
      if (!value.trim()) displayError(field, ERROR_MESSAGES.title);
      break;
    case 'description':
      if (!value.trim()) displayError(field, ERROR_MESSAGES.description);
      break;
    case 'price':
      if (!/^[0-9]+(\.[0-9]{1,2})?$/.test(value) || parseFloat(value) < 0) {
        displayError(field, ERROR_MESSAGES.price);
      }
      break;
    case 'stock':
      if (!/^[0-9]+$/.test(value) || parseInt(value) < 0) {
        displayError(field, ERROR_MESSAGES.stock);
      }
      break;
  }
}

watchEffect(() => {
  if (props.product) {
    newProductTitle.value = props.product.name;
    newProductDescription.value = props.product.description;
    newProductPrice.value = props.product.price.toString();
    newProductStocks.value = props.product.stock.toString();
  } else {
    newProductTitle.value = '';
    newProductDescription.value = '';
    newProductPrice.value = '';
    newProductStocks.value = '';
  }
});

function createNewProduct() {
  validateField('title', newProductTitle.value);
  validateField('description', newProductDescription.value);
  validateField('price', newProductPrice.value);
  validateField('stock', newProductStocks.value);

  if (Object.values(errors.value).some(error => error)) return;

  const price = newProductPrice.value ? parseFloat(newProductPrice.value) : 0;
  const stock = newProductStocks.value ? parseInt(newProductStocks.value) : 0;

  emits('create-product', {
    name: newProductTitle.value.trim(),
    description: newProductDescription.value,
    price: price,
    stock: stock
  });

  newProductTitle.value = '';
  newProductDescription.value = '';
  newProductPrice.value = '';
  newProductStocks.value = '';
}
</script>

<template>
  <div class="container py-4">
    <div class="card shadow">
      <div class="card-header bg-primary text-white">
        <h5 class="mb-0">Add New Product</h5>
      </div>
      <div class="card-body">
        <form @submit.prevent="createNewProduct" novalidate>
          <div class="mb-3">
            <label for="productName" class="form-label">Product Name</label>
            <input 
              type="text" 
              class="form-control" 
              id="productName"
              v-model="newProductTitle"
              @blur="validateField('title', newProductTitle)" 
              placeholder="Enter product name" 
              required
            />
            <div v-if="errors.title" class="text-danger">{{ errors.title }}</div>
          </div>

          <div class="mb-3">
            <label for="productDescription" class="form-label">Description</label>
            <textarea 
              class="form-control" 
              id="productDescription" 
              v-model="newProductDescription"
              @blur="validateField('description', newProductDescription)" 
              placeholder="Enter product description"
              rows="3"
            ></textarea>
            <div v-if="errors.description" class="text-danger">{{ errors.description }}</div>
          </div>

          <div class="mb-3">
            <label for="productPrice" class="form-label">Price</label>
            <div class="input-group">
              <span class="input-group-text">$</span>
              <input 
                type="text" 
                class="form-control" 
                id="productPrice" 
                v-model="newProductPrice" 
                @blur="validateField('price', newProductPrice)"
                placeholder="0.00" 
                inputmode="decimal"
              />
            </div>
            <div v-if="errors.price" class="text-danger">{{ errors.price }}</div>
          </div>

          <div class="mb-3">
            <label for="productStock" class="form-label">Stock Quantity</label>
            <input 
              type="number" 
              class="form-control" 
              id="productStock"
              v-model="newProductStocks"
              @blur="validateField('stock', newProductStocks)" 
              placeholder="Enter stock quantity" 
              min="0"
              step="1"
              required
            />
            <div v-if="errors.stock" class="text-danger">{{ errors.stock }}</div>
          </div>

          <div class="d-grid">
            <button type="submit" class="btn btn-success">Add Product</button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>


<style scoped>
</style>