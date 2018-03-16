<template>
    <div v-if="sequences.length > 0" id="trackList">
        <h3>Tracks</h3>
        <ul class="list-group">
            <DataListItem v-for="sequence in sortedSequences" :key="sequence.id" :sequence="sequence" :current="current" v-on:deleteSequence="sequences.splice(sequences.findIndex(x => x == sequence), 1)"/>
        </ul>
    </div>
</template>

<script>
import DataListItem from './DataListItem.vue'

export default {
	components: {
        DataListItem
    },
    props: {
        sequences: {
            type: Array,
            required: true
        },
        current: {
            type: Object,
            required: true
        }
    },
    computed: {
        sortedSequences() {
            var sorted = this.sequences.slice();
            sorted.reverse();
            return sorted
        }
    }
}
</script>

<style>
#trackList {
    width: 960px;
}
</style>
