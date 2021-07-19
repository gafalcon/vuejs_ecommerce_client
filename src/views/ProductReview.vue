<template>
    <div class="page-product-review">
        <h1 class="title">Product Review</h1>
    </div>
    <div class="columns">
        <ProductBox v-if="product !== {}" v-bind:product="product" />
        <div class="column">
            <h2 class="title">Write a review</h2>
            <star-rating
                v-model:rating="rating"
                v-bind:show-rating="false"
                v-bind:star-size="30"
            ></star-rating>
            <div class="field">
                <label class="label">Review</label>
                <div class="control">
                    <textarea class="textarea" placeholder="Textarea" v-model="review"></textarea>
                </div>
            </div>
            <div class="field">
                <div class="control">
                    <button class="button is-link" @click="submit()">Submit</button>
                </div>
            </div>

            <div class="notification is-danger mt-4" v-if="errors.length">
                <p v-for="error in errors" v-bind:key="error">{{ error }}</p>
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios'
import {toast} from 'bulma-toast'
import ProductBox from '../components/ProductBox.vue'
import StarRating from 'vue-star-rating'

export default {
    name: 'ProductReview',
    components: {
      ProductBox, StarRating
    },
    data(){
        return {
            product: {id:0, get_absolute_url:''},
            review: '',
            rating: 0,
            errors: []
        }
    },
    mounted(){
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

            this.$store.commit('setIsLoading', false)
        },
        async submit(){
            this.errors = []
            if (this.rating <= 0)
                this.errors.push('Rating cannot be empty!')

            this.$store.commit('setIsLoading', true)
            const category_slug = this.$route.params.category_slug
            const product_slug = this.$route.params.product_slug
            await axios.post(`/api/v1/products/${category_slug}/${product_slug}/reviews/`,
                             {rating: this.rating, review: this.review}
            ).then(response => {
                toast({
                    message: 'Review Submited!',
                    type: 'is-success',
                    dismissible: true,
                    pauseOnHover: true,
                    duration: 2000,
                    position: 'bottom-right'
                })
                this.$router.push(`/${category_slug}/${product_slug}`)
            }).catch(error => {
                console.log(error)
            })

            this.$store.commit('setIsLoading', false)
        }
    }

}
</script>

<style>

</style>
