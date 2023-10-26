<template>
    <v-dialog v-model="dialog" max-width="600px">
      <template v-slot:activator="{ on, attrs }">
        <slot :on="on" :attrs="attrs"></slot>
      </template>
      <v-card>
        <v-card-title>Edit Item</v-card-title>
        <v-card-text>
          <v-text-field 
            v-model="editedItem.name" 
            label="Item Name" 
            :error-messages="nameErrors" 
            @input="validateName"
            required
          ></v-text-field>
          <v-text-field 
            v-model="editedItem.description" 
            label="Description" 
            :error-messages="descriptionErrors" 
            @input="validateDescription"
            required
          ></v-text-field>
        </v-card-text>
        <v-card-actions>
          <v-btn text @click="closeDialog">Cancel</v-btn>
          <v-spacer></v-spacer>
          <v-btn :loading="saving" :disabled="saving" @click="saveItem">Save</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </template>
  
  <script>
  export default {
    props: {
      item: {
        type: Object,
        default: () => ({})
      }
    },
    data() {
      return {
        dialog: false,
        editedItem: {},
        saving: false,
        nameErrors: [],
        descriptionErrors: []
      };
    },
    watch: {
      item: {
        immediate: true,
        handler: "initEditedItem"
      }
    },
    methods: {
      initEditedItem() {
        this.editedItem = { ...this.item };
      },
      validateName() {
        this.nameErrors = [];
        if (!this.editedItem.name) {
          this.nameErrors.push("Name is required");
        }
      },
      validateDescription() {
        this.descriptionErrors = [];
        if (!this.editedItem.description) {
          this.descriptionErrors.push("Description is required");
        }
      },
      async saveItem() {
        if (this.validate()) {
          this.saving = true;
          try {
            await this.$emit('save-item', this.editedItem);
            await this.$nextTick();
            this.dialog = false;
          } catch (error) {
            console.error("Error saving item:", error);
            this.$notify({ 
              group: 'main', 
              title: 'Error',
              text: 'Failed to save item. Please try again.',
              type: 'error'
            });
          } finally {
            this.saving = false;
          }
        }
      },
      validate() {
        this.validateName();
        this.validateDescription();
        return this.nameErrors.length === 0 && this.descriptionErrors.length === 0;
      },
      closeDialog() {
        this.dialog = false;
        this.$emit('close');
      }
    }
  };
  </script>
  