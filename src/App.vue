<template>
    <div id="app">
        <v-app>
            <v-alert type="error" v-model="error" dismissible dense class="ma-0">{{ errorText }}</v-alert>
            <v-dialog max-width="500px" max-height="400px" v-model="outputDialog"><v-card>
                <v-card-title>Recursive Function Outputs</v-card-title>
                <v-card-text class="text-body-1">
                    <div v-for="(out, i) in outputs" :key="'K'+i">
                        Output {{ i+1 }}: {{ out === undefined ? "undefined" : out }}
                    </div>
                </v-card-text>
            </v-card></v-dialog>
            <v-dialog v-model="saveDialog" max-width="200px"><v-card>
                <div style="min-height: 100px;">
                    <v-container><v-row><v-col>
                        <v-text-field label="Filename" v-model="filename" hide-details :rules="rules.filename"></v-text-field>
                        <v-switch label="Save as text" v-model="outputText" dense hide-details></v-switch>
                        <v-btn icon @click="saveBoard" :disabled="filename.length == 0" style="float: right; margin-top: 5px; margin-bottom: -5px;"><v-icon>mdi-download</v-icon></v-btn>
                    </v-col></v-row></v-container>
                </div>
            </v-card></v-dialog>
            <v-dialog v-model="lemmaDialog" max-width="200px"><v-card>
                <div style="min-height: 100px;">
                    <v-container><v-row><v-col>
                        <v-file-input accept=".rfuncs" v-model="lemmaFile" label="Lemma File"></v-file-input>
                        <v-text-field label="Lemma Name" v-model="lemmaName" hide-details :rules="rules.filename"></v-text-field>
                        <v-btn icon @click="openBoard(lemmaFile, 'lemma')" :disabled="lemmaName.length == 0 || lemmaFile == undefined" style="float: right; margin-top: 5px; margin-bottom: -5px;"><v-icon>mdi-upload</v-icon></v-btn>
                    </v-col></v-row></v-container>
                </div>
            </v-card></v-dialog>
            <v-file-input accept=".rfuncs" v-model="file" v-show="false" ref="openInput"></v-file-input>
            <v-tabs-items v-model="tab">
                <v-tab-item :key="0" style="height: 100%" ref="boardTab">
                    <div class="scroll-window hide-scroll" ref="scrollWindow"><div ref="boardElem" :style="{width: boardWidth(), height: boardHeight()}" class="dropzone">
                        <div class="banner">
                            <v-btn @click="runFuncs" style="margin-left: 2px; padding: 0px 12px;">Run Functions</v-btn>
                            <v-btn @click="saveDialog = true" style="margin-left: 2px; padding: 0px 12px;">Save Board</v-btn>
                            <v-btn @click="clickInput" style="margin-left: 2px; padding: 0px 12px;">Load Board</v-btn>
                            <v-btn @click="expandBoard" style="margin-left: 2px; padding: 0px 12px;">Expand Board</v-btn>
                            <v-icon x-large style="margin-left: 5px; z-index: 1;" @mouseover="hoverTrash=true" @mouseleave="hoverTrash=false">
                                {{ hoverTrash && dragging ? "mdi-delete-empty" : "mdi-delete" }}
                            </v-icon>
                            <v-btn @click="clearBoard" style="margin-left: 2px; padding: 0px 12px;">Clear Board</v-btn>
                        </div>
                        <div
                            v-for="func in funcs"
                            :key="'B'+func.id"
                            :ref="'func'+func.id"
                            class="func prevent-select"
                            :id="func.component + '-' + func.id"
                        ><component :is="func.component" :jsplumb="instance" :id="func.id" :ref="'funccomponent'+func.id"></component></div>
                    </div></div>
                    <v-navigation-drawer app v-model="funcBar" permanent width="210px" style="position: absolute; background-color: rgb(200, 200, 200); z-index: 2;">
                        <v-list dense>
                            <v-list-item-title class="text-h6 pl-2 prevent-select">
                                Functions
                            </v-list-item-title>
                            <v-divider></v-divider>
                            <span v-for="(func, i) in funcNames" :key="i">
                                <v-subheader v-if="func.subheader">{{ func.name }}</v-subheader>
                                <v-list-item v-else-if="func.button" @click="func.name == 'AddLemma' ? lemmaDialog = true : ''">
                                    <v-list-item-icon>
                                        <v-icon v-text="func.icon"></v-icon>
                                    </v-list-item-icon>
                                    <v-list-item-content>
                                        <v-list-item-title v-text="func.displayName" class="prevent-select"></v-list-item-title>
                                    </v-list-item-content>
                                </v-list-item>
                                <v-list-item v-else style="background-color: rgb(220, 220, 220); cursor: grab" class="drag-drop" :componentName="func.name" :shortcut="func.shortcut" :lemma="func.lemma">
                                    <v-list-item-icon>
                                        <v-icon v-text="func.icon"></v-icon>
                                        <sub v-if="func.iconCount !== undefined">{{ func.iconCount }}</sub>
                                    </v-list-item-icon>
                                    <v-list-item-content>
                                        <v-list-item-title v-text="func.displayName" class="prevent-select"></v-list-item-title>
                                    </v-list-item-content>
                                </v-list-item>
                                <v-divider></v-divider>
                            </span>
                        </v-list>
                    </v-navigation-drawer>
                </v-tab-item>
            </v-tabs-items>
        </v-app>
    </div>
</template>

<script>
// CONTACT thenamor2@gmail.com FOR QUESTIONS
// Please mention "RecursiveFunctions" in the subject

import ZeroFunc from './components/ZeroFunc.vue';
import SuccFunc from './components/SuccFunc.vue';
import IdFunc from './components/IdFunc.vue';
import InputFunc from './components/InputFunc.vue';
import OutputFunc from './components/OutputFunc.vue';
import PrimitiveRecursion from './components/PrimitiveRecursion.vue';
import MinimizationFunc from './components/MinimizationFunc.vue';
import CompositionFunc from './components/CompositionFunc.vue';

const interact = require('interactjs')
const jsplumb = require('jsplumb')

export default {
    name: 'App',

    components: {
        ZeroFunc, SuccFunc, IdFunc, InputFunc, OutputFunc, PrimitiveRecursion, MinimizationFunc, CompositionFunc
    },

    watch: {
        file() {
            this.openBoard(this.file, "board")
        },
    },

    // Runs when the app is first loaded
    mounted() {

        document.addEventListener('dragstart', function (event) {
            // use interact.js' matchesSelector polyfil to
            // check for match with your draggable target
            if (event.target.prefixedMatchesSelector == undefined) {
                return
            }
            if (interact.matchesSelector(event.target, '.drag-drop, .drag-drop *')) {
                // prevent and stop the event if it's on a draggable target
                event.preventDefault();
                event.stopPropagation();
            }
        });

        let self = this
        
        // Add draggables for func bar
        interact('.drag-drop').draggable({
        'manualStart' : true,
        'restrict' : {
            'endOnly' : true,
            'elementRect' : { 
            'top' : 0, 
            'left' : 0, 
            'bottom' : 1, 
            'right' : 1 
            }
        },
        'onmove' : function (event) {

            var target = event.target;

            var x = (parseFloat(target.getAttribute('data-x')) || 0) + event.dx;
            var y = (parseFloat(target.getAttribute('data-y')) || 0) + event.dy;
    
            // translate the element
            target.style.webkitTransform = target.style.transform = 'translate(' + x + 'px, ' + y + 'px)';
    
            // update the position attributes
            target.setAttribute('data-x', x);
            target.setAttribute('data-y', y);

        },
        'onend' : function (event) {
            
            event.currentTarget.remove()

        }
        }).on('move', function (event) {

            var interaction = event.interaction;

            // if the pointer was moved while being held down
            // and an interaction hasn't started yet
            if (interaction.pointerIsDown && !interaction.interacting()) {
                
                // create a clone of the currentTarget element
                var clone = event.currentTarget.cloneNode(true);
                var targetBounding = event.currentTarget.getBoundingClientRect();

                clone.style.webkitTransform = clone.style.transform = "translate(" + (targetBounding.left + window.scrollX) + "px, " + (targetBounding.top + window.scrollY) + "px)";
                clone.style.position = "absolute";
                clone.style.zIndex = "10"
                clone.style.height = "40px"
                clone.style.width = "200px"

                clone.setAttribute('data-x', (targetBounding.left + window.scrollX))
                clone.setAttribute('data-y', (targetBounding.top + window.scrollY))

                self.$refs.boardTab.$el.appendChild(clone);

                // start a drag interaction targeting the clone
                interaction.start({ name: 'drag' }, event.interactable, clone);

            }

        });

        // Init the dropzone for the board
        interact('.dropzone').dropzone({
            // only accept elements matching this CSS selector
            accept: '.drag-drop',
            // Require a 50% element overlap for a drop to be possible
            overlap: 0.5,

            // Adds a new func to the board
            ondrop: function(event) {
                let target = event.relatedTarget
                var x = (parseFloat(target.getAttribute('data-x')) || 0) - 105 -
                    (self.componentWidth[target.getAttribute("componentName")] === undefined ? 
                    0 : self.componentWidth[target.getAttribute("componentName")]({n:1}))/2 +
                    (self.$refs.scrollWindow ? (window.pageXOffset || self.$refs.scrollWindow.scrollLeft) : 0)
                var y = (parseFloat(target.getAttribute('data-y')) || 0) + (self.$refs.scrollWindow ? (window.pageYOffset || self.$refs.scrollWindow.scrollTop) : 0)
                if (target.getAttribute("shortcut") == "true") {
                    self.loadBoard(self.shortcuts[target.getAttribute("componentName")], [x, y])
                    return
                }
                if (target.getAttribute("lemma") == "true") {
                    self.loadBoard(self.lemmas[target.getAttribute("componentName")], [x, y])
                    return
                }
                self.func_id++
                let id = self.func_id
                self.funcs.push({
                    id: id,
                    x: Math.max(0, x),
                    y: y,
                    inputs: [],
                    component: target.getAttribute("componentName")
                })
                self.$nextTick(() => {
                    self.addFunc(id)
                })
            },
        })

        this.instance = jsplumb.jsPlumb.getInstance({
            Container: this.$refs.boardElem,
            Connector : [ "Flowchart", {cornerRadius: 5, stub: 25} ],
            EndpointStyle: { fill: "black" },
        })

        this.instance.bind('beforeDrop', function(ci){ // Before new connection is created
            var src=ci.sourceId;
            if (
                    src == "InputFunc-0" ||
                    (
                        (src.includes("PrimitiveRecursion") || src.includes("MinimizationFunc")) &&
                        ci.connection.endpoints[0].getParameter("oneOut") === undefined
                    ) ||
                    ci.connection.endpoints[0].getParameter("oneOut") === false
                ) {
                return true
            }
            var con=self.instance.getConnections({source:src}); // Get all source el. connection(s) except the new connection which is being established 
            con = con.filter(connection => connection.endpoints[1] != ci.connection.endpoints[1])
            if(con.length!=0){
                for(var i=0;i<con.length;i++){
                    if (
                        !(src.includes("PrimitiveRecursion") || src.includes("MinimizationFunc")) ||
                        con[i].endpoints[0].getParameter("oneOut")
                    ) {
                        self.instance.deleteConnection(con[i])
                    }
                }
            }
            return true; // true for establishing new connection
        })

        this.instance.bind('connection', function(ci){
            ci.connection.endpoints[1].setStyle({fill: "black"})
            let source = ci.sourceId
            const to = ci.sourceEndpoint.getParameter("to")
            let targetOut = self.instance.getEndpoints(ci.targetId).filter(endpoint => endpoint.isSource)
            if (targetOut.length > 0) {
                let trueOuts = targetOut.filter(endpoint => endpoint.getParameter("trueOut"))
                if (trueOuts.length > 0) {
                    targetOut = trueOuts[0]
                } else {
                    targetOut = targetOut[0]
                }
            }
            if (to !== undefined) {
                let final = self.instance.getEndpoints(source).filter(endpoint => endpoint.getParameter("from") == to)
                if (final.length > 0) {
                    if (final[0].connections.length == 0 && targetOut.connections.length == 0) {
                        self.instance.connect({source: targetOut, target: final[0]})
                    }
                }
            }
        })

        this.addFunc(0, true)
        this.addFunc(1, true)

        /*jsplumb.jsPlumb.addEndpoint(this.$refs.func0[0], {
            isSource: true,
            anchor: "Right",
        })*/
    },

    data: () => ({
        error: false,
        errorText: "",
        tab: 0,
        funcBar: true,        // Dummy variable to keep the func bar open
        funcNames: [          // List of available funcs for the func bar
            { name: "Primitive Functions", subheader: true },
            { name: "ZeroFunc", displayName: "Zero", icon: "mdi-numeric-0-circle" },
            { name: "SuccFunc", displayName: "Successor", icon: "mdi-plus" },
            { name: "IdFunc", displayName: "Identity", icon: "mdi-identifier" },
            { name: "Operations on Functions", subheader: true },
            { name: "CompositionFunc", displayName: "Composition", icon: "mdi-call-split" },
            { name: "PrimitiveRecursion", displayName: "Primitive Recursion", icon: "mdi-restore" },
            { name: "MinimizationFunc", displayName: "Minimization", icon: "mdi-arrow-collapse-all" },
            { name: "Shortcuts", subheader: true },
            { name: "PredecessorCut", displayName: "Predecessor", icon: "mdi-skip-previous", shortcut: true},
            { name: "SumCut", displayName: "Sum", icon: "mdi-plus-box", shortcut: true},
            { name: "DiffCut", displayName: "Difference", icon: "mdi-minus-box", shortcut: true},
            { name: "SignCut", displayName: "Sign", icon: "mdi-timeline-plus-outline", shortcut: true},
            { name: "InSignCut", displayName: "Inverse Sign", icon: "mdi-timeline-plus", shortcut: true},
            { name: "Lemmas", subheader: true },
            { name: "AddLemma", displayName: "Add Lemma", icon: "mdi-plus", button: true }
        ],
        componentWidth: {
            ZeroFunc: () => { return 85 },
            SuccFunc: () => { return 100 },
            IdFunc: () => { return 120 },
            CompositionFunc: (vars) => { return 135 + vars.n*20 },
            PrimitiveRecursion: (vars) => { return 135 + vars.n*20 },
            MinimizationFunc: (vars) => { return 135 + vars.n*20 },
        },
        shortcuts: {
            PredecessorCut: [
                {"id":2,"x":0,"y":0,"inputs":[{"source":2,"outputInd":2,"sortBy":0},{"source":2,"outputInd":1,"sortBy":1}],"component":"PrimitiveRecursion","vars":{"n":1}}
            ],
            SumCut: [
                {"id":2,"x":588,"y":250,"inputs":[{"source":2,"outputInd":4,"sortBy":0},{"source":3,"sortBy":1}],"component":"PrimitiveRecursion","vars":{"n":1}},
                {"id":3,"x":633,"y":397,"inputs":[{"source":2,"outputInd":3,"sortBy":0}],"component":"SuccFunc","vars":{}}
            ],
            DiffCut: [
                {"id":2,"x":316,"y":217,"inputs":[{"source":2,"outputInd":4,"sortBy":0},{"source":3,"outputInd":0,"sortBy":1}],"component":"PrimitiveRecursion","vars":{"n":1}},
                {"id":3,"x":330,"y":399,"inputs":[{"source":3,"outputInd":2,"sortBy":0},{"source":3,"outputInd":1,"sortBy":1},{"source":2,"outputInd":3,"sortBy":2}],"component":"PrimitiveRecursion","vars":{"n":1}}
            ],
            SignCut: [
                {"id":4,"x":337.5,"y":262.4,"inputs":[{"source":4,"outputInd":2,"sortBy":0},{"source":7,"sortBy":1}],"component":"PrimitiveRecursion","vars":{"n":1}},
                {"id":6,"x":315,"y":413,"inputs":[{"source":4,"outputInd":1,"sortBy":0}],"component":"ZeroFunc","vars":{}},
                {"id":7,"x":450,"y":413,"inputs":[{"source":6,"sortBy":0}],"component":"SuccFunc","vars":{}}
            ],
            InSignCut: [
                {"id":2,"x":374.4,"y":222.3,"inputs":[{"source":4,"sortBy":0},{"source":5,"sortBy":1}],"component":"PrimitiveRecursion","vars":{"n":1}},
                {"id":4,"x":420,"y":155,"inputs":[{"source":2,"outputInd":2,"sortBy":0}],"component":"SuccFunc","vars":{}},
                {"id":5,"x":424,"y":372,"inputs":[{"source":2,"outputInd":1,"sortBy":0}],"component":"ZeroFunc","vars":{}}
            ]
        },
        lemmas: [],
        instance: undefined,
        func_id: 1,
        funcs: [
            { id: 0, x: 0, y: 120, inputs: [], component: "InputFunc" },
            { id: 1, x: 0, y: 60, inputs: [], component: "OutputFunc" },
        ],
        rules: {
            filename: [v => v.length > 0 || "Name Required"]
        },
        outputs: [],
        outputDialog: false,
        outputText: false,
        filename: "",
        saveDialog: false,
        file: undefined,
        dragging: false,
        hoverTrash: false,
        lemmaDialog: false,
        lemmaFile: undefined,
        lemmaName: "",
        expand: { x: 0, y: 0 },
    }),

    methods: {
        addFunc(id, fixed) {
            let funcEl = this.$refs["func"+id][0]
            let self = this
            if (fixed === undefined) {
                this.instance.draggable(funcEl, {
                    start: () => {
                        self.dragging = true
                    },
                    stop: () => {
                        self.dragging = false
                        if (self.hoverTrash) {
                            self.funcs = self.funcs.filter(func => func.id != id)
                            self.instance.remove(funcEl)
                        }
                    },
                })
                funcEl.classList.add("grab")
            }
            funcEl.style.left = this.funcById[id].x + "px"
            funcEl.style.top = this.funcById[id].y + "px"
        },
        clearBoard() {
            this.funcs.forEach((func, i) => {
                if (i > 1) {
                    this.instance.remove(this.$refs["func"+func.id][0])
                }
            })
            this.funcs.splice(2)
            this.expand.x = 0
            this.expand.y = 0
        },
        saveFuncs() {
            let funcData = {}
            let filename = "funcs.rfuncs"
            var file = new Blob([JSON.stringify(funcData)], {type: "text/plain"})
            if (window.navigator.msSaveOrOpenBlob) // IE10+
                window.navigator.msSaveOrOpenBlob(file, filename)
            else { // Others
                var a = document.createElement("a"),
                        url = URL.createObjectURL(file)
                a.href = url
                a.download = filename
                document.body.appendChild(a)
                a.click()
                setTimeout(function() {
                    document.body.removeChild(a)
                    window.URL.revokeObjectURL(url)
                }, 0)
            }
        },
        syncPositions() {
            let x
            let y
            let funcEl
            this.funcs.forEach(func => {
                funcEl = this.$refs["func" + func.id][0]
                x = parseFloat(funcEl.style.left.slice(0, -2))
                y = parseFloat(funcEl.style.top.slice(0, -2))
                func.x = x
                func.y = y
            })
        },
        syncVariables() {
            let funcComp
            this.funcs.forEach(func => {
                funcComp = this.$refs["funccomponent" + func.id][0]
                func.vars = { n: funcComp.n, m: funcComp.m }
            })
        },
        syncConnections() {
            this.funcs.forEach(func => {
                func.inputs = []
            })
            let connections = this.instance.getAllConnections()
            connections.forEach(connection => {
                let target = this.funcById[parseInt(connection.targetId.split('-')[1])]
                let source = this.funcById[parseInt(connection.sourceId.split('-')[1])]
                target.inputs.push({
                    source: source.id,
                    outputInd: connection.endpoints[0].getParameter("outputInd"),
                    sortBy: connection.endpoints[1].getParameter("sortBy") === undefined ? 0 : connection.endpoints[1].getParameter("sortBy"),
                })
            })
            this.funcs.forEach(func => {
                func.inputs.sort((a,b) => { return a.sortBy - b.sortBy })
            })
        },
        runFuncs() {
            this.error = false
            this.syncConnections()
            let funcComp
            this.funcs.forEach(func => {
                funcComp = this.$refs["funccomponent" + func.id][0]
                if (funcComp.checkInputs !== undefined) {
                    if (funcComp.checkInputs()) {
                        this.error = true
                        this.errorText = "Error: Missing input connections detected"
                    }
                }
            })
            if (this.error) {
                return
            }
            try {
                this.outputs = this.getInputs(1, 0)
                this.funcs.forEach(func => {
                    func.inputs = []
                    func.y0 = undefined
                    func.depth = undefined
                    func.depthR = undefined
                    func.xs = undefined
                })
                if (this.outputs.length > 0) {
                    this.outputDialog = true
                }
            } catch (ex) {
                this.error = true
                if (ex instanceof TypeError) {
                    this.errorText = "Error: Missing input connections"   
                } else {
                    this.errorText = "Error: infinite recursion detected"
                }
            }
        },
        getInputs(funcId, index, forceRefresh) {
            let func = this.funcById[funcId]
            const component = this.$refs["funccomponent" + func.id][0]
            if (funcId == 0) {
                return this.$refs["funccomponent0"][0].valueNums[index]
            }
            if (func.component == "CompositionFunc") {
                if (func.xs === undefined || forceRefresh !== undefined) {
                    func.xs = []
                    for (let i=0; i<func.inputs.length; i++) {
                        func.xs.push(this.getInputs(func.inputs[i].source, func.inputs[i].outputInd))
                    }
                }
                return func.xs[index]
            } else if (func.component == "PrimitiveRecursion") {
                if (func.y0 === undefined || forceRefresh !== undefined) {
                    if (func.xs === undefined) {
                        func.xs = []
                    }
                    if (func.xs.length < (func.inputs.length-3)) {
                        for (let i=0; i<(func.inputs.length-3-func.xs.length); i++) {
                            func.xs.push(undefined)
                        }
                    }
                    for (let i=3; i<func.inputs.length; i++) {
                        func.xs[i-3] = this.getInputs(func.inputs[i].source, func.inputs[i].outputInd, forceRefresh)
                    }
                    func.y0 = this.getInputs(func.inputs[2].source, func.inputs[2].outputInd, forceRefresh)
                }
                // Get true output
                if (index == 0) {
                    func.depth = func.y0
                    func.depthR = func.y0
                    if (func.depth == 0) {
                        return this.getInputs(func.inputs[0].source, func.inputs[0].outputInd)
                    }
                    func.depth--
                    let out = this.getInputs(func.inputs[1].source, func.inputs[1].outputInd)
                    func.y0 = undefined
                    return out
                }
                // Get y-0 outputs
                if (index % 2 == 0) {
                    if (index == 2) {
                        return 0
                    } else {
                        return func.xs[index/2-2]
                    }
                }
                // Get y>0 outputs
                if (index == 3) {
                    if (func.depth == 0) {
                        func.depth = func.y0
                        return this.getInputs(func.inputs[0].source, func.inputs[0].outputInd)
                    }
                    func.depth--
                    return this.getInputs(func.inputs[1].source, func.inputs[1].outputInd)
                } else if (index == 1) {
                    func.depthR--
                    return func.depthR
                } else if (index % 2 == 1) {
                    return func.xs[Math.floor(index/2)-2]
                }
            } else if (func.component == "MinimizationFunc") {
                if (func.xs === undefined || forceRefresh !== undefined) {
                    func.xs = []
                    for (let i=1; i<func.inputs.length; i++) {
                        func.xs.push(this.getInputs(func.inputs[i].source, func.inputs[i].outputInd, forceRefresh))
                    }
                }
                if (index == 0) {  // Get true output
                    const limit = 10000
                    func.depth = 0
                    while (func.depth <= limit) {
                        if (this.getInputs(func.inputs[0].source, func.inputs[0].outputInd, true) == 0) {
                            return func.depth
                        }
                        func.depth++
                    }
                    return undefined
                } else if (index == 1) {    // Get y
                    return func.depth
                } else if (index > 1) {     // Get xs
                    return func.xs[index-2]
                }
            } else {
                let inputs = []
                func.inputs.forEach(inputId => {
                    let outs = this.getInputs(inputId.source, inputId.outputInd)
                    if (Array.isArray(outs)) {
                        inputs.push(outs[index])
                    } else {
                        inputs.push(outs)
                    }
                })
                switch (func.component) {
                    case "ZeroFunc":
                        return this.zero(inputs[0])
                    case "SuccFunc":
                        return this.s(inputs[0])
                    case "IdFunc":
                        return this.id(component.n, component.m, inputs)
                    default:
                        return inputs
                }
            }
        },
        // eslint-disable-next-line
        zero(_) {
            return 0
        },
        s(x) {
            return x+1
        },
        id(n, m, x) {
            if (n != x.length) {
                console.log("error incorrect number of inputs:")
                console.log(x)
            }
            return x[m-1]
        },
        runText() {
            this.syncConnections()
            return this.getText(1, 0)
        },
        getText(funcId, index) {
            let func = this.funcById[funcId]
            const component = this.$refs["funccomponent" + func.id][0]
            let out = ""
            if (funcId == 0) {
                return "x" + (index+1)
            }
            if (func.component == "CompositionFunc") {
                out = ""
                return this.getText(func.inputs[index].source, func.inputs[index].outputInd)
            } else if (func.component == "PrimitiveRecursion") {
                // Get true output
                if (index == 0) {
                    let first = this.getText(func.inputs[0].source, func.inputs[0].outputInd)
                    let second = this.getText(func.inputs[1].source, func.inputs[1].outputInd)
                    if (first.includes("from")) {
                        first = first.slice(6)
                    }
                    if (second.includes("from")) {
                        second = second.slice(6)
                    }
                    out = "Pr[" +
                        first +
                        "," +
                        second +
                        "]("
                    let inputs = []
                    for (let i=3; i<func.inputs.length; i++) {
                        inputs.push(this.getText(func.inputs[i].source, func.inputs[i].outputInd))
                    }
                    inputs.push(this.getText(func.inputs[2].source, func.inputs[2].outputInd))
                    if (inputs.every(input => input.includes("from"))) {
                        return "Pr[" + first + "," + second + "]"
                    }
                    inputs.forEach((input, i) => {
                        if (input.includes("from")) {
                            out = out + input.slice(6)
                        } else {
                            out = out + input
                        }
                        if (i+1 != inputs.length) {
                            out = out + ", "
                        }
                    })
                    return out + ")"
                }
                // Get y-0 outputs
                if (index % 2 == 0) {
                    if (index == 2) {
                        return "fromPrid[" + (func.inputs.length-2) + "," + (func.inputs.length-2) + "]"
                    } else {
                        return "fromPrid[" + (func.inputs.length-2) + "," + (index/2-2+1) + "]"
                    }
                }
                // Get y>0 outputs
                if (index == 3) {
                    return "fromPrid[" + (func.inputs.length-1) + "," + (func.inputs.length-1) + "]"
                } else if (index == 1) {
                    return "fromPrid[" + (func.inputs.length-1) + "," + (func.inputs.length-2) + "]"
                } else if (index % 2 == 1) {
                    return "fromPrid[" + (func.inputs.length-1) + "," + (Math.floor(index/2)-2+1) + "]"
                }
            } else if (func.component == "MinimizationFunc") {
                if (index == 0) {  // Get true output
                    let first = this.getText(func.inputs[0].source, func.inputs[0].outputInd)
                    if (first.includes("from")) {
                        first = first.slice(6)
                    }
                    out = "Mn[" +
                        first +
                        "]("
                    for (let i=1; i<func.inputs.length; i++) {
                        out = out + this.getText(func.inputs[i].source, func.inputs[i].outputInd) + (i+1 == func.inputs.length ? "" : ", ")
                    }
                    return out + ")"
                } else if (index == 1) {    // Get y
                    return "fromMnid[" + func.inputs.length + "," + (func.inputs.length-1) + "]"
                } else if (index > 1) {     // Get xs
                    return "fromPrid[" + func.inputs.length + "," + (index-2+1) + "]"
                }
            } else {
                let inputs = []
                func.inputs.forEach(inputId => {
                    inputs.push(this.getText(inputId.source, inputId.outputInd))
                })
                switch (func.component) {
                    case "ZeroFunc":
                        if (inputs[0].includes("from")) {
                            inputs[0] = inputs[0].slice(6)
                        }
                        return "z(" + inputs[0] + ")"
                    case "SuccFunc":
                        if (inputs[0].includes("from")) {
                            inputs[0] = inputs[0].slice(6)
                        }
                        return "s(" + inputs[0] + ")"
                    case "IdFunc":
                        if (inputs.every(input => input.includes("from"))) {
                            return "id[" + component.n + "," + component.m + "]"
                        }
                        out = "id[" + component.n + "," + component.m + "]("
                        inputs.forEach((input, i) => {
                            if (input.includes("from")) {
                                out = out + input.slice(6)
                            } else {
                                out = out + input
                            }
                            if (i+1 != inputs.length) {
                                out = out + ","
                            }
                        })
                        return out + ")"
                    default:
                        if (inputs.length == 0) {
                            return ""
                        }
                        if (inputs.length == 1) {
                            return inputs[0]
                        }
                        return inputs
                }
            }
        },
        getBoard() {
            this.syncConnections()
            this.syncPositions()
            this.syncVariables()
            return this.funcs
        },
        loadBoard(funcsIn, coords) {
            if (coords !== undefined) {
                let funcXs = funcsIn.map(func => func.x)
                let funcYs = funcsIn.map(func => func.y)
                let x_range = [Math.min(...funcXs), Math.max(...funcXs)]
                let y_range = [Math.min(...funcYs), Math.max(...funcYs)]
                let mostRight = funcsIn[funcXs.indexOf(x_range[1])]
                let width = (x_range[1]-x_range[0]) + this.componentWidth[mostRight.component](mostRight.vars)
                let center = [(width)/2+x_range[0], (y_range[1]-y_range[0])/2+y_range[0]]
                funcsIn.forEach(func => {
                    func.x = func.x - center[0] + coords[0]
                    func.y = func.y - center[1] + coords[1]
                })
            }
            let self = this
            let idMap = {0: 0, 1: 1}
            funcsIn.filter(func => func.id > 1).forEach(func => {
                this.func_id++
                this.funcs.push({
                    id: this.func_id,
                    x: Math.max(0, func.x),
                    y: func.y,
                    inputs: [],
                    component: func.component
                })
                idMap[func.id] = this.func_id
            })
            this.$nextTick(() => {
                funcsIn.filter(func => func.id > 1).forEach(func => {
                    self.addFunc(idMap[func.id])
                })
                self.$nextTick(() => {
                    funcsIn.forEach(func => {
                        let funcComp = self.$refs["funccomponent" + idMap[func.id]][0]
                        if (funcComp.setData !== undefined) {
                            funcComp.setData(func.vars)
                        }
                        let funcEl = self.$refs["func" + idMap[func.id]]
                        let allEndpoints = self.instance.getEndpoints(funcEl)
                        func.sources = allEndpoints.filter(endpoint => endpoint.isSource == true).sort((a,b) => { return a.getParameter("outputInd") - b.getParameter("outputInd") })
                        func.targets = allEndpoints.filter(endpoint => endpoint.isTarget == true).sort((a,b) => { return a.getParameter("sortBy") - b.getParameter("sortBy") })
                    })
                    funcsIn.forEach(func => {
                        func.inputs.forEach(connection => {
                            if (coords === undefined || connection.source > 1) {
                                let target = func.targets[connection.sortBy]
                                let source = funcsIn.filter(id_search => id_search.id == connection.source)[0].sources[connection.outputInd || 0]
                                self.instance.connect({source: source, target: target})
                            }
                        })
                    })
                })
            })
        },
        saveBoard() {
            let filename
            let boardData
            var file
            if (!this.outputText) {
                filename = this.filename + ".rfuncs"
                boardData = this.getBoard()
                file = new Blob([JSON.stringify(boardData)], {type: "text/plain"})
            } else {
                filename = this.filename + ".txt"
                boardData = this.runText()
                file = new Blob([boardData], {type: "text/plain"})
            }
            
            if (window.navigator.msSaveOrOpenBlob) // IE10+
                window.navigator.msSaveOrOpenBlob(file, filename)
            else { // Others
                var a = document.createElement("a"),
                        url = URL.createObjectURL(file)
                a.href = url
                a.download = filename
                document.body.appendChild(a)
                a.click()
                setTimeout(function() {
                    document.body.removeChild(a)
                    window.URL.revokeObjectURL(url)
                }, 0)
            }
            this.filename = ""
            this.saveDialog = false
        },
        clickInput() {
            this.$refs.openInput.$refs.input.click()
        },
        openBoard(file, type) {
            if (file === undefined || file.name === undefined || file.name.length == 0 || file.name.split('.').pop().toLowerCase() != "rfuncs") {
                return
            }
            const reader = new FileReader()
            let self = this
            reader.onload=function(){
                let newBoard = JSON.parse(reader.result)
                if (type == "board") {
                    self.loadBoard(newBoard)
                    self.file = undefined
                } else if (type == "lemma") {
                    self.addLemma(newBoard, self.lemmaName)
                    self.lemmaFile = undefined
                    self.lemmaName = ""
                    self.lemmaDialog = false
                }
            }

            reader.readAsText(file)
        },
        addLemma(funcsIn, name) {
            let lemmaFunc = { name: this.lemmas.length, displayName: name, icon: "mdi-lambda", lemma: true, iconCount: this.lemmas.length}
            this.funcNames.splice(this.funcNames.length-1, 0, lemmaFunc)
            funcsIn = funcsIn.filter(func => func.id > 1)
            funcsIn.forEach(func => {
                func.inputs = func.inputs.filter(input => input.source != 0)
            })
            this.lemmas.push(funcsIn)
        },
        expandBoard() {
            this.expand.x += 500
            this.expand.y += 500
        },
        boardHeight() {
            let height = this.$refs.boardTab === undefined ? 500 : this.$refs.boardTab.$el.clientHeight-1
            return (height + this.expand.y) + "px"
        },
        boardWidth() {
            let width = this.$refs.boardTab === undefined ? 500 : this.$refs.boardTab.$el.clientWidth-210
            return (width + this.expand.x) + "px"
        }
    },

    computed: {
        funcById() {
            return this.funcs.reduce((a, v) => ({ ...a, [v.id]: v}), {})
        },
    },
}
</script>

<style>
#app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    height: 100%;
}

.v-tabs-items {
    height: 100%;
}

.dropzone {
    position: relative;
    left: 210px;
    background-image: radial-gradient(circle at 1px 1px, black 1px, transparent 0);
    background-size: 20px 20px;
    background-repeat: repeat;
}

.scroll-window {
    position: absolute;
    overflow: auto;
    height: 100%;
    width: 100%;
}

.hide-scroll {
    -ms-overflow-style: none;  /* Internet Explorer 10+ */
    scrollbar-width: none;  /* Firefox */
}
.hide-scroll::-webkit-scrollbar { 
    display: none;  /* Safari and Chrome */
}

.func {
    position: absolute;
    border-radius: 3px;
    border: solid black 2px;
}

.grab {
    cursor: grab;
}

.prevent-select {
  -webkit-user-select: none; /* Safari */
  -ms-user-select: none; /* IE 10 and IE 11 */
  user-select: none; /* Standard syntax */
}

.banner {
    height: 42px;
    background-color: lightgray;
    padding: 2px;
}

</style>
