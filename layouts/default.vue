<template>
  <v-app dark>
    <v-navigation-drawer
      v-model="drawer"
      permanent
      :mini-variant="true"
      fixed
      app
    >
      <v-list>
        <v-list-item
          v-for="(item, i) in items"
          :key="i"
          :to="item.to"
          router
          exact
        >
          <v-list-item-action>
            <v-icon>{{ item.icon }}</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title v-text="item.title" />
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>
    <v-main  style="margin-top: 40px">
      <v-container>
        <Nuxt />
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
export default {
  name: 'DefaultLayout',
  data() {
    return {
      drawer: true,
      fixed: false,
      items: [
        {
          icon: 'mdi-package-variant-closed',
          title: 'Register Package',
          to: '/register',
        },
        {
          icon: 'mdi-magnify',
          title: 'Find Package',
          to: '/find',
        },
        {
          icon: 'mdi-format-list-bulleted',
          title: 'All Packages',
          to: '/getall',
        }
      ],
      miniVariant: false,
      right: true,
      title: 'Valgus Package Management System',
    }
  },
}
</script>
