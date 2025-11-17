<script>
export default {
data() {
  return {
    lessons: [],
    cart: [],
    page: "lessons",
    API_URL: "http://localhost:3000",
    sortBy: "subject",
    sortOrder: "asc",
    searchQuery: "",
    name: "",
    phone: "",
  };
},


  methods: {
    async fetchLessons() {
      const res = await fetch(`${this.API_URL}/lessons`);
      this.lessons = await res.json();
    },

    sortLessons() {
      this.lessons.sort((a, b) => {
        let fieldA = a[this.sortBy];
        let fieldB = b[this.sortBy];

        if (typeof fieldA === "string") fieldA = fieldA.toLowerCase();
        if (typeof fieldB === "string") fieldB = fieldB.toLowerCase();

        if (this.sortOrder === "asc") {
          return fieldA > fieldB ? 1 : -1;
        } else {
          return fieldA < fieldB ? 1 : -1;
        }
      });
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

    removeFromCart(item, index) {
      const lesson = this.lessons.find((l) => l._id === item._id);
      if (lesson) {
        lesson.spaces++;
      }
      this.cart.splice(index, 1);
    },
  },

  computed: {
    filteredLessons() {
      if (!this.searchQuery) {
        return this.lessons;
      }

      const query = this.searchQuery.toLowerCase();

      return this.lessons.filter((lesson) => {
        return (
          lesson.subject.toLowerCase().includes(query) ||
          lesson.location.toLowerCase().includes(query) ||
          String(lesson.price).includes(query) ||
          String(lesson.spaces).includes(query)
        );
      });
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

      <button @click="switchPage('cart')" :disabled="cart.length === 0">
        Cart ({{ cart.length }})
      </button>
    </div>

    <!-- LESSONS PAGE -->
    <div v-if="page === 'lessons'">
      <h2>Lessons</h2>

      <!-- SEARCH BAR UI -->
<div class="search">
  <input 
    type="text" 
    v-model="searchQuery" 
    placeholder="Search lessons..."
  />
</div>


<!-- SORTING UI -->
<div class="sorting">
  <label>Sort by:</label>

  <select v-model="sortBy" @change="sortLessons">
    <option value="subject">Subject</option>
    <option value="location">Location</option>
    <option value="price">Price</option>
    <option value="spaces">Spaces</option>
  </select>

  <select v-model="sortOrder" @change="sortLessons">
    <option value="asc">Ascending</option>
    <option value="desc">Descending</option>
  </select>
</div>



     <div v-for="lesson in filteredLessons" :key="lesson._id" class="lesson-card">
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
<div 
  v-for="(item, index) in cart" 
  :key="item._id" 
  class="cart-item"
>
  <p>{{ item.subject }} - £{{ item.price }}</p>

  <button @click="removeFromCart(item, index)">
    Remove
  </button>

  <h3>Checkout</h3>

<form class="checkout-form">
  <input
    type="text"
    v-model="name"
    placeholder="Your Name"
  />

  <input
    type="text"
    v-model="phone"
    placeholder="Phone Number"
  />

  <button type="button">
    Place Order
  </button>
</form>

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

.search input {
  width: 100%;
  padding: 8px;
  margin-bottom: 20px;
  border-radius: 4px;
  border: 1px solid #ccc;
}

.checkout-form {
  margin-top: 20px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.checkout-form input {
  padding: 8px;
  border-radius: 4px;
  border: 1px solid #ccc;
}

.checkout-form button {
  padding: 8px;
}


</style>
