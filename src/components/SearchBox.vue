<template>
  <div class="search-box justify-center">
    <div class="flex flex-col">
      <!-- Barra de pesquisa -->
      <input @input="onInput" :value="query" type="text" class="search-bar"
        placeholder="Procure aqui a cidade, estado ou país" />

      <!-- Resultados de autocompletar -->
      <div class="autocomplete-results" v-if="autocompleteResults.length">
        <div v-for="result in autocompleteResults" :key="result.id" class="autocomplete-result"
          @click="selectAutocompleteResult(result)">
          {{ result.name }}, {{ result.region }}
        </div>
      </div>
    </div>

    <!-- Cidades salvas -->
    <div v-if="savedCities.length">
      <div v-for="(city, index) in savedCities" :key="city.name"
        class="saved-city flex px-2 py-2 flex-wrap right-0 relative">
        <p class="text-white flex">{{ city.name }}, {{ city.region }}, {{ city.temperature }}°C, {{ getConditionText(city)
        }}</p>
        <button @click="removeCity(index)" class="bg-red-500 text-white px-2 py-1 rounded-full ml-2">
          Remover
        </button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    query: String,
    autocompleteResults: Array,
    savedCities: Array,
    current: Object,
  },
  methods: {
    // Manipula o evento de entrada na barra de pesquisa
    onInput(event) {
      this.$emit('update:query', event.target.value);
      this.$emit('fetchAutocomplete');
    },
    // Seleciona um resultado de autocompletar
    selectAutocompleteResult(result) {
      this.$emit('selectAutocompleteResult', result);
    },
    // Remove uma cidade salva
    removeCity(index) {
      this.$emit('removeCity', index);
    },
    getConditionText(city) {
      return city.current && city.current.condition ? city.current.condition.text : 'N/A';
    },
  },
};
</script>


<style lang="scss" scoped>
.search-box {
  width: 100%;
  margin-bottom: 30px;
  display: flex;
  align-items: center;

  .search-bar {
    display: block;
    width: 450px;
    padding: 15px;
    color: #fff;
    font-size: 20px;
    appearance: none;
    border: none;
    outline: none;
    background: none;
    background-color: rgba(255, 255, 255, 0.5);
    border-radius: 0 16px 0 16px;
    transition: 0.4s;
  }

  .search-bar:focus {
    box-shadow: 0 0 16px rgba(0, 0, 0, 0.25);
    background-color: rgba(255, 255, 255, 0.75);
    border-radius: 16px 0 16px 0;
  }

  .saved-city {
    margin-left: 10%;
    width: 100%;
  }
}

@media (max-width: 768px) {
  .search-box {
    flex-wrap: wrap;

    .search-bar {
      max-width: 320px;
      width: 100%;
    }

    .saved-city {
      margin-left: 0;
      width: 100%;
    }
  }
}

.autocomplete-results {
  background-color: rgba(255, 255, 255, 0.25);
  max-height: 150px;
  overflow-y: auto;
}

.autocomplete-result {
  color: #fff;
  padding: 8px;
  cursor: pointer;
  transition: background-color 0.3s;
  border-radius: 30px;
}

.autocomplete-result:hover {
  background-color: #bc8282;
}
</style>
  