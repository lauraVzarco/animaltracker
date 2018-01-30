<template>
    <div id="interface">
    	<svg id="main" width="960" height="540" overflow="hidden">
            <g>
                <image v-if="images.length > 0" :href="background.src" :width="background.width" :height="background.height"/>
                <Sequence v-for="sequence in sequences" :key="sequence.id" :sequence="sequence" :transform="transform" :mode="mode" :frame="current.frame"/>
            </g>
        </svg>

        <div id="controls" class="container">
            <div class="btn-toolbar" role="toolbar" aria-label="Toolbar with button groups">
                <div class="btn-group" role="group">
                    <button class="btn btn-secondary" @click="prevFrame">Previous</button>
                    <button class="btn btn-secondary">{{ current.frame + 1 }}</button>
                    <button class="btn btn-secondary" @click="nextFrame">Next</button>
                </div>
                <button class="btn" @click="resetView">Reset view</button>
                <button class="btn" @click="switchMode">{{ mode == 'add' ? 'Add mode' : 'Edit mode' }}</button>
                <button class="btn" @click="newSequence">Add sequence</button>
                <button class="btn" @click="deletePoint" :disabled="current.sequence < 0">Delete point</button>
            </div>
        </div>
    </div>
</template>

<script>
import Sequence from './Sequence.vue'

export default {
    components: {
        Sequence
    },
    props: {
      sequences: {
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
      }
    },
    computed: {
        background: function () {
            return this.images[this.current.frame];
        }
    },
    data () {
        return {
            mode: 'edit',
            transform: {
                x: 0,
                y: 0,
                scale: 1
            },
            zoom: []
        }
    },
    methods: {
        addPoint (event) {
            var [mouse_x, mouse_y] = event;
            var x = (mouse_x - this.transform.x) / this.transform.scale;
            var y = (mouse_y - this.transform.y) / this.transform.scale;

            if (this.current.sequence >= 0) {
                if (!this.sequences.some(seq => seq.id == this.current.sequence)) {
                    this.sequences.push({points: [], id: this.current.sequence, name: 'new', hidden: false})
                }
                else if (this.sequences.find(seq => seq.id == this.current.sequence).points.some(p => p.frame == this.current.frame)) {
                    return
                }

                this.sequences.find(seq => seq.id == this.current.sequence).points.push({x: x, y: y, frame: this.current.frame})
            }
        },
        deletePoint () {
            var sequence_id = this.sequences.findIndex(seq => seq.id == this.current.sequence);
            var points = this.sequences[sequence_id].points;
            var current_point = points.findIndex(p => p.frame == this.current.frame);
            if (current_point >= 0) {
                points.splice(current_point, 1);
                if (points.length == 0) {
                    this.sequences.splice(sequence_id, 1);
                }
            }
        },
        newSequence () {
            var ids = this.sequences.map(sequence => sequence.id).sort();
            var new_id = 0;
            while (ids.includes(new_id)) {
                new_id++;
            }
            this.current.sequence = new_id;
            this.mode = 'add';
        },
        nextFrame () {
            this.current.frame = Math.min(this.current.frame + 1, Math.max(this.images.length - 1, 0))
        },
        prevFrame () {
            this.current.frame = Math.max(this.current.frame - 1, 0)
        },
        resetView () {
            if (this.images.length > this.current.frame) {
                var image = this.images[this.current.frame];
                if ((image.width / image.height) > (960 / 540)) {
                    this.transform.scale = 960 / image.width;
                    this.transform.x = 0;
                    this.transform.y = ((540 - image.height * this.transform.scale) / 2) / this.transform.scale;
                }
                else {
                    this.transform.scale = 540 / image.height;
                    this.transform.x = ((960 - image.width * this.transform.scale) / 2) / this.transform.scale;
                    this.transform.y = 0;
                }
                this.zoom.transform(d3.select("#main"), d3.zoomIdentity.scale(this.transform.scale).translate(this.transform.x, this.transform.y));
            }
        },
        switchMode () {
            if (this.mode == 'add') {
                this.mode = 'edit';
            }
            else {
                this.mode = 'add';
            }
        }
    },
    created: function () {
        var vm = this;
        window.addEventListener('keyup', function(event) {
            if (event.keyCode == 39) {
                vm.nextFrame()
            }
            else if (event.keyCode == 37) {
                vm.prevFrame()
            }
        })
    },
    mounted: function () {
        var ui = this;

        this.zoom = d3.zoom().on("zoom", function () {
                ui.transform.x = d3.event.transform.x;
                ui.transform.y = d3.event.transform.y;
                ui.transform.scale = d3.event.transform.k;
                d3.select("g").attr("transform", `translate(${ui.transform.x}, ${ui.transform.y}) scale(${ui.transform.scale})`);
        });

        d3.select("#main")
            .on("click", function () {
                if (ui.mode == 'add') {
                    ui.addPoint(d3.mouse(this, d3.event))
                }
            })
            .call(this.zoom);
    },
    watch: {
        images: function () {
            this.resetView()
        }
    }
}
</script>

<style>
#main {
    background-color: #aaa;
}
</style>
