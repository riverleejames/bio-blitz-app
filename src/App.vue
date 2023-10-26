<template>
  <v-app>
    <v-main>
      <CreateItemComponent @create-item="createItem" />

      <v-list>
        <v-list-item-group v-if="items.length > 0">
          <v-list-item v-for="item in items" :key="item._id">
            <v-list-item-content>
              <v-list-item-title>{{ item.name }}</v-list-item-title>
              <v-list-item-subtitle>{{
                item.description
              }}</v-list-item-subtitle>
            </v-list-item-content>
            <v-list-item-action>
              <v-btn icon @click="editItem(item)">
                <v-icon>mdi-pencil</v-icon>
              </v-btn>
              <v-btn icon @click="deleteItem(item._id)">
                <v-icon>mdi-delete</v-icon>
              </v-btn>
            </v-list-item-action>
          </v-list-item>
        </v-list-item-group>
        <v-list-item v-else>
          <v-list-item-content>No items found</v-list-item-content>
        </v-list-item>
      </v-list>
    </v-main>
  </v-app>
</template>

<script>
import PouchDB from "pouchdb-browser";
import CreateItemComponent from "./components/CreateItemComponent.vue";

export default {
  components: {
    CreateItemComponent,
  },
  data() {
  return {
    db: null,
    remoteCouch: false,
    items: [],
    // Other data properties
  };
},

  created() {
  this.db = new PouchDB('LocalBioBlitz');
  this.remoteCouch = 'http://admin:ca7uwaxa@localhost:5984/js_test';
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
    },
    async createItem(item) {
      try {
        item._id = new Date().toISOString();
        const response = await this.db.put(item);
        console.log("Item created successfully:", response);
      } catch (err) {
        console.error("Error creating item:", err);
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
    async editItem(item) {
      try {
        const updatedItem = await this.db.put(item);
        this.fetchItems();
        return updatedItem;
      } catch (err) {
        console.error("Error updating item:", err);
      }
    },

    async deleteItem(id) {
      try {
        const doc = await this.db.get(id);
        await this.db.remove(doc);
        this.fetchItems();
      } catch (err) {
        console.error("Error deleting item:", err);
      }
    },
  },
};
</script>
