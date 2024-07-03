<template>
  <div class="container" v-if="lessons.length">
  
                        <div class="row justify-content-evenly">
                            
                            <!-- Displaying sorted and filtered lessons -->
                            <div v-for="(item, index) in lessons" :key="index" class="my-3 col-sm-6 col-lg-4">
    <div class="card shadow-sm h-100">
        <!-- Lesson image -->
        <img :src="ServerImage(item.picture)" class="card-img-top" alt="Lesson Image" style="height: 200px; object-fit: cover;">
        <div class="card-body d-flex flex-column">
            <h5 class="card-title">{{ item.subject }}</h5>
            <p class="card-text text-muted">{{ item.description }}</p>
            <div class="mt-auto">
                <p class="mb-1"><strong>Location:</strong> {{ item.location }}</p>
                <p class="mb-1"><strong>Available Inventory:</strong> {{ item.availability }}</p>
                <p class="mb-1"><strong>Price:</strong> {{ item.price }} AED</p>
                <!-- Add to cart button -->
                <a v-if="canAddToCart(item)" v-on:click="addToCart(item)" href="#" class="btn btn-primary mt-3 w-100">
                    ADD TO CART
                </a>
            </div>
        </div>
    </div>
</div>

                                
                            </div>
                            
                        </div>
                
  <div>
   
  </div>
</template>
<script>
export default {
  name: "LessonComponent",
  props: {
    lessons: { type: Array, required: true },
    canAddToCart: { type: Function, required: true },
    cartCount: { type: Function, required: true },
    showProduct: { type: Boolean, required: true },
  },
  data() {
    return {};
  },
  computed: {},
  methods: {
    
    addToCart(lesson) {
      this.$emit("add-to-cart", lesson);
    },
   ServerImage(img) {
      // Construct the full URL for an image
      const NodeServerUrl = "https://coursework-2.onrender.com";
      const Image = img.split("/").pop().trim();
      const FullPath = NodeServerUrl + "/" + Image;
      return FullPath;
    },
  },
};
</script>
<style>.card {
    transition: transform 0.2s, box-shadow 0.2s;
}
.card:hover {
    transform: translateY(-5px);
    box-shadow: 0 8px 16px rgba(0, 0, 0, 0.2);
}
</style>