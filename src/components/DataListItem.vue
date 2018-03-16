<template>
    <li :class="['list-group-item', current.sequence == sequence.id ? 'selected' : '']" @click="selectSequence">
        <span v-for="field in fields" key="field.id">
            <span>{{field.charAt(0).toUpperCase() + field.slice(1)}}</span>
            <input type="text">
        </span>
        <button @click="fields.push('foo')" class="glyphicon glyphicon-plus"></button>
        <span style="float: right">
            <span style="float: left; margin-right: 10px;">
                {{sequence.points.length}} point{{sequence.points.length > 1 ? 's' : ''}} [Image{{lastFrame > firstFrame ? 's' : ''}} <button title="jump to first image" style="padding: 0px" @click="current.frame = firstFrame">{{ firstFrame + 1 }}</button><span v-if="lastFrame > firstFrame"> to <button style="padding: 0px" title="jump to last image" @click="current.frame = lastFrame">{{ lastFrame + 1 }}</button></span>]
            </span>
        <button :class="['glyphicon', sequence.hidden ? 'glyphicon-eye-close' : 'glyphicon-eye-open']" @click="hideSequence" :title="action"></button>
        <button class="glyphicon glyphicon-remove" @click="deleteSequence" title="delete entire track"></button>
        </span>
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
        },
        fields: {
            type: Array,
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
        },
        frameNumbers() {
            return this.sequence.points.map(point => point.frame)
        },
        firstFrame() {
            return Math.min(...this.frameNumbers)
        },
        lastFrame() {
            return Math.max(...this.frameNumbers)
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
.list-group-item {
    font-weight: bold;
}
.selected {
    background-color: #aaa;
    color: white;
}
input {
    max-width: 15%;
    font-weight: normal
}
.selected button,input{
    color: black;
}
</style>
