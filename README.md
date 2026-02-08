<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<title>Selic Music</title>

<style>
body {
    margin: 0;
    font-family: Arial, sans-serif;
    background: linear-gradient(to right, #1e1e2f, #00a89d);
    color: white;
    text-align: center;
}

.container {
    margin-top: 50px;
}

input, textarea {
    padding: 10px;
    margin: 10px;
    border-radius: 10px;
    border: none;
    width: 300px;
}

button {
    padding: 10px 20px;
    font-size: 16px;
    border: none;
    border-radius: 20px;
    cursor: pointer;
    background-color: #ff4d6d;
    color: white;
}

button:hover {
    background-color: #e63950;
}

ul {
    list-style: none;
    padding: 0;
}

li {
    margin: 8px 0;
    cursor: pointer;
    text-decoration: underline;
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 10px;
}

li:hover {
    color: #ff4d6d;
}

.playing {
    color: #ff4d6d;
    font-weight: bold;
}

.delete {
    cursor: pointer;
    color: #ff4d6d;
    font-weight: bold;
}

.delete:hover {
    color: #ff9aa2;
}

audio {
    width: 80%;
    margin-top: 20px;
}
</style>
</head>

<body>

<div class="container">
    <h1>🎵 Selic Music 🎵</h1>

    <input type="file" id="fileInput" multiple accept="audio/*">
    <br>

    <textarea id="urlInput" placeholder="Colle tes liens mp3 ici (1 par ligne)" rows="3"></textarea>
    <br>
    <button onclick="ajouterURL()">Ajouter les URLs</button>

    <h2>Playlist</h2>
    <ul id="playlist"></ul>

    <audio id="audio" controls></audio>
</div>

<script>
const fileInput = document.getElementById("fileInput");
const urlInput = document.getElementById("urlInput");
const playlist = document.getElementById("playlist");
const audio = document.getElementById("audio");

let savedPlaylist = [];
let currentIndex = -1;

// afficher playlist
function afficherPlaylist() {
    playlist.innerHTML = "";

    savedPlaylist.forEach((item, index) => {
        const li = document.createElement("li");

        const name = document.createElement("span");
        name.textContent = item.name;
        name.onclick = () => jouerMusique(index);

        const del = document.createElement("span");
        del.textContent = "❌";
        del.className = "delete";
        del.onclick = (e) => {
            e.stopPropagation();

            if (index === currentIndex) {
                audio.pause();
                audio.src = "";
                currentIndex = -1;
            } else if (index < currentIndex) {
                currentIndex--;
            }

            savedPlaylist.splice(index, 1);
            afficherPlaylist();
        };

        if (index === currentIndex) {
            li.classList.add("playing");
        }

        li.appendChild(name);
        li.appendChild(del);
        playlist.appendChild(li);
    });
}

// jouer une musique
function jouerMusique(index) {
    currentIndex = index;

    audio.pause();
    audio.src = savedPlaylist[index].src;
    audio.load();
    audio.play().catch(() => {
        alert("Impossible de lire ce fichier audio");
    });

    afficherPlaylist();
}

// lecture automatique du suivant
audio.addEventListener("ended", () => {
    if (currentIndex + 1 < savedPlaylist.length) {
        jouerMusique(currentIndex + 1);
    }
});

// ajouter fichiers locaux
fileInput.addEventListener("change", () => {
    const files = Array.from(fileInput.files);

    files.forEach(file => {
        const url = URL.createObjectURL(file);
        savedPlaylist.push({
            name: file.name,
            src: url
        });
    });

    afficherPlaylist();
});

// ajouter URLs
function ajouterURL() {
    const urls = urlInput.value.trim().split("\n");

    urls.forEach(url => {
        if (url.trim() !== "") {
            savedPlaylist.push({
                name: url.trim(),
                src: url.trim()
            });
        }
    });

    urlInput.value = "";
    afficherPlaylist();
}
</script>

</body>
</html>
