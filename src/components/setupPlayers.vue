<template>
    <v-form ref="myPlayersForm">
        <v-text-field v-model="players[0].name" label="1号玩家" required />
        <v-text-field v-model="players[1].name" label="2号玩家" required />
        <v-text-field v-model="players[2].name" label="3号玩家" required />
        <v-text-field v-model="players[3].name" label="4号玩家" required />
        <v-layout row>
            <v-spacer/>
            <v-btn @click="submit" color="primary">提交</v-btn>
            <v-btn @click="clear">重置</v-btn>
        </v-layout>
    </v-form>
</template>

<script>
    export default {
        name: "player-form",
        props: {
            oldPlayers: null
        },
        data() {
            return {
                players: [ {id:1,name:'1'},{id:2,name:'2'},{id:3,name:'3'},{id:4,name:'4'}], //玩家，必须是4个，隔一个为对家
            }
        },
        created() {
            this.players = this.oldPlayers;
        },
        methods: {
            submit() {
                // 表单校验
                if (this.$refs.myPlayersForm.validate()) {
                    this.$emit("updatePlayers",this.players);
                }
            },
            clear() {
                // 重置表单
                this.$refs.myPlayersForm.reset();
            }
        },
        watch:{
            oldPlayers: {
                immediate:true,
                handler:function (newValue) {
                    this.players = newValue;
                }
            }
        }
    }
</script>

<style scoped>

</style>
