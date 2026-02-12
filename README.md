<!DOCTYPE html>
<html lang="da">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>GunPlug</title>

<style>
body { font-family: Arial, sans-serif; margin: 0; background-color: #f4f4f4; }
header { background: #222; color: white; padding: 20px; text-align: center; }
main { padding: 20px; max-width: 1100px; margin: auto; background: white; margin-top: 20px; border-radius: 10px; }
.products { display: grid; grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); gap: 20px; }
.product { border: 1px solid #ddd; border-radius: 10px; padding: 15px; text-align: center; }
.product img { width: 100%; height: 200px; object-fit: cover; border-radius: 8px; }
.cart, .checkout, .login-box, .admin-panel { margin-top: 30px; padding: 20px; border: 1px solid #ccc; border-radius: 10px; }
.cart-item { margin-bottom: 10px; }
.total { font-weight: bold; font-size: 18px; }
input, select { width: 100%; padding: 8px; margin-bottom: 10px; }
button { padding: 8px 12px; cursor: pointer; margin-top: 5px; }
.admin-panel { display: none; border: 2px solid #222; }
.order-box { border-bottom: 1px solid #ccc; padding: 10px 0; }
</style>

<!-- Firebase -->
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

</head>
<body>

<header>
<h1>Bestilling</h1>
</header>

<main>

<h2>Produkter</h2>
<div class="products">

<div class="product">
<img src="https://i.gyazo.com/148d227b058218eac9f8a097c5453ec9.png">
<h3>Deagle</h3>
<p class="price">Pris: 1.800.000 kr</p>
<button class="add-to-cart">Læg i kurv</button>
</div>

<div class="product">
<img src="https://i.gyazo.com/fe27523c5e0cbcd06ab5d0c3c38d637a.png">
<h3>Pistol</h3>
<p class="price">Pris: 1.100.000 kr</p>
<button class="add-to-cart">Læg i kurv</button>
</div>

<div class="product">
<img src="https://i.gyazo.com/ab625f21a8129f2578c8d4431355a0a4.png">
<h3>SNS</h3>
<p class="price">Pris: 800.000 kr</p>
<button class="add-to-cart">Læg i kurv</button>
</div>

</div>

<div class="cart">
<h3>🛒 Kurv</h3>
<div id="cart-items"></div>
<div class="total" id="total-price">Samlet pris: 0 kr</div>

<div class="checkout">
<h3>📦 Checkout</h3>
<input type="text" id="customer-name" placeholder="Ingame Telefonnummer">
<button id="checkout-btn">Bestil</button>
<p id="checkout-message"></p>
</div>
</div>

<div class="login-box" id="login-box">
<h3>🔐 Admin Login</h3>
<input type="text" id="login-username" placeholder="Brugernavn">
<input type="password" id="login-password" placeholder="Adgangskode">
<button id="login-btn">Login</button>
<p id="login-message"></p>
</div>

<div class="admin-panel" id="admin-panel">
<h3>Admin Panel</h3>
<button id="logout-btn">Log ud</button>
<h4>📦 Ordrer</h4>
<div id="orders-list"></div>
</div>

</main>

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
container.innerHTML += `
<div class="cart-item">
<strong>${item.name}</strong><br>
<button data-action="decrease" data-index="${index}">-</button>
${item.qty} stk
<button data-action="increase" data-index="${index}">+</button>
<button data-action="remove" data-index="${index}">❌</button>
<br>${formatPrice(item.price * item.qty)}
</div>`;
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

container.innerHTML += `
<div class="order-box">
<strong>${order.id}</strong><br>
<strong>Kunde:</strong> ${order.name}<br>
<strong>Varer:</strong><br>
${itemsHtml}
<strong>Total:</strong> ${order.total}<br>
<button class="delete-order" data-id="${id}">🗑️ Slet ordre</button>
<br>Dato: ${order.date}
</div>`;
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
await fb.deleteDoc(fb.doc(db, "orders", e.target.dataset.id));
loadOrders();
}

if (e.target.id === "checkout-btn") checkout();
if (e.target.id === "login-btn") login();
if (e.target.id === "logout-btn") logout();
});

updateCart();
});
</script>

</body>
</html>
