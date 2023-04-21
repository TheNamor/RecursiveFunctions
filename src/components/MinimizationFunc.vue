<template>
    <div class="primitive-recursion" ref="wrapper" :style="'height: ' + boxHeight + 'px; width: ' + boxWidth + 'px;'">
        <div class="y-0-wrapper">
            <div v-for="i in n" :key="'H'+i" class="y-0">
                x<sub>{{ i }}</sub>
            </div>
            <div class="y-0" style="margin-left: 13px;">
                y
            </div>
        </div>
        <div v-for="i in n" :key="'I'+i" class="input-label">
            x<sub>{{ i }}</sub><br>
        </div>
        <div class="center" :style="'top: ' + (boxHeight/2-13) + 'px; left: ' + (boxWidth/2-62) + 'px;'">
            Mn[f<sub>{{ n+1 }}</sub>]
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
                anchor: [0, 0, -1, 0, 0, 44],
                endpoint: ["Rectangle", {height: 25, width: 15}],
                maxConnections: 1,
                parameters: {
                    sortBy: 1,
                }
            })
            // y output
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
                    outputInd: 1,
                }
            })
            // x1 output
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
                    outputInd: 2,
                }
            })
            // f(x1,..,xn, y) input
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
                    oneOut: true,
                    trueOut: true,
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
                anchor: [0, 0, -1, 0, 0, 17+27*this.n],
                endpoint: ["Rectangle", {height: 25, width: 15}],
                maxConnections: 1,
                parameters: {
                    sortBy: this.n
                }
            })
            // Add x output
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
                    outputInd: this.n+1,
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
            if (sources[0].connections.length == 0) {
                return false
            }
            let out = false
            if (targets[0].connections.length == 0) {
                targets[0].setStyle({fill: "#F44336"})
                out = true
            }
            for (let i=0; i<this.n; i++) {
                if (targets[i+1].connections.length == 0) {
                    targets[i+1].setStyle({fill: "#F44336"})
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
            return 60 + this.n*27
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