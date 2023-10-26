<template>
    <v-dialog v-model="dialog" max-width="600px">
      <template v-slot:activator="{ on, attrs }">
        <slot :on="on" :attrs="attrs"></slot>
      </template>
      <v-card>
        <v-card-title>Edit Item</v-card-title>
        <v-card-text>
          <v-text-field v-model="editedItem.name" label="Item Name"></v-text-field>
          <v-text-field v-model="editedItem.description" label="Description"></v-text-field>
        </v-card-text>
        <v-card-actions>
          <v-btn @click="saveItem">Save</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </template>
  
  <script>
  export default {
    props: {
      item: Object,
    },
    data() {
      return {
        dialog: false,
        editedItem: this.item,
      };
    },
    watch: {
      item: "setItem",
    },
    methods: {
      setItem() {
        this.editedItem = { ...this.item };
      },
      async saveItem() {
        await this.$emit('save-item', this.editedItem);
        this.dialog = false;
      },
    },
  };
  </script>
  