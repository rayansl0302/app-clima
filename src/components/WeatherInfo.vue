<template>
    <div class="weather-wrap">
        <!-- Informações da localização -->
        <div class="location-box">
            <div class="location">{{ location.name }}, {{ location.region }}</div>
            <div class="date">{{ dataBuilder() }}</div>
        </div>

        <!-- Condições climáticas atuais -->
        <div v-if="current && current.temp_c && current.condition && current.condition.text" class="weather-box">
            <div class="temp">{{ Math.round(current.temp_c) }}°c</div>
            <div class="weather">{{ current.condition.text }}</div>

            <!-- Adicionado: Direção e velocidade do vento -->
            <div class="wind-info">
                <p class="text-white">Direção do Vento: {{ current.wind_dir }}</p>
                <p class="text-white">Velocidade do Vento: {{ current.wind_kph }} km/h</p>
            </div>

        </div>

        <!-- Previsão do tempo para o dia por hora -->
        <div class="mt-[80px]">
            <h2 class="text-center text-[32px]">
                Previsão do Tempo para o Dia por Hora:
            </h2>

            <!-- Verificar se há previsão por hora disponível -->
            <div class="weather-daily" v-if="hasHourlyForecast">
                <!-- Iterar sobre os dados da previsão por hora -->
                <div v-for="hourData in hourlyForecast" :key="hourData.time_epoch" class="hourly-weather">
                    <!-- Exibir a hora formatada -->
                    <p>{{ formatTime(hourData.time) }}</p>

                    <!-- Exibir o ícone do clima -->
                    <img :src="getWeatherIconURL(hourData.condition.icon)" alt="Weather Icon" />

                    <!-- Exibir a temperatura -->
                    <p>{{ hourData.temp_c }}°C</p>

                    <!-- Adicionado: Direção e velocidade do vento na previsão por hora -->
                    <div class="wind-info">
                        <p>Direção do Vento: {{ hourData.wind_dir }}</p>
                        <p>Velocidade do Vento: {{ hourData.wind_kph }} km/h</p>
                    </div>
                </div>
            </div>

            <!-- Mensagem quando não há previsão disponível -->
            <div class="text-white text-center my-4 mx-4" v-else>
                <p>A previsão do tempo não está disponível.</p>
            </div>
        </div>

        <!-- Previsão do tempo para os 3 dias seguintes, por hora -->
        <div v-if="hasNext3DaysForecast" class="weather-three-days mt-[80px]">
            <h2 class="text-center text-[32px]">Previsão do Tempo para os Próximos Dias</h2>
            <!-- Iterar sobre os dados da previsão para os próximos 3 dias -->
            <div v-for="dayData in next3DaysForecast" :key="dayData.date" class="daily-forecast">
                <h3 class="text-white ">Dia: <br>{{ formatDate(dayData.date) }}</h3>
                <!-- Iterar sobre as horas do dia específico -->
                <div v-for="hourData in dayData.hour" :key="hourData.time_epoch" class="hourly-weather">
                    <p>{{ formatTime(hourData.time) }}</p>
                    <img :src="getWeatherIconURL(hourData.condition.icon)" alt="Weather Icon" />
                    <p>{{ hourData.temp_c }}°C</p>
                    <!-- Adicionado: Direção e velocidade do vento na previsão por hora -->
                    <div class="wind-info">
                        <p>Direção do Vento: {{ hourData.wind_dir }}</p>
                        <p>Velocidade do Vento: {{ hourData.wind_kph }} km/h</p>
                    </div>
                </div>
            </div>
        </div>
    </div>
    
</template>

<script>
export default {
    props: {
        location: Object,
        current: Object,
        hourlyForecast: Array,
        dataBuilder: Function,
        forecast: Object,
        next3DaysForecast: Array, // Adicionado: Propriedade para a previsão dos próximos 3 dias
    },
    computed: {
        // Verificar se há previsão por hora disponível
        hasHourlyForecast() {
            return this.hourlyForecast && this.hourlyForecast.length > 0;
        },
        // Verificar se há previsão para os próximos 3 dias disponível
        hasNext3DaysForecast() {
            return this.next3DaysForecast && this.next3DaysForecast.length > 0;
        },
    },
    methods: {
        // Formatar o tempo para exibição
        formatTime(time) {
            if (!time) {
                return ''; // ou outra string padrão, caso o valor seja indefinido
            }

            const date = new Date(time);

            // Verificar se a data é válida
            if (isNaN(date.getTime())) {
                return 'Formato de data inválido'; // ou outra string indicando o erro
            }

            return date.toLocaleTimeString('pt-BR', { hour: 'numeric', minute: 'numeric' });
        },
        // Obter a URL do ícone do clima
        getWeatherIconURL(icon) {
            return `http:${icon}`;
        },
        // Formatar a data para exibição
        formatDate(date) {
            // Adapte para o formato de data desejado
            return new Date(date).toLocaleDateString('pt-BR');
        },
        // Registrar os dados da previsão no console
        logForecastData() {
            console.log('Forecast Data:', this.forecast);
            if (this.hasHourlyForecast) {
                console.log('Hourly Forecast:', this.hourlyForecast);
            } else {
                console.log('No hourly forecast data available.');
            }

            if (this.hasNext3DaysForecast) {
                console.log('Next 3 Days Forecast:', this.next3DaysForecast);
            } else {
                console.log('No next 3 days forecast data available.');
            }
        },
    },
    mounted() {
        // Chamar a função para registrar os dados da previsão no console ao montar o componente
        this.logForecastData();
    },
};
</script>

  

  
  
<style lang="scss" scoped>
.weather-wrap {
    margin-top: 40px;

    .weather-box {
        text-align: center;

        .temp {
            display: inline-block;
            padding: 10px 25px;
            color: #fff;
            font-size: 102px;
            font-weight: 900;
            text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
            background-color: rgba(255, 255, 255, 0.25);
            border-radius: 16px;
            margin: 30px 0px;
            box-shadow: 3px 6px rgba(0, 0, 0, 0.25);
        }

        .weather {
            color: #fff;
            font-size: 28px;
            font-weight: 700;
            font-style: italic;
            text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
        }
    }

    .weather-daily {
        display: flex;
        overflow-x: scroll;
        margin: 0 200px;
        align-items: center;

    }

    .hourly-weather {
        display: flex;
        flex-direction: column;
        align-items: center;
        margin: 10px;
        padding: 10px;
        border: 1px solid #ccc;
        color: #fff;

    }

    .weather-three-days {
        .daily-forecast {
            margin: 0 200px;
            align-items: center;
            display: flex;
            overflow-x: scroll;
        }
    }

    .hourly-weather img {
        width: 50px;
        height: 50px;
    }
}

@media (max-width: 768px) {
    .weather-wrap {
        position: relative;

        .weather-daily {
            display: flex;
            overflow-x: scroll;
            margin: 0 0;
        }

        .weather-three-days {
            .daily-forecast {
                display: flex;
                overflow-x: scroll;
                margin: 0 0;

            }
        }
    }
}
</style>
  