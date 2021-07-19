<template>
    <div class="page-product">
        <div class="columns is-multiline">
            <div class="column is-6">
                <figure class="image mb-6">
                    <img v-bind:src="product.get_image" />
                </figure>

            </div>

            <div class="column is-6">
                <h1 class="title">{{product.name}}</h1>


                <h2 class="subtitle my-4">Information</h2>

                <p><strong>Price: </strong>${{product.price}}</p>
                <div class="field has-addons mt-4">
                    <div class="control">
                        <input type="number" class="input" min="1" v-model="quantity" />
                    </div>

                    <div class="control">
                        <a class="button is-dark" @click="addToCart">Add to cart</a>
                    </div>
                </div>

                <p>{{product.description}}</p>
            </div>

            <div class="column is-12 box p-5">
                <h4 class="title">Reviews</h4>
                <h6 class="subtitle is-4">{{reviews.length}} reviews</h6>
                <div v-for="review in reviews" class="mb-3" v-bind:id="review.id">
                    <star-rating v-model:rating="review.rating"
                                 v-bind:star-size="30"
                                 v-bind:read-only="true"
                                 v-bind:show-rating="false"
                    ></star-rating>
                    <h4 class="subtitle is-6">{{ parseDate(review.date_added) }}</h4>
                    <p>{{review.review}}</p>
                    <hr/>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios'
import { toast } from 'bulma-toast'
import StarRating from 'vue-star-rating'
import dayjs from 'dayjs'

export default {
    name: 'Product',
    data(){
        return {
            quantity: 1,
            product: {},
            reviews: []
        }
    },
    components: {
        StarRating
    },
    async mounted(){
        this.getProduct()
    },
    methods: {
        async getProduct(){
            this.$store.commit('setIsLoading', true)
            const category_slug = this.$route.params.category_slug
            const product_slug = this.$route.params.product_slug

            await axios.get(`/api/v1/products/${category_slug}/${product_slug}`)
                 .then(response => {
                     this.product = response.data
                     document.title = this.product.name + ' | Djackets'
                 }).catch(error => {
                     console.error(error)
                 })

            await this.getProductReviews(category_slug, product_slug)
            this.$store.commit('setIsLoading', false)
        },
        async getProductReviews(category_slug, product_slug){
            await axios.get(`/api/v1/products/${category_slug}/${product_slug}/reviews/`).then(response => {
                this.reviews = response.data
            })
        },
        addToCart(){
            if(isNaN(this.quantity) || this.quantity < 1)
                this.quantity = 1

            const item = {
                product: this.product,
                quantity: this.quantity
            }

            this.$store.commit('addToCart', item)
            toast({
                message: 'The product was added to the cart',
                type: 'is-success',
                dismissible: true,
                pauseOnHover: true,
                duration: 2000,
                position: 'bottom-right'
            })
        },
        parseDate(date){
            return dayjs(date).format('MMM D, hh:mm A')
        }
    }
}
</script>
