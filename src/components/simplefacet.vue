<template>
 <q-expansion-item
      
        icon="filter_alt"
        :label="title + ((selection.value.length>0)?' (' + selection.value.length +' selected)':'')"
        bordered
        dense
        :value="open"
      >
  <q-list  dense>

    <q-btn
      size="xs"
      outline
      color="brown"
      label="Show all"
      v-if="
        max < value.buckets.length &&
        !(
          (showAll && value.buckets.length > scrollMin) ||
          (alwaysAll && allValues.length > scrollMin)
        )
      "
      @click="show"
    />
    <q-btn
      size="xs"
      outline
      color="brown"
      label="Show less"
      @click="show"
      v-if="
        showAll &&
        !(
          (showAll && value.buckets.length > scrollMin) ||
          (alwaysAll && allValues.length > scrollMin)
        )
      "
    />

    <div
      v-if="
        (showAll && value.buckets.length > scrollMin) ||
        (alwaysAll && allValues.length > scrollMin)
      "
    >
      <q-input
        v-model="facetFilter"
        label="Filter"
        stack-label
        dense
        style="width: 60%; margin-left: 25px"
      >
        <template v-slot:append>
          <q-icon name="close" @click="facetFilter = ''" class="cursor-pointer">
            <q-tooltip> Remove filter </q-tooltip>
          </q-icon>
        </template>
        <template v-slot:after>
          <q-icon
            name="delete"
            @click="selection.value = []"
            class="cursor-pointer"
          >
            <q-tooltip> Empty current selection </q-tooltip>
          </q-icon>
        </template>
      </q-input>
      <q-scroll-area
        style="height: 200px"
        v-if="
          (showAll && value.buckets.length > scrollMin) ||
          (alwaysAll && allValues.length > scrollMin)
        "
      >
        <q-item
          tag="label"
          v-ripple
          dense
          v-for="v of getFilteredFacets(facetFilter)"
          v-bind:key="v.key"
        >
          <q-item-section side top>
            <q-checkbox
              v-model="selection.value"
              :val="v.key"
              size="xs"
              @input="criteriaChanged"
            />
          </q-item-section>

          <q-item-section>
            <q-item-label class="text-caption"
              >{{ v.key }}
              <q-chip
                size="sm"
                :color="selection.value.indexOf(v.key) >= 0 ? 'primary' : null"
                :text-color="
                  selection.value.indexOf(v.key) >= 0 ? 'white' : null
                "
              >
                {{ v.doc_count }}
              </q-chip></q-item-label
            >
          </q-item-section>
        </q-item>
      </q-scroll-area>
    </div>

    <q-item
      tag="label"
      v-ripple
      dense
      v-for="v of getFilteredFacets('')"
      v-bind:key="v.key"
      v-if="
        !(showAll && value.buckets.length > scrollMin) &&
        !(alwaysAll && allValues.length > scrollMin)
      "
    >
      <q-item-section side top>
        <q-checkbox
          v-model="selection.value"
          :val="v.key"
          size="xs"
          @input="criteriaChanged"
        />
      </q-item-section>

      <q-item-section>
        <q-item-label class="text-caption"
          >{{ v.key }}
          <q-chip
            size="sm"
            :color="selection.value.indexOf(v.key) >= 0 ? 'primary' : null"
            :text-color="selection.value.indexOf(v.key) >= 0 ? 'white' : null"
          >
            {{ v.doc_count }}
          </q-chip></q-item-label
        >
      </q-item-section>
    </q-item>
  </q-list>
 </q-expansion-item>
</template>

<script>
export default {
  // name: 'ComponentName',
  props: [
    "value",
    "title",
    "max",
    "selection",
    "scrollMin",
    "allValues",
    "alwaysAll",
    "azSort",
    "open"
  ],
  data() {
    return {
      showAll: false,
      maxSetting: null,
      facetFilter: "",
    };
  },

  methods: {
    show: function () {
      this.showAll = !this.showAll;
      if (this.showAll) {
        if (this.defaultMax == null) this.defaultMax = this.max;
        this.max = 500;
      } else {
        this.max = this.defaultMax;
      }
    },
    getDocCountFromValue: function (key) {
      for (let v of this.value.buckets) {
        if (v.key == key) return v.doc_count;
      }
      return 0;
    },

    getFilteredFacets: function (filter) {
      if (this.alwaysAll && this.allValues.length > 0) {
        let output = [];
        for (let v of this.allValues) {
          output.push({
            key: v,
            doc_count: this.getDocCountFromValue(v),
          });
        }

        if (filter != "")
          return output.filter(
            (item) =>
              String(item.key).toLowerCase().includes(filter.toLowerCase()) ||
              this.selection.value.indexOf(item.key) >= 0
          );
        else return output;
      } else {
        if (filter != "")
          return this.value.buckets
            .slice(0, this.max)
            .filter(
              (item) =>
                String(item.key).toLowerCase().includes(filter.toLowerCase()) ||
                this.selection.value.indexOf(item.key) >= 0
            );
        else return this.value.buckets.slice(0, this.max);
      }
    },
    criteriaChanged: function () {
      console.log("refreshResults");
      this.showAll=false
      this.$root.$emit("refreshResults", "");
    },
  },
};
</script>
