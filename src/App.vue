<template>
  <div id="app">
    <div class="text-center my-3">
      <b-btn v-b-tooltip.hover title="Узнай тех, кто находится рядом. Присоединяйся!">
        <h1>Карта</h1>
      </b-btn>
    </div>
    <div>
      <b-form inline>
        <label class="sr-only" for="inlineFormInputName2" required>Имя</label>
        <b-input
          class="mb-2 mr-sm-2 mb-sm-0"
          id="inlineFormInputName2"
          placeholder="Имя"
          v-model="userName"
          required
        />
        <b-form-select v-model="selected" :options="options" class="mb-3"/>
        <b-button variant="primary" @click="addPoint">Добавить меня на карту</b-button>
      </b-form>
    </div>
    <div id="map" style="width: 100%; height: 400px;"></div>
  </div>
</template>

<script>
import axios from "axios";
export default {
  name: "app",
  data() {
    return {
      selected: null,
      info: null,
      options: [
        { value: null, text: "Тим", disabled: true },
        { value: "Дон Кихот", text: "Дон Кихот" },
        { value: "Дюма", text: "Дюма" },
        { value: "Робеспьер", text: "Робеспьер" },
        { value: "Гюго", text: "Гюго" },
        { value: "Жуков", text: "Жуков" },
        { value: "Есенин", text: "Есенин" },
        { value: "Максим Горький", text: "Максим Горький" },
        { value: "Гамлет", text: "Гамлет" },
        { value: "Бальзак", text: "Бальзак" },
        { value: "Наполеон", text: "Наполеон" },
        { value: "Джек Лондон", text: "Джек Лондон" },
        { value: "Драйзер", text: "Драйзер" },
        { value: "Габен", text: "Габен" },
        { value: "Гексли", text: "Гексли" },
        { value: "Штирлиц", text: "Штирлиц" },
        { value: "Достоевский", text: "Достоевский" }
      ],
      myGeoObject: "",
      myMap: "",
      myPlacemark: "",
      userName: ""
    };
  },
  methods: {
    addPoint() {
      if (this.myGeoObject) {
        this.myGeoObject.get(0).properties.set({
          hintContent: this.userName,
          balloonContentBody: this.selected
        });
        this.myMap.geoObjects.add(this.myGeoObject);
      } else {
        var location = ymaps.geolocation.get();
        location.then(
          function(result) {
            var userCoodinates = result.geoObjects
              .get(0)
              .geometry.getCoordinates();
            result.geoObjects.get(0).properties.set({
              hintContent: this.userName,
              balloonContentBody: this.selected
            });
            this.myMap.geoObjects.add(result.geoObjects);
          },
          function(err) {
            console.log("Ошибка: ", err);
          }
        );
      }
    },
    getPoint(name, tim, location) {
      return new ymaps.Placemark(location, {
        balloonContentHeader: "Привет, я ",
        balloonContentBody: name,
        balloonContentFooter: ":)",
        hintContent: tim
      });
    },
    addPoints(jsonTims) {
      var timsPlacemarks = new ymaps.GeoObjectCollection(null, {
        preset: "islands#blueCircleIcon"
      });
      jsonTims.forEach(person => {
        timsPlacemarks.add(
          this.getPoint(person.name, person.tim, person.location)
        );
      });
      this.myMap.geoObjects.add(timsPlacemarks);
    },
    initMap() {
      ymaps.ready(init);
      let self = this;
      function init() {
        var geolocation = ymaps.geolocation;
        self.myMap = new ymaps.Map(
          "map",
          {
            center: [55, 34],
            zoom: 10
          },
          {
            searchControlProvider: "yandex#search"
          }
        );
        geolocation
          .get({
            provider: "yandex",
            mapStateAutoApply: true
          })
          .then(function(result) {
            result.geoObjects.options.set("preset", "islands#redCircleIcon");
            result.geoObjects.get(0).properties.set({
              balloonContentBody: "Мое местоположение"
            });
            self.myGeoObject = result.geoObjects;
            this.myMap.center = result.geoObjects
              .get(0)
              .geometry.getCoordinates();
          });
        geolocation
          .get({
            provider: "browser",
            mapStateAutoApply: true
          })
          .then(function(result) {
            result.geoObjects.options.set("preset", "islands#blueCircleIcon");
            self.myGeoObject = result.geoObjects;
          });

        axios
          .get("users.json")
          .then(response => {
            self.info = response.data;
            //console.log("inGet:", response);
            self.addPoints(self.info);
          })
          .catch(error => {
            console.log(error);
          });
      }
    }
  },
  mounted() {
    this.initMap();
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  padding: 1rem;
}

h1,
h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}

.mb-2,
.my-2,
.btn {
  margin-bottom: 1rem !important;
}

.btn {
  margin-left: 1rem;
}
</style>
