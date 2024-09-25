<template>
  <div class="app-container">
    <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/9/98/International_Pok%C3%A9mon_logo.svg/640px-International_Pok%C3%A9mon_logo.svg.png" alt="Pokémon Logo" class="pokemon-logo"/>

    <input 
      v-model="pokemonName" 
      @keyup.enter="fetchPokemonData" 
      placeholder="Choose your Pokemon" 
    />
    <button @click="fetchPokemonData">Buscar</button>

    <div v-if="errorMessage" class="error-message">{{ errorMessage }}</div>

    <div v-if="pokemon || showNoPokemonMessage" class="card-container">
      <div v-if="pokemon" class="card">
        <img :src="pokemon.sprites?.front_default" alt="Pokemon Image" class="pokemon-image" />
        
        <h2>{{ pokemon.name ? pokemon.name || capitalize : 'Nombre no disponible' }}</h2>
        
        <div class="types">
          <button 
            v-for="type in pokemon.types" 
            :key="type.slot" 
            :class="['type-button', typeColors[type.type.name]]"
          >
            {{ type.type.name }}
          </button>
        </div>

        <p><strong>Debilidades:</strong></p>
        <div class="weaknesses">
          <button 
            v-for="weakness in weaknesses" 
            :key="weakness" 
            :class="['type-button', typeColors[weakness]]"
          >
            {{ weakness }}
          </button>
        </div>
      </div>

      <div v-if="showNoPokemonMessage" class="no-pokemon-card">
        <img src="https://i.pinimg.com/564x/31/e7/36/31e736cb776e4d8781c1331d00eb7d3b.jpg" alt="Pokémon Not Found" class="no-pokemon-image" />
        <p>Pokémon no registrado en tu Pokédex</p>
      </div>

      <div v-if="pokemon" class="stats">
        <h3>Estadísticas</h3>
        <ul>
          <li v-for="stat in pokemon.stats" :key="stat.stat.name">
            <div class="stat-name">{{ stat.stat.name || capitalize }}: {{ stat.base_stat }}</div>
            <div class="stat-bar">
              <div 
                class="stat-bar-fill" 
                :style="{ width: stat.base_stat + '%' }"
              ></div>
            </div>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue';
export default {
  data() {
    return {
      pokemonName:'',
      pokemon: null,
      weaknesses: [],
      errorMessage: '',
      showNoPokemonMessage: false,
      typeColors: {
        normal: 'normal',
        fire: 'fire',
        water: 'water',
        electric: 'electric',
        grass: 'grass',
        ice: 'ice',
        fighting: 'fighting',
        poison: 'poison',
        ground: 'ground',
        flying: 'flying',
        psychic: 'psychic',
        bug: 'bug',
        rock: 'rock',
        ghost: 'ghost',
        dragon: 'dragon',
        dark: 'dark',
        steel: 'steel',
        fairy: 'fairy'
      }
    };
  },
  methods: {
    async fetchPokemonData() {
      try {
        let searchTerm = this.pokemonName.toLowerCase().trim();
        
        if (!searchTerm) {
          const randomId = Math.floor(Math.random() * 1010) + 1;
          searchTerm = randomId;
        }
        
        const response = await fetch(`https://pokeapi.co/api/v2/pokemon/${searchTerm}`);
        if (!response.ok) {
          this.showNoPokemonMessage = true;
          this.pokemon = null;
          this.weaknesses = [];
          return;
        }

        const data = await response.json();
        this.pokemon = data;
        this.showNoPokemonMessage = false;
        this.errorMessage = '';
        this.weaknesses = [];

        await this.fetchWeaknesses();
      } catch (error) {
        this.errorMessage = error.message;
        this.pokemon = null;
        this.weaknesses = [];
        this.showNoPokemonMessage = true;
      }
    },
    async fetchWeaknesses() {
      try {
        let weaknessesSet = new Set();
        
        for (const typeInfo of this.pokemon.types) {
          const response = await fetch(typeInfo.type.url);
          const typeData = await response.json();

          typeData.damage_relations.double_damage_from.forEach((weakness) => {
            weaknessesSet.add(weakness.name);
          });
        }
        this.weaknesses = Array.from(weaknessesSet);
      } catch (error) {
        console.error('Error al obtener debilidades:', error);
      }
    }
  },
  filters: {
    capitalize(value) {
      if (!value) return '';
      value = value.toString();
      return value.charAt(0).toUpperCase() + value.slice(1);
    }
  }
};
</script>



<style>
.app-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: 10px;
  font-family: 'Arial', sans-serif;
  background-image: url("https://w.wallhaven.cc/full/4l/wallhaven-4lmey2.png");
  background-repeat: no-repeat;
  background-size: 100%;
  background-position: center;
}

.pokemon-logo {
  width: 300px;
  margin-bottom: 20px;
}

input {
  padding: 8px;
  margin-bottom: 10px;
  font-size: 16px;
  border-radius: 5px;
  color:rgb(38, 0, 255);
  background: #c1cde6;
}

button {
  padding: 10px 12px;
  background-color: #ffcc00;
  border: none;
  cursor: pointer;
}

.card-container {
  display: flex;
  flex-direction: row;
  align-items: flex-start;
  gap: 20px;
  position: relative;
}

.card {
  display: flex;
  flex-direction: column;
  align-items: center;
  border: 1px solid #ddd;
  border-radius: 10px;
  padding: 20px;
  width: 300px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  background-image: url("https://images-wixmp-ed30a86b8c4ca887773594c2.wixmp.com/f/1e32d1c2-e8f4-41be-af8c-b35aff1fb04f/dcgd5q4-1a52d49f-4736-4e05-9073-d66d4b71f247.png/v1/fill/w_1024,h_1821,q_80,strp/blank_pokedex_phone_background_by_thealmightyl_dcgd5q4-fullview.jpg?token=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJ1cm46YXBwOjdlMGQxODg5ODIyNjQzNzNhNWYwZDQxNWVhMGQyNmUwIiwiaXNzIjoidXJuOmFwcDo3ZTBkMTg4OTgyMjY0MzczYTVmMGQ0MTVlYTBkMjZlMCIsIm9iaiI6W1t7ImhlaWdodCI6Ijw9MTgyMSIsInBhdGgiOiJcL2ZcLzFlMzJkMWMyLWU4ZjQtNDFiZS1hZjhjLWIzNWFmZjFmYjA0ZlwvZGNnZDVxNC0xYTUyZDQ5Zi00NzM2LTRlMDUtOTA3My1kNjZkNGI3MWYyNDcucG5nIiwid2lkdGgiOiI8PTEwMjQifV1dLCJhdWQiOlsidXJuOnNlcnZpY2U6aW1hZ2Uub3BlcmF0aW9ucyJdfQ.Bp44p5oW6-d0ctiXA4tASTcK5zgDR_pGOW2YnJgKP_A");
  background-repeat: no-repeat;
  background-size: 100%;
  background-position: center;
  margin-top: 4%;
}

.pokemon-image {
  width: 150px;
  height: 150px;
}

.types, .weaknesses {
  margin-top: 10px;
  display: flex;
  gap: 10px;
}

.type-button {
  padding: 5px 10px;
  border-radius: 5px;
  color: white;
  border: none;
  font-size: 14px;
  text-transform: capitalize;
  cursor: default;
}

.type-button.normal { background-color: #A8A77A; }
.type-button.fire { background-color: #EE8130; }
.type-button.water { background-color: #6390F0; }
.type-button.electric { background-color: #F7D02C; }
.type-button.grass { background-color: #7AC74C; }
.type-button.ice { background-color: #96D9D6; }
.type-button.fighting { background-color: #C22E28; }
.type-button.poison { background-color: #A33EA1; }
.type-button.ground { background-color: #E2BF65; }
.type-button.flying { background-color: #A98FF3; }
.type-button.psychic { background-color: #F95587; }
.type-button.bug { background-color: #A6B91A; }
.type-button.rock { background-color: #B6A136; }
.type-button.ghost { background-color: #735797; }
.type-button.dragon { background-color: #6F35FC; }
.type-button.dark { background-color: #705746; }
.type-button.steel { background-color: #B7B7CE; }
.type-button.fairy { background-color: #D685AD; }

.stats {
  background-image: url("https://i.pinimg.com/originals/e6/5f/4a/e65f4a73e2a09c6ef3661d1197587f57.jpg") ;
  background-repeat: no-repeat;
  background-size: 130%;
  background-position: center;
  width: 300px;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  margin-top: 10%;
}

.stats ul {
  list-style-type: none;
  padding: 0;
}

.stats li {
  margin-bottom: 10px;
}

.stat-name {
  margin-bottom: 5px;
}

.stat-bar {
  width: 100%;
  height: 10px;
  background-color: #ddd;
  border-radius: 5px;
  overflow: hidden;
}

.stat-bar-fill {
  height: 100%;
  background-color: #4CAF50;
  transition: width 0.3s ease;
}

.weaknesses {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-top: 10px;
}

.no-pokemon-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  border: 1px solid #ddd;
  border-radius: 10px;
  margin: 20%;
  width: 500px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  background-image: url("https://i.pinimg.com/564x/96/22/4c/96224c01e410fb08e7922e2ec958f974.jpg"); 
}

.no-pokemon-image {
  width: 150px;
  height: 150px;
  margin-bottom: 10px;
}
.p{
  margin-top: 4%;
}
  @media (max-width: 800px) {
    .card-container{
      display: flex;
      flex-direction: column;
    }
  }
</style>















