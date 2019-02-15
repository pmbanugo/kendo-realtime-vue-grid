<template>
  <td v-if="dataItem && !dataItem.inEdit" :class="className">{{ dataItem[field]}}</td>
  <td v-else>
    <select class="k-textbox" @change="change">
      <option>True</option>
      <option>False</option>
    </select>
  </td>
</template>

<script>
export default {
  name: "DropDownCell",
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
    change(e) {
      this.$emit("change", e, e.target.value);
    }
  }
};
</script>
