<template>
  <v-container>
    {{ windowWidth }}
    <v-tabs
      color="cyan"
      next-icon="mdi-menu-right-outline"
      prev-icon="mdi-menu-left-outline"
      show-arrows
      v-model="active"
    >
      <v-tabs-slider color="yellow"></v-tabs-slider>

      <v-tab v-for="i in 20" :key="i"> Q{{ i }} </v-tab>
      <v-tab-item v-for="i in 20" :key="i">
        <v-card>
          <v-container fill-height fluid style="height: 20vh">
            <v-row align="center" justify="center">
              <v-card-text style="text-align: center">
                <!-- q -->
                <span
                  :style="
                    windowWidth < 600 ? `font-size: 20px` : `font-size: 32px`
                  "
                >
                  {{ testing[i - 1].question.trans }}
                  {{ " (" + testing[i - 1].question.type + ")" }}
                </span>
              </v-card-text>
            </v-row>
          </v-container>

          <v-container fill-height fluid style="height: 40vh">
            <v-row align="center" justify="center">
              <!-- a -->
              <v-card-text style="text-align: center">
                <v-radio-group v-model="testing[i - 1].your_answer" color="red">
                  <v-row>
                    <v-col cols="12" md="4" sm="4" v-for="j in 3" :key="j">
                      <v-radio
                        :value="testing[i - 1].options[j - 1]"
                        @click="next(i - 1, testing[i - 1].options[j - 1])"
                      >
                        <template v-slot:label>
                          <span
                            :style="
                              windowWidth < 600
                                ? `font-size: 20px`
                                : `font-size: 32px`
                            "
                          >
                            {{ testing[i - 1].options[j - 1] }}
                          </span>
                        </template>
                      </v-radio>
                    </v-col>
                  </v-row>
                </v-radio-group>
              </v-card-text>
            </v-row>
          </v-container>
        </v-card>
      </v-tab-item>
    </v-tabs>

    <v-btn @click="submit" depressed x-large> 送出 </v-btn>

    <v-dialog v-model="dialog" fullscreen transition="dialog-top-transition">
      <v-card>
        <v-container fill-height fluid>
          <v-row align="center" justify="center">
            <span style="font-size: 3em; color: red">{{ score }} </span>
          </v-row>

          <hr
            width="50%"
            style="margin-left: 25% !important; margin-right: 25% !important"
            class="mt-3 mb-3"
          />

          <v-list-item v-for="(check_item, index) in check_list" :key="index">
            <v-list-item-content style="text-align: center">
              <v-list-item-title
                :style="
                  windowWidth < 600 ? `font-size: 14px` : `font-size: 20px`
                "
                >{{ check_item.answer }}</v-list-item-title
              >
            </v-list-item-content>

            <v-list-item-content style="text-align: center">
              <v-list-item-title
                :style="
                  windowWidth < 600 ? `font-size: 14px` : `font-size: 20px`
                "
              >
                {{
                  check_item.question.trans +
                  " (" +
                  check_item.question.type +
                  ")"
                }}
              </v-list-item-title>
            </v-list-item-content>
          </v-list-item>

          <v-row align="center" justify="center" class="mt-3 mb-3">
            <v-btn class="mr-3" depressed @click="tryAgain">再練一次</v-btn>
            <v-btn depressed @click="dialogAll = true">回顧一下錯的</v-btn>
          </v-row>
        </v-container>
      </v-card>
    </v-dialog>

    <v-dialog v-model="dialogAll" fullscreen transition="dialog-top-transition">
      <v-card>
        <v-container fill-height fluid>
          <v-list-item
            v-for="(check_item, index) in check_list_all"
            :key="index"
          >
            <v-list-item-content style="text-align: center">
              <v-list-item-title
                :style="
                  windowWidth < 600 ? `font-size: 14px` : `font-size: 20px`
                "
              >
                {{ check_item.answer }}
              </v-list-item-title>
            </v-list-item-content>

            <v-list-item-content style="text-align: center">
              <v-list-item-title
                :style="
                  windowWidth < 600 ? `font-size: 14px` : `font-size: 20px`
                "
              >
                {{
                  check_item.question.trans +
                  " (" +
                  check_item.question.type +
                  ")"
                }}
              </v-list-item-title>
            </v-list-item-content>
          </v-list-item>

          <v-row align="center" justify="center" class="mt-3 mb-3">
            <v-btn class="mr-3" depressed @click="tryAgain">再練一次</v-btn>
          </v-row>
        </v-container>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
import Vocabulary from "../assets/Vocabulary.json";

export default {
  name: "VocabularyPage",

  data: () => ({
    active: 0,
    dialog: false,
    dialogAll: false,

    vocabulary: Vocabulary,
    testing: [],

    // record
    score: 0,
    check_list: [],
    check_list_all: [],

    // rwd
    windowWidth: null,
  }),

  methods: {
    next(index, answer) {
      this.testing[index].your_answer = answer;
      const active = parseInt(this.active);
      this.active = active < 20 - 1 ? active + 1 : 19;
    },
    generateTesting() {
      let testing = [];
      while (testing.length !== 20) {
        let new_question = this.createQuestion();
        // if not duplicated, add created question
        if (
          testing.filter((e) => e.answer === new_question.answer).length === 0
        ) {
          new_question.num = testing.length + 1;
          testing.push(new_question);
        }
      }
      this.testing = testing;
    },
    createQuestion() {
      let random_pick_num = this.getRandomInt(this.vocabulary.length - 1, 1)[0];
      let random_pick_type = this.getRandomInt(
        this.vocabulary[random_pick_num].translation_al.length - 1,
        1
      )[0];

      // q
      let question =
        this.vocabulary[random_pick_num].translation_al[random_pick_type];
      // console.log(question.trans + " (" + question.type + ")");

      // answer
      let answer = this.vocabulary[random_pick_num].vocabulary;

      // get target letter
      let target_letter_vocabulary = this.vocabulary.filter(function (
        selected
      ) {
        return selected.vocabulary.substring(0, 1) === answer.substring(0, 1);
      });

      // get other answers
      let random_pick_others_list = this.getRandomInt(
        target_letter_vocabulary.length - 1,
        2
      );
      let random_pick_other_num1 = random_pick_others_list[0];
      let random_pick_other_num2 = random_pick_others_list[1];

      // check duplicated
      if (
        answer === target_letter_vocabulary[random_pick_other_num1].vocabulary
      ) {
        if (target_letter_vocabulary.length - 1 > random_pick_other_num1) {
          // 如果沒有超過
          random_pick_other_num1 += 1;
        } else {
          // 如果超過
          random_pick_other_num1 = -1;
        }
      } else if (
        answer === target_letter_vocabulary[random_pick_other_num2].vocabulary
      ) {
        if (target_letter_vocabulary.length - 1 > random_pick_other_num2) {
          // 如果沒有超過
          random_pick_other_num2 += 1;
        } else {
          // 如果超過
          random_pick_other_num2 = -1;
        }
      }

      let choices = [
        answer,
        target_letter_vocabulary[random_pick_other_num1].vocabulary,
        target_letter_vocabulary[random_pick_other_num2].vocabulary,
      ];
      choices = this.randomShuffle(choices);
      return {
        question: question,
        answer: answer,
        options: choices,
        your_answer: null,
      };
    },

    submit() {
      // // check all submitted
      // let check_submitted = this.testing.filter(function (question) {
      //   return question.your_answer === null;
      // });
      // if (check_submitted.length > 0) {
      //   let alert_msg = "第 ";
      //   for (let i = 0; i < check_submitted.length; i++) {
      //     let msg_num = check_submitted[i].num + ",";
      //     alert_msg = alert_msg + msg_num;
      //   }
      //   alert_msg = alert_msg.substring(0, alert_msg.length - 1);
      //   alert_msg = alert_msg + " 題<br>尚未填寫";

      //   this.$swal({
      //     icon: "info",
      //     title: "Oops...",
      //     html: alert_msg,
      //   });
      //   return;
      // }
      // submit
      let vm = this;
      let score = 0;
      let check_list = [];

      for (let i = 0; i < this.testing.length; i++) {
        if (this.testing[i].your_answer === this.testing[i].answer) {
          score += 100 / 20;
        } else {
          check_list.push(this.testing[i]);
          if (
            this.check_list_all.filter((e) => e.answer === vm.testing[i].answer)
              .length === 0
          ) {
            this.check_list_all.push(this.testing[i]);
          }
        }
      }
      this.score = score;
      this.check_list = check_list;
      this.dialog = true;
    },

    tryAgain() {
      this.generateTesting();
      this.active = 0;
      this.dialog = false;
      this.dialogAll = false;
    },

    getRandomInt(num, len) {
      let data_list = [];
      while (data_list.length !== len) {
        let data = Math.floor(Math.random() * num);
        if (!data_list.includes(data)) {
          data_list.push(data);
        }
      }
      return data_list;
    },
    randomShuffle(array) {
      for (let i = array.length - 1; i > 0; i--) {
        let j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]];
      }
      return array;
    },

    onResize() {
      this.windowWidth = window.innerWidth;
    },
  },

  created() {
    this.generateTesting();
  },
  mounted() {
    this.$nextTick(() => {
      window.addEventListener("resize", this.onResize);
    });
  },
  beforeDestroy() {
    window.removeEventListener("resize", this.onResize);
  },
};
</script>