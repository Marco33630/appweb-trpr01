<script setup lang="ts">
import { ref, defineEmits, watch, PropType, watchEffect } from 'vue';
import { Product } from '../script/types';

const props = defineProps<{
  product?: Product | null;
}>();

const newProductTitle = ref('');
const newProductDescription = ref('');
const newProductPrice = ref('');
const newProductStocks = ref('');

const emits = defineEmits(['create-product']);

//Watcher permettant de vérifier si modification ou création de produit
//watchEffect à été trouvé sur la documentation de vuejs : https://vuejs.org/guide/essentials/watchers.html
watchEffect(async() => {
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
  if (newProductTitle.value.trim()) {
    const price = newProductPrice.value ? parseFloat(newProductPrice.value) : 0;
    const stock = newProductStocks.value? parseInt(newProductStocks.value) : 0;
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
}
</script>

<template>
  <div class="container py-4">
    <div class="card shadow">
      <div class="card-header bg-primary text-white">
        <h5 class="mb-0">Add New Product</h5>
      </div>
      <div class="card-body">
        <form @submit.prevent="createNewProduct">
          <div class="mb-3">
            <label for="productName" class="form-label">Product Name</label>
            <input 
              type="text" 
              class="form-control" 
              id="productName"
              v-model="newProductTitle" 
              placeholder="Enter product name" 
              required
            />
          </div>

          <div class="mb-3">
            <label for="productDescription" class="form-label">Description</label>
            <textarea 
              class="form-control" 
              id="productDescription" 
              v-model="newProductDescription" 
              placeholder="Enter product description"
              rows="3"
            ></textarea>
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
                placeholder="0.00" 
                pattern="^\d*(\.\d{0,2})?$"
                title="Please enter a valid price (e.g. 10.99)"
                inputmode="decimal"
              />
            </div>
            <div class="form-text" v-if="newProductPrice && isNaN(parseFloat(newProductPrice))">
              Please enter a valid price
            </div>
          </div>

          <div class="mb-3">
            <label for="productStock" class="form-label">Stock Quantity</label>
            <input 
              type="number" 
              class="form-control" 
              id="productStock"
              v-model="newProductStocks" 
              placeholder="Enter stock quantity" 
              min="0"
              step="1"
              required
            />
          </div>

          <div class="d-grid">
            <button 
              type="submit" 
              class="btn btn-success" 
            >
              <i class="bi bi-plus-circle me-2"></i>Add Product
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>


<style scoped>

</style>