<script>
	export default {
		onLaunch: function() {

			// #ifdef MP
			// 获取小程序更新机制兼容
			if (uni.canIUse('getUpdateManager')) {
				const updateManager = uni.getUpdateManager();
				updateManager.onCheckForUpdate(function(res) {
					uni.setStorageSync('scene', wx.getLaunchOptionsSync().scene)
					// 请求完新版本信息的回调
					if (res.hasUpdate) {
						updateManager.onUpdateReady(function() {
							// updateManager.applyUpdate()
							uni.showModal({
								title: '更新提示',
								content: '新版本已经准备好，是否重启应用？',
								success: function(res) {
									if (res.confirm) {
										// 新的版本已经下载好，调用 applyUpdate 应用新版本并重启
										updateManager.applyUpdate()
									}
								}
							})
						})
						updateManager.onUpdateFailed(function() {
							// 新的版本下载失败
							uni.showModal({
								title: '已经有新版本了哟~',
								content: '新版本已经上线啦~，请您删除当前小程序，重新搜索打开哟~',
							})
						})
					}
				})
			} else {
				// 如果希望用户在最新版本的客户端上体验您的小程序，可以这样子提示
				uni.showModal({
					title: '提示',
					content: '当前微信版本过低，无法使用该功能，请升级到最新微信版本后重试。'
				})
			}
			//获取系统信息
			uni.getSystemInfo({
				success: (res)=> {
					console.log(res)
					
					//判断是否iphone x
					var arr= ['iPhone X', 'iPhone XR', 'iPhone XS', 'iPhone 11', 'iPhone12'],
						val= 0;
					arr.forEach((value, index, array)=> {
						if(res.model.search(value)!= -1){
							val= 1
						}
					})
					
					//判断运行平台
					var platform= '';
					if(res.platform == "ios"){
						platform= 'ios';
					}else if(res.platform == "android"){
						platform= 'android';
					}
					
					uni.setStorageSync('isIphonex', val)
					uni.setStorageSync('platform', platform)
				}
			})
			// #endif
		},
		onShow: function() {
		},
		onHide: function() {
		},
		globalData: {
		}
	}
</script>

<style lang="scss">
	/* 引入全局样式 */
	@import '@/styles/index.scss';
	/*每个页面公共css */
</style>
