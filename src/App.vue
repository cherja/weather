<template>
  <div id="app">
    <button class="button_open_weather" @click="isShowModalWeather = true"></button>
    <div class="widget-weather" @click="isShowWidgetWeather = false" v-show="isShowWidgetWeather">
      <div class="current-weather">
        <div class="icon-block">
          <weather-icon
            :code="data.list[0].weather[0].icon"
            :time-of-day="this.weather.timeOfDay"
          />
        </div>
        <div :class="['indication-block', { 'current-weather_night': night}]">
          <p class="indication-block__temp">{{roundRound}}°</p>
          <p>{{capitalizeFirstLetter}}</p>
        </div>
        <!-- <button class="widget-weather_close"  @click="isShowWidgetWeather = false">Х</button> -->
      </div>
    </div>
    <transition name="modal">
      <div v-show="isShowModalWeather" class="modal-mask">
        <div class="modal-container">
          <h1>Настройки</h1>
          <div class="settings-weather">
            <h2>Настройки погоды</h2>
            <p>Тема:</p>
            <select v-model="theme">
              <option v-for="(item,i) in themes" :key="i">{{item}}</option>
            </select>
            <p>Единица измерения</p>
            <select v-model="unit">
              <option v-for="(item,i) in units" :key="i">{{item}}</option>
            </select>
          </div>
          <div class="settings-location">
            <h2>Настройки локации</h2>
            <p><input type="checkbox" @click="getCoordinate">Использовать мою текущую позицию</p>
            <p>
              Адрес для погоды
            </p>
            <input
              ref="autocomplete"
              placeholder="Search"
              class="search-location"
              type="text"
            />
          </div>
        <button :disabled="dis" @click="isShowWidgetWeather = true, isShowModalWeather = false, $refs.autocomplete.value = ''" class="modal-container__save">
          Сохранить
        </button>
        <button class="modal-container__close" @click="isShowModalWeather = false">
          Закрыть
        </button>
        </div>
      </div>
    </transition>
    <input type="text" v-model="myCodes">
    <weather-icon
      :code="this.myCodes"
      :time-of-day="this.weather.timeOfDay"
    />
  </div>
</template>

<script>
import axios from 'axios'
import weatherIcon from './components/weatherIcon.vue'
const APPID = '0072928dcc9b5b634bfff2cbf46fe606'

export default {
  components: {
    weatherIcon
  },
  name: 'app',
  data () {
    return {
      myCodes: '02d',
      weather: {
        timeOfDay: 'day'
      },
      dis: true,
      isShowModalWeather: false,
      isShowWidgetWeather: false,
      night: false,
      themes: [
        'День',
        'Ночь'
      ],
      theme: 'День',
      units: [
        'По цельсию',
        'По Фаренгейту'
      ],
      unit: 'По цельсию',
      unitApi: 'metric',
      data: {
        list: [
          {
            main: {
              temp: ''
            },
            weather: [
              {
                description: '',
                icon: '02n'
              }
            ]
          }
        ]
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
      // navigator.geolocation.getCurrentPosition((position) => {
      //   this.getWeather(position.coords.longitude, position.coords.latitude)
      //   this.$refs.autocomplete.value = ''
      // })
      this.getWeather(39.72328, 47.23135)
      this.$refs.autocomplete.value = ''
    },
    getWeather (lon, lat) {
      const url = `http://api.openweathermap.org/data/2.5/forecast?lat=${lat}&lon=${lon}&units=${this.unitApi}&lang=ru&APPID=${APPID}`
      axios.get(url)
        .then(({data}) => {
          console.log(data)
          this.data = data
          this.dis = false
        })
        .catch(console.warn)
    }
  },
  computed: {
    capitalizeFirstLetter () {
      return this.data.list[0].weather[0].description.charAt(0).toUpperCase() + this.data.list[0].weather[0].description.slice(1)
    },
    roundRound () {
      return Math.round(this.data.list[0].main.temp)
    }
  },
  watch: {
    theme: function () {
      if (this.theme === 'Ночь') {
        this.night = true
      } else {
        this.night = false
      }
    },
    unit () {
      if (this.unit === 'По цельсию') {
        this.unitApi = 'metric'
      } else {
        this.unitApi = 'imperial'
      }
    }
  }
}
</script>

<style lang="scss">

.indication-block {
 font-size: 40px;
 p {
  margin: 10px;
 }
 .indication-block__temp {
   font-size: 50px;
 }
}

.current-weather {
  display: flex;
  position: relative;
}

.widget-weather_close {
  position: absolute;
  top: 1px;
  right: 1px;
  outline: none;
  border: none;
  background: transparent;
}

.search-location {
  width: 300px;
}
.settings-location {
  padding: 20px;
  margin: 20px;
  margin-left: 0px;
  padding-left: 0px;
}

.widget-weather {
  display: flex;
}

.current-weather_night {
  color: rgb(129, 129, 129);
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
