<template>
  <q-page>
    <q-select
      filled
      v-model="makeSelected"
      :options="makes"
      :option-value="opt => Object(opt) === opt && 'uuid' in opt ? opt.uuid : null"
      :option-label="opt => Object(opt) === opt && 'title' in opt ? opt.title : ''"
      :option-disable="opt => Object(opt) === opt ? opt.inactive === true : true"
      @update:model-value="
        (value) => {
          getListModels();
        }
      "
      emit-value
      map-options
      label="Make"
      style="min-width: 250px; max-width: 300px"
    />
    <q-select
      filled
      v-model="modelSelected"
      :options="models"
      :option-value="opt => Object(opt) === opt && 'value' in opt ? opt.value : null"
      :option-label="opt => Object(opt) === opt && 'label' in opt ? opt.label : ''"
      :option-disable="opt => Object(opt) === opt ? opt.inactive === true : true"
      @update:model-value="
        (value) => {
          getListYears();
        }
      "
      emit-value
      map-options
      label="Model"
      style="min-width: 250px; max-width: 300px"
    />
    <q-select
      filled v-model="yearSelected"
      :options="years"
      label="Year"
      style="min-width: 250px; max-width: 300px"
      @update:model-value="
        (value) => {
          getListVariants();
        }
      "
      />
    <q-select
      filled
      v-model="variantSelected"
      :options="variants"
      :option-value="opt => Object(opt) === opt && 'value' in opt ? opt.value : null"
      :option-label="opt => Object(opt) === opt && 'label' in opt ? opt.label : ''"
      :option-disable="opt => Object(opt) === opt ? opt.inactive === true : true"
      emit-value
      map-options
      label="Variant"
      style="min-width: 250px; max-width: 300px"
    />
    <q-btn color="primary" @click="calculate()" label="Calculate now" />
    

    <h1>Price {{priceMin}} -  {{priceMax}} $ </h1>
  </q-page>
</template>

<script>
import { defineComponent } from 'vue'
import axios from 'axios';
import { ref } from 'vue'

const slugs = [
  "922ee8e66f15e8d8",
  "c48c25e68b27c5dc",
  "c441ece68ae92c87",
  "13dc9ce6b87715b5",
  "010d10e6ad63c76d",
  "6cfa61eee6181557",
  "bde263008c819486",
  "bd9e82008c47fc67",
  "47ad62f4889522e8",
  "40f035f4854f028e",
  "ee3b69f4e77a748e",
  "54ab3fcaf3670331",
  "307e9bcade7b2948",
  "fdfeebcac24ba275",
  "7bb6f0d5d9a61819",
  "a437a2007e0f1abd",
  "b405a79b436517b3",
  "3c33d19afff0ab50",
  "d5b7f9a10e947cb7",
  "bd9a19a1915b1b0b",
  "4c6510ac22671bbd",
  "6ee89bac35e98b30",
  "7faf160068d50aef",
  "36287577a7c44f18",
  "36258d77a7c24209",
  "4301f400467370fe",
  "5a771c77bccd1480",
  "1b36f48269931fd9",
  "1b4e408269a67fde",
  "1b660b8269bab7b0",
  "09f19e825fcebcbf",
  "3bd18e827b7684d9",
  "9ac3aba14052d7ec",
  "603798829092d15b",
  "60239a8290822bb7",
  "bd317a887cf3650b",
  "e8044e58af084dcf",
  "3ef78a58e05f8866",
  "3e9fa458e0153ebe",
  "3357e9003e0c8ff4",
  "6f022c69842d19f8",
  "185c8f6953175bdf",
  "a7526f719fbf83c9",
  "60322d7176f2126d",
  "2def24715af5a27d",
  "2d9e20715ab14489",
  "58ffc33509bc1a90",
  "266fb034ed7f2b51",
  "53781447d3469482",
  "53780947d34681d1",
  "3621b6e46712b107",
  "47ad66f4889529b4",
  "abarth",
  "alfa-romeo",
  "aston-martin",
  "audi",
  "bentley",
  "bmw",
  "chery",
  "chevrolet",
  "chrysler",
  "citroen",
  "cupra",
  "dodge",
  "ferrari",
  "fiat",
  "ford",
  "ford-performance-vehicles",
  "foton",
  "great-wall",
  "gwm",
  "haval",
  "holden",
  "holden-special-vehicles",
  "honda",
  "hummer",
  "hyundai",
  "infiniti",
  "isuzu",
  "jaguar",
  "jeep",
  "kia",
  "lamborghini",
  "land-rover",
  "ldv",
  "lexus",
  "mahindra",
  "maserati",
  "mazda",
  "mclaren",
  "mercedes-benz",
  "mg",
  "mini",
  "mitsubishi",
  "morgan",
  "nissan",
  "opel",
  "peugeot",
  "polestar",
  "porsche",
  "proton",
  "ram",
  "renault",
  "rolls-royce",
  "skoda",
  "ssangyong",
  "subaru",
  "suzuki",
  "tesla",
  "toyota",
  "trd",
  "volkswagen",
  "volvo"
]

export default defineComponent({
  name: 'IndexPage',
  setup () {
    return {
      makeSelected: ref(null),
      makes: ref(null),
      slugs: slugs,
      modelSelected: ref(null),
      models: ref(null),
      yearSelected: ref(null),
      years: ref([]),
      variantSelected: ref(null),
      variants: ref(null),
      priceMin: ref(null),
      priceMax: ref(null),
    }
  },
  mounted() {
    this.getListMakes();
  },
  methods: {
    calculate() {
      const query = `
        query GET_DRIVE_VEHICLES_LIST($query: WhereOptionsVehicle) {
          driveVehicles: Vehicles(where: $query) {
            results {
              variantId: vehicleableId
              vehicleKey
              segments {
                id
                uuid
                __typename
              }
              gearType: gearTypeDescription
              driveCode
              photos {
                id
                external_id: externalId
                url
                publicId: externalId
                __typename
              }
              vehicleable {
                ... on RedbookVehicle {
                  redbookPrice {
                    tradeMin
                    tradeMax
                    privateMin
                    privateMax
                    avgKmsMax: avgKms
                    avgKmsMin: goodKms
                    avgRetail
                    __typename
                  }
                  __typename
                }
                __typename
              }
              vehicle_key: vehicleKey
              title: description
              doors: doorNum
              seats: seatCapacity
              engine: engineDescription
              gears: gearNum
              model: modelCode
              make: makeCode
              year: yearGroup
              vehicleType: vehicleTypeCode
              attributes {
                id
                longDescription
                __typename
              }
              __typename
            }
            __typename
          }
        }
      `;

      const variables = {
        "query": {
        "vehicleableUuid": {
            "eq": this.variantSelected
          }
        }
      };

      axios.post('https://drive-tailpipe-prod.graphcdn.app', {
        query: query,
        variables: variables
      }).then(response => {
        this.priceMin = response.data.data.driveVehicles.results[0].vehicleable.redbookPrice.privateMin
        this.priceMax = response.data.data.driveVehicles.results[0].vehicleable.redbookPrice.privateMax
      });
    },
    getListVariants() {
      const query = `
        query GET_TP_WMCW_VARIANTS_BY_MODEL($modelUuid: String, $year: Int) {
          model: Family(where: {uuid: $modelUuid}) {
            id
            type
            slug
            familyableId
            familyableUuid
            vehicles(where: {yearGroup: {eq: $year}}) {
              id
              type
              slug
              vehicleKey
              vehicleableId
              label: description
              value: vehicleableUuid
              bodyStyleDescription
              attributes {
                id
                longDescription
                __typename
              }
              __typename
            }
            __typename
          }
        }
      `;

      const variables = {
        modelUuid: this.modelSelected,
        year: this.yearSelected
      };

      axios.post('https://drive-tailpipe-prod.graphcdn.app', {
        query: query,
        variables: variables
      }).then(response => {
        console.log(response.data.data.model.vehicles,9999);
        this.variants = response.data.data.model.vehicles
      });
    },
    getListYears() {
      const query = `
        query GET_MODEL_YEARS($modelUuid: String) {
          model: Family(where: {uuid: $modelUuid}) {
            id
            familyableUuid
            startYear
            latestYear
            __typename
          }
        }
      `;

      const variables = { modelUuid: this.modelSelected };

      axios.post('https://drive-tailpipe-prod.graphcdn.app', {
        query: query,
        variables: variables
      }).then(response => {
        const startYear = response.data.data.model.startYear;
        const latestYear = response.data.data.model.latestYear;
        for (let i = startYear; i <= latestYear; i++) {
          this.years.push(i);
        }
      });
    },
    getListModels() {
      const query = `
        query GET_TP_WMCW_MAKE_MODELS($makeId: String, $query: WhereOptionsFamily) {
          models: Makes(where: {makeableUuid: {eq: $makeId}}) {
            results {
              id
              slug
              makeableId
              makeableUuid
              families(where: $query) {
                id
                type
                slug
                familyableId
                label: description
                value: familyableUuid
                __typename
              }
              __typename
            }
            __typename
          }
        }
      `;

      const variables = { makeId: this.makeSelected };

      axios.post('https://drive-tailpipe-prod.graphcdn.app', {
        query: query,
        variables: variables
      }).then(response => {
        this.models = response.data.data.models.results[0].families;
      });
    },
    getListMakes() {
      const query = `
        query GET_FIND_A_CAR_WP_MAKES_BY_SLUGS($slugs: [ID]) {
          vehicleMakePosts(where: {ids: $slugs, idType: SLUG}) {
            nodes {
              id
              slug
              uri
              uuid
              title
              isPopular
              __typename
            }
            __typename
          }
        }
      `;

      const variables = { slugs: this.slugs };

      axios.post('https://drive-boot.graphcdn.app', {
        query: query,
        variables: variables
      }).then(response => {
        this.makes = response.data.data.vehicleMakePosts.nodes;
      });
    }
  }
})
</script>
