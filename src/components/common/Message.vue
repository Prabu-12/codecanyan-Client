<template>
    <v-snackbar v-if="msgLong==false" v-model="snackbar" :timeout="timeout" top>
        <span>{{msg}}</span>
        <template v-slot:action="{ attrs }">
            <v-btn color="blue" text v-bind="attrs" @click="snackbar=false">close</v-btn>
        </template>
    </v-snackbar>
    <v-snackbar v-else-if="msgLong==true" v-model="snackbar" :timeout="timeout" top>
        <span>{{msg}}</span>
        <template v-slot:action="{ attrs }">
            <v-btn color="blue" text v-bind="attrs" @click="snackbar=false">close</v-btn>
        </template>
    </v-snackbar>
</template>

<script>
export default {
    name:'Message',
    data(){
        return{
            snackbar: false,
            msg: '',
            timeout: -1,
            msgLong:false
        }
    },
    mounted(){
        this.$root.$on('message_from_parent',(msg)=>{
            this.snackbar=true
            this.timeout=2000
            this.msg=msg
            this.msgLong=false
        })

        this.$root.$on('message_from_parent_long',(msg)=>{
            this.snackbar=true
            this.timeout=10000
            this.msg=msg
            this.msgLong=true
        })
    }
}
</script>