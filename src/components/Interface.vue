<template>
 <div id="interface">
  <svg id="main" width="720" height="405" overflow="hidden">
   <g>
    <image
     v-if="images.length > 0"
     :href="background.src"
     :width="background.width"
     :height="background.height"
    ></image>
    <Sequence
     v-for="sequence in sequences"
     :key="sequence.id"
     :sequence="sequence"
     :transform="transform"
     :mode="mode"
     :frame="current.frame"
     :options="options"
    ></Sequence>
   </g>
  </svg>
​
  <div id="controls" class="container">
   <div class="btn-toolbar" role="toolbar" aria-label="Toolbar with button groups">
    <div class="btn-group" role="group">
     <button class="btn btn-secondary" @click="current.frame--">Previous</button>
     <button class="btn btn-secondary">{{ current.frame + 1 }}</button>
     <button class="btn btn-secondary" @click="current.frame++">Next</button>
    </div>
    <button class="btn" @click="resetView">Reset view</button>
    <button class="btn" @click="switchMode">{{ mode == 'add' ? 'Add mode' : 'Edit mode' }}</button>
    <button class="btn" @click="newSequence">Add sequence</button>
    <button class="btn" @click="deletePoint" :disabled="current.sequence < 0">Delete point</button>
    <button class="btn" @click="undoDeletePoint">Undo Delete</button>
   </div>
  </div>
 </div>
</template>
​
<script>
import Sequence from "./Sequence.vue";
export default {
 components: {
  Sequence
 },
 props: {
  sequences: {
   type: Array,
   required: true
  },
  deletedSequences: {
   type: Array,
   required: true
  },
  images: {
   type: Array,
   required: true
  },
  current: {
   type: Object,
   required: true
  },
  options: {
   type: Object,
   required: true
  }
 },
 computed: {
  background() {
   return this.images[this.current.frame];
  }
 },
 data() {
  return {
   mode: "edit",
   transform: {
    x: 0,
    y: 0,
    scale: 1
   },
   zoom: []
  };
 },
 methods: {
  addPoint(event) {
   var [mouse_x, mouse_y] = event;
   var x = (mouse_x - this.transform.x) / this.transform.scale;
   var y = (mouse_y - this.transform.y) / this.transform.scale;
   if (this.current.sequence >= 0) {
    if (!this.sequences.some(seq => seq.id == this.current.sequence)) {
     this.sequences.push({
      points: [],
      deletedPoints: [],
      id: this.current.sequence,
      fields: {},
      hidden: false
     });
    } else if (
     this.sequences
      .find(seq => seq.id == this.current.sequence)
      .points.some(p => p.frame == this.current.frame)
    ) {
     return;
    }
    this.sequences
     .find(seq => seq.id == this.current.sequence)
     .points.push({ x: x, y: y, frame: this.current.frame });
   }
  },
  deletePoint() {
   const sequenceIndex = this.sequences.findIndex(
    seq => seq.id == this.current.sequence
   );
   const sequence = this.sequences[sequenceIndex];
   const points = sequence.points;
   const currentPointIndex = points.findIndex(
    p => p.frame == this.current.frame
   );
   if (currentPointIndex >= 0) {
    const [deletedPoint] = points.splice(currentPointIndex, 1);
    sequence.deletedPoints.push(deletedPoint);
    if (points.length == 0) {
     const [deletedSequence] = this.sequences.splice(sequenceIndex, 1);
     this.deletedSequences.push(deletedSequence);
     this.current.sequence = this.sequences.length - 1;
    }
   }
   console.log("delete", this);
  },
  undoDeletePoint() {
   const sequence = this.sequences.find(
    seq => seq.id == this.current.sequence
   );
   if (sequence && sequence.deletedPoints.length > 0) {
    const point = sequence.deletedPoints.pop();
    sequence.points.push(point);
   } else if (this.deletedSequences.length > 0) {
    const sequence = this.deletedSequences.pop();
    this.sequences.push(sequence);
   }
   console.log("chocu", this);
  },
  newSequence() {
   this.sequences.forEach(sequence => {
    sequence.hidden = true;
   });
   const ids = this.sequences.map(sequence => sequence.id).sort();
   let new_id = 0;
   while (ids.includes(new_id)) {
    new_id++;
   }
   this.current.sequence = new_id;
   this.mode = "add";
  },
  resetView() {
   if (this.images.length > this.current.frame) {
    var image = this.images[this.current.frame];
    if (image.width / image.height > 720 / 405) {
     this.transform.scale = 720 / image.width;
     this.transform.x = 0;
     this.transform.y =
      (405 - image.height * this.transform.scale) /
      2 /
      this.transform.scale;
    } else {
     this.transform.scale = 405 / image.height;
     this.transform.x =
      (720 - image.width * this.transform.scale) /
      2 /
      this.transform.scale;
     this.transform.y = 0;
    }
    this.zoom.transform(
     d3.select("#main"),
     d3.zoomIdentity
      .scale(this.transform.scale)
      .translate(this.transform.x, this.transform.y)
    );
   }
  },
  switchMode() {
   if (this.mode == "add") {
    this.mode = "edit";
   } else {
    this.mode = "add";
   }
  }
 },
 created: function() {
  var vm = this;
  window.addEventListener("keyup", function(event) {
   if (event.keyCode == 39) {
    vm.current.frame++;
   } else if (event.keyCode == 37) {
    vm.current.frame--;
   }
  });
 },
 mounted: function() {
  var ui = this;
  this.zoom = d3.zoom().on("zoom", function() {
   ui.transform.x = d3.event.transform.x;
   ui.transform.y = d3.event.transform.y;
   ui.transform.scale = d3.event.transform.k;
   d3.select("g").attr(
    "transform",
    `translate(${ui.transform.x}, ${ui.transform.y}) scale(${
     ui.transform.scale
    })`
   );
  });
  d3.select("#main")
   .on("click", function() {
    if (ui.mode == "add") {
     ui.addPoint(d3.mouse(this, d3.event));
    }
   })
   .call(this.zoom);
 },
 watch: {
  images: function() {
   this.resetView();
  }
 }
};
</script>
​
<style>
#main {
 background-color: #aaa;
 margin-left: 6%;
}
/* #interface {
 width: 75%;
} */
</style>