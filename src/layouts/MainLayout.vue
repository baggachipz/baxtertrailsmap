<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar class="bg-blue-grey">
        <q-btn
          flat
          dense
          round
          icon="menu"
          aria-label="Menu"
          @click="toggleLeftDrawer"
        />

        <q-toolbar-title>Baxter Trail Map</q-toolbar-title>
        <q-icon size="large" name="hiking"></q-icon>&nbsp;
        <q-icon size="large" name="pedal_bike"></q-icon>&nbsp;
        <q-icon size="large" name="forest"></q-icon>
      </q-toolbar>
    </q-header>

    <q-drawer v-model="leftDrawerOpen" show-if-above bordered>
      <q-img src="~assets/trail-club-logo.svg" alt="Baxter Trail Club"></q-img>
      <q-list>
        <q-item clickable @click="reportProblem">
          <q-item-section avatar>
            <q-icon name="warning" />
          </q-item-section>
          <q-item-section>
            <q-item-label>Report a Problem</q-item-label>
            <q-item-label caption>Downed tree? Broken sign?</q-item-label>
          </q-item-section>
        </q-item>
        <q-item
          clickable
          tag="a"
          target="_blank"
          href="https://www.facebook.com/BaxterTrailClub"
        >
          <q-item-section avatar>
            <q-icon name="facebook" />
          </q-item-section>
          <q-item-section>
            <q-item-label>Facebook</q-item-label>
            <q-item-label caption>Connect with us</q-item-label>
          </q-item-section>
        </q-item>
        <q-item
          clickable
          tag="a"
          target="_blank"
          href="https://www.baxtertrailclub.org"
        >
          <q-item-section avatar>
            <q-icon name="group_add" />
          </q-item-section>
          <q-item-section>
            <q-item-label>Become a Member</q-item-label>
            <q-item-label caption>We need your help!</q-item-label>
          </q-item-section>
        </q-item>
      </q-list>
    </q-drawer>

    <q-page-container>
      <router-view @location="onLocation" />
    </q-page-container>
  </q-layout>
</template>

<script>
import ReportProblem from "src/components/ReportProblem.vue";

export default {
  name: "MainLayout",
  methods: {
    toggleLeftDrawer() {
      this.leftDrawerOpen = !this.leftDrawerOpen;
    },
    reportProblem() {
      this.$q.dialog({
        component: ReportProblem,
        componentProps: {
          location: this.location,
        },
      });
    },
    onLocation(latlng) {
      this.location = latlng;
    },
  },
  data() {
    return {
      leftDrawerOpen: false,
      location: null,
    };
  },
};
</script>
