<html lang="es">
<head>
<meta charset="UTF-8">
<title>Vitvisor</title>

<style>
body{
    margin:0;
    background:#0d0d0d;
    color:#fff;
    font-family:Arial, sans-serif;
}
header{
    background:#141414;
    padding:20px;
    text-align:center;
}
select,input{
    width:100%;
    padding:12px;
    margin-top:10px;
    font-size:16px;
    border:none;
    border-radius:6px;
}
.grid{
    padding:20px;
    display:grid;
    grid-template-columns:repeat(auto-fill,minmax(180px,1fr));
    gap:15px;
}
.card{
    background:#1b1b1b;
    border-radius:10px;
    overflow:hidden;
    transition:.2s;
}
.card:hover{transform:scale(1.05)}
.card img{
    width:100%;
    height:260px;
    object-fit:cover;
}
.info{
    padding:10px;
}
button{
    width:100%;
    padding:8px;
    margin-top:5px;
    border:none;
    border-radius:5px;
    cursor:pointer;
}
.save{background:#00e676;color:#000}
.remove{background:#e53935;color:#fff}
.stars span{
    font-size:20px;
    cursor:pointer;
    color:#555;
}
.stars span.active{color:gold}
h2{padding-left:20px}
</style>
</head>

<body>

<header>
    <h1>Vitvisor</h1>
    <select id="type">
        <option value="books">Libros</option>
        <option value="movies">Películas / Series</option>
        <option value="games">Videojuegos</option>
    </select>
    <input type="text" id="search" placeholder="Buscar...">
</header>

<div class="grid" id="results"></div>

<h2>Mi Biblioteca</h2>
<div class="grid" id="library"></div>

<script>
const searchInput = document.getElementById("search");
const typeSelect = document.getElementById("type");
const results = document.getElementById("results");
const libraryDiv = document.getElementById("library");

let library = JSON.parse(localStorage.getItem("vitvisor")) || [];

searchInput.addEventListener("input", search);

function search(){
    const q = searchInput.value.trim().toLowerCase();
    if(q.length < 3){
        results.innerHTML="";
        return;
    }

    if(typeSelect.value==="books") searchBooks(q);
    else if(typeSelect.value==="movies") searchMovies(q);
    else searchGames(q);
}

function searchBooks(q){
    fetch(`https://www.googleapis.com/books/v1/volumes?q=${q}`)
    .then(r=>r.json())
    .then(d=>{
        if(!d.items || d.items.length===0){
            showNoResults();
        }else{
            renderResults(d.items,"book");
        }
    });
}

function searchMovies(q){
    fetch(`https://api.tvmaze.com/search/shows?q=${q}`)
    .then(r=>r.json())
    .then(d=>{
        if(!d || d.length===0){
            showNoResults();
        }else{
            renderResults(d,"movie");
        }
    });
}

// ================= Videojuegos =================
function searchGames(q){
    const qNormalized = q.normalize("NFD").replace(/[\u0300-\u036f]/g, "").toLowerCase();
    fetch("https://api.codetabs.com/v1/proxy?quest=https://www.freetogame.com/api/games")
    .then(r=>r.json())
    .then(data=>{
        const filtered = data.filter(g => {
            const title = g.title?.normalize("NFD").replace(/[\u0300-\u036f]/g, "").toLowerCase() || "";
            const desc  = g.short_description?.normalize("NFD").replace(/[\u0300-\u036f]/g, "").toLowerCase() || "";
            return title.includes(qNormalized) || desc.includes(qNormalized);
        });
        if(filtered.length===0){
            showNoResults();
        }else{
            renderResults(filtered.slice(0,20), "game"); // mostrar máximo 20
        }
    })
    .catch(()=>{
        results.innerHTML = `
            <div style="grid-column:1/-1;text-align:center;opacity:.7">
                Error al cargar videojuegos
            </div>`;
    });
}

function showNoResults(){
    results.innerHTML = `
        <div style="grid-column:1/-1;text-align:center;font-size:20px;opacity:.7">
            No se encontró ninguna coincidencia
        </div>`;
}

function renderResults(items,type){
    results.innerHTML="";
    items.forEach(i=>{
        let title,img;

        if(type==="book"){
            title = i.volumeInfo.title;
            img = i.volumeInfo.imageLinks?.thumbnail;
        }
        else if(type==="movie"){
            title = i.show.name;
            img = i.show.image?.medium;
        }
        else if(type==="game"){
            title = i.title;
            img = i.thumbnail;
        }

        const card=document.createElement("div");
        card.className="card";
        card.innerHTML=`
            <img src="${img||''}">
            <div class="info">
                <h4>${title}</h4>
                <div class="stars">${starsHTML(0)}</div>
                <button class="save">Guardar</button>
            </div>
        `;
        enableStars(card);
        card.querySelector(".save").onclick=()=>{
            const rating=card.querySelectorAll(".active").length;
            library.push({title,img,rating,type});
            save();
            renderLibrary();
        };
        results.appendChild(card);
    });
}

function renderLibrary(){
    libraryDiv.innerHTML="";
    library.forEach((i,idx)=>{
        const card=document.createElement("div");
        card.className="card";
        card.innerHTML=`
            <img src="${i.img||''}">
            <div class="info">
                <h4>${i.title}</h4>
                <div class="stars">${starsHTML(i.rating)}</div>
                <button class="remove">Eliminar</button>
            </div>
        `;
        card.querySelector(".remove").onclick=()=>{
            library.splice(idx,1);
            save();
            renderLibrary();
        };
        libraryDiv.appendChild(card);
    });
}

function starsHTML(n){
    let s="";
    for(let i=1;i<=5;i++)
        s+=`<span class="${i<=n?'active':''}">★</span>`;
    return s;
}

function enableStars(card){
    const stars=card.querySelectorAll(".stars span");
    stars.forEach((s,i)=>{
        s.onclick=()=>{
            stars.forEach(x=>x.classList.remove("active"));
            for(let j=0;j<=i;j++) stars[j].classList.add("active");
        };
    });
}

function save(){
    localStorage.setItem("vitvisor",JSON.stringify(library));
}

renderLibrary();
</script>

</body>
</html>
