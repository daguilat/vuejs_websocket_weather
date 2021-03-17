<template>
  <div id="app">
    <h2>Vue.js WebSocket Tutorial</h2> 
    <button v-on:click="sendMessage('hello')">Send Message</button>

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
      citiesData: [],
      currentWeather: [{
        name: 'Santiago',
        temp: 30.6,
        time: 'hola'
      }]
    }
  },
  created: function() {  

    console.log("Starting connection to WebSocket Server")
    this.connection = new WebSocket("ws://localhost:8000/weather")
    console.log(this.connection)

    var self = this;
    this.connection.onmessage = function(event) { 
      var data = JSON.parse(event.data);
      if(data.opt == "1"){
        var cityData = {name: data.name, lon: data.lon, lat: data.lat};
        self.citiesData.push(cityData);
      } else {
        console.log(event)

      }
    }
  },

  methods: {
    get_city_coords: function(data) {
      this.connection.onopen = () => this.connection.send(data);
    },

    get_city_weather: function(data) {
      this.connection.onopen = () => this.connection.send(data);
    }
  },
  mounted: function () {
    // Loading every city coords
    var cityNames = ['Santiago', 'Zúrich', 'Auckland', 'London', 'Georgia'];

    cityNames.forEach( city =>  {
      var data = '{"route": "get_city_coords", "name": "' + city + '"}';
      this.get_city_coords(data);
    });


    window.setInterval(() => {
      this.citiesData.forEach( city => {
        var data = '{"route": "get_city_weather", "lon": "' + city.lon + '", "lat": "' + city.lat + '"}';
        this.get_city_weather(data);
      });
    }, 1000)
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