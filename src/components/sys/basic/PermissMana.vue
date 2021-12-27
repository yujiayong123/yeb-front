<template>
    <div>
        <div class="PermissManaTool">
            <el-input placeholder="请输入角色英文名称" v-model="role.name" size="small">
                <template slot="prepend">ROLE_</template>
            </el-input>
            <el-input size="small" placeholder="请输入角色中文名称" v-model="role.nameZh" @keydown.native.enter="addRole"></el-input>
            <el-button icon="el-icon-circle-plus" type="primary" size="small" @click="addRole">添加角色</el-button>
        </div>
        <div class="PermissManaMain">
            <el-collapse accordion @change="change" v-model="activeName">
                <el-collapse-item :title="r.nameZh" :name="r.id" v-for="(r,index) in roles" :key='index'>
                    <el-card class="box-card">
                        <div slot="header" class="clearfix">
                            <span>可访问资源</span>
                            <el-button icon="el-icon-delete" style="float: right; padding: 3px 0; color: #ff0000"
                                       type="text" @click="doDelete(r)"></el-button>
                        </div>
                        <div>
                            <el-tree
                                    :data="allMenus"
                                    :props="defaultProps"
                                    show-checkbox
                                    ref="tree"
                                    :key="index"
                                    node-key="id"
                                    :default-checked-keys="selectMenus">
                            </el-tree>
                            <div style="justify-content: flex-end; display: flex">
                                <el-button size="mini" @click="cancelUpdate">取消修改</el-button>
                                <el-button size="mini" type="primary" @click="doUpdate(r.id,index)">确认修改</el-button>
                            </div>
                        </div>
                    </el-card>
                </el-collapse-item>
            </el-collapse>
        </div>
    </div>
</template>

<script>
    export default {
        name: "PermissMana",
        data() {
            return {
                role: {
                    name: '',
                    nameZh: ''
                },
                roles: [],
                allMenus: [],
                defaultProps: {
                    children: 'children',
                    label: 'name'
                },
                selectMenus: [],
                activeName: -1
            }
        },
        mounted() {
            this.initRoles();
        },
        methods: {
            // 初始化角色
            initRoles() {
                this.getRequest("/system/basic/permission/").then(res => {
                    if (res) {
                        this.roles = res;
                    }
                })
            },
            change(rid) {
                if (rid) {
                    this.initAllMenus();
                    this.initSelectedMenus(rid);
                }
            },
            // 初始化菜单
            initAllMenus() {
                this.getRequest('/system/basic/permission/getAllMenus').then(res => {
                    if (res) {
                        this.allMenus = res;
                    }
                })
            },
            // 初始化角色菜单
            initSelectedMenus(rid) {
                this.getRequest('/system/basic/permission/mid/' + rid).then(res => {
                    if (res) {
                        this.selectMenus = res;
                    }
                })
            },
            // 更新角色菜单
            doUpdate(rid, index) {
                let tree = this.$refs.tree[index];
                let selectedTree = tree.getCheckedKeys(true);
                // console.log(selectedTree)
                let url = '/system/basic/permission/updateMenuRole?rid=' + rid;
                selectedTree.forEach(key => {
                    url += '&mids' + key;
                })
                this.getRequest(url).then(res => {
                    this.activeName = -1;
                })
            },
            // 取消更新角色菜单
            cancelUpdate() {
                this.activeName = -1;
            },
            // 添加角色
            addRole() {
                if (this.role.name && this.role.nameZh) {
                    this.postRequest('/system/basic/permission/',this.role).then(res => {
                        if (res) {
                            this.initRoles();
                            this.role.name = '';
                            this.role.nameZh = '';
                        }
                    })
                }else {
                    this.$message.error('字段不能为空');
                }
            },
            // 删除角色
            doDelete(role) {
                this.$confirm('是否永久删除[' + role.nameZh + ']角色, 是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    this.deleteRequest('/system/basic/permission/role/' + role.id).then(res => {
                        if (res) {
                            this.initRoles();
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

<style>
    .PermissManaTool {
        display: flex;
        justify-content: flex-start;
    }

    .PermissManaTool .el-input {
        width: 300px;
        margin-left: 6px;
    }

    .PermissManaMain {
        margin-top: 10px;
        width: 700px;
    }
</style>
