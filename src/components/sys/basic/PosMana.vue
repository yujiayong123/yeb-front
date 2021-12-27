<template>
    <div>
        <div>
            <el-input
                    size="small"
                    class="addPosInput"
                    placeholder="添加职位"
                    suffix-icon="el-icon-circle-plus"
                    @keydown.enter.native="addPosition"
                    v-model="pos.name">
            </el-input>
            <el-button icon="el-icon-circle-plus" type="primary" size="small" @click="addPosition">添加</el-button>
            <el-button type="danger" size="small" :disabled="this.multipleSelection.length==0" @click="deleteIds">批量删除
            </el-button>
        </div>
        <div class="posManaMain">
            <el-table
                    stripe
                    border
                    :data="positions"
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
                        label="职位"
                        width="120">
                </el-table-column>
                <el-table-column
                        prop="createDate"
                        label="创建时间"
                        width="200">
                </el-table-column>
                <el-table-column label="操作">
                    <template slot-scope="scope">
                        <el-button
                                size="mini"
                                @click="showEditView(scope.$index, scope.row)">编辑
                        </el-button>
                        <el-button
                                size="mini"
                                type="danger"
                                @click="handleDelete(scope.$index, scope.row)">删除
                        </el-button>
                    </template>
                </el-table-column>
            </el-table>
        </div>
        <el-dialog
                title="编辑职位"
                :visible.sync="dialogVisible"
                width="30%">
            <div>
                <el-tag>职位名称</el-tag>
                <el-input v-model="updatePos.name" class="updatePosInput"></el-input>
            </div>
            <span slot="footer" class="dialog-footer">
                <el-button size="small" @click="dialogVisible = false">取 消</el-button>
                <el-button size="small" type="primary" @click="doUpdate">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        name: "PosMana",
        data() {
            return {
                pos: {
                    name: ''
                },
                positions: [],
                dialogVisible: false,
                updatePos: {
                    name: ''
                },
                multipleSelection: []
            }
        },
        mounted() {
            this.initPositions();
        },
        methods: {
            // 批量删除
            deleteIds() {
                this.$confirm('是否永久删除[' + this.multipleSelection.length + ']条职位, 是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    let ids = '?';
                    this.multipleSelection.forEach(item => {
                        ids += 'ids=' + item.id + '&';
                    })
                    this.deleteRequest('/system/basic/pos/' + ids).then(res => {
                        if (res) {
                            this.initPositions();
                        }
                    })
                }).catch(() => {
                    this.$message({
                        type: 'info',
                        message: '已取消删除'
                    });
                });
            },
            // 通过勾选来获取勾选对象（数组）
            handleSelectionChange(val) {
                this.multipleSelection = val;
            },
            // 添加职位
            addPosition() {
                if (this.pos.name) {
                    this.postRequest('/system/basic/pos/', this.pos).then(res => {
                        if (res) {
                            this.initPositions();
                            this.pos.name = ''
                        }
                    })
                } else {
                    this.$message.error('职位不能为空');
                }
            },
            // 编辑数据回显
            showEditView(index, data) {
                Object.assign(this.updatePos, data)
                this.updatePos.createDate = ''
                this.dialogVisible = true
            },
            // 编辑
            doUpdate() {
                this.putRequest('/system/basic/pos/', this.updatePos).then(res => {
                    if (res) {
                        this.initPositions();
                        this.dialogVisible = false;
                    }
                })
            },
            // 删除
            handleDelete(index, data) {
                this.$confirm('是否永久删除[' + data.name + ']职位, 是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    this.deleteRequest('/system/basic/pos/' + data.id).then(res => {
                        if (res) {
                            this.initPositions();
                        }
                    })
                }).catch(() => {
                    this.$message({
                        type: 'info',
                        message: '已取消删除'
                    });
                });
            },
            // 数据展示
            initPositions() {
                this.getRequest('/system/basic/pos/').then(res => {
                    if (res) {
                        this.positions = res;
                    }
                })
            }
        }
    }
</script>

<style>
    .addPosInput {
        width: 300px;
        margin-right: 8px;
    }

    .posManaMain {
        margin-top: 10px;
    }

    .updatePosInput {
        width: 200px;
        margin-left: 8px;
    }
</style>
