<template>
    <div class="id-style" ref="wrapper">
        id<sup>{{ n }}</sup><sub>{{ m }}</sub> = x<sub>{{ m }}</sub>
        <div style="padding-left: 15px">
            n = {{ n }}
            <v-btn x-small icon @click="changeVal('n', 1)"><v-icon>mdi-plus</v-icon></v-btn>
            <v-btn x-small icon @click="changeVal('n', -1)"><v-icon>mdi-minus</v-icon></v-btn>
        </div>
        <div style="padding-left: 15px">
            m = {{ m }}
            <v-btn x-small icon @click="changeVal('m', 1)"><v-icon>mdi-plus</v-icon></v-btn>
            <v-btn x-small icon @click="changeVal('m', -1)"><v-icon>mdi-minus</v-icon></v-btn>
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
            this.jsplumb.addEndpoint(this.drag, {
                isTarget: true,
                isSource: false,
                anchor: "Left",
                endpoint: ["Rectangle", {height: 25, width: 15}],
                maxConnections: 1,
            })
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
            })
        })
    },

    data: () => ({
        n: 1,
        m: 1,
    }),

    methods: {
        changeVal(which, val) {
            if (which == "n") {
                this.n += val
                this.n = Math.max(this.n, 1)
                this.m = Math.min(this.n, this.m)
                const inputs = this.jsplumb.getEndpoints(this.drag).filter(endpoint => { return endpoint.isTarget })[0]
                this.jsplumb.deleteEndpoint(inputs)
                this.jsplumb.addEndpoint(this.drag, {
                    isTarget: true,
                    isSource: false,
                    anchor: "Left",
                    endpoint: ["Rectangle", {height: 25, width: 15}],
                    maxConnections: this.n,
                })
            } else {
                this.m += val
                this.m = Math.min(this.n, Math.max(this.m, 1))
            }
        },
        setData(vars) {
            if (this.n != vars.n) {
                this.changeVal("n", vars.n - this.n)
            }
            if (this.m != vars.m) {
                this.changeVal("m", vars.m - this.m)
            }
        },
        checkInputs() {
            const endpoints = this.jsplumb.getEndpoints(this.drag)
            const source = endpoints.filter(endpoint => endpoint.isSource)[0]
            const target = endpoints.filter(endpoint => endpoint.isTarget)[0]
            if (source.connections.length > 0 && target.connections.length < this.n) {
                target.setStyle({fill: "#F44336"})
                return true
            }
        }
    },

    computed: {
        drag() {
            return this.$parent.$parent.$parent.$parent.$refs["func"+this.id]
        },
    }
}

</script>

<style scoped>

.id-style {
    height: 75px;
    width: 120px;
    text-align: center;
    line-height: 25px;
    background-color: grey;
}

</style>