<template>
  <v-app>
    <v-main>
      <CreateItemComponent @create-item="createItem" />
      <EditItemComponent ref="editItemComponent" v-if="editedItem" :item="editedItem" @save-item="editItem"
        @close-edit="editedItem = null" />

      <v-container>
        <v-row>
          <v-col v-for="item in items" :key="item._id" cols="12" md="4">
            <v-card>
              <v-card-title>Species: {{ item.species }}</v-card-title>
              <v-card-text>Total Count: {{ item.count }}</v-card-text>
              <v-card-actions>
                <v-btn icon @click="startEditing(item)">
                  <v-icon>mdi-pencil</v-icon>
                </v-btn>
                <v-btn icon @click="deleteItem(item._id)">
                  <v-icon>mdi-delete</v-icon>
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-col>
        </v-row>
        <v-row v-if="items.length === 0">
          <v-col>
            <v-card>
              <v-card-text>No items found</v-card-text>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
    <Notifications group="main" />
  </v-app>
</template>

<script>
import PouchDB from "pouchdb-browser";
import CreateItemComponent from "./components/CreateItemComponent.vue";
import EditItemComponent from "./components/EditItemComponent.vue";
import { v4 as uuidv4 } from "uuid";

export default {
  components: {
    CreateItemComponent,
    EditItemComponent,
  },
  data() {
    return {
      db: null,
      remoteCouch: false,
      items: [],
      editedItem: null,
    };
  },
  created() {
    this.db = new PouchDB("total_species_count");
    this.remoteCouch = "http://localhost:5984/total_species_count";
    this.sync();
    this.fetchItems();
  },
  methods: {
    sync() {
      const opts = { live: true };
      this.db.replicate.to(this.remoteCouch, opts, this.syncError);
      this.db.replicate.from(this.remoteCouch, opts, this.syncError);
    },
    syncError(err) {
      console.error("Database synchronization error:", err);
      this.notify("Database synchronization error", "error");
    },
    async createItem(item) {
      try {
        item._id = uuidv4();
        await this.db.put(item);
        this.fetchItems();
        this.notify("Item created successfully", "success");
      } catch (err) {
        console.error("Error creating item:", err);
        this.notify("Error creating item", "error");
      }
    },
    async fetchItems() {
      try {
        const result = await this.db.allDocs({ include_docs: true });
        this.items = result.rows.map((row) => row.doc);
      } catch (err) {
        console.error("Error fetching items:", err);
      }
    },
    startEditing(item) {
      this.editedItem = { ...item };
      this.$nextTick(() => {
        if (this.$refs.editItemComponent) {
          this.$refs.editItemComponent.dialog = true;
        } else {
          console.error(
            "EditItemComponent is not mounted or ref is not set correctly"
          );
        }
      });
    },
    editItem(item) {
      this.db
        .put(item)
        .then(() => {
          this.fetchItems();
          this.editedItem = null;
          this.notify("Item updated successfully!", "success");
        })
        .catch((err) => {
          console.error("Error updating item:", err);
          this.notify("Error updating item", "error");
        });
    },
    async deleteItem(id) {
      try {
        const doc = await this.db.get(id);
        await this.db.remove(doc);
        this.fetchItems();
        this.notify("Item deleted successfully!", "success");
      } catch (err) {
        console.error("Error deleting item:", err);
        this.notify("Error deleting item", "error");
      }
    },
    notify(message, type) {
      this.$notify({
        group: "main",
        title: type.charAt(0).toUpperCase() + type.slice(1),
        text: message,
        type: type,
      });
    },
  },
};
</script>

<style scoped>
/* Add your styles here */
</style>
