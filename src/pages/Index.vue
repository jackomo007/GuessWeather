<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-pt-lg q-px-md">
      <q-input
      v-model="search"
      @keyup.enter="getWeatherBySearch"
      placeholder="Tell me where..."
      borderless
      dark
      >
        <template v-slot:before>
          <q-icon
            @click="getLocation"
            name="my_location"
          />
        </template>

        <template v-slot:append>
          <q-btn round dense flat icon="search"  @click="getWeatherBySearch" />
        </template>
      </q-input>
    </div>

    <template v-if="weatherData">
      <div class="col text-white text-center">
        <div class="text-h4 text-weight-light">
          {{weatherData.name}}
        </div>
        <div class="text-h6 text-weight-light">
          {{weatherData.weather[0].main}}
        </div>
        <div class="text-h1 text-weight-thin q-my-lg">
          <span>{{ Math.round(weatherData.main.temp)}}</span>
          <span class="text-h4 relative-position degree">&deg;C</span>
        </div>
      </div>

      <div class="col text-center">
        <img :src="`https://openweathermap.org/img/wn/${weatherData.weather[0].icon}@2x.png`">
      </div>
    </template>

    <template v-else>
      <div class="col column text-center text-white">
        <div class="col text-h2 text-weight-thin">
          Guess<br>Weather
        </div>
        <q-btn flat class="col" @click="getLocation">
          <q-icon left size="3em" name="my_location" />
          <div>Guess where i am...</div>
        </q-btn>
      </div>
    </template>

    <div class="col city"></div>
  </q-page>
</template>

<script>
export default {
  name: 'PageIndex',
  data() {
    return {
      search: '',
      weatherData: null,
      weatherKey: 'c65ad845185d5c6fc0d1063d6e5634e4',
      weatherUrl: 'https://api.openweathermap.org/data/2.5/weather',
      lat: 0,
      lon: 0
    }
  },
  computed: {
    bgClass () {
      if(this.weatherData) {
        if (this.weatherData.weather[0].icon.endsWith('n')) {
          return 'bg-night'
        } else {
          return 'bg-day'
        }
      }
    }
  },
  methods: {
    getLocation() {
      this.$q.loading.show()
      if(this.$q.platform.is.electron) {
        // this.$axios.get('https://freegeoip.app/json/').then(response => {
        //   this.lat =  response.data.latitude;
        //   this.lon =  response.data.longitude;
        // }).catch(error => {
        //   console.log(error)
        //   this.$q.loading.hide()
        // }).finally(() => this.getWeatherByCoords())
      }else{
        navigator.geolocation.getCurrentPosition(position => {
          this.lat =  position.coords.latitude;
          this.lon =  position.coords.longitude;
          this.getWeatherByCoords(
              position.coords.latitude,
              position.coords.longitude
            );
        })
      }
    },
    getWeatherByCoords(lat, lon) {
      this.$axios(
        `${this.weatherUrl}?lat=${lat}&lon=${lon}&appid=${this.weatherKey}&units=metric`
      ).then(response => {
        this.weatherData = response.data
        this.$q.loading.hide()
      })
    },
     getWeatherBySearch() {
      this.$q.loading.show()
      this.$axios(`${this.weatherUrl}?q=${this.search}&appid=${this.weatherKey}&units=metric`)
      .then(response => {
        this.weatherData = response.data
        this.search = ''
        this.$q.loading.hide()
      })
    }
  }
}
</script>

<style lang="sass">
  .q-page
    background: linear-gradient(to bottom, #642b73, #c6426e)
    &.bg-night
      background: linear-gradient(to bottom, #232526, #414345)
    &.bg-day
      background: linear-gradient(to bottom, #00c6ff, #0072ff)
  .degree
    top: -44px
  .city
    flex: 0 0 200px
    background: url(/city2.png)
    background-size: contain
    background-position: center bottom
</style>
