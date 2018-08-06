<template>
  <div id="app">
    <p>Долгота : {{longitude}}</p>
    <p>Широта : {{latitude}}</p>
    <p>Темпервтура: {{this.data.main.temp}}</p>
    <!-- <p>Ощущается как: {{this.data.fact.feels_like}}</p> -->
    <img src="https://openweathermap.org/img/w/03d.png" alt="">
    <button @click="getCoordinate">Вычеслить координаты</button>
    <button @click="getWeather">Получить погоду</button>
    <button @click="getAutocomplete">Получить автозаполнение</button>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'app',
  data () {
    return {
      longitude: '',
      latitude: '',
      data: {
        main: {
          temp: '',
          feels_like: ''
        }
      }
    }
  },
  created () {
    this.getCoordinate()
    this.getWeather()
  },
  methods: {
    getCoordinate () {
      navigator.geolocation.getCurrentPosition((position) => {
        this.longitude = position.coords.longitude
        this.latitude = position.coords.latitude
      })
    },
    getWeather () {
      const key = '&APPID=0072928dcc9b5b634bfff2cbf46fe606'
      let url = 'http://api.openweathermap.org/data/2.5/weather?lat=47.22253036499023&lon=39.71870422363281&units=metric&lang=ru'
      axios.get(url + key)
        .then(({data}) => {
          // console.log(data)
          this.data = data
        })
        .catch(console.warn)
    },
    getAutocomplete () {
      const key = 'AIzaSyC9vBiV87p-d2p9Obp-uU3rf4OVAEKOpjg'
      const baseurl = 'https://maps.googleapis.com/maps/api/place/autocomplete/json?'
      let customurl = 'input=Kie&key='
      axios.get(baseurl + customurl + key)
        .then(({data}) => {
          console.log(data)
        })
        .catch(console.warn)
    }
  }
}
</script>

<style lang="scss">
img {
  width:50px;
}

</style>
