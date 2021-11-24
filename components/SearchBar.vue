<template>
  <div class="relative flex items-top justify-center min-h-screen bg-gray-100 sm:items-center sm:pt-0">
    <v-autocomplete
      v-model="selected"
      :items="items"
      :loading="isLoading"
      :search-input.sync="search"
      color="white"
      item-text="description"
      item-value="label"
      label="WikiData Items"
      placeholder="Start typing to Search"
      prepend-icon="mdi-database-search"
      no-filter
      return-object>
      <template v-slot:selection="data">
        <div>{{ data.item.label }}</div>
      </template>

      <template 
        cols="4"
        v-slot:item="data">
        <template v-if="typeof data.item==='object'">
          <v-list-item-content>
            <v-list-item-title>{{ truncate(data.item.label) }}</v-list-item-title>
            <v-list-item-subtitle>{{ truncate(data.item.description) }}</v-list-item-subtitle>
          </v-list-item-content>
        </template>
      </template>

      <template 
        v-slot:append-item>
        <div>
          <div 
            v-if="(items || []).length && searchContinue!==undefined"
            class="d-flex flex-row-reverse">
            <v-fab-transition>
              <v-btn
                @click.prevent.stop="fetchData(search, true)"
                class="mr-2"
                color="pink"
                fab
                dark
                small
                right
              >
                <v-icon>mdi-plus</v-icon>
              </v-btn>
            </v-fab-transition>  
          </div>
          <div
            class="pa-3"
            v-if="(selected || {}).label || search">
            <a :href="`https://www.wikidata.org/w/index.php?search=${search}&title=Special%3ASearch&fulltext=1&ns0=1&ns120=1`">Search for pages containing <em>{{ truncate((selected || {}).label) || search }}</em></a>
          </div>
        </div>
      </template>
    </v-autocomplete>
  </div>
</template>

<script>
  export default {
    data: () => ({
      items: [],
      isLoading: false,
      model: {},
      search: null,
      searchContinue: 0,
      fieldLength: 60,
    }),
    computed: {
      selected: {
        set (val) {
          this.model=val
          this.$emit('selected', val)
        },
        get () {
          return this.model
        },
      },
    },
    methods: {
      // Truncate strings and keep the search options window to a manageable length
      truncate(str) {
        if (!str) return ''
        return (str.length > this.fieldLength) ? str.substr(0, this.fieldLength-1) + '...' : str;
      },

      fetchData (val, add) {
        // If there is no search term, reset query params
        if (!val.trim()) {
          this.items=[]
          this.searchContinue=0
          return
        } 
        // Items have already been requested
        this.isLoading = true

        const cont = this.searchContinue || 0
        // Lazily load input items
        fetch(`https://www.wikidata.org/w/api.php?action=wbsearchentities&uselang=en&language=en&format=json&origin=*&continue=${cont}&search=${val.trim()}`)
          .then(res => res.json())
          .then(res => {
            // This sets the value for further results. Will return undefined if the list is exhausted
            this.searchContinue = res['search-continue']
            // If this is a new search, replace the existing items, but if it's adding to the existing results, then concat
            this.items = add ? this.items.concat(res.search) : res.search
          })
          .catch(err => {
            console.log(err)
          })
          .finally(() => (this.isLoading = false))

      },
    },
    watch: {
      search (val) {
        this.fetchData(val)
      },
    },
  }
</script>
