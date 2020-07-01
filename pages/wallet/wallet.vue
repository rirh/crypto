<template>
	<view>
		<view v-if="userInfo" class="wallet " :style="{ 'padding-top': `${statusBarHeight - 5}px` }">
			<view class="padding flex justify-start align-center text-white text-xxl text-bold">
				<text @click="secret = !secret" :class="secret ? 'cuIcon-attentionforbidfill' : 'cuIcon-attentionfill'"></text>
			</view>
			<view class="padding-left padding-right padding-bottom ">
				<view class="">
					<text class="text-sm " style="color: #c6c6c6;">总账户资产折合({{ wallet.valuation_currency || blan.valuation_currency }})</text>
				</view>
				<view class="margin-top-xs flex justify-center align-center">
					<view class="">
						<text @click="login" class="text-xl text-white ">
							{{ secret ? '********' : wallet.balance || blan.balance }}&nbsp;{{ secret ? '***' : wallet.valuation_currency || blan.valuation_currency }}
						</text>
					</view>
					<view class="text-white flex justify-end flex-sub align-center">
						<!-- <text>{{userInfo.userInfo.nickName}}</text> -->
						<image style="border-radius: 5px;border: 1rpx solid #fff;" class="cu-avatar margin-left" :src="userInfo.userInfo.avatarUrl" mode="scaleToFill"></image>
					</view>
				</view>
				<!-- <view class="margin-top"><button class="cu-btn bg-gradual-orange">划转</button></view> -->
			</view>
		</view>
		<view v-if="!userInfo" :style="{ 'padding-top': `${statusBarHeight}px` }" class="flex justify-center align-center padding">
			<button style="width: 100%;" class="cu-btn bg-grey lg" open-type="getUserInfo" @getuserinfo="handle_login">登录</button>
		</view>
		<scroll-view scroll-with-animation scroll-x class="bar padding">
			<text
				class="item margin-right  "
				v-for="(asset, index) in asset_type_arr"
				:class="cursor === asset.value ? 'text-blue text-bold' : 'text-white'"
				:key="index"
				@click="handle_async_asset(asset)"
			>
				{{ asset.lable }}
			</text>
		</scroll-view>
		<scroll-view scroll-y style="height: 70vh">
			<view class="text-white" v-if="cursor === 1">
				<view class="flex justify-between padding-left padding-top padding-right ">
					<text>币种</text>
					<text>可用</text>
					<text>冻结</text>
				</view>
				<view v-for="(account, i) in account_info" :key="i"><Account :account="account"></Account></view>
			</view>
			<view v-else-if="cursor === 9" class="padding text-white" :key="j" v-for="(swap, j) in swap_account"><Swap :swap="swap"></Swap></view>
			<view v-else-if="cursor === 16" class="padding text-white" :key="k" v-for="(swap, k) in swap_usdt_account"><SwapUsdt :swap="swap"></SwapUsdt></view>
		</scroll-view>
	</view>
</template>

<script>
import config from 'config';
import { get_asset_valuation, get_wx_user_info } from 'constant';
import Swap from './Swap.vue';
import SwapUsdt from './SwapUsdt.vue';
import Account from './Account.vue';
const userInfo = uni.getStorageSync(get_wx_user_info);
export default {
	components: {
		Swap,
		Account,
		SwapUsdt
	},
	data() {
		return {
			userInfo,
			secret: false,
			wallet: {},
			account_info: false,
			cursor: 1,
			asset_type_arr: [
				{
					value: 9,
					lable: '永续合约'
				},
				{
					value: 1,
					lable: '币币'
				},
				{
					value: 16,
					lable: '永续USDT保证金账户'
				},
				{
					value: 3,
					lable: '交割'
				},

				// {
				// 	value: 4,
				// 	lable: '法币'
				// },

				{
					value: 5,
					lable: '币币杠杆'
				},

				{
					value: 6,
					lable: '资金账户'
				},

				// {
				// 	value: 8,
				// 	lable: '余币宝账户'
				// },

				{
					value: 15,
					lable: '交割USDT保证金账户'
				}
			]
		};
	},
	computed: {
		swap_account() {
			let result = [];
			if (this.cursor === 9) {
				const data = this.account_info.info;
				if (Array.isArray(this.account_info.info)) result = data.filter(e => e.total_avail_balance > 0);
			}
			return result;
		},
		swap_usdt_account() {
			let result = [];
			if (this.cursor === 16) {
				const data = this.account_info.info;
				if (Array.isArray(this.account_info.info)) result = data.filter(e => e.total_avail_balance > 0);
			}
			return result;
		},
		statusBarHeight() {
			const { statusBarHeight } = uni.getSystemInfoSync();
			return statusBarHeight;
		},
		blan() {
			return uni.getStorageSync(get_asset_valuation);
		}
	},
	onTabItemTap() {
		uni.vibrateShort();
	},
	onShow() {
		this.fetch_asset_valuation();
		this.handle_async_asset({ value: this.cursor });
		uni.getUserInfo({
			provider: 'weixin',
			success: function(res) {
				console.log(res);
				if (res) {
					uni.setStorageSync(get_wx_user_info, res);
				}
			}
		});
	},
	methods: {
		handle_login(res) {
			uni.setStorageSync(get_wx_user_info, res.detail);
		},
		handle_async_asset(item) {
			this.cursor = item.value;
			uni.request({
				url: `${config.host}/app/get_account_info/`,
				method: 'GET',
				data: {
					account_type: item.value
				},
				success: res => {
					if (res.statusCode === 200) {
						this.account_info = res.data;
						uni.vibrateShort();
					}
				},
				fail: () => {},
				complete: () => {}
			});
		},
		fetch_asset_valuation() {
			uni.request({
				url: `${config.host}/app/get_asset_valuation/`,
				method: 'GET',
				data: {},
				success: ({ statusCode, data }) => {
					this.wallet = data;
					uni.setStorageSync(get_asset_valuation, data);
				},
				fail: () => {},
				complete: () => {}
			});
		}
	}
};
</script>

<style scoped lang="scss">
.wallet {
	background-color: $color-green;
}
.bar {
	white-space: nowrap;
	width: 100%;
	border-bottom: 1rpx solid #000;
	// box-shadow: 1rpx 1rpx #000000;
}
.item {
	display: inline-block;
	text-align: center;
	font-size: 28rpx;
}
</style>
