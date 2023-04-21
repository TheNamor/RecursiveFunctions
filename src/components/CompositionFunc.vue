<template>
    <div class="composition-func" ref="wrapper" :style="'height: ' + boxHeight + 'px; width: ' + boxWidth + 'px;'">
        <div style="padding-top: 10px;"></div>
        <div v-for="i in n" :key="'J'+i" class="input-label">
            x<sub>{{ i }}</sub>
            <div style="float: right; padding-right: 12px;">x<sub>{{ i }}</sub></div>
            <br>
        </div>
        <div class="center" :style="'top: ' + (boxHeight/2-27) + 'px; left: ' + (boxWidth/2-62) + 'px;'">
            Cn(x<sub>1</sub>{{ n>1 ? ',...,x' : ''}}<sub v-if="n>1">{{ n }}</sub>)
            <div style="padding-left: 15px">
                {{ n }} inputs
                <v-btn x-small icon @click="addVal()"><v-icon>mdi-plus</v-icon></v-btn>
                <v-btn x-small icon @click="removeVal()" :disabled="n == 1"><v-icon>mdi-minus</v-icon></v-btn>
            </div>
        </div>
    </div>
</template>

<script>

export default {

    props: {
        jsplumb: {
            required: true,
        },
        id: {
            required: true,
        }
    },

    mounted() {
        this.$nextTick(() => {
            // x1 input
            this.jsplumb.addEndpoint(this.drag, {
                isTarget: true,
                isSource: false,
                anchor: [0, 0, -1, 0, 0, 27],
                endpoint: ["Rectangle", {height: 25, width: 15}],
                maxConnections: 1,
                parameters: {
                    sortBy: 0,
                }
            })
            // x1 output
            this.jsplumb.addEndpoint(this.drag, {
                isSource: true,
                isTarget: false,
                anchor: [1, 0, 1, 0, 0, 27],
                endpoint: ["Rectangle", {height: 15, width: 25}],
                connectorOverlays:[ 
                    [ "Arrow", { location: 1, id:"arrow", foldback: 1 } ],
                ],
                maxConnections: -1,
                connectorStyle : { stroke: "black" },
                parameters: {
                    oneOut: false,
                    outputInd: 0,
                }
            })
        })
    },

    data: () => ({
        n: 1,
    }),

    methods: {
        addVal() {
            this.n++
            // Add x input
            this.jsplumb.addEndpoint(this.drag, {
                isTarget: true,
                isSource: false,
                anchor: [0, 0, -1, 0, 0, 27*this.n],
                endpoint: ["Rectangle", {height: 25, width: 15}],
                maxConnections: 1,
                parameters: {
                    sortBy: this.n-1
                }
            })
            // Add x output
            this.jsplumb.addEndpoint(this.drag, {
                isSource: true,
                isTarget: false,
                anchor: [1, 0, 1, 0, 0, 27*this.n],
                endpoint: ["Rectangle", {height: 15, width: 25}],
                connectorOverlays:[ 
                    [ "Arrow", { location: 1, id:"arrow", foldback: 1 } ],
                ],
                maxConnections: -1,
                connectorStyle : { stroke: "black" },
                parameters: {
                    oneOut: false,
                    outputInd: this.n-1,
                }
            })
            this.$nextTick(() => {
                this.jsplumb.repaintEverything()
            })
        },
        removeVal() {
            this.n--
            const endpoints = this.jsplumb.getEndpoints(this.drag)
            for (let i=0; i<2; i++) {
                this.jsplumb.deleteEndpoint(endpoints[endpoints.length-1-i])
            }
            this.$nextTick(() => {
                this.jsplumb.repaintEverything()
            })
        },
        setData(vars) {
            while(this.n < vars.n) {
                this.addVal()
            }
        },
        checkInputs() {
            const endpoints = this.jsplumb.getEndpoints(this.drag)
            const sources = endpoints.filter(endpoint => endpoint.isSource).sort((a,b) => { return a.getParameter("outputInd") - b.getParameter("outputInd") })
            const targets = endpoints.filter(endpoint => endpoint.isTarget).sort((a,b) => { return a.getParameter("sortBy") - b.getParameter("sortBy") })
            let out = false
            if (sources.every(source => source.connections.length == 0)) {
                return false
            }
            for (let i=0; i<this.n; i++) {
                if (targets[i].connections.length == 0) {
                    targets[i].setStyle({fill: "#F44336"})
                    out = true
                }
            }
            return out
        }
    },

    computed: {
        drag() {
            return this.$parent.$parent.$parent.$parent.$refs["func"+this.id]
        },
        boxHeight() {
            return this.n == 1 ? 36 + this.n*27 : 27 + this.n*27
        },
        boxWidth() {
            return 135 + this.n*20
        },
    }
}

</script>

<style scoped>

.composition-func {
    line-height: 27px;
    background-color: grey;
}

.center {
    text-align: center;
    position: absolute;
}

.input-label {
    padding-left: 8px;
}

.y-0-wrapper {
    border: solid black;
    border-width: 0px 0px 1px 1px;
    margin-left: 25px;
    padding-left: 5px;
}

.y-0 {
    padding-right: 5px;
    width: 20px;
    display: inline-block;
}

.y-wrapper {
    position: absolute;
    bottom: 0px;
    border: solid black;
    border-width: 1px 0px 0px 1px;
    margin-left: 25px;
    padding-left: 5px;
}

.y {
    padding-right: 5px;
    width: 20px;
    display: inline-block;
}

</style>