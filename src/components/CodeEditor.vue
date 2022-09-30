<script lang="ts">
    import {EditorView, basicSetup} from "codemirror";
    import {rust} from "@codemirror/lang-rust";
    import {oneDark} from '@codemirror/theme-one-dark';

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
                extensions: [basicSetup, rust(), oneDark],
                parent
            });
        },
        methods: {
            async Run() {
                let output = document.querySelector('#code-output');

                if (output === null) {
                    throw new Error('Querying #code-output returned null');
                }

                output.innerHTML = '';

                let parent = document.querySelector('#code-editor');
                if (parent === null) {
                    throw new Error('Querying #code-editor returned null');
                }

                let editor = EditorView.findFromDOM(parent as HTMLElement);
                if (editor === null) {
                    throw new Error('Failed finding editor within #code-editor');
                }

                let result = await this.Compile(editor);
                let correct = this.Validate(result.stdout);

                console.log(correct);

                let stderr = document.createElement('pre');
                let stdout = document.createElement('pre');

                output.appendChild(stderr);
                output.appendChild(stdout);

                stderr.innerHTML = result.stderr;
                stdout.innerHTML = result.stdout;

                stderr.style.whiteSpace = 'pre-wrap';
                stdout.style.whiteSpace = 'pre-wrap';
            },
            async Compile(editor: EditorView): Promise<CompileResult> {
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
                if (body.success) {
                    body.stderr = body.stderr.replace(/^ +/gm, ''); // Remove space at line start
                }
                return body;
            },
            Validate(output: string): boolean {
                let answersdoc = document.querySelectorAll('.answer');
                let answers: string[] = [];
                answersdoc.forEach(answerdoc => {
                    answers.push(answerdoc.innerHTML);
                });
                
                for (let i = 0; i < answers.length; i++) {
                    if (!output.includes(answers[i])) {
                        return false;
                    }
                }

                return true;
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
    <div id="code-editor-container">
        <div id="code-editor-header">
            <h2>Code</h2>
            <button id="code-run-btn" v-on:click="Run()">
                Run
            </button>
        </div>
        <div id="code-editor">
            
        </div>
    </div>
</template>

<style>
    #code-editor-container {
        grid-column: span 4;
    }
    #code-editor-header h2 {
        float: left;
    }
    #code-run-btn {
        padding: .5rem 2rem;
        border-radius: .5rem;
        background: rgb(32, 132, 178);
        outline: none;
        border: none;
        cursor: pointer;
        color: white;
        font-size: 1rem;
        font-weight: bolder;
        margin-top: 1.2rem;
        float: right;
    }
    #code-editor {
        padding-top: 5rem;
    }
</style>
