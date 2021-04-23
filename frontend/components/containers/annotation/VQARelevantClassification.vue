<template>
    <v-card
            v-if="isReady"
            v-shortkey="multiKeys"
            :loading="loading"
            class="mx-auto my-12"
            max-width="550"
            @shortkey="addOrRemoveLabel"
    >
        <template slot="progress">
            <v-progress-linear
                    color="deep-purple"
                    height="10"
                    indeterminate
            ></v-progress-linear>
        </template>

        <v-img
                contain
                position="center"
                max-width="500"
                max-height="400"
                :src="getCurrentImageLink()"
        ></v-img>


        <v-divider class="mx-4"></v-divider>

        <v-card-title> {{currentDoc.text}}</v-card-title>

        <v-divider class="mx-4"></v-divider>


        <v-textarea
                outlined
                clearable
                clear-icon="mdi-close-circle"
                name="input-7-4"
                label="Answer"
                :value="metaDataRetriever('answer')"
                @change="omitAns({ id: currentDoc.id, answer: $event }, 'answer')"
        ></v-textarea>

        <v-card-title>

            <multi-class-classification
                    :labels="items"
                    :annotations="currentDoc.annotations"
                    :add-label="addLabel"
                    :delete-label="removeLabel"
            />
        </v-card-title>

        <v-textarea
                outlined
                clearable
                clear-icon="mdi-close-circle"
                name="input-7-4"
                label="If yes, please explain how you reached your answer:"
                :value="metaDataRetriever('explain')"
                @change="omitAns({ id: currentDoc.id, explain: $event },'explain')"
        ></v-textarea>


        <v-card-text>
            <v-card-title class="font-weight-bold justify-center">Technical Annotation Guidelines</v-card-title>
            <v-divider class="mx-4"></v-divider>
            <v-timeline
            align-top
            dense
            >
                <v-timeline-item small>Use Filter -> Undone to annotate only unprocessed data points</v-timeline-item>
                <v-timeline-item small>Click <strong>LEFT/RIGHT</strong> arrows to browse between data points</v-timeline-item>
                <v-timeline-item
                    v-for="item in items"
                    :key="item"
                    small
                >
                    Click <strong>{{item.suffix_key}}</strong> to annotate with label <strong>{{item.text}}</strong>
                </v-timeline-item>
                <v-timeline-item small>Upon click annotations will be automatically saved into the DB</v-timeline-item>
            </v-timeline>
        </v-card-text>
    </v-card>
</template>

<script>

import Vue from 'vue';
import { mapActions, mapGetters, mapState } from 'vuex';
import MultiClassClassification from '@/components/organisms/annotation/MultiClassClassification'
Vue.use(require('vue-shortkey'))
export default {

    components: {
        MultiClassClassification
    },

    data: () => ({
        loading: false,
        selection: 1,
    }),
    computed: {
        ...mapState('labels', ['items']),
        ...mapState('documents', ['loading']),
        ...mapGetters('documents', ['currentDoc']),
        multiKeys() {
            const multiKeys = {}
            for (const item of this.items) {
                multiKeys[item.id] = [item.suffix_key]
            }
            return multiKeys
        },
        isReady() {
            return this.currentDoc && this.items && !this.loading
        }
    },

    created(){
        this.getLabelList({
            projectId: this.$route.params.id
        })
    },
    methods: {
        ...mapActions('labels', ['getLabelList']),
        ...mapActions('documents', ['getDocumentList', 'deleteAnnotation', 'updateAnnotation', 'addAnnotation', 'updateDocument']),

        metaDataRetriever(key){
            const meta = JSON.parse(this.currentDoc.meta);
            return meta[key] || '';
        },
        omitAns(payload, key) {
            console.log(payload);
            const meta = JSON.parse(this.currentDoc.meta);
            meta[key] = payload[key];
            console.log(meta);
            const data = {
                projectId: this.$route.params.id,
                id: payload.id,
                meta: JSON.stringify(meta)
            };
            this.updateDocument(data)
        },
        getCurrentImageLink() {
            const meta = JSON.parse(this.currentDoc.meta);
            const prefix = process.env.S3_BUCKET_PREFIX || `https://gqa-subset-images.s3.amazonaws.com/images-none-relevant-subset/`;
            return `${prefix}${meta.img_id}.jpg`;
        },
        removeLabel(annotationId) {
            const payload = {
                annotationId,
                projectId: this.$route.params.id
            }
            this.deleteAnnotation(payload)
        },
        updateLabel(labelId, annotationId) {
            const payload = {
                annotationId,
                label: labelId,
                projectId: this.$route.params.id
            }
            this.updateAnnotation(payload)
        },
        addLabel(labelId) {
            const payload = {
                label: labelId,
                projectId: this.$route.params.id
            }
            this.addAnnotation(payload)
        },
        addOrRemoveLabel(event) {
            const label = this.items.find(item => item.id === parseInt(event.srcKey, 10))
            const annotation = this.currentDoc.annotations.find(item => item.label === label.id)
            if (annotation) {
                this.removeLabel(annotation.id)
            } else {
                this.addLabel(label.id)
            }
        },
        async loadData(){
            const image = Promise.resolve(); // mock
            const question = Promise.resolve(); // mock
            return await Promise.all([image,question]);
        },

        annotate() {
            this.loading = true;
            console.log('Annotated');
            setTimeout(() => (this.loading = false), 2000)
        },
    },
}
</script>


<style scoped>

</style>