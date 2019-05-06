<template>
  <div v-if="sequences.length > 0" id="trackList">
    <h3>Tracks</h3>
    <form>
      <input type="text" v-model="newSequenceField">
      <button
        @click="addSequenceField"
        :disabled="!newSequenceField.length > 0"
      >Add field to all sequences</button>
    </form>
    <button @click="hideAllSequences">Show / Hide all sequences</button>
    <ul class="list-group">
      <DataListItem
        v-for="sequence in sortedSequences"
        :key="sequence.id"
        :sequence="sequence"
        :sequenceFields="sequenceFields"
        :current="current"
        v-on:deleteSequence="sequences.splice(sequences.findIndex(x => x == sequence), 1)"
      />
    </ul>
  </div>
</template>

<script>
import DataListItem from "./DataListItem.vue";

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
    },
    sequenceFields: {
      type: Array,
      required: true
    }
  },
  data() {
    return {
      newSequenceField: ""
    };
  },
  computed: {
    sortedSequences() {
      var sorted = this.sequences.slice();
      sorted.reverse();
      return sorted;
    }
  },
  methods: {
    addSequenceField(e) {
      e.preventDefault();
      const isNewField = !this.sequenceFields.includes(this.newSequenceField);
      if (isNewField) {
        this.sequenceFields.push(this.newSequenceField);
      }
      this.newSequenceField = "";
    },
    hideSequence(sequence) {
      sequence.hidden = !sequence.hidden;
    },
    hideAllSequences() {
      if (
        this.sequences.every(sequence => {
          return sequence.hidden;
        })
      ) {
        this.sequences.forEach(sequence => {
          sequence.hidden = false;
        });
      } else {
        this.sequences.forEach(sequence => {
          sequence.hidden = true;
        });
      }
    }
  }
};
</script>

<style>
#trackList {
  width: 960px;
}
</style>