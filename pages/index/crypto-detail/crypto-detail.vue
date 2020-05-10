<template>
	<view>
		{{data}}
	</view>
</template>

<script>
	export default {
		data() {
			return {
				data:{},
			}
		},
		onLoad(params) {
			this.data = params
		},
		methods: {
			
		}
	}
</script>

<style>

</style>
