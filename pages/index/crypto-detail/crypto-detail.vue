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
			console.log(params.instrument_id);
			this.data = params.instrument_id
		},
		methods: {
			
		}
	}
</script>

<style>

</style>
