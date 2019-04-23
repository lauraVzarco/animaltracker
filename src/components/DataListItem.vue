<template>
	<li
		:class="['list-group-item', current.sequence == sequence.id ? 'selected' : '']"
		@click="selectSequence"
	>
		<ul>
			<li v-for="field in sequenceFields">
				<span>{{field}}</span>
				<input type="text" v-model="sequence.fields[field]">
			</li>
		</ul>
		<span>
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
		</span>
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
		},
		sequenceFields() {
			return Object.keys(this.sequence.fields);
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
</style>
