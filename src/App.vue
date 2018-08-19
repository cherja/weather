<template>
  <div id="app">
    <button class="button_open_modal-weather " @click="isShowModalWeather = true"></button>
    <div class="widget-weather" v-show="isShowWidgetWeather">
      <div class="current-weather" @click="isShowWidgetWeather = false">
        <div class="icon-block">
          <weather-icon :code="data.list[0].weather[0].icon" />
        </div>
        <div :class="['indication-block', { 'current-weather_light': light}]">
          <p class="indication-block__temp">{{roundRound}}°</p>
          <p>{{capitalizeFirstLetter}}</p>
        </div>
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
              <option v-for="(theme,i) in themes" :key="i">{{theme}}</option>
            </select>
            <p>Единица измерения</p>
            <select v-model="unit">
              <option v-for="(unit,i) in units" :key="i">{{unit}}</option>
            </select>
          </div>
          <div class="settings-location">
            <h2>Настройки локации</h2>
            <p><input type="checkbox" id="checkbox" v-model="checked" @click="getCoordinate(), searchLocationInput = !searchLocationInput">Использовать мою текущую позицию</p>
            <div :class="{ 'search-location': searchLocationInput }">
              <p>
                Адрес для погоды
              </p>
              <input
                ref="autocomplete"
                placeholder="Search"
                class="search-location__input"
                type="text"
                @click="checked = false"
              />
            </div>
          </div>
          <div class="modal-container__btns">
            <button :disabled="disableBtnShowWeather" @click="isShowWidgetWeather = true, isShowModalWeather = false " class="modal-container__save">
              Сохранить
            </button>
            <button class="modal-container__close" @click="isShowModalWeather = false">
              Закрыть
            </button>
          </div>
        </div>
      </div>
    </transition>
    <div>
      <h1>Завтра</h1>
      <weather-day :obj="generetadArray[1]" />
    </div>
    <div>
      <h1>Послезавтра</h1>
      <weather-day :obj="generetadArray[2]" />
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import weatherIcon from './components/weatherIcon.vue'
import weatherDay from './components/weatherDay.vue'
const APPID = '0072928dcc9b5b634bfff2cbf46fe606'

export default {
  components: {
    weatherIcon,
    weatherDay
  },
  name: 'app',
  data () {
    return {
      array: [],
      arrayDay: [],
      generetadArray: [
        {
          temps: [0]
        },
        {
          temps: [0]
        },
        {
          temps: [0]
        }
      ],
      checked: false,
      myCodes: '02d',
      searchLocationInput: false,
      disableBtnShowWeather: true,
      isShowModalWeather: false,
      isShowWidgetWeather: false,
      light: false,
      themes: [
        'По умолчанию',
        'Светлая'
      ],
      theme: 'По умолчанию',
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
    },
    getWeather (lon, lat) {
      const url = `http://api.openweathermap.org/data/2.5/forecast?lat=${lat}&lon=${lon}&units=${this.unitApi}&lang=ru&APPID=${APPID}`
      axios.get(url)
        .then(({data}) => {
          this.data = data
          this.disableBtnShowWeather = false
          this.array = data.list
          console.log(data.list)
          this.array.forEach((item) => {
            let date = new Date(item.dt_txt).toLocaleDateString()
            if (!this.arrayDay.includes(date)) {
              this.arrayDay.push(date)
            }
          })
          var generetadArray = []
          for (var i = 0; i < this.arrayDay.length; i++) {
            generetadArray.push({date: this.arrayDay[i], temps: []})
          }
          for (let i = 0; i < generetadArray.length; i++) {
            for (var j = 0; j < this.array.length; j++) {
              var da = new Date(this.array[j].dt_txt)
              if (generetadArray[i].date === da.toLocaleDateString()) {
                generetadArray[i].temps.push(this.array[j].main.temp)
              }
            }
          }
          this.generetadArray = generetadArray
          console.log(this.generetadArray)
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
  filters: {
    maxTemp (value) {
      return Math.round(Math.max(...value))
    },
    minTemp (value) {
      return Math.round(Math.min(...value))
    }
  },
  watch: {
    theme: function () {
      if (this.theme === 'Светлая') {
        this.light = true
      } else {
        this.light = false
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
.button_open_modal-weather {
  width: 100px;
  height: 100px;
  outline: none;
  background-position: center center;
  background-size: contain;
  background-repeat: no-repeat;
  background-image: url(./weather.jpg);
  border: none;
}

.indication-block {
  font-size: 40px;
  p {
    margin: 10px;
  }
  .indication-block__temp {
    font-size: 50px;
  }
}

.widget-weather {
  display: flex;
}

.modal-container__btns {
  display: flex;
  justify-content: flex-end;
}

.settings-location {
  padding: 20px;
  margin: 20px;
  margin-left: 0px;
  padding-left: 0px;
}

.search-location {
  visibility: hidden;
}

.search-location__input {
  width: 300px;
  margin-bottom: 100px;
}

.current-weather {
  display: flex;
  position: relative;
}

.current-weather_light {
  color: rgb(129, 129, 129);
}

.modal-mask {
  position: absolute;
  z-index: 9;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.1);
  display: flex;
  align-items: center;
  justify-content: center;
  transition: opacity 0.3s ease;
}

.modal-container {
  width: 580px;
  margin: 0px auto;
  padding: 20px 30px;
  background-color: #ffffff;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.33);
  transition: all 0.3s ease;
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
