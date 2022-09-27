<script lang="ts">
    import {EditorView, basicSetup} from "codemirror";
    import {rust} from "@codemirror/lang-rust";

    export default {
        data() {
            return {
                
            }
        },
        mounted() {
            let parent = document.querySelector('.code-editor');
            if (parent === null) {
                throw new Error('Querying .code-editor returned null');
            }

            var view = new EditorView ({
                extensions: [basicSetup, rust()],
                parent
            });

            this.editor = view;

            function GetCodeResult() {
                let text = view.state.doc.lines;
                console.log(text);
            }
        },
        methods() {
            GetCodeResult: () => {
                let text = this.editor.state.doc.lines;
                console.log(text);
            }
        }
    }

    
</script>

<template>
    <div class="code-editor">

    </div>
    <button onclick="GetCodeResult()">
        Run
    </button>
</template>

<style>
    
</style>
