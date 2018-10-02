<template>
  <div class="results" v-if="night">
    <div v-if="!night.votes">
      <section>
        <h1>Aucun votes pour la Game Night du {{ night.date.toDate() | formatDate }} pour le moment</h1>
      </section>
    </div>
    <div v-else>
      <section>
        <h1>Résultats des votes pour la Game Night du {{ night.date.toDate() | formatDate }}</h1>

        <div class="results-container">
          <Results ref="chart" :chart-data="chartData" />
        </div>
      </section>
      <section>
        <div class="results-container">
          <b-table striped hover :items="items" :fields="fields" sortBy="score" :sortDesc="true" bordered>
            <template slot="name" slot-scope="data">
              <a href="javascript:void(0)" v-b-modal="'modal-' + data.item.id">{{ data.item.name }}</a>
            </template>
          </b-table>
        </div>
      </section>
    </div>
  </div>
</template>

<script>
// @ is an alias to /src
import Vue from 'vue'
import Game from '@/components/Game.vue'
import Results from '@/components/Results.vue'
import Firebase from '@/services/Firebase.vue'
import Moment from 'moment'

Vue.filter('formatDate', (value) => {
  if (value) {
    return Moment(String(value), "ddd MMM DD YYYY HH:mm:ss ZZ").format('DD/MM/YYYY')
  }
});

export default {
  name: 'results',
  components: {
    Game,
    Results
  },
  data: function() {
    return {
      night: null,
      games: [],
      items: [],
      fields:[
        { key: 'name', label: "Jeu", sortable: true },
        { key: 'score', label: "Score", sortable: true }
      ],
      chartData: null
    };
  },
  created () {
    // fetch the data when the view is created and the data is
    // already being observed
    this.fetchData();
  },
  watch: {
    // call again the method if the route changes
    '$route': 'fetchData'
  },
  methods: {
    fetchData () {

      var id = this.$route.params.id;

      var dynamicColors = function() {
        var r = Math.floor(Math.random() * 255);
        var g = Math.floor(Math.random() * 255);
        var b = Math.floor(Math.random() * 255);
        return "rgb(" + r + "," + g + "," + b + ")";
      };

      Firebase.db.collection("nights").doc(id).get().then((doc) => {
        this.night = doc.data();

        this.chartData = {
          labels: [],
          datasets: [{
            label: "Scores",
            borderColor: "rgba(200, 200, 200, 0.75)",
            hoverBorderColor: "rgba(200, 200, 200, 1)",
            backgroundColor: [],
            data: []
          }]
        };

        let votedGames = [];
        if (this.night.votes) {
          for (let i = 0; i < this.night.votes.length; i++) {
            let vote = this.night.votes[i];
            for (let j = 0; j < vote.scores.length; j++) {
              let score = vote.scores[j];
              score.game.get().then((doc) => {
                let game = doc.data();
                game.id = doc.id;
                
                this.games.push(game);

                if (score.score > 0) {
                  if (this.chartData.labels.indexOf(game.name) === -1) {
                    this.chartData.labels.push(game.name);
                    this.chartData.datasets[0].data.push(0);
                    this.chartData.datasets[0].backgroundColor.push(dynamicColors());
                  }
                  
                  let index = this.chartData.labels.indexOf(game.name);
                  this.chartData.datasets[0].data[index] += score.score;
                  this.$refs.chart.update();
                }

                if (votedGames.indexOf(game.id) === -1) {
                  votedGames.push(game.id);
                  this.items.push({
                    id: game.id,
                    name: game.name,
                    score: 0
                  });
                }

                let index = votedGames.indexOf(game.id);
                this.items[index].score += score.score;
              });
            }
          }
        }
      });
    }
  }
}
</script>

<style lang="scss">
section {
    margin: 2rem 0;
}

.results-container {
    max-width: 40rem;
    margin: auto;
}
</style>
