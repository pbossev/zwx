<!-- src/routes/+page.svelte -->
<script>
  import { onMount } from "svelte";

  let searchInput = "";
  let locations = [];
  let selectedLocation = null;
  let weatherData = null;

  async function searchLocations(query) {
    if (query.trim().length < 3) {
      locations = [];
      return;
    }

    const response = await fetch(
      `https://geocoding-api.open-meteo.com/v1/search?name=${encodeURIComponent(query)}&count=5&language=en&format=json`
    );
    const data = await response.json();
    locations = data.results;
  }

  async function fetchWeatherData(latitude, longitude) {
    const response = await fetch(
      `https://api.open-meteo.com/v1/forecast?latitude=${latitude}&longitude=${longitude}&current_weather=true`
    );
    const data = await response.json();
    weatherData = data.current_weather;
  }

  function selectLocation(location) {
    selectedLocation = location;
    searchInput = `${location.name}, ${location.admin1 ? location.admin1 + ", " : ""}${location.country}`;
    locations = [];
    fetchWeatherData(location.latitude, location.longitude);
  }

  onMount(() => {
    const inputElement = document.getElementById("search-input");
    inputElement.addEventListener("keydown", (event) => {
      if (event.key === "ArrowDown") {
        event.preventDefault();
        const activeIndex = locations.findIndex(
          (location) => location === selectedLocation
        );
        const nextIndex = (activeIndex + 1) % locations.length;
        selectedLocation = locations[nextIndex];
      } else if (event.key === "ArrowUp") {
        event.preventDefault();
        const activeIndex = locations.findIndex(
          (location) => location === selectedLocation
        );
        const prevIndex =
          (activeIndex - 1 + locations.length) % locations.length;
        selectedLocation = locations[prevIndex];
      } else if (event.key === "Enter" && selectedLocation) {
        event.preventDefault();
        selectLocation(selectedLocation);
      }
    });
  });
</script>

<main>
  <h3>
    This project (zwx) is a work in progress, this current commit is just to
    preserve the domain name
  </h3>
  <h1>Location Search</h1>
  <div>
    <input
      id="search-input"
      type="text"
      placeholder="Search for a location..."
      bind:value={searchInput}
      on:input={() => searchLocations(searchInput)}
    />
    {#if locations.length > 0}
      <ul>
        {#each locations as location}
          <li
            class:selected={selectedLocation === location}
            on:click={() => selectLocation(location)}
          >
            {location.name}, {location.admin1
              ? location.admin1 + ", "
              : ""}{location.country}
          </li>
        {/each}
      </ul>
    {/if}
  </div>
  {#if selectedLocation}
    <div>
      <h2>Selected Location:</h2>
      <p>Name: {selectedLocation.name}</p>
      <p>Country: {selectedLocation.country}</p>
      {#if selectedLocation.admin1}
        <p>State/Region: {selectedLocation.admin1}</p>
      {/if}
      <p>Latitude: {selectedLocation.latitude}</p>
      <p>Longitude: {selectedLocation.longitude}</p>
    </div>
    {#if weatherData}
      <div>
        <h2>Current Weather Conditions:</h2>
        <p>Temperature: {weatherData.temperature} °C</p>
        <p>Wind Speed: {weatherData.windspeed} m/s</p>
        <p>Wind Direction: {weatherData.winddirection} degrees</p>
        <p>Weather Code: {weatherData.weathercode}</p>
      </div>
    {/if}
  {/if}
</main>

<style>
  .selected {
    background-color: #f0f0f0;
  }
</style>
