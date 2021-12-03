<template>
  <v-app>
    <v-main>
      <HorizontalVirtualScrollTable/>
    </v-main>
  </v-app>
</template>

<script>
import HorizontalVirtualScrollTable from './components/HorizontalVirtualScrollTable';

export default {
  name: 'App',

  components: {
    HorizontalVirtualScrollTable,
  },

  data: () => ({
    //
  }),
};
</script>
