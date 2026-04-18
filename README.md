index.html
<html lang="fr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<title>Planning Bac Mobile</title>

<style>

body{
font-family:Arial;
margin:10px;
background:#f2f4f7;
}

.container{
background:#fff;
padding:15px;
border-radius:10px;
}

h1{font-size:20px;text-align:center;}

input,select,button{
width:100%;
padding:10px;
margin:5px 0;
font-size:16px;
}

button{
background:#3498db;
color:#fff;
border:none;
border-radius:6px;
}

.day{
background:#fafafa;
padding:10px;
margin-top:10px;
border-radius:8px;
}

.slot{
padding:6px;
margin:3px 0;
border-radius:5px;
color:#fff;
font-size:14px;
}

</style>
</head>

<body>

<div class="container">

<h1>Planning Bac</h1>

<select id="matiere">
<option>Français</option>
<option>Maths</option>
<option>Histoire-Géo</option>
<option>Anglais</option>
<option>Enseignement pro</option>
</select>

<select id="niveau">
<option value="1">Facile</option>
<option value="2">Moyen</option>
<option value="3">Difficile</option>
</select>

<button onclick="add()">Ajouter matière</button>

<ul id="list"></ul>

<input type="date" id="startDate">

<input type="time" id="startTime" value="08:00">

<label>Semaines</label>
<input type="number" id="weeks" value="2">

<label>Jours/semaine</label>
<input type="number" id="days" value="5">

<label>Heures/jour</label>
<input type="number" id="hours_week" value="3">

<label>Heures vacances</label>
<input type="number" id="hours_holiday" value="5">

<label>Pomodoro travail (min)</label>
<input type="number" id="work" value="25">

<label>Pause (min)</label>
<input type="number" id="break" value="5">

<button onclick="generate()">Générer planning</button>
<button onclick="exportICS()">Exporter iPhone</button>

<div id="planning"></div>

</div>

<script>

let subjects=[];
let colors=["#e74c3c","#3498db","#2ecc71","#9b59b6","#f39c12"];
let slots=[];

function add(){
subjects.push({
m:matiere.value,
n:parseInt(niveau.value),
color:colors[subjects.length%colors.length]
});
list.innerHTML+=`<li>${matiere.value}</li>`;
}

function weighted(){
let arr=[];
subjects.forEach(s=>{
for(let i=0;i<s.n;i++) arr.push(s);
});
return arr;
}

function generate(){

slots=[];
let listW=weighted();

let startDate=new Date(startDateInput.value);
let [h,m]=startTime.value.split(":").map(Number);

let idx=0;
planning.innerHTML="";

for(let i=0;i<weeks.value*7;i++){

let date=new Date(startDate);
date.setDate(date.getDate()+i);

let isHoliday=(i%7>=days.value);
let hours=isHoliday?hours_holiday.value:hours_week.value;

let div=document.createElement("div");
div.className="day";

div.innerHTML=`<strong>${date.toLocaleDateString()}</strong>`;

let currentH=h;
let currentM=m;

for(let j=0;j<hours;j++){

let s=listW[idx%listW.length];
let duration=work.value;

let start=new Date(date);
start.setHours(currentH,currentM);

let end=new Date(start.getTime()+duration*60000);

slots.push({title:s.m,start,end});

let slot=document.createElement("div");
slot.className="slot";
slot.style.background=s.color;
slot.innerHTML=`${currentH}:${String(currentM).padStart(2,"0")} - ${end.getHours()}:${String(end.getMinutes()).padStart(2,"0")}<br>${s.m}`;

div.appendChild(slot);

currentH=end.getHours();
currentM=end.getMinutes()+parseInt(break.value);

if(currentM>=60){currentH++;currentM-=60;}

idx++;
}

planning.appendChild(div);
}
}

/* ICS */
function format(d){
return d.toISOString().replace(/[-:]/g,"").split(".")[0];
}

function exportICS(){

let events="";
slots.forEach(s=>{
events+=`
BEGIN:VEVENT
SUMMARY:${s.title}
DTSTART:${format(s.start)}
DTEND:${format(s.end)}
END:VEVENT`;
});

let file=`BEGIN:VCALENDAR
VERSION:2.0
${events}
END:VCALENDAR`;

let blob=new Blob([file],{type:"text/calendar"});
let a=document.createElement("a");
a.href=URL.createObjectURL(blob);
a.download="planning.ics";
a.click();
}

</script>

</body>
</html>
