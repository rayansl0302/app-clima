<template>
  <div id="app" :class="isWarmClass">
    <main>
      <div class="container">
        <div>
          <!-- Título centralizado -->
          <h2 class="text-center">App de Clima</h2>
          <!-- Botão para salvar cidade -->
          <div v-if="!route.query.preview" class="flex flex-col ml-4 py-8 text-white items-center">
            <button @click="saveCity" class="bg-green-500 text-white px-4 py-2 rounded-full justify-center">
              Salvar cidade
            </button>
          </div>

          <!-- Componente de busca -->
          <search-box v-model:query="searchQuery" :autocomplete-results="autocompleteResults" :saved-cities="savedCities"
            @fetch-autocomplete="fetchAutocomplete" @select-autocomplete-result="selectAutocompleteResult"
            @remove-city="removeCity" />

          <!-- Informações meteorológicas -->
          <WeatherInfo v-if="weather.location.name && weather.current.temp_c" :location="weather.location"
            :current="weather.current" :hourly-forecast="hourlyForecast" :next3DaysForecast="next3DaysForecastData"
            :forecastData="forecastData" :data-builder="dataBuilder" />
        </div>
      </div>
    </main>
  </div>
</template>

<script>
import { useRoute } from 'vue-router';
import { ref, reactive, onMounted } from 'vue';
import axios from 'axios';
import SearchBox from './components/SearchBox.vue';
import WeatherInfo from './components/WeatherInfo.vue';

export default {
  name: 'App',
  components: {
    SearchBox,
    WeatherInfo,
  },
  setup() {
    const route = useRoute();
    const api_key = 'ebab97879b5e410fae9143336233011';
    const url_base = 'http://api.weatherapi.com/v1';

    // Variáveis reativas
    const searchQuery = ref('');
    const weather = reactive({
      location: {},
      current: {},
    });
    const isWarmClass = ref('');
    const autocompleteResults = ref([]);
    const savedCities = ref([]);
    const forecastData = ref({});
    const next3DaysForecastData = ref([]);
    const hourlyForecast = ref([]);

    
    // Inicialize dadosDoTempo com a propriedade pm10
  
    // Limpa os resultados de autocomplete
    const handleInput = () => {
      autocompleteResults.value = [];
    };

    // Busca autocompletar ao digitar na barra de pesquisa
    const fetchAutocomplete = async () => {
      try {
        if (searchQuery.value.trim() === '') {
          autocompleteResults.value = [];
          return;
        }

        const response = await axios.get(`${url_base}/search.json?key=${api_key}&lang=pt&q=${searchQuery.value}`);
        autocompleteResults.value = response.data || [];
      } catch (error) {
        console.error('Error fetching autocomplete:', error);
      }
    };

    // Busca dados da previsão do tempo
    const fetchForecastData = async () => {
      try {
        let locationName;

        if (weather.location && weather.location.name) {
          locationName = weather.location.name;
        } else if (searchQuery.value.trim() !== '') {
          locationName = searchQuery.value;
        } else {
          console.warn('Nome da localização ausente para a previsão do tempo.');
          return;
        }

        const response = await axios.get(`${url_base}/forecast.json?key=${api_key}&lang=pt&q=${locationName}&days=3`);
        console.log('Forecast API Response:', response.data);

        forecastData.value = response.data || {};

        // Certifique-se de que forecastday é uma matriz antes de acessar o primeiro item
        hourlyForecast.value = Array.isArray(response.data.forecast.forecastday)
          ? response.data.forecast.forecastday[0].hour || []
          : [];
        next3DaysForecastData.value = Array.isArray(response.data.forecast.forecastday)
          ? response.data.forecast.forecastday.slice(1, 4) || [] // Pegue os próximos 3 dias
          : [];
      } catch (error) {
        console.error('Error fetching forecast data:', error);
      }
    };

    // Busca dados do clima atual
    const fetchWeather = async () => {
      try {
        if (searchQuery.value.trim() === '') {
          return;
        }

        const response = await axios.get(`${url_base}/current.json?key=${api_key}&lang=pt&q=${searchQuery.value}`);
        weather.location = response.data.location || {};
        weather.current = response.data.current || {};
        console.log('Temperature:', weather.current.temp_c);
        updateWarmClass();
        fetchForecastData(); // Adicionado aqui
      } catch (error) {
        console.error('Error fetching weather:', error);
      }
    };

    // Função para obter a localização do usuário
    const getUserLocation = async () => {
      try {
        const position = await new Promise((resolve, reject) => {
          navigator.geolocation.getCurrentPosition(resolve, reject);
        });

        const { latitude, longitude } = position.coords;

        const response = await axios.get(`${url_base}/current.json?key=${api_key}&lang=pt&q=${latitude},${longitude}`);
        weather.location = response.data.location || {};
        weather.current = response.data.current || {};
        searchQuery.value = `${weather.location.name}, ${weather.location.region}`;
        updateWarmClass();
        fetchForecastData(); // Adicionado aqui
      } catch (error) {
        console.error('Error fetching user location:', error);
      }
    };

    // Atualiza a classe isWarmClass com base na temperatura
    const updateWarmClass = () => {
      isWarmClass.value = typeof weather.current !== 'undefined' && weather.current.temp_c > 25 ? 'warm' : '';
    };

    // Seleciona um resultado de autocompletar e busca o clima
    const selectAutocompleteResult = (result) => {
      searchQuery.value = `${result.name}, ${result.region}`;
      fetchWeather();
      autocompleteResults.value = [];
    };

    // Salva a cidade atual
    const saveCity = () => {
      if (searchQuery.value.trim() === '') {
        console.log('Consulta vazia, não salvo nada');
        return;
      }
      const savedCity = {
        name: weather.location.name,
        region: weather.location.region,
        temperature: weather.current.temp_c,
      };
      if (!savedCities.value.some(city => city.name === savedCity.name)) {
        savedCities.value.push(savedCity);
        localStorage.setItem('savedCities', JSON.stringify(savedCities.value));
        loadSavedCities();
      }
    };

    // Remove uma cidade da lista salva
    const removeCity = (index) => {
      savedCities.value.splice(index, 1);
      localStorage.setItem('savedCities', JSON.stringify(savedCities.value));
    };

    // Função auxiliar para formatar a data
    const dataBuilder = () => {
      const d = new Date();
      const months = [
        'Janeiro', 'Fevereiro', 'Março', 'Abril', 'Maio', 'Junho', 'Julho', 'Agosto', 'Setembro', 'Outubro', 'Novembro', 'Dezembro',
      ];
      const days = ['Domingo', 'Segunda-feira', 'Terça-feira', 'Quarta-feira', 'Quinta-feira', 'Sexta-feira', 'Sábado'];

      const day = days[d.getDay()];
      const date = d.getDate();
      const month = months[d.getMonth()];
      const year = d.getFullYear();

      return `${day} ${date} ${month} ${year}`;
    };

    // Função executada após a montagem do componente
    onMounted(() => {
      loadSavedCities();
      getUserLocation();
      fetchWeather();
    });

    // Carrega as cidades salvas do armazenamento local
    const loadSavedCities = () => {
      const savedCitiesJSON = localStorage.getItem('savedCities');
      if (savedCitiesJSON) {
        savedCities.value = JSON.parse(savedCitiesJSON);
      }
    };

    // Retorna todas as variáveis e funções necessárias para o componente
    return {
      route,
      searchQuery,
      weather,
      isWarmClass,
      autocompleteResults,
      savedCities,
      forecastData,
      hourlyForecast,
      next3DaysForecastData,
      handleInput,
      fetchAutocomplete,
      fetchWeather,
      updateWarmClass,
      selectAutocompleteResult,
      saveCity,
      removeCity,
      dataBuilder,

    };
  },
};
</script>


<style lang="scss">
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'montserrat', sans-serif;
}

#app {
  background-image: url('./assets/cold-bg.jpg');
  background-repeat: no-repeat;
  background-size: cover;
  background-position: bottom;
  transform: 0.4s;
}

#app.warm {
  background-image: url('./assets/warm-bg.jpg');
}

main {
  min-height: 100vh;
  padding: 25px;
  background-image: linear-gradient(to bottom, rgba(0, 0, 0, 0.25), rgba(0, 0, 0, 0.75));
  display: flex;
  justify-content: center;

  h2 {
    color: #fff;
    font-size: 45px;
  }
}


.location-box {
  .location {
    color: #fff;
    font-size: 32px;
    font-weight: 500;
    text-align: center;
    text-shadow: 1px 3px rgba(0, 0, 0, 0.25);
  }

  .date {
    color: #fff;
    font-size: 20px;
    font-weight: 300;
    font-style: italic;
    text-align: center;
  }
}
</style>
