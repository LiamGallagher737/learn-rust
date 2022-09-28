<script lang="ts">
    import {EditorView, basicSetup} from "codemirror";
    import {rust} from "@codemirror/lang-rust";

    export default {
        data() {
            return {
                
            }
        },
        mounted() {
            let parent = document.querySelector('#code-editor');
            if (parent === null) {
                throw new Error('Querying #code-editor returned null');
            }

            new EditorView ({
                extensions: [basicSetup, rust()],
                parent
            });
        },
        methods: {
            async Run() {
                let result = await this.Compile();
                let output = document.querySelector('#code-output');

                if (output === null) {
                    throw new Error('Querying #code-output returned null');
                }

                output.innerHTML = '';

                let stderr = document.createElement('p');
                let stdout = document.createElement('p');

                output.appendChild(stderr);
                output.appendChild(stdout);

                stderr.textContent = result.stderr;
                stdout.textContent = result.stdout;
            },
            async Compile(): Promise<CompileResult> {
                let parent = document.querySelector('#code-editor');
                if (parent === null) {
                    throw new Error('Querying #code-editor returned null');
                }

                let editor = EditorView.findFromDOM(parent as HTMLElement);
                if (editor === null) {
                    throw new Error('Failed finding editor within #code-editor');
                }

                let iter = editor.state.doc.iter();
                let line = iter.next();
                let code = '';
                while (!line.done) {
                    code += line.value;
                    line = iter.next();
                }

                if (!code.includes('fn main')) {
                    return {
                        success: false,
                        stderr: 'Failed to find main function',
                        stdout: '',
                    };
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

                let body = await response.json() as CompileResult;
                body.stderr = body.stderr.replace(/^ +/gm, ''); // Remove space at line start
                return body;
            }
        }
    }

    interface CompileResult {
        success: boolean,
        stderr: string,
        stdout: string,
    }

</script>

<template>
    <div id="code-editor">
        
    </div>
    <button v-on:click="Run()">
        Run
    </button>
</template>

<style>
    
</style>
