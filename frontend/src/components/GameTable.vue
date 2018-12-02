<template>
  <div>
    <center>Esses são os jogos que mais se parecem com o seu estilo :) Clique nos jogos para obter mais informações!</center>
    <v-data-table v-if="games.length!=0" :headers="headers" :items="games" item-key="name.value">
      <template slot="items" slot-scope="props">
        <tr @click="props.expanded = !props.expanded">
          <td> <a :href="props.item.link.value">{{ props.item.name.value }} </a></td>
        </tr>
      </template>
      <template slot="expand" slot-scope="props">
        <v-card flat>
          <v-card-text class="information"><strong>Gênero:</strong> {{ (props.item.genre.value).replace("http://dbpedia.org/resource/", "") }}</v-card-text>
          <v-card-text class="information"><strong>Prêmios:</strong> {{ (props.item.award.value).replace("http://dbpedia.org/resource/", "")}}</v-card-text>
          <v-card-text class="information">{{ (props.item.abstract.value)}}</v-card-text>
          <!-- <v-card-text class="information">{{ (props.item) }}</v-card-text> -->
        </v-card>
      </template>
    </v-data-table>
    <center v-else><v-progress-circular
      :size="70"
      :width="7"
      color="orange"
      indeterminate
    ></v-progress-circular></center>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: "Table",
  data() {
    return {
      headers: [
        {
          text: "Nome",
          align: "left",
          sortable: false,
          value: "name"
        }
      ],
      url: 'http://dbpedia.org/sparql?query=',
      genre_selected: "Action",
    };
  },
  props: { games: Array },
};
</script>

<style>
.information {
  padding: 10px 80px;
  background: gray;
  color: white;
}
</style>
