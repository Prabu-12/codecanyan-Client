<template>
    <v-footer padless light :color="getFooterColor">
        <v-col class="text-right font-weight-regular">
            {{footerText}}
        </v-col>
    </v-footer>
</template>

<script>
export default {
    name:'Footer',
    computed:{
        getFooterColor:function(){
            return this.$store.getters['settings/footerColor']
        },
        footerText:function(){
            return this.$store.getters['settings/footerText']
        }
    },
}
</script>

<style scoped>

</style>