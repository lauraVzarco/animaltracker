<script>
export default {
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
				<div>You are going to upload X images. What do you want to do?</div>
				<button onClick={this.addToCurrentSet}> Add to the current set </button>
				<button onClick={this.replaceSet}> Replace set</button>
			</div>
		);
	}
};
</script>
<style>
</style>

