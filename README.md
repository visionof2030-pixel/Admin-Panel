<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<title>Admin Panel – SmartTest</title>

<style>
body{
    font-family:Tahoma,Arial,sans-serif;
    background:#0b1f2a;
    color:#eaf6ff;
    padding:20px;
}

h2,h3{
    color:#4dd0e1;
}

input{
    padding:10px;
    margin:6px 0;
    border-radius:6px;
    border:none;
    width:100%;
    background:#102a36;
    color:#fff;
}

input::placeholder{
    color:#9fbfcf;
}

button{
    padding:10px 14px;
    margin:6px 4px;
    border:none;
    border-radius:6px;
    background:#1e88e5;
    color:#fff;
    font-weight:bold;
    cursor:pointer;
}

button:hover{
    background:#1565c0;
}

table{
    width:100%;
    border-collapse:collapse;
    margin-top:20px;
    background:#102a36;
    border-radius:8px;
    overflow:hidden;
}

th{
    background:#0d3b4f;
    color:#4dd0e1;
    padding:10px;
}

td{
    padding:8px;
    border-bottom:1px solid #1c4b63;
}

tr:nth-child(even){
    background:#0f3446;
}

tr:hover{
    background:#144a63;
}

.actions button{
    background:#26a69a;
}

.actions button:hover{
    background:#1f8f85;
}

button.delete{
    background:#e53935;
}

button.delete:hover{
    background:#c62828;
}

button.disable{
    background:#fb8c00;
}

button.disable:hover{
    background:#ef6c00;
}

.container{
    max-width:1100px;
    margin:auto;
}
</style>
</head>

<body>
<div class="container">

<h2>🔐 لوحة التحكم – SmartTest</h2>

<input id="secret" placeholder="ADMIN SECRET">
<button onclick="load()">تحميل المفاتيح</button>

<h3>➕ إنشاء مفتاح جديد</h3>
<input id="days" placeholder="مدة الاشتراك بالأيام" value="30">
<input id="max" placeholder="عدد الطلبات" value="1000">
<input id="owner" placeholder="اسم العميل">
<button onclick="create()">إنشاء المفتاح</button>

<table id="table"></table>

</div>

<script>
const API = "https://batching-project.onrender.com";

async function load(){
  const r = await fetch(API+"/admin/licenses",{headers:{"x-admin-key":secret.value}});
  const d = await r.json();
  render(d);
}

async function create(){
  await fetch(API+"/admin/create",{
    method:"POST",
    headers:{
      "Content-Type":"application/json",
      "x-admin-key":secret.value
    },
    body:JSON.stringify({
      days:Number(days.value),
      max_requests:Number(max.value),
      owner:owner.value
    })
  });
  load();
}

function render(data){
  let html="<tr><th>المفتاح</th><th>المستخدم</th><th>الحد</th><th>الحالة</th><th>العميل</th><th>إجراءات</th></tr>";
  data.forEach(l=>{
    html+=`<tr>
      <td>${l.license_key}</td>
      <td>${l.used_requests}</td>
      <td>${l.max_requests}</td>
      <td>${l.is_active ? "✅ نشط" : "❌ موقوف"}</td>
      <td>${l.owner || "-"}</td>
      <td class="actions">
        <button onclick="reset('${l.license_key}')">Reset</button>
        <button class="disable" onclick="disable('${l.license_key}')">Disable</button>
        <button class="delete" onclick="del('${l.license_key}')">Delete</button>
      </td>
    </tr>`;
  });
  table.innerHTML=html;
}

async function reset(k){
  await fetch(API+"/admin/reset-device/"+k,{method:"POST",headers:{"x-admin-key":secret.value}});
  load();
}
async function disable(k){
  await fetch(API+"/admin/update/"+k,{
    method:"PUT",
    headers:{
      "Content-Type":"application/json",
      "x-admin-key":secret.value
    },
    body:JSON.stringify({is_active:false})
  });
  load();
}
async function del(k){
  await fetch(API+"/admin/delete/"+k,{method:"DELETE",headers:{"x-admin-key":secret.value}});
  load();
}
</script>

</body>
</html>