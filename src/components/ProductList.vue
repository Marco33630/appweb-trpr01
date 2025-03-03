<script setup lang="ts">
import { ref, computed } from 'vue';
import { type Product, type ProductData } from '../script/types';
import ProductSearchBar from './ProductSearchBar.vue';
import ProductItem from './ProductItem.vue';
import ProductForm from './ProductForm.vue';

const products = ref<Product[]>([
    { id: 1, name: 'Definitely Iphone 16 (Trust)', description: 'This is P1', price: 10.59, stock: 90 },
    { id: 2, name: 'Real Rolex Watch', description: 'This is P2', price: 1.99, stock: 55 },
    { id: 3, name: 'Louis Vuiton Bag', description: 'This is P3', price: 15.99, stock: 15 },
    { id: 4, name: 'Samsung Smart Fridge', description: 'Best fridge ever created', price: 4999.99, stock: 15 },
]);

const showModal = ref(false);
const isEditingProduct = ref(false);
const currentProduct = ref<Product | null>(null);
const selectedProduct = ref<Product | null>(null);
const showDetailsModal = ref(false);
const showDeleteModal = ref(false);
const productToDelete = ref<number | null>(null);
const searchQuery = ref('');
const showNotificationModal = ref(false);
const notificationMessage = ref('');

//Cette fonction a été en partie générer par Claude 3.7 Sonnet 
function exportToCSV() {
  const headers = ['id', 'name', 'description', 'price', 'stock'];
  
  let csvContent = headers.join(',') + '\n';

  products.value.forEach(product => {
    const row = [
      product.id,
      product.name,
      product.description,
      product.price,
      product.stock
    ];
    csvContent += row.join(',') + '\n';
  });
  
  const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
  
  const link = document.createElement('a');
  
  const url = URL.createObjectURL(blob);
  
  link.setAttribute('href', url);
  link.setAttribute('download', 'product_list.csv');
  
  document.body.appendChild(link);
  
  link.click();
  
  document.body.removeChild(link);
  URL.revokeObjectURL(url);
}

const filteredProducts = computed(() => {
    if (!searchQuery.value) {
        return products.value;
    }
    return products.value.filter(product =>
        product.name.toLowerCase().includes(searchQuery.value.toLowerCase())
    );
});

function checkStock(product: Product) {
    if (product.stock === 0) {
        notificationMessage.value = `Warning: The stock for "${product.name}" is empty!`;
        showNotificationModal.value = true;
    }
}

//Proposition de Chat GPT d'utilisé un DAO ProductData pour la création d'un objet Product
function addProduct(newProduct: ProductData) {
    const nextProductId = Math.max(...products.value.map(p => p.id), 0) + 1;
    const product: Product = {
        id: nextProductId,
        name: newProduct.name,
        description: newProduct.description,
        price: newProduct.price,
        stock: newProduct.stock,
    };
    products.value.push(product);
    checkStock(product);
    closeModal();
}

function viewProductDetails(product: Product) {
    selectedProduct.value = product;
    showDetailsModal.value = true;
}

function closeDetailsModal() {
    showDetailsModal.value = false;
    selectedProduct.value = null;
}

function openModal(product: Product | null = null) {
    if(product){
        isEditingProduct.value = true;
        currentProduct.value = {...product };
    }else{
        isEditingProduct.value = false;
        currentProduct.value = null;
    }
    showModal.value = true;
}

function closeModal() {
    showModal.value = false;
    currentProduct.value = null;
}

function updateProduct(updatedProduct: ProductData) {
    if (currentProduct.value) {
        const index = products.value.findIndex(p => p.id === currentProduct.value!.id);
        if (index !== -1) {
            products.value[index] = { ...currentProduct.value, ...updatedProduct };
            checkStock(products.value[index]);
        }
    }
    closeModal();
}

function duplicateProduct(product: Product){
    const nextProductId = products.value.length > 0 ? Math.max(...products.value.map(product => product.id)) + 1 : 1;
    const duplicatedProduct = {
        ...product,
        id: nextProductId,
        name: `${product.name} (copy)`,
        description: product.description,
        price: product.price,
        stock: product.stock,
    }
    products.value.push(duplicatedProduct);
    checkStock(duplicatedProduct);
}

function confirmDelete(id: number) {
    productToDelete.value = id;
    showDeleteModal.value = true;
}

function deleteProduct() {
    if (productToDelete.value !== null) {
        products.value = products.value.filter(product => product.id !== productToDelete.value);
        closeDeleteModal();
    }
}

function closeDeleteModal() {
    showDeleteModal.value = false;
    productToDelete.value = null;
}
</script>

<template>
    <div class="h-100 d-flex align-items-center justify-content-center">
        <div class="container">
            <h2 class="text-center mb-4">Product List</h2>
            <div class="d-flex justify-content-center mb-3">
                <button class="btn btn-info" @click="() => openModal()">
                    Create a new product
                </button>
                <button class="btn btn-outline-secondary ms-2" @click="exportToCSV">
                    Export To CSV
                </button>
            </div>

            <ProductSearchBar @update-search="searchQuery = $event" />

            <div class="row row-cols-1 row-cols-md-3 row-cols-lg-6 g-3 mt-3">
                <div v-for="product in filteredProducts" :key="product.id" class="col">
                    <ProductItem 
                        :product="product" 
                        @view-details="viewProductDetails"
                        @edit-product="openModal"
                        @duplicate-product="duplicateProduct"
                        @delete-product="confirmDelete"
                    />
                </div>
            </div>
        </div>

        <!-- Modal -->
        <div class="modal fade" :class="{ show: openModal }" tabindex="-1" :style="{ display: showModal ? 'block' : 'none' }" aria-labelledby="exampleModalLabel" aria-modal="true" role="dialog">
            <div class="modal-dialog">
                <div class="modal-content">
                    <div class="modal-header">
                        <button type="button" class="btn-close" @click="closeModal" aria-label="Close"></button>
                    </div>
                    <div class="modal-body">
                        <ProductForm 
                            v-if="isEditingProduct"
                            :product="currentProduct" 
                            @create-product="updateProduct" 
                        />
                        <ProductForm 
                            v-else 
                            @create-product="addProduct" 
                        />
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" @click="closeModal">Close</button>
                    </div>
                </div>
            </div>
        </div>

        <!-- Details Modal -->
        <div 
            class="modal fade" 
            v-if="showDetailsModal"
            :class="{ show: showDetailsModal }" 
            tabindex="-1" 
            :style="{ display: showDetailsModal ? 'block' : 'none' }" 
            role="dialog"
        >
            <div class="modal-dialog">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title">{{ selectedProduct?.name }}</h5>
                        <button type="button" class="btn-close" @click="closeDetailsModal" aria-label="Close"></button>
                    </div>
                    <div class="modal-body">
                        <p><strong>Description:</strong> {{ selectedProduct?.description }}</p>
                        <p><strong>Price:</strong> ${{ selectedProduct?.price.toFixed(2) }}</p>
                        <p><strong>Stock:</strong> {{ selectedProduct?.stock }}</p>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" @click="closeDetailsModal">Close</button>
                    </div>
                </div>
            </div>
        </div>

        <!-- Delete Confirmation Modal -->
        <div 
            class="modal fade" 
            v-if="showDeleteModal"
            :class="{ show: showDeleteModal }" 
            tabindex="-1" 
            :style="{ display: showDeleteModal ? 'block' : 'none' }" 
            role="dialog"
        >
            <div class="modal-dialog">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title">Delete Product</h5>
                        <button type="button" class="btn-close" @click="closeDeleteModal" aria-label="Close"></button>
                    </div>
                    <div class="modal-body">
                        <p>Are you sure you want to delete this product? This action cannot be undone.</p>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" @click="closeDeleteModal">Cancel</button>
                        <button type="button" class="btn btn-danger" @click="deleteProduct">Delete</button>
                    </div>
                </div>
            </div>
        </div>

        <!-- Notification Modal -->
        <div 
            class="modal fade" 
            v-if="showNotificationModal" 
            :class="{ show: showNotificationModal }" 
            tabindex="-1" 
            :style="{ display: showNotificationModal ? 'block' : 'none' }" 
            role="dialog"
        >
            <div class="modal-dialog">
                <div class="modal-content bg-danger text-white">
                    <div class="modal-header">
                        <h5 class="modal-title">Warning</h5>
                        <button type="button" class="btn-close" @click="showNotificationModal = false" aria-label="Close"></button>
                    </div>
                    <div class="modal-body">
                        <p>{{ notificationMessage }}</p>
                    </div>
                </div>
            </div>
        </div>

        <div v-if="showModal || showDetailsModal || showDeleteModal || showNotificationModal" class="modal-backdrop fade show"></div>
    </div>
</template>

<style scoped>
:deep(body.modal-open) {
    overflow: hidden;
    padding-right: 17px;
}

.modal.show {
    background-color: rgba(0, 0, 0, 0.5);
}
</style>