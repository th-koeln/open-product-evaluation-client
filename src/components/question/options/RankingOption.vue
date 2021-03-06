<template>
<div>
    <div style="text-align: right">
    <h6 class="chooseInstruction" v-if="allItems.length>0">Click or drag to choose position #{{choosePosition+1}}</h6>
    <b-btn class="resetBtn" @click="preset">Reset</b-btn>
    </div>
  <div class="drag">
    <b-row class="dragzone d-flex mb-3 mx-2">
        <b-col draggable="true" 
        v-for="(item, index) in allItems" :key="index"
        @dragstart="onDragStart($event, item, index)" 
        @dragend="onDragEnd($event)"
        cols="6"
        class="p-2 col-md-4"
        >
          <b-card no-body class="h-100 dragCards shadow bg-white"
          @click="choseItem($event, item, index, choosePosition)" >
                <img class="card-img-top" draggable="false" v-if="item.image && item.image.url" v-img :src="`${item.image.url}`">
          <b-card-body class="p-2">
              <b-card-text>
                {{item.label}}
                </b-card-text>
          </b-card-body>
          </b-card>
        </b-col>
      </b-row>
      <b-row class="mx-2 justify-content-around" cols="6">
        <b-col v-for="(item, index) in position" 
          :key="index" 
          class="dragger p-0 mx-2 col col-md-6" 
          draggable="true" v-bind="item" 
          @drop.prevent="onDrop(index, $event)"
          @dragover.prevent="onOver(index)" 
          @dragleave.prevent="onDragLeave(index)">
            <b-card no-body 
            class="h-100 dropCards shadow bg-white" 
            v-if="item.label">
                <b-card-header v-if="item.image && item.image.url">
                  <img class="imageVote w-100 h-100" 
                  v-if="item.image && item.image.url" 
                  v-img 
                  :src="`${item.image.url}`">
                </b-card-header>
                <b-card-text>
                  <p> {{item.label}} </p>
                  <p> Position #{{index+1}} </p>
                  </b-card-text>
            </b-card>
            <p class="h-100 emptyCard" v-if="item.name" :id="'text'+index">{{item.name}}</p>
        </b-col>
        </b-row>
      </div>
      <b-row class="mx-2 neutral" align-h="end">
            <input type="checkbox" 
            :checked="selected==null" 
            @click="deselectAll()"/>
            <label> abstain from voting</label>
      </b-row>
    <b-modal centered 
    ref="warningModal" 
    hide-footer 
    title="Warning">
      <div class="d-block text-center">
        <h3>The answer is incomplete!</h3>
      </div>
    </b-modal>
  </div>
</template>

<script lang="ts">
export default {
  name: 'RankingOption',
  props: {
    id: String,
  },
  data() {
    return {
      selected: null,
      answered: false,
      allItems: [],
      dragedItem: {},
      droppedItem: [],
      position: [],
      choosePosition: 0,
    };
  },
  watch: {
    dragedItem(this: any) {
      this.allItems.reduce((pre, cur) => {
        return pre;
      }, {});
    },
  },
  computed: {
    question(this: any) {
      return this.$store.getters.getQuestion(this.id);
    },
  },
  created(this: any) {
    this.preset();
  },
  methods: {
    preset(this: any) {
      this.position = [];
      this.choosePosition = 0;
      for (let i = 0; i < this.question.items.length; i++) {
            this.position.push({
              name: 'Position #' + (i + 1),
            });
      }
      this.allItems = this.question.items.slice();
      this.selected = null;
      return this.position;
    },
    onDragStart(this: any, ev, item, index) {
      ev.dataTransfer.setData('text/plain', null);
      this.dragedItem = {item, index};
    },
    onDragEnd(this: any, ev) {
      this.dragedItem = {};
      this.selected = true;
    },
    onDrop(this: any, index, ev) {
      this.position[index] = this.dragedItem.item;
      this.allItems.splice(this.dragedItem.index, 1);
      const element = document.getElementById('text' + index);
      if (element != null) {element.style.backgroundColor = '#ffffff'; }
    },
    /* Change color if element over dropzone */
    onOver(this: any, index) {
      const element = document.getElementById('text' + index);
      if (element != null) {element.style.backgroundColor = '#ffaa66'; }
    },
    onDragLeave(this: any, index) {
      const element = document.getElementById('text' + index);
      if (element != null) {element.style.backgroundColor = '#eef1f5'; }
    },
    /* Only available on mobile */
    choseItem(this: any, $event, item, index, position) {
      this.selected = true;
      this.position[position] = item;
      this.allItems.splice(index, 1);
      this.choosePosition += 1;
    },
    deselectAll(this: any) {
      this.selected = null;
    },

    /** Get votes and change sequence of votes for Server */
    getAnswers(this: any) {
      if (this.selected === null ) {
        return null;
      } else {
        // Build array for rankingItems
        const favoriteArray: string[] = [];
        this.position.forEach( (element) => {
          favoriteArray.push(element.id);
        });
        return favoriteArray;
      }
    },
    sendAnswer(this: any) {
      this.$store.dispatch('createAnswerRanking', { question: this.id, rankingID: this.getAnswers() });
      this.answered = true;
      this.$root.$emit('answered');
    },
  },
  beforeDestroy(this: any) {
    this.$eventBus.$off();
  },
  mounted(this: any) {
    this.$eventBus.$on('answer', (data) => {
      // Check if all positions have an item.  If not, open modal to inform user
      if (this.allItems.length === 0 || this.selected == null) {
        this.sendAnswer();
      } else {
        this.$refs.warningModal.show();
      }
    });
  },
};
</script>

<style scoped="true" lang="scss">
@import "../../../scss/variables"; 

  .chooseInstruction{
    text-align: center;
  }
  /* .dragCards .card-header {
    padding: 0;
    height: 20vh;
  } */
  /* .dropCards .card-header {
    padding: 0;
    height: 10vh;
  } */
  .dragzone {
  display:flex;
  flex-direction: row;
}
.dragger {
  text-align: center;
  border: 1px dashed gray;
  height: 15vh;
  width: 15vw; 
  max-width: 250px;
  display: table;
  margin: 8px;
}
.emptyCard {
  display: table-cell;
  text-align: center;
  vertical-align: middle;
  background: $backgroundColor;
}
.imageVote {
    object-fit: contain;
    }

@media (min-width: 720px) {
  .imagePrev {
      object-fit: contain;
      }
  .mobileBtn{
    display: none;
  }
}
@media (max-width: 720px) {
  .imagePrev {
    object-fit: cover;
  }
  .mobileBtn {
    display: block;
  }
  .col-md-6 {
    min-width: 50%;
  }
}
.resetBtn {
  border-color: $primaryColor;
  background-color: transparent;
  color: $primaryColor;
  padding: 0.25rem 0.75rem;
}

.resetBtn:hover {
  background-color: $primaryColor;
  border-color: $primaryColor;
  color: white;
}

.card-header {
    padding: 0;
    max-height: 35vh;
}

.card-img-top {
  object-fit: cover;
}
.card-body {
  align-items: center;
  display: flex;
  justify-content: center;
  p{
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