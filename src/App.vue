<template>
  <div class="app-container">
    <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/9/98/International_Pok%C3%A9mon_logo.svg/640px-International_Pok%C3%A9mon_logo.svg.png" alt="Pokémon Logo" class="pokemon-logo" />

    <input 
      v-model="pokemonName" 
      @keyup.enter="fetchPokemonData" 
      placeholder="Choose your Pokemon" 
    />
    <button @click="fetchPokemonData">Buscar</button>

    <div v-if="pokemon || showNoPokemonMessage" class="card-container">
      <div v-if="pokemon" class="weakness-card">
        <h4 class="titleW">Debilidades</h4>
        <div class="weaknesses">
          <button 
            v-for="weakness in weaknesses" 
            :key="weakness" 
            :style="{ backgroundColor: typeColors[weakness] }"
            class="type-button"
          >
            {{ weakness }}
          </button>
        </div>
      </div>

      <div v-if="pokemon" class="card">
        <div class="pokemon-image-container" :style="{ backgroundColor: typeColors[pokemon.types[0].type.name] }">
          <img :src="pokemon.sprites?.front_default" alt="Pokemon Image" class="pokemon-image" />
        </div>
        
        <h4>{{ pokemon.name ? pokemon.name || capitalize : 'Nombre no disponible' }}</h4>
        
        <div class="types">
          <button 
            v-for="type in pokemon.types" 
            :key="type.slot" 
            :style="{ backgroundColor: typeColors[type.type.name] }"
            class="type-button"
          >
            {{ type.type.name }}
          </button>
        </div>
      </div>

      <div v-if="showNoPokemonMessage" class="no-pokemon-card">
        <img src="https://i.pinimg.com/564x/31/e7/36/31e736cb776e4d8781c1331d00eb7d3b.jpg" alt="Pokémon Not Found" class="no-pokemon-image" />
        <p>Pokémon no registrado en tu Pokédex</p>
      </div>

      <div v-if="pokemon" class="stats">
        <h6>Estadísticas</h6>
        <ul>
          <li v-for="stat in pokemon.stats" :key="stat.stat.name">
            <div class="stat-name">{{ stat.stat.name || capitalize }}: {{ stat.base_stat }}</div>
            <div class="stat-bar">
              <div 
                class="stat-bar-fill" 
                :style="{ width: (stat.base_stat / 225 * 100) + '%' }"
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
  setup() {
    const pokemonName = ref('');
    const pokemon = ref(null);
    const showNoPokemonMessage = ref(false);
    const errorMessage = ref('');
    const weaknesses = ref([]);
    
    const typeColors = ref({
      normal: '#A8A878',
      fire: '#F08030',
      water: '#6890F0',
      electric: '#F8D030',
      grass: '#78C850',
      ice: '#98D8D8',
      fighting: '#C03028',
      poison: '#A040A0',
      ground: '#E0C068',
      flying: '#A890F0',
      psychic: '#F85888',
      bug: '#A8B820',
      rock: '#B8A038',
      ghost: '#705898',
      dragon: '#7038F8',
      dark: '#705848',
      steel: '#B8B8D0',
      fairy: '#EE99AC',
    });

    const capitalize = (word) => {
      return word.charAt(0).toUpperCase() + word.slice(1);
    };

    const getRandomPokemonId = () => {
      return Math.floor(Math.random() * 1010) + 1;
    };

    const fetchPokemonData = async () => {
      let pokemonIdOrName = pokemonName.value.trim().toLowerCase();

      if (!pokemonIdOrName) {
        pokemonIdOrName = getRandomPokemonId();
      }

      try {
        const response = await fetch(`https://pokeapi.co/api/v2/pokemon/${pokemonIdOrName}`);

        if (!response.ok) {
          errorMessage.value = 'Pokémon no encontrado. Verifica el nombre.';
          pokemon.value = null;
          showNoPokemonMessage.value = true;
          weaknesses.value = [];
          return;
        }

        const data = await response.json();
        pokemon.value = data;
        showNoPokemonMessage.value = false;
        errorMessage.value = '';

        const typeResponses = await Promise.all(
          data.types.map((typeInfo) => fetch(`https://pokeapi.co/api/v2/type/${typeInfo.type.name}`))
        );
        const typeData = await Promise.all(typeResponses.map((res) => res.json()));

        weaknesses.value = typeData
          .flatMap((typeInfo) => typeInfo.damage_relations.double_damage_from)
          .map((type) => type.name);
      } catch (error) {
        errorMessage.value = 'Error al obtener los datos. Intenta nuevamente.';
        pokemon.value = null;
        showNoPokemonMessage.value = true;
        weaknesses.value = [];
      }
    };

    return {
      pokemonName,
      pokemon,
      showNoPokemonMessage,
      errorMessage,
      weaknesses,
      typeColors,
      capitalize,
      fetchPokemonData,
    };
  },
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
  color: rgb(38, 0, 255);
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

.weakness-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  width: 300px;
  margin-top: 4%;
  background-image: url(https://i.pinimg.com/564x/9f/13/ec/9f13ec9c73bd9916ab5d263e3fa89d78.jpg);
  background-size: 100%;
  background-position: inherit;
}

.titleW{
  margin-top: 30%;
  font-family: Arial, Helvetica, sans-serif;
}

.weaknesses {
  margin-top: 10px;
  display: flex;
  flex-wrap: wrap;
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

.card {
  display: flex;
  flex-direction: column;
  align-items: center;
  border-radius: 10px;
  padding: 20px;
  width: 300px;
  height: 300px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  background-image: url("https://images-wixmp-ed30a86b8c4ca887773594c2.wixmp.com/f/1e32d1c2-e8f4-41be-af8c-b35aff1fb04f/dcgd5q4-1a52d49f-4736-4e05-9073-d66d4b71f247.png/v1/fill/w_1024,h_1821,q_80,strp/blank_pokedex_phone_background_by_thealmightyl_dcgd5q4-fullview.jpg?token=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJ1cm46YXBwOjdlMGQxODg5ODIyNjQzNzNhNWYwZDQxNWVhMGQyNmUwIiwiaXNzIjoidXJuOmFwcDo3ZTBkMTg4OTgyMjY0MzczYTVmMGQ0MTVlYTBkMjZlMCIsIm9iaiI6W1t7ImhlaWdodCI6Ijw9MTgyMSIsInBhdGgiOiJcL2ZcLzFlMzJkMWMyLWU4ZjQtNDFiZS1hZjhjLWIzNWFmZjFmYjA0ZlwvZGNnZDVxNC0xYTUyZDQ5Zi00NzM2LTRlMDUtOTA3My1kNjZkNGI3MWYyNDcucG5nIiwid2lkdGgiOiI8PTEwMjQifV1dLCJhdWQiOlsidXJuOnNlcnZpY2U6aW1hZ2Uub3BlcmF0aW9ucyJdfQ.Bp44p5oW6-d0ctiXA4tASTcK5zgDR_pGOW2YnJgKP_A");
  background-repeat: no-repeat;
  background-size: 100%;
  background-position: center;
  margin-top: 4%;
}

.pokemon-image-container {
  width: 80%;
  height: 100%;
  border-radius: 7%;
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 10px;
}

.pokemon-image {
  width: 100px;
  height: 100px;
}

.stats {
  background-image: url("https://i.pinimg.com/originals/e6/5f/4a/e65f4a73e2a09c6ef3661d1197587f57.jpg");
  background-repeat: no-repeat;
  background-size: 130%;
  background-position: center;
  width: 300px;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  margin-top: 10%;
  padding: 20px;
}
.stats,h6{
  margin: 0%;
  margin-bottom: 15%;
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

@media (max-width: 800px) {
  .card-container {
    display: flex;
    flex-direction: column;
  }
  .no-pokemon-card {
    display: flex;
    flex-direction: column;
    margin: 0%;
  }
}

</style>

