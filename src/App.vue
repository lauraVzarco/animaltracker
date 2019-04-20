<template>
	<div id="app">
		<div id="header" class="container">
			<div
				class="btn-toolbar"
				style="width: 960px"
				role="toolbar"
				aria-label="Toolbar with button groups"
			>
				<input type="file" class="btn" @change="handleFiles" accept="image/*" multiple>
				<UploadImagesPrompt
					v-if="showPrompt"
					v-on:addToCurrentSet="addToCurrentSet"
					v-on:replaceSet="replaceSet"
				/>
				<button type="button" class="btn" @click="exportCSV">Export CSV</button>
				<div id="options">
					Marker size: {{ options.markerSize }}
					<input
						id="markerSize"
						v-model.number="options.markerSize"
						type="range"
						min="1"
						max="20"
						step="1"
					>
				</div>
			</div>
		</div>
		<div class="interfaceContainer">
			<Interface
				:sequences="sequences"
				:images="images"
				class="container"
				:current="current"
				:options="options"
			/>
			<Progress v-if="progress.loading" :progress="progress"></Progress>
			<MetaDataViewer v-if="images.length > 0" :metadata="images[current.frame]  || defaultMetadata"/>
		</div>
		<div class="container">
			<DataList :sequences="sequences" :current="current"/>
		</div>
	</div>
</template>

<script>
import DataList from "./components/DataList.vue";
import UploadImagesPrompt from "./components/UploadImagesPrompt";
import Interface from "./components/Interface.vue";
import Progress from "./components/Progress.vue";
import MetaDataViewer from "./components/MetaDataViewer.vue";
import EXIF from "exif-js";

const getUniqueArray = arr => {
	return [...new Set(arr)];
};

export default {
	components: {
		DataList,
		Interface,
		Progress,
		MetaDataViewer,
		UploadImagesPrompt
	},
	data() {
		const data = this;
		return {
			images: [],
			newImages: [],
			showPrompt: false,
			sequences: [],
			current: {
				_frame: 0,
				set frame(value) {
					this._frame = Math.max(
						Math.min(value, Math.max(data.images.length - 1, 0)),
						0
					);
				},
				get frame() {
					return this._frame;
				},
				sequence: -1
			},
			progress: { loading: false, loaded: 0, total: 0 },
			options: { markerSize: 5 },
			defaultMetadata: {
				exifdata: {},
				userData: {},
				selectedUserData: []
			}
		};
	},
	computed: {
		metadata() {
			return this.images[this.current.frame] || this.defaultMetadata;
		}
	},
	methods: {
		exportCSV() {
			var output = [
				[
					"x",
					"y",
					"frame_number",
					"sequence_id",
					"number_of_points",
					...this.getAllSelectedMetadata(),
					...this.getAllSequenceFields()
				]
			];
			var sep = ",";
			for (var i = 0; i < this.sequences.length; i++) {
				var points = this.sequences[i].points.concat();
				points.sort(function(a, b) {
					return a.frame - b.frame;
				});
				for (var j = 0; j < points.length; j++) {
					output.push(
						[
							points[j].x,
							points[j].y,
							points[j].frame,
							i,
							points.length,
							...this.getAllSelectedMetadata().map(field => {
								const image = this.images[points[j].frame];
								return image.exifdata[field] || image.userData[field];
							}),
							...this.getAllSequenceFields().map(field => {
								return this.sequences[i].fields[field];
							})
						].join(",")
					);
				}
			}
			output = output.join("\n");
			var exportLink = document.createElement("a");
			exportLink.setAttribute(
				"href",
				"data:text/csv;base64," + window.btoa(output)
			);
			exportLink.setAttribute("download", "coordinates.csv");
			document.body.appendChild(exportLink);
			exportLink.click();
			document.body.removeChild(exportLink);
		},
		handleFiles(event) {
			const files = [].slice.call(event.target.files);
			files.sort(function(a, b) {
				return a.name.localeCompare(b.name);
			});

			this.progress.total = files.length;
			this.progress.loaded = 0;
			this.progress.loading = true;

			console.log(files, "files");
			this.loadImages(files);
		},
		getImage(file, progress) {
			const vm = this;
			return new Promise(function(resolve, reject) {
				const reader = new FileReader();
				reader.onload = () => {
					EXIF.getData(file, function() {
						console.log("image info", this);
						console.log("exif data", this.exifdata);
						const { size, name, type, lastModified, exifdata } = this;
						resolve({
							src: reader.result,
							name,
							size,
							type,
							lastModified,
							exifdata
						});
					});
				};
				reader.readAsDataURL(file);
			}).then(function(result) {
				return new Promise(function(resolve, reject) {
					const image_element = new Image();
					image_element.onload = () => {
						progress.loaded++;
						const { width, height, src } = image_element;
						resolve({
							...result,
							width,
							height,
							userData: {},
							selectedUserData: [],
							selectedExifData: []
						});
					};
					image_element.src = result.src;
				});
			});
		},
		loadImages(files) {
			const promises = files.map(file => {
				return this.getImage(file, this.progress);
			});
			const app = this;
			Promise.all(promises)
				.then(function(image_elements) {
					if (app.images.length > 0) {
						app.newImages.push.apply(app.newImages, image_elements);
						app.showPrompt = true;
					} else {
						app.images.push.apply(app.images, image_elements);
						this.current.frame = 0;
					}
					app.progress.loading = false;
				})
				.catch(function(image_elements) {
					console.log("Error loading images");
				});
		},
		getAllSelectedMetadata() {
			return getUniqueArray(
				this.images.reduce((acc, { selectedUserData, selectedExifData }) => {
					return acc.concat(selectedUserData, selectedExifData);
				}, [])
			);
		},
		getAllSequenceFields() {
			return getUniqueArray(
				this.sequences.reduce((acc, sequence) => {
					return acc.concat(Object.keys(sequence.fields));
				}, [])
			);
		},
		addToCurrentSet() {
			this.images.push.apply(this.images, this.newImages);
			this.newImages = [];
			console.log(this);
			this.showPrompt = false;
			this.current.frame = 0;
		},
		replaceSet() {
			this.images = this.newImages;
			this.newImages = [];
			this.showPrompt = false;
			this.current.frame = 0;
		}
	}
};
</script>

<style>
#app {
	margin-top: 50px;
}
#options {
	display: inline-block;
}
#markerSize {
	width: 150px;
}
.container {
	width: 720px;
}
.interfaceContainer {
	display: flex;
}
</style>
