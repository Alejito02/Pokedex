<template>
  <div class="app-container">
    <img
      src="https://upload.wikimedia.org/wikipedia/commons/thumb/9/98/International_Pok%C3%A9mon_logo.svg/1280px-International_Pok%C3%A9mon_logo.svg.png"
      alt="Pokémon Logo" class="pokemon-logo" />

    <input v-model="pokemonName" @keyup.enter="fetchPokemonData" placeholder="Choose your Pokemon" />
    <button @click="fetchPokemonData">Buscar</button>

    <div v-if="pokemon || showNoPokemonMessage" class="card-container">
      <div v-if="pokemon" class="weakness-card">
        <h4 class="titleW">Weakness</h4>
        <div class="weaknesses">
          <button v-for="weakness in weaknesses" :key="weakness" :style="{ backgroundColor: typeColors[weakness] }"
            class="type-butt0n">
            {{ weakness }}
          </button>
        </div>
      </div>

      <div v-if="pokemon" class="card">
        <div class="pokemon-image-container" :style="{ backgroundColor: typeColors[pokemon.types[0].type.name] }">
          <img :src="pokemon.sprites?.front_default" alt="Pokemon Image" class="pokemon-image"
            style="margin-bottom: 10px;" />
        </div>

        <h4 class="NombreP">{{ pokemon.name ? pokemon.name || capitalize : 'Nombre no disponible' }}</h4>

        <div class="types">
          <button v-for="type in pokemon.types" :key="type.slot"
            :style="{ backgroundColor: typeColors[type.type.name] }" class="type-butt0n">
            {{ type.type.name }}
          </button>
        </div>
      </div>

      <div v-if="showNoPokemonMessage" class="no-pokemon-card">
        <img src="https://i.pinimg.com/564x/31/e7/36/31e736cb776e4d8781c1331d00eb7d3b.jpg" alt="Pokémon Not Found"
          class="no-pokemon-image" />
        <p>Pokémon no registrado en tu Pokédex</p>
      </div>

      <div v-if="pokemon" class="stats">
        <h6>Stats</h6>
        <ul>
          <li v-for="stat in pokemon.stats" :key="stat.stat.name">
            <div class="stat-name">{{ stat.stat.name || capitalize }}: {{ stat.base_stat }}</div>
            <div class="stat-bar">
              <div class="stat-bar-fill" :style="{ width: (stat.base_stat / 225 * 100) + '%' }"></div>
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
  height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  font-family: 'Arial', sans-serif;
}

.pokemon-logo {
  width: 300px;
  margin-bottom: 20px;
}

input {
  padding: 17px;
  margin-bottom: 10px;
  font-size: 16px;
  border-radius: 5px;
  color: rgb(0, 0, 0);
  background: #c1cde6;
  border: none;
  width: 300px;
  box-shadow: 0px 0px 0px 3px rgba(24, 78, 255, 0.788);
}

button {
  padding: 15px 21px;
  background-color: #ffcc00;
  border: none;
  border-radius: 15%;
  cursor: pointer;
  font-size: large;
  color: white;
  box-shadow: 0px 0px 0px 4px #ff0707c9;
}

.card-container {
   display: flex;
   flex-direction: row;
   align-items: flex-start;
   position: relative;
   margin-top: 10%;
}

.weakness-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  width: 300px;
  height: 343px;
  margin-top: 4%;
  background-image: url("https://www.pngfind.com/pngs/b/48-480534_text-box-png.png");
  background-size: cover;
  background-position: center;
}

.titleW {
  margin-top: 15%;
  font-family: Arial, Helvetica, sans-serif;
}

.weaknesses {
   margin-top: 10px;
    display: flex;
    flex-wrap: nowrap;
    gap: 8px;
    align-content: space-between;
    align-items: center;
    justify-content: space-around;
    flex-direction: column;
}

.type-butt0n {
  padding: 5px 10px;
  border-radius: 5px;
  color: white;
  border: none;
  font-size: 14px;
  text-transform: capitalize;
  cursor: default;
  box-shadow: none;
}

.card {
  display: flex;
  flex-direction: column;
  align-items: center;
  border-radius: 10px;
  padding: 10px;
  width: 330px;
  height: 500px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  background-image: url("https://i.pinimg.com/736x/14/bd/68/14bd686faedccee521a84e293f50cbb6.jpg");
  background-repeat: no-repeat;
  background-size: 100%;
  background-position: center;
}

.pokemon-image-container {
    width: 87%;
    height: 51%;
    border-radius: 1%;
    display: flex;
    justify-content: center;
    align-items: center;
    margin-bottom: 23%;
    margin-top: 27%;
}

.pokemon-image {
  width: 200px;
  height: 200px;
}

.stats {
  background-repeat: no-repeat;
  background-size: cover;
  background-position: center;
  width: 300px;
  height: 343px;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  margin-top: 4%;
  padding: 20px;
}

.stats h6 {
    font-size: x-large;
    margin: 0%;
    margin-bottom: 15%;
    margin-left: 28%;
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
  background-image: url("https://i.pinimg.com/736x/14/bd/68/14bd686faedccee521a84e293f50cbb6.jpg");
}

.no-pokemon-image {
  width: 150px;
  height: 150px;
  margin-bottom: 10px;
}

.NombreP {
    margin-left: 7%;
    margin-bottom: 10px;
    font: revert-layer;
    margin-top: 0;
}

.types{
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
    margin-left: 6%;
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
