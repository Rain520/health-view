<template>
    <el-row style="background-color: #FFFFFF;padding: 10px 0;border-radius: 5px;">
        <el-row style="padding: 10px 0 20px;border-bottom: 1px solid #f1f1f1;margin: 0 10px;">
            <el-row>
                <span class="top-bar">记录时间</span>
                <el-date-picker size="small" style="margin-left: 10px;width: 220px;" v-model="searchTime"
                    type="daterange" range-separator="至" start-placeholder="开始时间" end-placeholder="结束时间">
                </el-date-picker>
                <span class="top-bar">模型名</span>
                <el-select style="width: 120px;" size="small" v-model="healthModelConfigQueryDto.healthModelConfigId" placeholder="请选择"
                    clearable @clear="handleFilterClear">
                    <el-option v-for="model in modelList" :key="model.id" :label="model.name" :value="model.id">
                    </el-option>
                </el-select>
                <el-button size="small" class="customer"
                    style="margin-left: 20px;background-color: rgb(43, 121, 203);border: none;" type="primary"
                    @click="handleFilter">立即查询</el-button>
                <el-button size="small" class="customer reset"
                    style="background-color: #f1f1f1;border: none;color: #909399;border: 1px solid #f1f1f1;" type="info"
                    @click="resetQueryCondition">条件重置</el-button>
                <el-button size="small" class="customer"
                    :style="{ marginLeft: '10px', backgroundColor: selectedRows.length ? '#a7535a' : '#F1F1F1', border: 'none', color: selectedRows.length ? '#FFFFFF' : '#909399' }"
                    type="danger" @click="batchDelete()">批量删除</el-button>
            </el-row>
        </el-row>
        <el-row style="margin: 10px;">
            <el-table row-key="id" @selection-change="handleSelectionChange" :data="tableData" style="width: 100%">
                <el-table-column type="selection" width="55"></el-table-column>
                <el-table-column prop="name" width="88" label="状态">
                    <template slot-scope="scope">
                        <span class="status-success" v-if="statusCheck(scope.row)">正常</span>
                        <span class="status-error" v-else>异常</span>
                    </template>
                </el-table-column>
                <el-table-column prop="name" width="128" label="模型名"></el-table-column>
                <el-table-column prop="userName" width="168" label="用户"></el-table-column>
                <el-table-column prop="value" width="158" label="用户记录值"></el-table-column>
                <el-table-column prop="valueRange" width="128" label="正常值范围"></el-table-column>
                <el-table-column prop="unit" width="88" label="单位"></el-table-column>
                <el-table-column prop="symbol" width="88" label="符号"></el-table-column>
                <el-table-column prop="createTime" width="178" label="记录时间" sortable></el-table-column>
                <el-table-column label="操作">
                    <template slot-scope="scope">
                        <span class="text-button" @click="handleDelete(scope.row)">删除</span>
                    </template>
                </el-table-column>
            </el-table>
            <el-pagination style="margin: 20px 0;" @size-change="handleSizeChange" @current-change="handleCurrentChange"
                :current-page="currentPage" :page-sizes="[5, 7]" :page-size="pageSize"
                layout="total, sizes, prev, pager, next, jumper" :total="totalItems"></el-pagination>
        </el-row>
        <el-dialog :show-close="false" :visible.sync="dialogUserOperaion" width="26%">
            <div slot="title">
                <p class="dialog-title">{{ !isOperation ? '新增用户健康记录' : '编辑用户健康记录信息' }}</p>
            </div>
            <div style="padding:0 20px;">
                <!-- 图标 -->
                <el-row style="margin-top: 20px;">
                    <el-upload class="avatar-uploader" action="/api/personal-heath/v1.0/file/upload"
                        :show-file-list="false" :on-success="handleAvatarSuccess">
                        <img v-if="data.cover" :src="data.cover" style="height: 44px;width: 44px;">
                        <i v-else class="el-icon-plus avatar-uploader-icon"></i>
                    </el-upload>
                </el-row>
                <!-- 配置名 -->
                <el-row style="padding: 0 20px 0 0;">
                    <p>
                        <span class="modelName">*配置名</span>
                    </p>
                    <input class="input-title" v-model="data.name"
                        style="border-radius: 5px;background-color: #f1f1f1;">
                </el-row>
                <!-- 单位 -->
                <el-row style="padding: 0 20px 0 0;">
                    <p style="font-size: 12px;padding: 3px 0;">
                        <span class="modelName">*单位</span>
                    </p>
                    <input class="input-title" v-model="data.unit"
                        style="border-radius: 5px;background-color: #f1f1f1;">
                </el-row>
                <!-- 符号 -->
                <el-row style="padding: 0 20px 0 0;">
                    <p style="font-size: 12px;padding: 3px 0;">
                        <span class="modelName">*符号</span>
                    </p>
                    <input class="input-title" v-model="data.symbol"
                        style="border-radius: 5px;background-color: #f1f1f1;">
                </el-row>
                <!-- 简介 -->
                <el-row style="padding: 0 20px 0 0;">
                    <p style="font-size: 12px;padding: 3px 0;">
                        <span class="modelName">*简介</span>
                    </p>
                    <el-input style="border-radius: 5px;background-color: #f1f1f1;" type="textarea"
                        :autosize="{ minRows: 2, maxRows: 3 }" placeholder="简介" v-model="data.detail">
                    </el-input>
                </el-row>
                <!-- 简介 -->
                <el-row style="padding: 0 20px 0 0;">
                    <p style="font-size: 12px;padding: 3px 0;">
                        <span class="modelName">*正常值范围</span>
                    </p>
                    <el-slider v-model="valuesRange" range show-stops :max="1000">
                    </el-slider>
                </el-row>
            </div>
            <span slot="footer" class="dialog-footer">
                <el-button size="small" v-if="!isOperation" style="background-color: rgb(43, 121, 203);border: none;"
                    class="customer" type="info" @click="addOperation">新增</el-button>
                <el-button size="small" v-else style="background-color: rgb(43, 121, 203);border: none;"
                    class="customer" type="info" @click="updateOperation">修改</el-button>
                <el-button class="customer" size="small" style="background-color: rgb(241, 241, 241);border: none;"
                    @click="dialogUserOperaion = false">取消</el-button>
            </span>
        </el-dialog>
    </el-row>
</template>

<script>
export default {
    data() {
        return {
            data: {},
            filterText: '',
            currentPage: 1,
            pageSize: 7,
            totalItems: 0,
            dialogUserOperaion: false, // 开关
            isOperation: false, // 开关-标识新增或修改
            tableData: [],
            searchTime: [],
            selectedRows: [],
            status: null,
            healthModelConfigQueryDto: {}, // 搜索条件
            messsageContent: '',
            modelList: [],
            valuesRange: [10, 50]
        };
    },
    watch: {
        dialogUserOperaion(v1, v2) {
            if (!v1) {
                this.isOperation = !this.isOperation;
            }
            if (!v1 && v2) {
                this.data = {};
            }
        },
    },
    created() {
        this.fetchFreshData();
        this.loadAllTags()
    },
    methods: {
        // 处理用户输入的值，是正常的还是异常的，给个状态
        statusCheck(data) {
            // 用户输入的值
            const inputValue = data.value;
            // 正常值范围
            const valueRange = data.valueRange;
            if (valueRange !== null && inputValue !== null) {
                const aryValueRange = valueRange.split(',');
                const minValue = aryValueRange[0];
                const maxValue = aryValueRange[1];
                return (Number(inputValue) > Number(minValue) && Number(inputValue) < Number(maxValue))
            }
        },
        handleAvatarSuccess(res, file) {
            if (res.code !== 200) {
                this.$message.error(`用户健康记录封面上传异常`);
                return;
            }
            this.$message.success(`用户健康记录封面上传成功`);
            this.data.cover = res.data;
        },
        // 加载全部的资讯数据
        loadAllTags() {
            this.$axios.post(`/health-model-config/modelList`, {}).then(response => {
                const { data } = response;
                if (data.code === 200) {
                    this.modelList = data.data;
                }
            })
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
                title: '删除用户健康记录数据',
                text: `删除后不可恢复，是否继续？`,
                icon: 'warning',
            });
            if (confirmed) {
                try {
                    let ids = this.selectedRows.map(entity => entity.id);
                    const response = await this.$axios.post(`/user-health/batchDelete`, ids);
                    if (response.data.code === 200) {
                        this.$swal.fire({
                            title: '删除提示',
                            text: response.data.msg,
                            icon: 'success',
                            showConfirmButton: false,
                            timer: 2000,
                        });
                        this.fetchFreshData();
                        return;
                    }
                } catch (e) {
                    this.$swal.fire({
                        title: '错误提示',
                        text: e,
                        icon: 'error',
                        showConfirmButton: false,
                        timer: 2000,
                    });
                    console.error(`用户健康记录信息删除异常：`, e);
                }
            }
            this.selectedRows = [];
        },
        resetQueryCondition() {
            this.healthModelConfigQueryDto = {};
            this.searchTime = [];
            this.fetchFreshData();
        },
        // 修改信息
        async updateOperation() {
            try {
                this.data.valueRange = this.valuesRange.join(',');
                const response = await this.$axios.put('/user-health/update', this.data);
                this.$swal.fire({
                    title: '用户健康记录信息修改',
                    text: response.data.msg,
                    icon: response.data.code === 200 ? 'success' : 'error',
                    showConfirmButton: false,
                    timer: 1000,
                });
                if (response.data.code === 200) {
                    this.closeDialog();
                    this.fetchFreshData();
                    this.clearFormData();
                }
            } catch (error) {
                console.error('提交表单时出错:', error);
                this.$message.error('提交失败，请稍后再试！');
            }
        },
        // 信息新增
        async addOperation() {
            try {
                // [20,252] ---> 20,252
                this.data.valueRange = this.valuesRange.join(',');
                const response = await this.$axios.post('/user-health/save', this.data);
                this.$message[response.data.code === 200 ? 'success' : 'error'](response.data.msg);
                if (response.data.code === 200) {
                    this.closeDialog();
                    this.fetchFreshData();
                    this.clearFormData();
                }
            } catch (error) {
                console.error('提交表单时出错:', error);
                this.$message.error('提交失败，请稍后再试！');
            }
        },
        closeDialog() {
            this.dialogUserOperaion = false;
        },
        clearFormData() {
            this.data = {};
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
                    startTime: startTime,
                    endTime: endTime,
                    ...this.healthModelConfigQueryDto
                };
                const response = await this.$axios.post('/user-health/query', params);
                const { data } = response;
                this.tableData = data.data;
                this.totalItems = data.total;
                this.selectedRows = [];
            } catch (error) {
                console.error('查询用户健康记录信息异常:', error);
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
            if (row.valueRange !== null) {
                this.valuesRange = row.valueRange.split(',');
            }
            this.data = { ...row }
        },
        handleDelete(row) {
            this.selectedRows.push(row);
            this.batchDelete();
        }
    },
};
</script>
<style scoped lang="scss">
.status-success,
.status-error {
    display: inline-block;
    padding: 1px 5px;
    border-radius: 2px;
}

.status-success {
    background-color: rgb(232, 250, 158);
    color: #6d0ea1;
}

.status-error {
    background-color: rgb(227, 171, 147);
    color: #6d0ea1;
}

.tag-tip {
    display: inline-block;
    padding: 5px 10px;
    border-radius: 5px;
    background-color: rgb(245, 245, 245);
    color: rgb(104, 118, 130);
}

.input-def {
    height: 40px;
    line-height: 40px;
    outline: none;
    border: none;
    font-size: 20px;
    color: rgb(102, 102, 102);
    font-weight: 900;
    width: 100%;
}

.dialog-footer {
    /* 使按钮水平居中 */
    display: flex;
    justify-content: center;
    align-items: center;
}

/* 如果需要调整按钮之间的间距 */
.customer {
    margin: 0 8px;
    /* 根据需要调整间距 */
}
</style>