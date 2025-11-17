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
        let fieldA = a[this.sortBy];
        let fieldB = b[this.sortBy];

        if (typeof fieldA === "string") fieldA = fieldA.toLowerCase();
        if (typeof fieldB === "string") fieldB = fieldB.toLowerCase();

        return this.sortOrder === "asc"
          ? fieldA > fieldB ? 1 : -1
          : fieldA < fieldB ? 1 : -1;
      });
    },

    switchPage(page) {
      this.page = page;
    },

    addToCart(lesson) {
      const exists = this.cart.find((item) => item._id === lesson._id);
      if (exists) {
        alert("This lesson is already in your cart.");
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

      const nameRegex = /^[A-Za-z\s]+$/;
      if (!nameRegex.test(this.name)) {
        this.nameError = "Name must contain only letters.";
      }

      const phoneRegex = /^[0-9]+$/;
      if (!phoneRegex.test(this.phone)) {
        this.phoneError = "Phone must contain only digits.";
      }

      if (this.cart.length === 0) {
        this.checkoutError = "Your cart is empty.";
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
        const savedOrder = await res.json();
        this.lastOrder = savedOrder;

        this.name = "";
        this.phone = "";
        this.cart = [];
        await this.fetchLessons();

        this.page = "confirmation";
      } else {
        this.checkoutError = "Failed to place order. Please try again.";
      }
    },
  },

  computed: {
    filteredLessons() {
      const q = this.searchQuery.toLowerCase();
      return !q
        ? this.lessons
        : this.lessons.filter((lesson) =>
            lesson.subject.toLowerCase().includes(q) ||
            lesson.location.toLowerCase().includes(q) ||
            String(lesson.price).includes(q) ||
            String(lesson.spaces).includes(q)
          );
    },

    totalPrice() {
      return this.cart.reduce((sum, item) => sum + item.price, 0);
    }
  },

  mounted() {
    this.fetchLessons();
  },
};
</script>




<template>
  <div class="container">

    <!-- NAVIGATION -->
    <div style="margin-bottom: 20px;">
      <button @click="switchPage('lessons')">Lessons</button>
      <button @click="switchPage('cart')" :disabled="cart.length === 0">
        Cart ({{ cart.length }})
      </button>
    </div>

    <!-- LESSONS PAGE -->
    <div v-if="page === 'lessons'">
      <h2>Lessons</h2>

      <div class="search">
        <input v-model="searchQuery" placeholder="Search lessons..." />
      </div>

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
          :disabled="lesson.spaces === 0 || cart.some(i => i._id === lesson._id)"
        >
          <span v-if="lesson.spaces === 0">Fully Booked</span>
          <span v-else-if="cart.some(i => i._id === lesson._id)">In Cart</span>
          <span v-else>Add to Cart</span>
        </button>
      </div>
    </div>


    <!-- CART PAGE -->
    <div v-if="page === 'cart'">
      <h2>Your Cart</h2>

      <p v-if="cart.length === 0">Your cart is empty.</p>

      <div v-else>
        <div v-for="(item, index) in cart" :key="item._id" class="cart-item">
          <p>{{ item.subject }} - £{{ item.price }}</p>
          <button @click="removeFromCart(item, index)">Remove</button>
        </div>

        <p class="total">Total: £{{ totalPrice }}</p>

        <h3>Checkout</h3>

        <form class="checkout-form">
          <input v-model="name" placeholder="Your Name" />
          <p v-if="nameError" style="color:red;">{{ nameError }}</p>

          <input v-model="phone" placeholder="Phone Number" />
          <p v-if="phoneError" style="color:red;">{{ phoneError }}</p>

          <button type="button" @click="placeOrder">Place Order</button>

          <p v-if="checkoutError" style="color:red;">{{ checkoutError }}</p>
        </form>
      </div>
    </div>


    <!-- CONFIRMATION PAGE -->
    <div v-if="page === 'confirmation'">
      <h2>Order Confirmed</h2>

      <p v-if="lastOrder">
        Thank you, {{ lastOrder.name }}. Your order has been placed successfully.
      </p>

      <h3>Ordered Lessons:</h3>
      <ul>
        <li v-for="item in lastOrder.items" :key="item.lessonId">
          {{ item.subject }} - £{{ item.price }}
        </li>
      </ul>

      <button @click="switchPage('lessons')">Back to Lessons</button>
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

.total {
  font-weight: bold;
  margin: 15px 0;
  font-size: 18px;
}

button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}


ul {
  padding-left: 20px;
}



</style>
