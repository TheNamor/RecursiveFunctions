<template>
    <div class="zero" ref="wrapper">
        z(x) = 0
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

    }),
    
    methods: {
        checkInputs() {
            const endpoints = this.jsplumb.getEndpoints(this.drag)
            const source = endpoints.filter(endpoint => endpoint.isSource)[0]
            const target = endpoints.filter(endpoint => endpoint.isTarget)[0]
            if (source.connections.length > 0 && target.connections.length == 0) {
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

.zero {
    height: 40px;
    width: 85px;
    text-align: center;
    line-height: 40px;
    background-color: grey;
}

</style>