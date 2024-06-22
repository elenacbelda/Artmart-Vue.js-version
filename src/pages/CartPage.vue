<template>
  <main>
    <section class="cart">
      <div v-if="cartItems.length === 0">
        <p>There are no items in your shopping cart.</p>
      </div>
      <div v-else>
        <cart-item
          v-for="item in cartItems"
          :key="item.cartItemId"
          :cart-item="item"
          class="cart-item"
        />
      <div class="cart-total">
        <div class="price">Total: {{ finalPrice }}</div>
        <router-link to="/checkout" class="cart-checkout">Checkout</router-link>
        </div>
      </div>
    </section>
  </main>
</template>

<script>

import { computed , watchEffect} from 'vue';
import { useArtmartStore } from '@/store';
import CartItem from '@/components/CartItem.vue';



export default {
  name: "CartPage",
  components: {
    CartItem
  },
  setup() {
    const artmartStore = useArtmartStore();
    const cartItems = computed(() => artmartStore.cart);
    const totalPrice = computed(() => artmartStore.cart.reduce((total, item) => total + item.price, 0));
  
    const finalPrice = computed(() =>
      new Intl.NumberFormat('en-US', {
        style: 'currency',
        currency: 'EUR',
      }).format(totalPrice.value / 100).replace('€', '€ ')
    );

    return {
      cartItems,
      finalPrice
    };
  },
  
  
};
</script>

<style scoped>
.cart {
  display: flex;
  flex-direction: column;
}

.cart-total {
  align-self: flex-end;
  font-size: 1.5rem;
  font-weight: bold;
  margin: 1rem;
  text-align: right;
}

.cart-checkout {
  align-self: flex-end;
  margin-top: 1em;
  font-size: 1rem;
}

@media (max-width: 600px) {
  .cart-total {
    align-self: stretch;
    text-align: center;
  }

  .cart-checkout {
    align-self: stretch;
    width: 100%;
  }
}
</style>
