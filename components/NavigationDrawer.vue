<template>
  <v-navigation-drawer app>
    <v-list>
      <v-list-item to="/" router exact>
        <v-list-item-content>Home</v-list-item-content>
      </v-list-item>
      <v-list-item
        v-for="(item, i) in items"
        :key="i"
        :to="item.to"
        router
        exact
      >
        <v-list-item-content>
          <v-list-item-title v-text="item.title"></v-list-item-title>
        </v-list-item-content>
      </v-list-item>
    </v-list>
  </v-navigation-drawer>
</template>

<script>
export default {
  data() {
    return {
      items: [
        {
          title: 'Cidades',
          to: '/cidades',
        },
        {
          title: 'Estados',
          to: '/estados',
        },
      ],
    };
  },
};
</script>

<style></style>
