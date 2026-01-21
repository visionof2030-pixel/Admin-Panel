<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Admin Panel</title>
</head>
<body>

<h2>🔐 Admin Panel</h2>

<input id="secret" placeholder="ADMIN SECRET">
<input id="days" type="number" value="30">
<input id="max" type="number" value="1000">
<button onclick="create()">➕ إنشاء مفتاح</button>

<pre id="out"></pre>

<script>
const API = "https://batching-project.onrender.com";

async function create(){
  const res = await fetch(API+"/admin/create",{
    method:"POST",
    headers:{
      "Content-Type":"application/json",
      "x-admin-key": secret.value
    },
    body:JSON.stringify({
      days: Number(days.value),
      max_requests: Number(max.value)
    })
  });
  out.textContent = await res.text();
}
</script>

</body>
</html>
