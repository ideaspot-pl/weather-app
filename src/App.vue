<script>
// import HelloWorld from './components/HelloWorld.vue'
// import TheWelcome from './components/TheWelcome.vue'

const STATUS_IDLE = 'idle';
const STATUS_GEOLOCATION = 'geolocation';
const STATUS_API = 'api';

export default {
  data() {
    return {
      apiKey: "YOUR_KEY",
      location: '',
      latitude: undefined,
      longitude: undefined,
      count: 0,
      status: STATUS_IDLE,
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
      this.status = STATUS_API;
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
            this.status = STATUS_IDLE;
          })
    },

    checkCoords() {
      if (navigator.geolocation) {
        this.status = STATUS_GEOLOCATION;
        navigator.geolocation.getCurrentPosition((pos) => {
          this.latitude = pos.coords.latitude;
          this.longitude = pos.coords.longitude;

          this.status = STATUS_API;

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
                this.status = STATUS_IDLE;
              })

        }, (error) => {
          console.error(error);
          this.status = STATUS_IDLE;
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
      <button type="button" :disabled="status != 'idle'" @click="checkCoords" id="check-coords">
        <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-cursor" viewBox="0 0 16 16">
          <path d="M14.082 2.182a.5.5 0 0 1 .103.557L8.528 15.467a.5.5 0 0 1-.917-.007L5.57 10.694.803 8.652a.5.5 0 0 1-.006-.916l12.728-5.657a.5.5 0 0 1 .556.103zM2.25 8.184l3.897 1.67a.5.5 0 0 1 .262.263l1.67 3.897L12.743 3.52z"/>
        </svg>
      </button>
      <input type="text" v-model="location" :disabled="status != 'idle'">
      <button type="submit" :disabled="status != 'idle'" id="check">Check!</button>
    </form>
  </header>

  <div class="messages">
    <div v-if="status == 'geolocation'" class="message">Getting location...</div>
    <div v-if="status == 'api'" class="message">Processing...</div>
  </div>

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
