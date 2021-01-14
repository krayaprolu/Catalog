<template>
  <q-page>
    <div class="q-pa-md">
      <div class="row">
        <div style="width: 340px">
            <q-list bordered>
          <simplefacet
            v-if="this.aggregations"
            :value="this.aggregations.Industry_Vertical"
            title="Industry Vertical"
            max="5"
            :selection.sync="filters.Industry_VerticalFilter"
            :scrollMin="15"
            :open="true"
          />
          <q-separator spaced />
          <simplefacet
            v-if="this.aggregations"
            :value="this.aggregations.Partner_Tier"
            title="Partner Tier"
            max="5"
            :selection.sync="filters.Partner_TierFilter"
            :scrollMin="15"
            :alwaysAll="true"
            :allValues="['Elite', 'Premier', 'Select', 'Registered']"
            :azSort="true"
            :open="true"
          />
       
          <q-separator spaced />
          <simplefacet
            v-if="this.aggregations"
            :value="this.aggregations.WORKLOAD"
            title="Workload"
            max="5"
            :selection.sync="filters.WORKLOADFilter"
            :scrollMin="15"
            :alwaysAll="false"
            :azSort="false"
            :open="true"
          />
             <q-separator spaced />
          <simplefacet
            v-if="this.aggregations"
            :value="this.aggregations.Partner_Name"
            title="Partner Name"
            max="5"
            :selection.sync="filters.Partner_NameFilter"
            :scrollMin="15"
            :alwaysAll="true"
            :allValues="allPartners"
            :azSort="false"
            :open="false"
          />
          <q-separator spaced />
                    <simplefacet
            v-if="this.aggregations"
            :value="this.aggregations.Customer_Name"
            title="Customer Name"
            max="5"
            :selection.sync="filters.Customer_NameFilter"
            :scrollMin="15"
            :alwaysAll="true"
            :allValues="allCustomers"
            :azSort="false"
            :open="false"
          />
          <q-separator spaced />
          <simplefacet
            v-if="this.aggregations"
            :value="this.aggregations.Customer_Country"
            title="Customer Country"
            max="5"
            :selection.sync="filters.Customer_CountryFilter"
            :scrollMin="15"
            :alwaysAll="false"
            :azSort="false"
            :open="false"
          />
            </q-list>
        </div>
        <div class="col">
            <div class="row">
            <div class="col-6">
          <div class="q-pa-lg flex flex-left">
            <q-input
              bottom-slots
              v-model="textCriteria"
              @keydown.enter.prevent="refreshResults"
              label="Keywords"
              dense
              style="width: 500px"
            >
              <template v-slot:append>
                <q-icon
                  v-if="textCriteria !== ''"
                  name="close"
                  @click="
                    textCriteria = '';
                    refreshResults();
                  "
                  class="cursor-pointer"
                />
              </template>

              <template v-slot:after>
                <q-btn round dense flat icon="search" @click="refreshResults">
                  <q-tooltip> Search </q-tooltip>
                </q-btn>
                <q-separator vertical />
                <q-btn
                  round
                  dense
                  flat
                  icon="check_box_outline_blank"
                  @click="resetFilters"
                >
                  <q-tooltip> Reset all filters </q-tooltip>
                </q-btn>
              </template>
              <template v-slot:hint> Search by keywords </template>
            </q-input>
          </div>
            </div>
        
            <div class="col-6">
          <div class="row justify-left">
            <div class="col-12 justify-center">
              <q-pagination
                :max="Math.ceil(nbResults / resultsPerPage)"
                :input="true"
                v-model="currentPage"
                :min="1"
                class="justify-center"
              >
              </q-pagination>
            </div>
          </div>
          <div class="row">
            <div class="col-12">
              <center class="text-caption">{{ nbResults }} results</center>
            </div>
          </div>
            </div>
            </div>
          <div class="q-pa-md row items-start q-gutter-md" v-if="hits && hits.hits && hits.hits.length>0">
            <q-card
              v-if="hits && hits.hits"
              bordered
              class="my-card"
              style="width: 500px; height: 230px; min-width:400px"
              v-for="result of hits.hits"
              v-bind:key="result._id"
            >
              <q-card-section>
                <div class="row self-center">
                  <div class="col-9 self-center">
                    <div class="text-h6 self-center">
                      <img
                        width="50"
                        v-if="
                          result &&
                          result._source &&
                          result._source.Partner_Tier
                        "
                        class="float-left"
                        :src="
                          result._source.Partner_Tier.toLowerCase() + '.png'
                        "
                      />
                      {{ result._source.Partner_Name }}
                      <q-btn
                        round
                        dense
                        color="grey"
                        size="xs"
                        icon="more_horiz"
                        @click="showPartnerDetails(result._source.Partner_Name)"
                      >
                        <q-tooltip>Show partner profile</q-tooltip>
                      </q-btn>
                    </div>
                  </div>
                  <div class="col float-right self-center">
                    <img
                      height="40px"
                      class="float-right"
                      v-if="result && result._source && result._source.WORKLOAD"
                      :src="result._source.WORKLOAD.replace(/\s/g, '') + '.png'"
                    />
                    <img
                      v-if="
                        result && result._source && result._source.Cloud_Partner
                      "
                      height="30px"
                      class="float-right"
                      :src="
                        result._source.Cloud_Partner.replace(/\s/g, '') + '.png'
                      "
                    />
                  </div>
                </div>

                <div class="text-subtitle1">
                  {{
                    result._source && result._source.Industry_Vertical
                      ? result._source.Industry_Vertical
                      : ""
                  }}
                </div>
                <div class="row">
                  <div class="col">
                    <div class="text-subtitle2">
                      {{
                        result._source && result._source.Customer_Name
                          ? result._source.Customer_Name
                          : ""
                      }}
                         <q-btn
                        round
                        dense
                        color="grey"
                        size="xs"
                        icon="more_horiz"
                        @click="showClientDetails(result._source.Customer_Name)"
                      />
                    </div>
                  </div>
                  <div class="col">
                    <div class="text-subtitle2">
                      {{
                        result._source && result._source.Customer_Country
                          ? result._source.Customer_Country
                          : ""
                      }}
                    </div>
                  </div>
                </div>
              </q-card-section>

              <q-separator inset />

              <q-card-section class="q-pt-none " >
                <p class="shortParagraph"> {{ result._source.Project_Details }}...</p>
              </q-card-section>

              <q-card-actions style=" position: absolute;bottom:0px">
                <q-btn flat @click="URLToClipboard(result._id)"
                  >Get direct URL</q-btn
                >
                <q-btn flat @click="addToCart(result)">Add to basket</q-btn>
                <q-btn flat @click="showDetails(result)">Go to details</q-btn>
              </q-card-actions>
            </q-card>
          </div>
     

      <div class="row">
            <div class="col-6">
       
            </div>
        
            <div class="col-6">
          <div class="row justify-left">
            <div class="col-12 justify-center">
              <q-pagination
                :max="Math.ceil(nbResults / resultsPerPage)"
                :input="true"
                v-model="currentPage"
                :min="1"
                class="justify-center"
              >
              </q-pagination>
            </div>
          </div>
          <div class="row">
            <div class="col-12">
              <center class="text-caption">{{ nbResults }} results</center>
            </div>
          </div>
            </div>
            </div>
        </div>
      </div>
    </div>

    <q-dialog v-model="showDetailsDialog">
      <reference-details :partnerRef="selectedReference" />
    </q-dialog>

    <q-dialog v-model="showPartnerDialog">
      <partner-details :partnerName="selectedPartner" />
    </q-dialog>


    <q-dialog v-model="showClientDialog">
      <client-details :clientName="selectedClient" />
    </q-dialog>
  </q-page>
</template>

<script>
import simplefacet from "src/components/simplefacet.vue";
import partnerDetails from "src/components/partnerDetails.vue";
import clientDetails from "src/components/clientDetails.vue";
import { copyToClipboard } from "quasar";
import ReferenceDetails from "src/components/referenceDetails.vue";

export default {
  components: { simplefacet, ReferenceDetails, partnerDetails,clientDetails },
  name: "PageIndex",
  data() {
    return {
      reloading: false,
      hits: {},
      aggregations: null,
      textCriteria: "",
      current: 1,
      resultsPerPage: 10,
      nbResults: 0,
      allPartners: [],
      allCustomers: [],
      showDetailsDialog: false,
      showPartnerDialog: false,
      showClientDialog:false,
      selectedReference: null,
      selectedPartner: null,
      selectedClient:null,
      filters: {
        Industry_VerticalFilter: { value: [] },
        Partner_TierFilter: { value: [] },
        Partner_NameFilter: { value: [] },
        Customer_CountryFilter: { value: [] },
        WORKLOADFilter: { value: [] },
        Customer_NameFilter: { value: [] }
      },
    };
  },
  computed: {
    currentPage: {
      // getter
      get: function () {
        return this.current;
      },
      // setter
      set: function (newValue) {
        if (this.current != newValue) {
          this.current = newValue;
          this.refreshResults(true);
        }
      },
    },
  },
  methods: {
    resetFilters: function () {
      this.textCriteria = "";
      this.filters.Industry_VerticalFilter.value = [];
      this.filters.Partner_TierFilter.value = [];
      this.filters.Partner_NameFilter.value = [];
      this.filters.Customer_CountryFilter.value = [];
      this.filters.WORKLOADFilter.value = [];
      this.filters.Customer_NameFilter.value=[];
      this.refreshResults();
    },
    addToCart: function (item) {
      this.$root.$emit("addToCart", item);
    },
    showPartnerDetails(partnerName) {
      this.selectedPartner = partnerName;
      this.showPartnerDialog = true;
    },
    showClientDetails(clientName) {
      this.selectedClient = clientName;
      this.showClientDialog = true;
    },
    showDetails: function (reference) {
      this.showDetailsDialog = true;
      this.selectedReference = reference;
    },
    setQuery: function (
      query,
      Industry_VerticalFilter,
      Partner_TierFilter,
      Partner_NameFilter,
      Customer_CountryFilter,
      WORKLOADFilter,
      textCriteria,
      Customer_NameFilter
    ) {
      if (Industry_VerticalFilter && Industry_VerticalFilter.length > 0)
        query.query.bool.filter.push({
          terms: {
            "Industry_Vertical.keyword": Industry_VerticalFilter,
          },
        });

      if (Partner_TierFilter && Partner_TierFilter.length > 0)
        query.query.bool.filter.push({
          terms: {
            "Partner_Tier.keyword": Partner_TierFilter,
          },
        });

      if (Partner_NameFilter && Partner_NameFilter.length > 0)
        query.query.bool.filter.push({
          terms: {
            "Partner_Name.keyword": Partner_NameFilter,
          },
        });

      if (WORKLOADFilter && WORKLOADFilter.length > 0)
        query.query.bool.filter.push({
          terms: {
            "WORKLOAD.keyword": WORKLOADFilter,
          },
        });

      if (Customer_CountryFilter && Customer_CountryFilter.length > 0)
        query.query.bool.filter.push({
          terms: {
            "Customer_Country.keyword": Customer_CountryFilter,
          },
        });

      if (Customer_NameFilter && Customer_NameFilter.length > 0)
        query.query.bool.filter.push({
          terms: {
            "Customer_Name.keyword": Customer_NameFilter,
          },
        });

      if (textCriteria && textCriteria != "") {
        query.query.bool.must = { multi_match: {} };
        query.query.bool.must.multi_match.query = textCriteria;

        query.query.bool.must.multi_match.type = "most_fields";
        query.query.bool.must.multi_match.fields = [
          "Success_Metric",
          "Partner_Solution",
          "Project_Details",
        ];
      }
    },
    getQueryTemplate: function () {
      let query = {
        from: this.resultsPerPage * (this.current - 1),
        size: this.resultsPerPage,
        query: {
          bool: {
            filter: [],
          },
        },
        aggs: {
          Industry_Vertical: {
            terms: {
              field: "Industry_Vertical.keyword",
            },
          },
          WORKLOAD: {
            terms: {
              field: "WORKLOAD.keyword",
            },
          },
          Partner_Tier: {
            terms: {
              field: "Partner_Tier.keyword",
            },
          },

          Partner_Name: {
            terms: {
              field: "Partner_Name.keyword",
              size: 1000,
            },
          },
          Customer_Country: {
            terms: {
              field: "Customer_Country.keyword",
              size: 1000,
            },
          },
          Customer_Name: {
            terms: {
              field: "Customer_Name.keyword",
              size: 1000,
            },
          },
        },
      };
      return query;
    },
    URLToClipboard: function (id) {
      copyToClipboard("http://localhost:8080/#/partnerRef/" + id);
      this.$q.notify({
        message: "The direct access URL is copied in your clipboard",
        position: "top",
      });
    },
    criteriaChanged: function () {
      this.$root.$emit("refreshResults");
    },

    refreshResults: function (paging) {
      //console.log("refreshResults" + paging);

      if (!paging && this.current != 1) {
        this.current = 1;
      }

      let query = this.getQueryTemplate();

      this.setQuery(
        query,
        this.filters.Industry_VerticalFilter.value,
        this.filters.Partner_TierFilter.value,
        this.filters.Partner_NameFilter.value,
        this.filters.Customer_CountryFilter.value,
        this.filters.WORKLOADFilter.value,
        this.textCriteria,
        this.filters.Customer_NameFilter.value
      );

      //console.log(query);
      this.$axios.post("/elastic/_search", query).then((response) => {
        //console.log(response)
        this.hits = response.data.hits;
        this.aggregations = response.data.aggregations;
        if (response.data.hits.total.value != this.nbResults)
          this.nbResults = response.data.hits.total.value;

        if (this.allPartners == null || this.allPartners.length == 0) {
          this.allPartners = [];
          for (let p of this.aggregations.Partner_Name.buckets)
            this.allPartners.push(p.key);
        }
        this.allPartners = this.allPartners.sort();

        if (this.allCustomers == null || this.allCustomers.length == 0) {
          this.allPartners = [];
          for (let p of this.aggregations.Customer_Name.buckets)
            this.allCustomers.push(p.key);
        }
        this.allCustomers = this.allCustomers.sort();


        this.refreshFacets("Partner_Name");
        this.refreshFacets("Partner_Tier");
        this.refreshFacets("Customer_Name");
        //console.log(this.allPartners)
      });
    },
    refreshFacets: function (facetName) {
      if (this.filters[facetName + "Filter"].length == 0) return;
      let query = this.getQueryTemplate();

      this.setQuery(
        query,
        facetName == "Industry_Vertical"
          ? null
          : this.filters.Industry_VerticalFilter.value,
        facetName == "Partner_Tier"
          ? null
          : this.filters.Partner_TierFilter.value,
        facetName == "Partner_Name"
          ? null
          : this.filters.Partner_NameFilter.value,
        facetName == "Customer_Country"
          ? null
          : this.filters.Customer_CountryFilter.value,
        facetName == "WORKLOAD" ? null : this.filters.WORKLOADFilter.value,
        this.textCriteria,
         facetName == "Customer_Name" ? null : this.filters.Customer_NameFilter.value,
      );
      this.$axios.post("/elastic/_search", query).then((response) => {
        //console.log(response)

        this.aggregations[facetName] = response.data.aggregations[facetName];
      });
    },
  },

  mounted() {
    //console.log("mounted")
    this.refreshResults(false);
    this.$root.$on("refreshResults", this.refreshResults);
    if (this.$route.params.partnerRef && this.$route.params.partnerRef != "") {
      this.$axios
        .get("/elastic/_doc/" + this.$route.params.partnerRef)
        .then((response) => {
          this.showDetails(response.data);
        });
    }
  },
};
</script>
