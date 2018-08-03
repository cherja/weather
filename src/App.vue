<template>
  <div id="app">
    <p>Долгота : {{longitude}}</p>
    <p>Широта : {{latitude}}</p>
    <p>Темпервтура: {{this.data.fact.temp}}</p>
    <p>Ощущается как: {{this.data.fact.feels_like}}</p>
    <img src="https://yastatic.net/weather/i/icons/blueye/color/svg/skc_n.svg" alt="">
    <button @click="getCoordinate">Вычеслить</button>
    <button @click="getWeather">Получить погоду</button>
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
        fact: {
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
        console.log(this.data.fact.temp)
      })
    },
    getWeather () {
      let proxy = 'https://cors-anywhere.herokuapp.com/'
      let url = `http://api.weather.yandex.ru/v1/forecast?lat=${this.latitude}&lon=${this.longitude}&extra=true`
      axios.get(proxy + url, {
        'headers': {
          'X-Yandex-API-Key': 'f506555a-bb96-4c14-af56-e8643f58989d'
        }
      })
        .then(({data}) => (this.data = data))
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
