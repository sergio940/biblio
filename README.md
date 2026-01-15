<html lang="es">
<head>
<meta charset="UTF-8">
<title>Vitvisor</title>
<style>
/* ===== Body y tipografía ===== */
body {
    margin:0;
    background:#0a0a0a;
    color:#fff;
    font-family:'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

/* ===== Cabecera tipo navbar ===== */
header {
    position:sticky;
    top:0;
    z-index:1000;
    background:linear-gradient(90deg,#111,#1c1c1c);
    padding:15px 30px;
    display:flex;
    align-items:center;
    justify-content:space-between;
    box-shadow:0 4px 10px rgba(0,0,0,0.5);
}

/* Título a la izquierda */
header h1 {
    margin:0;
    font-size:1.8rem;
    color:#00e676;
    text-shadow:0 0 8px #00e676;
}

/* Barra de búsqueda a la derecha */
.search-bar {
    display:flex;
    gap:10px;
}
.search-bar select,
.search-bar input {
    padding:8px 10px;
    font-size:14px;
    border:none;
    border-radius:6px;
    background:#222;
    color:#fff;
    outline:none;
    box-shadow: inset 0 0 5px rgba(0,0,0,0.6);
    transition:.3s;
}
.search-bar select:hover,
.search-bar input:hover {
    background:#333;
}

/* ===== Grid de resultados ===== */
section {
    padding:20px 30px;
}
section h2 {
    font-size:1.6rem;
    color:#00e676;
    text-shadow:0 0 6px #00e676;
    margin-bottom:15px;
}
.grid {
    display:grid;
    grid-template-columns:repeat(auto-fill,minmax(220px,1fr));
    gap:20px;
}

/* ===== Tarjetas ===== */
.card {
    background:#1c1c1c;
    border-radius:12px;
    overflow:hidden;
    transition:.3s;
    box-shadow:0 5px 15px rgba(0,0,0,0.6);
}
.card:hover {
    transform:translateY(-5px) scale(1.03);
    box-shadow:0 10px 25px rgba(0,0,0,0.8);
}
.card img {
    width:100%;
    height:260px;
    object-fit:cover;
    transition:.3s;
}
.card img:hover {
    filter: brightness(1.1);
}
.info {
    padding:12px;
}
.info h4 {
    margin:0 0 6px 0;
    font-size:1.1rem;
    color:#00e676;
}

/* ===== Botones ===== */
button {
    width:100%;
    padding:8px;
    margin-top:6px;
    border:none;
    border-radius:6px;
    cursor:pointer;
    font-weight:bold;
    transition:.2s;
}
.save{background:#00e676;color:#000;}
.save:hover{background:#00ff88;}
.remove{background:#e53935;color:#fff;}
.remove:hover{background:#ff5c5c;}

/* ===== Estrellas ===== */
.stars {
    display:flex;
    margin-bottom:6px;
}
.stars span {
    font-size:18px;
    cursor:pointer;
    color:#555;
    transition:.2s;
}
.stars span.active {
    color:gold;
    text-shadow:0 0 5px gold;
}

/* ===== Mensaje sin resultados ===== */
.no-results {
    grid-column:1/-1;
    text-align:center;
    font-size:18px;
    opacity:.7;
}

/* ===== Responsive ===== */
@media(max-width:600px){
    header{
        flex-direction:column;
        align-items:flex-start;
        gap:10px;
    }
    .search-bar {
        width:100%;
    }
    .grid{
        grid-template-columns:repeat(auto-fill,minmax(180px,1fr));
    }
}
</style>
</head>
<body>

<header>
    <h1>Vitvisor</h1>
    <div class="search-bar">
        <select id="type">
            <option value="books">Libros</option>
            <option value="movies">Películas / Series</option>
            <option value="games">Videojuegos</option>
        </select>
        <input type="text" id="search" placeholder="Buscar...">
    </div>
</header>

<section>
    <h2>Resultados</h2>
    <div class="grid" id="results"></div>
</section>

<section>
    <h2>Mi Biblioteca</h2>
    <div class="grid" id="library"></div>
</section>

<script>
// ==== JS igual que tu versión ====
const searchInput=document.getElementById("search");
const typeSelect=document.getElementById("type");
const results=document.getElementById("results");
const libraryDiv=document.getElementById("library");
let library=JSON.parse(localStorage.getItem("vitvisor"))||[];

searchInput.addEventListener("input",search);

function search(){
    const q=searchInput.value.trim().toLowerCase();
    if(q.length<3){results.innerHTML="";return;}
    if(typeSelect.value==="books") searchBooks(q);
    else if(typeSelect.value==="movies") searchMovies(q);
    else searchGames(q);
}

function searchBooks(q){
    fetch(`https://www.googleapis.com/books/v1/volumes?q=${q}`)
    .then(r=>r.json())
    .then(d=>!d.items||d.items.length===0? showNoResults(): renderResults(d.items,"book"));
}

function searchMovies(q){
    fetch(`https://api.tvmaze.com/search/shows?q=${q}`)
    .then(r=>r.json())
    .then(d=>!d||d.length===0? showNoResults(): renderResults(d,"movie"));
}

function searchGames(q){
    const qNormalized=q.normalize("NFD").replace(/[\u0300-\u036f]/g,"").toLowerCase();
    fetch("https://api.codetabs.com/v1/proxy?quest=https://www.freetogame.com/api/games")
    .then(r=>r.json())
    .then(data=>{
        const filtered=data.filter(g=>{
            const title=g.title?.normalize("NFD").replace(/[\u0300-\u036f]/g,"").toLowerCase()||"";
            const desc=g.short_description?.normalize("NFD").replace(/[\u0300-\u036f]/g,"").toLowerCase()||"";
            return title.includes(qNormalized)||desc.includes(qNormalized);
        });
        filtered.length===0? showNoResults() : renderResults(filtered.slice(0,20),"game");
    })
    .catch(()=>{results.innerHTML=`<div class="no-results">Error al cargar videojuegos</div>`});
}

function showNoResults(){
    results.innerHTML=`<div class="no-results">No se encontró ninguna coincidencia</div>`;
}

function renderResults(items,type){
    results.innerHTML="";
    items.forEach(i=>{
        let title,img;
        if(type==="book"){title=i.volumeInfo.title; img=i.volumeInfo.imageLinks?.thumbnail;}
        else if(type==="movie"){title=i.show.name; img=i.show.image?.medium;}
        else if(type==="game"){title=i.title; img=i.thumbnail;}
        const card=document.createElement("div");
        card.className="card";
        card.innerHTML=`<img src="${img||''}"><div class="info"><h4>${title}</h4><div class="stars">${starsHTML(0)}</div><button class="save">Guardar</button></div>`;
        enableStars(card);
        card.querySelector(".save").onclick=()=>{
            const rating=card.querySelectorAll(".active").length;
            library.push({title,img,rating,type});
            save(); renderLibrary();
        };
        results.appendChild(card);
    });
}

function renderLibrary(){
    libraryDiv.innerHTML="";
    library.forEach((i,idx)=>{
        const card=document.createElement("div");
        card.className="card";
        card.innerHTML=`<img src="${i.img||''}"><div class="info"><h4>${i.title}</h4><div class="stars">${starsHTML(i.rating)}</div><button class="remove">Eliminar</button></div>`;
        card.querySelector(".remove").onclick=()=>{library.splice(idx,1); save(); renderLibrary();}
        libraryDiv.appendChild(card);
    });
}

function starsHTML(n){let s="";for(let i=1;i<=5;i++) s+=`<span class="${i<=n?'active':''}">★</span>`;return s;}
function enableStars(card){const stars=card.querySelectorAll(".stars span");stars.forEach((s,i)=>{s.onclick=()=>{stars.forEach(x=>x.classList.remove("active"));for(let j=0;j<=i;j++) stars[j].classList.add("active");};});}
function save(){localStorage.setItem("vitvisor",JSON.stringify(library));}
renderLibrary();
</script>

</body>
</html>
