<template>
    <div>
        <div style="display: flex; justify-content: center; margin-top: 10px">
            <el-input style="width: 400px; margin-right: 10px" icon="el-icon-search" placeholder="通过用户名搜索用户"></el-input>
            <el-button type="primary" icon="el-icon-search">搜索</el-button>
        </div>
        <div class="admin-container">
            <el-card class="admin-card" v-for="(admin,index) in admins" :key="index">
                <div slot="header" class="clearfix">
                    <span>{{admin.name}}</span>
                    <el-button style="float: right; padding: 3px 0; color: red" type="text" icon="el-icon-delete"></el-button>
                </div>
                <div>
                    <div class="img-container">
                        <img :src="admin.userFace" :alt="admin.name" :title="admin.name" class="userFace-img">
                    </div>
                </div>
                <div class="userInfo-container">
                    <div>用户名:{{admin.name}}</div>
                    <div>手机号码:{{admin.phone}}</div>
                    <div>电话号码:{{admin.telephone}}</div>
                    <div>地址:{{admin.address}}</div>
                    <div>用户状态:
                        <el-switch
                                v-model="admin.enabled"
                                active-color="#13ce66"
                                inactive-color="#ff4949"
                                active-text="启用"
                                inactive-text="禁用">
                        </el-switch>
                    </div>
                    <div>
                        用户角色:
                        <el-tag style="margin-right: 4px" type="success" v-for="(role,indexj) in admin.roles" :key="indexj">{{role.nameZh}}</el-tag>
                        <el-button type="text" icon="el-icon-more"></el-button>
                    </div>
                    <div>备注:{{admin.remark}}</div>
                </div>
            </el-card>
        </div>
    </div>
</template>

<script>
    export default {
        name: "SysAdmin",
        data() {
            return {
                admins: []
            }
        },
        mounted() {
            this.initAdmins();
        },
        methods:{
            initAdmins() {
                this.getRequest('/system/admin/getAllAdmin').then(res => {
                    if (res) {
                        this.admins = res;
                    }
                })
            }
        }
    }
</script>

<style>
    .admin-card{
        width: 350px;
        margin-bottom: 20px;
    }
    .admin-container{
        display: flex;
        justify-content: space-around;
        flex-flow: wrap;
        margin-top: 10px;
    }
    .userFace-img{
        width: 72px;
        height: 72px;
        border-radius: 72px;
    }
    .img-container{
        width: 100%;
        display: flex;
        justify-content: center;
    }
    .userInfo-container{
        font-size: 12px;
        color: #409eff;
    }
</style>
