<template>
  <v-app>
    <div class="container">
      <v-main>
        <h1 v-text="'Weather Forecast'" />
        <v-select
          class="cities-select"
          label="Cities"
          :items="cities"
          :item-text="'name'"
          :return-object="true"
          :persistent-hint="true"
          hint="Please select city to see the forecast"
          v-model="selectedCity"
        />
        <template v-if="weatherInfoForSelectedCity">
          <div class="label" v-text="weatherMain" />
          <div class="data" v-text="weatherDescription" />
          <br />
          <div class="label" v-text="weatherTemp" />
          <div class="data" v-text="weatherWind" />
          <br />
          <v-btn
            v-text="'See Forecast'"
            @click="getForecastArrayForSelectedCity"
            v-if="!showForecast"
          />
          <v-btn v-text="'Close'" @click="showForecast = false" v-else />
          <div class="forecast-container" v-if="showForecast">
            <div v-if="!forecastArrayForSelectedCity.length" v-text="'Loading...'" />
            <v-data-table
              class="elevation-1"
              :headers="headers"
              :items="forecastArrayForSelectedCityAndSelectedDay"
              :hide-default-footer="true"
              v-if="forecastArrayForSelectedCity.length"
            />
            <div class="btn-row">
              <v-btn
                class="day-btn"
                v-for="day in next6Days"
                :key="day"
                v-text="day"
                @click="selectDay(day)"
                :class="{'active': currSelectedDay === day}"
              />
            </div>
          </div>
        </template>
      </v-main>
    </div>
  </v-app>
</template>

<script lang="ts">
import Vue from 'vue';
import { format } from 'date-fns';

interface AppData {
  selectedCity: City | null;
  weatherInfoForSelectedCity: any;
  forecastArrayForSelectedCity: any[];
  showForecast: boolean;
  next6Days: string[];
  currSelectedDay: string;
  headers: any[];
  cities: City[];
}

interface City {
  id: number;
  name: string;
}

interface ForecastedDay {
  day: string;
  date: string;
  temp: string;
  minTemp: string;
  maxTemp: string;
  wind: string;
  description: string;
}

const dayFormat = 'd MMM';

export default Vue.extend({
  name: 'App',
  data(): AppData {
    const next6Days = [];

    for (let day = 0; day < 6; day++) {
      const currDate = new Date();
      currDate.setDate(currDate.getDate() + day);
      next6Days.push(format(currDate, dayFormat));
    }

    const appData: AppData = {
      selectedCity: null,
      weatherInfoForSelectedCity: null,
      forecastArrayForSelectedCity: [],
      showForecast: false,
      next6Days,
      currSelectedDay: '',
      headers: [
        { text: 'Date', value: 'date' },
        { text: 'Temp', value: 'temp' },
        { text: 'Min Temp', value: 'minTemp' },
        { text: 'Max Temp', value: 'maxTemp' },
        { text: 'Wind', value: 'wind' },
        { text: 'Description', value: 'description' },
      ],
      cities: [
        {
          id: 6167865,
          name: 'Toronto, CA',
        },
        {
          id: 6094817,
          name: 'Ottawa, CA',
        },
        {
          id: 1850147,
          name: 'Tokyo, JP',
        },
      ],
    };
    return appData;
  },
  computed: {
    weatherMain(): string {
      return this.$data.weatherInfoForSelectedCity.weather?.[0]?.main || '';
    },
    weatherDescription(): string {
      return this.$data.weatherInfoForSelectedCity.weather?.[0]?.description || '';
    },
    weatherTemp(): string {
      return `${this.$data.weatherInfoForSelectedCity.main.temp} °C`;
    },
    weatherWind(): string {
      return `Wind ${this.$data.weatherInfoForSelectedCity.wind.speed} m/sec`;
    },
    forecastArrayForSelectedCityAndSelectedDay(): any[] {
      return this.$data.forecastArrayForSelectedCity.filter((forecastedDay: ForecastedDay) => {
        return forecastedDay.day === this.$data.currSelectedDay;
      });
    },
  },
  watch: {
    selectedCity() {
      this.$data.weatherInfoForSelectedCity = null;
      this.$data.forecastArrayForSelectedCity = [];
      this.$data.showForecast = false;
      fetch(`https://api.openweathermap.org/data/2.5/weather?id=${this.$data.selectedCity.id}&appid=538882fc8387290c6cee83f313a6acf5&units=metric`)
        .then((response) => response.json())
        .then((data) => {
          this.$data.weatherInfoForSelectedCity = data;
        })
        .catch((error: any) => {
          alert('Error making weather call. ' + error?.message);
        });
    },
  },
  methods: {
    selectDay(day: string) {
      this.$data.currSelectedDay = day;
    },
    getForecastArrayForSelectedCity() {
      this.$data.showForecast = true;
      if (this.$data.forecastArrayForSelectedCity.length) {
        // Already have data for this city. So return to save a call.
        return;
      }
      fetch(`https://api.openweathermap.org/data/2.5/forecast?id=${this.$data.selectedCity.id}&appid=538882fc8387290c6cee83f313a6acf5&units=metric`)
        .then((response) => response.json())
        .then((data) => {
          console.log(data);
          this.$data.forecastArrayForSelectedCity = data?.list.map((forecastedDay: any, index: number) => {
            const day = format(new Date(forecastedDay.dt * 1000), dayFormat);
            if (index === 0) {
              this.$data.currSelectedDay = day;
            }
            return {
              day,
              date: format(new Date(forecastedDay.dt * 1000), 'd MMM ha'),
              temp: `${forecastedDay.main.temp} °C`,
              minTemp: `${forecastedDay.main.temp_min} °C`,
              maxTemp: `${forecastedDay.main.temp_max} °C`,
              wind: `${forecastedDay.wind.speed || '-'} m/sec`,
              description: forecastedDay.weather?.[0]?.description || 'N/A',
            } as ForecastedDay;
          });
        })
        .catch((error: any) => {
          alert('Error making forecast call. ' + error?.message);
        });
    },
  },
});
</script>

<style lang="scss">
.container {
  padding: 20px;
}
.forecast-container {
  margin-top: 20px;
}
.cities-select {
  max-width: 300px !important;
  margin-bottom: 20px;
}
.btn-row {
  margin: 10px -10px;
  .day-btn {
    margin: 10px;
    color: rgba(0, 0, 0, 0.47) !important;
    &.active {
      color: rgba(0, 0, 0, 0.87) !important;
    }
  }
}
.data {
  font-size: 14px;
}
</style>
