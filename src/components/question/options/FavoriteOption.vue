<template>
<div>
  <b-row class="px-3">
  <b-col v-for="item in question.items" 
  :key="item.id" 
  :class="((question.items.length%2)===0) ? 'col-md-6' : 'col-md-4'" 
  cols="6"
  class="p-2">
      <b-card no-body :class="{ selected: selected === item.id}"
          header-tag="header"
          class="h-100 shadow bg-white"
          @click="select($event, item.id)">
        <img v-if="item.image && item.image.url" class="card-img-top" v-img :src="`${item.image.url}`">
        <b-card-body class="p-2">
        <h5 class="card-title">{{ item.label }}</h5>
        <!--<b-button variant="primaryBtn" class="btn-block" @click="select($event, item.id)"> {{ item.label }}</b-button>-->
        </b-card-body>
      </b-card>
    </b-col>
    </b-row>
  <b-row class="mx-2 neutral" align-h="end">
          <input type="checkbox" :checked="selected==null" @click="deselectAll()"/>
          <label> abstain from voting</label>
    </b-row>
  </div>
</template>

<script lang="ts">
export default {
  name: 'FavoriteOption',
  props: {
    id: String,
  },
  data() {
    return {
      selected: null,
      answered: false,
    };
  },
  computed: {
    question(this: any) {
      return this.$store.getters.getQuestion(this.id);
    },
  },
  methods: {
    select(this: any, event, id) {
      event.preventDefault();
      this.selected = id;
    },
    deselectAll(this: any) {
      this.selected = null;
    },
    sendAnswer(this: any) {
      this.$store.dispatch('createAnswerFavorite', { question: this.id, favoriteID: this.selected});
      this.answered = true;
      this.$root.$emit('answered');
    },
  },
  beforeDestroy(this: any) {
    this.$eventBus.$off();
  },
  mounted(this: any) {
    this.$eventBus.$on('answer', (data) => {
      this.sendAnswer();
    });
  },
};
</script>


<style scoped="true" lang="scss">
@import "../../../scss/variables"; 
  .item {
    display: block;
    border: 3px solid #FFFFFF;
  }
  .images {
    height: 100%;
    width: 100%;
    object-fit: fill;
  }
  .selected {
    border: 3px solid $primaryColor;
  }
  .card-header {
    padding: 0;
    max-height: 35vh;
  }
  .card-img-top {
    object-fit: cover;
    height: 20vh;
  }
  .card-body {
    align-items: center;
    display: flex;
    justify-content: center;
    h5{
      margin: 0px;
      width: 100%;
    }
  }

@media (max-width: 1200px) {
.card-img-top {
    object-fit: cover;
    height: 15vh;
  }
}

@media (max-width: 540px) {
.card-img-top {
    object-fit: cover;
    height: 10vh;
  }
} 
</style>