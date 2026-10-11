<!DOCTYPE html>
<html lang="hi">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Movie World - Trailers</title>
  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    }

    body {
      background-color: #141414;
      color: #fff;
    }

    header {
      padding: 1rem 2rem;
      display: flex;
      justify-content: space-between;
      align-items: center;
      background-color: #000;
      position: sticky;
      top: 0;
      z-index: 100;
    }

    header h1 {
      font-size: 1.8rem;
      color: #e50914;
    }

    .search {
      background-color: #222;
      border: 2px solid #333;
      border-radius: 20px;
      color: #fff;
      padding: 10px 20px;
      font-size: 1rem;
      outline: none;
    }

    .search:focus {
      border-color: #e50914;
    }

    main {
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      padding: 20px;
      gap: 25px;
    }

    .movie {
      width: 220px;
      background-color: #1f1f1f;
      border-radius: 10px;
      overflow: hidden;
      box-shadow: 0 4px 10px rgba(0, 0, 0, 0.7);
      display: flex;
      flex-direction: column;
      justify-content: space-between;
    }

    .movie img {
      width: 100%;
      height: 300px;
      object-fit: cover;
    }

    .movie-info {
      padding: 12px;
      display: flex;
      justify-content: space-between;
      align-items: center;
    }

    .movie-info h3 {
      font-size: 0.95rem;
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
      max-width: 130px;
    }

    .rating {
      background-color: #000;
      padding: 4px 6px;
      border-radius: 4px;
      font-size: 0.85rem;
      font-weight: bold;
    }

    .rating.green { color: #21d07a; }
    .rating.orange { color: #cece1a; }
    .rating.red { color: #d52121; }

    .trailer-btn {
      background-color: #e50914;
      color: white;
      border: none;
      padding: 10px;
      width: 100%;
      font-weight: bold;
      cursor: pointer;
      transition: background 0.2s;
    }

    .trailer-btn:hover {
      background-color: #b20710;
    }

    .modal {
      display: none;
      position: fixed;
      z-index: 1000;
      left: 0;
      top: 0;
      width: 100%;
      height: 100%;
      background-color: rgba(0, 0, 0, 0.85);
      justify-content: center;
      align-items: center;
    }

    .modal-content {
      position: relative;
      width: 80%;
      max-width: 700px;
      background: #000;
      padding: 20px;
      border-radius: 8px;
    }

    .close-btn {
      position: absolute;
      top: -10px;
      right: 10px;
      color: #fff;
      font-size: 35px;
      font-weight: bold;
      cursor: pointer;
    }

    #playerContainer iframe {
      width: 100%;
      height: 380px;
      border: none;
    }
  </style>
</head>
<body>

  <header>
    <h1>🎬 Movie World</h1>
    <form id="form">
      <input type="text" id="search" class="search" placeholder="Search movie...">
    </form>
  </header>

  <main id="main">
    <!-- Movie Cards Yahan Aayenge -->
  </main>

  <!-- YouTube Trailer Popup Modal -->
  <div id="trailerModal" class="modal">
    <div class="modal-content">
      <span class="close-btn">&times;</span>
      <div id="playerContainer">
        <!-- YouTube iframe yahan inject hoga -->
      </div>
    </div>
  </div>

  <script>
    const API_URL = 'https://api.themoviedb.org/3/discover/movie?sort_by=popularity.desc&api_key=3fd2be6f0c70a2a598f084dd0b6009c0&page=1';
    const IMG_PATH = 'https://image.tmdb.org/t/p/w1280';
    const SEARCH_API = 'https://api.themoviedb.org/3/search/movie?api_key=3fd2be6f0c70a2a598f084dd0b6009c0&query="';

    const main = document.getElementById('main');
    const form = document.getElementById('form');
    const search = document.getElementById('search');
    const modal = document.getElementById('trailerModal');
    const playerContainer = document.getElementById('playerContainer');
    const closeBtn = document.querySelector('.close-btn');

    // Initial load
    getMovies(API_URL);

    async function getMovies(url) {
        const res = await fetch(url);
        const data = await res.json();
        showMovies(data.results);
    }

    function showMovies(movies) {
        main.innerHTML = '';

        movies.forEach((movie) => {
            const { id, title, poster_path, vote_average } = movie;

            const movieEl = document.createElement('div');
            movieEl.classList.add('movie');

            movieEl.innerHTML = `
                <img src="${poster_path ? IMG_PATH + poster_path : 'https://via.placeholder.com/220x330'}" alt="${title}">
                <div class="movie-info">
                    <h3>${title}</h3>
                    <span class="rating ${getClassByRate(vote_average)}">⭐ ${vote_average.toFixed(1)}</span>
                </div>
                <button class="trailer-btn" onclick="openTrailer(${id})">▶ Watch Trailer</button>
            `;
            main.appendChild(movieEl);
        });
    }

    function getClassByRate(vote) {
        if(vote >= 8) return 'green';
        else if(vote >= 5) return 'orange';
        else return 'red';
    }

    async function openTrailer(movieId) {
        const res = await fetch(`https://api.themoviedb.org/3/movie/${movieId}/videos?api_key=3fd2be6f0c70a2a598f084dd0b6009c0`);
        const data = await res.json();
        
        if(data.results && data.results.length > 0) {
            const trailer = data.results.find(vid => vid.site === 'YouTube' && vid.type === 'Trailer') || data.results[0];
            
            if(trailer) {
                playerContainer.innerHTML = `<iframe src="https://www.youtube.com/embed/${trailer.key}?autoplay=1" allowfullscreen></iframe>`;
                modal.style.display = 'flex';
            } else {
                alert('Trailer available nahi hai!');
            }
        } else {
            alert('Trailer mil nahi paya!');
        }
    }

    closeBtn.addEventListener('click', () => {
        modal.style.display = 'none';
        playerContainer.innerHTML = '';
    });

    window.addEventListener('click', (e) => {
        if(e.target === modal) {
            modal.style.display = 'none';
            playerContainer.innerHTML = '';
        }
    });

    form.addEventListener('submit', (e) => {
        e.preventDefault();
        const searchTerm = search.value;

        if(searchTerm && searchTerm !== '') {
            getMovies(SEARCH_API + searchTerm);
            search.value = '';
        } else {
            window.location.reload();
        }
    });
  </script>
</body>
</html>






# All-in-one
