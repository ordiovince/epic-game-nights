<script>
import { Doughnut, mixins } from 'vue-chartjs'
const { reactiveProp } = mixins

export default {
  extends: Doughnut,
  name: 'Results',
  mixins: [reactiveProp],
  mounted () {
    // Overwriting base render method with actual data.
    // this.chartData is created in the mixin.
    this.renderChart(this.chartData, {
      responsive: true,
      maintainAspectRatio: false,
      legend: {
        display: false
      }
    });
  },
  methods: {
    update() {
      this.$data._chart.update();
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
</style>
