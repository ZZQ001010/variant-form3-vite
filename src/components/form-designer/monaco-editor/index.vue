<template>
    <div :style="{height, width}" class="monaco-editor"></div>
</template>

<script>
    import * as monaco from 'monaco-editor'

    export default {
        props: {
            options: {
                type: Object,
                default: () => {
                }
            },
            height: {
                type: String,
                default: '300px'
            },
            width: {
                type: String,
                default: '100%'
            },
            code: {
                type: String,
                default: ''
            }
        },
        data() {
            return {
                defaultOptions: {
                    value: this.code, // 编辑器的值
                    language: 'javascript', //语言
                    theme: 'vs-dark', // 编辑器主题：vs, hc-black, or vs-dark
                    autoIndent: true, // 自动缩进
                    readOnly: false, // 是否只读
                }
            }
        },
        computed: {
            standaloneEditorConstructionOptions() {
                return Object.assign(this.defaultOptions, this.options)
            }
        },
        methods: {
            createMonacoEditor() {
                this.monacoEditor = monaco.editor.create(this.$el, this.standaloneEditorConstructionOptions)
                this.monacoEditor.onDidChangeModelContent(event => {
                    this.$emit('update:code', this.monacoEditor.getValue())
                })
            },
            reSize() {
                this.$nextTick(() => {
                    this.monacoEditor.layout()
                })
            },
            updateEditorContent(value) {
                this.monacoEditor.setValue(value);
            }
        },
        mounted() {
            this.createMonacoEditor()
        },
        watch: {
            height() {
                this.reSize()
            },
            width() {
                this.reSize()
            },
            options: {
                handler() {
                    this.$nextTick(() => {
                        this.monacoEditor.updateOptions(this.standaloneEditorConstructionOptions)
                    })
                },
                deep: true
            }
        }
    }
</script>

<style lang="less" scoped>
</style>
