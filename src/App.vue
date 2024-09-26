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

        <p><strong>Debilidades:</strong></p>
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
import { ref, onMounted } from 'vue';

export default {
  setup() {
    const pokemonName = ref('');
    const pokemon = ref(null);
    const weaknesses = ref([]);
    const errorMessage = ref('');
    const showNoPokemonMessage = ref(false);

    const typeColors = {
      normal: '#A8A77A',
      fire: '#EE8130',
      water: '#6390F0',
      electric: '#F7D02C',
      grass: '#7AC74C',
      ice: '#96D9D6',
      fighting: '#C22E28',
      poison: '#A33EA1',
      ground: '#E2BF65',
      flying: '#A98FF3',
      psychic: '#F95587',
      bug: '#A6B91A',
      rock: '#B6A136',
      ghost: '#735797',
      dragon: '#6F35FC',
      dark: '#705746',
      steel: '#B7B7CE',
      fairy: '#D685AD'
    };

    const fetchPokemonData = async () => {
      try {
        let searchTerm = pokemonName.value.toLowerCase().trim();

        if (!searchTerm) {
          const randomId = Math.floor(Math.random() * 1010) + 1;
          searchTerm = randomId;
        }

        const response = await fetch(`https://pokeapi.co/api/v2/pokemon/${searchTerm}`);
        if (!response.ok) {
          showNoPokemonMessage.value = true;
          pokemon.value = null;
          weaknesses.value = [];
          return;
        }

        const data = await response.json();
        pokemon.value = data;
        showNoPokemonMessage.value = false;
        errorMessage.value = '';
        weaknesses.value = [];

        await fetchWeaknesses();
      } catch (error) {
        errorMessage.value = error.message;
        pokemon.value = null;
        weaknesses.value = [];
        showNoPokemonMessage.value = true;
      }
    };

    const fetchWeaknesses = async () => {
      try {
        const weaknessesSet = new Set();

        for (const typeInfo of pokemon.value.types) {
          const response = await fetch(typeInfo.type.url);
          const typeData = await response.json();

          typeData.damage_relations.double_damage_from.forEach((weakness) => {
            weaknessesSet.add(weakness.name);
          });
        }

        weaknesses.value = Array.from(weaknessesSet);
      } catch (error) {
        console.error('Error al obtener debilidades:', error);
      }
    };

    const capitalize = (value) => {
      if (!value) return '';
      value = value.toString();
      return value.charAt(0).toUpperCase() + value.slice(1);
    };

    return {
      pokemonName,
      pokemon,
      weaknesses,
      errorMessage,
      showNoPokemonMessage,
      typeColors,
      fetchPokemonData,
      capitalize
    };
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

.pokemon-image-container {
  width: 50%;
  height: 50%;
  border-radius: 0%;
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 10px;
}

.pokemon-image {
  width: 100px;
  height: 100px;
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

.stats {
  background-image: url("https://i.pinimg.com/originals/e6/5f/4a/e65f4a73e2a09c6ef3661d1197587f57.jpg") ;
  background-repeat: no-repeat;
  background-size: 130%;
  background-position: center;
  width: 300px;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  margin-top: 5%;

}

.h6{
  margin: 0%;
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
  }

}
</style>















