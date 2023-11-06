<script lang="ts">
import { useQuery } from "@tanstack/vue-query";
import { Countries } from "../dto/Countries.ts";
import { defineComponent, ref } from "vue";
import Country from "../dto/Country.ts";
import countries from '../dto/countries.json';

const fetcherCountries = async (): Promise<Countries> =>
    await fetch("http://127.0.0.1:8080/countries/").then((response) =>
        response.json()
    );

const fetcherCountry = async (name: string): Promise<Country> =>
    await fetch(`http://127.0.0.1:8080/countries/${name}`).then((response) =>
        response.json()
    );

function useCountriesQuery() {
  return useQuery({ queryKey: ["countries"], queryFn: fetcherCountries });
}

export default defineComponent({
  setup() {
    const selectedCountry = ref<string>("");
    const dialog = ref<boolean>(false);
    const {
      isLoading: countriesLoading,
      isError: countriesError,
      error: countriesErrorData,
      data: countriesData,
    } = useCountriesQuery();
    const {
      isLoading: countryLoading,
      isError: countryError,
      error: countryErrorData,
      data: countryData,
      refetch,
    } = useQuery({
      queryKey: ["country", selectedCountry.value],
      queryFn: async () => await fetcherCountry(selectedCountry.value),
    });

    const closeDialog = () => {
      dialog.value = false;
    };

    const openDialog = async (country: Country) => {
      selectedCountry.value = country.name;
      dialog.value = true;
      await refetch();
    };

    return {
      countriesLoading,
      countriesError,
      countriesErrorData,
      countriesData,
      dialog,
      countries,
      selectedCountry,
      countryLoading,
      countryError,
      countryErrorData,
      countryData,
      refetch,
      closeDialog,
      openDialog,
    };
  },
});
</script>

<template>
  <v-alert v-if="countriesLoading" class="text-center" type="info" color="green" density="compact" text="Loading... Please wait." variant="tonal"></v-alert>
  <v-alert v-else-if="countriesError" class="text-center" type="error" density="compact" variant="tonal">
    {{ countriesErrorData }}
    <v-container>
      <v-row class="justify-center">
        <v-col
            xs="1"
            sm="6"
            md="5"
            lg="3"
            xl="3"
            v-for="(country, index) in countries.countries"
            :key="index"
        >
          <v-container>
            <v-card>
              <v-img :src="country.flag_file_url"></v-img>
              <v-card-title>{{ country.name }}</v-card-title>
              <v-card-text>
                <p> Country code: {{ country.country_code }} </p>
                <p> Capital: {{ country.capital }} </p>
                <p> Population: {{ country.population }} </p>
              </v-card-text>
            </v-card>
          </v-container>
        </v-col>
      </v-row>
    </v-container>
    Offline (demo) data
  </v-alert>
  <v-container v-else-if="countriesData">
    <v-dialog v-model="dialog" class="w-33 h-auto">
      <v-progress-circular v-if="countryLoading"></v-progress-circular>
      <v-card v-else-if="countryError">
        <v-alert>{{ countryErrorData }}</v-alert>
      </v-card>
      <v-card v-else-if="countryData" >
        <v-img :src="countryData.flag_file_url" width="auto" height="auto"></v-img>
        <v-card-title>{{ countryData.name }}</v-card-title>
        <v-card-text>
          <p>Country code: {{ countryData.country_code }}</p>
          <p>Capital: {{ countryData.capital }}</p>
          <p>Population: {{ countryData.population }}</p>
        </v-card-text>
        <v-btn @click="closeDialog" variant="tonal" color="red">Close</v-btn>
      </v-card>
    </v-dialog>
    <v-row class="justify-center">
      <v-col
          xs="1"
          sm="6"
          md="5"
          lg="3"
          xl="3"
          v-for="(country, index) in countriesData.countries"
          :key="index"
      >
        <v-container>
          <v-card @click=" async () =>{ await openDialog(country) }">
            <v-card-title>{{ country.name }}</v-card-title>
            <v-card-text>{{ country.country_code }}</v-card-text>
          </v-card>
        </v-container>
      </v-col>
    </v-row>
  </v-container>
</template>
