<template>
  <v-row>
    <v-col cols="12">
      <v-autocomplete
        v-model="id"
        :items="items"
        :counter="true"
        :search-input.sync="search"
        auto-select-first
        clearable
        label="Package id"
        outlined
        @click:clear="id = null"
      >
        <template #no-data>
          <v-list-item>
            <v-list-item-content>
              <v-list-item-title>
                No results matching "<strong>{{ search }}</strong>"
              </v-list-item-title>
            </v-list-item-content>
          </v-list-item>
        </template>
      </v-autocomplete>
    </v-col>

    <v-col cols="12">
      <v-row
        justify="end"
        dense
      >
        <v-col cols="auto">
          <v-btn outlined @click="cancel()">
            Cancel
          </v-btn>
          <v-btn outlined @click="find()">
            Find
          </v-btn>
        </v-col>
      </v-row>
    </v-col>
  </v-row>
</template>

<script>
export default {
  name: 'InspirePage',
  data() {
    return {
      id: null,
      search: null,
      items: []
    }
  },

  watch: {
    searchKey (key) {
      // if (!key) return
      // key = key.replace(/ /g, '').toUpperCase()
      // // if (key.length === 0 && this.clearAllFields) this.onResetData()
      // if (key.length >= 5 && key !== this.previousKey) {
      //   this.searchingByKey = true
      //   this.list = []
      //   this.previousKey = key
      //   this.axios.patch(`/find-package/${key}`)
      //     .then(({ data, status }) => {
      //       if (status === 200) {
      //         data.forEach(person => {
      //           person.name = person.shortName || `${person.firstName} ${person.lastName}`
      //           person.key = person.key || person.personCode
      //           this.list.push({
      //             name: person.name,
      //             key: person.key,
      //             id: person.id,
      //             value: null
      //           })
      //         })
      //       }
      //     })
      //     .finally(() => { this.searchingByKey = false })
      // }
    },
  },

  created() {
    this.getAll()
  },

  methods: {
    find () {
      this.$axios('http://127.0.0.1:8000/find', { params: { package_id: this.id } })
    },
    getAll () {
      this.$axios('http://127.0.0.1:8000/getall')
        .then(({ status, data }) => {
          // if (status === 200) this.items = data.message
          if (status === 200) {
            data.message.forEach(item => {
              this.items.push(item.id)
            })
          }
        })
    }
  }
}
</script>
