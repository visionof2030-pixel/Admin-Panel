<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<title>Admin Panel – SmartTest</title>
<style>
body{font-family:Tahoma;background:#0f3c4c;color:#fff;padding:20px}
input,button{padding:8px;margin:4px}
table{width:100%;border-collapse:collapse;margin-top:20px}
td,th{border:1px solid #ccc;padding:6px;text-align:center}
button{cursor:pointer}
</style>
</head>
<body>

<h2>🔐 Admin Panel</h2>

<input id="secret" placeholder="ADMIN SECRET">
<button onclick="load()">تحميل المفاتيح</button>

<h3>➕ إنشاء مفتاح</h3>
<input id="days" placeholder="الأيام" value="30">
<input id="max" placeholder="عدد الطلبات" value="1000">
<input id="owner" placeholder="اسم العميل">
<button onclick="create()">إنشاء</button>

<table id="table"></table>

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
  let html="<tr><th>Key</th><th>Used</th><th>Max</th><th>Active</th><th>Owner</th><th>Actions</th></tr>";
  data.forEach(l=>{
    html+=`<tr>
      <td>${l.license_key}</td>
      <td>${l.used_requests}</td>
      <td>${l.max_requests}</td>
      <td>${l.is_active}</td>
      <td>${l.owner}</td>
      <td>
        <button onclick="reset('${l.license_key}')">Reset</button>
        <button onclick="disable('${l.license_key}')">Disable</button>
        <button onclick="del('${l.license_key}')">Delete</button>
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