<template>
    <g v-show="!sequence.hidden">
    </g>
</template>

<script>
export default {
    props: {
      sequence: {
        type: Object,
        required: true
    },
      transform: {
          type: Object,
          required: true
      },
      mode: {
          type: String,
          required: true
      },
      frame: {
          type: Number,
          required: true
      },
      options: {
          type: Object,
          required: true
      }
    },
    data () {
        return {
            circles: [],
            lines: [],
            lineEnds: []
        }
    },
    mounted: function () {
        this.connectLineEnds();
        this.createLines();
        this.createPoints();
    },
    methods: {
        connectLineEnds () {
            var points = this.sequence.points.concat();
            this.lineEnds = [];

            points.sort(function(a, b) {
                return a.frame - b.frame;
            });

            for (var i = 0; i < points.length - 1; i++) {
                this.lineEnds.push({source: points[i], target: points[i + 1]});
            }
        },
        dragPoint (d) {
            if (this.mode == 'edit') {
                d.x = d3.event.x;
                d.y = d3.event.y;
                this.updateLines();
                this.updatePoints();
            }
        },
        createPoints () {
            var vm = this;
            this.circles = d3.select(this.$el)
                .selectAll("circle")
                .data(this.sequence.points)
                .enter().append("circle")
                .attr("cx", function(d) { return d.x; })
                .attr("cy", function(d) { return d.y; })
                .attr("r", function(d) { return vm.options.markerSize / vm.transform.scale })
                .attr("fill", function(d) { return d.frame == vm.frame ? "white" : "black" })
                .attr("stroke", function(d) { return d.frame == vm.frame ? "black" : "white" })
                .attr("stroke-width", function (d) { return 0.4 * vm.options.markerSize / vm.transform.scale })
                .call(d3.drag().on("drag", this.dragPoint));
        },
        updatePoints () {
            var vm = this;
            this.circles
                .attr("cx", function(d) { return d.x; })
                .attr("cy", function(d) { return d.y; })
                .attr("r", function(d) { return vm.options.markerSize / vm.transform.scale })
                .attr("fill", function(d) { return d.frame == vm.frame ? "white" : "black" })
                .attr("stroke", function(d) { return d.frame == vm.frame ? "black" : "white" })
                .attr("stroke-width", function (d) { return 0.4 * vm.options.markerSize / vm.transform.scale })
        },
        createLines () {
            var vm = this;
            this.lines = d3.select(this.$el)
                .selectAll("line")
                .data(this.lineEnds)
                .enter().append("line")
                .attr("x1", function(d) { return d.source.x; })
                .attr("y1", function(d) { return d.source.y; })
                .attr("x2", function(d) { return d.target.x; })
                .attr("y2", function(d) { return d.target.y; })
                .attr("stroke-width", vm.options.markerSize / 2)
                .attr("vector-effect", "non-scaling-stroke")
                .attr("stroke", function(d) { return d.target.frame > vm.frame ? "red" : "blue" });
        },
        updateLines () {
            var vm = this;
            this.lines
                .attr("x1", function(d) { return d.source.x; })
                .attr("y1", function(d) { return d.source.y; })
                .attr("x2", function(d) { return d.target.x; })
                .attr("y2", function(d) { return d.target.y; })
                .attr("stroke-width", vm.options.markerSize / 2)
                .attr("stroke", function(d) { return d.target.frame > vm.frame ? "red" : "blue"; })
        }
    },
    watch: {
        'sequence.points': function () {
            this.connectLineEnds();
            d3.select(this.$el).selectAll('line').remove();
            this.createLines();
            d3.select(this.$el).selectAll('circle').remove();
            this.createPoints();
        },
        'transform.scale': function () {
            this.updatePoints();
        },
        'options.markerSize': function () {
            this.updatePoints();
            this.updateLines();
        },
        frame: function () {
            this.updateLines();
            this.updatePoints();
        }
    }
}
</script>
