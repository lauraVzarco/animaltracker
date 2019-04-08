
<script>
import UserDefinedField from "./UserDefinedField";
export default {
	components: { UserDefinedField },
	props: {
		metadata: {
			type: Object,
			required: true
		}
	},
	methods: {
		toggleExifField(field) {
			if (this.metadata.selectedExifData.includes(field)) {
				const index = this.metadata.selectedExifData.indexOf(field);
				this.metadata.selectedExifData.splice(index, 1);
			} else {
				this.metadata.selectedExifData.push(field);
			}
			console.log(this.metadata.selectedExifData);
		},
		toggleUserDataField(field) {
			if (this.metadata.selectedUserData.includes(field)) {
				const index = this.metadata.selectedUserData.indexOf(field);
				this.metadata.selectedUserData.splice(index, 1);
			} else {
				this.metadata.selectedUserData.push(field);
			}
		},
		addField(key, value) {
			this.$set(this.metadata.userData, key, value);
			this.metadata.selectedUserData.push(key);
			console.log(key, value, this.metadata);
		}
	},
	render(h) {
		return (
			<div class="metadataContainer">
				<div>Image Metadata for {this.metadata.name}</div>
				<div>
					{Object.entries(this.metadata.exifdata).map(([key, value]) => {
						return (
							<div>
								<input
									type="checkbox"
									checked={this.metadata.selectedExifData.includes(key)}
									onInput={() => this.toggleExifField(key)}
								/>
								{key}: {value}
							</div>
						);
					})}
					{Object.entries(this.metadata.userData).map(([key, value]) => {
						return (
							<div>
								<input
									type="checkbox"
									checked={this.metadata.selectedUserData.includes(key)}
									onInput={() => this.toggleUserDataField(key)}
								/>
								{key}: {value}
							</div>
						);
					})}
				</div>
				<UserDefinedField addField={this.addField} />
			</div>
		);
	}
};
</script>
<style>
.metadataContainer {
	width: 200px;
	height: 405px;
	overflow: scroll;
	background: #aaaa;
	padding: 10px;
	margin-left: 8%;
	align-content: center;
}
</style>



