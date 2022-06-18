<template>
  <v-row justify="center" align="center">
    <v-spacer />
    <v-col cols="8">
      <v-row>
        <v-col cols="4">
          Package id:
        </v-col>
        <v-col cols="4">
          Shelf id:
        </v-col>
        <v-col cols="4">
          Created date:
        </v-col>
      </v-row>
      <v-row v-for="(item) in items" :key="item.id">
        <v-col cols="4">
          {{ item.id }}
        </v-col>
        <v-col cols="4">
          {{ item.shelf_id }}
        </v-col>
        <v-col cols="4">
          {{ Date(item.timestamp) }}
        </v-col>
      </v-row>

    </v-col>
    <v-spacer />
  </v-row>
</template>

<script>
export default {
  name: 'GetAllPage',
  data() {
    return {
      items: [],
    }
  },

  created() {
    this.getAll()
  },

  methods: {
    getAll () {
      this.$axios('http://127.0.0.1:8000/getall')
        .then(({ status, data }) => {
          if (status === 200) this.items = data.message
        })
    }
  }

}
</script>
