<template>
  <div>
    <template>
      <p>Selecione os gêneros de jogos que você costuma jogar:</p>
      <v-container fluid>
        <v-checkbox
          v-for="(genre, index) in min_genres"
          :key="index+genres"
          v-model="genres_selecteds"
          :label="genre"
          color="success"
          :value="genre"
           v-on:click="onClickButton"
        ></v-checkbox>
      </v-container>
    </template>
  </div>
</template>

<script>
import axios from "axios";
import GameTable from './GameTable.vue'

export default {
  name: "Genres",
  data() {
    return {
      genres: [],
      min_genres: ["Action", "MMORPG", "Adventure_game", "Strategy_video_game", "Shooter_video_game"],
      url: "http://dbpedia.org/sparql?query=",
      genres_selecteds: []
    };
  },
  components: {
    "game-table": GameTable,
  },
  computed: {
    getAllGenres: function() {
      let query = [
        "PREFIX db:<http://dbpedia.org/resource/>",
        "SELECT DISTINCT ?game ?genre",
        "WHERE { ?game ?a db:Game; dbo:genre ?genre. }"
      ].join(" ");
      return encodeURIComponent(query);
    },
  },
  mounted() {
    var self = this;
    axios
      .get(this.url + this.getAllGenres)
      .then(function(response) {
        let games = JSON.parse(JSON.stringify(response.data));
        self.cleanGenre(games.results.bindings);
      })
      .catch(function(error) {
        // handle error
      })
      .then(function() {
        // always executed
      });
  },
  methods: {
    cleanGenre: function(games_api) {
      let games_list_genre = new Set();
      let clean_games = [];
      for (let i = 0; i < games_api.length; ++i) {
        if (!games_list_genre.has(games_api[i].genre.value)) {
          games_list_genre.add(games_api[i].genre.value);
          clean_games.push(
            games_api[i].genre.value.replace("http://dbpedia.org/resource/", "")
          );
        }
      }

      this.genres = clean_games;
    },
    onClickButton (event) {
      var self = this
      this.$emit('clicked', self.genres_selecteds)
    }
  }
};
</script>

<style>
</style>
