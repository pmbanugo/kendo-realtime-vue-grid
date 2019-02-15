<template>
  <td v-if="dataItem && !dataItem['inEdit']">
    <button class="k-primary k-button k-grid-edit-command" @click="editHandler">Edit</button>
    <button class="k-button k-grid-remove-command" @click="removeHandler">Remove</button>
  </td>
  <td v-else>
    <button
      class="k-button k-grid-save-command"
      @click="addUpdateHandler"
    >{{this.dataItem.ProductID? 'Update' : 'Add'}}</button>
    <button
      class="k-button k-grid-cancel-command"
      @click="cancelDiscardHandler"
    >{{this.dataItem.ProductID? 'Cancel' : 'Discard'}}</button>
  </td>
</template>

<script>
export default {
  name: "CommandCell",
  props: {
    field: String,
    dataItem: Object,
    format: String,
    className: String,
    columnIndex: Number,
    columnsCount: Number,
    rowType: String,
    level: Number,
    expanded: Boolean,
    editor: String
  },
  methods: {
    onClick: function(e) {
      this.$emit("change", e, this.dataItem, this.expanded);
    },
    editHandler: function() {
      this.$emit("edit", this.dataItem);
    },
    removeHandler: function() {
      this.$emit("remove", this.dataItem);
    },
    addUpdateHandler: function() {
      this.$emit("save", this.dataItem);
    },
    cancelDiscardHandler: function() {
      this.$emit("cancel", this.dataItem);
    }
  }
};
</script>
