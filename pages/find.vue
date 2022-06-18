<template>
  <v-row>
    <v-col cols="12">
      <v-autocomplete
        v-model="id"
        :items="items"
        :search-input.sync="search"
        auto-select-first
        clearable
        label="Package id"
        outlined
        @click:clear="id = null"
        @change="shelfId = null"
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
      <v-row justify="end" dense>
        <v-col cols="auto">
          <v-btn :disabled="!id || ledIsOn" outlined @click="find()">
            Find
          </v-btn>
        </v-col>
      </v-row>
    </v-col>

    <v-col v-if="shelfId" cols="12">
      <v-card>
        <v-card-title primary-title>
          Package has been found
        </v-card-title>
        <v-card-text>
          Locate the package at section <b>{{ shelfId }}</b>
        </v-card-text>
        <v-card-actions>
          <v-row justify="end" dense>
            <v-col cols="auto">
              <v-btn @click="remove()">
                Mark as removed
              </v-btn>
            </v-col>
          </v-row>
        </v-card-actions>
      </v-card>
    </v-col>

    <v-snackbar v-model="removedSnackbar">
      <v-row>
        <v-col cols="auto">
          Package has been removed
        </v-col>
        <v-spacer></v-spacer>
        <v-col cols="auto">
          <v-btn icon @click.native="removedSnackbar = false">
            <v-icon>{{ 'mdi-close' }}</v-icon>
          </v-btn>
        </v-col>
      </v-row>
    </v-snackbar>
  </v-row>
</template>

<script>
export default {
  name: 'InspirePage',
  data() {
    return {
      id: null,
      search: null,
      removedSnackbar: false,
      ledIsOn: false,
      shelfId: null,
      items: []
    }
  },

  watch: {
    id () {
      this.shelfId = null
      this.ledIsOn = false
    },
  },

  created() {
    this.getAll()
  },

  methods: {
    find () {
      this.shelfId = null
      this.$axios('http://127.0.0.1:8000/find', { params: { package_id: this.id } })
        .then(({ status, data }) => {
          if (status === 200) {
            this.shelfId = data.message[0]
            this.ledIsOn = true
          }
        })
    },
    remove () {
      this.$axios.delete('http://127.0.0.1:8000/delete', { params: { package_id: this.id } })
        .then(({ status }) => {
          if (status === 200) {
            this.getAll()
            this.shelfId = null
            this.id = null
            this.removedSnackbar = true
          }
        })
    },
    getAll () {
      this.$axios('http://127.0.0.1:8000/getall')
        .then(({ status, data }) => {
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
