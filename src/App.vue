<template>
  <div id="app">
    <button class="button_open_weather" @click="isShowModalWeather = true, isShowWidgetWeather = true"></button>
    <div :class="['widget-weather', { 'widget-weather_night': night}]" v-show="isShowWidgetWeather">
      <p>Темпервтура: {{this.data.main.temp}}</p>
      <img :src="`https://openweathermap.org/img/w/${this.data.weather[0].icon}.png`" alt="">
    </div>
    <transition name="modal">
      <div v-show="isShowModalWeather" class="modal-mask">
        <div class="modal-container">
          <h1>Настройки</h1>
          <div class="settings-weather">
            <h2>Настройки погоды</h2>
            <p>Тема:</p>
            <select v-model="themes">
              <option v-for="(item,i) in theme" :key="i">{{item}}</option>
            </select>
          </div>
        <button @click="getCoordinate">Текущее местоположение</button>
        <input ref="autocomplete"
          placeholder="Search"
          class="search-location"
          type="text"
        />
        <button class="modal-container__close" @click="isShowModalWeather = false">
          Закрыть
        </button>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
import axios from 'axios'
const APPID = '0072928dcc9b5b634bfff2cbf46fe606'

export default {
  name: 'app',
  data () {
    return {
      isShowModalWeather: false,
      isShowWidgetWeather: false,
      themes: 'День',
      night: false,
      theme: [
        'День',
        'Ночь'
      ],
      data: {
        address: '',
        main: {
          temp: '',
          feels_like: ''
        },
        weather: [{
          icon: '03n'
        }]
      }
    }
  },
  mounted () {
    const google = window.google
    this.autocomplet = new google.maps.places.Autocomplete(
      (this.$refs.autocomplete),
      {types: ['geocode']}
    )
    this.autocomplet.addListener('place_changed', () => {
      let place = this.autocomplet.getPlace()
      let lat = place.geometry.location.lat()
      let lon = place.geometry.location.lng()
      this.getWeather(lon, lat)
    })
  },
  methods: {
    getCoordinate () {
      navigator.geolocation.getCurrentPosition((position) => {
        this.getWeather(position.coords.longitude, position.coords.latitude)
      })
    },
    getWeather (lon, lat) {
      const url = `http://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&units=metric&lang=ru&APPID=${APPID}`
      axios.get(url)
        .then(({data}) => {
          console.log(data)
          this.data = data
        })
        .catch(console.warn)
    }
  },
  watch: {
    themes: function () {
      if (this.themes === 'Ночь') {
        this.night = true
      } else {
        this.night = false
      }
    }
  }
}
</script>

<style lang="scss">

img {
  width:50px;
}
.search-location {
  width: 300px;
}

.widget-weather {
  background-color: rgba(30, 219, 233, 0.1);
}

.widget-weather_night {
  background-color: rgba(0, 0, 0, .3);
}

.button_open_weather {
  width: 100px;
  height: 100px;
  outline: none;
  background-position: center center;
  background-size: contain;
  background-repeat: no-repeat;
  background-image: url(./weather.jpg);
  border:none;
}

.modal-mask {
  position: fixed;
  z-index: 9;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, .1);
  display: flex;
  align-items: center;
  justify-content: center;
  transition: opacity .3s ease;
  padding: 0px 20px;
}

.modal-container {
  position: relative;
  width: 580px;
  margin: 0px auto;
  padding: 20px 30px;
  background-color: #ffffff;
  box-shadow: 0 2px 8px rgba(0, 0, 0, .33);
  transition: all .3s ease;
  font-family: Helvetica, Arial, sans-serif;
}

.modal-enter {
  opacity: 0;
}

.modal-leave-active {
  opacity: 0;
}

.modal-enter .modal-container,
.modal-leave-active .modal-container {
  -webkit-transform: scale(1.1);
  transform: scale(1.1);
}
</style>
