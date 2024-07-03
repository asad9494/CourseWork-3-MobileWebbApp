<template>
   <section>
    <div class="container">                                         
      <div v-if="cart.length">
    <div v-for="item in groupedCart" :key="item.id" class="mb-4">
        <div class="card shadow-sm">
            <div class="card-body">
                <div class="d-flex justify-content-between align-items-center">
                    <div class="d-flex flex-row align-items-center">
                        <img
                            :src="ServerImage(item.picture)"
                            class="img-fluid rounded-3"
                            alt="Shopping item"
                            style="width: 65px; height: 65px; object-fit: cover;"
                        >
                        <div class="ms-3">
                            <h5 class="mb-0">{{ item.subject }}</h5>
                        </div>
                        <div class="ms-3 ">
                                                                            <h5> {{ item.quantity }}</h5>
                                                                        </div>
                    </div>
                    <div class="d-flex flex-row align-items-center">
                        <div class="me-3" style="width: 50px;">
                            <h5 class="fw-normal mb-0">{{ CartCount(item) }}</h5>
                        </div>
                        <div class="me-3" style="width: 80px;">
                            <h5 class="mb-0">{{ itemTotalPrice(item) }} AED</h5>
                        </div>
                        <button class="btn btn-danger" v-on:click="removeFromCart(item.id)">
                            Remove
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <div class="d-flex justify-content-between text-black h4 mb-4">
        <p class="mb-2">Total:</p>
        <p class="mb-2">AED {{ cartTotal }}</p>
    </div>

    <div class="col-lg-12">
        <div class="text-white rounded-3">
            <div class="card-body">
                <div class="d-flex justify-content-between align-items-center text-black">
                    <h5 class="mb-0">User details</h5>
                </div>
                <p v-if="!isFormValid" class="text-danger">Please fill in all fields.</p>
                <form class="mt-2">
                    <div class="row">
                        <div class="col-12 col-md-6 mb-4">
                            <label class="form-label text-black" for="nameInput">Name</label>
                            <input
                                type="text"
                                id="nameInput"
                                class="form-control"
                                placeholder="Name"
                                v-model.trim="order.firstName"
                                required
                            >
                            <span id="nameMsg" class="text-danger"></span>
                        </div>
                        <div class="col-12 col-md-6 mb-4">
                            <label class="form-label text-black" for="phoneInput">Phone Number</label>
                            <input
                                type="number"
                                id="phoneInput"
                                class="form-control"
                                v-model.number="order.phone"
                                required
                                placeholder="Phone Number"
                            >
                            <span id="phoneMsg" class="text-danger"></span>
                        </div>
                    </div>
                </form>
                <div class="d-grid gap-2">
                    <button
                        :disabled="!isFormValid"
                        type="button"
                        data-bs-toggle="modal"
                        data-bs-target="#placeordermodal"
                        class="btn btn-primary btn-block"
                        v-on:click="ProcessOrder"
                    >
                        CheckOut
                    </button>
                </div>
            </div>
        </div>
    </div>
</div>
<div v-else>
    <p>Your cart is empty.</p>
</div>

<!-- Order confirmation modal -->
<div
    class="modal fade"
    id="placeordermodal"
    tabindex="-1"
    aria-labelledby="placeordermodalLabel"
    aria-hidden="true"
>
    <div class="modal-dialog">
        <div class="modal-content">
            <div class="modal-header">
                <h5 class="modal-title" id="placeordermodalLabel">Order Confirmation</h5>
                <button
                    type="button"
                    class="btn-close"
                    data-bs-dismiss="modal"
                    aria-label="Close"
                ></button>
            </div>
            <div class="modal-body">Order Confirmed</div>
            <div class="modal-footer">
                <button
                    type="button"
                    class="btn btn-primary"
                    data-bs-dismiss="modal"
                    v-on:click="showhomePageBtn"
                >
                    Close
                </button>
            </div>
        </div>
    </div>
</div>

                                    </div>
                                </section>
</template>

<script>
export default {
  name: "CheckoutForm",
  props: {
    cart: { type: Array, required: true },
    saveOrder: { type: Function, required: true },
    showhomePageBtn: {
      type: Function,
      required: true
    },
 
  },
  data() {
    return {
      order: {
        firstName: '',
        phone: ''
      },
      isFormValid: false
    }
  },
  watch: {
    order: {
      handler() {
        this.validateForm();
      },
      deep: true
    }
  },
  computed: {
    groupedCart() {
  let grouped = {};
  this.cart.forEach(item => {
    if (!grouped[item.id]) {
      grouped[item.id] = { ...item, quantity: 0 };
    }
    grouped[item.id].quantity += 1;

  });
      console.log("🚀 ~this.cart:", Object.values(grouped));
      return Object.values(grouped);
},
    cartTotal() {
      return this.groupedCart.reduce((sum, item) => sum + item.price * item.quantity, 0).toFixed(2);
    },
    CartItemCount() {
      return this.cart.length;
    },
  },
  methods: {
ServerImage(img) {
      // Construct the full URL for an image
      const NodeServerUrl = "https://coursework-2-after-school.onrender.com";
      const Image = img.split("/").pop().trim();
      const FullPath = NodeServerUrl + "/" + Image;
      return FullPath;
    },
    validateForm() {
      this.isFormValid = !!this.order.firstName && !!this.order.phone;
    },
    submitOrder() {
      if (this.isFormValid) {
 this.order.lessonItems = this.groupedCart.map(item => ({
      id: item._id,
      quantity: item.quantity
 }));
        this.saveOrder(this.order);
      }
    },
    CartCount(product) {
      this.$emit('cart-count', product);
    },
    removeFromCart(lessonId) {
      this.$emit('remove-item-from-cart', lessonId);
    },
    itemTotalPrice(item) {
      return (item.price * item.quantity).toFixed(2);
    },
    getImageUrl(item) {
      return item.picture || 'path/to/your/fallback/image.png'; // Provide a fallback image path
    },
    
  },

};
</script>

<style scoped>
/* Add your custom styles here */
</style>
