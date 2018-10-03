<template>
  <div class="add">
    <section>
      <h1>Prévoir une Game Night</h1>

      <b-form @submit="onSubmit">
        <b-form-group id="dateGroup"
                      label="Date"
                      label-for="date">
          <b-form-input id="date"
                        type="date"
                        required
                        v-model="form.date"
                        placeholder="01/01/2018">
          </b-form-input>
        </b-form-group>
        <b-form-group id="numberOfPlayersGroup"
                      label="Nombre de joueurs prévus"
                      label-for="numberOfPlayers">
          <b-form-input id="numberOfPlayers"
                        type="number"
                        required
                        v-model="form.numberOfPlayers"
                        placeholder="0">
          </b-form-input>
        </b-form-group>
        <b-button type="submit" variant="primary">Prévoir</b-button>
      </b-form>
    </section>
  </div>
</template>

<script>
// @ is an alias to /src
import Firebase from '@/services/Firebase.vue'

export default {
  name: 'create-night',
  data () {
    return {
      form: {
        date: new Date(),
        numberOfPlayers: null
      }
    }
  },
  methods: {
    onSubmit (evt) {
      evt.preventDefault();

      this.form.date = new Date(this.form.date);
      this.form.numberOfPlayers = Number(this.form.numberOfPlayers);
      Firebase.db.collection("nights").add(this.form)
        .then((docRef) => {
          console.log("Document written with ID: ", docRef.id);
          this.$router.push("/");
        })
        .catch((error) => {
          console.error("Error adding document: ", error);
          alert("Error adding document: ".error);
        });
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
</style>
