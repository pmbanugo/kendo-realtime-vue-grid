<template>
  <div>
    <Grid
      ref="grid"
      :data-items="gridData"
      :edit-field="'inEdit'"
      @edit="edit"
      @remove="remove"
      @save="save"
      @cancel="cancel"
      @itemchange="itemChange"
      :columns="columns"
    >
      <GridToolbar>
        <button title="Add new" class="k-button k-primary" @click="insert">Add new</button>
        <button
          v-if="hasItemsInEdit"
          title="Cancel current changes"
          class="k-button"
          @click="cancelChanges"
        >Cancel current changes</button>
      </GridToolbar>
    </Grid>
  </div>
</template>

<script>
import Vue from "vue";
import "@progress/kendo-theme-material/dist/all.css";
import { Grid, GridToolbar } from "@progress/kendo-vue-grid";
import Hamoni from "hamoni-sync";
import DropDownCell from "./components/DropDownCell.vue";
import CommandCell from "./components/CommandCell.vue";

Vue.component("kendo-dropdown-cell", DropDownCell);
Vue.component("kendo-command-cell", CommandCell);

const primitiveName = "kendo-grid";

export default {
  name: "app",
  components: {
    Grid,
    GridToolbar
  },
  data: function() {
    return {
      columns: [
        { field: "ProductID", editable: false, title: "ID", width: "50px" },
        { field: "ProductName", title: "Name" },
        {
          field: "FirstOrderedOn",
          editor: "date",
          title: "First Ordered",
          format: "{0:d}"
        },
        {
          field: "UnitsInStock",
          title: "Units",
          width: "150px",
          editor: "numeric"
        },
        {
          field: "Discontinued",
          title: "Discontinued",
          cell: "kendo-dropdown-cell"
        },
        { cell: "kendo-command-cell", width: "180px" }
      ],
      gridData: []
    };
  },
  mounted: async function() {
    const accountId = "YOUR_ACCOUNT_ID";
    const appId = "YOUR_APP_ID";
    let hamoni;

    const response = await fetch("https://api.sync.hamoni.tech/v1/token", {
      method: "POST",
      headers: {
        "Content-Type": "application/json; charset=utf-8"
      },
      body: JSON.stringify({ accountId, appId })
    });
    const token = await response.json();
    hamoni = new Hamoni(token);

    await hamoni.connect();
    try {
      const primitive = await hamoni.get(primitiveName);
      this.listPrimitive = primitive;
      this.gridData = [...primitive.getAll()];
      this.subscribeToUpdate();
    } catch (error) {
      if (error === "Error getting state from server") this.initialise(hamoni);
      else alert(error);
    }
  },
  computed: {
    hasItemsInEdit() {
      return this.gridData.filter(p => p.inEdit).length > 0;
    }
  },
  methods: {
    itemChange: function(e) {
      Vue.set(e.dataItem, e.field, e.value);
    },
    insert() {
      const dataItem = { inEdit: true, Discontinued: false };
      this.gridData.push(dataItem);
    },
    edit: function(e) {
      Vue.set(e.dataItem, "inEdit", true);
    },
    save: function(e) {
      if (!e.dataItem.ProductID) {
        const product = { ...e.dataItem };
        delete product.inEdit;
        product.ProductID = this.generateID();

        this.gridData.pop();
        this.listPrimitive.add(product);
      } else {
        const product = { ...e.dataItem };
        delete product.inEdit;
        const index = this.gridData.findIndex(
          p => p.ProductID === product.ProductID
        );
        this.listPrimitive.update(index, product);
      }
    },
    generateID() {
      let id = 1;
      this.gridData.forEach(p => {
        if (p.ProductID) id = Math.max(p.ProductID + 1, id);
      });
      return id;
    },
    update(data, item, remove) {
      let updated;
      let index = data.findIndex(
        p =>
          JSON.stringify({ ...p }) === JSON.stringify(item) ||
          (item.ProductID && p.ProductID === item.ProductID)
      );
      if (index >= 0) {
        updated = Object.assign({}, item);
        data[index] = updated;
      }

      if (remove) {
        data = data.splice(index, 1);
      }
      return data[index];
    },
    cancel(e) {
      if (e.dataItem.ProductID) {
        Vue.set(e.dataItem, "inEdit", undefined);
      } else {
        this.update(this.gridData, e.dataItem, true);
      }
    },
    remove(e) {
      e.dataItem.inEdit = undefined;
      const index = this.gridData.findIndex(
        p =>
          JSON.stringify({ ...p }) === JSON.stringify(e.dataItem) ||
          (e.dataItem.ProductID && p.ProductID === e.dataItem.ProductID)
      );
      this.listPrimitive.remove(index);
    },
    cancelChanges(e) {
      let dataItems = this.gridData.filter(p => p.inEdit === true);

      for (let i = 0; i < dataItems.length; i++) {
        this.update(this.gridData, dataItems[i], true);
      }
    },
    initialise(hamoni) {
      hamoni
        .createList(primitiveName, [
          {
            ProductID: 1,
            ProductName: "Chai",
            UnitsInStock: 39,
            Discontinued: false,
            FirstOrderedOn: new Date(1996, 8, 20)
          }
        ])
        .then(primitive => {
          this.listPrimitive = primitive;
          this.gridData = this.listPrimitive.getAll();
          this.subscribeToUpdate();
        })
        .catch(alert);
    },
    subscribeToUpdate() {
      this.listPrimitive.onItemAdded(item => {
        this.gridData.push(item.value);
      });

      this.listPrimitive.onItemUpdated(item => {
        //update the item at item.index
        this.gridData.splice(item.index, 1, item.value);
      });

      this.listPrimitive.onItemRemoved(item => {
        //remove the item at item.index
        this.gridData.splice(item.index, 1);
      });
    }
  }
};
</script>
