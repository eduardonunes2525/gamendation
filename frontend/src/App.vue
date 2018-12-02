<template>
  <v-app>
    <v-stepper v-model="e1">
      <v-stepper-header>
        <v-stepper-step :complete="e1 > 1" step="1">Escolha seus gêneros preferidos</v-stepper-step>
        <v-divider></v-divider>
        <!-- <v-stepper-step :complete="e1 > 2" step="2">Escolha seus modos preferidos</v-stepper-step>
        <v-divider></v-divider> -->
        <v-stepper-step step="3">Veja os jogos que encaixam se com seu perfil</v-stepper-step>
      </v-stepper-header>

      <v-stepper-items>
        <v-stepper-content step="1">
          <table-test @clicked="onClickChild"></table-test>
          <v-btn color="primary" v-on:click="getData" @click="e1 = 3">Continue</v-btn>
        </v-stepper-content>

        <!-- <v-stepper-content step="2">
          <v-card class="mb-5" color="grey lighten-1" height="200px"></v-card>

          <v-btn color="primary" @click="e1 = 3">Continue</v-btn>

          <v-btn flat @click="e1 = 1">Voltar</v-btn>
        </v-stepper-content> -->

        <v-stepper-content step="3">
          <game-table :games="games"/>

          <v-btn flat @click="e1 = 1">Voltar</v-btn>
        </v-stepper-content>
      </v-stepper-items>
    </v-stepper>
  </v-app>
</template>

<script>
import GameTable from "./components/GameTable.vue";
import Table from "./components/Table.vue";
import axios from "axios";

export default {
  name: "app",
  components: {
    "game-table": GameTable,
    "table-test": Table
  },
  data() {
    return {
      e1: 0,
      genres: [],
      games: [],
      url: 'http://dbpedia.org/sparql?query=',
    };
  },
  computed: {
    getQuery: function(){
      let query = [
        'PREFIX db:<http://dbpedia.org/resource/>',
        'SELECT ?game ?genre ?name ?link ?award ?abstract',
        'WHERE { ?game ?a db:Game; foaf:name ?name; foaf:homepage ?link; dbo:genre ?genre; dbo:abstract ?abstract. ',
        'OPTIONAL {?game dbp:award ?award} FILTER ( lang(?abstract)="pt" ',
        this.getAllGenres,
        '))}'
      ].join(" ");
      return encodeURIComponent(query);
    },
    getAllGenres: function(){
      var str = ""
      for(let i = 0; i < this.genres.length; ++i){
        (i>0) ? str+="||" : str+="&& ("
        str+=('regex(?genre, "'+ this.genres[i] + '","i") ')
      }
      return str;
    }
  },
  methods: {
    onClickChild (value) {
      this.genres = value
    },
    getData: function() {
      var self = this;
      axios
      .get(this.url + this.getQuery)
      .then(function(response) {
        let games = JSON.parse(JSON.stringify(response.data));
        self.cleanGames(games.results.bindings)
      })
      .catch(function(error) {
        // handle error
      })
      .then(function() {
        // always executed
      });
    },
    cleanGames: function(games_api){
        let games_list = new Set();
        let games_list_abstract = new Set();
        let clean_games = [];

        for(let i = 0; i < games_api.length; ++i){
            if(!games_list.has(games_api[i].name.value) && !games_list_abstract.has(games_api[i].abstract.value)){
                games_list.add(games_api[i].name.value)
                games_list_abstract.add(games_api[i].abstract.value)
                clean_games.push(games_api[i]);
            }
        }

        this.games = clean_games;
    }
  }
};
</script>

<style>
</style>
