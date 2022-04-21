<template>
  <b-container>
    <b-row>
      <b-col md="12">
        <b-card
          :title="pokemonData.name"
          :img-src="pokemonData.image"
          img-alt="Image"
          img-top
          tag="article"
          class="mb-2 w-25"
        >
          <b-card-text>
            {{ pokemonData.orden }}
          </b-card-text>

          <b-card-text>
            {{ pokemonData.description }}
          </b-card-text>

          <b-list-group>
            <b-list-group-item
              v-for="ability in pokemonData.abilities"
              :key="ability.name"
              >{{ ability.name }} - {{ ability.description }}</b-list-group-item
            >
          </b-list-group>

          <br />

          <b-list-group>
            <b-list-group-item
              v-for="stat in pokemonData.stats"
              :key="stat.name"
              >{{ stat.name }} - {{ stat.value }}</b-list-group-item
            >
          </b-list-group>

          <br />

          <b-list-group>
            <b-list-group-item
              v-for="type in pokemonData.types"
              :key="type.name"
              >{{ type.name }}</b-list-group-item
            >
          </b-list-group>
        </b-card>
      </b-col>
    </b-row>
  </b-container>
</template>

<script>
import { http, httpWithoutUrl } from "../utils/axios";

export default {
  data() {
    return {
      pokemonData: [],
    };
  },
  methods: {
    async makeAbilities(rawData) {
      const resp = await Promise.all(
        rawData.map(async (raw) => {
          return await httpWithoutUrl.get(raw.ability.url);
        })
      );

      const abilityData = resp.map((el) => el.data);

      return abilityData.map((data) => {
        return {
          name: data.names.find((name) => name.language.name === "es").name,
          description: data.flavor_text_entries.find(
            (flavor) => flavor.language.name === "es"
          ).flavor_text,
        };
      });
    },
    async makeStats(stats) {
      const resp = await Promise.all(
        stats.map(async (stat) => {
          return await http.get(`/stat/${stat.stat.name}`);
        })
      );

      const statData = resp.map((el) => el.data);

      return statData.map((data, index) => {
        return {
          name: data.names.find((name) => name.language.name === "es").name,
          value: stats[index].base_stat,
        };
      });
    },
    async makeTypes(types) {
      const resp = await Promise.all(
        types.map(async (type) => {
          return await httpWithoutUrl.get(type.type.url);
        })
      );

      const typeData = resp.map((el) => el.data);

      return typeData.map((data) => {
        return {
          name: data.names.find((name) => name.language.name === "es").name,
        };
      });
    },
  },
  async created() {
    const [pokemonResp, characteristicResp] = await Promise.all([
      http.get(`/pokemon/${this.$route.params.id}`),
      await http.get(`/characteristic/${this.$route.params.id}`),
    ]);

    const pokemonRawData = pokemonResp.data;
    const abilities = await this.makeAbilities(pokemonRawData.abilities);
    const stats = await this.makeStats(pokemonRawData.stats);
    const types = await this.makeTypes(pokemonRawData.types);

    this.pokemonData = {
      id: pokemonRawData.id,
      orden: pokemonRawData.order,
      name: pokemonRawData.name,
      image: pokemonRawData.sprites.front_default,
      abilities,
      exp: pokemonRawData.base_experience,
      stats,
      types,
      description: characteristicResp.data.descriptions.find(
        (desc) => desc.language.name === "es"
      ).description,
    };
  },
};
</script>

<style></style>
