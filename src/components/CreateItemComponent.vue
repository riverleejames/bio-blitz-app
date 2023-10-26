<template>
    <v-card>
        <v-card-title>Total Count Per Species</v-card-title>
        <v-card-text>
            <v-text-field v-model="newItem.species" label="Species"></v-text-field>
            <v-text-field v-model="newItem.count" label="Count" type="number"></v-text-field>
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
                species: "",
                count: 0,
            },
        };
    },
    methods: {
        createItem() {
            this.$emit("create-item", this.newItem);
            this.newItem = { species: "", count: 0 };
        },
    },
};
</script>
