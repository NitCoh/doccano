<template>
    <v-q-a-relevant-classification />
</template>

<script>
import  VQARelevantClassification  from "~/components/containers/annotation/VQARelevantClassification";

    export default {
        layout: 'annotation',

        components: {
            VQARelevantClassification
        },

        // validate({params, query}){
        //     return true; // TODO: Complete.
        // }
    }
</script>

<style scoped>

</style>