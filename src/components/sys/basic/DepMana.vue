<template>
    <div style="width: 500px">
        <el-input
                placeholder="请输入部门名称进行搜索"
                prefix-icon="el-icon-search"
                v-model="filterText">
        </el-input>
        <el-tree
                :data="deps"
                :props="defaultProps"
                :filter-node-method="filterNode"
                :expand-on-click-node="false"
                ref="tree">
            <span class="custom-tree-node" slot-scope="{ node, data }"
                  style="display: flex;justify-content: space-between;width: 100%">
                <span>{{ node.name }}</span>
                <span>
                  <el-button
                          type="primary"
                          size="mini"
                          class="depBtn"
                          @click="() => showAddDep(data)">
                    添加部门
                  </el-button>
                  <el-button
                          type="danger"
                          size="mini"
                          class="depBtn"
                          @click="() => deleteDep(data)">
                    删除部门
                  </el-button>
                </span>
            </span>
        </el-tree>
        <el-dialog
                title="添加部门"
                :visible.sync="dialogVisible"
                width="30%">
            <div>
                <table>
                    <tr>
                        <td>
                            <el-tag>上级部门</el-tag>
                        </td>
                        <td>
                            {{pname}}
                        </td>
                    </tr>
                    <tr>
                        <td>
                            <el-tag>部门名称</el-tag>
                        </td>
                        <td>
                            <el-input v-model="dep.name" placeholder="请输入部门名称"></el-input>
                        </td>
                    </tr>
                </table>
            </div>
            <span slot="footer" class="dialog-footer">
            <el-button @click="dialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="doAdd">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        name: "DepMana",
        data() {
            return {
                filterText: '',
                deps: [],
                defaultProps: {
                    children: 'children',
                    label: 'name'
                },
                dialogVisible: false,
                dep: {
                    name: '',
                    parentId: -1
                },
                pname: ''
            }
        },
        watch: {
            filterText(val) {
                this.$refs.tree.filter(val);
            }
        },
        mounted() {
            this.initDeps();
        },
        methods: {
            filterNode(value, data) {
                if (!value) return true;
                return data.name.indexOf(value) !== -1;
            },
            // 初始化所有部门
            initDeps() {
                this.getRequest('/system/basic/department/getAllDepartments').then(res => {
                    if (res) {
                        this.deps = res;
                    }
                })
            },
            // 点击添加部门出现弹窗
            showAddDep(data) {
                // console.log(data)
                this.dialogVisible = true;
                this.pname = data.name;
                this.dep.parentId = data.id;
            },
            initDep() {
                this.dep = {
                    name: '',
                    parentId: -1
                }
                this.pname = ''
            },
            addDep2Deps(deps, dep) {
                for (let i = 0; i < deps.length; i++) {
                    let d = deps[i];
                    if (d.id == dep.parentId) {
                        d.children = d.children.concat(dep);
                        if (d.children.length > 0) {
                            d.isParent = true
                        }
                        return;
                    } else {
                        this.addDep2Deps(d.children, dep);
                    }
                }
            },
            // 添加部门
            doAdd() {
                this.postRequest('/system/basic/department/addDep', this.dep).then(res => {
                    if (res) {
                        this.addDep2Deps(this.deps, res.object)
                        this.initDep();
                        this.dialogVisible = false;
                    }
                })
            },
            removeDepfromDeps(p, deps, id) {
                for (let i = 0; i < deps.length; i++) {
                    let d = deps[i];
                    if (d.id == id) {
                        deps.splice(i, 1);
                        p.parent = false;
                        return;
                    } else {
                        this.removeDepfromDeps(d, d.children, id);
                    }
                }
            },
            // 删除部门
            deleteDep(data) {
                if (data.parent) {
                    this.$message.error('父部门无法删除');
                } else {
                    this.$confirm('此操作将永久删除[' + data.name + ']部门, 是否继续?', '提示', {
                        confirmButtonText: '确定',
                        cancelButtonText: '取消',
                        type: 'warning'
                    }).then(() => {
                        this.deleteRequest('/system/basic/department/deleteDep/' + data.id).then(res => {
                            if (res) {
                                this.removeDepfromDeps(null, this.deps, data.id);
                            }
                        })
                    }).catch(() => {
                        this.$message({
                            type: 'info',
                            message: '已取消删除'
                        });
                    });
                }
            }
        }
    }
</script>

<style>
    .depBtn {
        padding: 2px;
    }
</style>
