<script>
export default {
	props: {
		imagesNumber: {
			type: Number,
			required: true
		}
	},
	methods: {
		addToCurrentSet() {
			this.$emit("addToCurrentSet");
		},
		replaceSet() {
			this.$emit("replaceSet");
		}
	},
	render(h) {
		return (
			<div>
				<div>
					You are going to upload {this.imagesNumber} image/s. What do you want
					to do?
				</div>
				<button onClick={this.addToCurrentSet}> Add to current set </button>
				<button onClick={this.replaceSet}> Replace set</button>
			</div>
		);
	}
};
</script>
<style>
</style>

