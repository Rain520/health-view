<template>
    <div>
        <div class="header">
            <el-row>
                <el-col :span="6">
                    <Logo sysName="运动健康" />
                </el-col>
                <el-col :span="18">
                    <span class="user-info">
                        <img class="user-avatar" :src="userInfo.userAvatar" />
                        <span class="user-name">{{ userInfo.userName }}</span>
                    </span>
                </el-col>
            </el-row>
        </div>

        <div class="divider"></div>

        <div class="message-container">
            <div class="message-header">
                <p class="message-title">
                    消息中心
                    <el-tooltip class="item" effect="dark" content="点击标记所有消息为已读" placement="left">
                    <span @click="clearMessage" class="clear-message">
                        <i class="el-icon-s-open"></i> 
                    </span>
                </el-tooltip>
                </p>
            </div>


            <div class="message-types">
                <span @click="messageTypeSelected(messageType.type)" class="message-type"
                    v-for="(messageType, index) in messageTypes" :key="index">
                    {{ messageType.detail }}
                </span>


            </div>

            <div class="message-list">
                <div class="message-item" :style="{ backgroundColor: !message.isRead ? '#F8F8F8' : '' }"
                    v-for="(message, index) in messageList" :key="index">
                    <el-row class="message-row">
                        <el-col :span="1" class="message-icon">
                            <span class="red-dot" v-if="!message.isRead"></span>
                            <span class="bell-icon">
                                <i v-if="message.messageType === 1" class="el-icon-chat-line-round"></i>
                                <i v-else-if="message.messageType === 2" class="el-icon-s-promotion"></i>
                                <i v-else-if="message.messageType === 3" class="el-icon-data-analysis"></i>
                                <i v-else class="el-icon-message-solid"></i>
                            </span>
                        </el-col>
                        <el-col :span="19">
                            <div class="message-content-wrapper">
                                <div>
                                    <span class="message-content">{{ message.content }}</span>
                                </div>
                                <div>
                                    <span class="message-time">{{ message.createTime }}</span>
                                </div>
                            </div>
                        </el-col>
                        <el-col :span="4" class="message-actions">
                            <el-button v-if="!message.isRead" size="mini" type="success" plain icon="el-icon-check"
                                @click="markAsRead(message)">
                                未读
                            </el-button>
                            <span v-else class="read-status">
                                <i class="el-icon-circle-check"></i> 已读
                            </span>
                        </el-col>
                    </el-row>
                </div>
            </div>
        </div>
    </div>
</template>


<script>
import Logo from '@/components/Logo';
import { message } from 'js-md5';
export default {
    components: { Logo },
    data() {
        return {
            userInfo: {},
            messageQueryDto: {},
            messageList: [],
            messageTypes: [],
            dialogEvaluationsOperation: false,
            message: {},
        };
    },
    created() {
        this.getUserInfo();
        this.loadAllUsersMessage();
        this.loadAllMessageType();
    },
    methods: {
        async markAsRead(message) {
            console.log(message)
            const confirmed = await this.$swalConfirm({
                title: '标记已读',
                text: `是否将消息设置为已读？`,
                icon: 'warning',
            });
            if (confirmed) {
                this.$axios.put('/message/clearOneMessage', message).then(response => {
                    const { data } = response;
                    if (data.code === 200) {
                        this.loadAllUsersMessage();
                    }
                })
            }
        },
        async clearMessage() {
            const confirmed = await this.$swalConfirm({
                title: '消息清除',
                text: `是否将全部的消息设置为已读？`,
                icon: 'warning',
            });
            if (confirmed) {
                this.$axios.put('/message/clearMessage').then(response => {
                    const { data } = response;
                    if (data.code === 200) {
                        this.loadAllUsersMessage();
                    }
                })
            }
        },
        replyEvalustions(message) {
            this.message = message;
            this.dialogEvaluationsOperation = true;
        },
        messageTypeSelected(messageType) {
            this.messageQueryDto.messageType = messageType;
            this.loadAllUsersMessage();
        },
        getUserInfo() {
            const userInfo = sessionStorage.getItem('userInfo');
            this.userInfo = JSON.parse(userInfo);
        },
        loadAllMessageType() {
            this.$axios.get('/message/types').then(response => {
                const { data } = response;
                if (data.code === 200) {
                    this.messageTypes = data.data;
                    const messageType = { type: null, detail: '全部消息' };
                    this.messageTypes.unshift(messageType);
                    this.messageTypes.map(entity => entity.isCheck = false);
                }
            })
        },
        loadAllUsersMessage() {
            const userInfo = sessionStorage.getItem('userInfo');
            const entity = JSON.parse(userInfo);
            this.messageQueryDto.userId = entity.id;
            this.$axios.post('/message/query', this.messageQueryDto).then(response => {
                const { data } = response;
                if (data.code === 200) {
                    this.messageList = data.data;
                }
            })
        },
    },
};
</script>

<style scoped lang="scss">
.header {
    line-height: 70px;
    padding: 0 50px;
    background-color: #fff;
}

.user-info {
    float: right;
    display: flex;
    align-items: center;
    flex-wrap: wrap;
}

.user-avatar {
    width: 30px;
    height: 30px;
    border-radius: 50%;
}

.user-name {
    margin-left: 8px;
    font-size: 14px;
    color: #333;
}

.divider {
    height: 20px;
    background-color: rgb(248, 248, 248);
}

.message-container {
    padding: 10px 50px;
}

.message-header {
    padding: 10px 0;
}

.message-title {
    font-size: 16px;
    display: flex;
    align-items: center;
}

.clear-message {
    margin-left: 10px;
    cursor: pointer;
}

.clear-message:hover {
    background-color: #f1f1f1;
    padding: 4px;
    border-radius: 5px;
}

.message-types {
    padding: 15px 0;
    display: flex;
    flex-wrap: wrap;
}

.message-type {
    font-size: 16px;
    margin-right: 25px;
    cursor: pointer;
    color: #3c3c3c;
    padding: 5px 10px;
    border-radius: 20px;
    transition: background-color 0.3s ease;
}

.message-type:hover {
    background-color: #e2f0f9;
}

.message-list {
    padding: 30px 0;
}

.message-item {
    margin-bottom: 10px;
    padding: 15px 20px;
    border-radius: 10px;
    transition: background-color 0.3s ease;
}

.message-row {
    padding: 10px 0;
}

.bell-icon {
    display: inline-block;
    height: 40px;
    width: 40px;
    border-radius: 20px;
    background-color: rgb(82, 152, 237);
    border: 3px solid rgb(212, 227, 230);

    i {
        line-height: 40px;
        width: 40px;
        text-align: center;
        font-size: 20px;
        color: #f1f1f1;
    }
}

.message-content-wrapper {
    margin-left: 10px;
}

.message-content {
    font-size: 14px;
    color: #333;
}

.message-time {
    font-size: 12px;
    color: rgb(131, 104, 102);
}

.message-actions {
    display: flex;
    align-items: center;
    justify-content: flex-end;
}

.el-button {
    font-size: 14px;
    border-radius: 20px;
    padding: 3px 12px;
    transition: all 0.3s ease;

    &:hover {
        background-color: #e0f7fa;
        color: #00796b;
    }

    .el-icon-check {
        margin-right: 5px;
    }
}

.read-status {
    display: flex;
    align-items: center;
    font-size: 14px;
    color: #67c23a;
    background-color: #f0f9eb;
    padding: 3px 10px;
    border-radius: 20px;

    i {
        margin-right: 5px;
    }
}

.red-dot {
    display: inline-block;
    width: 8px;
    height: 8px;
    background-color: #ff4d4f;
    border-radius: 50%;
    position: absolute;
    top: 6px;
    left: 6px;
}

/* 图标容器样式 */
.message-icon {
    position: relative;
}

/* 调整按钮样式 */
.message-actions .el-button {
    margin-top: 5px;
}

.read-status {
    font-size: 14px;
    color: #52c41a;
}
</style>
