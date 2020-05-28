<template>
	<view class="reginmiPicker">
		<view :class="['picker-txt', selectedAreaStr== ''? 'disabled': '']" @click="showModal">{{ selectedAreaStr== ''? placeholdertxt: selectedAreaStr }}</view>
		
		<view hover-stop-propagation :class="['png', isShowModal? 'active': '']" @click="hideModal"></view>
		
		<view :class="['reginmiPicker-box', isShowModal? 'active': '']">
			
			<view class="tab-box">
				<view :class="['li', tabIndex== 0? 'active': '', provincesSelectedIndex[0]!= -1? 'selected': '']" data-index="0" @click="tabChange">{{ provincesSelectedIndex[0]== -1? '省': provincesSelectedVal[0].name }}</view>
				<view :class="['li', tabIndex== 1? 'active': '', provincesSelectedIndex[1]!= -1? 'selected': '']" data-index="1" @click="tabChange">{{ provincesSelectedIndex[1]== -1? '城市': provincesSelectedVal[1].name }}</view>
				<view :class="['li', tabIndex== 2? 'active': '', provincesSelectedIndex[2]!= -1? 'selected': '']" data-index="2" @click="tabChange" v-show="isShowArea">{{ provincesSelectedIndex[2]== -1? '区县': provincesSelectedVal[2].name }}</view>
			</view>
			
			<view class="scroller-box">
				<!-- 省 -->
				<view class="co-box" v-show="tabIndex== 0">
					
					<view class="list-box" v-for="(item1, index1) in provincesArr" :key="index1">
						
						<view class="letter-box">{{ item1.letter }}</view>
						<view class="city-list fix">
							<view :class="['li', (provincesVal.pindex== index1 && provincesVal.cindex== index2)? 'active': '']" v-for="(item2, index2) in item1.data" :key="index2" :data-pindex="index1" :data-cindex="index2" :data-item="item2" @click="city_selectedFun">{{ item2.name }}</view>
						</view>
						
					</view>
					
				</view>
				<!-- 省 -->
				
				<!-- 市 -->
				<view class="co-box" v-show="tabIndex== 1">
					<view class="list-box">
						
						<view class="city-list no-letter-box fix">
							<view :class="['li', provincesSelectedIndex[1]== index? 'active': '']" v-for="(item, index) in cityArr" :key="index" :data-index="index" :data-item="item" @click="provinces_selectedFun">{{ item.name }}</view>
						</view>
						
					</view>
					
				</view>
				<!-- 市 -->
				
				<!-- 区县 -->
				<view class="co-box" v-show="tabIndex== 2">
					
					<view class="list-box">
						
						<view class="city-list no-letter-box fix">
							<view :class="['li', provincesSelectedIndex[2]== index? 'active': '']" v-for="(item, index) in areaArr" :key="index" :data-index="index" :data-item="item" @click="area_selectedFun">{{ item.name }}</view>
						</view>
						
					</view>
					
				</view>
				<!-- 区县 -->
				
			</view>
			
			<view class="op-box">
				<view class="btn-a" @click="hideModal">取消</view>
			</view>
			
		</view>
	</view>
	
</template>

<script>
import Pinyin from '@/utils/pinyin.js'

export default {
	props: {
		cityData: {
			type: Array,
			default: []
		},
		//省市区选中值的code
		areaValueArr: {
			type: Array,
			default: ['', '', '']
		},
		placeholdertxt: {
			type: String,
			default: '请选择区域'
		},
		//关闭前的回调，会暂停 Dialog 的关闭
		beforeClose: {
			type: Function,
			default: null
		}
	},
	watch: {
		cityData: function (newValue, oldValue) {
			console.log('cityData watch')
			
			this.provincesArrFun();
		},
		areaValueArr: function (newValue, oldValue) {
			console.log('areaValueArr watch', newValue)
			
			this.provincesSelectedIndexFun();
			this.cityArrFun();
			this.areaArrFun();
		},
	},
	data() {
		return {
			provincesArr: [],  //省数据
			cityArr: [],  //市数据
			areaArr: [],  //区数据
			provincesSelectedIndex: [-1, -1, -1], //省市区选中下标
			provincesSelectedVal: [],  //省市区选中值
			provincesVal: {
				pindex: -1,
				cindex: -1
			},  //省数据选中下标
			selectedAreaStr: '',  //省市区数据，显示字段
			tabIndex: 0,  //tab索引
			isShowArea: true,  //是否显示区的tab
			isShowModal: false,  //是否显示弹窗
		}
	},
	mounted() {
		console.log('mounted')
		
		this.provincesArrFun();
		this.provincesSelectedIndexFun();
		this.cityArrFun();
		this.areaArrFun();
	},
	methods: {
		//显示弹窗
		showModal: function () {
			console.log('showModal')
			
			this.isShowModal= true;
			this.$emit('show')
		},
		//取消
		hideModal: function () {
			if(this.beforeClose){
				this.beforeClose(this.closeModel)
			}else{
				this.closeModel();
			}
		},
		//关闭弹窗，0取消1省市区选择完成关闭弹窗
		closeModel: function (type) {
			if(type!= 1){
				//取消时，重置状态
				this.provincesSelectedIndexFun();
				this.cityArrFun();
				this.areaArrFun();
			}
			this.isShowModal= false;
			this.$emit('close')
		},
		//tab
		tabChange: function (e) {
			var Index= e.currentTarget.dataset.index;
			
			//市
			if(Index== 1 && this.provincesSelectedIndex[0]== -1){
				uni.showModal({
					title: '温馨提示',
					content: '请先选择省份',
					showCancel: false
				})
				return
			}
			//区
			if(Index== 2 && this.provincesSelectedIndex[1]== -1){
				uni.showModal({
					title: '温馨提示',
					content: this.provincesSelectedIndex[0]== -1? '请先选择省份': '请先选择城市',
					showCancel: false
				})
				return
			}
			this.tabIndex= Index;
		},
		//省选址
		city_selectedFun: function (e) {
			var item= e.currentTarget.dataset.item,
				pindex= e.currentTarget.dataset.pindex,
				cindex= e.currentTarget.dataset.cindex;
			
			this.cityData.forEach((value, index, array)=> {
				if(value.code== item.code){
					this.provincesSelectedIndex[0]= index;
				}
			})
			this.provincesSelectedVal[0]= {
				code: item.code,
				name: item.name
			}
			this.tabIndex= 1;
			this.isShowArea= true;  //是否显示区的tab
			this.provincesVal.pindex= pindex;  //省选中坐标
			this.provincesVal.cindex= cindex;  //省选中坐标
			this.provincesSelectedIndex[1]= -1;
			this.provincesSelectedIndex[2]= -1;
			
			this.cityArrFun();
			this.$forceUpdate();
		},
		//市选址
		provinces_selectedFun: function (e) {
			var Index= e.currentTarget.dataset.index,
				item= e.currentTarget.dataset.item;
			
			this.provincesSelectedIndex[1]= Index;
			this.provincesSelectedIndex[2]= -1;
			this.provincesSelectedVal[1]= {
				code: item.code,
				name: item.name
			}
			
			//选中的市，是否有区列表
			if(this.cityData[this.provincesSelectedIndex[0]].children[this.provincesSelectedIndex[1]].children.length!= 0){
				this.tabIndex= 2;
				this.isShowArea= true;  //是否显示区的tab
				
				this.areaArrFun();
				this.$forceUpdate();
			}else{
				this.isShowArea= false;  //是否显示区的tab
				this.provincesSelectedVal[2]= {
					code: '',
					name: ''
				}
				this.selectedAreaStr= this.provincesSelectedVal[0].name+ ' '+ this.provincesSelectedVal[1].name+ ' '+ this.provincesSelectedVal[2].name;
				
				this.$emit('change', this.provincesSelectedVal);
				this.closeModel(1);
			}
		},
		//区选址
		area_selectedFun: function (e) {
			var Index= e.currentTarget.dataset.index,
				item= e.currentTarget.dataset.item;
			
			this.provincesSelectedIndex[2]= Index;
			this.provincesSelectedVal[2]= {
				code: item.code,
				name: item.name
			}
			this.selectedAreaStr= this.provincesSelectedVal[0].name+ ' '+ this.provincesSelectedVal[1].name+ ' '+ this.provincesSelectedVal[2].name;
			
			this.$emit('change', this.provincesSelectedVal);
			this.closeModel(1);
			this.$forceUpdate();
		},
		//省数据计算
		provincesArrFun: function () {
			this.provincesArr= this.pySegSort(this.cityData).slice();			
		},
		//市数据计算
		cityArrFun: function () {
			var cityArr= [];
			
			if(this.provincesSelectedIndex[0]!= -1){
				this.cityData[this.provincesSelectedIndex[0]].children.forEach((value, index, array)=> {
					cityArr.push({
						code: value.code,
						name: value.name
					})
				})
			}
			
			this.cityArr= cityArr.slice();
		},
		//区数据计算
		areaArrFun: function () {
			var areaArr= [];
			
			if(this.provincesSelectedIndex[0]!= -1 && this.provincesSelectedIndex[1]!= -1){
				this.cityData[this.provincesSelectedIndex[0]].children[this.provincesSelectedIndex[1]].children.forEach((value, index, array)=> {
					areaArr.push({
						code: value.code,
						name: value.name
					})
				})
			}
			
			this.areaArr= areaArr.slice();
		},
		//provincesSelectedIndex下标值计算
		provincesSelectedIndexFun: function () {
			var arr= [-1, -1, -1];  //省市区选中下标
			
			if(this.areaValueArr[0]!= ''){
				this.selectedAreaStr= '';
				
				this.cityData.forEach((value1, index1, array1)=> {
					if(value1.code== this.areaValueArr[0]){
						arr[0]= index1;
						this.selectedAreaStr+= value1.name+ ' ';
						this.provincesSelectedVal[0]= {
							code: value1.code,
							name: value1.name
						}
						//省数据选中下标计算
						this.provincesArr.forEach((value2, index2, array2)=> {
							value2.data.forEach((value3, index3, array3)=> {
								if(value3.code== this.areaValueArr[0]){
									this.provincesVal.pindex= index2;
									this.provincesVal.cindex= index3;
								}
							})
						})
						
						value1.children.forEach((value2, index2, array2)=> {
							if(value2.code== this.areaValueArr[1]){
								arr[1]= index2;
								this.selectedAreaStr+= value2.name+ ' ';
								this.provincesSelectedVal[1]= {
									code: value2.code,
									name: value2.name
								}
								
								this.provincesSelectedVal[2]= {
									code: '',
									name: ''
								}
								value2.children.forEach((value3, index3, array3)=> {
									if(value3.code== this.areaValueArr[2]){
										arr[2]= index3;
										this.selectedAreaStr+= value3.name;
										this.provincesSelectedVal[2]= {
											code: value3.code,
											name: value3.name
										}
									}
								})
							}
						})
					}
				})
				
				//区县下无数据时，例如广东-东沙群岛下无区县数据
				if(arr[2]== -1){
					this.tabIndex= 1;
					this.isShowArea= false;  //是否显示区的tab
				}else{
					this.tabIndex= 2;
					this.isShowArea= true;  //是否显示区的tab
				}
			}else{
				this.tabIndex= 0;
				this.isShowArea= true;
				this.provincesVal.pindex= -1;
				this.provincesVal.cindex= -1;
				this.provincesSelectedVal= [];
			}
			
			this.provincesSelectedIndex= arr;
		},
		//拼音按首字母排序
		pySegSort: function (arr) {
			var letters = 'abcdefghijklmnopqrstuvwxyz'.split('');
			var segs = [];
			
			letters.forEach((value1, index1, array1)=> {
				var curr = {letter: value1, data:[]};
				arr.forEach((value2, index2, array2)=> {
					var str= Pinyin.getSpell(value2.name, (charactor, spell)=> {
					    return spell[1];
					})
					if(str.charAt(0)== value1){
						curr.data.push({
							code: value2.code,
							name: value2.name
						})
					}
				})
				if(curr.data.length) {
					segs.push(curr);
				}
			})
			
			//字母分类
			var arr= [],
				splitIndex= 7;
			for(var i= 0; i< Math.ceil(letters.length/ splitIndex); i++){
				var keyOb = {letter: '', letterArr: [], data:[]},
					start= i* splitIndex,
					end= (i+ 1)* splitIndex;
					
				keyOb.letterArr.push(...letters.slice(start, end));
				keyOb.letter= (keyOb.letterArr[0]+ '-'+ keyOb.letterArr[keyOb.letterArr.length- 1]).toLocaleUpperCase();
				
				segs.forEach((value1, index1, array1)=> {
					if(keyOb.letterArr.indexOf(value1.letter)!= -1){
						keyOb.data.push(...value1.data)
					}
				})
				arr.push(keyOb)
			}
			
			return arr;			
		},
	}
}
</script>

<style lang="scss">
	.reginmiPicker{
		.picker-txt{
			display: block;
			line-height: 40rpx;
			padding: 28rpx 40rpx 28rpx 0;
			color: #1C2023;
			font-size: 28rpx;
			word-break: break-all;
			background: url('http://images.junhaoshun.cn/file/common_web/wechat/merchants/icon/icon_arrow_right.png') right center no-repeat;
			background-size: 18rpx 28rpx;
			&.disabled{
				color: #C3C8CC;
			}
		}
		.png{
			position: fixed;
			z-index: 100;
			top: 0;
			left: 0;
			display: none;
			width: 100%;
			height: 100%;
			background: rgba(0, 0, 0, 0.7);
			&.active{
				display: block;
			}
		}
	}
	.reginmiPicker-box{
		position: fixed;
		z-index: 101;
		bottom: 0;
		left: 0;
		width: 100%;
		height: 65%;
		background: #fff;
		transform: translate(0, 100%);
		&.active{
			transform: translate(0, 0);
		}
		.tab-box{
			position: absolute;
			z-index: 1;
			top: 0;
			left: 0;
			display: -webkit-box;
			display: -webkit-flex;
			display: flex;
			width: 100%;
			height: 100rpx;
			.li{
				position: relative;
				z-index: 0;
				flex: 1;
				height: 100rpx;
				line-height: 100rpx;
				text-align: center;
				padding: 0 6rpx;
				color: #969696;
				font-size: 28rpx;
				background: #f2f2f2;
				white-space: nowrap;
				text-overflow: ellipsis;
				overflow: hidden;
				&:after{
					content: '';
					position: absolute;
					z-index: 1;
					top: 0;
					right: 0;
					width: 1px;
					height: 100%;
					font-size: 0;
					background: #e7e7e7;
				}
				&:last-child{
					&::after{
						content: '';
						display: none;
					}
				}
				&.active{
					background: #fff;
				}
				&.selected{
					color: #20A478;
				}
			}
		}
		.op-box{
			position: absolute;
			z-index: 1;
			bottom: 0;
			left: 0;
			width: 100%;
			height: 100rpx;
			background: #fff;
			.btn-a{
				position: relative;
				z-index: 0;
				height: 100rpx;
				line-height: 100rpx;
				text-align: center;
				color: #0b0b0b;
				font-size: 36rpx;
				background: #f7f7f7;
			}
		}
		.scroller-box{
			position: absolute;
			z-index: 1;
			top: 100rpx;
			bottom: 100rpx;
			left: 0;
			width: 100%;
			overflow-y: auto;
			-webkit-overflow-scrolling: touch;
			.co-box{
				padding: 30rpx;
			}
			.list-box{
				position: relative;
				z-index: 0;
				padding: 16rpx 0;
				border-bottom: 1px solid #e9e9e9;
				&:last-child{
					border-bottom: 0;
				}
				.letter-box{
					position: absolute;
					z-index: 1;
					top: 50%;
					left: 0;
					margin-top: -20rpx;
					width: 100rpx;
					height: 40rpx;
					line-height: 40rpx;
					text-align: center;
					color: #828282;
					font-size: 22rpx;
				}
				.city-list{
					margin-left: 100rpx;
					&.no-letter-box{
						margin-left: 0;
					}
					.li{
						float: left;
						display: inline-block;
						height: 78rpx;
						line-height: 78rpx;
						padding: 0 22rpx;
						color: #0b0b0b;
						font-size: 28rpx;
						border-radius: 8rpx;
						&.active{
							color: #fff;
							background: #20A478;
						}
					}
				}
			}
		}
	}
</style>
