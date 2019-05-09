<template>
  <tr
    :class="['list-group-item', current.sequence == sequence.id ? 'selected' : '']"
    @click="selectSequence"
  >
    <td class="field-data" v-for="field in sequenceFields" :key="field">
      <input type="text" v-model="sequence.fields[field]">
    </td>
    <td class="field-data">
      {{sequence.points.length}} point{{sequence.points.length > 1 ? 's' : ''}} [Image{{lastFrame > firstFrame ? 's' : ''}}
      <button
        title="jump to first image"
        style="padding: 0px"
        @click="current.frame = firstFrame"
      >{{ firstFrame + 1 }}</button>
      <span v-if="lastFrame > firstFrame">
        to
        <button
          style="padding: 0px"
          title="jump to last image"
          @click="current.frame = lastFrame"
        >{{ lastFrame + 1 }}</button>
      </span>]
    </td>
    <td class="field-data">
      <button
        style="float:right"
        class="glyphicon glyphicon-remove"
        @click="deleteSequence"
        title="delete entire track"
      ></button>
      <button
        style="float:right"
        :class="['glyphicon', sequence.hidden ? 'glyphicon-eye-close' : 'glyphicon-eye-open']"
        @click="hideSequence"
        :title="action"
      ></button>
    </td>
  </tr>
</template>

<script>
export default {
  props: {
    sequence: {
      type: Object,
      required: true
    },
    sequenceFields: {
      type: Array,
      required: true
    },
    current: {
      type: Object,
      required: true
    }
  },
  data() {
    return {
      visibility: "visible",
      newSequenceField: ""
    };
  },
  computed: {
    action: function() {
      return this.sequence.hidden ? "show" : "hide";
    },
    frameNumbers() {
      return this.sequence.points.map(point => point.frame);
    },
    firstFrame() {
      return Math.min(...this.frameNumbers);
    },
    lastFrame() {
      return Math.max(...this.frameNumbers);
    }
  },
  methods: {
    deleteSequence() {
      this.$emit("deleteSequence");
    },
    hideSequence() {
      this.sequence.hidden = !this.sequence.hidden;
    },
    selectSequence() {
      this.current.sequence = this.sequence.id;
    }
  }
};
</script>

<style>
.list-group-item {
  font-weight: bold;
  display: flex;
  flex-direction: row;
  padding: 0;
}
.selected {
  background-color: #aaa;
  color: white;
}
input {
  font-weight: normal;
}
.selected button,
input {
  color: black;
}
.field-item {
  display: flex;
  flex-direction: row;
  list-style: none;
  text-align: left;
  max-width: 100%;
}
.field-data {
  width: 178px;
  border: 1px solid black;
  border-top: none;
}
</style>