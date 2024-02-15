<script>
// import HelloWorld from './components/HelloWorld.vue'
// import TheWelcome from './components/TheWelcome.vue'

export default {
  data() {
    return {
      apiKey: "1453d962089c33d0c55be24fd6147c88",
      location: '',
      latitude: undefined,
      longitude: undefined,
      count: 0,
      isProcessing: false,
      forecasts: [],
      language: 'en',
    }
  },
  mounted() {
    this.location = localStorage.getItem('location') || '';
    this.language = navigator.language;
  },
  computed: {
    forecastLink() {
      return `https://api.openweathermap.org/data/2.5/forecast?q=${this.location}&appid=${this.apiKey}&units=metric&lang=${this.language}`;
    },
    forecastLinkCoords() {
      return `https://api.openweathermap.org/data/2.5/forecast?lat=${this.latitude}&lon=${this.longitude}&appid=${this.apiKey}&units=metric&lang=${this.language}`;
    }
  },
  methods: {
    check() {
      this.isProcessing = true;
      localStorage.setItem('location', this.location);
      fetch(this.forecastLink)
          .then((response) => response.json())
          .then((data) => {
            console.debug(data);

            if (data.list) {
              this.forecasts = data.list;
            }
          })
          .finally(() => {
            this.isProcessing = false;
          })
    },

    checkCoords() {
      if (navigator.geolocation) {
        this.isProcessing = true;
        navigator.geolocation.getCurrentPosition((pos) => {
          this.latitude = pos.coords.latitude;
          this.longitude = pos.coords.longitude;

          fetch(this.forecastLinkCoords)
              .then((response) => response.json())
              .then((data) => {
                console.debug(data);
                if (data.city && data.city.name && data.city.country) {
                  this.location = `${data.city.name}, ${data.city.country}`;
                  localStorage.setItem('location', this.location);
                }
                if (data.list) {
                  this.forecasts = data.list;
                }
              })
              .finally(() => {
                this.isProcessing = false;
              })

        }, () => {
          this.isProcessing = false;
        })

      }
    },

    iconLink(iconName) {
      return `https://openweathermap.org/img/wn/${iconName}@2x.png`;
    },
  }
}
</script>

<template>
  <header>
    <form id="query-form" @submit.prevent="check">
      <button type="button" :disabled="isProcessing" @click="checkCoords" id="check-coords">
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 512 512"><!--!Font Awesome Free 6.5.1 by @fontawesome - https://fontawesome.com License - https://fontawesome.com/license/free Copyright 2024 Fonticons, Inc.--><path d="M256 0c17.7 0 32 14.3 32 32V66.7C368.4 80.1 431.9 143.6 445.3 224H480c17.7 0 32 14.3 32 32s-14.3 32-32 32H445.3C431.9 368.4 368.4 431.9 288 445.3V480c0 17.7-14.3 32-32 32s-32-14.3-32-32V445.3C143.6 431.9 80.1 368.4 66.7 288H32c-17.7 0-32-14.3-32-32s14.3-32 32-32H66.7C80.1 143.6 143.6 80.1 224 66.7V32c0-17.7 14.3-32 32-32zM128 256a128 128 0 1 0 256 0 128 128 0 1 0 -256 0zm128-80a80 80 0 1 1 0 160 80 80 0 1 1 0-160z"/></svg>
      </button>
      <input type="text" v-model="location" :disabled="isProcessing">
      <button type="submit" :disabled="isProcessing" id="check">Check!</button>
    </form>
  </header>

  <main id="weather-results-container">
    <div v-for="forecast in forecasts" :key="forecast.dt" class="weather-block">
      <div class="weather-date">{{ forecast.dt_txt }}</div>
      <div class="weather-temperature">{{ forecast.main.temp }} &deg;C</div>
      <div class="weather-temperature-fl">Like {{ forecast.main.feels_like }} &deg;C</div>
      <img class="weather-icon" :src="iconLink(forecast.weather[0].icon)">
      <div class="weather-description">{{ forecast.weather[0].description }}</div>
    </div>
  </main>

</template>

<style scoped>

</style>
