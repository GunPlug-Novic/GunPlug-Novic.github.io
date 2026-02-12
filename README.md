
<html lang="da">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>GunPlug</title>

<script src="https://www.gstatic.com/firebasejs/10.12.0/firebase-app.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.0/firebase-firestore.js"></script>

<style>
body { font-family: Arial; background:#f4f4f4; margin:0 }
main { max-width:1000px; margin:auto; background:white; padding:20px; margin-top:20px; border-radius:10px }
.product { border:1px solid #ddd; padding:15px; margin-bottom:10px }
.admin-panel { display:none; margin-top:30px; border:2px solid black; padding:20px }
</style>
</head>
<body>

<main>

<h2>Produkter</h2>

<div class="product">
<h3>Deagle</h3>
<p>Pris: 1.800.000 kr</p>
<button onclick="addToCart('Deagle',1800000)">Læg i kurv</button>
</div>

<div class="product">
<h3>Pistol</h3>
<p>Pris: 1.100.000 kr</p>
<button onclick="addToCart('Pistol',1100000)">Læg i kurv</button>
</div>

<h3>Kurv</h3>
<div id="cart"></div>
<p id="total"></p>

<input type="text" id="customer" placeholder="Telefonnummer">
<button onclick="checkout()">Bestil</button>

<hr>

<h3>Admin Login</h3>
<input type="password" id="adminpass" placeholder="Kode">
<button onclick="login()">Login</button>

<div class="admin-panel" id="admin">
<h3>Ordrer</h3>
<div id="orders"></div>
</div>

</main>

<script type="module">

import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.0/firebase-app.js";
import { getFirestore, collection, addDoc, getDocs, updateDoc, deleteDoc, doc } 
from "https://www.gstatic.com/firebasejs/10.12.0/firebase-firestore.js";

// 🔥 INDSÆT DIN FIREBASE CONFIG HER
// Import the functions you need from the SDKs you need
import { initializeApp } from "firebase/app";
// TODO: Add SDKs for Firebase products that you want to use
// https://firebase.google.com/docs/web/setup#available-libraries

// Your web app's Firebase configuration
const firebaseConfig = {
  apiKey: "AIzaSyDbHH0sAjhr99twzXv3qABC53DGCb8iarg",
  authDomain: "gunplug-cdf00.firebaseapp.com",
  projectId: "gunplug-cdf00",
  storageBucket: "gunplug-cdf00.firebasestorage.app",
  messagingSenderId: "374028571209",
  appId: "1:374028571209:web:c0b70e14dac98114fbe598"
};

// Initialize Firebase
const app = initializeApp(firebaseConfig);

const app = initializeApp(firebaseConfig);
const db = getFirestore(app);

let cart = [];

window.addToCart = function(name, price){
  const existing = cart.find(p => p.name===name);
  if(existing) existing.qty++;
  else cart.push({name,price,qty:1});
  renderCart();
}

function renderCart(){
  const div = document.getElementById("cart");
  div.innerHTML="";
  let total=0;
  cart.forEach(i=>{
    total += i.price*i.qty;
    div.innerHTML += `${i.name} (${i.qty})<br>`;
  });
  document.getElementById("total").innerText="Total: "+total.toLocaleString()+" kr";
}

window.checkout = async function(){
  if(cart.length===0) return;

  await addDoc(collection(db,"orders"),{
    customer: document.getElementById("customer").value,
    items: cart,
    total: cart.reduce((a,b)=>a+b.price*b.qty,0),
    status:"Afventer",
    date:new Date().toLocaleString()
  });

  alert("Ordre oprettet!");
  cart=[];
  renderCart();
  loadOrders();
}

window.login = function(){
  if(document.getElementById("adminpass").value==="2121"){
    document.getElementById("admin").style.display="block";
    loadOrders();
  }
}

async function loadOrders(){
  const querySnapshot = await getDocs(collection(db,"orders"));
  const container=document.getElementById("orders");
  container.innerHTML="";

  querySnapshot.forEach(docSnap=>{
    const o = docSnap.data();
    const id = docSnap.id;

    container.innerHTML += `
      <div style="border-bottom:1px solid #ccc;margin-bottom:10px">
      <strong>${o.customer}</strong><br>
      ${o.items.map(i=>i.name+"("+i.qty+")").join("<br>")}<br>
      Total: ${o.total.toLocaleString()} kr<br>
      Status: ${o.status}
      <button onclick="deleteOrder('${id}')">Slet</button>
      </div>
    `;
  });
}

window.deleteOrder = async function(id){
  await deleteDoc(doc(db,"orders",id));
  loadOrders();
}

</script>

</body>
</html>
