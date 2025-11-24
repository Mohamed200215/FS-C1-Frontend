<script>
export default {
  data() {
    return {
      lessons: [],
      cart: [],
      page: "lessons",
      API_URL:"https://backend-cxhb.onrender.com",
      sortBy: "subject",
      sortOrder: "asc",
      searchQuery: "",
      name: "",
      phone: "",
      nameError: "",
      phoneError: "",
      checkoutError: "",
      lastOrder: null,
    };
  },

  methods: {
    async fetchLessons() {
      const res = await fetch(`${this.API_URL}/lessons`);
      this.lessons = await res.json();
    },

    sortLessons() {
      this.lessons.sort((a, b) => {
        let A = a[this.sortBy];
        let B = b[this.sortBy];
        if (typeof A === "string") A = A.toLowerCase();
        if (typeof B === "string") B = B.toLowerCase();
        return this.sortOrder === "asc" ? (A > B ? 1 : -1) : (A < B ? 1 : -1);
      });
    },

    switchPage(page) {
      this.page = page;
    },

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

    removeFromCart(item, index) {
      const lesson = this.lessons.find((l) => l._id === item._id);
      if (lesson) lesson.spaces++;
      this.cart.splice(index, 1);
    },

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

      const res = await fetch(`${this.API_URL}/orders`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(order),
      });

      if (res.ok) {
        const saved = await res.json();
        this.lastOrder = saved;
        this.name = "";
        this.phone = "";
        this.cart = [];
        await this.fetchLessons();

        // UPDATE SPACES IN DATABASE FOR EACH LESSON ORDERED
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

    // SUBJECT IMAGES
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

  computed: {
    filteredLessons() {
      const q = this.searchQuery.toLowerCase();
      if (!q) return this.lessons;

      return this.lessons.filter(
        (lesson) =>
          lesson.subject.toLowerCase().includes(q) ||
          lesson.location.toLowerCase().includes(q)
      );
    },

    totalPrice() {
      return this.cart.reduce((sum, item) => sum + item.price, 0);
    },
  },

  mounted() {
    this.fetchLessons();
  },
};
</script>


<!-- ========================= TEMPLATE ============================= -->

<template>
  <div>

    <!-- HEADER -->
    <header class="banner">
      <h1 class="banner-title">AFTER SCHOOL CLASSES</h1>
    </header>

    <!-- NAV -->
    <nav class="nav-buttons">
      <button @click="switchPage('lessons')" :class="{ active: page === 'lessons' }">Lessons</button>
      <button @click="switchPage('cart')" :class="{ active: page === 'cart' }">Cart ({{ cart.length }})</button>
      <button @click="switchPage('about')" :class="{ active: page === 'about' }">About</button>
    </nav>

    <!-- LESSON PAGE -->
    <div class="container" v-if="page === 'lessons'">

      <input v-model="searchQuery" class="search-bar" placeholder="Search lessons..." />

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

      <!-- GRID -->
      <div class="grid">
        <div v-for="lesson in filteredLessons"
             :key="lesson._id"
             class="subject-box"
             :style="{ backgroundImage: 'url(' + subjectImage(lesson.subject) + ')' }">

          <div class="overlay">


           <h3 class="subject-title text-box">{{ lesson.subject }}</h3>

<p class="location-box text-box">📍 {{ lesson.location }}</p>

<p class="price-box text-box">£{{ lesson.price }}</p>

<p class="spaces-box text-box">Spaces: {{ lesson.spaces }}</p>


            <button 
              @click="addToCart(lesson)"
              :disabled="lesson.spaces === 0 || cart.some(i => i._id === lesson._id)">
              <span v-if="lesson.spaces === 0">Full</span>
              <span v-else-if="cart.some(i => i._id === lesson._id)">In Cart</span>
              <span v-else>Add to Cart</span>
            </button>

          </div>
        </div>
      </div>
    </div>

    <!-- CART PAGE -->
    <div v-if="page === 'cart'" class="cart-container">

  <h2 class="cart-title">Your Cart</h2>

  <p v-if="cart.length === 0">Your cart is empty.</p>

  <div 
    v-for="(item, index) in cart" 
    :key="item._id"
    class="cart-item"
  >
    <p>{{ item.subject }} - £{{ item.price }}</p>
    <button class="remove-btn" @click="removeFromCart(item, index)">Remove</button>
  </div>

  <div class="total-box">
    Total: £{{ totalPrice }}
  </div>

  <div class="checkout-form">
    <input v-model="name" placeholder="Your Name" />
    <p v-if="nameError" class="error">{{ nameError }}</p>

    <input v-model="phone" placeholder="Phone Number" />
    <p v-if="phoneError" class="error">{{ phoneError }}</p>

    <button class="place-order-btn" @click="placeOrder">
      Place Order
    </button>

    <p v-if="checkoutError" class="error">{{ checkoutError }}</p>
  </div>

</div>


    <!-- ABOUT PAGE -->
    <div v-if="page === 'about'" class="about-container">
      <h2>About This Website</h2>
      <p>
        This platform allows students to explore and book after-school classes easily.
        You can view available subjects, check locations, prices, and spaces.
      </p>
    </div>

    <!-- CONFIRMATION -->
    <div v-if="page === 'confirmation'" class="confirm-wrapper">
      <div class="confirm-card">
        <h2>Order Confirmed!</h2>
        <p>Thank you {{ lastOrder.name }}</p>
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

body {
  background: url('/ASC.jpg') no-repeat center center fixed;
  background-size: cover;
}
body::before {
  content:"";
  position:fixed;
  inset:0;
  background:rgba(0,0,0,0.6);
  z-index:-1;
}

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

.search-bar {
  width:100%;
  padding:10px;
  margin-bottom:15px;
  border-radius:6px;
  border:5px solid #005ab5;
  background:rgb(255, 255, 255);
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

.grid {
  display:grid;
  grid-template-columns: repeat(3, 1fr);
  gap:20px;
}

/* SUBJECT CARD */
.subject-box {
  height:350px;
  width: 250PX;
  background-size:cover;
  background-position:center;
  border-radius:10px;
  position:relative;
  overflow:hidden;
  box-shadow:0 4px 10px rgba(0, 0, 0, 0.5);
}

.overlay {
  position: absolute;
  inset: 0;
  color: white;
  display: flex;
  flex-direction: column;
  justify-content: space-between;  
  align-items: center;
padding: 10px 12px;
  gap: 4px; 
}

/* SUBJECT TITLE */
.subject-title {
  font-size: 22px;
  font-weight: 900;
  margin: 0;
  text-shadow: 0 0 6px black;
}


/* PIN LOCATION */
.location {
  font-size: 16px;
  font-weight: 600;
  margin: 4px 0 10px 0;
  color: #e8e8e8;
}

/* PRICE */
.price {
  font-size:18px;
  font-weight:900;
  margin-bottom:4px;
}

/* SPACES */
.spaces {
  font-size:16px;
  margin-bottom:12px;
}

/* BUTTON */
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

.about-container {
  width:700px;
  margin:auto;
  background:rgb(0, 0, 0);
  padding:20px;
  border-radius:10px;
}

/* ========================= CART PAGE ========================= */

.cart-container {
  width: 700px;
  margin: auto;
  background: rgba(0, 0, 0, 0.75);
  padding: 25px;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.6);
  color: white;
}

.cart-title {
  font-size: 32px;
  font-weight: 900;
  text-align: center;
  margin-bottom: 20px;
  color: #00c853;
  text-shadow: 0 0 6px black;
}

.cart-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: rgba(255, 255, 255, 0.1);
  padding: 14px 18px;
  margin-bottom: 12px;
  border-radius: 10px;
  font-size: 18px;
  font-weight: 700;
  box-shadow: 0 0 6px rgba(0,0,0,0.4);
}

.remove-btn {
  background: #ff4444;
  border: none;
  padding: 8px 14px;
  border-radius: 6px;
  font-weight: 800;
  cursor: pointer;
  color: white;
  box-shadow: 0 0 6px rgba(0,0,0,0.5);
}
.remove-btn:hover {
  background: #cc0000;
}

.total-box {
  margin-top: 20px;
  background: #005ab5;
  padding: 15px;
  border-radius: 10px;
  font-size: 22px;
  font-weight: 900;
  color: white;
  text-align: center;
  box-shadow: 0 0 8px rgba(0,0,0,0.5);
}

/* Checkout Form */
.checkout-form {
  margin-top: 25px;
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.checkout-form input {
  padding: 12px 14px;
  border-radius: 8px;
  border: 3px solid #00c853;
  font-size: 18px;
  background: rgb(246, 245, 255);
}

.place-order-btn {
  margin-top: 10px;
  padding: 14px 20px;
  font-size: 20px;
  font-weight: 900;
  background: #00c853;
  border-radius: 10px;
  border: none;
  cursor: pointer;
  color: black;
  box-shadow: 0 0 8px rgba(0,0,0,0.7);
}
.place-order-btn:hover {
  background: #11d964;
}

.error {
  color: #ff4444;
  font-weight: 800;
  text-shadow: 0 0 4px black;
}


input {
  color: black !important;
}

/* CONFIRMATION POPUP */
.confirm-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 60vh;
}

.confirm-card {
  background: #000000d0; 
  padding: 30px;
  width: 450px;
  border-radius: 12px;
  border-left: 8px solid #00c853;
  text-align: center;
  box-shadow: 0 5px 18px rgba(0,0,0,0.6);
  color: white;
}

.confirm-card h2 {
  font-size: 28px;
  margin-bottom: 10px;
  color: #00c853;
}

.confirm-card ul {
  list-style: none;
  padding: 0;
  margin-top: 10px;
}

.confirm-card li {
  background: white;
  color: black;
  padding: 10px;
  margin-bottom: 8px;
  border-radius: 6px;
  font-weight: 700;
}

.confirm-card button {
  margin-top: 18px;
  padding: 12px 20px;
  background: #005ab5;
  color: white;
  border: none;
  border-radius: 8px;
  font-weight: bold;
  cursor: pointer;
  font-size: 16px;
  box-shadow: 0 0 8px rgba(0,0,0,0.6);
}

.text-box {
  background: rgba(0, 0, 0, 0.55);
  padding: 6px 10px;
  border-radius: 8px;
  box-shadow: 0 0 6px rgba(0,0,0,0.6);
  backdrop-filter: blur(3px);
}

.subject-title {
  font-size: 22px;
  font-weight: 900;
  margin: 0;
  color: white;
  text-shadow: 0 0 6px black;
}

.location-box,
.price-box,
.spaces-box {
  color: white;
  font-weight: 700;
  font-size: 16px;
  text-shadow: 0 0 4px black;
  text-align: center;
}

</style>
