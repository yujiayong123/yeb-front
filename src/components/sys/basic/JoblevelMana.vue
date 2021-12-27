<template>
    <div>
        <el-input size="small"
                  v-model="jl.name"
                  placeholder="请添加职称名称"
                  prefix-icon="el-icon-circle-plus"
                  style="width: 300px"
                  @keydown.enter.native="addJobLevel"></el-input>
        <el-select size="small" v-model="jl.titleLevel" placeholder="职称等级" style="margin-left: 6px">
            <el-option
                    v-for="item in titleLevel"
                    :key="item"
                    :label="item"
                    :value="item">
            </el-option>
        </el-select>
        <el-button type="primary"
                   size="small"
                   style="margin-left: 6px"
                   icon="el-icon-circle-plus"
                   @click="addJobLevel">添加
        </el-button>
        <el-button type="danger"
                   size="small"
                   :disabled="this.multipleSelection.length==0"
                   @click="deleteIds">批量删除
        </el-button>
            <div style="margin-top: 10px">
                <el-table
                        :data="jls"
                        stripe
                        border
                        size="small"
                        @selection-change="handleSelectionChange"
                        style="width: 70%">
                    <el-table-column
                            type="selection"
                            width="55">
                    </el-table-column>
                    <el-table-column
                            prop="id"
                            label="编号"
                            width="55">
                    </el-table-column>
                    <el-table-column
                            prop="name"
                            label="名称"
                            width="100">
                    </el-table-column>
                    <el-table-column
                            prop="titleLevel"
                            label="职称等级"
                            width="150">
                    </el-table-column>
                    <el-table-column
                            prop="createDate"
                            label="创建时间"
                            width="150">
                    </el-table-column>
                    <el-table-column
                            prop="enabled"
                            label="是否启用"
                            width="150">
                        <template slot-scope="scope">
                            <el-tag type="success" v-if="scope.row.enabled">已启用</el-tag>
                            <el-tag type="danger" v-else>未启用</el-tag>
                        </template>
                    </el-table-column>
                    <el-table-column label="操作">
                        <template slot-scope="scope">
                            <el-button size="small"
                                       @click="showEditView(scope.row)">编辑
                            </el-button>
                            <el-button size="small"
                                       type="danger"
                                       @click="handleDelete(scope.row)">删除
                            </el-button>
                        </template>
                    </el-table-column>
                </el-table>
            </div>
            <el-dialog
                    title="编辑职称"
                    :visible.sync="dialogVisible"
                    width="30%">
                <table>
                    <tr>
                        <td>职位名称:</td>
                        <td>
                            <el-input v-model="updateJl.name" size="small" style="margin-left: 6px"></el-input>
                        </td>
                    </tr>
                    <tr>
                        <td>职称等级:</td>
                        <td>
                            <el-select size="small" v-model="updateJl.titleLevel" style="margin-left: 6px">
                                <el-option
                                        v-for="item in titleLevel"
                                        :key="item"
                                        :label="item"
                                        :value="item">
                                </el-option>
                            </el-select>
                        </td>
                    </tr>
                    <tr>
                        <td>是否启用:</td>
                        <td>
                            <el-switch style="margin-left: 6px"
                                       v-model="updateJl.enabled"
                                       active-color="#13ce66"
                                       inactive-color="#ff4949"
                                       active-text="已启用"
                                       inactive-text="未启用">
                            </el-switch>
                        </td>
                    </tr>
                </table>
                <span slot="footer" class="dialog-footer">
                <el-button size="small" @click="dialogVisible = false">取 消</el-button>
                <el-button size="small" type="primary" @click="doUpdate">确 定</el-button>
            </span>
            </el-dialog>
    </div>
</template>

<script>
    export default {
        name: "JoblevelMana",
        data() {
            return {
                jl: {
                    name: '',
                    titleLevel: ''
                },
                titleLevel: [
                    '正高级',
                    '副高级',
                    '中级',
                    '初级',
                    '员级'
                ],
                jls: [],
                jobLevel: '',
                dialogVisible: false,
                updateJl: {
                    name: '',
                    titleLevel: '',
                    enabled: false
                },
                multipleSelection: []
            }
        },
        mounted() {
            this.initJobLevel();
        },
        methods: {
            // 页面展示
            initJobLevel() {
                this.getRequest('/system/basic/joblevel/').then(res => {
                    if (res) {
                        this.jls = res
                        this.jl.name = ''
                        this.jl.titleLevel = ''
                    }
                })
            },
            // 添加职位
            addJobLevel() {
                if (this.jl.titleLevel && this.jl.name) {
                    this.postRequest('/system/basic/joblevel/', this.jl).then(res => {
                        if (res) {
                            this.initJobLevel();
                        }
                    })
                } else {
                    this.$message.error('字段不能为空');
                }
            },
            // 删除职位
            handleDelete(data) {
                this.$confirm('是否永久删除[' + data.name + ']职称, 是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    this.deleteRequest('/system/basic/joblevel/' + data.id).then(res => {
                        if (res) {
                            this.initJobLevel();
                        }
                    })
                }).catch(() => {
                    this.$message({
                        type: 'info',
                        message: '已取消删除'
                    });
                });
            },
            // 编辑数据回显
            showEditView(data) {
                Object.assign(this.updateJl, data)
                this.updateJl.createDate = ''
                this.dialogVisible = true
            },
            // 编辑职称
            doUpdate() {
                this.putRequest('/system/basic/joblevel/', this.updateJl).then(res => {
                    if (res) {
                        this.initJobLevel();
                        this.dialogVisible = false
                    }
                })
            },
            // 通过勾选来获取勾选对象（数组）
            handleSelectionChange(val) {
                this.multipleSelection = val;
                // console.log(val)
            },
            // 批量删除
            deleteIds() {
                this.$confirm('是否永久删除[' + this.multipleSelection.length + ']条职称, 是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    let ids = '?';
                    this.multipleSelection.forEach(item => {
                        ids += 'ids=' + item.id + '&';
                    })
                    this.deleteRequest('/system/basic/joblevel/' + ids).then(res => {
                        if (res) {
                            this.initJobLevel();
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
</script>

<style scoped>

</style>
