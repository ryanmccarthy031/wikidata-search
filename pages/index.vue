<template>
  <v-row justify="center" align="center">
    <v-col cols="12" sm="8" md="6">
      <v-card>
        <v-card-title class="headline">
          Enter a search term
        </v-card-title>
        <v-card-text>
          <SearchBar 
            @selected="value=$event"/>
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn
            color="primary"
            :disabled="!value"
            nuxt
            target="_blank"
            :href="value ? value.url : '#'"
          >
            <v-icon>mdi-wikipedia</v-icon>
            Learn More
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-col>
  </v-row>
</template>

<script>
import SearchBar from '@/components/SearchBar.vue'

export default {
  data () {
    return {
      value: '',
    }
  },
  components: {
    SearchBar,
  },
}
</script>
