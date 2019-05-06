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
     :imagesNumber="newImages.length"
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
    :deletedSequences="deletedSequences"
    :images="images"
    class="container"
    :current="current"
    :options="options"
   />
   <Progress v-if="progress.loading" :progress="progress"></Progress>
   <div v-if="images.length > 0">
    <button
     type="button"
     class="btn"
     @click="showMetadataViewer = !showMetadataViewer"
    >{{showMetadataViewer? "Hide": "Show"}} image metadata</button>
    <MetaDataViewer
     v-if="showMetadataViewer"
     :currentImage="images[current.frame] || defaultMetadata"
     :exifdata="exifdata"
     :userData="userData"
    />
   </div>
  </div>
  <div class="container">
   <DataList :sequences="sequences" :current="current" :sequenceFields="sequenceFields"/>
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
   deletedSequences: [],
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
    userData: {}
   },
   exifdata: [],
   userData: [],
   sequenceFields: [],
   showMetadataViewer: true
  };
 },
 computed: {
  allImagesMetadataKeys() {}
 },
 methods: {
  exportCSV() {
   const output = [
    [
     "image_name",
     "x",
     "y",
     "frame_number",
     "sequence_id",
     "number_of_points",
     ...this.getAllSelectedMetadata(),
     ...this.sequenceFields
    ]
   ];
   const sep = ",";
   for (let i = 0; i < this.sequences.length; i++) {
    const points = this.sequences[i].points.concat();
    points.sort(function(a, b) {
     return a.frame - b.frame;
    });
    for (let j = 0; j < points.length; j++) {
     const image = this.images[points[j].frame];
     output.push(
      [
       image.name,
       points[j].x,
       points[j].y,
       points[j].frame,
       i,
       points.length,
       ...this.getAllSelectedMetadata().map(field => {
        return image.exifdata[field] || image.userData[field];
       }),
       ...this.sequenceFields.map(field => {
        return this.sequences[i].fields[field];
       })
      ].join(",")
     );
    }
   }
   const outputString = output.join("\n");
   var exportLink = document.createElement("a");
   exportLink.setAttribute(
    "href",
    "data:text/csv;base64," + window.btoa(outputString)
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
   this.loadImages(files);
  },
  getImage(file, progress) {
   const vm = this;
   return new Promise(function(resolve, reject) {
    const reader = new FileReader();
    reader.onload = () => {
     EXIF.getData(file, function() {
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
       userData: {}
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
     }
     app.exifdata = app.getExifData();
     app.progress.loading = false;
    })
    .catch(function(image_elements) {
     console.log("Error loading images");
    });
  },
  getExifData() {
   return getUniqueArray(
    [...this.newImages, ...this.images].reduce((acc, image) => {
     return acc.concat(Object.keys(image.exifdata));
    }, [])
   ).map(key => ({ key, selected: false }));
  },
  getAllSelectedMetadata() {
   return [...this.exifdata, ...this.userData]
    .filter(el => el.selected)
    .map(({ key }) => key);
  },
  addToCurrentSet() {
   this.images.push.apply(this.images, this.newImages);
   this.newImages = [];
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