<template>
  <div class="pokemonPage" :key="pokemon">
    <Loader v-if="isLoading" :full-page="true" />

    <DexNavigation v-if="!isLoading" :nextNum="nextNum" :prevNum="prevNum"/>

    <div class="pokemon-data" v-if="!isLoading">
      <div id="head" class="poke-head">
        <h1 class="pokedex-num">#{{ findIndex(speciesInfo.id) }}</h1>

        <img :src="getPhotoUrl()" alt="" class="pokemon-image" id="poke-img" v-on:click="showEasterEgg()" />

        <!-- Grab the base name of the Pokémon -->
        <h1 class="pokemon-name" id="pokemon-name">
          {{ pokeName }}
        </h1>

        <!-- Grab the type(s) of the Pokémon -->
        <div class="pokemon-types">
          <div
            :class="'type-' + typeInfo.type.name"
            v-for="typeInfo in pokeInfo.types"
            v-bind:key="typeInfo.slot"
          >
            {{ toUpper(typeInfo.type.name) }}
          </div>
        </div>

        <div class="pokemon-desc">
          <p>
            A {{ getTyping(pokeInfo.types) }} type Pokémon introduced in
            {{ getGeneration(speciesInfo.generation.name) }}.
            {{ getFlavorText(speciesInfo.flavor_text_entries) }}
          </p>
        </div>
      </div>

      <!-- <div class="alternateForms" v-if="pokeInfo.forms.length > 1 || speciesInfo.varieties.length > 1">
        <strong>Alternate Forms: </strong>
        <p>
          {{ getAlternateForms(speciesInfo.varieties, pokeInfo.forms) }}
        </p>
        <span class="form" v-for="(form, index) in pokeInfo.forms" :key="index" >
          {{ form.name }}
        </span>
      </div> -->

      <div
        id="basic-info"
        class="pokemon-basic-info"
        v-bind:class="'info-box border-' + speciesInfo.color.name"
      >
        <h3>Pokédex Data</h3>

        <p style="margin-bottom: 0"><strong>Abilities:</strong></p>

        <details v-for="(abilityInfo, index) in pokeInfo.abilities" :key="index" >
          <summary
            :id="'ab-' + abilityInfo.ability.name"
            class="poke-abilities"
            :class="'poke-ab-' + speciesInfo.color.name"
          >
            {{ getAbilityName(abilityInfo, index) }}
          </summary>
          <p
            :id="'ab-txt-' + abilityInfo.ability.name"
            :class="'poke-ab-txt-' + abilityInfo.ability.name"
          >
            {{ getAbilityDesc(index)  }}
          </p>
        </details>

        <p><strong>Species:</strong> {{ getSpecies(speciesInfo.genera) }}</p>

        <p>
          <strong>Color:</strong>
          <span :class="'color-' + speciesInfo.color.name">
            {{ toUpper(speciesInfo.color.name) }}
          </span>
        </p>

        <p><strong>Shape:</strong> {{ toUpper(speciesInfo.shape.name) }}</p>

        <p>
          <strong>Height:</strong> {{ getHeight(pokeInfo.height) }} <br />
          <strong>Weight:</strong> {{ getWeight(pokeInfo.weight) }}
        </p>

        <p>
          <strong>Japanese Name:</strong>
          {{ getJapaneseName(speciesInfo.names) }}
        </p>
      </div>

      <div id="type-defenses" class="typeDefenses" :class="'info-box border-' + speciesInfo.color.name">
        <h3>Type Defenses</h3>
        <p>Effectiveness of each move typing on {{ toUpper(speciesInfo.name) }}</p>
        <TypeEffectiveness :typing="pokeInfo.types" />
      </div>

      <!-- Pokemon Training Info Box -->
      <div id="training" class="pokemon-training" :class="'info-box border-' + speciesInfo.color.name">
        <h3>Training</h3>
        <div class="poke-training-box">
          <div class="poke-evs-all">
            <h4>EV Yield</h4>

            <div class="poke-evs-3">
              <span class="poke-evs" v-for="(statInfo, index) in getStats(pokeInfo.stats)" :key="index" :class="'poke-ev-' + statInfo.stat_names.short.replace('.','').toLowerCase()">
                <strong>{{ statInfo.stat_names.short }}</strong>
                <span>{{ statInfo.effort }}</span>
              </span>
            </div>
          </div>

          <div class="poke-training">
            <table class="poke-train-table">
              <tbody>
                <tr>
                  <td class="poke-train-c1"><strong>Base EXP:</strong></td>
                  <td class="poke-train-c2">{{ pokeInfo.base_experience }}</td>
                </tr>
                <tr>
                  <td class="poke-train-c1">
                    <strong>Base Happiness:</strong>
                  </td>
                  <td class="poke-train-c2">
                    {{ speciesInfo.base_happiness }}
                  </td>
                </tr>
                <tr>
                  <td class="poke-train-c1"><strong>Capture Rate:</strong></td>
                  <td class="poke-train-c2">{{ speciesInfo.capture_rate }}</td>
                </tr>
                <tr>
                  <td class="poke-train-c1"><strong>Growth Rate:</strong></td>
                  <td class="poke-train-c2">
                    {{ toUpper(speciesInfo.growth_rate.name) }}
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>

      <div class="statsAndBreeding">
        <!-- Pokemon Stats Info Box -->
        <div
          id="stats"
          class="pokemon-stats"
          v-bind:class="'info-box border-' + speciesInfo.color.name"
        >
          <h3>Base Stats</h3>

          <table class="base-stats">
            <tbody>
              <tr v-for="(statInfo, index) in getStats(pokeInfo.stats)" :key="index">
                <td class="base-stats-c1" :class="'base-stat-' + statInfo.stat_names.short.replace('.','').toLowerCase()">
                  <strong>{{ statInfo.stat_names.short }}:</strong>
                </td>
                <td class="base-stats-c2">{{ statInfo.base_stat }}</td>
              </tr>
            </tbody>
          </table>
        </div>

        <!-- Pokemon Breeding Info Box -->
        <div
          id="breeding"
          class="pokemon-breeding"
          v-bind:class="'info-box border-' + speciesInfo.color.name"
        >
          <h3>Breeding Info</h3>
          <p>
            <strong>Gender Rate:</strong>
            {{ getGenderRate(speciesInfo.gender_rate) }}
          </p>
          <p>
            <strong>Egg-Group(s):</strong>
            {{ getEggGroups(speciesInfo.egg_groups) }}
          </p>
          <p>
            <strong>Hatching:</strong>
            {{ calcHatching(speciesInfo.hatch_counter) }}
          </p>
        </div>
      </div>

      <!-- Evolution Chain data -->
      <div id="evolutions" class="evoChain" :class="'info-box border-' + speciesInfo.color.name">
        <!-- <h3>Evolution Chain</h3> -->
        <h3>Evolutions</h3>
        <EvolutionChain :chain="getId(speciesInfo.evolution_chain.url)" />
      </div>

      <!-- Pokemon Dex Entries Info Box -->
      <!-- <div class="pokemon-generation-dex-entries" v-bind:class="'poke-info' + speciesInfo.color.name">
                COMING SOON
            </div> -->
    </div>
  </div>
</template>

<script>
import router from '@/router'
import { RepositoryFactory } from '@/repositories/repositoryFactory'
import Loader from '@/components/Loader'
import TypeEffectiveness from '@/components/pokemon/TypeEffectiveness'
import DexNavigation from '@/components/pokemon/DexNavigation'
import EvolutionChain from '@/components/pokemon/EvolutionChain'

const pokeApi = RepositoryFactory.get('pokeApi')
const util = RepositoryFactory.get('util')

const statMap = {
  hp: {short: 'HP', long: 'HP'},
  attack: {short: 'Atk', long: 'Attack'},
  defense: {short: 'Def', long: 'Defense'},
  special_attack: {short: 'Sp.Atk', long: 'Special Attack'},
  special_defense: {short: 'Sp.Def', long: 'Special Defense'},
  speed: {short: 'Speed', long: 'Speed'}
}

const StatRepo = {
  get: stat => statMap[stat.replace('-','_')]
}

export default {

  name: 'Pokemon',
  components: {
    Loader,
    TypeEffectiveness,
    DexNavigation,
    EvolutionChain
  },
  data () {
    return {
      pokemon: router.currentRoute.params.name,
      form: router.currentRoute.query.form,
      isLoading: true,
      speciesInfo: null,
      pokeInfo: null,
      pokeName: null,
      abilityList: [],
      nextNum: null,
      prevNum: null,
      navigating: false,
      locales: [],
      title: ' - Pokémon'
    }
  },
  mounted () {
    this.fetch()
    this.locales = util.getUserLocales()
  },
  updated () {
    if (this.navigating) {
      this.fetch()
      this.navigating = false
    }
  },
  methods: {
    async fetch () {
      this.isLoading = true

      var { data } = await pokeApi.getPokemonSpecies(this.pokemon)
      this.speciesInfo = data

      var { data } = await pokeApi.getPokemon(this.pokemon)
      this.pokeInfo = data

      var { data } = await pokeApi.getCurrentTotalPokemon()
      this.totalPokemon = data.count

      // Set the prev Pokedex num and next Pokedex num
      if ((this.speciesInfo.id - 1) < 1) this.prevNum = this.totalPokemon
      else this.prevNum = this.speciesInfo.id - 1
      if ((this.speciesInfo.id + 1) > this.totalPokemon) this.nextNum = 1
      else this.nextNum = this.speciesInfo.id + 1

      // Get the details for abilities of Pokemon
      this.abilityList = []
      for (var i = 0; i < this.pokeInfo.abilities.length; i++) {
        var abilityInfo = this.pokeInfo.abilities[i]
        var { data } = await pokeApi.getAbility(abilityInfo.ability.name)
        this.storeAbilityData(data)
      }

      this.pokeName = this.getEntryForLocale(this.speciesInfo.names).name
      document.title = this.pokeName + this.title // set site title to pokemon name

      this.isLoading = false
    },

    toUpper (value) {
      return util.toUpper(value)
    },

    formatText (text) {
      return text.trim().replace('/\s+/', '')
    },

    getPhotoUrl () {
      return util.getPokemonImageUrl(util.findIndex(this.speciesInfo.id))
    },

    findIndex (value) {
      return util.findIndex(value)
    },

    getTyping (types) {
      if (types.length > 1) {
        return util.toUpper(types[0].type.name) + ' & ' + util.toUpper(types[1].type.name)
      } else {
        return util.toUpper(types[0].type.name)
      }
    },

    getGeneration (gen) {
      var split = gen.split('-')
      return util.toUpper(split[0]) + ' ' + split[1].toUpperCase()
    },

    getFlavorText (data) {
      return this.getEntryForLocale(data).flavor_text
    },

    getAlternateForms(varieties, forms) {

      var varieties = varieties.filter(variety => {
        if (item.name != 'unknown' && item.name != 'shadow') return item
      })

      return varieties.concat(forms)
    },

    storeAbilityData (data) {
      var effectEntry = this.getEntryForLocale(data.effect_entries)
      var flavorTextEntry = this.getEntryForLocale(data.flavor_text_entries)

      var shortDesc, longDesc, flavorText

      if (this.checkNull(effectEntry)) {
        shortDesc = null
        longDesc = null
      } else {
        shortDesc = effectEntry.short_effect
        longDesc = effectEntry.effect
      }

      if (this.checkNull(flavorTextEntry)) flavorText = null; else flavorText = flavorTextEntry.flavor_text

      this.abilityList.push({
        name: data.name,
        names: data.names,
        short_desc: shortDesc,
        long_desc: longDesc,
        flavor_text: flavorText
      })
    },

    getAbilityName (info, num) {
      var ability = this.abilityList[num]
      var name = this.getEntryForLocale(ability.names).name
      if (info.is_hidden) {
        return name + ' (hidden)'
      } else {
        return name
      }
    },

    getAbilityDesc (num) {
      if (this.abilityList[num].short_desc != null) {
        return this.abilityList[num].short_desc
      } else {
        return this.abilityList[num].flavor_text
      }
    },

    getSpecies (data) {
      return this.getEntryForLocale(data).genus
    },

    getWeight (data) {
      var weight_metric = data / 10
      var weight_us = Math.round(weight_metric * 2.20462262185)
      return weight_metric + ' kg  |  ' + weight_us + ' lbs'
    },

    getHeight (data) {
      var height_metric = data / 10

      var inches = height_metric * 39.37007874
      var feet = Math.floor(inches / 12)

      var inches_r = Math.round(inches % 12)

      return height_metric + ' m  |  ' + feet + ' ft ' + inches_r + ' in'
    },

    getJapaneseName (data) {
      var japaneseName = ''
      for (var i = 0; i < data.length; i++) {
        var entry = data[i]
        if (entry.language.name == 'ja') {
          japaneseName += entry.name
        }
      }
      for (var i = 0; i < data.length; i++) {
        var entry = data[i]
        if (entry.language.name == 'roomaji') {
          japaneseName += ' (' + entry.name + ')'
        }
      }

      return japaneseName
    },

    getStats (stats) {
      var statArr = [];
      stats.forEach(stat => {
        statArr.push({
          base_stat: stat.base_stat,
          effort: stat.effort,
          stat_names: StatRepo.get(stat.stat.name)
        })
      })
      return statArr
    },

    getGenderRate (rate) {
      if (rate >= 0) {
        var f_rate = (rate / 8) * 100
        var m_rate = 100 - f_rate

        if (f_rate == 100) {
          return '100% Female'
        } else if (m_rate == 100) {
          return '100% Male'
        } else {
          return m_rate + '% Male, ' + f_rate + '% Female'
        }
      } else {
        return 'Genderless'
      }
    },

    calcHatching (data) {
      var egg_walk_amt = data * 256
      return data + ' egg cycles (' + egg_walk_amt + ' steps)'
    },

    getEggGroups (data) {
      var groups = ''

      for (var i = 0; i < data.length; i++) {
        if (i > 0 && i <= data.length) {
          groups += ' & '
        }

        switch (data[i].name) {
          case 'no-eggs':
            groups += 'Undiscovered'
            break
          case 'water1':
            groups += 'Water 1'
            break
          case 'water2':
            groups += 'Water 2'
            break
          case 'water3':
            groups += 'Water 3'
            break
          case 'humanshape':
            groups += 'Human-Like'
            break
          case 'indeterminate':
            groups += 'Amorphous'
            break
          case 'ground':
            groups += 'Field'
            break
          default:
            groups += util.toUpper(data[i].name)
            break
        }
      }

      return groups
    },

    showEasterEgg () {
      var origName = this.pokeName

      switch (origName) {
        case 'Lapras':
          this.pokeName = 'Joy'
          break
        case 'Charjabug':
          this.pokeName = 'Strugglebus'
          break
        case 'Omanyte':
          this.pokeName = 'Lord Helix'
          break
        case 'Farfetch’d':
          this.pokeName = 'Bird Jesus'
          break
      }

      var that = this
      setTimeout(function () {
        that.pokeName = origName
      }, 2000)
    },

    // navigateDex (event) {
    //   this.pokemon = event
    //   this.navigating = true
    // },

    checkNull (data) {
      if (data == null) return true
      else return false
    },

    getId (url) {
      return util.getId(url)
    },

    getEntryForLocale (data) {
      var entry
      // this.locales.forEach(lang => {
      //   data.forEach(item => {
      //     if (item.language.name == lang) {
      //       entry = item;
      //     }
      //   })

      // })

      if (entry == null) {
        data.forEach(item => {
          if (item.language.name == 'en') {
            entry = item
          }
        })
      }

      return entry
    }
  },
  watch: {
    $route: function (to, from) {
      this.pokemon = to.params.name
      this.navigating = true
    }
  }

}
</script>

<style scoped lang="scss">

@import '../styling/types.css';
@import '../styling/colors.css';

.poke-head {
  max-width: 46.875rem;
  margin: 0 auto;
}

.pokemon-image {
  margin-top: 1rem;
  height: 12rem;
}

/* Type styling */
.pokemon-types {
  display: flex;
  flex-direction: row;
  justify-content: center;
}

[class*="type-"] {
  border-radius: 0.625rem;

  padding: 0.2rem 1rem;
  margin: 0 0.5rem;
  min-width: 5.625rem;
  text-align: center;

  /* cursor: pointer; */
}

/* Info styling */

/* Info boxes */
.pokemon-desc {
  text-align: left;
  margin: 0 1rem;
}

.info-box {
  border-radius: 0.625rem;
  padding: 0 1rem;
  margin: 1rem 0;
  text-align: left;
}


/* Pokedex Data */
.poke-abilities {
  cursor: pointer;
}

/* Training info */

.poke-training-box {
  display: flex;
  flex-direction: column;
}

.poke-evs-all {
  display: flex;
  flex-direction: column;
}
.poke-evs-all > h4 {
  margin: 0;
  margin-bottom: 0.5rem;
}

.poke-evs-3 {
  display: flex;
  flex-direction: row;
  justify-content: center;
  flex-wrap: wrap;
  margin: 0 1rem 1rem;
  max-width: 15rem;
}

.poke-evs {
  display: flex;
  flex-direction: column;

  align-items: center;
  justify-content: center;
}

[class*="poke-ev-"] {
  min-width: 4rem;

  border-radius: 0.625rem;

  padding: 0.2rem 0;
  margin: 0.5rem;

  text-align: center;
}
.poke-ev-hp {
  border: 2px solid #ff5959;
  background-color: rgba(255, 89, 89, 0.3);
}
.poke-ev-atk {
  border: 2px solid #f5ac78;
  background-color: rgba(245, 172, 120, 0.3);
}
.poke-ev-def {
  border: 2px solid #fae078;
  background-color: rgba(250, 224, 120, 0.3);
}
.poke-ev-spatk {
  border: 2px solid #65aff7;
  background-color: rgba(101, 175, 247, 0.3);
}
.poke-ev-spdef {
  border: 2px solid #7ddb7d;
  background-color: rgba(125, 219, 125, 0.3);
}
.poke-ev-speed {
  border: 2px solid #f755c1;
  background-color: rgba(247, 85, 193, 0.3);
}

.poke-train-table {
  border-collapse: separate;
  border-spacing: 0 1rem;
}
.poke-train-c1 {
  text-align: right;
}
.poke-train-c2 {
  padding-left: 1rem;
}


/* Base Stats & Breeding Box styling */

.statsAndBreeding {
  display: flex;
  flex-direction: column;
}

/* Base Stats info */
.pokemon-stats > h3 {
  margin-bottom: 0;
}

.base-stats {
  display: flex;
  flex-direction: column;
  margin: 0.5rem 0 1rem;
  border-spacing: 0 0.5rem;
}
.base-stats-c1 {
  text-align: right;
}
.base-stats-c2 {
  padding-left: 1rem;
}

.base-stat-hp {
  color: #ff5959;
}
.base-stat-atk {
  color: #f5ac78;
}
.base-stat-def {
  color: #e7cf6d;
}
.base-stat-spatk {
  color: #65aff7;
}
.base-stat-spdef {
  color: #7ddb7d;
}
.base-stat-speed {
  color: #f755c1;
}

@media screen and (min-width: 25.9375rem) {
  .pokemon-data {
    margin: 0 auto;
    padding: 0;
    max-width: 46.875rem;
    display: flex;
    flex-direction: column;
  }

  .poke-training-box {
    flex-direction: row;
  }

  .pokemon-image {
    height: 16rem;
  }

  .statsAndBreeding {
    flex-direction: row;
    justify-content: space-between;
  }
  .pokemon-stats {
    width: 28%;
  }
  .base-stats {
    text-align: center;
  }
  .pokemon-breeding {
    width: 68%;
  }

  .poke-ab-black:hover { color: #323232;}
  .poke-ab-blue:hover { color: #3482DE;}
  .poke-ab-brown:hover { color: #AF891F;}
  .poke-ab-gray:hover { color: #707070;}
  .poke-ab-green:hover { color: #64A743;}
  .poke-ab-pink:hover { color: #E97698;}
  .poke-ab-purple:hover { color: #7C63B8;}
  .poke-ab-red:hover { color: #EF4036;}
  .poke-ab-white:hover { color: #aaaaaa;}
  .poke-ab-yellow:hover { color: #F8D030;}
}
</style>