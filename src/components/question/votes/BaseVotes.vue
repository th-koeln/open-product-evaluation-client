
<script lang="ts">
import VueApexCharts from 'vue-apexcharts';
import { mapGetters } from 'vuex';

export default {
    name: 'RegulatorVotes',
    components: {
        apexchart: VueApexCharts,
    },
    props: {
        id: String,
    },
    computed: {
        votes(this: any) {
            return this.$store.getters.getVote(this.id) || [];
        },
        answers(this: any) {
            return this.$store.getters.getAnswers(this.id) || [];
        },
        question(this: any) {
            return this.$store.getters.getQuestion(this.id);
        },
        activeTab(this: any) {
            return this.$store.getters.getActiveVoteTabe;
        },
    },
    watch: {
        answers(this: any, val: any) {
            this.getVotesDiagramm();
        },
        votes(this: any) {
            this.getVotesDiagramm();
        },
        activeTab(this: any) {
            if (this.activeTab) {
                this.getVotesDiagramm();
            }
        },
    },
    created(this: any) {
        // Change Votes Structure for Diagram
        this.getVotesDiagramm();
    },
};
</script>

<style scoped="true" lang="scss">
.chartDiagramm {
    margin: 0 auto; 
    width: 60%;
}
</style>