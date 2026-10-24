
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>StudyTrack</title>
<style>
*{box-sizing:border-box}
body{margin:0;font-family:Arial,sans-serif;background:#f1f5f9;color:#172033}
header{background:#172554;color:white;padding:22px}
header h1{margin:0 0 6px}
main{max-width:850px;margin:auto;padding:16px}
.card{background:white;padding:18px;border-radius:16px;margin-bottom:16px;box-shadow:0 3px 12px #0000000d}
h2{font-size:19px;margin-top:0}
input,select,button{padding:11px;border-radius:9px;border:1px solid #cbd5e1;font-size:15px;max-width:100%}
input,select{margin:4px 0}
button{background:#2563eb;color:white;border:0;cursor:pointer;margin:4px}
button.secondary{background:#475569}
.row{display:flex;gap:8px;flex-wrap:wrap}
.stat{display:inline-block;background:#dbeafe;padding:12px;border-radius:10px;margin:4px}
li{margin:10px 0;overflow-wrap:anywhere}
.small{color:#64748b;font-size:13px}
</style>
</head>
<body>
<header>
<h1>📘 StudyTrack</h1>
<div>Your personal study dashboard</div>
</header>

<main>
<section class="card">
<h2>🎯 Daily study goal</h2>
<label>Goal in minutes: <input id="goal" type="number" min="1" value="180"></label>
<div id="goalStatus">Loading progress...</div>
</section>

<section class="card">
<h2>⏱️ Study timer</h2>
<select id="subject">
<option>Physics</option>
<option>Chemistry</option>
<option>Maths</option>
<option>English</option>
<option>Hindi</option>
<option>Other</option>
</select>
<div class="stat">Time: <strong id="clock">00:00:00</strong></div>
<div class="row">
<button id="startBtn" onclick="startTimer()">Start</button>
<button onclick="stopTimer()">Stop & Save</button>
</div>
<p class="small">Keep this page open while the timer runs.</p>
</section>

<section class="card">
<h2>📚 Chapter tracker</h2>
<div class="row">
<input id="chapter" placeholder="Chapter name">
<select id="chapterSubject">
<option>Physics</option><option>Chemistry</option>
<option>Maths</option><option>English</option>
<option>Hindi</option><option>Other</option>
</select>
<button onclick="addChapter()">Add chapter</button>
</div>
<ul id="chapters"></ul>
</section>

<section class="card">
<h2>📝 Test scores</h2>
<div class="row">
<input id="testName" placeholder="Test name">
<input id="marks" type="number" min="0" placeholder="Marks obtained">
<input id="total" type="number" min="1" placeholder="Total marks">
<button onclick="addTest()">Save test</button>
</div>
<div id="testResults"></div>
</section>

<section class="card">
<h2>📊 Analytics</h2>
<div id="analytics"></div>
</section>

<section class="card">
<h2>🔔 Study reminder</h2>
<input id="reminderText" placeholder="What should you study?">
<input id="reminderTime" type="datetime-local">
<button onclick="addReminder()">Set reminder</button>
<ul id="reminders"></ul>
<p class="small">Reminders are checked while this page is open. Browser notification permission may be required.</p>
</section>

<section class="card">
<h2>🤖 Study assistant</h2>
<p id="aiAdvice">Your study suggestions will appear here.</p>
<button onclick="getAdvice()">Get study advice</button>
</section>

<section class="card">
<h2>💾 Your data</h2>
<button onclick="exportData()">Export backup</button>
<button class="secondary" onclick="clearData()">Clear all data</button>
<p class="small">This starter version stores data in this browser on this device. It does not yet have cloud accounts or a real AI model.</p>
</section>
</main>

<script>
const KEY="studytrack_v1";
let data;
try {
  data=JSON.parse(localStorage.getItem(KEY)) ||
    {chapters:[],tests:[],sessions:[],reminders:[],goal:180};
} catch(e) {
  data={chapters:[],tests:[],sessions:[],reminders:[],goal:180};
}
let startedAt=null, tick=null;

function save(){
  localStorage.setItem(KEY,JSON.stringify(data));
  render();
}
function today(){
  return new Date().toLocaleDateString();
}
function todayMinutes(){
  return data.sessions.filter(s=>s.date===today())
    .reduce((sum,s)=>sum+s.minutes,0);
}
function startTimer(){
  if(startedAt) return;
  startedAt=Date.now();
  tick=setInterval(()=>{
    const sec=Math.floor((Date.now()-startedAt)/1000);
    document.getElementById("clock").textContent=
      [Math.floor(sec/3600),Math.floor(sec%3600/60),sec%60]
      .map(n=>String(n).padStart(2,"0")).join(":");
  },1000);
}
function stopTimer(){
  if(!startedAt) return;
  const minutes=Math.round((Date.now()-startedAt)/60000);
  clearInterval(tick);
  startedAt=null; tick=null;
  document.getElementById("clock").textContent="00:00:00";
  if(minutes>0){
    data.sessions.push({
      subject:document.getElementById("subject").value,
      minutes,date:today()
    });
    save();
    alert("Study session saved!");
  } else {
    alert("Study for at least 30 seconds before saving.");
  }
}
function addChapter(){
  const name=document.getElementById("chapter").value.trim();
  if(!name) return alert("Enter a chapter name.");
  data.chapters.push({
    name,
    subject:document.getElementById("chapterSubject").value,
    done:false
  });
  document.getElementById("chapter").value="";
  save();
}
function toggleChapter(i){
  data.chapters[i].done=!data.chapters[i].done;
  save();
}
function addTest(){
  const name=document.getElementById("testName").value.trim();
  const marks=Number(document.getElementById("marks").value);
  const total=Number(document.getElementById("total").value);
  if(!name || !Number.isFinite(marks) || !Number.isFinite(total) ||
     total<=0 || marks<0 || marks>total){
    return alert("Enter a test name and valid marks.");
  }
  data.tests.push({name,marks,total});
  document.getElementById("testName").value="";
  document.getElementById("marks").value="";
  document.getElementById("total").value="";
  save();
}
function addReminder(){
  const text=document.getElementById("reminderText").value.trim();
  const time=document.getElementById("reminderTime").value;
  if(!text || !time) return alert("Enter reminder details.");
  data.reminders.push({text,time,notified:false});
  document.getElementById("reminderText").value="";
  save();
  if("Notification" in window && Notification.permission==="default"){
    Notification.requestPermission();
  }
}
function checkReminders(){
  const now=Date.now();
  data.reminders.forEach(r=>{
    if(!r.notified && new Date(r.time).getTime()<=now){
      r.notified=true;
      alert("Study reminder: "+r.text);
      save();
    }
  });
}
function getAdvice(){
  const remaining=Math.max(0,Number(data.goal)-todayMinutes());
  const pending=data.chapters.filter(c=>!c.done).length;
  let advice=remaining===0
    ?"Great work! You have reached your daily study-time goal. Review what you learned."
    :"You have "+remaining+" minutes left to reach today's study goal. Try one focused session.";
  if(pending) advice+=" You have "+pending+" chapters left to complete.";
  if(data.tests.length){
    const avg=data.tests.reduce((s,t)=>s+t.marks/t.total*100,0)/data.tests.length;
    advice+=" Your average test percentage is "+avg.toFixed(1)+"%. Review mistakes from your tests.";
  }
  document.getElementById("aiAdvice").textContent=advice;
}
function exportData(){
  const blob=new Blob([JSON.stringify(data,null,2)],{type:"application/json"});
  const url=URL.createObjectURL(blob);
  const a=document.createElement("a");
  a.href=url;a.download="StudyTrack-backup.json";a.click();
  URL.revokeObjectURL(url);
}
function clearData(){
  if(confirm("Delete all StudyTrack data from this browser?")){
    data={chapters:[],tests:[],sessions:[],reminders:[],goal:180};
    save();
  }
}
function render(){
  document.getElementById("goal").value=data.goal;
  document.getElementById("goalStatus").textContent=
    todayMinutes()+" of "+data.goal+" minutes studied today.";
  document.getElementById("chapters").innerHTML="";
  data.chapters.forEach((c,i)=>{
    const li=document.createElement("li");
    const label=document.createElement("label");
    const cb=document.createElement("input");
    cb.type="checkbox";cb.checked=c.done;
    cb.addEventListener("change",()=>toggleChapter(i));
    label.append(cb,document.createTextNode(" "+c.subject+": "+c.name));
    li.append(label);
    document.getElementById("chapters").append(li);
  });
  document.getElementById("testResults").innerHTML=
    data.tests.length ? data.tests.map(t=>
      "<p>"+escapeHTML(t.name)+": "+t.marks+"/"+t.total+
      " ("+(t.marks/t.total*100).toFixed(1)+"%)</p>"
    ).join("") : "No tests recorded yet.";
  const avg=data.tests.length
    ? data.tests.reduce((s,t)=>s+t.marks/t.total*100,0)/data.tests.length : 0;
  document.getElementById("analytics").textContent=
    "Total study time: "+data.sessions.reduce((s,x)=>s+x.minutes,0)+
    " minutes | Chapters completed: "+
    data.chapters.filter(c=>c.done).length+"/"+data.chapters.length+
    " | Average test score: "+avg.toFixed(1)+"%";
  document.getElementById("reminders").innerHTML="";
  data.reminders.forEach((r,i)=>{
    const li=document.createElement("li");
    li.textContent=r.text+" — "+new Date(r.time).toLocaleString()+
      (r.notified?" (triggered)":"");
    document.getElementById("reminders").append(li);
  });
}
function escapeHTML(s){
  return s.replace(/[&<>"']/g,c=>({
    "&":"&amp;","<":"&lt;",">":"&gt;",'"':"&quot;","'":"&#39;"
  })[c]);
}
document.getElementById("goal").addEventListener("change",e=>{
  data.goal=Math.max(1,Number(e.target.value)||180);save();
});
render();
setInterval(checkReminders,15000);
</script>
</body>
</html>
