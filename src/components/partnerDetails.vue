<template>
  <q-card
    bordered
    class="my-card"
    style="width: 900px; max-width: 80vw; height: 500px"
  >
    <q-card-section>
      <div class="row">
        <div class="col-8">
          <div class="text-h6">Partner profile from references</div>
        </div>
      </div>
    </q-card-section>
    <q-card-section>
      <div class="row">
        <div class="col-8">
          <div class="text-h6">
            <img
              width="50"
              v-if="partnerTier"
              class="float-left"
              :src="partnerTier.toLowerCase() + '.png'"
            />
            {{ partnerName }}
          </div>
        </div>
      </div>
    </q-card-section>
    <q-card-section>
      <div class="row">
        <div class="col">
          <echarts
            v-if="showStats"
            :option="getChartOption()"
            :height="300"
            :width="300"
          ></echarts>
        </div>
      </div>
    </q-card-section>
     
           <q-banner rounded class="text-white" style="position:absolute;bottom:0px;background:#d45b90">
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
  </q-card>
</template>

<script>
import echarts from "src/components/echarts.vue";
export default {
  // name: 'ComponentName',
  components: { echarts },
  data() {
    return {
      selectedPartner: {
        name: null,
        data: {},
      },

      showStats: false,
      partnerTier: null,
    };
  },
  methods: {
    getChartOption: function () {
      let option = {
        tooltip: {
          trigger: "item",
          formatter: "{a} <br/>{b}: {c} ({d}%)",
        },
        title: [
          {
            left: "center",
            left: "7%",
            top: "10%",
            text: "Workloads",
          },
          {
            left: "50%",
            top: "10%",
            left: "center",
            text: "Industry Verticals",
          },
          {
            left: "73%",
            top: "10%",

            text: "Customer Countries",
          },
        ],
        series: [
          {
            name: "Industry Vertical",
            type: "pie",
            radius: ["20%", "50%"],
            center: ["50%", "50%"],
            avoidLabelOverlap: false,

            itemStyle: {
              borderRadius: 10,
              borderColor: "#fff",
              borderWidth: 2,
            },
            label: {
              show: true,
              position: "center",
            },
            emphasis: {
              label: {
                show: true,
                fontSize: "20",
                fontWeight: "bold",
              },
            },
            labelLine: {
              show: false,
            },
            data: this.selectedPartner.data["Industry_Vertical"],
          },
          {
            name: "Workload",
            type: "pie",

            radius: ["20%", "50%"],
            center: ["15%", "50%"],
            avoidLabelOverlap: false,
            itemStyle: {
              borderRadius: 10,
              borderColor: "#fff",
              borderWidth: 2,
            },
            label: {
              show: true,
              position: "center",
            },
            emphasis: {
              label: {
                show: true,
                fontSize: "20",
                fontWeight: "bold",
              },
            },
            labelLine: {
              show: false,
            },
            data: this.selectedPartner.data["WORKLOAD"],
          },
          ,
          {
            name: "Customer countries",
            type: "pie",

            radius: ["20%", "50%"],
            center: ["85%", "50%"],
            avoidLabelOverlap: false,
            itemStyle: {
              borderRadius: 10,
              borderColor: "#fff",
              borderWidth: 2,
            },
            label: {
              show: true,
              position: "center",
            },
            emphasis: {
              label: {
                show: true,
                fontSize: "20",
                fontWeight: "bold",
              },
            },
            labelLine: {
              show: false,
            },
            data: this.selectedPartner.data["Customer_Country"],
          },
        ],
      };

      return option;
    },
    getQueryTemplate: function () {
      let query = {
        from: 0,
        size: 50,
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
        },
      };
      return query;
    },

    setQuery: function (
      query,
      Industry_VerticalFilter,
      Partner_TierFilter,
      Partner_NameFilter,
      Customer_CountryFilter,
      WORKLOADFilter,
      textCriteria
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
    getPartnerStats(partnerName) {
      let query = this.getQueryTemplate();

      this.setQuery(query, null, null, [partnerName], null, null, null);
      console.log(query);
      this.$axios.post("/elastic/_search", query).then((response) => {
        this.selectedPartner.name = partnerName;
        this.partnerTier =
          response.data.aggregations["Partner_Tier"].buckets[0].key;

        for (let dimension of [
          "Industry_Vertical",
          "WORKLOAD",
          "Customer_Country",
        ]) {
          this.selectedPartner.data[dimension] = response.data.aggregations[
            dimension
          ].buckets.map((item) => {
            let res = {
              value: item.doc_count,
              name: item.key,
            };
            return res;
          });
        }
        //
        this.showStats = true;
      });
    },
  },
  props: ["partnerName"],
  mounted() {
    this.getPartnerStats(this.partnerName);
  },
};
</script>
