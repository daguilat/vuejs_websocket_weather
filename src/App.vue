<template>
  <div id="app">
    <table id="weather">
      <tr>
        <th>City</th>
        <th>Temperature</th>
        <th>Time</th>
      </tr>
      <tr v-for="city in currentWeather" v-bind:key="city.name">
        <td>{{city.name}}</td>
        <td>{{city.temp}}°</td>
        <td>{{city.time}}</td>
      </tr>
    </table>
  </div>
</template>

<script>
export default {
  name: 'App',
  data: function() {
    return {
      connection: null,
      connected: false,
      currentWeather: [],
      cityNames : ['Santiago', 'Zúrich', 'Auckland', 'London', 'Georgia']
    }
  },
  created: function() {  

    console.log("Starting connection to WebSocket Server")
    this.connection = new WebSocket("ws://localhost:8000/weather")
    console.log(this.connection)

    var self = this;
    this.connection.onopen = function() {
      self.connected = true;
      console.log("Successfully connected to the echo websocket server...")
    }

    this.connection.onmessage = function(event) { 
      var data = JSON.parse(event.data);
      self.currentWeather = []
      data.forEach(dataTemp => {
        var city = JSON.parse(dataTemp)
        var weatherData = {name: city.name, temp: city.main.temp, time: self.parse_timezone(city.timezone)};
        self.currentWeather.push(weatherData)
      });
    }

  },

  methods: {
    get_city_weather: function(data) {
      this.connection.send(data);
    },

    parse_timezone: function(timezone){
      var d = new Date()
      var localTime = d.getTime()
      var localOffset = d.getTimezoneOffset() * 60000
      var utc = localTime + localOffset
      var cityTime = utc + (1000 * timezone)
      return new Date(cityTime)
    }
  },
  mounted: function () {
    this.interval = setInterval(() => {
      if(this.connected){
        this.get_city_weather(this.cityNames);
      }
    },10000);
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

#weather {
  font-family: Arial, Helvetica, sans-serif;
  border-collapse: collapse;
  width: 100%;
}

#weather td, #weather th {
  border: 1px solid #ddd;
  padding: 8px;
}

#weather tr:nth-child(even){background-color: #f2f2f2;}

#weather tr:hover {background-color: #ddd;}

#weather th {
  padding-top: 12px;
  padding-bottom: 12px;
  text-align: left;
  background-color: #4CAF50;
  color: white;
}
</style>