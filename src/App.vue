<script setup>
import { computed, onMounted, ref } from "vue";

const token = ref((import.meta.env.VITE_SPOTIFY_TOKEN || "").trim());
const results = ref([]);
const currentTrack = ref(null);
const loading = ref(false);
const error = ref("");

const headers = computed(() => ({
  Authorization: `Bearer ${token.value}`,
  "Content-Type": "application/json",
}));

async function spotifyFetch(url) {
  const response = await fetch(url, { headers: headers.value });

  if (!response.ok) {
    const message = await response.text();
    if (response.status === 401) {
      throw new Error(
        "Token Spotify invalide ou expiré. Remplace VITE_SPOTIFY_TOKEN dans .env par un nouveau token.",
      );
    }
    throw new Error(`Spotify API error (${response.status}): ${message}`);
  }

  return response.json();
}

async function loadTracks() {
  if (!token.value) {
    error.value = "Token Spotify manquant dans le fichier .env";
    return;
  }

  loading.value = true;
  error.value = "";

  try {
    const data = await spotifyFetch(
      "https://api.spotify.com/v1/search?q=daft%20punk&type=track&limit=10",
    );

    results.value = data.tracks?.items || [];
  } catch (err) {
    error.value = err.message;
    results.value = [];
  } finally {
    loading.value = false;
  }
}

onMounted(() => {
  loadTracks();
});

function playTrack(track) {
  currentTrack.value = track;
}
</script>

<template>
  <div class="app-shell">
    <aside class="sidebar">
      <a class="brand" href="#" aria-label="Accueil Spotify">
        <span class="brand-mark">●</span>
        <span>Spotify</span>
      </a>

      <nav class="main-nav" aria-label="Navigation principale">
        <a class="nav-link active" href="#"><span>⌂</span> Accueil</a>
        <a class="nav-link" href="#"><span>⌕</span> Rechercher</a>
        <a class="nav-link" href="#"><span>▣</span> Ta bibliothèque</a>
      </nav>

      <div class="playlist-heading">
        <span>Ta bibliothèque</span>
        <button type="button" aria-label="Créer une playlist">+</button>
      </div>
      <nav class="playlist-links" aria-label="Playlists">
        <a href="#">Musiques likées</a>
        <a href="#">Mes favoris</a>
        <a href="#">Daft Punk Radio</a>
      </nav>

      <div class="sidebar-footer">
        <a href="#">Installer l'application</a>
      </div>
    </aside>

    <main class="content">
      <header class="topbar">
        <div class="history-buttons" aria-label="Historique">
          <button type="button" aria-label="Retour">‹</button>
          <button type="button" aria-label="Avancer">›</button>
        </div>
        <div class="profile-pill">
          <span class="profile-avatar">A</span> ZAKARIA <span>⌄</span>
        </div>
      </header>

      <section class="content-inner">
        <div class="intro">
          <p class="eyebrow">Découvre de nouveaux sons</p>
          <h1>Bonjour, ZAKARIA</h1>
          <p class="intro-copy">
            Les meilleurs titres de Daft Punk, réunis pour toi.
          </p>
        </div>

        <p v-if="error" class="error-message">{{ error }}</p>

        <section
          v-if="currentTrack"
          class="player-section"
          aria-label="Lecteur Spotify"
        >
          <div class="section-heading">
            <div>
              <p class="eyebrow">En cours de lecture</p>
              <h2>{{ currentTrack.name }}</h2>
            </div>
            <span class="playing-indicator">● Lecture</span>
          </div>
          <iframe
            :key="currentTrack.id"
            class="spotify-player"
            :src="`https://open.spotify.com/embed/track/${currentTrack.id}?utm_source=generator`"
            :title="`Lecteur Spotify : ${currentTrack.name}`"
            allow="
              autoplay;
              clipboard-write;
              encrypted-media;
              fullscreen;
              picture-in-picture;
            "
            loading="lazy"
          ></iframe>
        </section>

        <section class="tracks-section">
          <div class="section-heading">
            <div>
              <p class="eyebrow">Pour toi</p>
              <h2>Daft Punk — titres populaires</h2>
            </div>
            <span class="track-count">{{ results.length }} titres</span>
          </div>

          <div v-if="loading" class="loading">
            <span class="spinner"></span> Chargement de tes titres...
          </div>

          <ul v-else class="music-list">
            <li
              v-for="(track, index) in results"
              :key="track.id"
              class="music-item"
            >
              <span class="track-number">{{
                String(index + 1).padStart(2, "0")
              }}</span>
              <img
                :src="
                  track.album?.images?.[0]?.url ||
                  'https://images.unsplash.com/photo-1516280440614-37939bbacd81?auto=format&fit=crop&w=300&q=80'
                "
                :alt="track.name"
              />
              <div class="music-info">
                <h3>{{ track.name }}</h3>
                <p>
                  {{
                    track.artists?.map((artist) => artist.name).join(", ") ||
                    "Artiste inconnu"
                  }}
                </p>
              </div>
              <span class="album-name">{{
                track.album?.name || "Album inconnu"
              }}</span>
              <button
                class="listen-button"
                type="button"
                @click="playTrack(track)"
              >
                <span>▶</span> Écouter
              </button>
            </li>
          </ul>
        </section>
      </section>
    </main>
  </div>
</template>
