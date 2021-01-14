<template>
  <q-page padding>
    <q-btn to="/" label="Go back to search" />
    <reference-details :partnerRef="selectedReference" />
  </q-page>
</template>

<script>
import ReferenceDetails from "src/components/referenceDetails.vue";
export default {
  components: { ReferenceDetails },
  // name: 'PageName',
  data() {
    return {
      selectedReference: null,
    };
  },
  mounted() {
    if (this.$route.params.partnerRef && this.$route.params.partnerRef != "") {
      this.$axios
        .get("/elastic/_doc/" + this.$route.params.partnerRef)
        .then((response) => {
          this.selectedReference = response.data;
          this.showDetailsDialog = true;
        });
    }
  },
};
</script>
