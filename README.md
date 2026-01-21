<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Admin Panel – SmartTest</title>

<style>
*{box-sizing:border-box}

body{
    margin:0;
    padding:12px;
    font-family:Tahoma,Arial,sans-serif;
    background:#f2f5f7;
    color:#000;
}

.container{
    max-width:100%;
    margin:auto;
}

h2,h3{
    text-align:center;
    margin:10px 0;
    color:#000;
}

input{
    width:100%;
    padding:12px;
    margin:6px 0;
    border-radius:8px;
    border:1px solid #ccc;
    font-size:16px;
    color:#000;
    background:#fff;
}

button{
    width:100%;
    padding:12px;
    margin:6px 0;
    border:none;
    border-radius:8px;
    background:#1976d2;
    color:#fff;
    font-size:16px;
    font-weight:bold;
    cursor:pointer;
}

button:hover{background:#1565c0}

table{
    width:100%;
    border-collapse:collapse;
    margin-top:12px;
    background:#fff;
    border-radius:10px;
    overflow:hidden;
    font-size:14px;
}

th,td{
    padding:8px;
    border-bottom:1px solid #ddd;
    text-align:center;
    color:#000;
}

th{
    background:#e3e7ea;
    font-weight:bold;
}

.actions{
    display:flex;
    flex-direction:column;
    gap:4px;
}

.actions button{
    padding:8px;
    font-size:14px;
}

button.copy{background:#2e7d32}
button.copy:hover{background:#1b5e20}

button.disable{background:#fb8c00}
button.disable:hover{background:#ef6c00}

button.delete{background:#e53935}
button.delete:hover{background:#c62828}

@media (min-width:768px){
    .actions{flex-direction:row}
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
  const r = await fetch(API+"/admin/licenses",{
    headers:{"x-admin-key":secret.value}
  });
  const d = await r.json();
  render(d);
}

async function create(){
  const r = await fetch(API+"/admin/create",{
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
  const data = await r.json();
  alert("تم إنشاء المفتاح:\n"+data.license_key);
  load();
}

function daysLeft(exp){
  const now = new Date();
  const end = new Date(exp);
  const diff = Math.ceil((end-now)/(1000*60*60*24));
  return diff>0 ? diff : 0;
}

function copyKey(key){
  navigator.clipboard.writeText(key);
  alert("تم نسخ المفتاح");
}

function render(data){
  let html=`<tr>
    <th>المفتاح</th>
    <th>الأيام المتبقية</th>
    <th>المستخدم</th>
    <th>الحد</th>
    <th>الحالة</th>
    <th>العميل</th>
    <th>إجراءات</th>
  </tr>`;

  data.forEach(l=>{
    html+=`<tr>
      <td>${l.license_key}</td>
      <td>${daysLeft(l.expires_at)}</td>
      <td>${l.used_requests}</td>
      <td>${l.max_requests}</td>
      <td>${l.is_active ? "نشط" : "موقوف"}</td>
      <td>${l.owner || "-"}</td>
      <td class="actions">
        <button class="copy" onclick="copyKey('${l.license_key}')">Copy</button>
        <button onclick="reset('${l.license_key}')">Reset</button>
        <button class="disable" onclick="disable('${l.license_key}')">Disable</button>
        <button class="delete" onclick="del('${l.license_key}')">Delete</button>
      </td>
    </tr>`;
  });
  table.innerHTML=html;
}

async function reset(k){
  await fetch(API+"/admin/reset-device/"+k,{
    method:"POST",
    headers:{"x-admin-key":secret.value}
  });
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
  if(!confirm("هل أنت متأكد من حذف المفتاح؟")) return;
  await fetch(API+"/admin/delete/"+k,{
    method:"DELETE",
    headers:{"x-admin-key":secret.value}
  });
  load();
}
</script>

</body>
</html>