<template>
  <div>
    <h1>Tabela todos os jogos.</h1>
    <button v-on:click="getData">Buscar</button>
    <table v-if="games.length!=0">
        <tr>
            <th>Nome</th>
            <th>Gênero</th>
            <!--<th>Resumo</th>-->
        </tr>
        <tr v-for="(game,index) in games" :key="index">
            <td>{{(game.name.value)}}</td>
            <td>{{(game.genre.value).replace("http://dbpedia.org/resource/", "")}}</td>
            <!--<td>{{(game.abstract.value)}}</td>-->
        </tr>
        </table>
  </div>
</template>

<script>
import axios from 'axios';
export default {
  name: 'GameTable',
  data(){
    return {
        games: [],
        url: 'http://dbpedia.org/sparql?query=',
        genre_selected: "Action"
    }
  },
  computed: {
    getQuery: function(){
      let query = [
        'PREFIX db:<http://dbpedia.org/resource/>',
        'SELECT ?game ?genre ?name ?link ?award ?abstract',
        'WHERE { ?game ?a db:Game; foaf:name ?name; foaf:homepage ?link; dbo:genre ?genre; dbo:abstract ?abstract. OPTIONAL {?game dbp:award ?award} FILTER ( lang(?abstract)="en" && regex(?genre, ',
        '"'+this.genre_selected+'"',
        ',"i" ))}'
      ].join(" ");
      return encodeURIComponent(query);
    }
  },
  methods: {
    getData: function() {
      var self = this;
      axios
      .get(this.url + this.getQuery)
      .then(function(response) {
        let games = JSON.parse(JSON.stringify(response.data));
        self.cleanData(games.results.bindings)
      })
      .catch(function(error) {
        // handle error
      })
      .then(function() {
        // always executed
      });
    },
    cleanData: function(games_api){
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
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
