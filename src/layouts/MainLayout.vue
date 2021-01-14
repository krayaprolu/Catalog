<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <q-btn
          flat
          dense
          round
          icon="menu"
          aria-label="Menu"
          @click="leftDrawerOpen = !leftDrawerOpen"
        />

        <q-toolbar-title>
          <q-img src="snowflake-logo-white.png" width="10%" />
          Partner Catalog
        </q-toolbar-title>

        <q-btn dense round flat icon="shopping_cart">
          <q-badge color="red" floating transparent>
            {{ cart.length }}
          </q-badge>
          <q-menu>
            <div class="row no-wrap q-pa-md items-center">
              <div class="column">
                <div class="text-h6 q-mb-md">Share direct links</div>
                <q-toggle
                  v-model="exportSettings.clientName"
                  label="Add client name"
                />
                <q-toggle
                  v-model="exportSettings.partnerName"
                  label="Add partner name"
                />
                <q-toggle
                  v-model="exportSettings.partnerTier"
                  label="Add partner tier"
                />
                <q-toggle
                  v-model="exportSettings.description"
                  label="Add description"
                />
                <q-toggle v-model="exportSettings.url" label="Add URL" />
              </div>

              <q-separator vertical inset class="q-mx-lg" />

              <div class="column items-center">
                {{ cart.length }} References
                <q-btn
                  color="primary"
                  label="Copy to Clipboard"
                  push
                  size="sm"
                  v-close-popup
                  @click="toClipboard"
                />
                <center>
                  This feature is for <br /><u> internal sharing only.</u>
                </center>
              </div>
            </div>
          </q-menu>
        </q-btn>

        <div>Snowflake Partner Network</div>
      </q-toolbar>
    </q-header>

    <q-drawer v-model="leftDrawerOpen" bordered content-class="bg-grey-1">
      <q-list>
        <q-item-label header class="text-grey-8">
          Essential Links
        </q-item-label>
        <EssentialLink
          v-for="link in essentialLinks"
          :key="link.title"
          v-bind="link"
        />
      </q-list>
    </q-drawer>

    <q-page-container>
      <router-view />
    </q-page-container>

    <q-footer elevated>
      <q-banner rounded class="text-white" style="background:#d45b90">
        <template v-slot:avatar>
          <q-icon name="warning" color="white" />
          <div class="text-overline">BE AWARE</div>
        </template>
        <center>
          All information in this portal is considered Snowflake Proprietary and
          may not be shared externally. Any violations are subject to
          disciplinary action
        </center>
      </q-banner>
    </q-footer>
  </q-layout>
</template>

<script>
import EssentialLink from "components/EssentialLink.vue";
import { copyToClipboard } from "quasar";

const linksData = [
  {
    title: "SPN",
    caption: "The Snowflake Partner Network",
    icon: "school",
    link: "https://spn.snowflake.com",
  },
];

export default {
  name: "MainLayout",
  components: { EssentialLink },
  data() {
    return {
      leftDrawerOpen: false,
      essentialLinks: linksData,
      cart: [],
      exportSettings: {
        clientName: true,
        partnerName: true,
        partnerTier: true,
        description: true,
        url: true,
      },
    };
  },
  methods: {
    addReference: function (newItem) {
      if (this.cart.filter((item) => item._id == newItem._id).length == 0)
        this.cart.push(newItem);
    },
    toClipboard: function () {
      let results = `*************************************************************
*  All information in this portal is considered             
*  Snowflake Proprietary and may not be shared externally.  
*                                                           
*  Any violations are subject to disciplinary action        
*************************************************************
      
      `
      
      
      for (let selection of this.cart) {
        let result = "";
        let partnerLine = [];
        let clientLine = [];
        if (
          this.exportSettings.partnerName &&
          selection._source.Partner_Name != ""
        )
          partnerLine.push(selection._source.Partner_Name);
        if (
          this.exportSettings.partnerTier &&
          selection._source.Partner_Tier != ""
        )
          partnerLine.push("[" + selection._source.Partner_Tier + "]");
        result = partnerLine.join(" ");
        if (result != "") result += "\n";

        if (
          this.exportSettings.clientName &&
          selection._source.Customer_Name != ""
        )
          clientLine.push(selection._source.Customer_Name);
        if (clientLine.length > 0) {
          result += clientLine.join(" - ");
          result += "\n";
        }

        if (
          this.exportSettings.description &&
          selection._source.Project_Details != ""
        )
          result += selection._source.Project_Details + "\n";
        if (this.exportSettings.url)
          result += "http://localhost:8080/#/partnerRef/" + selection._id;

        results += result + "\n\n";
      }

      copyToClipboard(results);
      this.$q.notify({
        message: "The direct access URL is copied in your clipboard",
        position: "top",
      });
    },
  },
  mounted() {
    this.$root.$on("addToCart", this.addReference);
  },
};
</script>
