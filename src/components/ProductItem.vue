<script setup lang="ts">
import { defineProps, defineEmits } from 'vue';
import { type Product } from '../script/types';

const props = defineProps<{ product: Product }>();
const emits = defineEmits(['view-details', 'edit-product', 'duplicate-product', 'delete-product']);
</script>

<template>
  <div class="product-card bg-dark text-white h-100">
    <div class="product-name">{{ props.product.name }}</div>

    <div class="product-price">PRICE ${{ props.product.price.toFixed(2) }}</div>

    <div class="product-stock" :class="[
      props.product.stock === 0 ? 'stock-empty' :
      props.product.stock < 10 ? 'stock-low' : 'stock-good'
    ]">
      STOCK {{ props.product.stock }}
    </div>

    <div class="product-actions">
      <button class="btn btn-sm btn-outline-light mb-1 w-100" @click="emits('view-details', props.product)">Détail</button>
      <button class="btn btn-sm btn-outline-success mb-1 w-100" @click="emits('edit-product', props.product)">Modifier</button>
      <button class="btn btn-sm btn-outline-warning mb-1 w-100" @click="emits('duplicate-product', props.product)">Dupliquer</button>
      <button class="btn btn-sm btn-outline-danger w-100" @click="emits('delete-product', props.product.id)">Supprimer</button>
    </div>

  </div>
</template>

<style scoped>

/*Claude 3.7 à été utilisé pour créer les bases des classes CSS suivantes*/
.product-card {
  position: relative;
  border: 1px solid #333;
  height: 400px;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.product-name {
  padding: 10px;
  text-align: center;
  font-weight: bold;
  border-bottom: 1px solid #333;
  font-size: 0.9rem;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  margin: 1em;
}

.product-price, .product-stock {
  padding: 5px 10px;
  text-align: center;
  font-size: 0.8rem;
  border-bottom: 1px solid #333;
  margin: 1em;
}

.stock-empty {
  background-color: rgba(220, 53, 69, 0.3);
}

.stock-low {
  background-color: rgba(255, 193, 7, 0.3);
}

.stock-good {
  background-color: rgba(40, 167, 69, 0.3);
}

.product-actions {
  padding: 10px;
  display: flex;
  flex-direction: column;
  gap: 5px;
  margin: 1em;
}
</style>
