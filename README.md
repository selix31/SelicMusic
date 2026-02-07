<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<title>Selic Music</title>
<style>
body {
    margin: 0;
    font-family: Arial, sans-serif;
    background: linear-gradient(to right, #1e1e2f, #2b2b4f);
    color: white;
    text-align: center;
}

.container {
    margin-top: 50px;
}

#dropZone {
    margin: 20px auto;
    padding: 30px;
    width: 80%;
    border: 2px dashed #ff4d6d;
    border-radius: 20px;
    color: #ff4d6d;
    font-size: 18px;
}

ul {
    list-style: none;
    padding: 0;
}

li {
    margin: 5px 0;
    display: flex;
    justify-content: space-between;
    align-items: center;
}

li span {
    cursor: pointer;
}

li button {
    background: transparent;
    color: #ff4d6d;
    border: none;
    cursor: pointer;
    font-size: 16px;
}

li:hover span {
    color: #00c896;
}

audio {
    width: 80%;
    margin-top: 20px;
}

#visualizerContainer {
    margin: 30px auto;
    font-size: 100px; /* taille du happy face */
    transition: transform 0.1s ease;
}
</style>
</head>
<body>

<div class="container">
    <h1>🎵 Selic Music 🎵</h1>

    <div id="dropZone">Glisse tes fichiers mp3 ou liens mp3 ici</div>

    <h2>Playlist</h2>
    <ul id="playlist"></ul>

    <audio id="audio" controls></audio>

    <div id="visualizerContainer">😀</div>
</div>

<script>
const dropZone = document.getElementById("dropZone");
const playlist = document.getElementById("playlist");
const audio = document.getElementById("audio");
const visualizer = document.getElementById("visualizerContainer");

// AudioContext et analyser
const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
const analyser = audioCtx.createAnalyser();
let sourceCreated = false;

let savedPlaylist = JSON.parse(localStorage.getItem("selicPlaylist")) || [];

// Réveiller AudioContext sur premier clic
document.body.addEventListener("click", () => {
    if(audioCtx.state === "suspended") audioCtx.resume();
}, {once: true});

// Afficher la playlist avec bouton supprimer
function afficherPlaylist() {
    playlist.innerHTML = "";
    savedPlaylist.forEach((item, index) => {
        const li = document.createElement("li");

        const nameSpan = document.createElement("span");
        nameSpan.textContent = item.name;
        nameSpan.onclick = () => {
            audio.src = item.src;
            audio.play();
            // Créer le MediaElementSource **une seule fois** après le premier play
            if(!sourceCreated) {
                const source = audioCtx.createMediaElementSource(audio);
                source.connect(analyser);
                analyser.connect(audioCtx.destination);
                analyser.fftSize = 64;
                sourceCreated = true;
            }
        }

        const removeBtn = document.createElement("button");
        removeBtn.textContent = "❌";
        removeBtn.onclick = () => {
            savedPlaylist.splice(index, 1);
            sauvegarderPlaylist();
            afficherPlaylist();
        }

        li.appendChild(nameSpan);
        li.appendChild(removeBtn);
        playlist.appendChild(li);
    });
}

// Sauvegarder la playlist
function sauvegarderPlaylist() {
    localStorage.setItem("selicPlaylist", JSON.stringify(savedPlaylist));
}

// Drag & Drop
dropZone.addEventListener("dragover", e => {
    e.preventDefault();
    dropZone.style.borderColor = "#00c896";
});

dropZone.addEventListener("dragleave", e => {
    e.preventDefault();
    dropZone.style.borderColor = "#ff4d6d";
});

dropZone.addEventListener("drop", e => {
    e.preventDefault();
    dropZone.style.borderColor = "#ff4d6d";

    // Ajouter fichiers locaux
    const files = Array.from(e.dataTransfer.files);
    files.forEach(file => {
        if(file.type === "audio/mp3" || file.type === "audio/mpeg") {
            const url = URL.createObjectURL(file);
            savedPlaylist.push({name: file.name, src: url});
        }
    });

    // Ajouter liens glissés
    const text = e.dataTransfer.getData("text");
    if(text && text.endsWith(".mp3")) {
        savedPlaylist.push({name: text, src: text});
    }

    sauvegarderPlaylist();
    afficherPlaylist();
});

// Afficher playlist au chargement
afficherPlaylist();

// Visualizer Happy Face
function animateVisualizer() {
    const dataArray = new Uint8Array(analyser.frequencyBinCount);
    analyser.getByteFrequencyData(dataArray);
    let avg = dataArray.reduce((a,b)=>a+b,0)/dataArray.length;

    visualizer.style.transform = `scale(${1 + avg/200})`;

    if(!audio.paused) {
        requestAnimationFrame(animateVisualizer);
    } else {
        visualizer.style.transform = "scale(1)";
    }
}

// Lancer visualizer quand audio joue
audio.addEventListener("play", () => {
    animateVisualizer();
});
</script>

</body>
</html>
