<template>
    <div class="input" ref="wrapper" :style="'height: ' + (height+values.length*30) + 'px'">
        {{ n }} inputs
        <v-btn x-small icon @click="addVal()"><v-icon>mdi-plus</v-icon></v-btn>
        <v-btn x-small icon @click="removeVal()" :disabled="values.length == 1"><v-icon>mdi-minus</v-icon></v-btn>
        <v-text-field
            v-for="(val, i) in values"
            v-model="val.value"
            :key="'C'+i"
            type="number"
            :rules="rules.noNeg"
            hide-details
            dense
            style="width: 80px; margin-top: 0px; margin-left: 20px; height: 30px"
            class="shrink"
        ></v-text-field>
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

    watch: {
        values: {
            handler(val) {
                val.forEach((item, i) => {
                    if (item.value.includes("-")) {
                        this.values[i].value = "0"
                    }
                })
            },
            deep: true
        }
    },

    mounted() {
        this.$nextTick(() => {
            this.jsplumb.addEndpoint(this.drag, {
                isSource: true,
                isTarget: false,
                anchor: [1, 0, 1, 0, 0, 47],
                endpoint: ["Rectangle", {height: 15, width: 25}],
                connectorOverlays:[
                    [ "Arrow", { location: 1, id:"arrow", foldback: 1 } ],
                ],
                maxConnections: -1,
                connectorStyle : { stroke: "black" },
                parameters: {
                    outputInd: this.n-1
                },
            })
        })
    },

    data: () => ({
        n: 1,
        values: [{ value: "0"}],
        dialog: false,
        height: 32,
        rules: {
            noNeg: [v => (v >= 0 && Math.floor(v) == v) || "< 0"]
        }
    }),

    methods: {
        addVal() {
            this.n++
            this.values.push({ value: "0"})
            this.jsplumb.addEndpoint(this.drag, {
                isSource: true,
                isTarget: false,
                anchor: [1, 0, 1, 0, 0, 17+30*this.n],
                endpoint: ["Rectangle", {height: 15, width: 25}],
                connectorOverlays:[
                    [ "Arrow", { location: 1, id:"arrow", foldback: 1 } ],
                ],
                maxConnections: -1,
                connectorStyle : { stroke: "black" },
                parameters: {
                    outputInd: this.n-1
                },
            })
        },
        removeVal() {
            this.n--
            this.values.pop()
            const endpoints = this.jsplumb.getEndpoints(this.drag)
            this.jsplumb.deleteEndpoint(endpoints[endpoints.length-1])
        },
        setData(vars) {
            while(this.n < vars.n) {
                this.addVal()
            }
        }
    },

    computed: {
        drag() {
            return this.$parent.$parent.$parent.$parent.$refs["func"+this.id]
        },
        valueNums() {
            return this.values.map(val => parseInt(val.value))
        },
    }
}

</script>

<style scoped>

.input {
    width: 120px;
    text-align: center;
    line-height: 30px;
    background-color: lightgray;
}

</style>