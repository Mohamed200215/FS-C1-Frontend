<script>
export default {
  data() {
    return {
      lessons: [],
      cart: [],
      page: "lessons", // lessons | cart
      API_URL: "http://localhost:3000",
      sortBy: "subject",
      sortOrder: "asc",
       //backend URL
 };
  },
  methods: {
    async fetchLessons() {
      const res = await fetch(`${this.API_URL}/lessons`);
      this.lessons = await res.json();
    },
    switchPage(page) {
      this.page = page;
    },
    addToCart(lesson) {
      if (lesson.spaces > 0) {
        lesson.spaces--;
        this.cart.push(lesson);
      }
    },
  },
  mounted() {
    this.fetchLessons();
  }
};
</script>

<template>
  <div class="container">

    <!-- NAVIGATION BUTTONS -->
    <div style="margin-bottom: 20px;">
      <button @click="switchPage('lessons')">
        Lessons
      </button>

      <button @click="switchPage('cart')" :disabled="cart.length === 0">
        Cart ({{ cart.length }})
      </button>
    </div>

    <!-- LESSONS PAGE -->
    <div v-if="page === 'lessons'">
      <h2>Lessons</h2>

      <!-- SORTING UI -->
<div class="sorting">
  <label>Sort by:</label>
  <select v-model="sortBy">
    <option value="subject">Subject</option>
    <option value="location">Location</option>
    <option value="price">Price</option>
    <option value="spaces">Spaces</option>
  </select>

  <select v-model="sortOrder">
    <option value="asc">Ascending</option>
    <option value="desc">Descending</option>
  </select>
</div>


      <div v-for="lesson in lessons" :key="lesson._id" class="lesson-card">
        <h3>{{ lesson.subject }}</h3>
        <p>Location: {{ lesson.location }}</p>
        <p>Price: £{{ lesson.price }}</p>
        <p>Spaces: {{ lesson.spaces }}</p>

        <button 
          @click="addToCart(lesson)"
          :disabled="lesson.spaces === 0"
        >
          Add to Cart
        </button>
      </div>
    </div>

    <!-- CART PAGE -->
    <div v-if="page === 'cart'">
      <h2>Your Cart</h2>

      <div v-for="item in cart" :key="item._id" class="cart-item">
        <p>{{ item.subject }} - £{{ item.price }}</p>
      </div>

    </div>

  </div>
</template>

<style>
.container {
  width: 600px;
  margin: auto;
}
.lesson-card, .cart-item {
  border: 1px solid #ddd;
  padding: 12px;
  margin-bottom: 12px;
}
button {
  padding: 6px 12px;
  margin-top: 10px;
}

.sorting {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

</style>
