<script lang="ts">
import {defineComponent, ref} from "vue";
import {useQuery} from "@tanstack/vue-query";
import Country from "../dto/Country.ts";

const fetcherCountry = async (name: string): Promise<Country> =>
    await fetch(`http://127.0.0.1:8080/countries/${name}`).then(
        (response) => response.json(),
    )

export default defineComponent({
  name: "Country",
  props: {
    name: {
      type: String,
      required: true
    },
    isOpen: {
      type: Boolean,
      required: true
    }
  },

  setup(props) {
    const dialog = ref(props.isOpen)
    const { isLoading: countryLoading, isError: countryError, error: countryErrorData, data: countryData } = useQuery({
      queryKey: ["country", props.name],
      queryFn: () => fetcherCountry(props.name)
  })
    console.log(countryData)

    return{ countryLoading, countryError, countryErrorData, countryData, dialog }
  },
})
</script>

<template>
  <v-dialog v-model="dialog" activator="parent" class="w-25 h-25">
    <v-progress-circular v-if="countryLoading"></v-progress-circular>
    <v-dialog v-else-if="countryError">
      <v-alert>
        {{ countryErrorData }}
      </v-alert>
    </v-dialog>
    <v-card v-else-if="countryData" >
      <v-img :src="countryData.flag_file_url" cover></v-img>
      <v-card-title>
        {{ countryData.name }}
      </v-card-title>
      <v-card-text>
        <p>Country code: {{ countryData.country_code }}</p>
        <p>Capital: {{ countryData.capital }}</p>
        <p>Population: {{ countryData.population }}</p>
      </v-card-text>
      <v-btn @click="dialog = false" variant="tonal" color="red">Close</v-btn>
    </v-card>
  </v-dialog>
</template>