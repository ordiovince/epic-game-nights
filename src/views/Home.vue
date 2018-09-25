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
