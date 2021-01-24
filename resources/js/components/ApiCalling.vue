<template>
    <div class="api-calling container mt-5">
        <h1>Create Product</h1>
        <transition name="fade">
            <div class="alert alert-danger alert-dismissible" role="alert" v-if="error">
                <b>{{ error.message }}</b>
                <ul>
                    <li v-for="(errorName, index) in error.errors" :key="index">
                        {{ errorName[0] }}
                    </li>
                </ul>
                <button type="button" class="close" @click="error = null">
                    <span aria-hidden="true">&times;</span>
                </button>
            </div>
        </transition>
        
        <div class="form-group">
            <label>Name</label>
            <input v-model="product.name" type="text" class="form-control" placeholder="Name...">
        </div>
        <div class="form-group">
            <label>Price</label>
            <input v-model="product.price" type="text" class="form-control" placeholder="Price...">
        </div>
        <button class="btn btn-primary" @click="createProduct">Create</button>

        <hr>
        <h1>List Products</h1>
        <table class="table">
            <thead>
                <tr>
                    <th scope="col">ID</th>
                    <th scope="col">Name</th>
                    <th scope="col">Price</th>
                    <th scope="col">Actions</th>
                </tr>
            </thead>
            <transition-group name="slide-fade" tag="tbody">
                <tr v-for="(product, index) in listProducts" :key="product.id">
                    <th scope="row">{{ product.id }}</th>
                    <td v-if="!product.isEdit">
                        {{ product.name }}
                    </td>
                    <td v-else>
                        <input type="text" v-model="selectedProduct.name" class="form-control">
                    </td>
                    <td v-if="!product.isEdit">
                        {{ product.price }}
                    </td>
                    <td v-else>
                        <input type="text" v-model="selectedProduct.price" class="form-control">
                    </td>
                    <td v-if="!product.isEdit">
                        <button class="btn btn-primary" @click="selecteProduct(product)">Edit</button>
                        <button class="btn btn-danger" @click="deleteProduct(product, index)">Delete</button>
                    </td>
                    <td v-else>
                        <button class="btn btn-primary" @click="updateProduct(index)">Save</button>
                        <button class="btn btn-danger" @click="product.isEdit = false">Cancel</button>
                    </td>
                </tr>
            </transition-group>
        </table>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                product: {
                    name: '',
                    price: 0
                },
                listProducts: [],
                error: null,
                selectedProduct: null
            }
        },
        created() {
            this.getListProducts()
        },
        methods: {
            async createProduct() {
                try {
                    this.error = null
                    const response = await axios.post('/products', {
                        name: this.product.name,
                        price: this.product.price
                    })
                    this.listProducts.unshift({
                        ...response.data.product,
                        isEdit: false
                    })

                    // reset giá trị form về ban đầu
                    this.product = {
                        name: '',
                        price: 0
                    }
                } catch (error) {
                    this.error = error.response.data
                }
            },
            async getListProducts() {
                try {
                    const response = await axios.get('/products')
                    this.listProducts = response.data
                    this.listProducts.forEach(item => {
                        Vue.set(item, 'isEdit', false)
                    })
                } catch (error) {
                    this.error = error.response.data
                }
           },
           selecteProduct (product) {
                product.isEdit = true
				this.selectedProduct = { ...product }
			},
			async updateProduct(index) {
                try {
                    const response = await axios.put('/products/' + this.selectedProduct.id, {
                        name: this.selectedProduct.name,
                        price: this.selectedProduct.price
                    })
                  
                    this.listProducts[index].name = response.data.product.name
                    this.listProducts[index].price = response.data.product.price
                    this.listProducts[index].isEdit = false
                } catch (error) {
                    this.error = error.response.data
                }
            },
            async deleteProduct(product, index) {
                try {
                    const response = await axios.delete('/products/' + product.id)
                    this.listProducts.splice(index, 1)
                } catch (error) {
                    this.error = error.response.data
                }
            }
        }
    }
</script>

<style lang="scss" scoped>
.fade-enter-active, .fade-leave-active {
  transition: opacity .5s;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}

.slide-fade-enter-active {
  transition: all .3s ease;
}
.slide-fade-leave-active {
  transition: all .8s cubic-bezier(1.0, 0.5, 0.8, 1.0);
}
.slide-fade-enter, .slide-fade-leave-to {
  transform: translateX(10px);
  opacity: 0;
}
</style>