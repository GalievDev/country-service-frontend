<script lang="ts">
import { useQuery } from "@tanstack/vue-query";
import { Countries } from "../dto/Countries.ts";
import {defineComponent, ref} from "vue";
import Country from "./Country.vue";
const fetcherCountries = async (): Promise<Countries> =>
    await fetch('http://127.0.0.1:8080/countries/').then(
        (response) => response.json(),
    )

function useCountriesQuery() {
  return useQuery({ queryKey: ["countries"], queryFn: fetcherCountries })
}

export default defineComponent({
  components: {Country},
  setup() {
    let selectedCountry = ref('')
    const dialog = ref(false)
    if (!dialog){ selectedCountry = ref('') }
    const { isLoading: countriesLoading, isError: countriesError, error: countriesErrorData, data: countriesData } = useCountriesQuery()

    return { countriesLoading, countriesError, countriesErrorData, countriesData, dialog, selectedCountry }
  },
})

</script>


<template>
  <v-alert v-if="countriesLoading" class="text-center" type="info" color="green" density="compact" text="Loading... Please wait." variant="tonal"></v-alert>
  <v-alert v-else-if="countriesError" class="text-center" type="error" density="compact" text="Error: " variant="tonal">An error has occurred {{ countriesErrorData }} </v-alert>
  <v-container v-else-if="countriesData">
    <v-row class="justify-center">
      <v-col
        xs="1"
        sm="6"
        md="5"
        lg="3"
        xl="3"
        v-for="(countries, index) in countriesData.countries" :key="index">
        <v-container>
          <v-card @click="dialog = true; selectedCountry = countries.name">
            <v-card-title>
              {{ countries.name }}
            </v-card-title>
            <v-card-text>
              {{ countries.country_code }}
            </v-card-text>
          </v-card>
        </v-container>
      </v-col>
      <Country :is-open="dialog" :name="selectedCountry" />
    </v-row>
  </v-container>
</template>