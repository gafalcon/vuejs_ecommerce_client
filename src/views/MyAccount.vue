<template>
    <div class="page-my-account">
        <div class="columns is-multiline">
            <div class="colum is-12">
                <h1 class="title">My account</h1>
            </div>

            <div class="column is-12">
                <button class="button is-danger" @click="logout()">Log out</button>
            </div>

            <hr/>

            <div class="column is-12">
                <h2 class="subtitle">My orders</h2>

                <OrderSummary
                    v-for="order in orders"
                    v-bind:key="order.id"
                    v-bind:order="order"
                />

            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios'
import OrderSummary from '../components/OrderSummary.vue'
export default {
    name: 'MyAccount',
    components: {OrderSummary},
    data(){
        return {
            orders: []
        }
    },
    mounted(){
        this.getOrders()
    },
    methods: {
        logout(){
            axios.defaults.headers.common['Authorization'] = ""

            localStorage.removeItem('token')
            localStorage.removeItem('username')
            localStorage.removeItem('userid')

            this.$store.commit('removeToken')

            this.$router.push("/")
        },
        async getOrders(){
            this.$store.commit('setIsLoading', true)
            await axios.get('/api/v1/orders/')
                 .then(response => {
                     this.orders = response.data
                 })
                .catch(err => {
                    console.log(err)
                })

            this.$store.commit('setIsLoading', false)
        }
    }
}
</script>
