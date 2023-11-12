<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-pt-lg q-px-md">
      <q-input
        v-model="search"
        @keyup.enter="getWeatherBySearch"
        placeholder="Search"
        dark
        borderless
      >
        <template v-slot:before>
          <q-icon
            @click="getLocation"
            name="my_location"
          />
        </template>

        <template v-slot:hint>
          Field hint
        </template>

        <template v-slot:append>
          <q-btn
            @click="getWeatherBySearch"
            icon="search"
            dense
            flat
            round
          />
        </template>
      </q-input>
    </div>
    <template v-if="weatherData">
      <div class="col text-white text-center">
        <div class="text-h4 text-weight-light">
          {{ weatherData.name }}
        </div>
        <div class="text-h6 text-weight-light">
          {{ weatherData.weather[0].main }}
        </div>
        <div class="text-h1 text-weight-thin q-my-lg relative-position">
          <span>{{ Math.round(weatherData.main.temp) }}</span>
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
          Quasar<br>Weather
        </div>
        <q-btn
          @click="getLocation"
          class="col"
          flat
        >
          <q-icon left size="3em" name="my_location" />
          <div>Find my location</div>
        </q-btn>
      </div>
    </template>

    <div class="col city">

    </div>
  </q-page>
</template>

<script setup>

  import { ref, computed } from 'vue' 
  import axios from 'axios';
  import { Loading, QSpinnerFacebook } from 'quasar'

  const weatherData = ref(null);
  let lat = null;
  let lon = null;
  const apiKey = '2d535052407f70a1221bb14d35eead03';
  const search = ref(null);

  const LoadingSpinner = () => {
    Loading.show({
      spinner: QSpinnerFacebook,
      spinnerColor: 'white',
      spinnerSize: 140,
      backgroundColor: 'black',
      message: 'Getting weather info',
      messageColor: 'white'
    });
  } 

  const getLocation = () => {
    LoadingSpinner();
    navigator.geolocation.getCurrentPosition(position => {
      lat = position.coords.latitude;
      lon = position.coords.longitude;
      getWeatherByCoords();
      Loading.hide();
    })
  }

  const getWeatherByCoords = () => {
    LoadingSpinner();
    axios(`https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=${apiKey}&units=metric`)
    .then(response => {
      weatherData.value = response.data;
      Loading.hide();
    })
  }

  const getWeatherBySearch = () => {
    LoadingSpinner();
    axios(`https://api.openweathermap.org/data/2.5/weather?q=${search.value}&appid=${apiKey}&units=metric`)
    .then(response => {
      weatherData.value = response.data;
      Loading.hide();
    })
  }

  const bgClass = computed(() => {
    if (weatherData.value) {
      if (weatherData.value.weather[0].icon.endsWith('n')){
        return 'bg-night';
      }
      else {
        return 'bg-day'
      }
    }
  })

</script>

<style lang="sass">
.q-page
  background: linear-gradient(to bottom, #2b5876, #4e4376) 
  &.bg-night
    background: linear-gradient(to top, #141e30, #243b55) 
  &.bg-day
    background: linear-gradient(to bottom, #3a7bd5, #3a6073)
.degree
  top: -44px
.city
  flex: 0 0 240px
  background: url(../img/city.png)
  background-size: contain
  background-position: center bottom
</style>