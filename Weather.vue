<template>
  <div id="wrapper">
    <main>
      <router-link class="home-btn" to="LandingPage">⚏</router-link>
    </main>
  </div>
</template>

<script>
  import config from './morsecodemirror.config.json'
  import axios  from 'axios'
  export default {
    name: 'Weather',
    data: function() {
      return {
        openWeatherMapAPI: config.openWeatherMapAPIKey,
        darkSkyAPI: config.darkSkyAPIKey,
        lang: 'en',
        unitsOfMeasurement: 'imperial',
        currentForecast: {},
        dailyForecast: {},
        weatherAlerts: {},
        weatherRefreshInterval: 600000,
        lat: config.home.latitude,
        long: config.home.longitude,
        zip: config.home.address.zip
      }
    },
    created() {
      this.getForecast()
    },
    methods: {
      getForecast() {
        axios.get('https://api.darksky.net/forecast/'
          + this.darkSkyAPI + '/'
          + this.lat + ',' + this.long
        )
        .then(response => {
          window.console.log(response)
          let currentResponse = response.data.currently
          let dailyResponse = response.data.daily.data
          let alertResponse = response.data.alerts
          this.currentForecast = {
            'summary': currentResponse.summary,
            'icon': currentResponse.icon,
            'currentTemp': Math.round(currentResponse.temperature),
            'feelsLike': Math.round(currentResponse.apparentTemperature),
            'uvIndex': currentResponse.uvIndex,
            'windSpeed': Math.round(currentResponse.windSpeed),
            'windBearing': currentResponse.windBearing
          }
          dailyResponse.map((day) => {
            this.dailyForecast[day.time] = {
              'time': new Date(day.time*1000),
              'summary': day.summary,
              'uvIndex': day.uvIndex,
              'windSpeed': Math.round(day.windSpeed),
              'windBearing': day.windBearing,
              'sunriseTime': new Date(day.sunriseTime*1000),
              'sunsetTime': new Date(day.sunsetTime*1000),
              'temperatureHigh': Math.round(day.temperatureHigh),
              'temperatureLow': Math.round(day.temperatureLow),
              'feelsLikeHigh': Math.round(day.apparentTemperatureHigh),
              'feelsLikeLow': Math.round(day.apparentTemperatureLow),
              'precipProbability': (day.precipType) ? (Math.round((day.precipProbability / 1) * 100)) + '% chance of ' + day.precipType : ''
            }
          })
          if (alertResponse) {
            this.weatherAlerts = {
              'description':  (alertResponse.description) ? alertResponse.description : '',
              'expires': (alertResponse.expires) ? alertResponse.expires : '',
              'regions': (alertResponse.regions) ? alertResponse.regions : '',
              'severity': (alertResponse.severity) ? alertResponse.severity : '', //"advisory", "watch", "warning"
              'time': (alertResponse.time) ? alertResponse.time : '',
              'title': (alertResponse.title) ? alertResponse.title : ''
            }
          }

          window.console.log('currentForecast',this.currentForecast)
          window.console.log('dailyForecast',this.dailyForecast)
          window.console.log('weatherAlerts',this.weatherAlerts)
        })
        .catch(error => {
          window.console.log(error)
        })
      },
    },
    beforeMount () {
      if (config.lang && config.lang.length > 0) {
        this.lang = config.lang
      }
      if (config.unitsOfMeasurement && config.unitsOfMeasurement.length > 0) {
        this.unitsOfMeasurement = config.unitsOfMeasurement
      }
      if (config.weatherRefreshInterval && config.weatherRefreshInterval.length > 0) {
        this.weatherRefreshInterval = config.weatherRefreshInterval
      }
    },
  }
</script>

<style scoped>

  * {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }

  .home-btn {
    position: absolute;
    font-size: 20px;
    bottom: 20px;
    left: 20px;
  }

</style>

