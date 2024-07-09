<template>
  <div>
    正在处理 GitHub 登录...
  </div>
</template>

<script>
import axios from 'axios';
import { Message } from 'element-ui'; // 确保已经安装并正确引入
import { mapActions, mapMutations } from 'vuex';
export default {
  name: 'GitHubCallback',
  methods: {
    ...mapActions('user', ['login']),
    ...mapMutations('user', ['SET_TOKEN']),
    async handleCallback() {
      const code = this.$route.query.code;
      const state = this.$route.query.state;

      if (!code) {
        Message.error("登录失败: code 为空");
        return;
      }

      try {
        // 将 code 和 state 发送到后端进行处理
        const response = await axios.post('http://localhost:9999/api/auth/github', { code, state });
        if (response.data.message="success") {
          console.log(response.data);
          const token = response.data.data.token;
          // 将 token 存储在 Vuex 中
          await this.SET_TOKEN (token);
          const user = {
            userName: response.data.data.user.userName,
            githubId: response.data.data.user.githubId,
          }
          console.log(user);
          

          this.$store.dispatch('user/oauthLogin', user).then(() => {
              this.$router.push({ path: this.redirect || '/' }) 
              this.loading = false})

          
        } else {
          Message.error("登录失败");
        }
      } catch (error) {
        console.error(error);
        Message.error("登录过程中出现错误");
      }
    },
  },
  mounted() {
    this.handleCallback();
  }
};
</script>

<style scoped>
/* 添加必要的样式 */
</style>
