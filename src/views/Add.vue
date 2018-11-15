<template>
  <div class="add">
    <section>
      <h1>Ajouter un jeu</h1>

      <b-form-group id="searchGroup"
                    label="Rechercher un jeu"
                    label-for="search">
        <b-form-input id="search"
                      type="text"
                      required
                      placeholder="Tapper une recherche ici"
                      @input="search">
        </b-form-input>
      </b-form-group>
      <b-list-group>
        <b-list-group-item v-for="(result, index) in results" :key="index" button @click="gameSelected(result.id, result.name)" style="cursor: pointer; z-index: 99;">
          {{ result.name }} ({{ result.yearPublished }})
        </b-list-group-item>
      </b-list-group>
      <b-form @submit="onSubmit" v-if="show">
        <b-form-group id="nameGroup"
                      label="Nom"
                      label-for="name">
          <b-form-input id="name"
                        type="text"
                        required
                        v-model="form.name"
                        placeholder="Nom">
          </b-form-input>
        </b-form-group>
        <b-form-group id="ownersGroup"
                      label="Propriétaires"
                      label-for="owners">
          <b-form-input id="owners"
                        type="text"
                        required
                        v-model="form.owners"
                        placeholder="VOR,CGE">
          </b-form-input>
        </b-form-group>
        <b-form-group id="minPlayersGroup"
                      label="Nombre de joueurs minimum"
                      label-for="minPlayers">
          <b-form-input id="minPlayers"
                        type="number"
                        required
                        v-model="form.minPlayers"
                        placeholder="0">
          </b-form-input>
        </b-form-group>
        <b-form-group id="maxPlayersGroup"
                      label="Nombre de joueurs maximum"
                      label-for="maxPlayers">
          <b-form-input id="maxPlayers"
                        type="number"
                        required
                        v-model="form.maxPlayers"
                        placeholder="0">
          </b-form-input>
        </b-form-group>
        <b-form-group id="isPlayableInTeamsGroup">
          <b-form-checkbox  id="isPlayableInTeams"
                            v-model="form.isPlayableInTeams">
            Jouable en équipes ?
          </b-form-checkbox>
        </b-form-group>
        <b-form-group id="averagePlayTimeGroup"
                      label="Temps de jeu moyen"
                      label-for="averagePlayTime">
          <b-form-input id="averagePlayTime"
                        type="text"
                        required
                        v-model="form.averagePlayTime"
                        placeholder="0">
          </b-form-input>
        </b-form-group>
        <b-form-group id="linkGroup"
                      label="Lien GameBoardGeek"
                      label-for="link">
          <b-form-input id="link"
                        type="text"
                        required
                        v-model="form.link"
                        placeholder="https://">
          </b-form-input>
        </b-form-group>
        <b-form-group id="thumbnailLinkGroup"
                      label="Lien vers l'image"
                      label-for="thumbnailLink">
          <b-form-input id="thumbnailLink"
                        type="text"
                        required
                        v-model="form.thumbnailLink"
                        placeholder="https://">
          </b-form-input>
          <img :src="form.thumbnailLink" class="game-img" />
        </b-form-group>
        <b-button type="submit" variant="primary">Ajouter</b-button>
      </b-form>
    </section>
  </div>
</template>

<script>
// @ is an alias to /src
import Firebase from '@/services/Firebase.vue'
var parseString = require('xml2js').parseString;

export default {
  name: 'add',
  data () {
    return {
      form: {
        name: '',
        minPlayers: null,
        maxPlayers: null,
        isPlayableInTeams: false,
        averagePlayTime: '',
        link: '',
        thumbnailLink: '',
        owners: ''
      },
      results: [],
      request: undefined,
      show: true
    }
  },
  created() {
  },
  methods: {
    onSubmit (evt) {
      evt.preventDefault();

      this.form.minPlayers = Number(this.form.minPlayers);
      this.form.maxPlayers = Number(this.form.maxPlayers);
      this.form.owners = this.form.owners.split(",");
      Firebase.db.collection("games").add(this.form)
        .then((docRef) => {
          console.log("Document written with ID: ", docRef.id);
          this.$router.push("/");
        })
        .catch((error) => {
          console.error("Error adding document: ", error);
          alert("Error adding document: ".error);
        });
    },
    search (search) {
      if (this.request) {
        this.request.abort();
      }
      if (search === "") {
        this.results = [];
        return;
      }

      this.request = new XMLHttpRequest();
      this.request.open("GET", "https://www.boardgamegeek.com/xmlapi2/search?type=boardgame&query=" + search, true);
      this.request.onreadystatechange = function() {
        if (this.request.readyState == 4 && this.request.status == 200)
        {
          var doc = this.request.responseText;
          parseString(doc, function(err, result) {
            let results = [];
            for (let i = 0; i < result.items.item.length; i++) {
              results.push({
                id: result.items.item[i].$.id,
                name: result.items.item[i].name[0].$.value,
                yearPublished: result.items.item[i].yearpublished ? result.items.item[i].yearpublished[0].$.value : undefined
              })
            }
            if(results.length !== this.results.length) {
              this.results = results;
            }
            for(var i = results.length; i--;) {
              if(results[i] !== this.results[i]) {
                this.results = results;
              }
            }
          }.bind(this));
        }
      }.bind(this);
      this.request.send(null);
    },
    gameSelected (gameId, gameName) {
      this.results = [];
      if (this.request) {
        this.request.abort();
      }

      this.request = new XMLHttpRequest();
      this.request.open("GET", "https://www.boardgamegeek.com/xmlapi2/thing?type=boardgame&id=" + gameId, true);
      this.request.onreadystatechange = function() {
        if (this.request.readyState == 4 && this.request.status == 200)
        {
          var doc = this.request.responseText;
          parseString(doc, function(err, result) {
            this.form.name = gameName;
            this.form.minPlayers = result.items.item[0].minplayers[0].$.value;
            this.form.maxPlayers = result.items.item[0].maxplayers[0].$.value;
            let minPlayingTime = result.items.item[0].minplaytime[0].$.value;
            let maxPlayingTime = result.items.item[0].maxplaytime[0].$.value;
            this.form.averagePlayTime = minPlayingTime === maxPlayingTime ? minPlayingTime : minPlayingTime + "-" + maxPlayingTime;
            this.form.link = "https://boardgamegeek.com/boardgame/" + gameId;
            this.form.thumbnailLink = result.items.item[0].thumbnail[0];
          }.bind(this));
        }
      }.bind(this);
      this.request.send(null);
    }
  }
}
</script>

<style lang="scss">
section {
  margin: 2rem 0;
}
.add {
  margin: auto;
  padding: 0 1rem;
  max-width: 30rem;
}
.list-group {
  position: absolute;
  margin-top: -17px;
  max-width: 28rem;
}

.game-img {
  height: 10rem;
  width: 10rem;
  margin: 1rem auto;
}
</style>
