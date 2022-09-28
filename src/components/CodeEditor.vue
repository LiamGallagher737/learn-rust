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

            new EditorView ({
                extensions: [basicSetup, rust()],
                parent
            });
        },
        methods: {
            Compile: async () => {
                let parent = document.querySelector('.code-editor');
                if (parent === null) {
                    throw new Error('Querying .code-editor returned null');
                }

                let editor = EditorView.findFromDOM(parent as HTMLElement);
                if (editor === null) {
                    throw new Error('Failed finding editor within .code-editor');
                }

                let iter = editor.state.doc.iter();
                let line = iter.next();
                let code = '';
                while (!line.done) {
                    code += line.value;
                    line = iter.next();
                }

                let post_data = {
                    channel: "stable",
                    mode: "debug",
                    edition: "2021",
                    crateType: "bin",
                    tests: false,
                    code: code,
                    backtrace: false,
                }

                let response = await fetch("https://play.rust-lang.org/execute", {
                    method: "POST",
                    headers: {'Content-Type': 'application/json'},
                    body: JSON.stringify(post_data)
                });

                let body = await response.json();

                console.log(body);
            }
        }
    }

</script>

<template>
    <div class="code-editor">
        
    </div>
    <button v-on:click="Compile()">
        Run
    </button>
</template>

<style>
    
</style>
