<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>AI EV Charging Allocation - Competition Version</title>
<style>
body{margin:0;font-family:Arial;background:#0b1220;color:#e6f2ff}
header{background:#111c33;padding:15px 25px;font-weight:bold}
section{padding:20px}
.card{background:#121e33;padding:20px;border-radius:10px;margin-bottom:20px}
input,select{width:100%;padding:8px;margin:5px 0;background:#1a2742;border:1px solid #00c8ff44;color:white;border-radius:6px}
button{padding:8px 14px;background:#0066ff;border:none;border-radius:6px;color:white;cursor:pointer;font-weight:bold;margin-right:5px}
button:hover{opacity:.8}
.grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(140px,1fr));gap:10px}
.slot{padding:15px;border-radius:8px;text-align:center;font-weight:bold}
.free{background:#0f5132}
.busy{background:#7f1d1d}
.queue-box{background:#020d1a;padding:15px;border-radius:8px;min-height:120px;font-family:monospace}
.metric{font-size:14px;margin-bottom:5px}
</style>
</head>
<body>

<header>⚡ AI EV Charging Allocation (AOA + OS Competition Level)</header>

<section>

<div class="card">
<h3>Add EV</h3>
<label>Vehicle ID</label>
<input type="text" id="vid">
<label>Battery (%)</label>
<input type="number" id="battery" min="0" max="100">
<label>Charger Type</label>
<select id="ctype">
<option>DC Fast</option>
<option>AC Level 2</option>
<option>Ultra Fast</option>
</select>
<button onclick="addEV()">Add to Queue</button>
<button onclick="allocate()">Allocate</button>
</div>

<div class="card">
<h3>Charging Slots</h3>
<div id="slots" class="grid"></div>
</div>

<div class="card">
<h3>Priority Queue (Min-Heap)</h3>
<div id="queue" class="queue-box"></div>
</div>

<div class="card">
<h3>System Metrics</h3>
<div class="metric" id="throughput"></div>
<div class="metric" id="waiting"></div>
<div class="metric" id="safeState"></div>
</div>

</section>

<script>

// CONFIG
const TOTAL_SLOTS = 5;
let totalAllocations = 0;

// SLOT STATE
let slots = Array.from({length:TOTAL_SLOTS},(_,i)=>({
    id:i+1,
    busy:false,
    vehicle:null
}));

// MIN HEAP IMPLEMENTATION (AOA)
class MinHeap{
    constructor(){this.heap=[];}

    insert(node){
        this.heap.push(node);
        this.bubbleUp();
    }

    bubbleUp(){
        let index=this.heap.length-1;
        while(index>0){
            let parent=Math.floor((index-1)/2);
            if(this.heap[parent].priority <= this.heap[index].priority) break;
            [this.heap[parent],this.heap[index]]=[this.heap[index],this.heap[parent]];
            index=parent;
        }
    }

    extractMin(){
        if(this.heap.length===1) return this.heap.pop();
        const min=this.heap[0];
        this.heap[0]=this.heap.pop();
        this.bubbleDown();
        return min;
    }

    bubbleDown(){
        let index=0;
        const length=this.heap.length;
        while(true){
            let left=2*index+1;
            let right=2*index+2;
            let smallest=index;

            if(left<length && this.heap[left].priority < this.heap[smallest].priority)
                smallest=left;

            if(right<length && this.heap[right].priority < this.heap[smallest].priority)
                smallest=right;

            if(smallest===index) break;

            [this.heap[index],this.heap[smallest]]=[this.heap[smallest],this.heap[index]];
            index=smallest;
        }
    }

    size(){return this.heap.length;}
    getAll(){return this.heap;}
}

let pq = new MinHeap();

// RENDER SLOTS
function renderSlots(){
    let html="";
    slots.forEach(slot=>{
        if(slot.busy){
            html+=`<div class="slot busy">
            Slot ${slot.id}<br>${slot.vehicle.id}
            <br><button onclick="release(${slot.id})">Release</button>
            </div>`;
        }else{
            html+=`<div class="slot free">Slot ${slot.id}<br>Available</div>`;
        }
    });
    document.getElementById("slots").innerHTML=html;
}

// RENDER QUEUE
function renderQueue(){
    let text="";
    pq.getAll().forEach(ev=>{
        text+=`ID:${ev.id} | Battery:${ev.battery}% | Priority:${ev.priority}\n`;
    });
    document.getElementById("queue").innerText=text || "Queue Empty";
}

// BANKER SAFE STATE CHECK (OS)
function isSafe(){
    let available=slots.filter(s=>!s.busy).length;
    return available>0;
}

// ADD EV
function addEV(){
    let id=document.getElementById("vid").value;
    let battery=parseInt(document.getElementById("battery").value);
    let type=document.getElementById("ctype").value;

    if(!id || isNaN(battery)){alert("Invalid Input");return;}

    // Priority = battery level (lower battery → higher priority)
    let priority=battery;

    // Aging factor to prevent starvation
    priority = priority - (pq.size()*2);

    pq.insert({id,battery,type,priority});
    renderQueue();
    updateMetrics();
}

// ALLOCATE (Priority Scheduling + Mutex)
function allocate(){

    if(pq.size()===0){alert("Queue Empty");return;}

    if(!isSafe()){
        alert("Unsafe State: All Ports Busy");
        return;
    }

    let ev=pq.extractMin();

    for(let slot of slots){
        if(!slot.busy){
            slot.busy=true; // Mutex Lock
            slot.vehicle=ev;
            totalAllocations++;
            break;
        }
    }

    renderSlots();
    renderQueue();
    updateMetrics();
}

// RELEASE SLOT
function release(id){
    let slot=slots.find(s=>s.id===id);
    slot.busy=false;
    slot.vehicle=null;
    renderSlots();
    updateMetrics();
}

// METRICS
function updateMetrics(){
    document.getElementById("throughput").innerText=
        "Total Allocations: "+totalAllocations;
    document.getElementById("waiting").innerText=
        "Waiting EVs: "+pq.size();
    document.getElementById("safeState").innerText=
        "Safe State: "+(isSafe()?"YES":"NO");
}

renderSlots();
renderQueue();
updateMetrics();

</script>
</body>
</html>