<template>
  <div class="surveycards">
    <div class="header">
      <h2>Surveys</h2>
      <h5>Choose a survey</h5>
      <p>You can answer it directly by using the <b>"Start"</b>-button OR begin a survey in <b>Live mode</b> to receive a QR-Code and control the survey.</p>
    </div>

    <b-row align-v="center" class="m-0">
      <b-col cols="1" >
        <label v-if="pages > 1">
            <input type="button"
                @click="goto(-1)"/>
            <v-icon variant="primary" class="icon" name="angle-left" ></v-icon>
            </label>
        </b-col>

      <b-col cols="10">
        <b-row class="m-0 w-100">
        <transition :name="animation" >
          <b-card-group deck :key="pageIndex">
                <b-col cols="12" sm="6" lg="4" v-for="survey in currentPageSurveys" :key="survey.id">
                  <b-card class="shadow bg-white rounded" no-body>
                  <b-card-header v-if="survey.activeSurvey.previewImage && survey.activeSurvey.previewImage.url">
                    <img class="rounded-top previewImage" v-img  :src="survey.activeSurvey.previewImage.url"/>
                  </b-card-header>
                  <b-card-body>
                    <b-card-title> {{survey.activeSurvey.title}} </b-card-title>
                    <b-card-text> {{survey.activeSurvey.description}} </b-card-text>
                  </b-card-body>
                  <b-card-footer>
                      <b-button variant="primaryBtn" @click="startSurvey(survey.id)">Start</b-button>
                      <b-button variant="primaryBtn" @click="startLive(survey.id)">Live mode</b-button>
                      <b-button variant="primaryBtn" @click="showQR(survey.id)">Show QR-Code</b-button>
                    </b-card-footer>
                  </b-card>
                </b-col>
          </b-card-group>
        </transition>
         </b-row>
  
      <div class="card-pagination" v-if="pages > 1">
      <div
        class="page-index"
        v-for="i in pages"
        :key="i"
        :class="{ active: (i - 1 === pageIndex)} "
      ></div>
      
    </div>
        </b-col>
      <b-col cols="1">
        <label v-if="pages > 1">
            <input type="button"
                @click="goto(1)"/>
            <v-icon variant="primary" class="icon" name="angle-right" ></v-icon>
            </label>
      </b-col>
    </b-row>
    <b-modal 
    centered
    hide-footer 
    ref="qrModal"
    title="Join"
    @ok="hide()">
      <qrcode :value="joinLink" :options="{ width: 400 }"></qrcode>
      <b-button id="modalButton" variant="primaryBtn" block @click="$refs.qrModal.hide()">Ok</b-button>
    </b-modal>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator';
import VueQrcode from '@chenfengyuan/vue-qrcode';

export default {
  name: 'SurveyEdit',
  components: {
    qrcode: VueQrcode,
  },
  data() {
    return {
      splittedSurveys: [],
      pages: 0,
      maxCards: 3,
      pageIndex: 0,
      animation: null,
      subscription: null,
      joinLink: null,
    };
  },
  created(this: any) {
      this.$store.dispatch('getSurveys').then((result) => {
        // received Surveys
      },
      (error) => {
        // console.log(error.message);
        // Authorization failed
        this.$router.push('login');
      });
      const clientID = localStorage.getItem('client');
      this.$store.dispatch('subscribeClient', { clientID }).then( (data) => this.subscription = data);
  },
  beforeDestroy(this: any) {
    this.$store.dispatch('unsubscribe', this.subscription);
  },
  computed: {
    surveys(this: any) {
      return this.$store.getters.getSurveys;
    },
    currentPageSurveys(this: any) {
      this.createPages();
      return this.splittedSurveys[this.pageIndex];
    },
  },
  mounted(this: any) {
    window.onresize = () => {
      this.pageIndex = 0;
      this.createPages();
    };
  },
  methods: {
    // Update Client
    startSurvey(this: any, domainID) {
      const client = localStorage.getItem('client');
      this.$store.dispatch('updateClient', {
        id: client,
        domainId: domainID,
      });
      this.$router.push({name: 'question', params: {cID: domainID}});
    },
    startLive(this: any, domainID) {
      const client = localStorage.getItem('client');
      this.$store.dispatch('updateClient', {
        id: client,
        domainId: domainID,
      }).then(
        () => this.$router.push({name: 'master', params: {cID: domainID}}),
      );
    },
    showQR(this: any, domainID) {
      this.joinLink = window.location.protocol + '//' + window.location.host + '/#/survey/' + domainID;
      this.$refs.qrModal.show();
    },
    createPages(this: any) {
      // Check WindowWidth for maxCards/page
      switch (true) {
        case (window.innerWidth <= 590):
          this.maxCards = 1;
          break;
        case (window.innerWidth > 590 && window.innerWidth <= 991):
          this.maxCards = 2;
          break;
        default: this.maxCards = 3;
      }
      // Count all Pages
      this.pages = 0;
      this.splittedSurveys = [];
      for (let i = 0; i < this.surveys.length; i = i + this.maxCards) {
        // Seperate Pages
        this.splittedSurveys[this.pages] = this.surveys.slice(i, i + this.maxCards);
        this.pages++;
      }
    },
    goto(this: any, value) {
      value > 0 ? this.animation = 'next' : this.animation = 'prev';
      switch (true) {
        case ((this.pageIndex + value) < 0):
          this.pageIndex = this.pages - 1;
          break;
        case ( (this.pageIndex + value) >= this.pages):
          this.pageIndex = 0;
          break;
        default: this.pageIndex += value;
      }
    },
  },
};

</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
@import "../../scss/variables"; 

#modalButton{
  margin: 0px;
}

.header {
  padding: 1.5rem 0;
}
.card-pagination {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 30px;
}
.page-index {
  margin-left: 5px;
  margin-right: 5px;
  width: 50px;
  height: 3px;
  background: $secondaryBackgroundColor;
}
.page-index.active {
  background: $primaryColor;
  width: 55px;
  height: 4px;
}
.icon {
  width: 3rem;
  height: 3rem;
  color: $secondaryBackgroundColor;
}
input[type="button"] {
  display: none;
}
.icon:hover {
  color: $primaryColor;
  width: 60px;
  height: 60px;
}
.card-text {
  text-align: left;
  word-wrap: break-word;
}
.card-body {
  padding: 1rem;
  overflow: scroll;
}
.card-title {
  font-size:1.1rem;
}
.previewImage {
  max-height: 20rem;
  width: 100%;
  object-fit: cover;
}
.card-deck .card, .card-deck {
  height: 65vh;
  margin: 0;
  width: 100%;
}
.col-10 {
  height: 80vh;
}
.col-1, .col-10, .card-header {
  padding: 0;
}
.next-leave-to {
  transform: translateX(-20px);
  transition: all .7s;
  opacity: 0;
}
.prev-leave-to {
  transform: translateX(20px);
  transition: all .7s;
  opacity: 0;
}
.next-enter-active{
  transition: all .5s ease;
  transition-delay: .8s;
}
.prev-enter-active{
  transition: all .5s ease;
  transition-delay: .8s;
}
.next-enter {
  transform: translateX(10px);
  opacity: 0;
}
.prev-enter {
  transform: translateX(-10px);
  opacity: 0;
}
</style>
