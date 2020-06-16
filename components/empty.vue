<template>
	<view class="padding flex justify-center align-center flex-direction margin-top-lg padding-top-lg">
		<view class="maingi-top padding-top"><text class="cuIcon-pick text-white" style="font-size: 230rpx;"></text></view>
		<view class="margin-top">
			<text class="text-xxl text-white">{{ msg }}</text>
		</view>
	</view>
</template>
<script>
export default {
	data() {
		return {
			msg: `NO DATA`
		};
	}
};
</script>

<style scoped></style>
