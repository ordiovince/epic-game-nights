<template>
  <div class="card game">
    <div class="card-body">
      <h5 class="card-title">{{ game.name }}</h5>
      <p class="card-text">Nombre de joueurs : {{ game.minPlayers }}-{{ game.maxPlayers }}</p>
      <p class="card-text">Jouable en équipe : {{ game.isPlayableInTeams ? "oui" : "non" }}</p>
      <p class="card-text">Temps de jeu moyen : {{ game.averagePlayTime }} min</p>
    </div>
    <div class="card-footer">
      <a :href="game.link" class="card-link" target="_blank"><small>Lien vers GameBoardGeek</small></a>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Game',
  props: {
    game: Object
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
.game {
  min-width: 20rem;
  margin: 1rem;
}
</style>
