<template>
    <li :class="['list-group-item', current.sequence == sequence.id ? 'selected' : '']" @click="selectSequence">
        <span>Name</span>
        <input type="text" v-model="sequence.name">
        <select>
            <option disabled value="">Species</option>
            <option>Species 1</option>
            <option>Species 2</option>
            <option>Species 3</option>
        </select>
        <button :class="['glyphicon', sequence.hidden ? 'glyphicon-eye-close' : 'glyphicon-eye-open']" @click="hideSequence" :title="action"></button>
        <button class="glyphicon glyphicon-remove" @click="deleteSequence" title="delete entire track"></button>
    </li>
</template>

<script>
export default {
    props: {
        sequence: {
            type: Object,
            required: true
        },
        current: {
            type: Object,
            required: true
        }
    },
    data () {
        return {
            visibility: 'visible'
        }
    },
    computed: {
        action: function () {
            return this.sequence.hidden ? 'show' : 'hide'
        }
    },
    methods: {
        deleteSequence () {
            this.$emit('deleteSequence')
        },
        hideSequence () {
            this.sequence.hidden = !this.sequence.hidden
        },
        selectSequence () {
            this.current.sequence = this.sequence.id
        }
    }
}
</script>

<style>
.selected {
    background-color: #bbb;
}
</style>
