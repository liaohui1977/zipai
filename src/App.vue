<template>
    <v-app>
        <v-app-bar app>
            <v-app-bar-nav-icon @click="openSetup"></v-app-bar-nav-icon>
            <v-toolbar-title>字牌计数</v-toolbar-title>
            <v-spacer></v-spacer>
            <v-btn icon @click="openRecord">
                <v-icon>fas fa-edit</v-icon>
            </v-btn>
            <v-btn icon @click="openRemove">
                <v-icon>fas fa-times</v-icon>
            </v-btn>
            <v-btn icon @click="openJiezhang">
                <v-icon>fas fa-check</v-icon>
            </v-btn>
            <v-btn icon @click="log2history">
                <v-icon>fas fa-times</v-icon>
            </v-btn>
        </v-app-bar>

        <!-- Sizes your content based upon application components -->
        <v-content>
            <!-- Provides the application the proper gutter -->
            <v-container fluid>
                <v-simple-table fixed-header class="table-view">
                    <thead>
                    <tr>
                        <th class="text-center">庄家</th>
                        <th v-for="(player) in players" :key="player.id" class="text-center">{{player.name}}</th>
<!--                        <th class="text-center">{{players.no1}}</th>-->
<!--                        <th class="text-center">{{players.no2}}</th>-->
<!--                        <th class="text-center">{{players.no3}}</th>-->
<!--                        <th class="text-center">{{players.no4}}</th>-->
                    </tr>
                    </thead>
                    <tbody>
                    <tr v-for="score in scores" :key="score.id">
                        <td class="text-center">{{ getZhuangName(score.zhuang) }}</td>
                        <td class="text-center">{{score.score[0]==0?'-':score.score[0]}}</td>
                        <td class="text-center">{{score.score[1]==0?'-':score.score[1]}}</td>
                        <td class="text-center">{{score.score[2]==0?'-':score.score[2]}}</td>
                        <td class="text-center">{{score.score[3]==0?'-':score.score[3]}}</td>
                    </tr>
                    </tbody>
                    <tfoot>
                    <tr>
                        <td class="text-center">合:{{scores.length}}</td>
                        <td class="text-center">{{totalScores[0].score==0?'-':totalScores[0].score}}</td>
                        <td class="text-center">{{totalScores[1].score==0?'-':totalScores[1].score}}</td>
                        <td class="text-center">{{totalScores[2].score==0?'-':totalScores[2].score}}</td>
                        <td class="text-center">{{totalScores[3].score==0?'-':totalScores[3].score}}</td>
                    </tr>
                    </tfoot>
                </v-simple-table>
            </v-container>
        </v-content>

        <v-dialog max-width="550" v-model="showSetupPlayers" persistent scrollable>
            <v-card>
                <!--对话框的标题-->
                <v-toolbar dense dark color="primary">
                    <v-toolbar-title>玩家设置</v-toolbar-title>
                    <v-spacer/>
                    <!--关闭窗口的按钮-->
                    <v-btn icon @click="closeWindow">
                        <v-icon>fas fa-times</v-icon>
                    </v-btn>
                </v-toolbar>
                <!--对话框的内容，表单-->
                <v-card-text class="px-5" style="height:400px">
                    <PlayersForm @close="closeWindow" @updatePlayers="updatePlayers" :old-players="oldPlayers"/>
                </v-card-text>
            </v-card>
        </v-dialog>
        <v-dialog max-width="550" v-model="showInputScores" persistent scrollable>
            <v-card>
                <!--对话框的标题-->
                <v-toolbar dense dark color="primary">
                    <v-toolbar-title>输入得分</v-toolbar-title>
                    <v-spacer/>
                    <!--关闭窗口的按钮-->
                    <v-btn icon @click="showInputScores=false">
                        <v-icon>fas fa-times</v-icon>
                    </v-btn>
                </v-toolbar>
                <!--对话框的内容，表单-->
                <v-card-text class="px-5" style="height:400px">
                    <InputScoresForm ref="update-score-form" @updateScores="updateScores" :scoreId="scoreId" :zhuang-prop="zhuang" :players="players"/>
                </v-card-text>
            </v-card>
        </v-dialog>
        <v-dialog max-width="550" v-model="showRemoveDialog" persistent scrollable>
            <v-card>
                <!--对话框的标题-->
                <v-toolbar dense dark color="primary">
                    <v-toolbar-title>删除得分</v-toolbar-title>
                    <v-spacer/>
                    <!--关闭窗口的按钮-->
                    <v-btn icon @click="showRemoveDialog = false">
                        <v-icon>fas fa-times</v-icon>
                    </v-btn>
                </v-toolbar>
                <!--对话框的内容，表单-->
                <v-card-text class="px-5">
                    删除最后一次记分?
                </v-card-text>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn
                            text
                            @click="showRemoveDialog = false"
                    >
                        取消
                    </v-btn>
                    <v-btn
                            color="green darken-1"
                            text
                            @click="removeScore"
                    >
                        好的
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
        <v-dialog max-width="550" v-model="showJiezhangDialog" persistent scrollable>
            <v-card>
                <!--对话框的标题-->
                <v-toolbar dense dark color="primary">
                    <v-toolbar-title>结账</v-toolbar-title>
                    <v-spacer/>
                    <!--关闭窗口的按钮-->
                    <v-btn icon @click="showJiezhangDialog=false">
                        <v-icon>fas fa-times</v-icon>
                    </v-btn>
                </v-toolbar>
                <!--对话框的内容，表单-->
                <v-card-text class="px-5" style="height:400px">
                    <JiezhangForm ref="jie-zhang-form" :total-scores="totalScores" :players="players"/>
                </v-card-text>
            </v-card>
        </v-dialog>
        <v-footer app>
            <!-- -->
        </v-footer>
    </v-app>
</template>

<script>
    // 导入自定义的表单组件
    import SetupPlayersForm from './components/setupPlayers'
    import InputScoresForm from './components/InputScores'
    import JiezhangForm from './components/JieZhang'

    let KeyOfZiPaiPlayers = "PLAYERS_e5euUPxeascl1qb@_20200520";
    let KeyOfZiPaiScores = "SCORES_e5euUPxeascl1qb@_20200520";
    let KeyOfZiPaiHistory = "History_e5euUPxeascl1qb@_20200520";

    export default {
        name: 'App',
        components: {
            PlayersForm: SetupPlayersForm,
            InputScoresForm: InputScoresForm,
            JiezhangForm: JiezhangForm
        },

        data: () => ({
            showSetupPlayers: false,
            showInputScores: false,
            showRemoveDialog: false,
            showJiezhangDialog: false,
            zhuang: 0,   //当前庄家
            totalScores: [
                {id:1,score:0}, // 玩家1
                {id:2,score:0}, // 玩家2
                {id:3,score:0}, // 玩家3
                {id:4,score:0}, // 玩家4
            ],
            players: [ {id:1,name:'1'},{id:2,name:'2'},{id:3,name:'3'},{id:4,name:'4'}], //玩家，必须是4个，隔一个为对家
            oldPlayers: null,
            scoreId: null,   //新的得分的no
            scores: [],   //得分   [{id:1,zhuang:0,score:[10,0,5,0]},...]
            histroy: [],     //历史
        }),
        created() {
            this.loadData();
            this.computeTotalScore();
            this.computeZhuang();
        },
        methods: {
            openSetup: function () {    //打开设置窗口
                this.oldPlayers = JSON.parse(JSON.stringify(this.players));
                this.showSetupPlayers = true;
            },
            openRecord: function () {   //打开记分窗口
                this.scoreId = this.scores.length + 1;
                this.showInputScores = true;
            },
            openRemove: function () {   //打开删除末行提示
                if (this.scores.length < 1) return;
                this.showRemoveDialog = true;
            },
            openJiezhang: function () {  //结账
                this.showJiezhangDialog = true;
            },
            updatePlayers: function (newPlayers) {  //更新玩家
                this.players = JSON.parse(JSON.stringify(newPlayers));
                this.saveData();
                this.closeWindow();
            },
            updateScores: function (scores) {   //更新分数
                this.scores.push(Object.assign({}, scores));
                this.saveData();
                this.closeWindow();
            },
            removeScore: function () { //删除分数
                if (this.scores.length < 1) return;
                this.scores.pop();
                this.showRemoveDialog = false;
                this.saveData();
            },
            computeTotalScore: function () { //计算总分
                this.totalScores = [
                    {id:1,score:0}, // 玩家1
                    {id:2,score:0}, // 玩家2
                    {id:3,score:0}, // 玩家3
                    {id:4,score:0}, // 玩家4
                ];
                for (let i = 0; i < this.scores.length; i++) {
                    this.totalScores[0].score += parseInt(this.scores[i].score[0]);
                    this.totalScores[1].score += parseInt(this.scores[i].score[1]);
                    this.totalScores[2].score += parseInt(this.scores[i].score[2]);
                    this.totalScores[3].score += parseInt(this.scores[i].score[3]);
                }
            },
            computeZhuang:function () {    //计算庄家
                console.log(this.scores);
                if (this.scores.length < 1) {
                    this.zhuang = 0;
                    return 0;
                }
                let score = this.scores[this.scores.length - 1];
                let scores = score.score;
                let maxScore = 0;
                let zhuang = 0;
                for (let i = 0; i < scores.length; i++) {
                    if (scores[i] < 0) {
                        zhuang = i;
                        break;
                    }
                    if (scores[i] > maxScore) {
                        maxScore = scores[i];
                        zhuang = i;
                    }
                }
                this.zhuang = zhuang;
            },
            getZhuangName: function (zhuang) {  //根据玩家id获取玩家姓名
                return this.players[zhuang].name;
            },
            closeWindow() {   // 关闭窗口
                this.showSetupPlayers = false;
                this.showInputScores = false;
            },
            saveData: function () {  //保存数据到缓存
                localStorage.setItem(KeyOfZiPaiPlayers, JSON.stringify(this.players));
                localStorage.setItem(KeyOfZiPaiScores, JSON.stringify(this.scores));
            },
            loadData: function () {  //从缓存加载数据
                Object.assign(this.players, JSON.parse(localStorage.getItem(KeyOfZiPaiPlayers)));
                Object.assign(this.scores, JSON.parse(localStorage.getItem(KeyOfZiPaiScores)));
            },
            log2history: function () {   //记录入历史
                let history = JSON.parse(localStorage.getItem(KeyOfZiPaiHistory));
                if (history == null) history = [];

                let hh = {
                    time:Date.now(),
                    players: this.players,
                    score: this.scores
                };
                history.push(hh);

                localStorage.setItem(KeyOfZiPaiHistory, JSON.stringify(history));

                this.reset();
            },
            reset(){    //重置
                this.scores = [];
                this.totalScores= [
                    {id:1,score:0}, // 玩家1
                    {id:2,score:0}, // 玩家2
                    {id:3,score:0}, // 玩家3
                    {id:4,score:0}, // 玩家4
                ];
                this.saveData();
            },
        },
        watch: {
            scores: {
                deep: true,
                immediate: true,
                handler: function () {
                    this.computeTotalScore();
                    this.computeZhuang();
                }
            }
        }
    };
</script>
