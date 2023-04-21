<template>
    <div class="primitive-recursion" ref="wrapper" :style="'height: ' + boxHeight + 'px; width: ' + boxWidth + 'px;'">
        <div class="y-0-wrapper">
            <div v-for="i in n" :key="'F'+i" class="y-0">
                x<sub>{{ i }}</sub>
            </div>
            <div class="y-0" style="margin-left: 5px;">
                y=0
            </div>
        </div>
        <div v-for="i in n" :key="'E'+i" class="input-label">
            x<sub>{{ i }}</sub><br>
        </div>
        <div class="input-label" style="margin-top: 5px;">
            y
        </div>
        <div class="center" :style="'top: ' + (boxHeight/2-21) + 'px; left: ' + (boxWidth/2-62) + 'px;'">
            Pr[f<sub>{{ n }}</sub>,g<sub>{{ n+2 }}</sub>]
            <div style="padding-left: 15px">
                {{ n }} inputs
                <v-btn x-small icon @click="addVal()"><v-icon>mdi-plus</v-icon></v-btn>
                <v-btn x-small icon @click="removeVal()" :disabled="n == 1"><v-icon>mdi-minus</v-icon></v-btn>
            </div>
        </div>
        <div class="y-wrapper" :style="'width: ' + (boxWidth-25) + 'px;'">
            <div v-for="i in n" :key="'G'+i" class="y">
                x<sub>{{ i }}</sub>
            </div>
            <div class="y" style="margin-left: 5px;">
                y>0
            </div>
            <div class="y" style="margin-left: 10px;">
                Pr
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
                anchor: [0, 0, -1, 0, 0, 44],
                endpoint: ["Rectangle", {height: 25, width: 15}],
                maxConnections: 1,
                parameters: {
                    sortBy: 3,
                }
            })
            // y input
            this.jsplumb.addEndpoint(this.drag, {
                isTarget: true,
                isSource: false,
                anchor: [0, 1, -1, 0, 0, -44],
                endpoint: ["Rectangle", {height: 25, width: 15}],
                maxConnections: 1,
                parameters: {
                    sortBy: 2,
                }
            })
            // y=0 output
            this.jsplumb.addEndpoint(this.drag, {
                isSource: true,
                isTarget: false,
                anchor: [1, 0, 0, -1, -89, -3],
                endpoint: ["Rectangle", {height: 25, width: 15}],
                connectorOverlays:[ 
                    [ "Arrow", { location: 1, id:"arrow", foldback: 1 } ],
                ],
                maxConnections: 1,
                connectorStyle : { stroke: "black" },
                parameters: {
                    to: "y-0",
                    outputInd: 2,
                }
            })
            // x1 y-0 output
            this.jsplumb.addEndpoint(this.drag, {
                isSource: true,
                isTarget: false,
                anchor: [0, 0, 0, -1, 37, -3],
                endpoint: ["Rectangle", {height: 25, width: 15}],
                connectorOverlays:[ 
                    [ "Arrow", { location: 1, id:"arrow", foldback: 1 } ],
                ],
                maxConnections: 1,
                connectorStyle : { stroke: "black" },
                parameters: {
                    to: "y-0",
                    outputInd: 4,
                }
            })
            // y-0 input
            this.jsplumb.addEndpoint(this.drag, {
                isTarget: true,
                isSource: false,
                anchor: [1, 0, 0, -1, -31, 0],
                endpoint: ["Rectangle", {height: 15, width: 25}],
                maxConnections: 1,
                parameters: {
                    from: "y-0",
                    sortBy: 0,
                }
            })
            // y>0 output
            this.jsplumb.addEndpoint(this.drag, {
                isSource: true,
                isTarget: false,
                anchor: [1, 1, 0, 1, -89, 6],
                endpoint: ["Rectangle", {height: 25, width: 15}],
                connectorOverlays:[ 
                    [ "Arrow", { location: 1, id:"arrow", foldback: 1 } ],
                ],
                maxConnections: 1,
                connectorStyle : { stroke: "black" },
                parameters: {
                    to: "y>0",
                    outputInd: 1,
                }
            })
            // Pr[y-1] output
            this.jsplumb.addEndpoint(this.drag, {
                isSource: true,
                isTarget: false,
                anchor: [1, 1, 0, 1, -64, 6],
                endpoint: ["Rectangle", {height: 25, width: 15}],
                connectorOverlays:[ 
                    [ "Arrow", { location: 1, id:"arrow", foldback: 1 } ],
                ],
                maxConnections: 1,
                connectorStyle : { stroke: "black" },
                parameters: {
                    to: "y>0",
                    outputInd: 3,
                }
            })
            // x1 y>0 output
            this.jsplumb.addEndpoint(this.drag, {
                isSource: true,
                isTarget: false,
                anchor: [0, 1, 0, 1, 37, 6],
                endpoint: ["Rectangle", {height: 25, width: 15}],
                connectorOverlays:[ 
                    [ "Arrow", { location: 1, id:"arrow", foldback: 1 } ],
                ],
                maxConnections: 1,
                connectorStyle : { stroke: "black" },
                parameters: {
                    to: "y>0",
                    outputInd: 5,
                }
            })
            // y>0 input
            this.jsplumb.addEndpoint(this.drag, {
                isTarget: true,
                isSource: false,
                anchor: [1, 1, 0, 1, -31, 0],
                endpoint: ["Rectangle", {height: 15, width: 25}],
                maxConnections: 1,
                parameters: {
                    from: "y>0",
                    sortBy: 1,
                }
            })
            // true output
            this.jsplumb.addEndpoint(this.drag, {
                isSource: true,
                isTarget: false,
                anchor: "Right",
                endpoint: ["Rectangle", {height: 15, width: 25}],
                connectorOverlays:[ 
                    [ "Arrow", { location: 1, id:"arrow", foldback: 1 } ],
                ],
                maxConnections: -1,
                connectorStyle : { stroke: "black" },
                parameters: {
                    outputInd: 0,
                    oneOut: true,
                    trueOut: true,
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
                anchor: [0, 0, -1, 0, 0, 17+27*this.n],
                endpoint: ["Rectangle", {height: 25, width: 15}],
                maxConnections: 1,
                parameters: {
                    sortBy: 2+this.n
                }
            })
            // Add y-0 x output
            this.jsplumb.addEndpoint(this.drag, {
                isSource: true,
                isTarget: false,
                anchor: [0, 0, 0, -1, 17+20*this.n, -3],
                endpoint: ["Rectangle", {height: 25, width: 15}],
                connectorOverlays:[ 
                    [ "Arrow", { location: 1, id:"arrow", foldback: 1 } ],
                ],
                maxConnections: 1,
                connectorStyle : { stroke: "black" },
                parameters: {
                    to: "y-0",
                    outputInd: this.n*2+2,
                }
            })
            // Add y>0 x output
            this.jsplumb.addEndpoint(this.drag, {
                isSource: true,
                isTarget: false,
                anchor: [0, 1, 0, 1, 17+20*this.n, 6],
                endpoint: ["Rectangle", {height: 25, width: 15}],
                connectorOverlays:[ 
                    [ "Arrow", { location: 1, id:"arrow", foldback: 1 } ],
                ],
                maxConnections: 1,
                connectorStyle : { stroke: "black" },
                parameters: {
                    to: "y>0",
                    outputInd: this.n*2+3,
                }
            })
            this.$nextTick(() => {
                this.jsplumb.repaintEverything()
            })
        },
        removeVal() {
            this.n--
            const endpoints = this.jsplumb.getEndpoints(this.drag)
            for (let i=0; i<3; i++) {
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
            if (sources[0].connections.length == 0) {
                return false
            }
            if (targets[0].connections.length == 0) {
                targets[0].setStyle({fill: "#F44336"})
                out = true
            }
            if (targets[1].connections.length == 0) {
                targets[1].setStyle({fill: "#F44336"})
                out = true
            }
            if (targets[2].connections.length == 0) {
                targets[2].setStyle({fill: "#F44336"})
                out = true
            }
            for (let i=1; i<this.n; i++) {
                if (targets[i+3].connections.length == 0) {
                    targets[i+3].setStyle({fill: "#F44336"})
                    out = true
                }
            }
            if (targets[3].connections.length == 0) {
                if (sources[4].connections.length > 0) {
                    targets[3].setStyle({fill: "#F44336"})
                    out = true
                }
                if (sources[5].connections.length > 0) {
                    targets[3].setStyle({fill: "#F44336"})
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
            return 90 + this.n*27
        },
        boxWidth() {
            return 135 + this.n*20
        },
    }
}

</script>

<style scoped>

.primitive-recursion {
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