<template>
    <el-scrollbar height="100vh">
        <p />
        <br>
        <div class="header">
            <el-menu :default-active="activeIndex" class="el-menu-demo" background-color="#545c64" text-color="#fff"
                active-text-color="#ffd04b" mode="horizontal" :ellipsis="false" @select="handleSelect">
                <el-menu-item index="1" style="font-size: 16px;">树洞</el-menu-item>
                <div class="flex-grow" />
                <el-menu-item index="2" style="font-size: 16px;">个人中心</el-menu-item>
            </el-menu>
        </div>
        <div class="container">
            <br />
            <div v-show="activeIndex === '1'">
                <el-text tag="b" size="large" style="font-style: italic;">##
                    每个人都需要一个树洞，存放那些不可轻易示人的秘密、引而不发的情绪、难以启齿的柔弱。</el-text>
                <br />
                <el-divider />
                <el-row v-if="activities.length === 0">
                    <el-empty description="暂无发贴信息" />
                </el-row>
                <el-row v-else v-for="activity in activities" :key="activity._id">
                    <el-col :span="24">
                        <div class="activity-container">
                            <div class="left-section">
                                <div class="name">
                                    <el-text :style="{ fontWeight: 'bold' }">{{ activity.author }}</el-text>
                                </div>
                                <div class="time">
                                    <el-text :style="{ fontSize: '12px', color: '#888' }">@{{ formatTime(activity.time)
                                    }}</el-text>
                                </div>
                            </div>
                            <div class="right-section">
                                <el-text class="description">{{ activity.description }}</el-text>
                                <div class="action-buttons flex justify-space-between flex-wrap gap-2">
                                    <el-link :underline="false" size="small" type="danger"
                                        :style="{ fontSize: '12px', color: '#red' }"
                                        @click="handleAction(activity._id, 'like')">
                                        oo [ {{ activity.like }} ]
                                    </el-link>
                                    <el-link :underline="false" size="small" type="success"
                                        :style="{ fontSize: '12px', color: '#green' }"
                                        @click="handleAction(activity._id, 'dislike')">
                                        xx [ {{ activity.dislike }} ]
                                    </el-link>
                                </div>
                            </div>
                        </div>
                        <el-divider />
                    </el-col>
                </el-row>
                <alert :message="message" v-if="showMessage" />
                <el-form :model="activity" ref="activityForm" label-width="auto">
                    <el-form-item label="昵称:" prop="author">
                        <el-input v-model="activity.author" size="small" style="width: 20%;" clearable></el-input>
                    </el-form-item>
                    <el-form-item label="邮箱:" prop="email" :rules="emailRules">
                        <el-input v-model="activity.email" size="small" style="width: 20%;" clearable></el-input>
                    </el-form-item>
                    <el-input v-model="activity.description" maxlength="1000" show-word-limit
                        :autosize="{ minRows: 3, maxRows: 6 }" type="textarea" />
                    <p />
                    <el-form-item>
                        <el-button @click="submitActivity" :loading="submitting"
                            style="width: 100%; background-color: #f7f9fc;">点击发布</el-button>
                    </el-form-item>
                </el-form>
            </div>
            <div v-show="activeIndex === '2'">
                <el-text size="large" style="font-style: italic;">## 个人中心</el-text>
                <el-divider />
                <p>功能开发中...</p>
            </div>
            <p />
            <Footer />
        </div>
    </el-scrollbar>
</template>  
  
<script>
import Alert from '@/components/Alert.vue';
import Footer from '@/components/Footer.vue';
export default {
    data() {
        return {
            activeIndex: '1',
            activity: { author: '', email: '', description: '', time: new Date().toISOString(), 'like': 0, 'dislike': 0 },
            activities: [],
            submitting: false,
            emailRules: [
                { type: 'email', message: '输入有效的邮箱', trigger: ['blur', 'change'] },
            ],
            message: '',
            showMessage: false,
        };
    },
    components: { alert: Alert, Footer },
    methods: {
        handleSelect(index) {
            this.activeIndex = index;
        },
        formatTime(time) {
            const now = new Date();
            const activityTime = new Date(time);
            const timeDiff = now - activityTime;
            const seconds = Math.floor(timeDiff / 1000);
            const minutes = Math.floor(seconds / 60);
            const hours = Math.floor(minutes / 60);
            const days = Math.floor(hours / 24);

            if (days > 0) {
                return `${days}天前`;
            } else if (hours > 0) {
                return `${hours}小时前`;
            } else if (minutes > 0) {
                return `${minutes}分钟前`;
            } else {
                return '刚刚';
            }
        },
        async fetchActivities() {
            this.activities = (await this.$axios.get('/post')).data;
        },
        async submitActivity() {
            this.$refs.activityForm.validate();
            if (this.activity.author.trim() && this.activity.email.trim() && this.activity.description.trim()) {
                try {
                    this.submitting = true;
                    await this.$axios.post('/post', this.activity);
                    this.fetchActivities();
                    this.clearForm();
                    this.message = '发布成功！';
                    this.showMessage = true;
                } finally {
                    this.submitting = false;
                }
            } else {
                this.message = '请填写所有信息！';
                this.showMessage = true;
            }
        },
        async handleAction(_id, actionType) {
            try {
                await this.$axios.post(`/${actionType}`, { _id });
                this.fetchActivities();
            } catch (error) {
                this.showMessage = true;
            }
        },
        clearForm() {
            this.activity.description = '';
        },
    },
    mounted() {
        this.fetchActivities();
    },
};
</script>
  
<style>
.header {
    width: 40% !important;
    margin: 0 auto;
}

.container {
    width: 40% !important;
    margin: 0 auto;
    display: flex;
    flex-direction: column;
    padding: 10px;
    border: 1px solid #ddd;
    margin-bottom: 10px;
}

.flex-grow {
    flex-grow: 1;
}

.left-section {
    display: flex;
    flex-direction: column;
    width: 24%;
    margin-bottom: 5px;
    align-items: flex-start;
}

.name,
.time {
    align-self: flex-start;
}

.right-section {
    flex-direction: column;
    width: 100%;
    margin-bottom: 5px;
}

.description {
    font-size: 14px;
    color: #333;
}

.action-buttons {
    display: flex;
    justify-content: flex-end;
}

.action-buttons el-button {
    margin-left: 10px;
}

.activity-container {
    display: flex;
    justify-content: space-between;
    padding: 10px;
}
</style>