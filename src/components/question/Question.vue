<template>
    <div class="question">
      <b-card no-body class="shadow bg-white rounded">
        <b-card-header>
          <h4>{{ survey.title }}</h4>
        </b-card-header>
      <b-tabs card>
      <b-tab no-body title="Question" active>
        <b-card-body class="py-0">
            <div class="progress w-100" v-if="survey.questions && survey.questions.length > 8">
              <b-progress :max="100">
                <b-progress-bar :value="counter" show-progress :label="`${counter}%`">
                </b-progress-bar>
              </b-progress>
            </div>
            <b-row class="m-0">
              <div class="w-100 my-3" v-if="survey.questions && survey.questions.length <= 8">
                <step-indicator current-color='#ffaa11' :current="index" :total="survey.questions.length"></step-indicator>
              </div>
              <div class="w-100" v-if="survey.questions
                                      && survey.questions.length
                                      && survey.questions.length > 0">
              <!-- display question title and description -->
              <questionMeta :id="survey.questions[index].id"></questionMeta>

              <!-- display question items -->
              <questionItems :id="survey.questions[index].id"
                    v-if="displayItems(survey.questions[index].type)">
              </questionItems>

              <!-- display choices -->
              <choice :id="survey.questions[index].id"
                      v-if="survey.questions[index].type === 'CHOICE'">
              </choice>

              <!-- display like / dislike options -->
              <likeDislike :id="survey.questions[index].id"
                          v-if="survey.questions[index].type === 'LIKEDISLIKE'">
              </likeDislike>

              <!-- display like option -->
              <like :id="survey.questions[index].id"
                    v-if="survey.questions[index].type === 'LIKE'">
              </like>

              <!-- display regulator option -->
              <regulator :id="survey.questions[index].id"
                        v-if="survey.questions[index].type === 'REGULATOR'">
              </regulator>

              <!-- display special ranking options -->
              <ranking :id="survey.questions[index].id"
                      v-if="survey.questions[index].type === 'RANKING'">
              </ranking>

              <!-- display special favorite options -->
              <favorite :id="survey.questions[index].id"
                        v-if="survey.questions[index].type === 'FAVORITE'">
              </favorite>
            </div>
          </b-row>
        </b-card-body>
      </b-tab>

      <b-tab no-body title="Votes">
        <b-card-body class="py-0">
          <div class="progress w-100" v-if="survey.questions && survey.questions.length > 8">
              <b-progress :max="100">
                <b-progress-bar :value="counter" show-progress :label="`${counter}%`">
                </b-progress-bar>
              </b-progress>
          </div>
          <b-row class="m-0">
              <div class="w-100 my-3" v-if="survey.questions && survey.questions.length <= 8">
                <step-indicator current-color='#ffaa11' :current="index" :total="survey.questions.length"></step-indicator>
              </div>
              <div class="w-100" v-if="survey.questions
                                      && survey.questions.length
                                      && survey.questions.length > 0">
              <!-- display question title and description -->
              <questionMeta :id="survey.questions[index].id"></questionMeta>
              </div>
          </b-row>
          <div class="votes" v-if="survey.questions
                                        && survey.questions.length
                                        && survey.questions.length > 0 ">
            <h5>Previous votes</h5>
            <!-- display  Votes -->
            <choiceVotes :id="survey.questions[index].id"
                      v-if="survey.questions[index].type === 'CHOICE'">
            </choiceVotes>
            <favoriteVotes :id="survey.questions[index].id"
                      v-if="survey.questions[index].type ==='FAVORITE'">
            </favoriteVotes>
            <likeVotes :id="survey.questions[index].id"
                      v-if="survey.questions[index].type === 'LIKE'">
            </likeVotes>
            <likeDislikeVotes :id="survey.questions[index].id"
                      v-if="survey.questions[index].type === 'LIKEDISLIKE'">
            </likeDislikeVotes>
            <regulatorVotes :id="survey.questions[index].id"
                      v-if="survey.questions[index].type === 'REGULATOR'">
            </regulatorVotes>
            <rankingVotes :id="survey.questions[index].id"
                      v-if="survey.questions[index].type === 'RANKING'">
            </rankingVotes>
          </div>
        </b-card-body>
      </b-tab>
    </b-tabs>

      <b-card-footer>
          <b-btn class="float-right" variant="primaryBtn" @click="next"
                  v-if="index !== survey.questions.length - 1"> Next
          </b-btn>
          <b-btn variant="primaryBtn" @click="next"
                  v-if="index == survey.questions.length -1"> Finish
          </b-btn>
      </b-card-footer>
    </b-card>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator';
import QuestionItem from '@/components/question/QuestionItem.vue';
import ChoiceOptions from '@/components/question/options/ChoiceOption.vue';
import RankingOptions from '@/components/question/options/RankingOption.vue';
import FavoriteOptions from '@/components/question/options/FavoriteOption.vue';
import RegulatorOptions from '@/components/question/options/RegulatorOption.vue';
import LikeOptions from '@/components/question/options/LikeOption.vue';
import LikeDislikeOptions from '@/components/question/options/LikeDislikeOption.vue';
import QuestionValue from '@/components/question/QuestionValue.vue';
import ChoiceVotes from '@/components/question/votes/ChoiceVotes.vue';
import FavoriteVotes from '@/components/question/votes/FavoriteVotes.vue';
import LikeVotes from '@/components/question/votes/LikeVotes.vue';
import LikeDislikeVotes from '@/components/question/votes/LikeDislikeVotes.vue';
import RegulatorVotes from '@/components/question/votes/RegulatorVotes.vue';
import RankingVotes from '@/components/question/votes/RankingVotes.vue';
import StepIndicator from 'vue-step-indicator';

export default {
  name: 'Question',
  components: {
    questionItems: QuestionItem,
    choice: ChoiceOptions,
    ranking: RankingOptions,
    favorite: FavoriteOptions,
    regulator: RegulatorOptions,
    like: LikeOptions,
    likeDislike: LikeDislikeOptions,
    questionMeta: QuestionValue,
    StepIndicator,
    choiceVotes: ChoiceVotes,
    favoriteVotes: FavoriteVotes,
    likeVotes: LikeVotes,
    likeDislikeVotes: LikeDislikeVotes,
    regulatorVotes: RegulatorVotes,
    rankingVotes: RankingVotes,
  },
  data() {
    return {
      index: 0,
      counter: 0,
      counting: 0,
    };
  },
  created(this: any) {
    const domainID = this.$route.params.cID;
    this.$store.dispatch('getSurvey', {
        domain: domainID,
    }).then((data) => {
        this.$store.dispatch('getVotes', {
          surveyID: this.survey.id,
        });
      }, (error) => {
        console.log(error.message);
      });
  },
  computed: {
    survey(this: any) {
      return this.$store.getters.getSurvey;
    },
    votes(this: any) {
      return this.$store.getters.getVotes;
    },
  },
  methods: {
    displayItems(type) {
      return !(type === 'RANKING' || type === 'FAVORITE');
    },
    next(this: any) {
        if (this.index < this.survey.questions.length - 1) {
          this.$eventBus.$emit('answer');
          this.counter = Math.floor(this.index / this.survey.questions.length * 100);
          this.index++;
        } else {
          this.$eventBus.$emit('answer');
          this.$router.push({name: 'surveyList'});
        }
    },
  },
};
</script>
<style src="vue-step-indicator/dist/vue-step-indicator.css"></style>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
@import "../../scss/variables.scss"; 
  @media (min-width: 540px) {
  .question {
    width: 80%;
    margin: 3rem auto;
    }
}
@media (max-width: 540px) {
  .question {
    width: 100%;
    margin: 0 auto;
    }
}
.question {
  object-fit: cover; 
  max-width: 1200px;
}
.card-header h4 {
  margin-top: 0.5rem;
}
.votes {
  padding-top: 1rem;
  margin-top: 1rem;
}
.collapsed > .when-opened,
:not(.collapsed) > .when-closed {
  display: none;
}
.tab-content > .active {
  outline: none;
}
</style>
