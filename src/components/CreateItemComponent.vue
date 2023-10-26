<template>
    <v-card>
      <v-card-title>Create New Item</v-card-title>
      <v-card-text>
        <v-text-field v-model="newItem.name" label="Item Name"></v-text-field>
        <v-text-field v-model="newItem.description" label="Description"></v-text-field>
      </v-card-text>
      <v-card-actions>
        <v-btn @click="createItem">Create</v-btn>
      </v-card-actions>
    </v-card>
  </template>
  
  <script>
  export default {
    data() {
      return {
        newItem: {
          name: '',
          description: '',
        },
      };
    },
    methods: {
      createItem() {
        this.$emit('create-item', this.newItem);
        this.newItem = { name: '', description: '' };
      },
    },
  };
  </script>
  