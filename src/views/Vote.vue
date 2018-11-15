<template>
  <div class="vote">
    <section v-if="night">
      <h1>Voter pour la Game Night du {{ night.date.toDate() | formatDate }}</h1>
      <small>Seuls les jeux pouvants être joués à {{ night.numberOfPlayers }} sont affichés</small>

      <div class="vote-container">
        <b-form @submit="onSubmit">
          <b-table striped hover :items="items" :fields="fields" bordered responsive>
            <template slot="name" slot-scope="data">
              <a href="javascript:void(0)" v-b-modal="'modal-' + data.item.id">{{ data.item.name }}</a>
              <br v-if="data.item.owners.length > 0" />
              <b-badge pill v-for="(owner, index) in data.item.owners" :key="index">{{ owner }}</b-badge>
            </template>
            <template slot="score" slot-scope="data">
              <b-form-radio-group
                v-model="form.score[data.item.id]"
                buttons button-variant="outline-primary" size="sm"
                :id="data.item.id"
                :name="data.item.id"
                :options="data.item.score"
              >
              </b-form-radio-group>
            </template>
          </b-table>
          <b-button type="submit" variant="primary">Voter !</b-button>
        </b-form>
      </div>
    </section>

    <!-- Modal Component -->
    <b-modal v-for="(game, index) in games" :key="index" :game="game" :id="'modal-' + game.id" :title="game.name" hide-header ok-only lazy>
      <Game :game="game" />
    </b-modal>
  </div>
</template>

<script>
// @ is an alias to /src
import Vue from 'vue'
import Game from '@/components/Game.vue'
import Firebase from '@/services/Firebase.vue'
import Moment from 'moment'

Vue.filter('formatDate', (value) => {
  if (value) {
    return Moment(String(value), "ddd MMM DD YYYY HH:mm:ss ZZ").format('DD/MM/YYYY')
  }
});

export default {
  name: 'vote',
  components: {
    Game
  },
  data: function() {
    return {
      night: null,
      games: [],
      items: [],
      fields:[
        { key: 'name', label: "Jeu" },
        'score'
      ],
      form: {
        score: []
      }
    };
  },
  created () {
    // fetch the data when the view is created and the data is
    // already being observed
    this.fetchData()
  },
  watch: {
    // call again the method if the route changes
    '$route': 'fetchData'
  },
  methods: {
    onSubmit (evt) {
      evt.preventDefault();

      var id = this.$route.params.id;
      var vote = {
        date: new Date(),
        scores: []
      }
      
      for (let i = 0; i < this.items.length; i++) {
        vote.scores.push({
          game: Firebase.db.collection("games").doc(this.items[i].id),
          score: this.form.score[this.items[i].id] ? this.form.score[this.items[i].id] : 0
        });
      }

      Firebase.db.collection("nights").doc(id).update({
        votes: Firebase.const.FieldValue.arrayUnion(vote)
      })
      .then(() => {
        this.$router.push({ name: 'results', params: { id: this.$route.params.id }})
      })
      .catch((error) => {
        console.error("Error updating document: ", error);
        alert("Error updating document: ".error);
      });
    },
    fetchData () {

      var id = this.$route.params.id;

      var results = [];
      var resultsNames = {};
      var resultsMin = [];
      var resultsMax = [];
      var resultsTeam = [];

      Firebase.db.collection("nights").doc(id).get().then((doc) => {
        this.night = doc.data();

        Firebase.db.collection("games").where("minPlayers", "<=", this.night.numberOfPlayers).get().then((querySnapshot) => {
          for (let i = 0; i < querySnapshot.docs.length; i++) {
            resultsMin.push(querySnapshot.docs[i].id);
            resultsNames[querySnapshot.docs[i].id] = querySnapshot.docs[i].data().name;
          }
          Firebase.db.collection("games").where("maxPlayers", ">=", this.night.numberOfPlayers).get().then((querySnapshot) => {
            for (let i = 0; i < querySnapshot.docs.length; i++) {
              resultsMax.push(querySnapshot.docs[i].id);
              resultsNames[querySnapshot.docs[i].id] = querySnapshot.docs[i].data().name;
            }
            Firebase.db.collection("games").where("isPlayableInTeams", "==", true).get().then((querySnapshot) => {
              for (let i = 0; i < querySnapshot.docs.length; i++) {
                resultsTeam.push(querySnapshot.docs[i].id);
                resultsNames[querySnapshot.docs[i].id] = querySnapshot.docs[i].data().name;
              }

              // Union
              results = resultsMax;
              for (let i = 0; i < resultsTeam.length; i++) {
                if (results.indexOf(resultsTeam[i]) === -1) {
                  results.push(resultsTeam[i]);
                }
              }

              // Intersect
              results = results.filter(value => -1 !== resultsMin.indexOf(value));

              results = results.sort((a, b) => {
                return resultsNames[a].toLowerCase().localeCompare(resultsNames[b].toLowerCase());
              });

              for (let i = 0; i < results.length; i++) {
                Firebase.db.collection("games").doc(results[i]).get().then((doc) => {
                  let game = doc.data();
                  game.id = doc.id;
                  this.games.push(game);
                  this.items.push({
                    id: game.id,
                    name: game.name,
                    owners: game.owners ? game.owners : [],
                    score: [
                      { text: "1⭐", value: 1 },
                      { text: "2⭐", value: 2 },
                      { text: "3⭐", value: 3 },
                      { text: "4⭐", value: 4 },
                      { text: "5⭐", value: 5 }
                    ]
                  });
                });
              }
            });
          });
        });
      });
    }
  }
}
</script>

<style lang="scss">
section {
  margin: 2rem 0;
}

.vote-container {
  max-width: 40rem;
  margin: auto;
}

td {
  vertical-align: middle !important;
}
</style>
