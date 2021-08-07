<template>

  <div>
    <h2>Language Picker using browsers built-in translation.</h2>
    <div class="select">
      Select Language
      <select v-model="selectedLanguage" v-on:change="persistChange">
        <option v-for="(language) in languages" :key="language.code" :value="language.code">
          {{language.name}}
        </option>
      </select>
    </div>
    <div class="select">
      Select Region
      <select v-model="selectedRegion" v-on:change="persistChange">
        <option v-for="(region) in regions" :key="region.code" :value="region.code">
          {{region.name}}
        </option>
      </select>
    </div>
    <div class="country-flag" :style="{ backgroundImage: `url(https://www.countryflags.io/${selectedRegion}/flat/64.png)` }">
    </div>
  </div>


</template>

<script>
let languageCodes = ['en', 'zh', 'fr', 'de']
let defaultLanguage = 'en-US'
let preferredLanguage = defaultLanguage
try {
  preferredLanguage = navigator.language
  let lang = localStorage.getItem('selectedLanguage')
  if (lang !== null) {
    preferredLanguage = lang
  }
} catch(e) {}

if (!languageCodes.includes(preferredLanguage)) {
  let parts = preferredLanguage.split('-')
  if (languageCodes.includes(parts[0])) {
    preferredLanguage = parts[0]
  } else {
    preferredLanguage = defaultLanguage
  }
}

// TODO: magical function that guesses country by language, time or IP
let regionCodes = ['ch', 'at', 'de', 'jp', 'gb', 'cn', 'tw', 'us', 'lu']
let defaultRegion = 'us'
let preferredRegion = defaultRegion
let queryRegion = true

try {
  let region = localStorage.getItem('selectedRegion')
  if (region !== null) {
    preferredRegion = region
    queryRegion = false
  }
} catch(e) {}

export default {
  name: 'LanguagePicker',
  data () {
    return {
      selectedRegion: preferredRegion,
      selectedLanguage: preferredLanguage,
      languages: [],
      regions: [],
    }
  },
  mounted() {
    if (queryRegion) {
      fetch('https://ipinfo.io/country?token=heheehe')
      .then(response => response.text())
      .then(data => {
        let region = data.trim().toLowerCase()
        if (regionCodes.includes(region)) {
          this.selectedRegion = data.trim().toLowerCase()
          this.update()
        }
      })
    }
    this.update()
  },
  methods: {
    persistChange() {
      localStorage.setItem('selectedLanguage', this.selectedLanguage)
      localStorage.setItem('selectedRegion', this.selectedRegion)
      this.update()
    },
    update() {
      let options = {type: 'language'}
      let displayNames = new Intl.DisplayNames(this.selectedLanguage, options)
      let result = []
      for (let code of languageCodes) {
        result.push({code: code, name: displayNames.of(code)})
      }
      this.languages = result

      {
        let options = {type: 'region'}
        let displayNames = new Intl.DisplayNames(this.selectedLanguage, options)
        let result = []
        for (let code of regionCodes) {
          result.push({code: code, name: displayNames.of(code)})
        }
        this.regions = result
      }
    }
  }
}

</script>

<style scoped>
a {
  color: #42b983;
}

.country-flag {
  margin-top: 5px;
  display: inline-block;
  vertical-align: middle;
  border-radius: 50%;
  color: transparent;
  background-size: 80px;
  background-position: 50%;
  background-repeat: no-repeat;
  width: 50px;
  height: 50px;
}

</style>
