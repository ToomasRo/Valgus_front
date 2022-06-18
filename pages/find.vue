<template>
  <v-row>
    <v-col>
      <v-combobox
        v-model="id"
        :items="list"
        :search-input.sync="searchKey"
        :loading="searchingByKey"
        :counter="true"
        :maxlength="13"
        auto-select-first
        clearable
        label="Package id"
        hide-no-data
        outlined
        @click:clear="onResetData()"
      >
      <!-- a regex to validate the id -->
        <!-- hide-selected -->
        <!-- <template #item="data">
          <v-list-item-content>
            <v-list-item-title>{{ data.item.select }}</v-list-item-title>
          </v-list-item-content>
        </template> -->
      </v-combobox>
    </v-col>
  </v-row>
</template>

<script>
export default {
  name: 'InspirePage',
  data() {
    return {
      id: "1",
      searchKey: null,
      searchingByKey: false,
      list: ["1", "2", "3", "4", "5"]
    }
  },

  watch: {
    searchKey (key) {
      if (!key) return
      key = key.replace(/ /g, '').toUpperCase()
      // if (key.length === 0 && this.clearAllFields) this.onResetData()
      if (key.length >= 5 && key !== this.previousKey) {
        this.searchingByKey = true
        this.list = []
        this.previousKey = key
        this.axios.patch(`/api/find-package/${key}`)
          .then(({ data, status }) => {
            if (status === 200) {
              data.forEach(person => {
                person.name = person.shortName || `${person.firstName} ${person.lastName}`
                person.key = person.key || person.personCode
                this.list.push({
                  name: person.name,
                  key: person.key,
                  id: person.id,
                  value: null
                })
              })
            }
          })
          .finally(() => { this.searchingByKey = false })
      }
    },
  }
}
</script>
