<template>
  <div class="home">
    <b-card-group deck class="game-list">
      <Game v-for="(game, index) in games" :key="index" :game="game" />
    </b-card-group>
  </div>
</template>

<script>
// @ is an alias to /src
import Game from '@/components/Game.vue'
import Firebase from '@/services/Firebase.vue'

var games = [];
Firebase.db.collection("games").orderBy("name").get().then((querySnapshot) => {
  querySnapshot.forEach((doc) => {
    games.push(doc.data());
  });
});

var results = [];
var resultsA = [];
var resultsB = [];
var resultsC = [];
Firebase.db.collection("games").where("minPlayers", "<=", 7).get().then((querySnapshot) => {
  for (let i = 0; i < querySnapshot.docs.length; i++) {
    resultsA.push(querySnapshot.docs[i].id);
  }
  Firebase.db.collection("games").where("maxPlayers", ">=", 7).get().then((querySnapshot) => {
    for (let i = 0; i < querySnapshot.docs.length; i++) {
      resultsB.push(querySnapshot.docs[i].id);
    }
    Firebase.db.collection("games").where("isPlayableInTeams", "==", true).get().then((querySnapshot) => {
      for (let i = 0; i < querySnapshot.docs.length; i++) {
        resultsC.push(querySnapshot.docs[i].id);
      }

      console.log(resultsA);
      console.log(resultsB);
      console.log(resultsC);

      results = resultsA.filter(value => -1 !== resultsB.indexOf(value));
      for (let i = 0; i < resultsC.length; i++) {
        if (results.indexOf(resultsC[i]) === -1) {
          results.push(resultsC[i]);
        }
      }
      console.log(results);
    });
  });
});
/*
Firebase.db.collection("nights").get().then((querySnapshot) => {
  querySnapshot.forEach((doc) => {
    console.log(doc.data());
    let night = doc.data();
    for (let i = 0; i < night.votes.length; i++) {
      let vote = night.votes[i];
      for (let j = 0; j < vote.scores.length; j++) {
        let score = vote.scores[j];
        score.game.get().then((game) => {
          console.log(game.data());
        });
      }
    }
  });
});
*/

export default {
  name: 'home',
  components: {
    Game
  },
  data: function() {
    return {
      games
    };
  }
}
</script>

<style lang="scss">
.game-list {
  padding: 0 1rem;
}
</style>
