<template>
	<view class="flex justify-between padding">
		<text>{{ account.currency }}</text>
		<text>{{ account.available }}</text>
		<text>{{ account.frozen }}</text>
	</view>
</template>

<script>
export default {
	props: {
		account: {
			type: Object|Array,
			default: () => {}
		}
	}
};
</script>

<style scoped></style>
