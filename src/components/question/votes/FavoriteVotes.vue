<template>
<div class="chartDiagramm">
<h5 v-if="votes.length === 0 && answers.length === 0"> No votes submitted. </h5>
<span v-if="votes.length > 0 || answers.length > 0">
    <h5> Number of abstentions: {{neutral}}</h5>
    <apexchart type="bar" :options="chartOptions" :series="series"></apexchart>
</span>
</div>
</template>

<script lang="ts">
import VueApexCharts from 'vue-apexcharts';
import BaseVotes from './BaseVotes.vue';

export default {
  name: 'FavoriteVotes',
  extends: BaseVotes,
  data() {
      return {
        neutral: 0,
        chartOptions: {
          chart: {
            id: 'vuechartChoice',
            toolbar: { show: false },
          },
          theme: {
              palette: 'palette4',
          },
          dataLabels: {
            style: {
                fontSize: '1.25rem',
            },
          },
          yaxis: {
              axisBorder: {
                  show: true,
              },
              title: {
                  text: 'Number of votes',
              },
          },
          tooltip: {
                y: {
                    title: {
                        formatter() {
                            return '';
                        },
                    },
                },
            },
        },
        series: [{
            data: [],
        }],
      };
    },
    methods: {
        countInArray(this: any, votes, id) {
            let counter = 0;
            votes.forEach( (answer) => {
                answer.forEach( (element) => {
                    if (element.favoriteItem === id) {
                        counter ++;
                    }
                });
            });
            this.answers.forEach( (element) => {
                if (element.favoriteItem === id) {
                    counter ++;
                }
            });
            return counter;
        },
        getVotesDiagramm(this: any) {
            if (this.answers.length > 0 || this.votes.length > 0) {
                const result: any[] = [];
                this.question.items.forEach( (element) => {
                    result.push({
                        x: element.label,
                        y: this.countInArray(this.votes, element.id),
                    });
                });
                this.neutral = this.countInArray(this.votes, null);
                this.series = [{
                    data: result,
                }];
            }
        },
    },
};
</script>

<style scoped="true" lang="scss">
.chartDiagramm {
    margin: 0 auto; 
    width: 80%;
}
@media (max-width: 576px) {
.chartDiagramm {
    width: 100%;
}
}
</style>