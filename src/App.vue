<script>
export default {
  data() {
    return {
      page: "lessons",
      API_URL: "http://localhost:3000",
      lessons: [],
    };
  },
  methods: {
    switchPage(page) {
      this.page = page;
    },
    async fetchLessons() {
      const res = await fetch(`${this.API_URL}/lessons`);
      this.lessons = await res.json();
    },
  },
  mounted() {
    this.fetchLessons();
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

      <div v-for="lesson in lessons" :key="lesson._id" class="lesson-card">
        <h3>{{ lesson.subject }}</h3>
      </div>
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
.lesson-card {
  border: 1px solid #ddd;
  padding: 12px;
  margin-bottom: 12px;
}
button {
  padding: 6px 12px;
  margin-top: 10px;
}
</style>
