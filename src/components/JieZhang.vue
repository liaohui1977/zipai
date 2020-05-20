<template>
    <v-form ref="myJieZhangForm">
        <v-radio-group v-model="dw" label="单位:" row>
            <v-radio label="0.5" value="0.5"/>
            <v-radio label="1" value="1"/>
        </v-radio-group>
        <v-simple-table>
            <thead>
            <tr>
                <td class="text-center">玩家</td>
                <td class="text-center">得分</td>
                <td class="text-center">金额</td>
            </tr>
            </thead>
            <tbody v-for="(item) in win" :key="item.id">
            <tr>
                <td class="text-center">{{item.name}}</td>
                <td class="text-center">{{item.score}}</td>
                <td class="text-center">{{item.money}}</td>
            </tr>
            </tbody>
        </v-simple-table>
    </v-form>
</template>

<script>
    export default {
        name: "jiezhang-form",
        props: {
            players: null,  //[ {id:1,name:'1'},{id:2,name:'2'},{id:3,name:'3'},{id:4,name:'4'}], //玩家，必须是4个，隔一个为对家
            totalScores: null
        },
        data() {
            return {
                dw: "0.5", //每分5毛，
                win: [
                    {id: 0, name: '', score: 0, money: 0}, // 玩家1的名称、得分和金额
                    {id: 0, name: '', score: 0, money: 0}, // 玩家1的名称、得分和金额
                    {id: 0, name: '', score: 0, money: 0}, // 玩家1的名称、得分和金额
                    {id: 0, name: '', score: 0, money: 0}, // 玩家1的名称、得分和金额
                ]
            }
        },
        methods: {
            doJieZhang: function () {  //结账计算
                let dw = this.dw;   //每分金额
                if(dw<0||dw==null) dw=0
                let win = [0, 0, 0, 0];
                let scores = [
                    Math.round(this.totalScores[0].score / 10) * 10,
                    Math.round(this.totalScores[1].score / 10) * 10,
                    Math.round(this.totalScores[2].score / 10) * 10,
                    Math.round(this.totalScores[3].score / 10) * 10];

                for (let i = 0; i < scores.length; i++) {
                    let score = scores[i];
                    for (let j = 0; j < scores.length; j++) {
                        if (i == j) continue;
                        win[i] += score - scores[j];
                    }
                }
                //计算输赢得分合计
                this.win = [];
                for (let i = 0; i < win.length; i++) {
                    this.win.push({
                        id: i,
                        name: this.players[i].name,
                        score: win[i],
                        money: win[i] * dw
                    });
                }
            }
        },
        watch: {
            dw:function(){
                this.doJieZhang();
            },
            totalScores: {
                deep: true,
                immediate: true,
                handler: function () {
                    this.doJieZhang();
                }
            }
        }
    }
</script>

<style scoped>

</style>
