<template>
	<view class="reginmiPicker">
		<view :class="['picker-txt', select_value== ''? 'disabled': '']" @click="showFun">{{ cityStr }}</view>
		
		<view hover-stop-propagation :class="['png', isShow? 'active': '']" @click="hideFun"></view>
		
		<view :class="['reginmiPicker-box', isShow? 'active': '']">
			
			<view :class="['tab-box', is_boxShadow? 'boxShadow': '']">
				<view :class="['li', tabIndex== index? 'active': '', cityList_index[index]!= undefined? 'selected': '']" v-for="(item, index) in tabList" :key="index" @click="tabChange(index)">{{ tabStr(item, index) }}</view>
			</view>
			
			<view class="scroller-box">
				
				<div v-for="(item, index) in cityList" :key="index">
					
					<view class="co-box" v-if="index== 0" v-show="tabIndex== 0">
						<view class="list-box" v-for="(item1, index1) in item" :key="index1">
							
							<view class="letter-box">{{ item1.letter }}</view>
							<view class="city-list fix">
								<view :class="['li', (provinces_p=== index1 && provinces_c=== index2)? 'active': '']" v-for="(item2, index2) in item1.data" :key="index2" @click="provinces_selected_fun(index1, index2)">{{ item2.name }}</view>
							</view>
							
						</view>
					</view>
					
					<view class="co-box" v-if="index!= 0" v-show="tabIndex== index">
						<view class="list-box">
							<view class="city-list no-letter-box fix">
								<view :class="['li', cityList_index[index]== index1? 'active': '']" v-for="(item1, index1) in item" :key="index1" @click="city_selected_fun(index, index1)">{{ item1.name }}</view>
							</view>
							
						</view>
						
					</view>
					
				</div>
				
				
			</view>
			
			<view class="op-box">
				<view class="btn-a" @click="hideFun">取消</view>
			</view>
			
		</view>
	</view>
	
</template>

<script>
import Pinyin from '@/utils/pinyin.js'

export default {
	props: {
		//选中项绑定值
		value: {
			type: [String, Array],
			default: ''
		},
		//省市区数据
		options: {
			type: Array,
			default: []
		},
		placeholder: {
			type: String,
			default: '请选择区域'
		},
		//关闭前的回调，会暂停 Dialog 的关闭
		beforeClose: {
			type: Function,
			default: null
		},
		list: {
			type: Array,
			default: []
		}
	},
	data() {
		return {
			cityList: [],  //省市区选中对应数据列表
			cityList_index: [],  //省市区选中下标
			provinces_list: [],  //省数据
			provinces_p: null,  //省数据，父节点
			provinces_c: null,  //省数据，子节点
			provinces_code: '',  //省数据，code
			tabList: [],  //tab列表
			tabIndex: 0,  //tab索引
			isShow: false,  //是否显示弹窗
			select_value: '',  //省市区选中值
		}
	},
	watch: {
		value: function (newVal, oldVal) {
			console.log('value')
			this.select_value= newVal;
			this.init()
		},
		options: function (newVal, oldVal) {
			console.log('options')
			this.init()
		},
	},
	computed: {
		is_boxShadow: function () {
			return this.tabList.length== 1? true: false
		},
		tabStr: function (item, Index) {
			return (item, Index)=> {
				var str= item;
				
				if(this.cityList_index[Index]!= undefined){
					if(Index== 0){
						//第一级，省
						str= this.cityList[Index][this.provinces_p].data[this.provinces_c].name;
					}else{
						str= this.cityList[Index][this.cityList_index[Index]].name;
					}
				}
				
				return str
			}
		},
		cityStr: function () {
			var str= '';
			
			if(this.select_value){
				var level= 0;  //多维数组，第几层
					
				var recursive= (list)=> {
					if(level< this.select_value.length){
						list.forEach((value, index, array)=> {
							if(value.code== this.select_value[level]){
								str+= value.name+ ' ';
								level++
								recursive(value.children)
							}
						})
					}
				}
				
				recursive(this.options)
			}
			
			
			return str || this.placeholder
		},
	},
	mounted() {
		console.log('mounted')
		
		this.init()
	},
	methods: {
		//初始化
		init: function () {
			this.select_value= this.value;
			this.tabList= this.list.slice();
			
			if(this.options.length> 0){
				if(this.provinces_list.length== 0){
					this.provinces_list= this.pySegSort(this.options);
				}
				this.cityList= [this.provinces_list];
				this.cityList_index= [];
				
				if(this.select_value){
					this.computed_cityList_index_fun()
				}
			}
			
			console.log(this.cityList, this.cityList_index)
		},
		//展示
		showFun: function () {
			console.log('showFun')
			
			this.isShow= true;
			this.init()
			this.$emit('show')
		},
		//隐藏
		hideFun: function () {
			console.log('hideFun')
			
			if(this.beforeClose){
				this.beforeClose(()=> {
					this.isShow= false;
					this.$emit('close')
				})
			}else{
				this.isShow= false;
				this.$emit('close')
			}
		},
		//确定
		confirmFun: function () {
			var arr= [],  //选中值code
				level= 0;  //多维数组，第几层
		
			var recursive= (list)=> {
				if(level< this.cityList_index.length){
					list.forEach((value, index, array)=> {
						if(index== this.cityList_index[level]){
							arr.push(value.code)
							level++
							recursive(value.children)
						}
					})
				}
			}
			
			recursive(this.options)
			this.$emit('confirm', arr)
			this.hideFun()
		},
		//计算cityList_index值
		computed_cityList_index_fun: function () {
			var level= 0;  //多维数组，第几层
			
			var recursive= (list)=> {
				if(level< this.select_value.length){
					this.tabIndex= level;
					
					if(level== 0){
						this.provinces_list.forEach((value1, index1, array1)=> {
							value1.data.forEach((value2, index2, array2)=> {
								if(value2.code== this.select_value[level]){
									this.provinces_p= index1;
									this.provinces_c= index2;
								}
							})
						})
					}
					
					list.forEach((value, index, array)=> {
						if(value.code== this.select_value[level]){
							if(value.children && value.children.length> 0){
								this.cityList.push(value.children)
							}
							this.cityList_index.push(index)
							level++
							recursive(value.children)
						}
					})
				}
			}
			
			recursive(this.options)
		},
		//tab
		tabChange: function (Index) {
			if(Index== 0){
				this.tabIndex= Index;
			}else{
				if(this.cityList_index[Index- 1]== undefined){
					uni.showModal({
						title: '温馨提示',
						content: `请先选择${ this.tabList[Index- 1] }`,
						showCancel: false
					})
				} else if (this.cityList[Index]== undefined){
					return
				}else{
					this.tabIndex= Index;
				}
			}
		},
		//省选址
		provinces_selected_fun: function (pindex, cindex) {
			var code= this.cityList[0][pindex].data[cindex].code;
			this.provinces_p= pindex;
			this.provinces_c= cindex;
			
			this.options.forEach((value, index, array)=> {
				if(value.code== code){
					//是否有下一级children
					if(value.children && value.children.length> 0){
						this.cityList.splice(1, this.cityList.length, value.children)
						this.cityList_index.splice(0, this.cityList_index.length, index)
						this.tabIndex= 1;
					}else{
						this.cityList.splice(1, this.cityList.length)
						this.cityList_index.splice(0, this.cityList_index.length, index)
						this.confirmFun()
					}
				}
			})
		},
		//市、区、街选址
		city_selected_fun: function (tabIndex, Index) {
			var level= 0;  //多维数组，第几层
				
			this.cityList_index.splice(tabIndex, this.cityList_index.length, Index)
			
			//数组遍历、递归
			var recursive= (list)=> {
				if(level== tabIndex){
					list.forEach((value, index, array)=> {
						if(index== Index){
							if(value.children && value.children.length> 0){
								this.cityList.splice(level+ 1, this.cityList.length, value.children)
								this.tabIndex= level+ 1;
							}else{
								//没有下一级children
								this.cityList.splice(level+ 1, this.cityList.length)
								this.confirmFun()
							}
						}
					})
				}else{
					list.forEach((value, index, array)=> {
						if(index== this.cityList_index[level]){
							level++
							recursive(value.children)
						}
					})
				}
			}
			
			recursive(this.options)
		},
		//拼音按首字母排序
		pySegSort: function (list) {
			var letters = 'abcdefghijklmnopqrstuvwxyz'.split('');
			var segs = [];
			
			letters.forEach((value1, index1, array1)=> {
				var curr = {letter: value1, data:[]};
				list.forEach((value2, index2, array2)=> {
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
				var keyOb = { letter: '', letterArr: [], data: [] },
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
			&.boxShadow{
				box-shadow: 0 3rpx 3rpx #e9e9e9;
				.li{
					max-width: 200rpx;
				}
			}
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
