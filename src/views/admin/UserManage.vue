<template>
    <el-row style="background-color: #FFFFFF;padding: 5px 0;border-radius: 5px;">
        <el-row style="padding: 10px;margin-left: 10px;">
            <el-row>
                <el-date-picker size="small" style="width: 220px;" v-model="searchTime" type="daterange"
                    range-separator="至" start-placeholder="注册开始" end-placeholder="注册结束">
                </el-date-picker>
                <span class="top-bar">用户名</span>
                <el-input size="small" style="width: 120px;" v-model="userQueryDto.userName" placeholder="请输入" clearable
                    @clear="handleFilterClear">
                </el-input>
                <span class="top-bar">角色</span>
                <el-select size="small" style="width: 120px; user-select: none; margin-right: 20px;"
                    v-model="userQueryDto.userRole" placeholder="请选择" clearable
                    @clear="handleFilterClear">
                    <el-option :value="1" label="管理员"></el-option>
                    <el-option :value="2" label="用户"></el-option>
                </el-select>
                <span class="top-bar">封号</span>
                <el-select size="small" style="width: 120px; user-select: none; margin-right: 20px;"
                    v-model="userQueryDto.isLogin" placeholder="请选择" clearable
                    @clear="handleFilterClear">
                    <el-option :value="false" label="正常"></el-option>
                    <el-option :value="true" label="已封号"></el-option>
                </el-select>


                <span class="top-bar">禁言</span>
                <el-select size="small" style="width: 120px; user-select: none; margin-right: 20px;"
                    v-model="userQueryDto.isWord" placeholder="请选择" clearable
                    @clear="handleFilterClear">
                    <el-option :value="false" label="正常"></el-option>
                    <el-option :value="true" label="已禁言"></el-option>
                </el-select>
                <el-row style="margin-top: 20px;">
                    <el-button size="small" class="customer"
                        style="margin-left: 20px;background-color: rgb(43, 121, 203);border: none;" type="primary"
                        @click="handleFilter">立即查询</el-button>
                    <el-button size="small" class="customer reset"
                        style="background-color: #f1f1f1;border: none;color: #909399;border: 1px solid #f1f1f1;"
                        type="info" @click="resetQueryCondition">条件重置</el-button>
                    <el-button size="small" style="background-color: rgb(43, 121, 203);border: none;" class="customer"
                        type="info" @click="add()">新增用户</el-button>
                    <el-button size="small" class="customer"
                        :style="{ marginLeft: '10px', backgroundColor: selectedRows.length ? '#a7535a' : '#F1F1F1', border: 'none', color: selectedRows.length ? '#FFFFFF' : '#909399' }"
                        type="danger" @click="batchDelete()">批量删除</el-button>
                </el-row>

            </el-row>
        </el-row>
        <el-row style="margin: 0 20px;border-top: 1px solid rgb(245,245,245);">
            <el-table row-key="id" :data="tableData" style="width: 100%" @selection-change="handleSelectionChange">
                <!-- 多选框列 -->
                <el-table-column type="selection" width="55"></el-table-column>

                <!-- 标签名列 -->
                <!-- <el-table-column prop="name" label="标签名"></el-table-column> -->

                <!-- 操作列 -->
                <el-table-column prop="userAvatar" width="68" label="头像">
                    <template slot-scope="scope">
                        <el-avatar :size="25" :src="scope.row.userAvatar" style="margin-top: 10px;"></el-avatar>
                    </template>
                </el-table-column>
                <el-table-column prop="userName" label="名称"></el-table-column>
                <el-table-column prop="userAccount" width="128" label="账号"></el-table-column>
                <el-table-column prop="userEmail" width="168" label="邮箱"></el-table-column>
                <el-table-column prop="userRole" width="68" label="角色">
                    <template slot-scope="scope">
                        <span>{{ scope.row.userRole === 1 ? '管理员' : '用户' }}</span>
                    </template>
                </el-table-column>
                <el-table-column prop="isLogin" width="108" label="封号">
                    <template slot-scope="scope">
                        <i v-if="scope.row.isLogin" style="margin-right: 5px;" class="el-icon-warning"></i>
                        <i v-else style="margin-right: 5px;color: rgb(253, 199, 50);" class="el-icon-success"></i>
                        <el-tooltip v-if="scope.row.isLogin" class="item" effect="dark"
                            content="账号一经封号，不可登录系统。经由管理员解禁后，方可登录" placement="bottom-end">
                            <span style="text-decoration: underline;text-decoration-style: dashed;">已封号</span>
                        </el-tooltip>
                        <span v-else>正常</span>
                    </template>
                </el-table-column>
                <el-table-column prop="isWord" width="108" label="禁言">
                    <template slot-scope="scope">
                        <i v-if="scope.row.isWord" style="margin-right: 5px;" class="el-icon-warning"></i>
                        <i v-else style="margin-right: 5px;color: rgb(253, 199, 50);" class="el-icon-success"></i>
                        <el-tooltip v-if="scope.row.isWord" class="item" effect="dark"
                            content="账号一经禁言，不可评论互动。经由管理员解禁后，方可评论" placement="bottom-end">
                            <span style="text-decoration: underline;text-decoration-style: dashed;">已禁言</span>
                        </el-tooltip>
                        <span v-else>正常</span>
                    </template>
                </el-table-column>
                <el-table-column :sortable="true" prop="createTime" width="168" label="注册于"></el-table-column>
                <el-table-column label="操作" width="170">
                    <template slot-scope="scope">
                        <span class="text-button" @click="handleStatus(scope.row)" >账号状态</span>
                        <span class="text-button" @click="handleEdit(scope.row)">编辑</span>
                        <span class="text-button" @click="handleDelete(scope.row)">删除</span>
                    </template>
                </el-table-column>
            </el-table>


            <el-pagination style="margin:10px 0;" @size-change="handleSizeChange" @current-change="handleCurrentChange"
                :current-page="currentPage" :page-sizes="[7, 10]" :page-size="pageSize"
                layout="total, sizes, prev, pager, next, jumper" :total="totalItems"></el-pagination>
        </el-row>
        <!-- 操作面板 -->
        <el-dialog :show-close="false" :visible.sync="dialogUserOperaion" width="25%">
            <div slot="title">
                <p class="dialog-title">{{ !isOperation ? '新增用户' : '修改用户信息' }}</p>
            </div>

            <div style="padding:0 20px;">
                <el-row>
                    <el-upload class="avatar-uploader"
                        action="http://localhost:21090/api/personal-heath/v1.0/file/upload" :show-file-list="false"
                        :on-success="handleAvatarSuccess">
                        <img v-if="userAvatar" :src="userAvatar" class="dialog-avatar">
                        <i v-else class="el-icon-plus avatar-uploader-icon"></i>
                    </el-upload>
                </el-row>
                <el-row>
                    <span class="dialog-hover">账号</span>
                    <el-tooltip content="账号不可修改" placement="top" v-if="isOperation">
                        <input class="dialog-input" v-model="data.userAccount" placeholder="账号" readonly />
                    </el-tooltip>
                    <input class="dialog-input" v-model="data.userAccount" placeholder="账号" v-else />
                    <span class="dialog-hover">用户名</span>
                    <input class="dialog-input" v-model="data.userName" placeholder="用户名" />
                    <span class="dialog-hover">邮箱</span>
                    <input class="dialog-input" v-model="data.userEmail" placeholder="邮箱" />
                    <span class="dialog-hover">密码</span>
                    <input class="dialog-input" v-model="userPwd" type="password" placeholder="密码" />
                </el-row>
            </div>

            <span slot="footer" class="dialog-footer">
                <el-button size="small" v-if="!isOperation"
                    style="background-color: rgb(96, 98, 102);color: rgb(247,248,249);border: none;" class="customer"
                    type="info" @click="addOperation()">新增</el-button>
                <el-button size="small" v-else
                    style="background-color: rgb(96, 98, 102);color: rgb(247,248,249);border: none;" class="customer"
                    type="info" @click="updateOperation()">修改</el-button>
                <el-button class="customer" size="small" style="background-color: rgb(211, 241, 241);border: none;"
                    @click="cancel">取消</el-button>
            </span>
        </el-dialog>
        <el-dialog :show-close="false" :visible.sync="dialogStatusOperation" width="25%">
            <div slot="title">
                <p class="dialog-title">账号状态</p>
            </div>
            <div style="padding:0 20px;">
                <el-row>
                    <el-switch active-color="rgb(230, 62, 49)" inactive-color="rgb(246,246,246)" v-model="data.isLogin"
                        active-text="封号" inactive-text="正常状态">
                    </el-switch>
                </el-row>
                <el-row style="margin: 20px 0;">
                    <el-switch active-color="rgb(230, 62, 49)" inactive-color="rgb(246,246,246)" v-model="data.isWord"
                        active-text="禁言" inactive-text="正常状态">
                    </el-switch>
                </el-row>
                <span class="dialog-hover">设为管理员</span>
                <el-switch v-model="roleStatus" active-color="rgb(230, 62, 49)" inactive-color="rgb(246,246,246)">
                </el-switch>
            </div>
            <span slot="footer" class="dialog-footer">
                <el-button size="small" style="background-color: rgb(96, 98, 102);color: rgb(247,248,249);border: none;"
                    class="customer" type="info" @click="comfirmStatus">确认</el-button>
                <el-button class="customer" size="small" style="background-color: rgb(241, 241, 241);border: none;"
                    @click="cancel">取消</el-button>
            </span>
        </el-dialog>
    </el-row>
</template>

<script>
export default {
    data() {
        return {
            roleStatus: false,
            userPwd: '',
            userAvatar: '',
            data: {},
            filterText: '',
            currentPage: 1,
            pageSize: 7,
            totalItems: 0,
            dialogStatusOperation: false,
            dialogUserOperaion: false, // 开关
            isOperation: false, // 开关-标识新增或修改
            tableData: [],
            searchTime: [],
            selectedRows: [],
            status: null,
            userQueryDto: {}, // 搜索条件
            messsageContent: ''
        };
    },
    created() {
        this.fetchFreshData();
    },
    methods: {
        comfirmStatus() {
            const userUpdateDto = {
                id: this.data.id,
                isLogin: this.data.isLogin,
                isWord: this.data.isWord,
                userRole: this.roleStatus ? 1 : 2
            }
            this.$axios.put(`/user/backUpdate`, userUpdateDto).then(res => {
                if (res.data.code === 200) {
                    this.$notify({
                        duration: 2000,
                        title: '操作反馈',
                        message: '操作成功',
                        type: 'success'
                    });
                    this.dialogStatusOperation = false;
                    this.cancel();
                    this.fetchFreshData();
                }
            }).catch(error => {
                console.log("修改状态异常：" + error);
            })
        },
        handleStatus(data) {
            this.dialogStatusOperation = true;
            this.roleStatus = data.userRole === 1
            this.data = data;
        },
        handleAvatarSuccess(res, file) {
            if (res.code !== 200) {
                this.$notify({
                    duration: 2000,
                    title: '头像上传',
                    message: '异常',
                    type: 'error'
                });
                return;
            }
            this.$notify({
                duration: 2000,
                title: '头像上传',
                message: '成功',
                type: 'success'
            });
            this.userAvatar = res.data;
        },
        switchChange() {
            this.fetchFreshData();
        },
        async handleSwitchChange(id, status, operation) {
            try {
                let param = { id: id }
                // 登录状态
                if (operation) {
                    param.isLogin = status;
                } else { // 评论状态
                    param.isWord = status;
                }
                const response = await this.$axios.put(`/user/backUpdate`, param);
                if (response.data.code === 200) {
                    this.$notify({
                        duration: 2000,
                        title: '操作提示',
                        message: '成功',
                        type: 'success'
                    });
                    this.cancel();
                }
            } catch (e) {
                console.error(`更新用户状态异常：${e}`);
            }
        },
        // 多选框选中
        handleSelectionChange(selection) {
            this.selectedRows = selection;
        },
        // 批量删除数据
        async batchDelete() {
            if (!this.selectedRows.length) {
                this.$message(`未选中任何数据`);
                return;
            }
            const confirmed = await this.$swalConfirm({
                title: '删除用户数据',
                text: `删除后不可恢复，是否继续？`,
                icon: 'warning',
            });
            if (confirmed) {
                try {
                    let ids = this.selectedRows.map(entity => entity.id);
                    const response = await this.$axios.post(`/user/batchDelete`, ids);
                    if (response.data.code === 200) {
                        this.$notify({
                            duration: 2000,
                            title: '删除操作',
                            message: '成功',
                            type: 'success'
                        });
                        this.cancel();
                        this.fetchFreshData();
                        return;
                    }
                } catch (e) {
                    console.error(`用户信息删除异常：`, e);
                }
            }
            this.selectedRows = [];
        },
        resetQueryCondition() {
            this.userQueryDto = {};
            this.searchTime = [];
            this.fetchFreshData();
        },
        // 修改信息
        async updateOperation() {
            if(!this.data.userName){
                this.$message.error('用户名不能为空！')
                return
            }
            if (this.userPwd !== '') {
                const pwd = this.$md5(this.$md5(this.userPwd));
                this.data.userPwd = pwd;
            } else {
                this.data.userPwd = null;
            }
            this.data.userAvatar = this.userAvatar;
            try {
                const response = await this.$axios.put('/user/backUpdate', this.data);
                this.$message[response.data.code === 200 ? 'success' : 'error'](response.data.msg);

                if (response.data.code === 200) {
                    this.cancel();
                    this.fetchFreshData();
                }
            } catch (error) {
                console.error('提交表单时出错:', error);
                this.$message.error('提交失败，请稍后再试！');
            }
        },
        // 信息新增
        async addOperation() {
            if (!this.data.userAccount) {
                this.$message.error('账号不能为空！')
                return
            } else if (!this.data.userName) {
                this.$message.error('用户名不能为空！')
                return
            } else if (!this.userPwd) {
                this.$message.error('密码不能为空！')
                return
            }
            if (this.userPwd !== '') {
                this.data.userPwd = this.$md5(this.$md5(this.userPwd));
            } else {
                this.data.userPwd = null;
            }
            this.data.userAvatar = this.userAvatar;
            try {
                const response = await this.$axios.post('/user/insert', this.data);
                this.$message[response.data.code === 200 ? 'success' : 'error'](response.data.msg);
                if (response.data.code === 200) {
                    this.fetchFreshData();
                    this.cancel();
                    this.$notify({
                        duration: 2000,
                        title: '新增操作',
                        message: '成功',
                        type: 'success'
                    });
                }
            } catch (error) {
                console.error('提交表单时出错:', error);
                this.$message.error('提交失败，请稍后再试！');
            }
        },
        cancel() {
            this.userAvatar = '';
            this.userPwd = '';
            this.data = {};
            this.isOperation = false;
            this.dialogStatusOperation = false;
            this.dialogUserOperaion = false;
            this.fetchFreshData();

        },
        async fetchFreshData() {
            try {
                this.tableData = [];
                let startTime = null;
                let endTime = null;
                if (this.searchTime != null && this.searchTime.length === 2) {
                    const [startDate, endDate] = await Promise.all(this.searchTime.map(date => date.toISOString()));
                    startTime = `${startDate.split('T')[0]}T00:00:00`;
                    endTime = `${endDate.split('T')[0]}T23:59:59`;
                }
                // 请求参数
                const params = {
                    current: this.currentPage,
                    size: this.pageSize,
                    key: this.filterText,
                    startTime: startTime,
                    endTime: endTime,
                    ...this.userQueryDto
                };
                const response = await this.$axios.post('/user/query', params);
                const { data } = response;
                this.tableData = data.data;
                this.totalItems = data.total;
                this.selectedRows = [];
            } catch (error) {
                console.error('查询用户信息异常:', error);
            }
        },
        add() {
            this.dialogUserOperaion = true;
        },
        handleFilter() {
            this.currentPage = 1;
            this.fetchFreshData();
        },
        handleFilterClear() {
            this.filterText = '';
            this.handleFilter();
        },
        handleSizeChange(val) {
            this.pageSize = val;
            this.currentPage = 1;
            this.fetchFreshData();
        },
        handleCurrentChange(val) {
            this.currentPage = val;
            this.fetchFreshData();
        },
        handleEdit(row) {
            this.dialogUserOperaion = true;
            this.isOperation = true;
            row.userPwd = null;
            this.userAvatar = row.userAvatar;
            this.data = { ...row }
        },
        handleDelete(row) {
            this.selectedRows.push(row);
            this.batchDelete();
        }
    },
};
</script>
