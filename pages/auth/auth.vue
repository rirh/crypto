<template>
	<view>
		<button open-type="getUserInfo" @getuserinfo="handle_add_wx_user">微信登录</button>
		count: {{items.length}} 
		<view class="flex text-white justify-center align-center padding border solid radius" v-for="item in items" :key="item.id">
			<rich-text style="word-break: break-all;" :nodes="JSON.stringify(item)"></rich-text>
		</view>
		<form>
			<view class="cu-form-group margin-top">
				<view class="title">用户名</view>
				<input v-model="form.account" placeholder="account" name="input" />
			</view>
			<view class="cu-form-group">
				<view class="title">密码</view>
				<input v-model="form.password" placeholder="password" name="input" />
			</view>
			<view class="flex justify-center align-center margin-top">
				<button :loading="loading" style="width: 80%;" class="cu-btn lg bg-blue" @click="handle_add_user">login/register</button>
			</view>
		</form>
	</view>
</template>

<script>
export default {
	data() {
		return {
			loading: false,
			form: {
				account: '',
				password: ''
			},
			items: []
		};
	},
	onLoad() {
		this.fetch_get_user();
	},
	methods: {
		handle_add_wx_user(res) {
			this.fetch_add_user(res);
		},
		fetch_get_user() {
			uniCloud.callFunction({
				name: 'get_user',
				success: ({ result: { data } }) => {
					this.items = data.reverse();
				}
			});
		},
		fetch_add_user(data) {
			this.loading = true;
			if (!this.loading) return;
			uniCloud.callFunction({
				name: 'add_user',
				data,
				success: res => {
					this.fetch_get_user()
					console.log(res);
					this.loading = false;

					this.form.account = '';
					this.form.password = '';
					uni.navigateTo({
						url: '../index/index'
					});
				},
				fail: res => {
					console.log(res);
					this.loading = false;
				}
			});
		},
		handle_add_user() {
			this.fetch_add_user(this.form);
		}
	}
};
</script>

<style></style>
