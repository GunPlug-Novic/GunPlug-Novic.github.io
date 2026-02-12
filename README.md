<script type="module">
  import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.0/firebase-app.js";
  import { getFirestore, collection, addDoc, getDocs, deleteDoc, doc } 
  from "https://www.gstatic.com/firebasejs/10.12.0/firebase-firestore.js";

  const firebaseConfig = {
    apiKey: "AIzaSyDbHH0sAjhr99twzXv3qABC53DGCb8iarg",
    authDomain: "gunplug-cdf00.firebaseapp.com",
    projectId: "gunplug-cdf00",
    storageBucket: "gunplug-cdf00.firebasestorage.app",
    messagingSenderId: "374028571209",
    appId: "1:374028571209:web:c0b70e14dac98114fbe598"
  };

  const app = initializeApp(firebaseConfig);
  const db = getFirestore(app);

  window.db = db;
  window.fb = { collection, addDoc, getDocs, deleteDoc, doc };
</script>

<script>
document.addEventListener("DOMContentLoaded", function() {

  let cart = JSON.parse(localStorage.getItem("cart")) || [];
  let currentAdmin = null;

  const defaultAdmin = { username: "MK12", password: btoa("2121") };
  localStorage.setItem("admins", JSON.stringify([defaultAdmin]));
  let admins = JSON.parse(localStorage.getItem("admins"));

  function parsePrice(text) {
    return parseInt(text.replace("Pris:", "").replace("kr", "").replace(/\./g, "").trim());
  }

  function formatPrice(num) {
    return num.toLocaleString("da-DK") + " kr";
  }

  function saveCart() { localStorage.setItem("cart", JSON.stringify(cart)); }

  function addToCart(productElement) {
    const name = productElement.querySelector("h3").innerText;
    const price = parsePrice(productElement.querySelector(".price").innerText);

    const existing = cart.find(p => p.name === name);
    if (existing) existing.qty++;
    else cart.push({ name, price, qty: 1 });

    saveCart();
    updateCart();
  }

  function updateCart() {
    const container = document.getElementById("cart-items");
    container.innerHTML = "";
    let total = 0;

    cart.forEach((item, index) => {
      total += item.price * item.qty;

      const div = document.createElement("div");
      div.className = "cart-item";
      div.innerHTML = `
        <strong>${item.name}</strong><br>
        <button data-action="decrease" data-index="${index}">-</button>
        ${item.qty} stk
        <button data-action="increase" data-index="${index}">+</button>
        <button data-action="remove" data-index="${index}">❌</button>
        <br>
        ${formatPrice(item.price * item.qty)}
      `;
      container.appendChild(div);
    });

    document.getElementById("total-price").innerText = "Samlet pris: " + formatPrice(total);
  }

  async function checkout() {
    if (cart.length === 0) return;

    const order = {
      id: "ORD-" + Math.floor(100000 + Math.random() * 900000),
      name: document.getElementById("customer-name").value,
      total: document.getElementById("total-price").innerText,
      items: cart,
      status: "Afventer",
      date: new Date().toLocaleString()
    };

    await fb.addDoc(fb.collection(db, "orders"), order);

    document.getElementById("checkout-message").innerText = "Ordre oprettet!";

    cart = [];
    saveCart();
    updateCart();
    loadOrders();
  }

  function login() {
    const user = document.getElementById("login-username").value;
    const pass = btoa(document.getElementById("login-password").value);

    const found = admins.find(a => a.username === user && a.password === pass);

    if (found) {
      currentAdmin = user;
      document.getElementById("login-box").style.display = "none";
      document.getElementById("admin-panel").style.display = "block";
      loadOrders();
    } else {
      document.getElementById("login-message").innerText = "Forkert login";
    }
  }

  function logout() {
    currentAdmin = null;
    document.getElementById("admin-panel").style.display = "none";
    document.getElementById("login-box").style.display = "block";
  }

  async function loadOrders() {
    const snapshot = await fb.getDocs(fb.collection(db, "orders"));
    const container = document.getElementById("orders-list");
    container.innerHTML = "";

    snapshot.forEach(docSnap => {
      const order = docSnap.data();
      const id = docSnap.id;

      let itemsHtml = "";
      order.items.forEach(item => {
        itemsHtml += `${item.name} (${item.qty} stk) - ${formatPrice(item.price * item.qty)}<br>`;
      });

      const div = document.createElement("div");
      div.className = "order-box";
      div.innerHTML = `
        <strong>${order.id}</strong><br>
        <strong>Kunde:</strong> ${order.name || "Ukendt"}<br>
        <strong>Varer:</strong><br>
        ${itemsHtml}
        <strong>Total:</strong> ${order.total}<br>
        <button data-id="${id}" class="delete-order">🗑️ Slet ordre</button>
        <br>Dato: ${order.date}
      `;
      container.appendChild(div);
    });
  }

  document.body.addEventListener("click", async function(e) {

    if (e.target.classList.contains("add-to-cart")) {
      addToCart(e.target.closest(".product"));
    }

    if (e.target.dataset.action) {
      const index = parseInt(e.target.dataset.index);
      if (e.target.dataset.action === "increase") cart[index].qty++;
      if (e.target.dataset.action === "decrease") {
        if (cart[index].qty > 1) cart[index].qty--;
        else cart.splice(index, 1);
      }
      if (e.target.dataset.action === "remove") cart.splice(index, 1);
      saveCart(); updateCart();
    }

    if (e.target.classList.contains("delete-order")) {
      const id = e.target.dataset.id;
      await fb.deleteDoc(fb.doc(db, "orders", id));
      loadOrders();
    }

    if (e.target.id === "checkout-btn") checkout();
    if (e.target.id === "login-btn") login();
    if (e.target.id === "logout-btn") logout();
  });

  updateCart();
});
</script>
