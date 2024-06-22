<template>
  <main>

    <!-- ERROR -->
    <div v-show="status === 'error'">
      <div class="error-message">An error occurred during payment. Please try again.</div>
    </div>

    <form v-if="status === 'ready' || status === 'error'" class="checkout-form" id="checkout-form"
      v-on:submit.prevent="pay">
      <fieldset>
        <legend>Contact information</legend>
        <div class="grid">
          <label for="email">Email</label>
          <input type="email" name="email" id="email" v-model="customer.email" required />
        </div>
      </fieldset>

      <fieldset>
        <legend>Shipping address</legend>
        <div class="grid">
          <label for="name">Name</label>
          <input type="text" name="name" id="name" v-model="customer.shipping_address.name" required />

          <label for="address">Address</label>
          <input type="text" name="address" id="address" v-model="customer.shipping_address.address" required />

          <label for="city">City</label>
          <input type="text" name="city" id="city" v-model="customer.shipping_address.city" required />

          <label for="country">Country</label>
          <select name="country" id="country" v-model="customer.shipping_address.country">

            <option v-for="destination in destinations" :key="destination[1].isoCode" :value="destination[1].isoCode">
              {{ destination[1].displayName }}
            </option>

          </select>

          <label for="postalcode">Postal code</label>
          <input type="text" name="postalcode" id="postalcode" v-model="customer.shipping_address.postal_code"
            required />

          <label for="phone">Phone (optional)</label>
          <input type="tel" name="phone" id="phone" v-model="customer.shipping_address.phone"
            placeholder="+43 123456789" />
        </div>
      </fieldset>

      <fieldset>
        <legend>Card details</legend>
        <div class="grid">
          <label for="cardholder">Name on card</label>
          <input type="text" name="cardholder" id="cardholder" v-model="card.cardholder" required />

          <label for="cardnumber">Card number</label>
          <input type="text" name="cardnumber" id="cardnumber" v-model="card.cardnumber" required />

          <label for="cardexpiry">Expiration</label>
          <input type="text" name="cardexpiry" id="cardexpiry" v-model="cardexpiry" pattern="\d{2}/\d{4}"
            placeholder="MM/YYYY" required />

          <label for="cardcvc">CVC</label>
          <input name="cardcvc" id="cardcvc" v-model.number="card.cvc" type="text" pattern="\d{3}" required />
        </div>
      </fieldset>

      <div>
        <div>
          Subtotal: €
          <span id="price-subtotal">{{ subtotal }}</span>
        </div>
        <div>
          Shipping Costs:
          <span id="price-shipping" :style="{ fontWeight: (!isShippingFree ? 'normal' : 'bold') }">
            {{ shippingCostText }}
          </span>
        </div>
        <div v-show="possibleFreeShipping && !isShippingFree" id="free-shipping-from">(Free shipping from: €
          <!--TODO: only display 'free-shipping-from' if free shipping is possible and the threshold is not yet reached-->
          <span id="free-shipping-threshold">{{ freeShippingThreshold }}</span>)
        </div>
      </div>

      <div>
        <div class="checkout-total">
          Total: €
          <span id="price-total">{{ total }}</span>
        </div>
      </div>

      <div class="button-row">
        <router-link to="/cart">&larr; Back to Cart</router-link>
        <button type="submit" id="pay-button" v-on:click="pay">Pay</button>
      </div>
    </form>


    <!-- PROCESSING-->
    <div v-show="status === 'processing'">
      <h2>Processing payment...</h2>
      <img src="@/assets/images/spinner.gif" width="50" height="50" />
    </div>

    <!-- SUCCESS-->
    <div v-show="status === 'success'">
      <div>Your payment was completed successfully.</div>
      <h2>Thank you for your purchase!</h2>
      <div>
        <router-link to="/search">&larr; Back to Search</router-link>
      </div>
    </div>


  </main>
</template>

<script>
import { mapStores } from 'pinia';
import { useArtmartStore } from '@/store';
import * as ArtmartService from '@/services/ArtmartService';
import * as BlingService from '@/services/BlingService';




export default {
  name: "CheckoutPage",
  data: function () {
    return {
      status: "ready",
      customer: {
        email: "",
        shipping_address: {
          name: "",
          address: "",
          city: "",
          country: "AT",
          postal_code: "",
          phone: "",
        },
      },
      card: {
        cardholder: "",
        cardnumber: "",
        exp_month: "",
        exp_year: "",
        cvc: null,
      }
      
    };
  },

  //CUIDADO A PARTIR DE AQUÍ ****************
  computed: {
    
    artmartStore: mapStores(useArtmartStore).artmartStore,
    destinations() {
      return this.artmartStore.destinations;
    },
    cartIsEmpty() {
      return this.artmartStore.cartIsEmpty;
    },
    subtotal() {
      return (this.artmartStore.cartTotal / 100).toFixed(2);
    },
    //Calculate if there's free shipping
    isShippingFree() {
      const country = this.customer.shipping_address.country;
      const destination = this.destinations.get(country);
      return destination && destination.freeShippingPossible && this.artmartStore.cartTotal >= destination.freeShippingThreshold;

    },
    shippingCost() {

      if (this.isShippingFree) {
        return 0;
      }

      const country = this.customer.shipping_address.country;
      const destination = this.destinations.get(country);

      return destination ? destination.price / 100 : 0;
    },
    shippingCostText() {
      const cost = this.shippingCost;
      return cost == 0 ? 'Free' : `€ ${(cost).toFixed(2)}`;

    },

    possibleFreeShipping() {
      const country = this.customer.shipping_address.country;
      return this.destinations.get(country)?.freeShippingPossible;
    },

    freeShippingThreshold() {
      const country = this.customer.shipping_address.country;
      const destination = this.destinations.get(country);
      if (!destination || !destination.freeShippingThreshold) {
        console.error(`No destination found for country: ${country}`);
        return 0;
      }      
      return destination.freeShippingThreshold / 100;
    },
    total() {
      return (parseFloat(this.subtotal) + parseFloat(this.shippingCost)).toFixed(2);
    }
  },
  mounted() {
    if (this.cartIsEmpty) {
      this.$router.replace('/cart');
    }
  },
  methods: {
    async pay() {
      
      this.status = 'processing';
      try {
        this.processCardExpiry();
        const artmartResponse = await ArtmartService.checkout({
          email: this.customer.email,
          shipping_address: this.customer.shipping_address,
        });
        
        if (artmartResponse) {
          
          const { payment_intent_id: paymentIntentId, client_secret: clientSecret } = artmartResponse;
          console.log("Variables: ", paymentIntentId, clientSecret);
          const blingResponse = await BlingService.confirmPaymentIntent(paymentIntentId, clientSecret, this.card);

          if (blingResponse) {
            this.status = 'success';
            this.$router.replace('/confirm');
          } else {
            this.status = 'error';
            console.error('Payment failed at Bling');
          }
        } else {
          this.status = 'error';
          console.error('Payment failed at Artmart');
        }
      } catch (error) {
        this.status = 'error';
        console.error('Payment error:', error);
      }
    },

    processCardExpiry() {
    if (this.cardexpiry) {
      const [month, year] = this.cardexpiry.split('/');
      if (month && year) {
        this.card.exp_month = parseInt(month);
        this.card.exp_year = parseInt(year);
      } else {
        this.card.exp_month = "";
        this.card.exp_year = "";
      }
    }
  }
  },
  watch: {
    'customer.shipping_address.country': function (newVal) {
      console.log('Country changed:', newVal);
      this.$nextTick(() => {
        console.log('Subtotal:', this.subtotal);
        console.log('Shipping Cost:', this.shippingCostText);
        console.log('Total:', this.total);
      });
    }
  }
};


</script>

<style scoped>
.error-message {
  color: red;
}

.checkout-form>div {
  margin: 1rem 0;
  text-align: right;
}

/* this is a workaround for a Chrome bug that disallows display:grid on fieldset elements */
.checkout-form div.grid {
  display: grid;
  grid-template-columns: 1fr 300px;
  grid-gap: 0.5em 1em;
  align-items: center;
}

.checkout-form fieldset {
  border: none;
  margin: 2rem 0;
  padding: 0;
}

.checkout-form fieldset legend {
  font-weight: bold;
  font-size: 1.5em;
  margin-bottom: 0.5rem;
}

.checkout-form input {
  -moz-appearance: textfield;
  font-family: inherit;
  font-size: 1em;
  height: 1.25rem;
  line-height: 1.25rem;
  padding: 3px;
  text-indent: 1.25px;
  border: 1px solid rgba(0, 0, 0, 0.1);
}

.checkout-total {
  font-size: 1.5rem;
  font-weight: bold;
}

.checkout-form .button-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

#free-shipping-from {
  font-size: 0.65em;
}

@media (max-width: 600px) {
  .checkout-form {
    width: 100%;
  }

  .checkout-form label {
    margin-bottom: -0.25em;
    margin-top: 0.25em;
  }

  .checkout-form input {
    margin: 0;
  }

  .checkout-form select {
    width: 100%;
  }

  .checkout-form div.grid {
    grid-template-columns: 1fr;
  }

  .checkout-form .button-row {
    flex-direction: column-reverse;
    align-items: flex-start;
  }

  .checkout-form .button-row button {
    width: 100%;
    margin-bottom: 1em;
  }
}
</style>
