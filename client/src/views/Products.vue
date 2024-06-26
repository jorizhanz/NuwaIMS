<template>
    <div class="products-page-container">
      <main class="products-page">
        <header class="header">
          <h1 class="header-title">Products</h1>
          <p class="details"> Create, Update, Delete, Edit products on sale.</p>
            <div class="search-container">
                <div class="column-select-container">
                    <select v-model="selectedColumn" class="column-select">
                    <option value="">All Columns</option>
                    <option value="product_name">Product Name</option>
                    <option value="product_brand">Brand</option>
                    <option value="category_label">Category</option>
                    <option value="subcategory_label">Sub Category</option>
                    <!-- Add more options for other columns -->
                    </select>
                </div>
                <div class="search-input-container">
                    <input
                    type="text"
                    v-model="search"
                    @input="fetchProducts"
                    placeholder="Search"
                    class="search-input"
                    />
                </div>
                <button @click="fetchProducts" class="search-button">
                <span class="material-icons">search</span>
                </button>
                <button @click="createProductModal" class="create-button">
                <span class="material-icons">add</span>
                <span class="create-product">Add Product</span>
                </button>
            </div>
        </header>
        <div class="product-table-container">
        <table>
          <thead>
            <tr>
              <th @click="sortBy('product_name')" :class="{ 'sortable': true, 'sorted': sortKey === 'product_name' }">
                  Product Name
                  <span v-if="sortKey === 'product_name'" class="material-icons">
                    {{ sortOrder === 'asc' ? 'arrow_drop_up' : 'arrow_drop_down' }}
                  </span>
                </th>
                <th @click="sortBy('product_brand')" :class="{ 'sortable': true, 'sorted': sortKey === 'product_brand' }">
                  Brand
                  <span v-if="sortKey === 'product_brand'" class="material-icons">
                    {{ sortOrder === 'asc' ? 'arrow_drop_up' : 'arrow_drop_down' }}
                  </span>
                </th>
                <th @click="sortBy('category_label')" :class="{ 'sortable': true, 'sorted': sortKey === 'category_label' }">
                  Category
                  <span v-if="sortKey === 'category_label'" class="material-icons">
                    {{ sortOrder === 'asc' ? 'arrow_drop_up' : 'arrow_drop_down' }}
                  </span>
                </th>
                <th @click="sortBy('subcategory_label')" :class="{ 'sortable': true, 'sorted': sortKey === 'subcategory_label' }">
                  Sub Category
                  <span v-if="sortKey === 'subcategory_label'" class="material-icons">
                    {{ sortOrder === 'asc' ? 'arrow_drop_up' : 'arrow_drop_down' }}
                  </span>
                </th>
                <th @click="sortBy('price')" :class="{ 'sortable': true, 'sorted': sortKey === 'price' }">
                  Price
                  <span v-if="sortKey === 'tier_access'" class="material-icons">
                    {{ sortOrder === 'asc' ? 'arrow_drop_up' : 'arrow_drop_down' }}
                  </span>
                </th>
                <th @click="sortBy('created_dt')" :class="{ 'sortable': true, 'sorted': sortKey === 'created_dt' }">
                  Created
                  <span v-if="sortKey === 'created_dt'" class="material-icons">
                    {{ sortOrder === 'asc' ? 'arrow_drop_up' : 'arrow_drop_down' }}
                  </span>
                </th>
                <th @click="sortBy('last_modified_dt')" :class="{ 'sortable': true, 'sorted': sortKey === 'last_modified_dt' }">
                  Last Modified
                  <span v-if="sortKey === 'last_modified_dt'" class="material-icons">
                    {{ sortOrder === 'asc' ? 'arrow_drop_up' : 'arrow_drop_down' }}
                  </span>
                </th>
              <th>Action</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="product in sortedProducts" :key="product.product_id" @dblclick="viewProductModal(product)" class="clickable-row">
              <td>{{ product.product_name }}</td>
              <td>{{ product.product_brand }}</td>
              <td>{{ product.category_label }}</td>
              <td>{{ product.subcategory_label }}</td>
              <td>{{ product.product_price }}</td>
              <td>{{ product.created_dt }}</td>
              <td>{{ product.last_modified_dt }}</td>
              <td>
                <div class="row-actions">
                  <button @click="updateProductModal(product)" class="icon-button">
                    <span class="material-icons">edit</span>
                  </button>
                  <button @click="confirmDeleteProduct(product)" class="icon-button">
                    <span class="material-icons">delete</span>
                  </button>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
      
      </main>
    </div>
    <Modal :show="isModalOpen" :mode="modalMode" :product="selectedProduct" @create="createProduct" @update="updateProduct" @close="closeModal" />
  
    <v-dialog v-model="isDeleteDialogOpen" max-width="500">
      <v-card>
        <v-card-title class="headline">Are you sure to delete this?</v-card-title>
        <v-card-actions class="justify-center">
          <v-spacer></v-spacer>
          <v-btn color="red" @click="deleteProduct">Yes</v-btn>
          <v-btn color="grey" @click="isDeleteDialogOpen = false">No</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    
    <v-snackbar
      v-model="isSnackBarOpen"
      timeout=3000
      class="text-center"
    >
      {{ snackBarText }}
      <template v-slot:actions>
        <v-btn
          color="blue"
          variant="text"
          @click="isSnackBarOpen = false"
        >
          Close
        </v-btn>
      </template>
    </v-snackbar>
  </template>
  
  <script>
  import Modal from '@/components/ProductModal.vue';
  import ProductService from '@/services/product.service';
  
  export default {
    components: {
      Modal
    },
    name: "Product",
    data() {
      return {
        products: [],
        selectedColumn: '',
        search: '',
        sortKey: '',
        sortOrder: 'asc',
        isModalOpen: false,
        selectedProduct: null,
        modalMode: 'view',
        isDeleteDialogOpen: false,
        productToDelete: null,
        isSnackBarOpen:false,
        snackBarText:''
      };
    },
    created() {
        this.fetchProducts();
    },
    methods: {
      async fetchProducts() {
            try {
                const queryParams = {};

                if (this.selectedColumn) {
                    // If a specific column is selected, use it for search
                    queryParams[this.selectedColumn] = this.search;
                } else {
                    // If no specific column is selected, apply search to all columns
                    queryParams['all_search'] = this.search;
                }
                    
                const response = await ProductService.findMany(queryParams);

                this.products = response.data;
            } catch (error) {
                console.error('Error fetching products:', error);
            }
      },
      updateProductModal(product) {
        this.selectedProduct = product;
        this.modalMode = 'update';
        this.isModalOpen = true;
      },
  
      viewProductModal(product) {
        this.selectedProduct = product;
        this.modalMode = 'view';
        this.isModalOpen = true;
      },
  
      createProductModal(product) {
        this.selectedProduct = product;
        this.modalMode = 'create';
        this.isModalOpen = true;
      },
  
      confirmDeleteProduct(product) {
        this.productToDelete = product;
        this.isDeleteDialogOpen = true;
      },
  
      showSnackBar(text){
        this.snackBarText = text;
        this.isSnackBarOpen = true
      },
  
      async updateProduct(newProductForm) {
        try {
          
          const response = await ProductService.updateProduct(newProductForm);
          console.log(response);
          this.showSnackBar('Product successfully updated!');
        } catch(err) {
          console.error(err);
          this.showSnackBar('Product update failed! Something went wrong');
        }
        console.log("UPDATE!");
        
        this.closeModal()
      },
  
      async createProduct(newProductForm) {
        try{
          const response = await ProductService.createProduct(newProductForm);
          console.log(response);
          this.showSnackBar("Product successfully created!");
        } catch(err) {
          console.error(err);
          this.showSnackBar('Product creation failed! Something went wrong');
        }
        await this.fetchProducts();
        this.closeModal()
      },
      
      async deleteProduct() {
        try {
          response = await ProductService.deleteProduct(this.productToDelete.product_id);
          console.log(response);
          this.showSnackBar("Product successfully deleted!");
        } catch (error) {
          console.error('Error deleting product:', error);
          this.showSnackBar("Delete failed! Something went wrong.");
        }
        await this.fetchProducts()
        this.isDeleteDialogOpen = false;
      },
      sortBy(key) {
        if (this.sortKey == key) {
          this.sortOrder = this.sortOrder === 'asc' ? 'desc' : 'asc';
        } else {
          this.sortKey = key;
          this.sortOrder = 'asc';
        }
      },
      async closeModal() {
        await this.fetchProducts();
        this.isModalOpen = false;
        this.selectedProduct = null;
        this.modalMode = 'view'; // Reset mode to 'view' after closing modal
      },
    },
    computed: {
      sortedProducts() {
        return this.products.sort((a,b) => {
          let modifier = 1;
          if (this.sortOrder === 'desc') modifier = -1;
          if (a[this.sortKey] < b[this.sortKey]) return -1 * modifier;
          if (a[this.sortKey] > b[this.sortKey]) return 1 * modifier;
        });
      }
    }
  };
  </script>
  
  <style>
  html, body {
    height: 100%;
    margin: 0;
  }
  
  .products-page-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: flex-start;
    height: 100vh;
    width: 100%;
    padding: 20px;
    box-sizing: border-box;
    background-color: #f9f9f9;
  }
  
  .products-page {
    text-align: center;
    margin-bottom: 20px;
    width: 100%;
  }
  
  .header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    width: 100%;
    margin-bottom: 10px;
  }
  
  .header-title {
    margin: 0;
    vertical-align: middle;
  }
  
  .search-container {
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
    margin: 0;
    justify-content: center;
  }
  
  .search-input {
    padding: 8px;
    border: 1px solid #ddd;
    border-radius: 4px;
    flex: 1;
    min-width: 300px
  }

  .column-select-container {
    margin-right: 10px;
    }

  .column-select {
    padding: 8px;
    border: 1px solid #ddd;
    border-radius: 4px;
    box-shadow: rgba(14, 63, 126, 0.06) 0px 0px 0px 1px, rgba(42, 51, 70, 0.03) 0px 1px 1px -0.5px, rgba(42, 51, 70, 0.04) 0px 2px 2px -1px, rgba(42, 51, 70, 0.04) 0px 3px 3px -1.5px, rgba(42, 51, 70, 0.03) 0px 5px 5px -2.5px, rgba(42, 51, 70, 0.03) 0px 10px 10px -5px, rgba(42, 51, 70, 0.03) 0px 24px 24px -8px;
  }
  
  .create-button {
    padding: 8px 12px;
    border: 1px solid #ddd;
    border-radius: 4px;
    background-color: #F8C963;
    cursor: pointer;
    display: block;
    position: relative;
    font-size: 16px;
    letter-spacing: 2px;
    text-transform: uppercase;
    box-shadow: 0 6px #ffbf5e;
    color: white;
  }
  
  .create-button:hover {
    box-shadow: 0 4px #fbae33;
    top: 2px;
  }
  .create-button:active {
    box-shadow: none;
    top: 6px;
  }
  
  .create-product {
    vertical-align: middle;
    margin-left: 5px;
  }
  
  .sort-select {
    padding: 8px;
    border: 1px solid #ddd;
    border-radius: 4px;
  }
  
  .product-table-container {
    width: 100%;
    overflow-x: auto;
  }
  
  table {
    width: 100%;
    border-collapse: collapse;
    background-color: #ffffff;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  }
  
  th, td {
    border: 1px solid #ddd;
    padding: 12px 15px;
    text-align: left;
  }
  
  th {
    background-color: #F8C963;
    color: white;
  }
  
  tr:nth-child(even) {
    background-color: #f2f2f2;
  }
  
  .sortable {
    cursor: pointer;
  }
  
  .sorted {
    color: #f2f2f2;
  }
  
  th:hover {
    background-color: #f2f2f2;
    color: #F8C963;
  }
  
  
  tr:hover {
    background-color: #ddd;
  }
  
  .icon-button {
    background: none;
    border: none;
    cursor: pointer;
    padding: 0;
    color: #311f19;
  }
  
  .material-icons {
    font-size: 18px;
    vertical-align: middle;
  }
  
  .icon-button .material-icons {
    font-size: 24px;
    vertical-align: middle;
  }
  
  .icon-button:hover .material-icons {
    color: #4CAF50;
  transform: scale(1.2);
}
  
  .clickable-row {
    cursor: pointer;
  }
  
  .clickable-row:hover {
    background-color: #FFE7AD;
  }
  
  .row-actions {
    display: flex;
    align-items: center;
    justify-content: space-between;
  }
  </style>
  