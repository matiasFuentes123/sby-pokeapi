<template>
  <template v-if="!pokemonWithoutEvolutionListLoading">
    <v-table style="background-color: #1c1b1f">
      <thead>
        <tr>
          <th style="border-bottom: 1px solid rgba(255, 255, 255, 0.2); color: #fff">
            Nombre Pokemon
          </th>
        </tr>
      </thead>
      <tbody style="color: #fff">
        <tr v-for="(pokemon, index) in currentDisplayedPokemonWithoutEvolutionList" :key="index">
          <td style="border-bottom: 1px solid rgba(255, 255, 255, 0.2)">
            <span v-if="pokemon.name">
              {{ pokemon.name }}
            </span>
            <span v-else>-</span>
          </td>
        </tr>
      </tbody>
    </v-table>
    <div style="display: flex; justify-content: center">
      <v-btn color="#fff" :disabled="page === 1" @click="goPrevPage">Anterior</v-btn>
      <v-btn color="#fff" :disabled="page === totalPages" @click="goNextPage">Siguiente</v-btn>
    </div>
    <div style="display: flex; justify-content: center; margin-top: 24px">
      <div style="display: flex; flex-direction: column; align-items: center; width: 50%; padding: 24px; border: 1px solid rgba(255, 255, 255, 0.2)">
        <span style="color: #fff">
          Pokemon sin evolución con menor peso
        </span>
        <span style="color: #fff">
          {{ pokemonWithoutEvolutionWithLessWeight && pokemonWithoutEvolutionWithLessWeight.name
              ? pokemonWithoutEvolutionWithLessWeight.name
              : 'Nombre no disponible' 
          }}
        </span>
      </div>
    </div>
  </template>
  <!-- <div v-else>cargando....</div> -->
  <div v-else style="display: flex; justify-content: center; align-items: center; height: 100%">
    <v-progress-circular indeterminate color="#fff" size="100"/>
  </div>
</template>

<script>
import { Pokedex } from 'pokeapi-js-wrapper'
export default {
  data() {
    return {
      pokedexApi: new Pokedex(),
      pokemonWithoutEvolutionList: [],
      currentDisplayedPokemonWithoutEvolutionList: [],
      pokemonWithoutEvolutionListLoading: true,
      page: 1,
      totalPages: 0,
      pagination: null,
      pokemonWithoutEvolutionWithLessWeight: null
    }
  },
  watch: {
    page() {
      const initialIndex = (this.page * 10) - 9
      const finalIndex = (this.page * 10) + 1
      this.currentDisplayedPokemonWithoutEvolutionList = this.pokemonWithoutEvolutionList.slice(initialIndex, finalIndex)
    }
  },
  mounted() {
    // this.getPokemonList()
  },
  methods: {
    async getPokemonList() {
      const pokemonList = await this.pokedexApi.getPokemonsList()

      const pokemonListWithData = []

      for await (const pokemon of pokemonList.results.map(pokemon => this.pokedexApi.getPokemonByName(pokemon.name))) {
        pokemonListWithData.push(pokemon)
      }

      for (const pokemon of pokemonListWithData) {

        const species = await this.pokedexApi.getPokemonSpeciesByName(pokemon.species.name)
        const chainEvolutionId = species.evolution_chain.url.split('/')[6]
        const chainEvolution = await this.pokedexApi.getEvolutionChainById(chainEvolutionId)

        if (chainEvolution.chain.evolves_to && chainEvolution.chain.evolves_to.length > 0) {
          chainEvolution.chain.evolves_to.forEach(firstEvolution => {
            if (firstEvolution.evolves_to && firstEvolution.evolves_to.length > 0) {
              firstEvolution.evolves_to.forEach(secondEvolution => {
                if (secondEvolution.evolves_to && secondEvolution.evolves_to.length > 0) {
                  secondEvolution.evolves_to.forEach(thirdEvolution => {
                    if (thirdEvolution.species.name.localeCompare(pokemon.species.name) === 0) {
                      this.evaluatePokemonWithLessWeight(pokemon)
                      this.pokemonWithoutEvolutionList.push(pokemon)
                      this.evaluateTotalPages()
                    }
                  })
                } else if (secondEvolution.species.name.localeCompare(pokemon.species.name) === 0) {
                  this.evaluatePokemonWithLessWeight(pokemon)
                  this.pokemonWithoutEvolutionList.push(pokemon)
                  this.evaluateTotalPages()
                }   
              })
            } else if (firstEvolution.species.name.localeCompare(pokemon.species.name) === 0) {
              this.evaluatePokemonWithLessWeight(pokemon)
              this.pokemonWithoutEvolutionList.push(pokemon)
              this.evaluateTotalPages()
            }
          })
        } else {
          this.evaluatePokemonWithLessWeight(pokemon)
          this.pokemonWithoutEvolutionList.push(pokemon)
          this.evaluateTotalPages()
          // this.pushToPokemonWithoutEvolutionList(pokemon)
        }
      }
      this.currentDisplayedPokemonWithoutEvolutionList = this.pokemonWithoutEvolutionList.slice(0, 10)
      this.pokemonWithoutEvolutionListLoading = false
    },
    evaluateTotalPages() {
      if (this.pokemonWithoutEvolutionList.length % 10 === 0) {
        this.totalPages++
      }
    },
    goNextPage() {
      if (this.page < this.totalPages) {
        this.page++
      }
    },
    goPrevPage() {
      if (this.page !== 1) {
        this.page--
      }
    },
    evaluatePokemonWithLessWeight(pokemon) {
      if (!this.pokemonWithoutEvolutionWithLessWeight) {
        this.pokemonWithoutEvolutionWithLessWeight = pokemon
      } else if (pokemon.weight < this.pokemonWithoutEvolutionWithLessWeight.weight) {
        this.pokemonWithoutEvolutionWithLessWeight = pokemon
      }
    }
  }
}
</script>

<style>

</style>