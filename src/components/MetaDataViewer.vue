<template>
	<div>
		<div>Image Metadata</div>
		<div>{{formattedMetadata}}</div>
	</div>
</template>
<script>
export default {
	props: {
		metadata: {
			type: Object,
			required: true
		}
	},
	computed: {
		formattedMetadata() {
			return JSON.stringify(this.metadata, null, 2);
		}
	}
};
</script>


