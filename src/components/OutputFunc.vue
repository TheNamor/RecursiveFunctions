<template>
    <div class="output" ref="wrapper">
        Output
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
                anchor: "Right",
                endpoint: ["Rectangle", {height: 25, width: 15}],
                maxConnections: -1,
            })
        })
    },

    data: () => ({

    }),

    computed: {
        drag() {
            return this.$parent.$parent.$parent.$parent.$refs["func"+this.id]
        },
    }
}

</script>

<style scoped>

.output {
    height: 40px;
    width: 85px;
    text-align: center;
    line-height: 40px;
    background-color: grey;
}

</style>