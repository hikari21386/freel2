<!doctype html>
recs.forEach(r=>{
const v=r.categoryValue; if(!sumsByCategory[v]) sumsByCategory[v]=0; sumsByCategory[v]+=r.amount;
if(v.startsWith('income')) totalIncome+=r.amount; else totalExpense+=r.amount;
});
for(const k in sumsByCategory){ labels.push(k); barData.push(sumsByCategory[k]); pieLabels.push(k); pieData.push(sumsByCategory[k]); }


const ctx=document.getElementById('barChart').getContext('2d');
if(barChart) barChart.destroy();
barChart=new Chart(ctx,{type:'bar',data:{labels:labels, datasets:[{label:'カテゴリ別',data:barData}]}, options:{responsive:true}});


const ctx2=document.getElementById('pieChart').getContext('2d'); if(pieChart) pieChart.destroy();
pieChart=new Chart(ctx2,{type:'pie',data:{labels:pieLabels,datasets:[{data:pieData}]}, options:{responsive:true}});
}


function getAllRecordsForMonth(month){
// month is YYYY-MM or empty -> all
if(!state.active) return [];
const all=state.accounts[state.active].records||[];
if(!month) return all;
return all.filter(r=>r.date.slice(0,7)===month);
}


// render UI
function renderAll(){
const list=document.getElementById('recordsList'); list.innerHTML='';
const catSumsDiv=document.getElementById('categorySums'); catSumsDiv.innerHTML='';
if(!state.active){acctMsg('ログインしてね'); return}
const recs = state.accounts[state.active].records||[];
const personFilter=document.getElementById('personFilter').value;


const sumsByCat={}; let houseIncome=0, houseExpense=0;


recs.forEach(r=>{
const li=document.createElement('li'); li.className='record';
const left=document.createElement('div'); left.className='left';
const badge=document.createElement('div'); badge.className='badge'; badge.textContent=r.owner; badge.style.background=(r.owner==='hikari'?getComputedStyle(document.documentElement).getPropertyValue('--hikari'):(r.owner==='akito'?getComputedStyle(document.documentElement).getPropertyValue('--akito'):getComputedStyle(document.documentElement).getPropertyValue('--ayumi')));
left.appendChild(badge);
const txt=document.createElement('div'); txt.innerHTML=`<strong>${r.categoryLabel}</strong><div class="small">${r.date} / ${r.payType}</div>`;
left.appendChild(txt);
const right=document.createElement('div'); right.innerHTML=`<div style="text-align:right"><div>${r.amount} 円</div><div class="small">${r.id}</div></div>`;
li.appendChild(left); li.appendChild(right);


// apply filter
if(personFilter==='all' || personFilter===r.owner) list.appendChild(li);


// sums
if(!sumsByCat[r.categoryLabel]) sumsByCat[r.categoryLabel]=0; sumsByCat[r.categoryLabel]+=r.amount;
if(r.categoryValue.startsWith('income')) houseIncome += r.amount; else houseExpense += r.amount;
});


// category list
const ul=document.createElement('ul'); for(const k in sumsByCat){const it=document.createElement('li'); it.textContent=`${k}: ${sumsByCat[k]} 円`; ul.appendChild(it);} catSumsDiv.appendChild(ul);
document.getElementById('houseIncome').textContent = houseIncome;
document.getElementById('houseExpense').textContent = houseExpense;
document.getElementById('net').textContent = (houseIncome - houseExpense);
}


// initial
renderAll();


// draw charts button
document.getElementById('drawBtn').addEventListener('click', ()=>{
const month=document.getElementById('monthPicker').value; drawCharts(month);
});


// load sample data helper (for demo) — optional
function loadSample(){
const name='demo'; state.accounts[name]={passwordHash:'',records:[
{id:uid(),date:'2025-09-01',categoryValue:'income_salary',categoryLabel:'基本給',amount:250000,owner:'hikari',payType:'bank'},
{id:uid(),date:'2025-09-02',categoryValue:'expense_rent',categoryLabel:'家賃',amount:60000,owner:'hikari',payType:'bank'},
{id:uid(),date:'2025-09-05',categoryValue:'expense_util',categoryLabel:'光熱費',amount:12000,owner:'akito',payType:'card'},
{id:uid(),date:'2025-09-07',categoryValue:'expense_cat',categoryLabel:'猫の支出',amount:7000,owner:'hikari',payType:'cash'},
]}; state.active=name; saveState(); renderAll(); acctMsg('デモデータ読み込み'); }


// optional demo loader
if(!state.active && Object.keys(state.accounts).length===0){loadSample();}


</script>
</body>
</html>
        
