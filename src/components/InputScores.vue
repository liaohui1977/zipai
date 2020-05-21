<template>
    <v-form ref="myScoresForm">
        <v-select
                v-model="zhuang"
                :items="players"
                item-text="name"
                item-value="id"
                label="庄家:"

        >
        </v-select>
<!--        <v-radio-group v-model="zhuang" row label="庄家:" dense>-->
<!--            <v-radio v-for="(player,index) in players"-->
<!--                     :key="player.id"-->
<!--                     :label="player.name"-->
<!--                     :value="index"-->
<!--            ></v-radio>-->
<!--        </v-radio-group>-->
        <v-radio-group v-model="isNoneHuPai" row label="流局?">
            <v-radio key="true" value="true" label="是"/>
            <v-radio key="false" value="false" label="否"/>
        </v-radio-group>
        <v-select
                v-model="huPaiNo"
                :items="players"
                item-text="name"
                item-value="id"
                label="胡牌:"
                :disabled="isNoneHuPai=='true'"
        >
        </v-select>
<!--        <v-radio-group v-model="huPaiNo" row label="胡牌:" :disabled="isNoneHuPai=='true'">-->
<!--            <v-radio v-for="(player,index) in players"-->
<!--                     :key="player.id"-->
<!--                     :label="player.name"-->
<!--                     :value="index"-->
<!--            ></v-radio>-->
<!--        </v-radio-group>-->
        <v-text-field label="胡牌分数"
                      type="number" inputmode="numeric" pattern="[0-9]*"
                      single-line
                      v-model="score"
                      :disabled="isNoneHuPai=='true'"
                      clearable
                      @keydown="handleInput"
        ></v-text-field>

        <v-layout row>
            <v-spacer/>
            <v-btn @click="submit" color="primary">提交</v-btn>
        </v-layout>
    </v-form>
</template>

<script>
    export default {
        name: "scores-form",
        props: {
            scoreId: null,   //牌局顺序号
            zhuangProp: null,   //庄家    0：1号，1:2号...
            players: null   //[{id: 1, name: '1'}, {id: 2, name: '2'}, {id: 3, name: '3'}, {id: 4, name: '4'}], //玩家，必须是4个，隔一个为对家
        },
        data() {
            return {
                zhuang: 0,
                isNoneHuPai: "false", //是否没人胡牌
                score: null,
                huPaiNo: 0,      //胡牌的玩家no
                scores: {
                    id: null,
                    zhuang: null,
                    score: [0, 0, 0, 0]
                }
            }
        },
        created() {
            this.zhuang = this.zhuangProp;
            this.scores.id = this.scoreId;
            this.scores.zhuang = this.zhuang;
        },
        methods: {
            submit() {
                // 表单校验
                if (this.$refs.myScoresForm.validate()) {
                    this.scores.id = this.scoreId;
                    this.$emit("updateScores", this.scores);
                    this.isNoneHuPai = "false";
                    this.huPaiNo = 0;
                    this.score = null;
                }
            },
            clear() {
                // 重置表单
                this.$refs.myScoresForm.reset();
            },
            handleInput(e) {
                let a = e.key.replace(/[^\d]/g, "");
                if (!a) {
                    e.preventDefault();
                }
            },
            computeScore() {    //计算分数
                let zhuang = this.zhuang;
                let huPaiNo = this.huPaiNo;
                let score = parseInt(this.score), xianScore = 0;
                if (score == null) score = 0;
                if ("true" == this.isNoneHuPai) {
                    huPaiNo = zhuang;
                }
                let xian = (zhuang + 2) > 3 ? (zhuang - 2) : (zhuang + 2);  //庄家的对家
                if ("true" == this.isNoneHuPai) {
                    score = -10;
                    xianScore = 10;
                } else {
                    xianScore = Math.round(score / 2);
                }
                this.scores = {
                    id: this.scoreId,
                    zhuang: this.zhuang,
                    score: [0, 0, 0, 0]
                };
                for (let i = 0; i < 4; i++) {
                    if (i == huPaiNo) {
                        this.scores.score[i] = score;
                    } else if (i == xian) {
                        this.scores.score[i] = xianScore;
                    }
                }
            }
        },
        watch: {
            id: function () {
                this.scores.id = this.id;
            },
            zhuangProp: function () {
                this.zhuang = this.zhuangProp;
            },
            zhuang: function () {
                this.computeScore();
            },
            isNoneHuPai: function () {
                this.computeScore();
            },
            huPaino: function () {
                this.computeScore();
            },
            score: function () {
                this.isNoneHuPai = "false";
                this.computeScore();
            }
        }
    }
</script>

<style scoped>

</style>
