<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<title>Vitvisor Album</title>
<style>
body {margin:0; background:#111; color:#fff; font-family:'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;}
header {position:sticky; top:0; z-index:1000; background:#141414; padding:15px 20px; display:flex; align-items:center; gap:10px; box-shadow:0 4px 10px rgba(0,0,0,0.7);}
header h1 {margin:0; font-size:1.8rem; color:#e50914; flex-shrink:0; text-shadow:0 0 8px #e50914;}
.hamburger {font-size:1.8rem; cursor:pointer; color:#fff;}
.hamburger:hover {color:#e50914;}
.search-bar {flex:1; display:flex; gap:10px; align-items:center;}
.search-bar input {padding:8px 10px; font-size:14px; border:none; border-radius:6px; background:#222; color:#fff; outline:none; width:100%; box-shadow: inset 0 0 5px rgba(0,0,0,0.6); transition:.3s;}
.search-bar input:hover {background:#333;}
.search-bar select {padding:8px 10px; border:none; border-radius:6px; background:#222; color:#fff; cursor:pointer;}
#side-menu {position:fixed; top:0; left:-250px; width:250px; height:100%; background:#141414; box-shadow:5px 0 15px rgba(0,0,0,0.8); padding:20px; transition:.3s; z-index:2000; display:flex; flex-direction:column; gap:10px;}
#side-menu h3 {margin-top:0; color:#e50914;}
#side-menu button {padding:10px; border:none; border-radius:6px; background:#222; color:#fff; cursor:pointer; font-weight:bold; text-align:left; transition:.2s;}
#side-menu button.active {background:#e50914; color:#fff;}
#side-menu button:hover:not(.active){background:#333;}
section {padding:20px 20px;}
section h2 {font-size:1.6rem; color:#e50914; margin-bottom:10px;}
.grid {display:grid; grid-template-columns:repeat(auto-fill,minmax(180px,1fr)); gap:15px;}
.card {background:#1c1c1c; border-radius:12px; overflow:hidden; transition:.3s; box-shadow:0 5px 15px rgba(0,0,0,0.6);}
.card:hover {transform:translateY(-5px) scale(1.05); box-shadow:0 10px 25px rgba(0,0,0,0.8);}
.card img {width:100%; height:260px; object-fit:cover; transition:.3s;}
.card img:hover {filter: brightness(1.1);}
.info {padding:12px;}
.info h4 {margin:0 0 6px 0; font-size:1.1rem; color:#e50914;}
button.save {background:#e50914;color:#fff;width:100%;margin-top:6px;border:none;border-radius:6px;cursor:pointer;font-weight:bold; transition:.2s;}
button.save:hover{background:#ff1a2d;}
button.remove {background:#ff3b3b;color:#fff;width:100%;margin-top:6px;border:none;border-radius:6px;cursor:pointer;font-weight:bold; transition:.2s;}
button.remove:hover{background:#ff5c5c;}
.stars {display:flex; margin-bottom:6px;}
.stars span {font-size:18px; cursor:pointer; color:#555; transition:.2s;}
.stars span.active {color:gold; text-shadow:0 0 5px gold;}
.no-results {grid-column:1/-1; text-align:center; font-size:18px; opacity:.7;}
@media(max-width:600px){ .card{width:150px;} }
</style>
</head>
<body>

<header>
    <div class="hamburger">&#9776;</div>
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

<!-- Menú lateral -->
<div id="side-menu">
    <h3>Mis Guardados</h3>
    <button class="active" data-filter="all">Todos</button>
    <button data-filter="books">Libros</button>
    <button data-filter="movies">Películas</button>
    <button data-filter="games">Videojuegos</button>
</div>

<section>
    <h2>Resultados</h2>
    <div class="grid" id="results"></div>
</section>

<section>
    <h2>Mi Biblioteca</h2>
    <div class="grid" id="library"></div>
</section>

<script>
const hamburger=document.querySelector(".hamburger");
const sideMenu=document.getElementById("side-menu");
const filterButtons=document.querySelectorAll("#side-menu button");
const searchInput=document.getElementById("search");
const typeSelect=document.getElementById("type");
const resultsDiv=document.getElementById("results");
const libraryDiv=document.getElementById("library");

let library=JSON.parse(localStorage.getItem("vitvisor"))||[];
let activeFilter="all";
let menuOpen=false;

// Hamburger toggle
hamburger.addEventListener("click",()=>{menuOpen=!menuOpen; sideMenu.style.left=menuOpen?"0px":"-250px";});
document.addEventListener("click",(e)=>{if(menuOpen && !sideMenu.contains(e.target) && !hamburger.contains(e.target)){menuOpen=false; sideMenu.style.left="-250px";}});

// Biblioteca filtros
filterButtons.forEach(btn=>{btn.addEventListener("click",()=>{
    filterButtons.forEach(b=>b.classList.remove("active"));
    btn.classList.add("active");
    activeFilter=btn.getAttribute("data-filter");
    renderLibrary();
    menuOpen=false; sideMenu.style.left="-250px";
});});

// Búsqueda
searchInput.addEventListener("input",()=>{ 
    const q=searchInput.value.trim().toLowerCase(); 
    resultsDiv.innerHTML=""; // Limpiar resultados siempre
    if(q.length<3) return; // no buscar si es muy corto
    const t = typeSelect.value; 
    if(t==="books") searchBooks(q); 
    else if(t==="movies") searchMovies(q); 
    else searchGames(q);
});

// Funciones búsqueda
function searchBooks(q){ 
    fetch(`https://www.googleapis.com/books/v1/volumes?q=${q}`).then(r=>r.json()).then(d=>{ 
        if(!d.items||d.items.length===0){resultsDiv.innerHTML=`<div class="no-results">No se encontró ninguna coincidencia</div>`;} 
        else renderResults(d.items,"book");
    }); 
}
function searchMovies(q){ 
    fetch(`https://api.tvmaze.com/search/shows?q=${q}`).then(r=>r.json()).then(d=>{ 
        if(!d||d.length===0){resultsDiv.innerHTML=`<div class="no-results">No se encontró ninguna coincidencia</div>`;} 
        else renderResults(d,"movie");
    }); 
}
function searchGames(q){ 
    const qN=q.normalize("NFD").replace(/[\u0300-\u036f]/g,"").toLowerCase(); 
    fetch("https://api.codetabs.com/v1/proxy?quest=https://www.freetogame.com/api/games").then(r=>r.json()).then(data=>{ 
        const filtered=data.filter(g=>{const title=g.title?.normalize("NFD").replace(/[\u0300-\u036f]/g,"").toLowerCase()||""; const desc=g.short_description?.normalize("NFD").replace(/[\u0300-\u036f]/g,"").toLowerCase()||""; return title.includes(qN)||desc.includes(qN);}); 
        filtered.length===0? resultsDiv.innerHTML=`<div class="no-results">No se encontró ninguna coincidencia</div>`:renderResults(filtered.slice(0,50),"game");
    }).catch(()=>{resultsDiv.innerHTML=`<div class="no-results">Error al cargar videojuegos</div>`;}); 
}

// Render resultados
function renderResults(items,type){ resultsDiv.innerHTML="";
    items.forEach(i=>{
        let title,img;
        if(type==="book"){title=i.volumeInfo.title; img=i.volumeInfo.imageLinks?.thumbnail;}
        else if(type==="movie"){title=i.show.name; img=i.show.image?.medium;}
        else {title=i.title; img=i.thumbnail;}
        const card=document.createElement("div");
        card.className="card";
        card.innerHTML=`<img src="${img||''}"><div class="info"><h4>${title}</h4><div class="stars">${starsHTML(0)}</div><button class="save">Guardar</button></div>`;
        enableStars(card);
        card.querySelector(".save").onclick=()=>{
            let typeMapped = type==="book"?"books":type==="movie"?"movies":"games";
            library.push({title,img,rating:card.querySelectorAll(".active").length,type:typeMapped});
            activeFilter="all";
            filterButtons.forEach(b=>b.classList.remove("active"));
            filterButtons[0].classList.add("active");
            save(); renderLibrary();
        };
        resultsDiv.appendChild(card);
    });
}

// Render biblioteca
function renderLibrary(){ 
    libraryDiv.innerHTML=""; 
    let filtered=library; 
    if(activeFilter!=="all") filtered=library.filter(i=>i.type===activeFilter); 
    if(filtered.length===0){libraryDiv.innerHTML=`<div class="no-results">No hay elementos guardados</div>`; return;} 
    filtered.forEach((i,idx)=>{ 
        const card=document.createElement("div"); 
        card.className="card"; 
        card.innerHTML=`<img src="${i.img||''}"><div class="info"><h4>${i.title}</h4><div class="stars">${starsHTML(i.rating)}</div><button class="remove">Eliminar</button></div>`; 
        card.querySelector(".remove").onclick=()=>{ const originalIndex=library.indexOf(i); library.splice(originalIndex,1); save(); renderLibrary(); }; 
        libraryDiv.appendChild(card);
    });
}

// Estrellas
function starsHTML(n){let s="";for(let i=1;i<=5;i++) s+=`<span class="${i<=n?'active':''}">★</span>`; return s;}
function enableStars(card){const stars=card.querySelectorAll(".stars span"); stars.forEach((s,i)=>{ s.onclick=()=>{ stars.forEach(x=>x.classList.remove("active")); for(let j=0;j<=i;j++) stars[j].classList.add("active"); }; }); }
function save(){localStorage.setItem("vitvisor",JSON.stringify(library));}
renderLibrary();
</script>

</body>
</html>
