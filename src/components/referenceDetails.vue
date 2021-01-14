<template>
  <q-card
    bordered
    class="my-card"
    style="width: 900px; max-width: 80vw; height: 700px; max-height: 1024px"
  >
    <q-card-section>
      <div class="row">
        <div class="col-8">
          <div class="text-h6">
            <img
              width="50"
              v-if="
                partnerRef &&
                partnerRef._source &&
                partnerRef._source.Partner_Tier
              "
              class="float-left"
              :src="partnerRef._source.Partner_Tier.toLowerCase() + '.png'"
            />
            {{ partnerRef._source.Partner_Name }}
          </div>
        </div>
        <div class="col float-right">
          <img
            height="50px"
            class="float-right"
            v-if="
              partnerRef && partnerRef._source && partnerRef._source.WORKLOAD
            "
            :src="partnerRef._source.WORKLOAD.replace(/\s/g, '') + '.png'"
          />
          <img
            v-if="
              partnerRef &&
              partnerRef._source &&
              partnerRef._source.Cloud_Partner
            "
            height="50px"
            class="float-right"
            :src="partnerRef._source.Cloud_Partner.replace(/\s/g, '') + '.png'"
          />
        </div>
      </div>

      <div class="text-subtitle1">
        {{
          partnerRef._source && partnerRef._source.Industry_Vertical
            ? partnerRef._source.Industry_Vertical
            : ""
        }}
      </div>
      <div class="row">
        <div class="col">
          <div class="text-subtitle2">
            {{
              partnerRef._source && partnerRef._source.Customer_Name
                ? partnerRef._source.Customer_Name
                : ""
            }}
          </div>
        </div>
        <div class="col">
          <div class="text-subtitle2">
            {{
              partnerRef._source && partnerRef._source.Customer_Country
                ? partnerRef._source.Customer_Country
                : ""
            }}
          </div>
        </div>
      </div>
    </q-card-section>

    <q-separator inset />

    <q-card-section class="q-pt-none" style="height: 100px">
      {{ partnerRef._source.Project_Details }}
    </q-card-section>

    <q-separator inset />

    <q-card-section class="q-pt-none" style="height: 100px">
      <div class="row">
        <div class="col">
          <q-item-label header>Problems Before Snowflake</q-item-label>

          <q-scroll-area style="height: 300px; max-width: 300px">
            <q-list dense padding class="rounded-borders">
              <q-item
                clickable
                v-ripple
                v-for="item of getlistItems(
                  partnerRef._source.Customer_Problems_Before_Snowflake
                )"
                v-bind:key="item"
              >
                <q-item-section>
                  {{ item }}
                </q-item-section>
              </q-item>
            </q-list>
          </q-scroll-area>
        </div>
        <div class="col">
          <q-item-label header>Success Metric</q-item-label>
          <q-scroll-area style="height: 300px; max-width: 300px">
            <q-list dense padding class="rounded-borders">
              <q-item
                clickable
                v-ripple
                v-for="item of getlistItems(partnerRef._source.Success_Metric)"
                v-bind:key="item"
              >
                <q-item-section>
                  {{ item }}
                </q-item-section>
              </q-item>
            </q-list>
          </q-scroll-area>
        </div>
        <div class="col">
          <q-item-label header>Partner Solution</q-item-label>
          <q-scroll-area style="height: 300px; max-width: 300px">
            <q-list dense padding class="rounded-borders">
              <q-item
                clickable
                v-ripple
                v-for="item of getlistItems(
                  partnerRef._source.Partner_Solution
                )"
                v-bind:key="item"
              >
                <q-item-section>
                  {{ item }}
                </q-item-section>
              </q-item>
            </q-list>
          </q-scroll-area>
        </div>
      </div>
   
    </q-card-section>


 
           <q-banner rounded class="text-white" style="position:absolute;bottom:0px;background:#d45b90" >
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
export default {
  // name: 'ComponentName',
  props: ["partnerRef"],

  data() {
    return {};
  },
  methods: {
    getlistItems: function (text) {
      if (text)
        return text
          .replace(/[\\n\s][*\-]/g, "---")
          .replace(/\\n/g, "")
          .replace(/\d\)\s*/g, "---")
          .replace(/\d\.\s*/g, "---")
          .split("---")
          .filter((item) => item != "");
      else return [];
    },
  },
};
</script>
