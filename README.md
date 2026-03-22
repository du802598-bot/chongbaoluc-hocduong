<!DOCTYPE html>
<html lang="vi">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Anti School Violence - Pro</title>

<link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;500;700&display=swap" rel="stylesheet">

<script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-firestore-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-auth-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-storage-compat.js"></script>

<style>
*{margin:0;padding:0;box-sizing:border-box;font-family:Poppins}
body{background:#f4f7fb}

/* NAV */
nav{
position:fixed;width:100%;top:0;background:white;
display:flex;justify-content:space-between;
padding:15px 10%;box-shadow:0 2px 10px rgba(0,0,0,.1);
z-index:999;
}

/* HERO */
.hero{
height:100vh;
background:url('https://images.unsplash.com/photo-1588072432836-e10032774350') center/cover;
display:flex;align-items:center;justify-content:center;
color:white;text-align:center;position:relative;
}
.hero::after{
content:"";position:absolute;width:100%;height:100%;
background:rgba(0,0,0,.5);
}
.hero-content{position:relative;z-index:2}
.hero h1{font-size:50px}

/* SECTION */
.section{padding:80px 10%}
.grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(250px,1fr));gap:20px}

/* CARD */
.card{
background:white;padding:20px;border-radius:10px;
box-shadow:0 5px 15px rgba(0,0,0,.1);
}
.card img, video{width:100%;border-radius:10px}

/* FORM */
input,textarea{width:100%;margin:10px 0;padding:10px}
button{padding:10px;background:#007bff;color:white;border:none}

/* ADMIN */
#adminPanel{
position:fixed;right:0;top:0;width:350px;height:100%;
background:#111;color:white;padding:20px;overflow:auto;
}
</style>
</head>

<body>

<nav>
<h2>NO VIOLENCE</h2>
<button onclick="toggleAdmin()">Admin</button>
</nav>

<section class="hero">
<div class="hero-content">
<h1>Nói không với bạo lực học đường</h1>
<p>Môi trường học tập an toàn</p>
</div>
</section>

<section class="section">
<h2>Bài viết</h2>
<div id="posts" class="grid"></div>
</section>

<section class="section">
<h2>Gửi ý kiến</h2>
<form id="form">
<input id="name" placeholder="Tên">
<input id="email" placeholder="Email">
<textarea id="msg"></textarea>
<button>Gửi</button>
</form>
</section>

<!-- ADMIN PANEL -->
<div id="adminPanel" style="display:none">
<h2>Admin</h2>

<input id="email" placeholder="Email">
<input id="password" type="password">
<button onclick="login()">Login</button>

<hr>

<h3>Đăng bài</h3>
<input id="title" placeholder="Tiêu đề">
<textarea id="content"></textarea>

<input type="file" id="image">
<input type="file" id="video">

<button onclick="uploadPost()">Đăng</button>

<hr>

<div id="adminList"></div>

</div>

<script>

// 🔥 FIREBASE CONFIG (DÁN CỦA BẠN)
const firebaseConfig = {
  apiKey: "YOUR_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  projectId: "YOUR_PROJECT",
  storageBucket: "YOUR_PROJECT.appspot.com"
};

firebase.initializeApp(firebaseConfig);

const db = firebase.firestore();
const auth = firebase.auth();
const storage = firebase.storage();

// UI
function toggleAdmin(){
document.getElementById("adminPanel").style.display="block";
}

// LOGIN
function login(){
auth.signInWithEmailAndPassword(email.value,password.value)
.then(()=>alert("OK"));
}

// LOAD POSTS
function loadPosts(){
db.collection("posts").onSnapshot(snapshot=>{
let html="";
snapshot.forEach(doc=>{
let d=doc.data();
html+=`
<div class="card">
<h3>${d.title}</h3>
<p>${d.content}</p>
${d.image?`<img src="${d.image}">`:''}
${d.video?`<video controls src="${d.video}"></video>`:''}
</div>
`;
});
posts.innerHTML=html;
});
}
loadPosts();

// UPLOAD POST
async function uploadPost(){

let imgFile=image.files[0];
let vidFile=video.files[0];

let imgURL="", vidURL="";

if(imgFile){
let ref=storage.ref("images/"+imgFile.name);
await ref.put(imgFile);
imgURL=await ref.getDownloadURL();
}

if(vidFile){
let ref=storage.ref("videos/"+vidFile.name);
await ref.put(vidFile);
vidURL=await ref.getDownloadURL();
}

db.collection("posts").add({
title:title.value,
content:content.value,
image:imgURL,
video:vidURL,
time:new Date()
});

alert("Đăng thành công");
}

// FORM USER
form.addEventListener("submit",e=>{
e.preventDefault();

db.collection("messages").add({
name:name.value,
email:email.value,
msg:msg.value
});

alert("Đã gửi!");
});

// ADMIN LIST
db.collection("posts").onSnapshot(snapshot=>{
let html="";
snapshot.forEach(doc=>{
html+=`
<div>
${doc.data().title}
<button onclick="del('${doc.id}')">X</button>
</div>`;
});
adminList.innerHTML=html;
});

function del(id){
db.collection("posts").doc(id).delete();
}

</script>

</body>
</html>
