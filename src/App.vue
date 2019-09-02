<template>
  <div id="app">
    <div class="page-wrapper">
        <Grid
            :columns="columns"
            @removeColumn="onRemoveColumnAtIndex"
            @swapColumns="onSwapColumns" />
    </div>
  </div>
</template>

<script>

import { swapItemPosition } from './model/arrayWrapper';

export default {
  name: 'app',
  components: {
    Grid: () => import('./components/Grid'),
  },
  data() {
    return {
      columns: [1, 2, 3, 4, 5, 6, 7, 8, 9],
    };
  },
  methods: {
    onRemoveColumnAtIndex(index) {
      this.columns.splice(index, 1);
    },
    onSwapColumns({ from, to }) {
      this.columns = swapItemPosition(this.columns, from, to);
    },
  },
};
</script>

<style lang="scss" scoped>
    #app {
        width: 100vw;
        height: 100vh;
    }

    .page-wrapper {
        display: flex;
        align-items: center;
        justify-content: center;
        width: 100%;
        height: 100%;
    }
</style>
