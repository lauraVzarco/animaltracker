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
      <MetaDataViewer
        v-if="images.length > 0"
        :metadata="images[current.frame]  || defaultMetadata"
      />
    </div>
    <div class="container">
      <DataList :sequences="sequences" :current="current"/>
    </div>
  </div>
</template>

<script>
import DataList from "./components/DataList.vue";
import Interface from "./components/Interface.vue";
import Progress from "./components/Progress.vue";
import MetaDataViewer from "./components/MetaDataViewer.vue";
import EXIF from "exif-js";

export default {
  components: {
    DataList,
    Interface,
    Progress,
    MetaDataViewer
  },
  data() {
    const data = this;
    return {
      images: [],
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
          "sequence_annotation",
          "number_of_points",
          ...this.getAllSelectedMetadata()
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
              this.sequences[i].name,
              points.length,
              ...this.getAllSelectedMetadata().map(field => {
                const image = this.images[points[j].frame];
                return image.exifdata[field] || image.userData[field];
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
      const images = this.images;
      images.splice(0, images.length);

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
          app.images.push.apply(app.images, image_elements);
          console.log("img el", image_elements);
          app.progress.loading = false;
        })
        .catch(function(image_elements) {
          console.log("Error loading images");
        });
    },
    getAllSelectedMetadata() {
      return [
        ...new Set(
          this.images.reduce((acc, { selectedUserData, selectedExifData }) => {
            return acc.concat(selectedUserData, selectedExifData);
          }, [])
        )
      ];
    }
  }
};
</script>

<style>
#app {
  height: 100vh;
}
#options {
  display: inline-block;
}
#markerSize {
  width: 150px;
}

.interfaceContainer {
  display: flex;
}
</style>
