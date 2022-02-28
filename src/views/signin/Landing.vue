<template>
  <v-app>
    <navigation :color="color" :flat="flat" />
    <v-main class="pt-0">
      <home />
      <about />
      <register />
      <feature />
      <contact />
    </v-main>
    <v-scale-transition>
      <v-btn
        fab
        v-show="fab"
        v-scroll="onScroll"
        dark
        fixed
        bottom
        right
        color="secondary"
        @click="toTop"
      >
        <v-icon>mdi-arrow-up</v-icon>
      </v-btn>
    </v-scale-transition>
    <foot />
  </v-app>
</template>

<style scoped>
    .v-main {
        background-image: url("~@/assets/img/landing/bgMain.png");
        background-attachment: fixed;
        background-position: center;
        background-size: cover;
    }
</style>

<script>
import navigation from '../../components/landing/Navigation'
import home from '../../components/landing/HomeSection'
import about from '../../components/landing/AboutSection'
import register from '../../components/landing/RegisterSection'
import feature from '../../components/landing/FeatureSection.vue'
import contact from '../../components/landing/ContactSection'
import foot from '../../components/landing/Footer'

export default {
    name:'Landing',
    components: {
        navigation,
        foot,
        home,
        about,
        feature,
        register,
        contact,
    },
    data(){
        return{
            fab: null,
            color: "",
            flat: null,
        }
    },
    watch: {
        fab(value) {
            if (value) {
                this.color = "secondary";
                this.flat = false;
            } else {
                this.color = "transparent";
                this.flat = true;
            }
        },
    },

    methods: {
        onScroll(e) {
            if (typeof window === "undefined") return;
            const top = window.pageYOffset || e.target.scrollTop || 0;
            this.fab = top > 60;
        },
        toTop() {
            this.$vuetify.goTo(0);
        },
    },
    created(){
        const top = window.pageYOffset || 0;
        if (top <= 60) {
            this.color = "transparent";
            this.flat = true;
        }
    }
}
</script>

