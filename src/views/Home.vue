<template>
  <div class="home">
    <section v-if="nights.length > 0">
      <h1>Liste des Game Nights prévues</h1>
      <div class="nights">
        <b-card-group deck class="game-list">
          <div class="card game" v-for="(night, index) in nights" :key="index">
            <div class="card-body">
              <b-button class="float-left" v-b-modal.deleteNight @click="selectedNight = night"><font-awesome-icon icon="trash" /></b-button>
              <b-button class="float-right" v-b-modal.editNight @click="selectedNight = night"><font-awesome-icon icon="edit" /></b-button>
              <h5 class="card-title">{{ night.date.toDate() | formatDate }}</h5>
              <p class="card-text">Nombre de joueurs prévus : {{ night.numberOfPlayers }}</p>
            </div>
            <div class="card-footer">
              <router-link class="float-left" :to="{ name: 'vote', params: { id: night.id }}"><small>Voter !</small></router-link>
              <router-link class="float-right" :to="{ name: 'results', params: { id: night.id }}"><small>Voir les résultats</small></router-link>
            </div>
          </div>
        </b-card-group>
      </div>
    </section>

    <hr v-if="nights.length > 0" />

    <section>
      <h1>Liste des jeux</h1>
      <b-tabs lazy pills>
        <b-tab title="Vue en vignettes" title-item-class="mx-auto" active>
          <b-card-group deck class="game-list">
            <Game v-for="(game, index) in games" :key="index" :game="game" />
          </b-card-group>
        </b-tab>
        <b-tab title="Vue en liste" title-item-class="mx-auto">
          <b-table striped hover :items="games" :fields="fields" sortBy="name">
            <template slot="isPlayableInTeams" slot-scope="data">
              {{ data.item.isPlayableInTeams ? "oui" : "non" }}
            </template>
            <template slot="averagePlayTime" slot-scope="data">
              {{ data.item.averagePlayTime }} min
            </template>
            <template slot="link" slot-scope="data">
              <a :href="data.item.link">Lien vers GameBoardGeek</a>
            </template>
          </b-table>
        </b-tab>
      </b-tabs>
    </section>

    <!-- Modal Component -->
    <b-modal id="deleteNight" hide-header lazy @ok="deleteNight">
      <div v-if="selectedNight">
        Veux-tu vraiment supprimer cette Game Night du {{ selectedNight.date.toDate() | formatDate }} ?
      </div>
    </b-modal>
    <b-modal id="editNight" hide-header lazy @ok="editNight">
      <div v-if="selectedNight">
        <b-form-group id="dateGroup"
                      label="Date"
                      label-for="date">
          <b-form-input id="date"
                        type="date"
                        required
                        v-model="selectedNight.dateFormated"
                        placeholder="01/01/2018">
          </b-form-input>
        </b-form-group>
        <b-form-group id="numberOfPlayersGroup"
                      label="Nombre de joueurs prévus"
                      label-for="numberOfPlayers">
          <b-form-input id="numberOfPlayers"
                        type="number"
                        required
                        v-model="selectedNight.numberOfPlayers"
                        placeholder="0">
          </b-form-input>
        </b-form-group>
      </div>
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
  name: 'home',
  components: {
    Game
  },
  data: function() {
    return {
      games: [],
      nights: [],
      fields:[
        { key: 'name', label: "Nom", sortable: true },
        { key: 'minPlayers', label: "Nombre de joueurs minimum", sortable: true },
        { key: 'maxPlayers', label: "Nombre de joueurs maximum", sortable: true },
        { key: 'isPlayableInTeams', label: "Jouable en équipe ?", sortable: true },
        { key: 'averagePlayTime', label: "Temps moyen de jeu", sortable: true },
        { key: 'link', label: "Lien", sortable: false }
      ],
      selectedNight: null
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
    fetchData () {
      this.games = [];
      Firebase.db.collection("games").orderBy("name").get().then((querySnapshot) => {
        for (let i = 0; i < querySnapshot.docs.length; i++) {
          let game = querySnapshot.docs[i].data();
          game.id = querySnapshot.docs[i].id;
          this.games.push(game);
        }

        this.games = this.games.sort((a, b) => {
          return a.name.toLowerCase().localeCompare(b.name.toLowerCase());
        });
      });

      this.nights = [];
      Firebase.db.collection("nights").where("date", ">=", new Date()).get().then((querySnapshot) => {
        querySnapshot.forEach((doc) => {
          let night = doc.data();
          night.id = doc.id;
          night.dateFormated = new Moment(String(night.date.toDate()), "ddd MMM DD YYYY HH:mm:ss ZZ").format('YYYY-MM-DD');
          this.nights.push(night);
        });
      });
    },
    deleteNight () {
      Firebase.db.collection("nights").doc(this.selectedNight.id).delete()
      .then(() => {
        console.log("Document successfully deleted!");
        this.fetchData();
      }).catch((error) => {
        console.error("Error removing document: ", error);
      });
    },
    editNight () {
      Firebase.db.collection("nights").doc(this.selectedNight.id).update({
        date: new Date(this.selectedNight.dateFormated),
        numberOfPlayers: Number(this.selectedNight.numberOfPlayers)
      })
      .then(() => {
        console.log("Document successfully edited");
        this.fetchData();
      })
      .catch((error) => {
        console.error("Error updating document: ", error);
        alert("Error updating document: ".error);
      });
    }
  }
}
</script>

<style lang="scss">
section {
    margin: 2rem 0;
}
.game-list {
  padding: 0 1rem;
}
.game {
  min-width: 20rem;
  margin: 1rem;
}
.nights {
    margin: auto;
    max-width: 30rem;
}
</style>
