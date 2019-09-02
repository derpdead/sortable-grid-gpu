<template>
    <div :class="['column', { 'column--dragged': draggedIndex === index}]" v-text="column"
         draggable
         @dragstart="onDragStart"
         @dragend="onDragEnd"
         @dragover="onDragOver">
    </div>
</template>

<script>
export default {
  name: 'GridColumn',
  props: {
    index: {
      type: Number,
      required: true,
    },
    draggedIndex: {
      type: Number,
      default: -1,
    },
    fixedDraggedIndex: {
      type: Number,
      default: -1,
    },
    column: {
      type: Number,
      required: true,
    },
  },
  methods: {
    onDragStart(event) {
      this.createColumnElClone(event);

      this.$emit('draggedIndex', this.index);
      this.$emit('fixedDraggedIndex', this.index);
    },
    onDragEnd() {
      this.$emit('swapColumns', { from: this.draggedIndex, to: this.fixedDraggedIndex });

      this.removeColumnElClone();
      this.resetColumnsTransform();

      this.$emit('draggedIndex', -1);
      this.$emit('fixedDraggedIndex', -1);
    },
    onDragOver(event) {
      const { clientX } = event;
      const isBefore = this.determinateDraggedColumnPositionState(clientX);

      const columnFixedIndex = this.getColumnFixedIndex();

      if (this.draggedIndex === this.index
        || (isBefore && this.fixedDraggedIndex === columnFixedIndex - 1)
        || (!isBefore && this.fixedDraggedIndex === columnFixedIndex + 1)) {
        return false;
      }

      this.updateColumnsTransform(isBefore);
      this.updateFixedDraggedIndex(isBefore);

      return true;
    },
    createColumnElClone(event) {
      const { target } = event;
      const { width, height } = target.getBoundingClientRect();
      const clonedDOMElement = target.cloneNode(true);
      const clonedDOMElementStyle = `
        position: absolute;
        background-color: cadetblue;
        height: ${height}px;
        width: ${width}px;
      `;

      clonedDOMElement.setAttribute('style', clonedDOMElementStyle);
      clonedDOMElement.classList.add('cloned-column-element');
      document.body.appendChild(clonedDOMElement);
      event.dataTransfer.setDragImage(clonedDOMElement, clonedDOMElement.offsetWidth / 2, 0);
    },
    removeColumnElClone() {
      const clonedDOMElement = document.documentElement.querySelector('.cloned-column-element');
      document.body.removeChild(clonedDOMElement);
    },
    determinateDraggedColumnPositionState(clientX) {
      const { x: columnXPos, width: columnWidth } = this.$el.getBoundingClientRect();
      const normalizedHalfWidthFactor = 0.5;

      return (clientX - columnXPos) / columnWidth < normalizedHalfWidthFactor;
    },
    updateColumnsTransform(isBefore) {
      const grid = document.documentElement.querySelector('.grid');
      const draggedElTransform = +grid.children[this.draggedIndex].style.transform.replace(/[^0-9\-.,]/g, '');
      const { offsetWidth: colElWidth } = this.$el;
      const { offsetWidth: draggedColElWidth } = grid.children[this.draggedIndex];

      let newDraggedElTransform = draggedElTransform;
      let columnElTransform = 0;
      const columnElCurrentTransform = +this.$el.style.transform.replace(/[^0-9\-.,]/g, '');
      const gridGap = 12;

      if (isBefore) {
        newDraggedElTransform = draggedElTransform - colElWidth - gridGap;
        columnElTransform = 0;
        if (columnElCurrentTransform === 0) {
          columnElTransform = draggedColElWidth + gridGap;
        }
      } else {
        newDraggedElTransform = draggedElTransform + colElWidth + gridGap;
        columnElTransform = -draggedColElWidth - gridGap;
        if (columnElCurrentTransform > 0) {
          columnElTransform = 0;
        }
      }

      grid.children[this.draggedIndex].style.transform = `translateX(${newDraggedElTransform}px)`;
      this.$el.style.transform = `translateX(${columnElTransform}px)`;
    },
    updateFixedDraggedIndex(isBefore) {
      if (this.index < this.draggedIndex) {
        this.$emit('fixedDraggedIndex', isBefore ? this.index : this.index + 1);
      } else {
        this.$emit('fixedDraggedIndex', isBefore ? this.index - 1 : this.index);
      }
    },
    getColumnFixedIndex() {
      if (this.$el.style.transform) {
        const xTransform = this.$el.style.transform.replace(/[^0-9\-.,]/g, '');

        if (+xTransform) {
          if (+xTransform > 0) return this.index + 1;

          return this.index - 1;
        }
      }

      return this.index;
    },
    resetColumnsTransform() {
      const grid = document.documentElement.querySelector('.grid');
      const { length } = grid.children;

      for (let i = 0; i < length; i += 1) grid.children[i].style.transform = null;
    },
  },
};
</script>

<style lang="scss" scoped>
    .column {
        display: flex;
        align-items: center;
        justify-content: center;
        background-color: cadetblue;
        color: white;
        will-change: transform;

        &--dragged {
            background-color: cornflowerblue;
        }
    }
</style>
