  <template>
    <div>
      <header>
                <!-- Navigation bar -->
                <nav class="navbar navbar-expand-lg navbar-light bg-white border-bottom">
    <div class="container">
        <!-- Brand name displayed from Vue.js data -->
        <a class="navbar-brand" href="#" v-text="sitename"></a>
        
        <!-- Toggler button for mobile view -->
        <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav" aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
            <span class="navbar-toggler-icon"></span>
        </button>
        
        <div class="collapse navbar-collapse" id="navbarNav">
            <!-- Search input -->
            <div class="d-flex align-items-center ms-auto">
                <div class="input-group mx-3">
                    <input
                        type="text"
                        v-on:keyup="Searching()"
                        v-model="search"
                        placeholder="Search Lesson"
                        id="search-input"
                        class="form-control form-control-sm"
                        aria-describedby="searchButton"
                    >
                </div>
                <!-- Cart button with item count -->
                <button v-on:click="toggleCheckout()" type="button" class="btn btn-primary position-relative">
                    <i class="fa-solid fa-cart-shopping fs-6"></i>
                    <span class="position-absolute top-0 start-100 translate-middle badge rounded-pill bg-danger">
                        {{ cart.length }}
                        <span class="visually-hidden">items in cart</span>
                    </span>
                </button>
            </div>
        </div>
    </div>
</nav>

                </header>
        
                <div v-if="showProduct" class="container my-5">
    <div>
        <h2 class="mb-4">Lessons</h2>
        <div class="row align-items-center mb-4">
            <!-- Sorting options -->
            <div class="col-md-3 mb-2 mb-md-0">
                <select v-model="sortBy" class="form-select form-select-sm">
                    <option value="subject">Subject</option>
                    <option value="location">Location</option>
                    <option value="price">Price</option>
                    <option value="availability">Availability</option>
                </select>
            </div>
            <div class="col-md-3 mb-2 mb-md-0">
                <select v-model="sortOrder" class="form-select form-select-sm">
                    <option value="asc">Ascending</option>
                    <option value="desc">Descending</option>
                </select>
            </div>
            <!-- Search input -->
            <div class="col-md-6">
                <div class="input-group">
                    <input
                        type="text"
                        v-on:keyup="Searching()"
                        v-model="search"
                        placeholder="Search Lesson"
                        id="search-input"
                        class="form-control form-control-sm"
                        aria-describedby="searchButton"
                    >
                </div>
            </div>
        </div>
    </div>
</div>

<div v-if="showProduct" class="my-5">
    <LessonComponent
        :show-product="showProduct"
        :cart-count="cartCount"
        :lessons="filteredLessons"
        :can-add-to-cart="canAddToCart"
        @add-to-cart="addToCart"
    />
</div>

<div class="cart mt-5" v-else>
    <CheckoutForm
        :cart-count="cartCount"
        :save-order="saveOrder"
        @remove-item-from-cart="removeFromCart"
        :cart="cart"
        :showhome-page-btn="showhomePageBtn"
    />
</div>


  
    </div>
  </template>

  <script>
  import CheckoutForm from './components/CheckoutForm.vue';
  import LessonComponent from './components/LessonComponent.vue';

  export default {
    name: 'App',
    components: {
      CheckoutForm,
      LessonComponent
    },
    data() {
      return {
        sitename: 'After School Activities',
        products: [],
        cart: [],
        search: '',
        showProduct: true, // Assuming you want to show products by default
        sortBy: "subject", // Default sorting attribute
        sortOrder: 'asc', // Default sorting order
        ascendingOrder: true, // Default ascending order
        baseUrl: 'https://coursework-2.onrender.com',
        showInstallPrompt: false,
        deferredPrompt: null
      };
    },
    created() {
      this.fetchLessons();

      window.addEventListener('beforeinstallprompt', (e) => {
        // Prevent the mini-infobar from appearing on mobile
        e.preventDefault();
        // Stash the event so it can be triggered later.
        this.deferredPrompt = e;
        // Update UI to notify the user they can install the PWA
        this.showInstallPrompt = true;
      });
    },
    methods: {
      fetchLessons() {
        fetch(`${this.baseUrl}/collection/products`) // Assuming this is the correct endpoint on your server
          .then(response => response.json())
          .then(data => {
            this.products = data;
          })
          .catch(error => {
            console.error('Error fetching lessons:', error);
          });
      },
      saveOrder(orderData) {
        console.log(orderData)
        // Send the new order to the server
        fetch(
          `${this.baseUrl}/collection/orders`,
          {
            method: "POST",
            headers: {
              "Content-Type": "application/json"
            },
            body: JSON.stringify(orderData)
          }
        )
          .then((res) => res.json())
          .then((resjson) => {
            console.log("🚀 ~ ProcessOrder ~ resjson:", resjson);
          });
        console.log(orderData["lessonItems"])
        // Update the product inventory on the server
        orderData["lessonItems"].forEach((item) => {
          this.UpdateProduct(item.id, item.quantity);
        });

        // Clear the cart
        this.cart = [];
        this.name = ""; // Customer's name
        this.phone = ""; // Customer's phone number
      },

      UpdateProduct(id, spaceValue) {
        // Update the product inventory on the server
        fetch(
          `${this.baseUrl}/collection/products/${id}/reduce/availableSpace/${spaceValue}`,
          {
            method: "PUT",
            headers: {
              "Content-Type": "application/json",
              mode: "no-cors"
            }
          }
        )
          .then((res) => res.json())
          .then((resjson) =>
            console.log("🚀 ~ UpdateProduct ~ resjson:", resjson)
          );
      },
      addToCart(product) {
        if (this.canAddToCart(product)) {
          const index = this.products.findIndex(p => p.id === product.id);
          if (index !== -1) {
            this.products[index].availability--;
            this.cart.push({ ...product });
          }
        }
      },
      canAddToCart(product) {
        return product.availability > this.cartCount(product.id) && product.availability > 0;
      },
      cartCount(product) {
        // Count the number of occurrences of a product in the cart
        let cartProduct = this.cart.find((item) => item.id === product.id);
        return cartProduct ? cartProduct.quantity : 0;
      },
      toggleCheckout() {
        this.showProduct = !this.showProduct;
      },
      removeFromCart(productId) {
        const index = this.cart.findIndex(item => item.id === productId);
        if (index !== -1) {
          const productIndex = this.products.findIndex(p => p.id === productId);
          if (productIndex !== -1) {
            this.products[productIndex].availability++;
          }
          this.cart.splice(index, 1);
        }
      },
      promptInstall() {
        if (this.deferredPrompt) {
          this.deferredPrompt.prompt();
          this.deferredPrompt.userChoice.then(choiceResult => {
            if (choiceResult.outcome === 'accepted') {
              console.log('User accepted the install prompt');
            } else {
              console.log('User dismissed the install prompt');
            }
            this.deferredPrompt = null;
            this.showInstallPrompt = false;
          });
        }
      },
      Searching() {
        // Perform a search based on the search input
        let searchTerm = this.search.toLowerCase();
        fetch(
          `${this.baseUrl}/collection/products/search`,
          {
            method: "POST",
            headers: {
              "Content-Type": "application/json"
            },
            body: JSON.stringify({ query: searchTerm })
          }
        ).then(response => response.json())
          .then(data => {
            this.products = data;
          })
          .catch(error => {
            console.error('Error fetching lessons:', error);
          });
      },
   
      showhomePageBtn:function () {
        // Toggle the view between the product list and the checkout page
        try {
          fetch(
            `${this.baseUrl}/collection/products`,
            {
              method: "GET",
              headers: {
                "Content-Type": "application/json"
              },
            }

          ).then(response => response.json())
          .then(data => {
            this.products = data;
          })
          .catch(error => {
            console.error('Error fetching lessons:', error);
          });
          this.toggleCheckout()
        } catch (ex) {
          console.error("🚀 ~ ex:", ex);
        }
      }
    },
    computed: {
    filteredLessons() {
    let lessonsCopy = [...this.products];
        return lessonsCopy.sort((a, b) => {
          let comparison = 0;
          switch (this.sortBy) {
            case "subject":
              comparison = a.subject.localeCompare(b.subject);
              break;
            case "location":
              comparison = a.location.localeCompare(b.location);
              break;
            case "price":
              comparison = a.price - b.price;
              break;
            case "availability":
              comparison = a.availability - b.availability;
              break;
          }
          return this.sortOrder === "asc" ? comparison : -comparison;
        });
    }
    }
  };
  </script>
