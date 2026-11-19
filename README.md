<!DOCTYPE html>
<html lang="sw">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>TUMA - Student Election</title>
<style>
*{box-sizing:border-box}body{margin:0;font-family:'Segoe UI',Arial;background:linear-gradient(135deg,#0B2C4A,#123F66);min-height:100vh;color:#333}
.topbar{background:#0B2C4A;color:#fff;padding:15px 20px;display:flex;justify-content:space-between;align-items:center}
.card{background:#fff;max-width:500px;margin:25px auto;padding:28px;border-radius:20px;box-shadow:0 10px 30px rgba(0,0,0,.25)}
.hero{height:280px;position:relative;overflow:hidden}.hero img{width:100%;height:100%;object-fit:cover}
.hero .overlay{position:absolute;left:0;right:0;bottom:0;background:rgba(0,0,0,.7);color:#fff;padding:15px;text-align:center}
.hero .overlay h1{margin:0;color:#F2B705;font-size:32px}input{width:100%;padding:13px;margin:8px 0;border:1px solid #ddd;border-radius:12px}
button{background:#F2B705;color:#0B2C4A;padding:14px;border:none;border-radius:12px;width:100%;font-weight:800;cursor:pointer;font-size:16px;margin-top:8px}
button.danger{background:#e74c3c;color:#fff}button.secondary{background:#0B2C4A;color:#fff}
.candidate-card{background:#fff;border-radius:16px;padding:15px;text-align:center;box-shadow:0 4px 15px rgba(0,0,0,.12);margin:12px 0}
.candidate-card img{width:140px;height:140px;border-radius:50%;object-fit:cover;border:4px solid #0B2C4A}
.candidate-card .sela{background:#FFF3CD;padding:6px;border-radius:8px;font-size:13px}.container{max-width:700px;margin:20px auto;padding:0 15px}
.admin-stats{background:#fff;padding:20px;border-radius:16px}table{width:100%;border-collapse:collapse;background:#fff;border-radius:12px;overflow:hidden}
td,th{padding:10px;border-bottom:1px solid #eee;text-align:left;font-size:14px}.center{text-align:center}.hidden{display:none}
</style>
</head>
<body>

<div id="page-login">
<div class="hero">
<img src="images/tuma-campus.jpg" onerror="this.src='https://images.unsplash.com/photo-1541339907198-e08756dedf3f?w=1000'">
<div class="overlay"><h1>TUMA - Student Election</h1><p>Tumaini University Makumira | Chagua Kiongozi Wako</p></div>
</div>
<div class="card">
<h2 class="center">INGIA KUPIGA KURA</h2>
<p class="center" style="font-size:13px;color:#666">Tumia Registration Number uliyopewa na Admin</p>
<input id="regNo" placeholder="Reg No e.g TUMA001 / ADMIN001">
<input id="password" type="password" placeholder="Password">
<button onclick="login()">INGIA</button>
<p class="center" id="status" style="font-size:13px;color:#0B2C4A">Inaunganisha na Firebase...</p>
<p class="center" style="font-size:13px">Jaribu: <b>TUMA001 / 12345</b> | Admin: <b>ADMIN001 / admin123</b></p>
</div>
</div>

<div id="page-ballot" class="hidden">
<div class="topbar"><b>TUMA VOTE</b><button style="width:auto;padding:8px 15px" onclick="logout()">Toka</button></div>
<div class="container">
<div class="card"><h2 class="center">Karibu, <span id="sName"></span></h2><p class="center">Chagua mgombea wako. Kura yako ni <b>SIRI</b>.</p></div>
<div id="candidatesList"></div>
</div>
</div>

<div id="page-admin" class="hidden">
<div class="topbar"><b>ADMIN DASHBOARD</b><button style="width:auto;padding:8px 15px" onclick="logout()">Toka</button></div>
<div class="container">
<div class="admin-stats">
<h2>Matokeo - Admin Tu</h2>
<p><b>Wanafunzi Waliosajiliwa:</b> <span id="totalVoters">0</span></p>
<p><b>Wameshapiga Kura:</b> <span id="totalVoted">0</span></p>
<p><b>Idadi ya Admin:</b> <span id="totalAdmins">0</span></p>
<div id="results"></div>
<button class="danger" onclick="resetElection()">RESET UCHAGUZI MZIMA - Anza Upya</button>
</div>
<div class="card"><h3>Ongeza Mwanafunzi Mpya</h3>
<input id="newReg" placeholder="Reg No e.g TUMA010"><input id="newName" placeholder="Jina Kamili">
<button onclick="addVoter()">Ongeza Mwanafunzi</button></div>
<div class="card"><h3>Ongeza Admin Mpya</h3>
<input id="adminReg" placeholder="Admin Reg e.g ADMIN016"><input id="adminName" placeholder="Jina la Admin">
<button class="secondary" onclick="addAdmin()">Ongeza Admin</button></div>
<h3 style="color:white">Orodha ya Wanafunzi</h3>
<table><thead><tr><th>Reg No</th><th>Jina</th><th>Amepiga?</th><th>Futa</th></tr></thead><tbody id="voterList"></tbody></table>
</div>
</div>

<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";
import { getFirestore, collection, addDoc, getDocs, deleteDoc, doc, query, where, updateDoc } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-firestore.js";

const firebaseConfig = {
  apiKey: "AIzaSyAq6j45UWoNkWCCRHyybz4gi4v_4pTHMFk",
  authDomain: "tuma-vote.firebaseapp.com",
  projectId: "tuma-vote",
  storageBucket: "tuma-vote.firebasestorage.app",
  messagingSenderId: "571802402269",
  appId: "1:571802402269:web:afe1c30fb7b651d68d0a34",
  measurementId: "G-3XWXS3LNC8"
};
const app = initializeApp(firebaseConfig);
const db = getFirestore(app);

let currentUserDocId = null;
let currentVoter = null;

function show(id){['page-login','page-ballot','page-admin'].forEach(p=>document.getElementById(p).classList.add('hidden'));document.getElementById(id).classList.remove('hidden');window.scrollTo(0,0)}
window.logout = ()=>{ currentUserDocId=null; currentVoter=null; show('page-login'); }

async function seedIfEmpty(){
  try{
    let adminsSnap = await getDocs(collection(db,"admins"));
    if(adminsSnap.empty){
      for(let i=1;i<=15;i++){
        let n=String(i).padStart(3,'0');
        await addDoc(collection(db,"admins"),{regNo:"ADMIN"+n,name:"Admin "+i,password:"admin123"});
      }
    }
    let votersSnap = await getDocs(collection(db,"voters"));
    if(votersSnap.empty){
      await addDoc(collection(db,"voters"),{regNo:"TUMA001",name:"Juma Mushi",password:"12345",hasVoted:false});
      await addDoc(collection(db,"voters"),{regNo:"TUMA002",name:"Neema Joseph",password:"12345",hasVoted:false});
      await addDoc(collection(db,"voters"),{regNo:"TUMA003",name:"Baraka Mushi",password:"12345",hasVoted:false});
    }
    let candSnap = await getDocs(collection(db,"candidates"));
    if(candSnap.empty){
      await addDoc(collection(db,"candidates"),{name:"Neema Joseph",position:"President",sela:"TUMA / BIT 2",manifesto:"Elimu bora na huduma za wanafunzi",photo:"images/neema.jpg"});
      await addDoc(collection(db,"candidates"),{name:"Baraka Mushi",position:"President",sela:"TUMA / BIT 2",manifesto:"Uwazi na maendeleo ya chuo",photo:"images/baraka.jpg"});
      await addDoc(collection(db,"candidates"),{name:"Grace Samwel",position:"Vice President",sela:"TUMA / BIT 1",manifesto:"Michezo na afya ya wanafunzi",photo:"images/default.jpg"});
    }
    document.getElementById('status').textContent="Imeunganishwa! Unaweza kuingia sasa.";
  }catch(e){ document.getElementById('status').textContent="Kosa: "+e.message; }
}
seedIfEmpty();

window.login = async function(){
  let reg=document.getElementById('regNo').value.trim().toUpperCase();
  let pass=document.getElementById('password').value.trim();
  if(!reg||!pass) return alert("Jaza Reg No na Password");
  document.getElementById('status').textContent="Inaangalia...";
  let q=query(collection(db,"admins"),where("regNo","==",reg),where("password","==",pass));
  let snap=await getDocs(q);
  if(!snap.empty){ await loadAdmin(); show('page-admin'); return; }
  let q2=query(collection(db,"voters"),where("regNo","==",reg),where("password","==",pass));
  let snap2=await getDocs(q2);
  if(snap2.empty){ alert("Hujasajiliwa! Wasiliana na Admin akusajili kwanza."); return; }
  let voter=snap2.docs[0].data();
  if(voter.hasVoted){ alert("Umeshapiga kura tayari! Asante."); return; }
  currentUserDocId=snap2.docs[0].id; currentVoter=voter;
  document.getElementById('sName').textContent=voter.name;
  let cSnap=await getDocs(collection(db,"candidates"));
  let cands=cSnap.docs.map(d=>({id:d.id,...d.data()}));
  document.getElementById('candidatesList').innerHTML=cands.map(c=>`
  <div class="candidate-card">
  <img src="${c.photo}" onerror="this.src='https://i.pravatar.cc/150?u=${c.name}'">
  <h3>${c.name}</h3><div style="color:#666">${c.position}</div>
  <p>${c.manifesto}</p><div class="sela">Sela: ${c.sela}</div><br>
  <button onclick="doVote('${c.id}','${c.name}')">Mpigie Kura ${c.name}</button>
  </div>`).join('');
  show('page-ballot');
}

window.doVote = async function(cid,cname){
  if(!confirm("Unampigia "+cname+"? Hutoweza kubadilisha.")) return;
  await addDoc(collection(db,"votes"),{voterRegNo:currentVoter.regNo,candidateId:cid,time:new Date().toLocaleString()});
  await updateDoc(doc(db,"voters",currentUserDocId),{hasVoted:true});
  alert("Asante "+currentVoter.name+"! Kura yako imepokelewa na ni SIRI.");
  show('page-login');
}

window.addVoter = async function(){
  let reg=document.getElementById('newReg').value.trim().toUpperCase();
  let name=document.getElementById('newName').value.trim();
  if(!reg||!name) return alert("Jaza Reg No na Jina");
  await addDoc(collection(db,"voters"),{regNo:reg,name:name,password:"12345",hasVoted:false});
  alert("Ameongezwa! Sasa hata akifungua simu yake ataingia. Reg: "+reg+" Pass: 12345");
  loadAdmin();
}

window.deleteVoter = async function(docId,regNo){
  if(!confirm("Mfute "+regNo+"? Kura yake itafutika pia.")) return;
  await deleteDoc(doc(db,"voters",docId));
  let vSnap=await getDocs(query(collection(db,"votes"),where("voterRegNo","==",regNo)));
  for(let d of vSnap.docs) await deleteDoc(doc(db,"votes",d.id));
  alert("Amefutwa kabisa na kura yake imefutika.");
  loadAdmin();
}

window.resetElection = async function(){
  if(!confirm("Kufuta KURA ZOTE na kuanza upya?")) return;
  let vSnap=await getDocs(collection(db,"votes"));
  for(let d of vSnap.docs) await deleteDoc(doc(db,"votes",d.id));
  let votersSnap=await getDocs(collection(db,"voters"));
  for(let d of votersSnap.docs) await updateDoc(doc(db,"voters",d.id),{hasVoted:false});
  alert("Uchaguzi umeanzishwa upya!");
  loadAdmin();
}

window.addAdmin = async function(){
  let reg=document.getElementById('adminReg').value.trim().toUpperCase();
  let name=document.getElementById('adminName').value.trim();
  if(!reg||!name) return alert("Jaza Reg na Jina la Admin");
  await addDoc(collection(db,"admins"),{regNo:reg,name:name,password:"admin123"});
  alert("Admin "+name+" ameongezwa.");
  loadAdmin();
}

window.loadAdmin = async function(){
  let votersSnap=await getDocs(collection(db,"voters"));
  let votesSnap=await getDocs(collection(db,"votes"));
  let candsSnap=await getDocs(collection(db,"candidates"));
  let adminsSnap=await getDocs(collection(db,"admins"));
  let voters=votersSnap.docs.map(d=>({docId:d.id,...d.data()}));
  let votes=votesSnap.docs.map(d=>d.data());
  let cands=candsSnap.docs.map(d=>({id:d.id,...d.data()}));
  document.getElementById('totalVoters').textContent=voters.length;
  document.getElementById('totalVoted').textContent=voters.filter(v=>v.hasVoted).length;
  document.getElementById('totalAdmins').textContent=adminsSnap.size;
  document.getElementById('results').innerHTML=cands.map(c=>{
    let count=votes.filter(v=>v.candidateId===c.id).length;
    return <p><b>${c.name} (${c.position}):</b> ${count} kura</p><div style="background:#eee;height:12px;border-radius:10px;overflow:hidden"><div style="width:${votes.length?(count/votes.length*100):0}%;background:#F2B705;height:12px"></div></div>;
  }).join('');
  document.getElementById('voterList').innerHTML=voters.map(v=><tr><td>${v.regNo}</td><td>${v.name}</td><td>${v.hasVoted?'✅ Ndio':'❌ Bado'}</td><td><button class="danger" style="padding:6px" onclick="deleteVoter('${v.docId}','${v.regNo}')">Futa</button></td></tr>).join('');
}
</script>
</body>
</html>
