<template>
  <div class="add">
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
      <b-button type="submit" variant="primary">Ajouter</b-button>
    </b-form>
  </div>
</template>

<script>
// @ is an alias to /src
import Firebase from '@/services/Firebase.vue'

export default {
  name: 'add',
  data () {
    return {
      form: {
        name: '',
        minPlayers: 0,
        maxPlayers: 0,
        isPlayableInTeams: false,
        averagePlayTime: '',
        link: ''
      },
      show: true
    }
  },
  methods: {
    onSubmit (evt) {
      evt.preventDefault();

      Firebase.db.collection("games").add(this.form)
        .then(function(docRef) {
          console.log("Document written with ID: ", docRef.id);
        })
        .catch(function(error) {
          console.error("Error adding document: ", error);
        });
    }
  }
}
</script>

<style lang="scss">
.add {
    margin: auto;
    padding: 0 1rem;
    max-width: 30rem;
}
</style>
