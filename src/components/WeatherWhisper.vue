<template>

  <div class="container">
    <div class="title">
      <h1 id="city">{{ cityName }}</h1>
      <div id="graphic" class="weather-icon">
        <img v-bind:src="weatherIcon" alt="Weather Icon" />
      </div>
      <div id="description" class="important-info text">{{ weatherDescription }}</div>
      <div id="farenheit" class="temp-info"></div>
      <div id="celsius" class="temp-info"></div>
    </div>
  </div>
  <div class="description-container">
    <div class="block-1">
      <div class="text">
        <span>Máx</span>
        <span id="max" class="important-info"> </span>
      </div>
      <div class="text">
        <span>Min</span>
        <span id="min" class="important-info"> </span>
      </div>
    </div>
    <div class="block-2">
      <div id="longitude" class=" text important-info">Lon: </div>
      <div id="latitude" class="text important-info">Lat: </div>
    </div>
    <div class="block3">
      <div id="humidity" class="text important-info">Humidade: </div>
      <div id="pressure" class="text important-info">Pressão Atmosférica: </div>
      <div id="windSpeed" class="text important-info">Vento: </div>
    </div>
  </div>
  <div class="container-button">
    <div class="switch-button">
      <span class="active"></span>
      <button class="text switch-button-case left" :class="{ 'active-case': displayCelsius }" @click="toggleTemperatureUnit('celsius')">Celsius</button>
      <button class="text switch-button-case right" :class="{ 'active-case': !displayCelsius }" @click="toggleTemperatureUnit('fahrenheit')">Fahrenheit</button>
    </div>
  </div>

</template>

<script>
import $ from 'jquery';
import _ from 'lodash';

export default {
  data() {
    return {
      lat: null,
      lon: null,
      cityName: '',
      weatherIcon: '',
      weatherDescription: '',
      celsius: null,
      displayCelsius: true,
      farenheit: null,
      apiCache: {}
    };
  },
  mounted() {
    this.fetchWeatherData();
  },
  methods: {
    // Usando a biblioteca Lodash para gerenciamento de cash
    async fetchWeatherData() {
    // ...
    const cacheKey = `${this.lat},${this.lon}`;
    if (this.apiCache[cacheKey] && Date.now() - this.apiCache[cacheKey].timestamp < 900 * 1000) {
      // Usar os dados em cache
      this.updateOtherElements(this.apiCache[cacheKey].data);
    } else {
      // Fazer solicitação à API
      $.getJSON(`https://api.openweathermap.org/data/2.5/weather?lat=${this.lat}&lon=${this.lon}&appid=059dcee9c15c93a942eb1f38b72876be`, (weatherData) => {
        // ...
        this.updateOtherElements(weatherData);
        // Armazenar dados no cache por 15 minutos (900 segundos)
        this.apiCache[cacheKey] = {
          data: weatherData,
          timestamp: Date.now()
        };
      });
    }
  },
    toggleTemperatureUnit(unit) {
      if (unit === 'celsius') {
        this.displayCelsius = true;
      } else {
        this.displayCelsius = false;
      }
      console.log('displayCelsius:', this.displayCelsius); // Adiciona esta linha
      this.fetchWeatherData(); // Chama novamente o método para buscar os dados meteorológicos
    },
    fetchWeatherData() {
      // Utiliza a geolocalização do usuário
      $.getJSON('http://ip-api.com/json', (data) => {
        this.lat = data.lat;
        this.lon = data.lon;

        $.getJSON(`https://api.openweathermap.org/data/2.5/weather?lat=${this.lat}&lon=${this.lon}&appid=059dcee9c15c93a942eb1f38b72876be`, (weatherData) => {
          this.cityName = weatherData.name;
          this.weatherIcon = `http://openweathermap.org/img/w/${weatherData.weather[0].icon}.png`;
          this.weatherDescription = weatherData.weather[0].description;

          this.celsius = Math.floor(weatherData.main.temp - 273.15);
          this.farenheit = Math.floor(weatherData.main.temp * 9 / 5 - 459.67);

          // Atualizar outros elementos conforme necessário
          this.updateOtherElements(weatherData);
        });
      });
    },
    updateOtherElements(weatherData) {
    const cacheKey = `${this.lat},${this.lon}`;
    if (this.apiCache[cacheKey] && Date.now() - this.apiCache[cacheKey].timestamp < 900 * 1000) {
      // Usar os dados em cache
      weatherData = this.apiCache[cacheKey].data;
    } else {
      // Fazer solicitação à API e atualizar cache
      $.getJSON(`https://api.openweathermap.org/data/2.5/weather?lat=${this.lat}&lon=${this.lon}&appid=059dcee9c15c93a942eb1f38b72876be`, (newWeatherData) => {
        weatherData = newWeatherData;
        // Atualizar cache com os novos dados
        this.apiCache[cacheKey] = {
          data: newWeatherData,
          timestamp: Date.now()
        };

        // Atualizar elementos conforme necessário
        this.updateTemperatureDisplay(weatherData);
        this.updateOtherInfo(weatherData);
      });
    }

    // Agora, use os dados (sejam eles do cache ou da API)
    this.updateTemperatureDisplay(weatherData);
    this.updateOtherInfo(weatherData);
  },
  updateTemperatureDisplay(weatherData) {
    const tempMax = weatherData.main.temp_max;
    const tempMin = weatherData.main.temp_min;

    if (this.displayCelsius) {
      $("#max").text(Math.floor(tempMax - 273.15) + 'ºC');
      $("#min").text(Math.floor(tempMin - 273.15) + 'ºC');
    } else {
      $("#max").text(Math.floor(tempMax * 9 / 5 - 459.67) + '°F');
      $("#min").text(Math.floor(tempMin * 9 / 5 - 459.67) + '°F');
    }
  },
  updateOtherInfo(weatherData) {
    // Atualize outras informações, como longitude, latitude, umidade, pressão, velocidade do vento, etc.
    $("#longitude").text('Lon: ' + weatherData.coord.lon);
    $("#latitude").text('Lat: ' + weatherData.coord.lat);
    $("#humidity").text('Humidade: ' + weatherData.main.humidity + '%');
    $("#pressure").text('PAtm: ' + weatherData.main.pressure + ' hPa');
    $("#windSpeed").text('Vento: ' + weatherData.wind.speed + ' km/h');
    }
  }
};
</script>

<style scoped>

@import url('https://fonts.googleapis.com/css?family=Space+Mono');  
@import url('https://fonts.googleapis.com/css?family=Open+Sans:200,400,600,700,800');

  body{
    /* CSS3 fallback */ 
    height: 100%;
    background: #c95e67;
  }

  h1{
    font-family: 'Open Sans', Helvetica, sans-serif;
    font-size: 29px;
    color: #c95e67;
  }

  .text{
    font-family: 'Space Mono', Helvetica, sans-serif;
    font-size: 14px;
    color: #333;
  }

  .container{
    display: grid;
    grid-template-columns: 1fr 1fr 1fr;
    width: 100%;
    margin: 40px 0px 10px 0;
  }

  .container-button{
    display: grid;
    grid-template-columns: 1fr;
    margin-top: 10px;
  }

  .weather-icon,
  .title{
    grid-column: 1/4;
    margin: 0 auto;
    text-align: center;
    background: #efefef;
    border-radius: 4px;
    padding: 15px 100px;
  }

  .important-info{
    grid-column: 1;
  }

  .description-container{
    background: #efefef;
    border-radius: 4px;
    display: grid;
    grid-template-columns: 1fr 1fr 1fr;
    width: 450px;
    margin: 0 auto;
    padding: 20px 0px;
  }

  .block-1{
    grid-column: 1;
    padding: 10px;
  }

  .block-2{
    grid-column: 2;
    padding: 10px;
  }

  .block-3{
    grid-column: 3;
    padding: 10px;
  }

  .weatherInfo{
    height: 100%;
  }

  .temp-info{
    font-family: 'Open Sans', Helvetica, sans-serif;
    font-size: 65px;
    color: #c95e67;
    font-weight: 600;
  }

  .switch-button{
    grid-column: 1 1 1;
    border-radius: 4px;
    margin: 0 auto;
    width: 400px;
    height: 40px;
    text-align: center;
    position: relative;
    will-change: transform;
    z-index: 197 !important;
    cursor: pointer;
    -webkit-transition: .3s ease all;
    border: 1px solid #efefef;
    display: inline-block;
  }

  .switch-button-case{
    display: inline-block;
    background: none;
    width: 49%;
    height: 100%;
    color: #efefef;
    position: relative;
    border: none;
    -webkit-transition: .3s ease all;
    transition: .3s ease all;
    text-transform: uppercase;
    letter-spacing: 5px;
    padding-bottom: 1px;
  }

  .switch-button-case:focus{
    outline: none;
  }

  .active{
    position: absolute;
    color: #151515;
    background-color: #efefef;
    left: 0;
    top: 0;
    width: 50%;
    height: 100%;
    z-index: -1;
    -webkit-transition: .3s ease-out all;
    transition: .3s ease-out all;
  }

  .switch-button .active-case{
    color: #151515;
  }

  .signature{
    position: fixed;
    font-family: sans-serif;
    font-weight: 100;
    bottom: 10px;
    left: 0;
    letter-spacing: 4px;
    font-size: 10px;
    width: 100vw;
    text-align: center;
    color: #efefef;
    text-transform: uppercase;
    text-decoration: none;
  }

</style>