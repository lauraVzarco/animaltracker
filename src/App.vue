<template>
	<div id="app">
        <div id="header" class="container">
            <div class="btn-toolbar" role="toolbar" aria-label="Toolbar with button groups">
                <input type="file" class="btn" @change="handleFiles" accept="image/*" multiple>
                <button type="button" class="btn" @click="exportCSV">Export CSV</button>
            </div>
        </div>

        <Interface :sequences="sequences" :images="images" class="container" :current="current"/>

        <div class="container">
            <DataList :sequences="sequences" :current="current"/>
        </div>
	</div>
</template>

<script>
import DataList from './components/DataList.vue'
import Interface from './components/Interface.vue'

export default {
	components: {
        DataList,
		Interface
	},
    data () {
        return {
            images: [],
            sequences: [],
            current: {frame: 0, sequence: -1}
        }
    },
    methods: {
        exportCSV () {
            var output = [["x", "y", "frame_number", "sequence_id", "sequence_annotation"]];
            var sep = ',';
            for (var i = 0; i < this.sequences.length; i++) {
                var points = this.sequences[i].points.concat();
                points.sort(function(a, b) {
                    return a.frame - b.frame;
                });
                for (var j = 0; j < points.length; j++) {
                    output.push([points[j].x, points[j].y, points[j].frame, i, this.sequences[i].name].join(','));
                }
            }
            output = output.join('\n');
            var exportLink = document.createElement('a');
            exportLink.setAttribute('href', 'data:text/csv;base64,' + window.btoa(output));
            exportLink.setAttribute('download', 'coordinates.csv');
            document.body.appendChild(exportLink);
            exportLink.click();
            document.body.removeChild(exportLink);
        },
        handleFiles (event) {
            var images = this.images;
            images.splice(0, images.length);

            var files = [].slice.call(event.target.files);
            files.sort(function(a, b) {
                return a.name.localeCompare(b.name);
            });

            this.loadImages(files);
        },
        getImage (file) {
            return new Promise(function (resolve, reject) {
                var reader = new FileReader();
                reader.onload = () => resolve(reader.result);
                reader.readAsDataURL(file);
            })
            .then(function (result) {
                return new Promise(function (resolve, reject) {
                     var image_element = new Image;
                     image_element.onload = () => resolve(image_element);
                     image_element.src = result;
                })
            })
        },
        loadImages (files) {
            var promises = files.map(this.getImage);
            var images = this.images;

            Promise.all(promises).then(function(image_elements) {
                images.push.apply(images, image_elements);
            }).catch(function(image_elements) {
                console.log('Error loading images')
            })
        }
    }
}
</script>
