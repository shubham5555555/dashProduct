
<template>
  <div class="container mt-4">
    <h1 class="mb-4">Product Dashboard</h1>
    <button class="btn btn-primary mb-3" @click="showAddModal">Add Product</button>

    <!-- Add/Edit Product Modal -->
    <div class="modal" :class="{ 'is-active': isModalVisible }">
      <div class="modal-background" @click="hideModal"></div>
      <div class="modal-content">
        <div class="card">
          <header class="card-header">
            <p class="card-header-title">{{ modalTitle }}</p>
            <button class="delete" aria-label="close" @click="hideModal"></button>
          </header>
          <section class="card-content">
            <div class="content">
              <form @submit.prevent="addOrUpdateProduct">
                <div class="field">
                  <label class="label">Product Name</label>
                  <div class="control">
                    <input type="text" class="input" v-model="productForm.name" required>
                  </div>
                </div>
                <div class="field">
                  <label class="label">Product Price</label>
                  <div class="control">
                    <input type="number" class="input" v-model="productForm.price" required>
                  </div>
                </div>
                <div class="field">
                  <label class="label">Product Quantity</label>
                  <div class="control">
                    <input type="number" class="input" v-model="productForm.quantity" required>
                  </div>
                </div>
                <div class="field">
                  <label class="label">Product Description</label>
                  <div class="control">
                    <textarea class="textarea" v-model="productForm.description"></textarea>
                  </div>
                </div>
                <div class="field d-flex  is-grouped">
                  <div class="control">
                    <button type="submit" class="button is-primary btn btn-primary m-2">{{ modalAction }}</button>
                  </div>
                  <div class="control">
                    <button class="button btn btn-primary m-2" @click="hideModal">Cancel</button>
                  </div>
                </div>
              </form>
            </div>
          </section>
        </div>
      </div>
    </div>

    <table class="table table-bordered">
      <thead>
        <tr>
          <th>ID</th>
          <th>Name</th>
          <th>Price</th>
          <th>Quantity</th>
          <th>Description</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="product in products" :key="product.id">
          <td>{{ product.id }}</td>
          <td>{{ product.name }}</td>
          <td>{{ product.price }}</td>
          <td>{{ product.quantity }}</td>
          <td>{{ product.description }}</td>
          <td>
            <button class="btn btn-warning me-2" @click="showEditModal(product)">Edit</button>
            <button class="btn btn-danger" @click="deleteProduct(product.id)">Delete</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import axios from 'axios';
import WebSocketService from '../services/websocket';

export default {
  data() {
    return {
      products: [],
      productForm: {
        id: null,
        name: '',
        price: null,
        quantity: null,
        description: ''
      },
      modalTitle: '',
      modalAction: '',
      isModalVisible: false,
    };
  },
  created() {
    this.fetchProducts();
    WebSocketService.connect('ws://localhost:3000', this.handleWebSocketMessage);
  },
  methods: {
    fetchProducts() {
      axios
        .get('http://localhost:3000/api/products')
        .then((response) => {
          this.products = response.data;
        })
        .catch((error) => {
          console.error('Error fetching products:', error);
        });
    },
    showAddModal() {
      this.modalTitle = 'Add Product';
      this.modalAction = 'Add';
      this.productForm = { id: null, name: '', price: null, quantity: null, description: '' };
      this.isModalVisible = true;
    },
    showEditModal(product) {
      this.modalTitle = 'Edit Product';
      this.modalAction = 'Save';
      this.productForm = { ...product };
      this.isModalVisible = true;
    },
    hideModal() {
      this.isModalVisible = false;
    },
    addOrUpdateProduct() {
      const { id, name, price, quantity, description } = this.productForm;
      if (id) {
        axios
          .put(`http://localhost:3000/api/products/${id}`, { name, price, quantity, description })
          .then((response) => {
            const updatedProductIndex = this.products.findIndex((p) => p.id === id);
            this.products[updatedProductIndex] = response.data;
            this.hideModal();
          })
          .catch((error) => {
            console.error('Error updating product:', error);
          });
      } else {
        axios
          .post('http://localhost:3000/api/products', { name, price, quantity, description })
          .then((response) => {
            // this.products.push(response.data);
            this.productForm = { id: null, name: '', price: null, quantity: null, description: '' }; // Reset form after adding
            this.hideModal();
          })
          .catch((error) => {
            console.error('Error adding product:', error);
          });
      }
    },
    deleteProduct(id) {
      axios
        .delete(`http://localhost:3000/api/products/${id}`)
        .then(() => {
          this.products = this.products.filter((product) => product.id !== id);
        })
        .catch((error) => {
          console.error('Error deleting product:', error);
        });
    },
    handleWebSocketMessage(message) {
      console.log('Received WebSocket message:', message);

      if (message.type === 'new_product') {
        // Check if the product already exists in the list
        const existingProduct = this.products.find((product) => product.id === message.product.id);
        if (!existingProduct) {
          this.products.push(message.product);
        }
      } else if (message.type === 'update_product') {
        const updatedProductIndex = this.products.findIndex((p) => p.id === message.product.id);
        this.products[updatedProductIndex] = message.product;
      } else if (message.type === 'delete_product') {
        this.products = this.products.filter((product) => product.id !== message.productId);
      }
    },
  },
  beforeUnmount() {
    WebSocketService.disconnect();
  },
};
</script>

<style scoped>
table {
  width: 100%;
  margin-top: 20px;
}

th,
td {
  text-align: center;
  vertical-align: middle;
}

/* Modal */
.modal {
  display: none;
}

.modal.is-active {
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal-background {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
}

.modal-content {
  background-color: #f5f5f5;
  border-radius: 5px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  max-width: 400px;
  width: 100%;
}

.card {
  border-radius: 5px;
}

.card-header {
  background-color: #f0f0f0;
  border-bottom: 1px solid #ccc;
}

.card-header-title {
  font-size: 20px;
  font-weight: bold;
}

.card-content {
  padding: 20px;
}

.delete {
  cursor: pointer;
}
</style>
