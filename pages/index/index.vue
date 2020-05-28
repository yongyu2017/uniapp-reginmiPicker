<template>
	<view class="content">
		<reginmiPicker :areaValueArr="selectCity" :cityData="cityData" @change="cityChange" :beforeClose="beforeCloseFun"></reginmiPicker>
	</view>
</template>

<script>
	import reginmiPicker from '@/components/common/reginmiPicker.vue'
	import { city } from '@/utils/city.js'
	
	export default {
		components: {
			reginmiPicker,
		},
		data() {
			return {
				title: 'Hello',
				cityData: [],  //省市区数据
				selectCity: ['', '', ''],  //省市区选中值
			}
		},
		onLoad() {
			console.log('onLoad')
			this.cityData= city;
		},
		onReady() {
			console.log('onReady')
		},
		methods: {
			//省市区，change
			cityChange: function (e) {
				this.selectCity= e.map((value, index, array)=> {
					return value.code
				})				
			},
			beforeCloseFun: function (closeModel) {
				uni.showModal({
					title: '提示',
					content: '确定关闭？',
					success: (res)=> {
						if(res.confirm){
							closeModel()
						}
					},
				})
			},
		}
	}
</script>

<style lang="scss" scoped>
	.m-wrap{
		position: relative;
		z-index: 0;
	}
	.map-dom{
		position: relative;
		z-index: 0;
		.txt{
			position: absolute;
			z-index: -1;
			top: 86rpx;
			left: 10rpx;
			width: 100%;
			height: 44rpx;
			line-height: 44rpx;
			background: #fff;
		}
	}
	.bt-dom{
		position: absolute;
		z-index: 2;
		top: 0;
		left: 0;
		width: 100%;
		height: 110rpx;
		background: #fff;
	}
</style>
