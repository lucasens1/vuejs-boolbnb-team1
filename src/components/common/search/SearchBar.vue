<script>
import { mapState, mapMutations, mapActions } from 'vuex';
import { storejs } from '../../../store';

export default {
  data() {
    return {
      clicked: true,
      // selected: '',
      avServices: [],
      checcate: [],
      radius: 20,
      beds: 1,
      rooms: 2,
      storejs,
      serviceString: ''
    };
  },
  created() {
    storejs.selected = '';
  },
  mounted() {
    // this.$store.dispatch('fetchFlats');
    document.addEventListener('click', this.handleClickOutside);
    fetch(`http://127.0.0.1:8000/api/services`)
      .then(response => response.json())
      .then(data => {
        this.avServices = data;
      })
      .catch(error => {
        console.error('Errore durante il fetch:', error);
      });
  },
  // beforeDestroy() {
  //   document.removeEventListener('click', this.handleClickOutside);
  // },
  computed: {
    ...mapState(['query', 'suggestions', 'selectedSuggestion', 'filtersDropdownVisible', 'lat', 'lon'])
  },
  methods: {
    ...mapMutations(['setQuery', 'toggleFiltersDropdown', 'closeFiltersDropdown', 'setFilterServices', 'setRadius', 'setBeds', 'setRooms']),
    ...mapActions(['fetchSuggestions', 'selectSuggestion', 'fetchFlats', 'cercaAppartamenti']),
    handleInput(event) {
      this.setQuery(event.target.value);
      this.fetchSuggestions();
      this.clicked = true;
    },
    toggleFilters() {
      this.toggleFiltersDropdown(!this.filtersDropdownVisible);
    },
    toggleClick() {
      this.clicked = false;
    },
    handleClickOutside(event) {
      const searchBar = this.$refs.searchBar;
      const filtersDropdown = this.$refs.filtersDropdown;
      const filtersButton = this.$refs.filtersButton;
      const searchButton = this.$refs.searchButton;

      // console.log('Clicked outside:', !searchBar.contains(event.target));
      // console.log('Dropdown:', !filtersDropdown.contains(event.target));
      // console.log('Filters button:', !filtersButton.contains(event.target));
      // console.log('Search button:', !searchButton.contains(event.target));

      if (
        (filtersDropdown && !filtersDropdown.contains(event.target)) &&
        (searchBar && !searchBar.contains(event.target)) &&
        (filtersButton && !filtersButton.contains(event.target)) &&
        (searchButton && !searchButton.contains(event.target))
      ) {
        if (this.filtersDropdownVisible) {
          this.$store.commit('closeFiltersDropdown');

        }
      }
      this.clicked = false;
      this.handleSearchBarClick(event);
    },
    handleSearchBarClick(event) {
      const searchBar = this.$refs.searchBar;
      const filtersDropdown = this.$refs.filtersDropdown;

      if (searchBar && searchBar.contains(event.target)) {
        // Chiudi il dropdown dei filtri quando clicchi sulla search bar
        if (this.filtersDropdownVisible) {
          this.$store.commit('closeFiltersDropdown');
        }
      }
    },
    getFilter() {
      // prendo tutti elementi con attributo name=services[] e che sono checked
      let arrayCheckedElem = document.querySelectorAll('[name = "services[]"]:checked');

      const services = [];

      arrayCheckedElem.forEach(element => {
        services.push(parseInt(element.value));
      });

      this.serviceString = '['+ services.join(',') + ']';


      this.setFilterServices(stringArr);
  
      this.$store.commit('closeFiltersDropdown');
      
    },
    getImagePath(image) {
      return new URL(`../../../assets/img/services/${image}`, import.meta.url).href;
    },

  }
};
</script>

<template>
  <div class="container-fluid d-flex flex-column flex-md-row align-items-center justify-content-center"
    id="ms_cont-s-bar">
    <div class="ms_searchbar rounded-4 my-md-5 pe-4" ref="searchBar">
      <input type="text" id="ms_input" placeholder="Cerca qualcosa.." v-model="storejs.selected" @input="handleInput"
        class="w-100" />
      <div id="dropdown" v-if="suggestions.length" class="dropdown-menu" :class="clicked ? 'show' : ''">
        <a v-for="(suggestion, index) in suggestions" :key="index" class="dropdown-item"
          @click.prevent="selectSuggestion(suggestion), storejs.selected = suggestion.address.freeformAddress , storejs.lat = suggestion.position.lat, storejs.lon = suggestion.position.lon">
          {{ suggestion.address.freeformAddress }}
        </a>
      </div>
    </div>
    <!-- div button -->
    <div class="d-flex">
      <!-- Bottone per filtri -->
      <button class="ms_button rounded-4" @click="toggleFilters()" ref="filtersButton">Filtri</button>
      <div v-show="filtersDropdownVisible" class="filters-dropdown" ref="filtersDropdown">
        <!-- Aggiungi qui i tuoi filtri -->
        <div class="filter-item">
          <div class="container">
            <div class="row g-2">
              <div class="col-12">
                <div class="container">
                  <div class="row g-4 justify-content-between">
                    <div class="col-12 range-guests d-flex">

                      <div class="container">
                        <div class="row">
                          <div class="col-12 col-lg-4 p-2 text-center">
                            <label class="mb-2" for="rangeValore">Raggio di ricerca:</label> <output
                              for="rangeValore">{{ radius }} Km</output>
                            <input type="range" id="rangeValore" min="1" max="100" v-model="radius">

                          </div>

                          <div class="col-md-6 col-sm-12 col-lg-4 p-2 text-center">

                            <span class="d-inline-block mb-2">Numero letti</span>
                            <div class="num-filter">
                              <i class="fa-solid fa-circle-minus" @click="beds > 1 ? beds-- : ''"></i>
                              <span class="mx-4">
                                {{ beds }}
                              </span>
                              <i class="fa-solid fa-circle-plus" @click="beds < 99 ? beds++ : ''"></i>
                            </div>

                          </div>

                          <div class="col-md-6 col-sm-12 col-lg-4 p-2 text-center">

                            <span class="d-inline-block mb-2">Numero stanze</span>
                            <div class="num-filter">
                              <i class="fa-solid fa-circle-minus" @click="rooms > 1 ? rooms-- : ''"></i>
                              <span class="mx-4">
                                {{ rooms }}
                              </span>
                              <i class="fa-solid fa-circle-plus" @click="rooms < 99 ? rooms++ : ''"></i>
                            </div>

                          </div>
                        </div>
                      </div>



                    </div>
                  </div>
                </div>
              </div>
              <div class="col-6 col-md-4 col-lg-3 service-list" v-for="service in avServices" :key="service">

                <input type="checkbox" :id="service.name" class="check-service" name="services[]" :value="service.id" />
                <label :for="service.name" class="text-truncate">
                  <img :src="getImagePath(service.icon)" :alt="'Icona ' + service.name">
                  {{ service.name }}
                </label>
              </div>
            </div>
          </div>


        </div>
        <!-- Aggiungi altri filtri qui -->
      </div>
      <router-link :to="{ name: 'travel',  query: { beds: beds, rooms: rooms, services: serviceString , lat: storejs.lat, lon: storejs.lon, radius: radius }  }">
        <button @click="getFilter()" class="ms_button rounded-4" ref="searchButton" :disabled = "storejs.selected.trim() == '' ">Cerca</button>
      </router-link>
    </div>
    
    <!-- / div button -->
  </div>
</template>

<style lang="scss" scoped></style>
