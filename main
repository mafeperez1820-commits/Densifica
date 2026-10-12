import { DEFAULT_RATES, DEFAULT_HEIGHTS, calculateTotalHeight, calculateConstructionArea, calculateVolume, calculateCoreArea, calculateUsableArea, calculateLevelCost, calculateTotalCost, formatNumber, formatMoney } from './math.js';
import { createViewer } from './viewer.js';

const $ = id => document.getElementById(id);
const uses = Object.keys(DEFAULT_HEIGHTS);
let nextId = 5, selectedId = 4, toastTimer;
let state = {
  width:20,length:30,rates:{...DEFAULT_RATES},fixedCosts:50000000,coreB:12,
  levels:[
    {id:1,use:'Comercial',height:3.25},
    {id:2,use:'Oficinas',height:3.0},
    {id:3,use:'Residencial',height:2.8},
    {id:4,use:'Residencial',height:2.8}
  ]
};
const defaultState = () => ({width:20,length:30,rates:{...DEFAULT_RATES},fixedCosts:50000000,coreB:12,levels:[
  {id:1,use:'Comercial',height:3.25},{id:2,use:'Oficinas',height:3},{id:3,use:'Residencial',height:2.8},{id:4,use:'Residencial',height:2.8}
]});
let viewer;
const money = formatMoney;
const num = n => formatNumber(n,2);
const lotArea = () => state.width*state.length;
const calculations = () => {
  const area=lotArea(), height=calculateTotalHeight(state.levels), built=calculateConstructionArea(state.levels,area);
  const core=calculateCoreArea(state.levels.length,state.coreB,area), usable=calculateUsableArea(built,core), volume=calculateVolume(area,state.levels);
  const levelCosts=state.levels.map(l=>calculateLevelCost(l,area,state.rates));
  const variableCost=levelCosts.reduce((a,b)=>a+b,0), total=calculateTotalCost(state.levels,area,state.rates,state.fixedCosts);
  return {area,height,built,core,usable,volume,levelCosts,variableCost,total,avg:built?variableCost/built:0};
};
function toast(message){const el=$('toast');el.textContent=message;el.classList.add('show');clearTimeout(toastTimer);toastTimer=setTimeout(()=>el.classList.remove('show'),2600);}
function validateInput(el,min,max,label){const value=Number(el.value);if(el.value.trim()===''||!Number.isFinite(value)||value<min||value>max){el.classList.add('invalid');toast(`${label}: ingresa un valor entre ${min} y ${max}.`);return null;}el.classList.remove('invalid');return value;}
function renderLevelList(){
  const wrap=$('levelsList');wrap.innerHTML='';
  [...state.levels].reverse().forEach(level=>{
    const idx=state.levels.findIndex(l=>l.id===level.id);
    const card=document.createElement('article');card.className='level-card'+(level.id===selectedId?' selected':'');card.tabIndex=0;card.setAttribute('aria-label',`Editar nivel ${idx+1}`);
    const color=level.use==='Comercial'?'#c5a66a':level.use==='Oficinas'?'#7296a8':'#55a58c';
    card.innerHTML=`<div class="level-card-head"><span class="level-dot" style="background:${color}"></span>NIVEL ${String(idx+1).padStart(2,'0')}<span class="level-actions"><button class="icon-btn edit-btn" aria-label="Editar nivel ${idx+1}" title="Editar">✎</button><button class="icon-btn delete-btn" aria-label="Eliminar nivel ${idx+1}" title="Eliminar">×</button></span></div><div class="level-card-sub">${level.use}<small>Altura: ${num(level.height)} m · ${money(state.rates[level.use])}/m²</small></div>`;
    card.addEventListener('click',e=>{if(e.target.closest('.delete-btn')){deleteLevel(level.id);return;}selectedId=level.id;renderLevelList();renderMetrics();viewer?.updateSelection(idx);});
    card.addEventListener('keydown',e=>{if(e.key==='Enter'||e.key===' '){e.preventDefault();selectedId=level.id;renderLevelList();renderMetrics();viewer?.updateSelection(idx);}});
    const edit=document.createElement('div');edit.className='level-edit hidden';
    edit.innerHTML=`<label>Uso<select class="use-select">${uses.map(u=>`<option ${u===level.use?'selected':''}>${u}</option>`).join('')}</select></label><label>Altura (m)<input class="height-input" type="number" min=".5" max="10" step=".05" value="${level.height}"></label>`;
    card.appendChild(edit);
    card.querySelector('.edit-btn').addEventListener('click',e=>{e.stopPropagation();edit.classList.toggle('hidden');});
    edit.querySelector('.use-select').addEventListener('change',e=>{level.use=e.target.value;level.height=DEFAULT_HEIGHTS[level.use];renderAll();toast(`Nivel ${idx+1}: uso actualizado a ${level.use}.`);});
    edit.querySelector('.height-input').addEventListener('change',e=>{const v=validateInput(e.target,.5,10,'Altura');if(v===null){e.target.value=level.height;return;}level.height=v;renderAll();toast('Altura del nivel actualizada.');});
    wrap.appendChild(card);
  });
  $('levelCountLabel').textContent=String(state.levels.length).padStart(2,'0');
  $('modelLevels').textContent=`${String(state.levels.length).padStart(2,'0')} NIVELES`;
}
function renderMetrics(){
  const c=calculations();
  $('lotArea').textContent=`${num(c.area)} m²`;
  $('heightMetric').innerHTML=`${num(c.height)} <small>m</small>`;
  $('heightEquation').textContent='H = '+(state.levels.length?state.levels.map(l=>num(l.height)).join(' + '):'0')+` = ${num(c.height)} m`;
  $('levelsMetric').textContent=state.levels.length;
  $('builtMetric').innerHTML=`${num(c.built)} <small>m²</small>`;
  $('usableMetric').innerHTML=`${num(c.usable)} <small>m²</small>`;
  $('volumeMetric').innerHTML=`${num(c.volume)} <small>m³</small>`;
  $('coreMetric').innerHTML=`${num(c.core)} <small>m²</small>`;
  $('budgetMetric').textContent=money(c.total);
  $('budgetArea').textContent=`${num(c.built)} m²`;
  $('avgCost').textContent=money(c.avg);
  $('fixedDisplay').textContent=money(state.fixedCosts);
  $('budgetTotal').textContent=money(c.total);
  $('selectedFloorLabel').textContent=selectedId===null?'Ningún nivel seleccionado':`Nivel seleccionado: ${String(state.levels.findIndex(l=>l.id===selectedId)+1).padStart(2,'0')}`;
  renderCharts(c);
}
function svgChart(content){return `<svg viewBox="0 0 300 150" role="img" aria-label="Gráfica actualizada con datos del proyecto">${content}</svg>`;}
function renderCharts(c){
  const n=state.levels.length;
  const costs=[];let running=state.fixedCosts;state.levels.forEach(l=>{running+=calculateLevelCost(l,c.area,state.rates);costs.push(running);});
  const maxCost=Math.max(1,...costs);
  const x=i=>30+(n<=1?0:i*240/(n-1)), y=v=>126-v/maxCost*102;
  let grid='';[0,.25,.5,.75,1].forEach(t=>{const yy=126-t*102;grid+=`<line x1="30" y1="${yy}" x2="282" y2="${yy}" stroke="#e7efeb" stroke-width="1"/>`;});
  let path='',dots='',labels='';
  costs.forEach((v,i)=>{path+=(i?' L':'M')+x(i)+' '+y(v);dots+=`<circle cx="${x(i)}" cy="${y(v)}" r="3.2" fill="#187f76"/>`;labels+=`<text x="${x(i)}" y="143" text-anchor="middle" fill="#81938b" font-size="9">${i+1}</text>`;});
  $('costChart').innerHTML=svgChart(grid+`<path d="${path}" fill="none" stroke="#187f76" stroke-width="2.4" stroke-linecap="round" stroke-linejoin="round"/>${dots}${labels}<text x="30" y="10" fill="#81938b" font-size="9">${money(maxCost).replace('$ ','$')}</text>`);
  const maxArea=Math.max(1,c.built), barW=Math.min(22,170/Math.max(1,n));
  let bars='', alabels='';
  state.levels.forEach((l,i)=>{const bx=40+i*240/Math.max(1,n), usableH=c.usable/n/maxArea*102, coreH=c.core/n/maxArea*102;bars+=`<rect x="${bx}" y="${126-usableH}" width="${barW}" height="${usableH}" rx="2" fill="#4a9d8a"/><rect x="${bx+barW+2}" y="${126-coreH}" width="${barW*.65}" height="${coreH}" rx="2" fill="#c5a66a"/>`;alabels+=`<text x="${bx+barW}" y="143" text-anchor="middle" fill="#81938b" font-size="9">${i+1}</text>`;});
  $('areaChart').innerHTML=svgChart(grid+bars+alabels+`<text x="30" y="10" fill="#81938b" font-size="9">${num(maxArea)} m²</text>`);
  const heights=[];let acc=0;state.levels.forEach(l=>{acc+=l.height;heights.push(acc);});const maxH=Math.max(1,...heights), hx=i=>30+(n<=1?0:i*240/(n-1)), hy=v=>126-v/maxH*102;
  let hgrid='';[0,.25,.5,.75,1].forEach(t=>{const yy=126-t*102;hgrid+=`<line x1="30" y1="${yy}" x2="282" y2="${yy}" stroke="#e7efeb" stroke-width="1"/>`;});
  let hpath='',hdots='',hlabels='';heights.forEach((v,i)=>{hpath+=(i?' L':'M')+hx(i)+' '+hy(v);hdots+=`<circle cx="${hx(i)}" cy="${hy(v)}" r="3" fill="#187f76"/>`;hlabels+=`<text x="${hx(i)}" y="143" text-anchor="middle" fill="#81938b" font-size="9">${i+1}</text>`;});
  $('heightChart').innerHTML=svgChart(hgrid+`<path d="${hpath} L ${hx(Math.max(0,n-1))} 126 L ${hx(0)} 126 Z" fill="#d9efea" opacity=".75"/><path d="${hpath}" fill="none" stroke="#187f76" stroke-width="2.4"/>${hdots}${hlabels}<text x="30" y="10" fill="#81938b" font-size="9">${num(maxH)} m</text>`);
}
function renderAll(){
  $('lotWidth').value=state.width;$('lotLength').value=state.length;
  $('rate-commercial').value=state.rates.Comercial;$('rate-office').value=state.rates.Oficinas;$('rate-residential').value=state.rates.Residencial;
  $('fixedCosts').value=state.fixedCosts;$('coreB').value=state.coreB;
  renderLevelList();renderMetrics();
  if(viewer){const c=calculations();viewer.rebuild(state.levels,state.width,state.length,c.core);viewer.updateSelection(state.levels.findIndex(l=>l.id===selectedId));}
}
function addLevel(use='Residencial',height){
  if(state.levels.length>=20){toast('Has alcanzado el máximo de 20 niveles.');return;}
  const id=nextId++;state.levels.push({id,use,height:height??DEFAULT_HEIGHTS[use]});selectedId=id;renderAll();toast(`Nivel ${state.levels.length} agregado al edificio.`);
}
function deleteLevel(id){
  if(state.levels.length<=1){toast('El edificio debe conservar al menos un nivel.');return;}
  const idx=state.levels.findIndex(l=>l.id===id);state.levels=state.levels.filter(l=>l.id!==id);
  if(selectedId===id)selectedId=state.levels[Math.max(0,idx-1)].id;
  renderAll();toast('Nivel eliminado. Métricas recalculadas.');
}
function loadDemo(){
  state={width:20,length:30,rates:{...DEFAULT_RATES},fixedCosts:50000000,coreB:12,levels:[
    {id:1,use:'Comercial',height:3.25},{id:2,use:'Oficinas',height:3},{id:3,use:'Residencial',height:2.8},{id:4,use:'Oficinas',height:3},{id:5,use:'Residencial',height:2.8}
  ]};nextId=6;selectedId=5;renderAll();toast('Proyecto de demostración cargado.');
}
function resetProject(){state=defaultState();nextId=5;selectedId=4;renderAll();toast('Proyecto reiniciado.');}
function showTooltip(index){
  const el=$('floorTooltip');if(index<0||!state.levels[index]){el.classList.add('hidden');return;}
  const l=state.levels[index],c=calculations(),cost=calculateLevelCost(l,c.area,state.rates);
  el.innerHTML=`<strong><span class="level-dot" style="display:inline-block;background:${l.use==='Comercial'?'#c5a66a':l.use==='Oficinas'?'#7296a8':'#55a58c'}"></span> NIVEL ${String(index+1).padStart(2,'0')}</strong><div>Uso <b>${l.use}</b></div><div>Altura <b>${num(l.height)} m</b></div><div>Área <b>${num(c.area)} m²</b></div><div>Costo estimado <b>${money(cost)}</b></div>`;
  el.classList.remove('hidden');
}
$('startBtn').addEventListener('click',()=>{$('welcome').classList.add('hidden');$('app').classList.remove('hidden');if(!viewer){try{viewer=createViewer($('viewer'),{onSelect:index=>{selectedId=state.levels[index]?.id;renderLevelList();renderMetrics();viewer.updateSelection(index);},onHover:index=>showTooltip(index)});$('viewerLoading').classList.add('hidden');}catch(e){$('viewerLoading').innerHTML='No se pudo iniciar WebGL. Revisa la compatibilidad del navegador y la conexión.';console.error(e);}}renderAll();});
$('addLevelBtn').addEventListener('click',()=>addLevel());
$('demoBtn').addEventListener('click',loadDemo);
$('resetBtn').addEventListener('click',resetProject);
$('lotWidth').addEventListener('change',()=>{const v=validateInput($('lotWidth'),1,500,'Ancho del lote');if(v!==null){state.width=v;renderAll();}});
$('lotLength').addEventListener('change',()=>{const v=validateInput($('lotLength'),1,500,'Largo del lote');if(v!==null){state.length=v;renderAll();}});
[['rate-commercial','Comercial'],['rate-office','Oficinas'],['rate-residential','Residencial']].forEach(([id,use])=>$(id).addEventListener('change',()=>{const v=validateInput($(id),0,100000000,'Valor por m²');if(v!==null){state.rates[use]=v;renderAll();}}));
$('fixedCosts').addEventListener('change',()=>{const v=validateInput($('fixedCosts'),0,1000000000000,'Costos fijos');if(v!==null){state.fixedCosts=v;renderAll();}});
$('coreB').addEventListener('change',()=>{const v=validateInput($('coreB'),1,20,'Parámetro b');if(v!==null){state.coreB=v;renderAll();}});
$('gridToggle').addEventListener('change',e=>viewer?.setGrid(e.target.checked));
$('zoomIn').addEventListener('click',()=>viewer?.zoom(.82));
$('zoomOut').addEventListener('click',()=>viewer?.zoom(1.22));
$('resetView').addEventListener('click',()=>viewer?.resetView());
$('perspectiveBtn').addEventListener('click',()=>{ $('perspectiveBtn').classList.add('active');$('orthoBtn').classList.remove('active');viewer?.setCameraMode(false);});
$('orthoBtn').addEventListener('click',()=>{ $('orthoBtn').classList.add('active');$('perspectiveBtn').classList.remove('active');viewer?.setCameraMode(true);});
window.addEventListener('beforeunload',()=>viewer?.dispose());
