<!DOCTYPE html>
<html lang="sq">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Bioeng - Sistemi Laboratorik Profesional</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
<style>
body {font-family:'Times New Roman', Times, serif; background:#f5f7fa; margin:0; padding:20px; color:#333;}
h1,h2,h3 {margin:0; color:#b30000;}
h2 {margin-top:10px;}
h3 {margin-top:20px; padding-bottom:5px; border-bottom:2px solid #b30000;}
.logo {width:30px; height:30px; background:#b30000; border-radius:50%; display:inline-block; margin-right:10px;}
.card {background:white; padding:20px; border-radius:12px; box-shadow:0 2px 6px rgba(0,0,0,0.1);}
table {width:100%; border-collapse:collapse; margin-top:10px; font-size:14px;}
th, td {border:1px solid #ccc; padding:8px; text-align:center;}
th {background:linear-gradient(to right,#fce4e4,#fff);}
.report-section {margin-top:15px;}
.final-signs {display:flex; justify-content:space-between; margin-top:60px; width:100%;}
.final-signs div {width:45%;}
.final-signs div:first-child {text-align:left;}   /* Vertetoi majtas */
.final-signs div:last-child {text-align:right;}  /* Punoi djathtas */
#reportContainer {display:none; background:white; padding:20px; border-radius:12px;}
input, select {width:100%; padding:8px; border-radius:6px; border:1px solid #ccc; margin-top:5px;}
button {margin-top:10px; padding:10px; background:#b30000; color:white; border:none; border-radius:8px; cursor:pointer;}
button:hover {background:#d90000;}
</style>
</head>
<body>

<h1 style="text-align:center;"><span class="logo"></span>POLIKLINIKA BIOENG</h1>
<p style="text-align:center; font-size:14px;">Adresa: Rruga Sadulla Brestovci nr. 100, Gjilan</p>

<!-- Login -->
<div class="card" id="loginCard">
  <h2>Login</h2>
  <label>Email<input type="email" id="loginEmail" placeholder="shembull@bioeng.com"></label>
  <label>Password<input type="password" id="loginPassword"></label>
  <button id="loginBtn">Kyçu</button>
</div>

<!-- Sistemi -->
<div id="system" style="display:none;">
<div class="card">
  <h2>Regjistro Pacient</h2>
  <label>Emri<input id="patientName"></label>
  <label>Mbiemri<input id="patientSurname"></label>
  <label>Adresa<input id="patientAddress"></label>
  <label>Gjinia
    <select id="patientGender">
      <option>Femer</option>
      <option>Mashkull</option>
    </select>
  </label>
  <label>Datëlindja (DD-MM-YYYY)<input id="patientDOB" placeholder="12-05-1990"></label>
  <button id="savePatient">Ruaj Pacientin</button>

  <h3>Lista Pacientëve</h3>
  <input id="searchPatient" placeholder="Kërko pacient...">
  <select id="patientsList" size="6" style="width:100%; margin-top:5px;"></select>
  <button id="deletePatient" style="margin-top:5px; background:#555;">Fshij Pacientin</button>
</div>

<div class="card">
  <h2>Zgjidh Analizat për Pacientin</h2>
  <label>Lloji i Analizës
    <select id="analysisType">
      <option value="Biokimi">Biokimi</option>
      <option value="Hematologji">Hematologji</option>
      <option value="Hormonale">Hormonale</option>
      <option value="Urine">Urine</option>
      <option value="Reumatologjike">Reumatologjike</option>
      <option value="Tumor Marker">Tumor Marker</option>
    </select>
  </label>
  <label>Emri Analizës
    <select id="analysisName"></select>
  </label>
  <button id="addAnalysis">Shto Analizë për Pacientin</button>

  <h3>Analizat e Zgjedhura për Vlerësim</h3>
  <table class="report-table">
    <thead><tr><th>Lloji</th><th>Analiza</th><th>Vlera</th><th>Referenca</th><th>Fshij</th></tr></thead>
    <tbody id="analysisList"></tbody>
  </table>
</div>

<div class="card" style="margin-top:20px;">
  <h2>Raporti Profesional</h2>
  <button id="generateReport">Gjenero Raport</button>
  <button id="printReport">Printo</button>
  <button id="downloadPDF">Shkarko PDF</button>
</div>
</div>

<div id="reportContainer"></div>

<script>
// LOGIN
const USER_EMAIL="admin@bioeng.com";
const USER_PASS="Bioeng2025";
document.getElementById("loginBtn").addEventListener("click",()=>{
  const e=document.getElementById("loginEmail").value;
  const p=document.getElementById("loginPassword").value;
  if(e===USER_EMAIL && p===USER_PASS){
    document.getElementById("loginCard").style.display="none";
    document.getElementById("system").style.display="block";
  } else alert("Email ose password i gabuar!");
});

// ANALIZAT
const analysesByType={
  "Biokimi": {"Glukoza esull (mmol/L)":"3.3-6.1","Urea (mmol/L)":"1.7-8.3","Kreatinina Mashkull (µmol/L)":"70-108","Kreatinina Femër (µmol/L)":"44-88","Kolesteroli Total (mmol/L)":"0-200","Trigliceridet (mmol/L)":"0-150","Albumina (g/L)":"35-50","Bilirubina Total (µmol/L)":"5-21"},
  "Hematologji": {"Hemoglobina Mashkull (g/dL)":"13.5-17.5","Hemoglobina Femër (g/dL)":"12-16","Hematokriti Mashkull (%)":"41-53","Hematokriti Femër (%)":"36-46","Leukocitet (10^9/L)":"4-10"},
  "Hormonale": {"TSH (µIU/mL)":"0.3-4.5","T4 (µg/dL)":"0.9-1.75","T3 (ng/dL)":"80-200","Progesteroni (ng/mL)":"1-20","Estradiol (pg/mL)":"10-350"},
  "Urine": {"Pamja":"Normale","Ngjyra":"Verdhë e lehtë","pH":"4.5-8","Protein":"Negativ"},
  "Reumatologjike": {"Factor Reumatoid (IU/mL)":"0-14","ANA":"Negativ"},
  "Tumor Marker": {"CA 125 (U/mL)":"0-35","CA 19-9 (U/mL)":"0-37","AFP (ng/mL)":"0-10"}
};

// INDEXEDDB
let db;
const request=indexedDB.open("LabDB",1);
request.onupgradeneeded=function(e){
  db=e.target.result;
  if(!db.objectStoreNames.contains("patients")) db.createObjectStore("patients",{keyPath:"id",autoIncrement:true});
};
request.onsuccess=function(e){db=e.target.result; renderPatients();}
request.onerror=function(e){alert("IndexedDB Error");}

// DOM
const patientName=document.getElementById('patientName');
const patientSurname=document.getElementById('patientSurname');
const patientAddress=document.getElementById('patientAddress');
const patientGender=document.getElementById('patientGender');
const patientDOB=document.getElementById('patientDOB');
const savePatient=document.getElementById('savePatient');
const searchPatient=document.getElementById('searchPatient');
const patientsList=document.getElementById('patientsList');
const deletePatient=document.getElementById('deletePatient');
const analysisType=document.getElementById('analysisType');
const analysisName=document.getElementById('analysisName');
const addAnalysis=document.getElementById('addAnalysis');
const analysisList=document.getElementById('analysisList');
const generateReport=document.getElementById('generateReport');
const printReport=document.getElementById('printReport');
const downloadPDF=document.getElementById('downloadPDF');
const reportContainer=document.getElementById('reportContainer');
let selectedPatientId=null;

// Popullo Analizat
function populateAnalyses(){
  analysisName.innerHTML="";
  const type=analysisType.value;
  Object.keys(analysesByType[type]).forEach(a=>{
    const opt=document.createElement('option');
    opt.value=a; opt.textContent=a; analysisName.appendChild(opt);
  });
}
analysisType.addEventListener('change',populateAnalyses);
populateAnalyses();

// Ruaj pacient
savePatient.addEventListener('click',()=>{
  const name=patientName.value.trim(), surname=patientSurname.value.trim(), address=patientAddress.value.trim();
  const gender=patientGender.value, dob=patientDOB.value.trim();
  if(!name||!surname||!dob) return alert("Plotëso emrin, mbiemrin dhe datëlindjen!");
  const tx=db.transaction("patients","readwrite");
  const store=tx.objectStore("patients");
  store.add({name,surname,address,gender,dob,analyses:[],createdAt:new Date().toISOString()});
  tx.oncomplete=()=>{renderPatients(); patientName.value=""; patientSurname.value=""; patientAddress.value=""; patientDOB.value="";}
});

// Render pacientet sipas date (më të fundit lartë)
function renderPatients(filter=""){
  patientsList.innerHTML="";
  const tx=db.transaction("patients","readonly");
  const store=tx.objectStore("patients");
  let patientsArr=[];
  store.openCursor().onsuccess=function(e){
    const cursor=e.target.result;
    if(cursor){
      const p=cursor.value;
      if((p.name+" "+p.surname).toLowerCase().includes(filter.toLowerCase())) patientsArr.push(p);
      cursor.continue();
    } else {
      patientsArr.sort((a,b)=>new Date(b.createdAt)-new Date(a.createdAt));
      patientsArr.forEach(p=>{
        const opt=document.createElement('option'); opt.value=p.id; opt.textContent=p.name+" "+p.surname;
        if(selectedPatientId===p.id) opt.selected=true;
        patientsList.appendChild(opt);
      });
    }
  }
}
searchPatient.addEventListener('input',e=>renderPatients(e.target.value));
patientsList.addEventListener('change',e=>{selectedPatientId=Number(e.target.value); renderAnalysis();});
deletePatient.addEventListener('click',()=>{
  if(!selectedPatientId) return;
  const tx=db.transaction("patients","readwrite");
  tx.objectStore("patients").delete(selectedPatientId);
  tx.oncomplete=()=>{selectedPatientId=null; renderPatients(); analysisList.innerHTML="";}
});

// Analizat e pacientit
function renderAnalysis(){
  if(!selectedPatientId) return;
  const tx=db.transaction("patients","readonly");
  const store=tx.objectStore("patients");
  store.get(selectedPatientId).onsuccess=function(e){
    const patient=e.target.result;
    analysisList.innerHTML="";
    patient.analyses.forEach((a,i)=>{
      const tr=document.createElement('tr');
      tr.innerHTML=`<td>${a.type}</td><td>${a.name}</td><td><input value="${a.value||''}" data-index="${i}" class="valInput"></td><td>${a.reference}</td><td><button class="delBtn" data-index="${i}">Fshij</button></td>`;
      analysisList.appendChild(tr);
    });
    document.querySelectorAll(".delBtn").forEach(btn=>btn.addEventListener('click',e=>{
      const idx=Number(e.target.dataset.index);
      const store=db.transaction("patients","readwrite").objectStore("patients");
      store.get(selectedPatientId).onsuccess=function(evt){
        const p=evt.target.result;
        p.analyses.splice(idx,1);
        store.put(p);
        renderAnalysis();
      }
    }));
    document.querySelectorAll(".valInput").forEach(input=>input.addEventListener('input',e=>{
      const idx=Number(e.target.dataset.index);
      const store=db.transaction("patients","readwrite").objectStore("patients");
      store.get(selectedPatientId).onsuccess=function(evt){
        const p=evt.target.result;
        p.analyses[idx].value=e.target.value;
        store.put(p);
      }
    }));
  }
}

// Shto Analize
addAnalysis.addEventListener('click',()=>{
  if(!selectedPatientId) return alert("Zgjidh pacientin!");
  const type=analysisType.value, name=analysisName.value;
  const reference=analysesByType[type][name];
  const store=db.transaction("patients","readwrite").objectStore("patients");
  store.get(selectedPatientId).onsuccess=function(e){
    const p=e.target.result;
    if(p.analyses.find(a=>a.name===name)) return alert("Analiza tashmë ekziston!");
    p.analyses.push({type,name,value:"",reference});
    store.put(p);
    renderAnalysis();
  }
});

// Raporti Profesional
function generateReportFunc(){
  if(!selectedPatientId) return alert("Zgjidh pacientin!");
  const store=db.transaction("patients","readonly").objectStore("patients");
  store.get(selectedPatientId).onsuccess=function(e){
    const p=e.target.result;
    let html=`<div style="text-align:center; margin-bottom:20px;">
                <span style="display:inline-block;width:30px;height:30px;background:#b30000;border-radius:50%; margin-right:10px;"></span>
                <h1 style="display:inline-block; vertical-align:middle;">POLIKLINIKA BIOENG</h1>
                <h2>Raport Profesional</h2>
              </div>
              <p><strong>Emri:</strong> ${p.name} &nbsp;&nbsp; <strong>Mbiemri:</strong> ${p.surname}</p>
              <p><strong>Adresa:</strong> ${p.address} &nbsp;&nbsp; <strong>Gjinia:</strong> ${p.gender}</p>
              <p><strong>Datëlindja:</strong> ${p.dob} &nbsp;&nbsp; <strong>Data:</strong> ${new Date().toLocaleDateString()}</p>`;
    
    const grouped={};
    p.analyses.forEach(a=>{ if(!grouped[a.type]) grouped[a.type]=[]; grouped[a.type].push(a); });

    for(const type in grouped){
      html+=`<div class="report-section"><h3>${type}</h3>
             <table>
             <tr><th>Analiza</th><th>Vlera</th><th>Referenca</th></tr>`;
      grouped[type].forEach(a=>{ html+=`<tr><td>${a.name}</td><td>${a.value}</td><td>${a.reference}</td></tr>`; });
      html+=`</table></div>`;
    }

    html+=`<div class="final-signs">
              <div><p>Vertetoi: Dr. Mamudije Luma<br>Specialiste i Biokimise Klinike</p></div>
              <div><p>Punoi: Mr. sc. Ardian Hyseni<br>Biokimist</p></div>
            </div>`;
    reportContainer.innerHTML=html;
    reportContainer.style.display="block";
  }
}

// Print me logo dhe titull
printReport.addEventListener('click', () => {
    if(!selectedPatientId) return alert("Zgjidh pacientin!");
    generateReportFunc();
    setTimeout(()=>{
        const printWindow = window.open('', '', 'width=900,height=700');
        printWindow.document.write('<html><head><title>Raporti Profesional</title>');
        printWindow.document.write('<style>body{font-family:Times New Roman, serif; color:#333; margin:20px;} h1,h2,h3{color:#b30000;} table{width:100%; border-collapse:collapse; margin-top:10px;} th, td{border:1px solid #ccc; padding:8px; text-align:center;} th{background:#fce4e4;} .final-signs{display:flex; justify-content:space-between; width:100%; margin-top:60px;} .final-signs div:first-child{text-align:left;} .final-signs div:last-child{text-align:right;}</style>');
        printWindow.document.write('</head><body>');
        printWindow.document.write(reportContainer.innerHTML);
        printWindow.document.write('</body></html>');
        printWindow.document.close();
        printWindow.focus();
        printWindow.print();
    },300);
});

// Shkarko PDF
downloadPDF.addEventListener('click',()=>{
  generateReportFunc();
  setTimeout(()=>{
    const { jsPDF } = window.jspdf;
    const doc = new jsPDF();
    doc.html(reportContainer, {
      callback:function(d){d.save("Raporti_Analizave.pdf");},
      x:10, y:10, html2canvas:{scale:0.5}
    });
  },300);
});
</script>
</body>
</html>
