<script>
export default {
  data() {
    return {
      page: "lessons", // "lessons" or "cart"
    };
  },
  methods: {
    switchPage(page) {
      this.page = page;
    },
  },
};
</script>

<template>
  <div class="container">
    <!-- NAVIGATION BUTTONS -->
    <div style="margin-bottom: 20px;">
      <button @click="switchPage('lessons')">
        Lessons
      </button>

      <button @click="switchPage('cart')">
        Cart
      </button>
    </div>

    <!-- LESSONS PAGE -->
    <div v-if="page === 'lessons'">
      <h2>Lessons</h2>
      <p>This is the lessons page.</p>
    </div>

    <!-- CART PAGE -->
    <div v-if="page === 'cart'">
      <h2>Your Cart</h2>
      <p>This is the cart page.</p>
    </div>
  </div>
</template>

<style>
.container {
  width: 600px;
  margin: auto;
}
button {
  padding: 6px 12px;
  margin-top: 10px;
}
</style>
