<template>
    <div class="grid">
        <GridColumn
            v-for="(column, index) in columns"
            :key="index"
            :index="index"
            :dragged-index="draggedIndex"
            :fixed-dragged-index="fixedDraggedIndex"
            :column="column"
            @swapColumns="onSwapColumns"
            @draggedIndex="onDraggedIndex"
            @fixedDraggedIndex="onFixedDraggedIndex" />
    </div>
</template>

<script>
export default {
  name: 'Grid',
  components: {
    GridColumn: () => import('./GridColumn'),
  },
  props: {
    columns: {
      type: Array,
      default: () => [],
    },
  },
  data() {
    return {
      draggedIndex: -1,
      fixedDraggedIndex: -1,
    };
  },
  methods: {
    onSwapColumns({ from, to }) {
      this.$emit('swapColumns', { from, to });
    },
    onDraggedIndex(index) {
      this.draggedIndex = index;
    },
    onFixedDraggedIndex(index) {
      this.fixedDraggedIndex = index;
    },
  },
};
</script>

<style lang="scss" scoped>
    .grid {
        display: grid;
        grid-template-columns: repeat(10, 1fr);
        grid-column-gap: 12px;
        width: 1000px;
        height: 300px;
    }
</style>
