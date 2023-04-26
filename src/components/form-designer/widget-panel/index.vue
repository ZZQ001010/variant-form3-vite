<template>
    <el-scrollbar class="side-scroll-bar" :style="{height: scrollerHeight}">
        <div class="panel-container">

            <el-tabs v-model="firstTab" class="no-bottom-margin indent-left-margin">
                <el-tab-pane name="componentLib" >
                    <template #label>
                        <span @click="closeEditor"><svg-icon icon-class="el-set-up"/> {{i18nt('designer.componentLib')}}</span>
                    </template>

                    <el-collapse v-model="activeNames" class="widget-collapse">
                        <el-collapse-item name="1" :title="i18nt('designer.containerTitle')">
                            <draggable tag="ul" :list="containers" item-key="key"
                                       :group="{name: 'dragGroup', pull: 'clone', put: false}"
                                       :clone="handleContainerWidgetClone" ghost-class="ghost" :sort="false"
                                       :move="checkContainerMove" @end="onContainerDragEnd">
                                <template #item="{ element: ctn }">
                                    <li class="container-widget-item" :title="ctn.displayName"
                                        @dblclick="addContainerByDbClick(ctn)">
                                        <span><svg-icon :icon-class="ctn.icon" class-name="color-svg-icon"/>{{i18n2t(`designer.widgetLabel.${ctn.type}`, `extension.widgetLabel.${ctn.type}`)}}</span>
                                    </li>
                                </template>
                            </draggable>
                        </el-collapse-item>

                        <el-collapse-item name="2" :title="i18nt('designer.basicFieldTitle')">
                            <draggable tag="ul" :list="basicFields" item-key="key"
                                       :group="{name: 'dragGroup', pull: 'clone', put: false}"
                                       :move="checkFieldMove"
                                       :clone="handleFieldWidgetClone" ghost-class="ghost" :sort="false">
                                <template #item="{ element: fld }">
                                    <li class="field-widget-item" :title="fld.displayName"
                                        @dblclick="addFieldByDbClick(fld)">
                                        <span><svg-icon :icon-class="fld.icon" class-name="color-svg-icon"/>{{i18n2t(`designer.widgetLabel.${fld.type}`, `extension.widgetLabel.${fld.type}`)}}</span>
                                    </li>
                                </template>
                            </draggable>
                        </el-collapse-item>

                        <el-collapse-item name="3" :title="i18nt('designer.advancedFieldTitle')">
                            <draggable tag="ul" :list="advancedFields" item-key="key"
                                       :group="{name: 'dragGroup', pull: 'clone', put: false}"
                                       :move="checkFieldMove"
                                       :clone="handleFieldWidgetClone" ghost-class="ghost" :sort="false">
                                <template #item="{ element: fld }">
                                    <li class="field-widget-item" :title="fld.displayName"
                                        @dblclick="addFieldByDbClick(fld)">
                                        <span><svg-icon :icon-class="fld.icon" class-name="color-svg-icon"/>{{i18n2t(`designer.widgetLabel.${fld.type}`, `extension.widgetLabel.${fld.type}`)}}</span>
                                    </li>
                                </template>
                            </draggable>
                        </el-collapse-item>

                        <el-collapse-item name="4" :title="i18nt('designer.customFieldTitle')">
                            <draggable tag="ul" :list="customFields" item-key="key"
                                       :group="{name: 'dragGroup', pull: 'clone', put: false}"
                                       :move="checkFieldMove"
                                       :clone="handleFieldWidgetClone" ghost-class="ghost" :sort="false">
                                <template #item="{ element: fld }">
                                    <li class="field-widget-item" :title="fld.displayName"
                                        @dblclick="addFieldByDbClick(fld)">
                <span>
                  <svg-icon :icon-class="fld.icon" class-name="color-svg-icon"/>{{i18n2t(`designer.widgetLabel.${fld.type}`, `extension.widgetLabel.${fld.type}`)}}</span>
                                    </li>
                                </template>
                            </draggable>
                        </el-collapse-item>

                    </el-collapse>

                </el-tab-pane>

                <!-- 代码视图-->
                <el-tab-pane>
                    <template #label>
                        <span @click="openEditor"> <svg-icon icon-class="code"/> 代码</span>
                    </template>
                    <el-tree style="height: 1000px"
                            :data="files"
                            node-key="id"
                            @node-contextmenu="rightClick"
                            @node-drag-start="handleDragStart"
                            @node-drag-enter="handleDragEnter"
                            @node-drag-leave="handleDragLeave"
                            @node-drag-over="handleDragOver"
                            @node-drag-end="handleDragEnd"
                            @node-drop="handleDrop"
                            @node-click="handleNodeClick"
                            draggable
                            :allow-drop="allowDrop"
                            :allow-drag="allowDrag">
                    </el-tree>


                        <el-card class="box-card" ref="card" v-show="menuVisible">
                            <div @click="createFile">add</div>
                            <el-divider class="small-divider"></el-divider>
                            <div @click="delFile">delete</div>
                            <el-divider class="small-divider"></el-divider>
                            <div @click="renameFile">rename</div>
                        </el-card>

                        <el-dialog v-model="dialogVisible" append-to-body>
                            <el-form-item label="文件名">
                                <el-input v-model="filename"></el-input>
                            </el-form-item>
                          <span slot="footer" class="dialog-footer">
                            <el-button @click="dialogClose">取 消</el-button>
                            <el-button type="primary" @click="doSubmit">确 定</el-button>
                          </span>
                        </el-dialog>

                </el-tab-pane>

                <!-- 模板 -->
                <el-tab-pane  v-if="showFormTemplates()" name="formLib" style="padding: 8px">
                    <template #label>
                        <span @click="closeEditor"><svg-icon icon-class="el-form-template"/> {{i18nt('designer.formLib')}}</span>
                    </template>

                    <template v-for="(ft, idx) in formTemplates">
                        <el-card :bord-style="{ padding: '0' }" shadow="hover" class="ft-card">
                            <el-popover placement="right" trigger="hover">
                                <template #reference>
                                    <img :src="ft.imgUrl" style="width: 200px">
                                </template>
                                <img :src="ft.imgUrl" style="height: 600px;width: 720px">
                            </el-popover>
                            <div class="bottom clear-fix">
                                <span class="ft-title">#{{idx+1}} {{ft.title}}</span>
                                <el-button link type="primary" class="right-button"
                                           @click="loadFormTemplate(ft.jsonUrl)">
                                    {{i18nt('designer.hint.loadFormTemplate')}}
                                </el-button>
                            </div>
                        </el-card>
                    </template>
                </el-tab-pane>

            </el-tabs>

        </div>
    </el-scrollbar>
</template>

<script>
    import {containers as CONS, basicFields as BFS, advancedFields as AFS, customFields as CFS} from "./widgetsConfig"
    import {formTemplates} from './templatesConfig'
    import {addWindowResizeHandler, generateId} from "@/utils/util"
    import i18n from "@/utils/i18n"
    import axios from 'axios'
    import SvgIcon from "@/components/svg-icon/index";


    // import ftImg1 from '@/assets/ft-images/t1.png'
    // import ftImg2 from '@/assets/ft-images/t2.png'
    // import ftImg3 from '@/assets/ft-images/t3.png'
    // import ftImg4 from '@/assets/ft-images/t4.png'
    // import ftImg5 from '@/assets/ft-images/t5.png'
    // import ftImg6 from '@/assets/ft-images/t6.png'
    // import ftImg7 from '@/assets/ft-images/t7.png'
    // import ftImg8 from '@/assets/ft-images/t8.png'

    export default {
        name: "FieldPanel",
        mixins: [i18n],
        components: {
            SvgIcon
        },
        props: {
            designer: Object,
        },
        inject: ['getBannedWidgets', 'getDesignerConfig'],
        data() {
            return {
                designerConfig: this.getDesignerConfig(),

                firstTab: 'componentLib',

                scrollerHeight: 0,

                activeNames: ['1', '2', '3', '4'],

                containers: [],
                basicFields: [],
                advancedFields: [],
                customFields: [],

                formTemplates: formTemplates,
                files: [],
                menuVisible: false,
                currentNode: null,
                currentNodeData: null,
                filesProps: {
                    children: 'children',
                    label: 'label'
                },
                dialogVisible: false,
                filename: "",
                dialogOpt: "add" // add rename copy
                // ftImages: [
                //   {imgUrl: ftImg1},
                //   {imgUrl: ftImg2},
                //   {imgUrl: ftImg3},
                //   {imgUrl: ftImg4},
                //   {imgUrl: ftImg5},
                //   {imgUrl: ftImg6},
                //   {imgUrl: ftImg7},
                //   {imgUrl: ftImg8},
                // ]
            }
        },
        computed: {
            //
        },
        created() {
            this.loadWidgets()
        },
        mounted() {
            //this.loadWidgets()

            this.scrollerHeight = window.innerHeight - 56 + 'px'
            addWindowResizeHandler(() => {
                this.$nextTick(() => {
                    this.scrollerHeight = window.innerHeight - 56 + 'px'
                    //console.log(this.scrollerHeight)
                })
            })
            this.loadFiles();
        },
        methods: {
            doSubmit() {
              const  opt = this.dialogOpt;
              if (opt == "add")  {
                  this.doCreateFile();
              } else if (opt == "rename") {
                  this.doRename();
              } else if (opt == "copy") {
                  this.doCopy();
              } else {
                  this.$message.error("不支持的操作")
              }
            },
            dialogClose() {
                this.dialogVisible = false;
                this.filename  = "";
            },

            rightClick(event, data, node, element) {
                console.info(event, data, node, element)
                this.currentNode = node
                this.currentNodeData = data;
                this.menuVisible = true
                document.addEventListener('click', this.foo)
                this.$refs.card.$el.style.left = event.clientX + 30 + 'px'
                this.$refs.card.$el.style.top = event.clientY - 100 + 'px'
            },
            foo() {
                this.menuVisible = false
                //  要及时关掉监听，不关掉的是一个坑，不信你试试，虽然前台显示的时候没有啥毛病，加一个alert你就知道了
                document.removeEventListener('click', this.foo)
            },
            delFile() {
                this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                   this.doDelFile();
                }).catch(() => {
                    this.$message({
                        type: 'info',
                        message: '已取消删除'
                    });
                });
            },
            doDelFile() {
                axios.delete('http://localhost:8866/files', {
                    params: {
                        path: this.currentNodeData.path
                    }
                })
                    .then(response => {
                        this.$message({
                            message: '创建成功',
                            type: 'success'
                        });
                        this.loadFiles();
                    })
                    .catch(error => {
                        console.log(error);
                    });
            },
            renameFile() {
                this.dialogOpt = "rename";
                this.dialogVisible = true;
                this.filename = this.currentNodeData.label;

            },
            doRename() {
                axios.put('http://localhost:8866/files', {
                    oldName: this.currentNodeData.path,
                    newName: this.filename
                })
                    .then(response => {
                        this.$message({
                            message: '重命名成功',
                            type: 'success'
                        });
                       this.loadFiles();
                    })
                    .catch(error => {
                        this.$message.error("重命名失败");
                    });
                this.dialogVisible = false;
                this.filename = "";
            },
            createFile() {
                // 开启一个弹窗填写file名字
                if (this.currentNodeData.isLeaf) {
                    this.$message.error('节点不支持的操作')
                    return
                }
                this.dialogVisible = true
                console.info("createFile click", this.dialogVisible);
            },
            doCreateFile() {
                const path = this.currentNodeData.path + "/" + this.filename;
                console.info("cerate file", this.currentNodeData.path + "/" + this.filename)
                if (this.currentNodeData.isLeaf) {
                    this.$message.error('节点不支持的操作')
                }  else {
                    axios
                        .post(`http://localhost:8866/files`, {
                            path: path,
                            isDir: false,
                        })
                        .then((response) => {
                            this.$message({
                                message: '创建成功',
                                type: 'success'
                            });
                            this.loadFiles()
                        })
                        .catch((error) => {
                            console.error(error);
                        });
                }
                this.filename = '';
                this.dialogVisible = false;
            },
            openEditor() {
                this.$emit('openEditorEvent')
            },
            closeEditor() {
                this.$emit('closeEditorEvent')
            },
            handleNodeClick(data) {
                this.$emit('updatePathEvent', data.path);
                if (data.isLeaf) { // 如果点击的节点是叶子节点
                    axios
                        .get(`http://localhost:8866/file/content?path=${data.path}`)
                        .then((response) => {
                            console.log(response.data);
                            this.$emit('setCodeEvent', response.data)
                        })
                        .catch((error) => {
                            console.error(error);
                        });
                }
            },
            loadFiles() {
                axios
                    .get("http://localhost:8866/files") // 相对路径
                    .then((response) => {
                        this.files = response.data;
                    })
                    .catch((error) => {
                        console.error(error);
                    });
            },
            isBanned(wName) {
                return this.getBannedWidgets().indexOf(wName) > -1
            },

            showFormTemplates() {
                if (this.designerConfig['formTemplates'] === undefined) {
                    return true
                }

                return !!this.designerConfig['formTemplates']
            },

            loadWidgets() {
                this.containers = CONS.map(con => {
                    return {
                        key: generateId(),
                        ...con,
                        displayName: this.i18n2t(`designer.widgetLabel.${con.type}`, `extension.widgetLabel.${con.type}`)
                    }
                }).filter(con => {
                    return !con.internal && !this.isBanned(con.type)
                })

                this.basicFields = BFS.map(fld => {
                    return {
                        key: generateId(),
                        ...fld,
                        displayName: this.i18n2t(`designer.widgetLabel.${fld.type}`, `extension.widgetLabel.${fld.type}`)
                    }
                }).filter(fld => {
                    return !this.isBanned(fld.type)
                })

                this.advancedFields = AFS.map(fld => {
                    return {
                        key: generateId(),
                        ...fld,
                        displayName: this.i18n2t(`designer.widgetLabel.${fld.type}`, `extension.widgetLabel.${fld.type}`)
                    }
                }).filter(fld => {
                    return !this.isBanned(fld.type)
                })

                this.customFields = CFS.map(fld => {
                    return {
                        key: generateId(),
                        ...fld,
                        displayName: this.i18n2t(`designer.widgetLabel.${fld.type}`, `extension.widgetLabel.${fld.type}`)
                    }
                }).filter(fld => {
                    return !this.isBanned(fld.type)
                })
            },

            handleContainerWidgetClone(origin) {
                return this.designer.copyNewContainerWidget(origin)
            },

            handleFieldWidgetClone(origin) {
                return this.designer.copyNewFieldWidget(origin)
            },

            checkContainerMove(evt) {
                return this.designer.checkWidgetMove(evt)
            },

            checkFieldMove(evt) {
                return this.designer.checkFieldMove(evt)
            },

            onContainerDragEnd(evt) {
                //console.log('Drag end of container: ')
                //console.log(evt)
            },

            addContainerByDbClick(container) {
                this.designer.addContainerByDbClick(container)
            },

            addFieldByDbClick(widget) {
                this.designer.addFieldByDbClick(widget)
            },

            loadFormTemplate(jsonUrl) {
                this.$confirm(this.i18nt('designer.hint.loadFormTemplateHint'), this.i18nt('render.hint.prompt'), {
                    confirmButtonText: this.i18nt('render.hint.confirm'),
                    cancelButtonText: this.i18nt('render.hint.cancel')
                }).then(() => {
                    axios.get(jsonUrl).then(res => {
                        let modifiedFlag = false
                        if (typeof res.data === 'string') {
                            modifiedFlag = this.designer.loadFormJson(JSON.parse(res.data))
                        } else if (res.data.constructor === Object) {
                            modifiedFlag = this.designer.loadFormJson(res.data)
                        }
                        if (modifiedFlag) {
                            this.designer.emitHistoryChange()
                        }

                        this.$message.success(this.i18nt('designer.hint.loadFormTemplateSuccess'))
                    }).catch(error => {
                        this.$message.error(this.i18nt('designer.hint.loadFormTemplateFailed') + ':' + error)
                    })
                }).catch(error => {
                    console.error(error)
                })
            }

        }

    }
</script>

<style lang="scss" scoped>
    .color-svg-icon {
        -webkit-font-smoothing: antialiased;
        color: #7c7d82;
    }

    .side-scroll-bar {
        :deep(.el-scrollbar__wrap) {
            overflow-x: hidden;
        }
    }

    div.panel-container {
        padding-bottom: 10px;
    }

    .no-bottom-margin :deep(.el-tabs__header) {
        margin-bottom: 0;
    }

    .indent-left-margin {
        :deep(.el-tabs__nav) {
            margin-left: 20px;
        }
    }

    .el-collapse-item :deep(ul) > li {
        list-style: none;
    }

    .widget-collapse {
        border-top-width: 0;

        :deep(.el-collapse-item__header) {
            margin-left: 8px;
            font-style: italic;
            font-weight: bold;
        }

        :deep(.el-collapse-item__content) {
            padding-bottom: 6px;

            ul {
                padding-left: 10px; /* 重置IE11默认样式 */
                margin: 0; /* 重置IE11默认样式 */
                margin-block-start: 0;
                margin-block-end: 0.25em;
                padding-inline-start: 10px;

                &:after {
                    content: "";
                    display: block;
                    clear: both;
                }

                .container-widget-item, .field-widget-item {
                    display: inline-block;
                    height: 32px;
                    line-height: 32px;
                    width: 98px;
                    float: left;
                    margin: 2px 6px 6px 0;
                    cursor: move;
                    white-space: nowrap;
                    text-overflow: ellipsis;
                    overflow: hidden;
                    background: #fff;
                    border: 1px solid #e8e9eb;
                    border-radius: 4px;
                    padding: 0 8px;
                }

                .container-widget-item:hover, .field-widget-item:hover {
                    background: #F1F2F3;
                    border-color: $--color-primary;

                    .color-svg-icon {
                        color: $--color-primary;
                    }
                }

                .drag-handler {
                    position: absolute;
                    top: 0;
                    left: 160px;
                    background-color: #dddddd;
                    border-radius: 5px;
                    padding-right: 5px;
                    font-size: 11px;
                    color: #666666;
                }
            }
        }
    }

    .el-card.ft-card {
        border: 1px solid #8896B3;
    }

    .ft-card {
        margin-bottom: 10px;

        .bottom {
            margin-top: 10px;
            line-height: 12px;
        }

        /*
        .image-zoom {
          height: 500px;
          width: 620px
        }
        */

        .ft-title {
            font-size: 13px;
            font-weight: bold;
        }

        .right-button {
            padding: 0;
            float: right;
        }

        .clear-fix:before, .clear-fix:after {
            display: table;
            content: "";
        }

        .clear-fix:after {
            clear: both;
        }

    }

    .box-card {
        font-size: 12px;
        position: absolute;
        z-index: 1000;
    }
    .small-divider {
        margin: 5px 0;
    }
    .box-card>div:hover {
        background-color: #f01300;
    }

</style>
