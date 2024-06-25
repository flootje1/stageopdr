<script>
	import { writable } from 'svelte/store';
  
	let country = '';
	let artist = '';
	let weatherData = writable(null);
	let spotifyData = writable(null);
	let loadingWeather = writable(false);
	let loadingSpotify = writable(false);
	let errorWeather = writable(null);
	let errorSpotify = writable(null);
	let activeSection = writable('weather'); 
  
	const weatherApiKey = 'YOUR-KEY-HERE'; 
	const spotifyClientId = 'YOUR-KEY-HERE';
	const spotifyClientSecret = 'YOUR-KEY-HERE'; 
  
	const fetchWeather = async () => {
	  loadingWeather.set(true);
	  errorWeather.set(null);
  
	  try {
		const response = await fetch(`https://api.weatherapi.com/v1/current.json?key=${weatherApiKey}&q=${country}`);
  
		if (!response.ok) {
		  throw new Error('Failed to fetch weather data');
		}
  
		const data = await response.json();
		weatherData.set(data);
	  } catch (err) {
		errorWeather.set(err.message);
	  } finally {
		loadingWeather.set(false);
	  }
	};
  
	const fetchSpotifyToken = async () => {
	  const response = await fetch('https://accounts.spotify.com/api/token', {
		method: 'POST',
		headers: {
		  'Content-Type': 'application/x-www-form-urlencoded',
		  'Authorization': `Basic ${btoa(`${spotifyClientId}:${spotifyClientSecret}`)}`
		},
		body: 'grant_type=client_credentials'
	  });
  
	  const data = await response.json();
	  return data.access_token;
	};
  
	const fetchSpotifyData = async () => {
	  loadingSpotify.set(true);
	  errorSpotify.set(null);
  
	  try {
		const token = await fetchSpotifyToken();
		const response = await fetch(`https://api.spotify.com/v1/search?q=${encodeURIComponent(artist)}&type=artist`, {
		  headers: {
			'Authorization': `Bearer ${token}`
		  }
		});
  
		if (!response.ok) {
		  throw new Error('Failed to fetch artist data');
		}
  
		const data = await response.json();
		const artistId = data.artists.items[0].id;
  
		const topTracksResponse = await fetch(`https://api.spotify.com/v1/artists/${artistId}/top-tracks?market=US`, {
		  headers: {
			'Authorization': `Bearer ${token}`
		  }
		});
  
		if (!topTracksResponse.ok) {
		  throw new Error('Failed to fetch top tracks');
		}
  
		const topTracksData = await topTracksResponse.json();
		spotifyData.set(topTracksData.tracks);
	  } catch (err) {
		errorSpotify.set(err.message);
	  } finally {
		loadingSpotify.set(false);
	  }
	};
  </script>
  
  <style>
	body {
	  margin: 0;
	  padding: 0;
	  font-family: 'Arial', sans-serif;
	  background: linear-gradient(135deg, #ff7e5f, #feb47b);
	  min-height: 100vh;
	  display: flex;
	  justify-content: center;
	  align-items: center;
	}
	main {
	  background-color: white;
	  padding: 20px;
	  border-radius: 10px;
	  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
	  max-width: 600px;
	  width: 100%;
	  text-align: center;
	  display: flex;
	  flex-direction: column;
	  align-items: center;
	}
	.section-buttons {
	  display: flex;
	  justify-content: center;
	  margin-bottom: 20px;
	}
	.section-buttons button {
	  padding: 10px 20px;
	  margin: 0 10px;
	  background-color: rgb(73, 73, 73);
	  color: white;
	  border: none;
	  border-radius: 10px;
	  transition: transform 0.3s, background-color 0.3s;
	  font-size: 16px;
	}
	.section-buttons button:hover {
	  background-color: rgba(38, 38, 38, 0.729);
	  cursor: pointer;
	  transform: scale(1.1);
	}
	.active {
	  background-color: black;
	}
	input {
	  margin: 10px 0;
	  padding: 10px;
	  border: 2px solid #ccc;
	  border-radius: 10px;
	  width: calc(100% - 24px);
	  font-size: 16px;
	  transition: border-color 0.3s;
	}
	input:focus {
	  outline: none;
	  border-color: #007BFF;
	}
	button {
	  padding: 10px 20px;
	  margin: 10px;
	  background-color: rgb(73, 73, 73);
	  color: white;
	  border: none;
	  border-radius: 10px;
	  transition: transform 0.3s, background-color 0.3s;
	  font-size: 16px;
	}
	button:hover {
	  background-color: rgba(38, 38, 38, 0.729);
	  cursor: pointer;
	  transform: scale(1.1);
	}
	.error {
	  color: red;
	}
	.data-container {
	  margin-top: 20px;
	  padding: 20px;
	  background-color: white;
	  border-radius: 10px;
	  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
	  width: 100%;
	  max-width: 500px;
	  text-align: center;
	  display: flex;
	  flex-direction: column;
	  align-items: center;
	}
	ul {
	  list-style-type: none;
	  padding: 0;
	}
	li {
	  margin: 10px 0;
	}
	audio {
	  margin-top: 10px;
	}
  </style>
  
  <main>
	<h1>Weather and Spotify App</h1>
	<div class="section-buttons">
	  <button class:active={$activeSection === 'weather'} on:click={() => activeSection.set('weather')}>Weather</button>
	  <button class:active={$activeSection === 'spotify'} on:click={() => activeSection.set('spotify')}>Spotify</button>
	</div>
  
	{#if $activeSection === 'weather'}
	  <div>
		<h2>Get Weather</h2>
		<input type="text" placeholder="Enter country" bind:value={country} />
		<button on:click={fetchWeather}>Get Weather</button>
	  </div>
  
	  {#if $loadingWeather}
		<p>Loading weather data...</p>
	  {:else if $errorWeather}
		<p class="error">Error: {$errorWeather}</p>
	  {:else if $weatherData}
		<div class="data-container">
		  <h2>Weather in {$weatherData.location.name}, {$weatherData.location.country}</h2>
		  <p>Temperature: {$weatherData.current.temp_c} °C</p>
		  <p>Condition: {$weatherData.current.condition.text}</p>
		  <img src="{$weatherData.current.condition.icon}" alt="{$weatherData.current.condition.text}" />
		</div>
	  {/if}
	{/if}
  
	{#if $activeSection === 'spotify'}
	  <div>
		<h2>Get Spotify Top Tracks</h2>
		<input type="text" placeholder="Enter artist" bind:value={artist} />
		<button on:click={fetchSpotifyData}>Get Top Tracks</button>
	  </div>
  
	  {#if $loadingSpotify}
		<p>Loading Spotify data...</p>
	  {:else if $errorSpotify}
		<p class="error">Error: {$errorSpotify}</p>
	  {:else if $spotifyData}
		<div class="data-container">
		  <h2>Top Tracks for {artist}</h2>
		  <ul>
			{#each $spotifyData as track}
			  <li>
				<p>{track.name}</p>
				<audio controls>
				  <source src={track.preview_url} type="audio/mpeg" />
				  Your browser does not support the audio element.
				</audio>
			  </li>
			{/each}
		  </ul>
		</div>
	  {/if}
	{/if}
  </main>
  
