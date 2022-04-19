<template>
  <b-container>
    <b-row>
      <b-col md="12">
        <b-table striped hover :items="pokemonList">
          <template #cell(image)="data">
            <img :src="data.item.image" alt="" />
          </template>

          <template #cell(type)="data">
            <p v-for="(type, index) in data.item.type" :key="index" class="badge bg-secondary">{{ type }}</p>
          </template>
        </b-table>
      </b-col>
    </b-row>
  </b-container>
</template>

<script>
// // @ is an alias to /src
// import HelloWorld from '@/components/HelloWorld.vue'
import { http } from "../utils/axios";

export default {
  name: "Home",
  data() {
    return {
      pokemonList: [],
      colors: {
        "poison": "#2312Fsa"
      }
    };
  },
  methods: {
    async getItems() {
      try {
        const response = await http.get("/pokemon?limit=10");

        const pokemonRawList = response.data.results;

        const fullList = await Promise.all(
          pokemonRawList.map((pokemonInfo) => {
            return http.get(`/pokemon/${pokemonInfo.name}`);
          })
        );

        this.pokemonList = fullList
          .map((el) => el.data)
          .map((el) => ({
            image: el.sprites.front_default,
            name: el.name,
            type: el.types.map((type) => type.type.name),
          }));
      } catch (error) {
        console.log(error);
      }
    },
  },
  created() {
    this.getItems();
  },
};
</script>
