<!DOCTYPE html><html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>🐝 Lucky Bee POS ULTRA</title><!-- Chart.js CDN --><script src="https://cdn.jsdelivr.net/npm/chart.js"></script><style>
body{font-family:Arial;background:#eef2f7;margin:0;padding:0;}
header{background:#111827;color:white;padding:15px;text-align:center;font-size:22px;}
.container{padding:15px;}
.card{background:white;padding:15px;border-radius:10px;margin-bottom:15px;box-shadow:0 3px 8px rgba(0,0,0,0.15);}
input,select{width:95%;padding:10px;margin:5px;border:1px solid #ccc;border-radius:6px;}
button{padding:10px;background:#2563eb;border:none;color:white;border-radius:6px;cursor:pointer;margin:5px;}
button:hover{background:#1d4ed8;}
table{width:100%;border-collapse:collapse;margin-top:10px;}
th,td{border:1px solid #ddd;padding:8px;text-align:left;}
#dashboard,#inventory,#checkout,#sales,#reports{display:none;}
footer{background:#111827;color:white;text-align:center;padding:10px;}
.dashboard{display:grid;grid-template-columns:repeat(2,1fr);gap:10px;}
.dashboard button{font-size:16px;padding:15px;}
</style></head>
<body><header>🐝 Lucky Bee POS ULTRA</header><div class="container"><!-- Login --><div id="login" class="card">
<h3>Login</h3>
<input id="username" placeholder="Username">
<input id="password" type="password" placeholder="Password">
<button onclick="login()">Login</button>
<p>Admin / 1234<br>Cashier / 1111</p>
</div><!-- Dashboard --><div id="dashboard" class="card">
<h3>Dashboard</h3>
<div class="dashboard">
<button onclick="openInventory()">Inventory</button>
<button onclick="openCheckout()">Checkout</button>
<button onclick="openSales()">Sales</button>
<button onclick="openReports()">Reports</button>
<button onclick="downloadCSV()">Download CSV</button>
<button onclick="logout()">Logout</button>
</div>
</div><!-- Inventory --><div id="inventory" class="card">
<h3>Inventory Manager</h3>
<input id="name" placeholder="Product name">
<input id="price" placeholder="Price (K)">
<input id="qty" placeholder="Quantity">
<select id="sellable">
<option value="true">Sellable</option>
<option value="false">Non-Sellable</option>
</select>
<button onclick="addProduct()">Add Product</button>
<table id="inventoryTable">
<tr><th>Product</th><th>Price</th><th>Qty</th><th>Sellable</th><th>Delete</th></tr>
</table>
</div><!-- Checkout --><div id="checkout" class="card">
<h3>Checkout</h3>
<select id="productList"></select>
<input id="sellQty" placeholder="Quantity">
<button onclick="checkout()">Sell Product</button>
<button onclick="printReceipt()">Print Receipt</button>
</div><!-- Sales History --><div id="sales" class="card">
<h3>Sales History</h3>
<table id="salesTable">
<tr><th>Product</th><th>Qty</th><th>Total (K)</th></tr>
</table>
</div><!-- Reports --><div id="reports" class="card">
<h3>Sales & Revenue Charts</h3>
<canvas id="salesChart"></canvas>
<canvas id="revenueChart" style="margin-top:20px;"></canvas>
</div></div><footer>All Rights Reserved © Lucky Bee</footer><script>
// Users
let users={admin:"1234",cashier:"1111"}

// Data
let inventory=JSON.parse(localStorage.getItem("inventory"))||[]
let sales=JSON.parse(localStorage.getItem("sales"))||[]

function save(){localStorage.setItem("inventory",JSON.stringify(inventory));localStorage.setItem("sales",JSON.stringify(sales));}

// Login
function login(){
let u=document.getElementById("username").value.toLowerCase().trim()
let p=document.getElementById("password").value.trim()
if(users[u] && users[u]==p){
document.getElementById("login").style.display="none"
document.getElementById("dashboard").style.display="block"
loadInventory();loadProducts();loadSales()
}else{alert("Wrong login")}
}

// Logout
function logout(){location.reload()}

// Hide all sections
function hideAll(){document.getElementById("inventory").style.display="none";document.getElementById("checkout").style.display="none";document.getElementById("sales").style.display="none";document.getElementById("reports").style.display="none";}

// Open sections
function openInventory(){hideAll();document.getElementById("inventory").style.display="block"}
function openCheckout(){hideAll();document.getElementById("checkout").style.display="block"}
function openSales(){hideAll();document.getElementById("sales").style.display="block"}
function openReports(){hideAll();document.getElementById("reports").style.display="block";drawCharts()}

// Inventory functions
function addProduct(){
let name=document.getElementById("name").value
let price=document.getElementById("price").value
let qty=document.getElementById("qty").value
let sellable=document.getElementById("sellable").value=="true"
if(name==""||price==""||qty==""){alert("Fill all fields");return}
inventory.push({name:name,price:parseFloat(price),qty:parseInt(qty),sellable:sellable})
save();loadInventory();loadProducts()
}

function loadInventory(){
let table=document.getElementById("inventoryTable")
table.innerHTML="<tr><th>Product</th><th>Price</th><th>Qty</th><th>Sellable</th><th>Delete</th></tr>"
inventory.forEach((p,i)=>{
let row=table.insertRow()
row.insertCell(0).innerHTML=p.name
row.insertCell(1).innerHTML="K "+p.price
row.insertCell(2).innerHTML=p.qty
row.insertCell(3).innerHTML=p.sellable?"Yes":"No"
row.insertCell(4).innerHTML="<button onclick='deleteItem("+i+")'>Delete</button>"
})
}

function deleteItem(i){inventory.splice(i,1);save();loadInventory();loadProducts()}

// Checkout
function loadProducts(){
let select=document.getElementById("productList")
select.innerHTML=""
inventory.forEach((p,i)=>{if(p.sellable){let opt=document.createElement("option");opt.value=i;opt.text=p.name+" (K"+p.price+")";select.appendChild(opt)}})
}

function checkout(){
let id=document.getElementById("productList").value
let qty=parseInt(document.getElementById("sellQty").value)
let item=inventory[id]
if(qty>item.qty){alert("Not enough stock");return}
item.qty-=qty
let total=item.price*qty
sales.push({name:item.name,qty:qty,total:total})
save();loadInventory();loadSales()
alert("Sale completed")
}

// Sales
function loadSales(){
let table=document.getElementById("salesTable")
table.innerHTML="<tr><th>Product</th><th>Qty</th><th>Total (K)</th></tr>"
sales.forEach(s=>{
let row=table.insertRow()
row.insertCell(0).innerHTML=s.name
row.insertCell(1).innerHTML=s.qty
row.insertCell(2).innerHTML="K "+s.total
})
}

// CSV & Receipt
function downloadCSV(){
let csv="Product,Quantity,Total\n"
sales.forEach(s=>{csv+=s.name+","+s.qty+","+s.total+"\n"})
let blob=new Blob([csv],{type:"text/csv"})
let url=URL.createObjectURL(blob)
let a=document.createElement("a");a.href=url;a.download="luckybee_sales.csv";a.click()
}

function printReceipt(){
let receipt="<h2>Lucky Bee Shop</h2><hr>"
sales.slice(-1).forEach(s=>{receipt+="<p>"+s.name+" x"+s.qty+" = K"+s.total+"</p>"})
let w=window.open();w.document.write(receipt);w.print()
}

// Charts
function drawCharts(){
let productCount={}, revenueCount={}
sales.forEach(s=>{productCount[s.name]=(productCount[s.name]||0)+s.qty; revenueCount[s.name]=(revenueCount[s.name]||0)+s.total})
let ctx1=document.getElementById("salesChart").getContext("2d")
let ctx2=document.getElementById("revenueChart").getContext("2d")
new Chart(ctx1,{type:"bar",data:{labels:Object.keys(productCount),datasets:[{label:"Units Sold",data:Object.values(productCount),backgroundColor:"#2563eb"}]}})
new Chart(ctx2,{type:"bar",data:{labels:Object.keys(revenueCount),datasets:[{label:"Revenue (K)",data:Object.values(revenueCount),backgroundColor:"#f59e0b"}]}})
}
</script></body>
</html>
