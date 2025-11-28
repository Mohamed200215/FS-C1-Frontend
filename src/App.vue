<script>
// -----------------------------------------
// MAIN APP LOGIC (Vue Component)
// -----------------------------------------
export default {
  // -----------------------
  // COMPONENT STATE (DATA)
  // -----------------------
  data() {
    return {
      lessons: [],
      cart: [],
      page: "lessons",
      API_URL:"https://backend-cxhb.onrender.com",

      // Sorting / searching
      sortBy: "subject",
      sortOrder: "asc",
      searchQuery: "",

      // Checkout form
      name: "",
      phone: "",
      nameError: "",
      phoneError: "",
      checkoutError: "",

      // Confirmation
      lastOrder: null,
    };
  },

  // -----------------------
  // METHODS (APP FUNCTIONS)
  // -----------------------
  methods: {
    // Fetch lessons from backend API
    async fetchLessons() {
      const res = await fetch(`${this.API_URL}/lessons`);
      this.lessons = await res.json();
    },

    // Sort lessons based on dropdown selection
    sortLessons() {
      this.lessons.sort((a, b) => {
        let A = a[this.sortBy];
        let B = b[this.sortBy];

        if (typeof A === "string") A = A.toLowerCase();
        if (typeof B === "string") B = B.toLowerCase();

        return this.sortOrder === "asc"
          ? (A > B ? 1 : -1)
          : (A < B ? 1 : -1);
      });
    },

    // Switch between pages (lessons/cart/about/confirmation)
    switchPage(page) {
      this.page = page;
    },

    // Add lesson to cart and reduce spaces locally
    addToCart(lesson) {
      const exists = this.cart.find((i) => i._id === lesson._id);
      if (exists) {
        alert("Already in cart.");
        return;
      }
      if (lesson.spaces > 0) {
        lesson.spaces--;
        this.cart.push(lesson);
      }
    },

    // Remove item from cart and restore lesson spaces
    removeFromCart(item, index) {
      const lesson = this.lessons.find((l) => l._id === item._id);
      if (lesson) lesson.spaces++;
      this.cart.splice(index, 1);
    },

    // Validate checkout form inputs
    validateCheckout() {
      this.nameError = "";
      this.phoneError = "";
      this.checkoutError = "";

      if (!/^[a-zA-Z\s]+$/.test(this.name)) {
        this.nameError = "Name must contain only letters.";
      }
      if (!/^[0-9]+$/.test(this.phone)) {
        this.phoneError = "Phone must contain only digits.";
      }
      if (this.cart.length === 0) {
        this.checkoutError = "Cart is empty.";
      }

      return !this.nameError && !this.phoneError && !this.checkoutError;
    },

    // Submit an order to backend and update lesson spaces
    async placeOrder() {
      if (!this.validateCheckout()) return;

      const order = {
        name: this.name,
        phone: this.phone,
        items: this.cart.map((item) => ({
          lessonId: item._id,
          subject: item.subject,
          price: item.price,
        })),
      };

      // Send POST request
      const res = await fetch(`${this.API_URL}/orders`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(order),
      });

      if (res.ok) {
        const saved = await res.json();
        this.lastOrder = saved;

        // Reset form + cart
        this.name = "";
        this.phone = "";
        this.cart = [];

        // Refresh lessons from DB
        await this.fetchLessons();

        // -----------------------------------------
        // UPDATE SPACES IN DATABASE (PUT Request)
        // -----------------------------------------
        for (const item of this.lastOrder.items) {
          const lesson = this.lessons.find(l => l._id === item.lessonId);

          if (lesson) {
            await fetch(`${this.API_URL}/lessons/${item.lessonId}`, {
              method: "PUT",
              headers: { "Content-Type": "application/json" },
              body: JSON.stringify({ spaces: lesson.spaces })
            });
          }
        }

        this.page = "confirmation";
      } else {
        this.checkoutError = "Order failed.";
      }
    },

    // Return correct image based on lesson subject
    subjectImage(subject) {
      const images = {
        "Science": "/images/science.jpg",
        "Math": "/images/math.jpg",
        "Music": "/images/music.jpg",
        "Physical Education": "/images/pe.jpg",
        "Drama": "/images/drama.jpg",
        "Art": "/images/art.jpg",
        "Food Technology": "/images/food.jpg",
        "Media Studies": "/images/media.jpg",
        "History": "/images/history.jpg",
        "Geography": "/images/geography.jpg",
      };

      return images[subject] || "/images/default.jpg";
    },
  },

  // -----------------------
  // COMPUTED PROPERTIES
  // -----------------------
  computed: {
    // Apply search filter
    filteredLessons() {
      const q = this.searchQuery.toLowerCase();
      if (!q) return this.lessons;

      return this.lessons.filter(
        (lesson) =>
          lesson.subject.toLowerCase().includes(q) ||
          lesson.location.toLowerCase().includes(q)
      );
    },

    // Calculate total checkout price
    totalPrice() {
      return this.cart.reduce((sum, item) => sum + item.price, 0);
    },
  },

  // -----------------------
  // LIFECYCLE HOOK
  // -----------------------
  mounted() {
    this.fetchLessons();
  },
};
</script>


<!-- ========================= TEMPLATE ============================= -->

<template>
  <div>

    <!-- ================= HEADER ================= -->
    <!-- Main title banner for the website -->
    <header class="banner">
      <h1 class="banner-title">AFTER SCHOOL CLASSES</h1>
    </header>

    <!-- ================= NAVIGATION ================= -->
    <!-- Navigation buttons to switch between pages -->
    <nav class="nav-buttons">
      <button @click="switchPage('lessons')" :class="{ active: page === 'lessons' }">Lessons</button>
      <button @click="switchPage('cart')" :class="{ active: page === 'cart' }">Cart ({{ cart.length }})</button>
      <button @click="switchPage('about')" :class="{ active: page === 'about' }">About</button>
    </nav>

    <!-- ================= LESSONS PAGE ================= -->
    <div class="container" v-if="page === 'lessons'">

      <!-- Search bar -->
      <input v-model="searchQuery" class="search-bar" placeholder="Search lessons..." />

      <!-- Sorting dropdowns -->
      <div class="sorting">
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

      <!-- Lessons Grid -->
      <div class="grid">
        <div 
          v-for="lesson in filteredLessons"
          :key="lesson._id"
          class="subject-box"
          :style="{ backgroundImage: 'url(' + subjectImage(lesson.subject) + ')' }"
        >

          <!-- Lesson content overlay -->
          <div class="overlay">

            <h3 class="subject-title text-box">{{ lesson.subject }}</h3>
            <p class="location-box text-box">📍 {{ lesson.location }}</p>
            <p class="price-box text-box">£{{ lesson.price }}</p>
            <p class="spaces-box text-box">Spaces: {{ lesson.spaces }}</p>

            <!-- Add to cart button (disabled if full or already added) -->
            <button 
              @click="addToCart(lesson)"
              :disabled="lesson.spaces === 0 || cart.some(i => i._id === lesson._id)"
            >
              <span v-if="lesson.spaces === 0">Full</span>
              <span v-else-if="cart.some(i => i._id === lesson._id)">In Cart</span>
              <span v-else>Add to Cart</span>
            </button>

          </div>
        </div>
      </div>
    </div>

    <!-- ================= CART PAGE ================= -->
    <div v-if="page === 'cart'" class="cart-container">

      <h2 class="cart-title">Your Cart</h2>

      <!-- Empty cart message -->
      <p v-if="cart.length === 0">Your cart is empty.</p>

      <!-- Cart item list -->
      <div 
        v-for="(item, index) in cart" 
        :key="item._id"
        class="cart-item"
      >
        <p>{{ item.subject }} - £{{ item.price }}</p>
        <button class="remove-btn" @click="removeFromCart(item, index)">Remove</button>
      </div>

      <!-- Total price display -->
      <div class="total-box">
        Total: £{{ totalPrice }}
      </div>

      <!-- Checkout form -->
      <div class="checkout-form">
        <input v-model="name" placeholder="Your Name" />
        <p v-if="nameError" class="error">{{ nameError }}</p>

        <input v-model="phone" placeholder="Phone Number" />
        <p v-if="phoneError" class="error">{{ phoneError }}</p>

        <button class="place-order-btn" @click="placeOrder">Place Order</button>

        <p v-if="checkoutError" class="error">{{ checkoutError }}</p>
      </div>

    </div>

    <!-- ================= ABOUT PAGE ================= -->
    <div v-if="page === 'about'" class="about-container">
      <h2>About This Website</h2>
      <p>
        This platform allows students to explore and book after-school classes easily.
        You can view available subjects, check locations, prices, and spaces.
      </p>
    </div>

    <!-- ================= CONFIRMATION PAGE ================= -->
    <div v-if="page === 'confirmation'" class="confirm-wrapper">
      <div class="confirm-card">
        <h2>Order Confirmed!</h2>
        <p>Thank you {{ lastOrder.name }}</p>

        <!-- Order summary -->
        <ul>
          <li v-for="item in lastOrder.items" :key="item.lessonId">
            {{ item.subject }} - £{{ item.price }}
          </li>
        </ul>

        <button @click="switchPage('lessons')">Back to Lessons</button>
      </div>
    </div>

  </div>
</template>




<!-- ========================= STYLE ============================= -->

<style>

/* ========================= GLOBAL BACKGROUND ========================= */
body {
  background: url('/ASC.jpg') no-repeat center center fixed;
  background-size: cover;
}
body::before {
  content:"";
  position:fixed;
  inset:0;
  background:rgba(0,0,0,0.6); /* Dark overlay */
  z-index:-1;
}

/* ========================= HEADER ========================= */
.banner {
  width:100%;
  background:#005ab5;
  padding:25px 0;
  text-align:center;
}
.banner-title {
  color:#00c853;
  font-size:40px;
  font-weight:900;
}

/* ========================= NAVIGATION ========================= */
.nav-buttons {
  display:flex;
  justify-content:center;
  gap:20px;
  margin:20px 0;
}
.nav-buttons button {
  padding:10px 20px;
  background:#005ab5;
  color:white;
  border:none;
  border-radius:8px;
  font-weight:700;
  cursor:pointer;
}
.nav-buttons .active {
  background:#00c853;
  color:black;
}

/* ========================= SEARCH + SORTING ========================= */
.search-bar {
  width:100%;
  padding:10px;
  margin-bottom:15px;
  border-radius:6px;
  border:5px solid #005ab5;
  background:white;
}

.sorting {
  display:flex;
  justify-content:center;
  gap:20px;
  margin-bottom:20px;
}
.sorting select {
  padding:10px;
  background:#0088ff;
  border-radius:6px;
  border:none;
  color:white;
}

/* ========================= LESSON GRID ========================= */
.grid {
  display:grid;
  grid-template-columns: repeat(3, 1fr);
  gap:20px;
}

/* ========================= LESSON CARD ========================= */
.subject-box {
  height:350px;
  width:250px;
  background-size:cover;
  background-position:center;
  border-radius:10px;
  position:relative;
  overflow:hidden;
  box-shadow:0 4px 10px rgba(0, 0, 0, 0.5);
}

.overlay {
  position:absolute;
  inset:0;
  color:white;
  display:flex;
  flex-direction:column;
  justify-content:space-between;
  align-items:center;
  padding:10px 12px;
  gap:4px;
}

/* Text inside lesson cards */
.subject-title {
  font-size:22px;
  font-weight:900;
  margin:0;
  text-shadow:0 0 6px black;
}

.location {
  font-size:16px;
  font-weight:600;
  color:#e8e8e8;
}

.price {
  font-size:18px;
  font-weight:900;
}

.spaces {
  font-size:16px;
}

/* Add to Cart button */
.subject-box button {
  margin-top:10px;
  background:#00c853;
  padding:10px 14px;
  border:none;
  border-radius:8px;
  font-weight:900;
  cursor:pointer;
  box-shadow:0 0 8px rgba(0,0,0,0.8);
}
.subject-box button:disabled {
  background:#777;
  color:#eee;
}

/* ========================= ABOUT PAGE ========================= */
.about-container {
  width:700px;
  margin:auto;
  background:black;
  padding:20px;
  border-radius:10px;
}

/* ========================= CART PAGE ========================= */
.cart-container {
  width:700px;
  margin:auto;
  background:rgba(0, 0, 0, 0.75);
  padding:25px;
  border-radius:12px;
  box-shadow:0 4px 12px rgba(0,0,0,0.6);
  color:white;
}

.cart-title {
  font-size:32px;
  font-weight:900;
  text-align:center;
  margin-bottom:20px;
  color:#00c853;
  text-shadow:0 0 6px black;
}

/* Cart item box */
.cart-item {
  display:flex;
  justify-content:space-between;
  align-items:center;
  background:rgba(255,255,255,0.1);
  padding:14px 18px;
  margin-bottom:12px;
  border-radius:10px;
  font-size:18px;
  font-weight:700;
}

/* Remove button */
.remove-btn {
  background:#ff4444;
  border:none;
  padding:8px 14px;
  border-radius:6px;
  font-weight:800;
  cursor:pointer;
  color:white;
}
.remove-btn:hover {
  background:#cc0000;
}

/* Total box */
.total-box {
  margin-top:20px;
  background:#005ab5;
  padding:15px;
  border-radius:10px;
  font-size:22px;
  font-weight:900;
  text-align:center;
  color:white;
}

/* Checkout form */
.checkout-form {
  margin-top:25px;
  display:flex;
  flex-direction:column;
  gap:12px;
}
.checkout-form input {
  padding:12px 14px;
  border-radius:8px;
  border:3px solid #00c853;
  font-size:18px;
  background:white;
}

.place-order-btn {
  margin-top:10px;
  padding:14px 20px;
  font-size:20px;
  font-weight:900;
  background:#00c853;
  border-radius:10px;
  cursor:pointer;
  color:black;
}
.place-order-btn:hover {
  background:#11d964;
}

.error {
  color:#ff4444;
  font-weight:800;
}

/* ========================= CONFIRMATION PAGE ========================= */
.confirm-wrapper {
  display:flex;
  justify-content:center;
  align-items:center;
  min-height:60vh;
}

.confirm-card {
  background:#000000d0;
  padding:30px;
  width:450px;
  border-radius:12px;
  border-left:8px solid #00c853;
  text-align:center;
  color:white;
}

.confirm-card h2 {
  font-size:28px;
  margin-bottom:10px;
  color:#00c853;
}

.confirm-card li {
  background:white;
  color:black;
  padding:10px;
  margin-bottom:8px;
  border-radius:6px;
  font-weight:700;
}

.confirm-card button {
  margin-top:18px;
  padding:12px 20px;
  background:#005ab5;
  color:white;
  border:none;
  border-radius:8px;
  cursor:pointer;
}

.text-box {
  background:rgba(0,0,0,0.55);
  padding:6px 10px;
  border-radius:8px;
}

.subject-title {
  font-size:22px;
  font-weight:900;
  color:white;
}

.location-box,
.price-box,
.spaces-box {
  color:white;
  font-weight:700;
  font-size:16px;
  text-align:center;
}

</style>
