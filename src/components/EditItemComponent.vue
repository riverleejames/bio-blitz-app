<template>
    <v-dialog v-model="dialog" max-width="600px">
        <template v-slot:activator="{ on, attrs }">
            <slot :on="on" :attrs="attrs"></slot>
        </template>
        <v-card>
            <v-card-title>Edit Item</v-card-title>
            <v-card-text>
                <v-text-field v-model="editedItem.species" label="Species" :error-messages="speciesErrors" @input="validateSpecies"
                    required></v-text-field>
                <v-text-field v-model="editedItem.count" label="Count" :error-messages="countErrors" @input="validateCount"
                    required></v-text-field>
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
            speciesErrors: [],
            countErrors: []
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
        validateSpecies() {
            this.speciesErrors = [];
            if (!this.editedItem.species) {
                this.speciesErrors.push("Species is required");
            }
        },
        validateCount() {
            this.countErrors = [];
            if (!this.editedItem.count && this.editedItem.count !== 0) {
                this.countErrors.push("Count is required");
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
            this.validateSpecies();
            this.validateCount();
            return this.speciesErrors.length === 0 && this.countErrors.length === 0;
        },
        closeDialog() {
            this.dialog = false;
            this.$emit('close');
        }
    }
};
</script>
