<template>
	<view class="content">
		<u-toast ref="uToast" style="top:70rpx"/>
		<u-form :model="form" ref="uForm">
			<u-form-item label="用户名" label-width="150rpx"><u-input v-model="form.username" placeholder="请输入用户名" :disabled="state" /></u-form-item>
			<u-form-item label="密码" label-width="150rpx"><u-input v-model="form.password" type="password" placeholder="请输入密码" :disabled="state" /></u-form-item>
			<u-form-item label="打卡接口" label-width="150rpx">
				<u-radio-group v-model="form.method" :disabled="state">
					<u-radio name="auto">自动选择</u-radio>
					<u-radio name="ehhu">E河海</u-radio>
					<u-radio name="myhhu">信息门户</u-radio>
				</u-radio-group>
			</u-form-item>
			<u-row gutter="16">
				<u-col span="6">
			<u-button @click="submit" type="primary">开始打卡</u-button>
			</u-col>
			<u-col span="6">
			<u-button @click="task">定时任务</u-button>
			</u-col>
			</u-row>
		</u-form>
		<view v-show="table != null">
		<center style="margin: 15rpx;">
		<view class="divider" style="margin-top: 30rpx;margin-bottom: 15rpx;"></view>
		<text >打卡结果</text>
		<u-table style="width: 90%;margin-top: 15rpx;">
				<u-tr>
					<u-th>项目</u-th>
					<u-th>内容</u-th>
				</u-tr>
				<u-tr v-for="(item,i) in table">
					<u-td :style="{height:(item.key.length<=15?'50rpx':'200rpx')}">{{item.key}}</u-td>
					<u-td :style="{height:(item.key.length<=15?'50rpx':'200rpx')}">{{item.data}}</u-td>
				</u-tr>
				
			</u-table>
		</center>
		</view>
	</view>
</template>

<script>
	import timer from '../../js_sdk/xbc-timer/timer.js'
	timer.start(false)
	import CryptoJS from "../../node_modules/crypto-js/crypto-js.js"
	function _gas(data, key0, iv0) {
		key0 = key0.replace(/(^\s+)|(\s+$)/g, "");
		var key = CryptoJS.enc.Utf8.parse(key0);
		var iv = CryptoJS.enc.Utf8.parse(iv0);
		var encrypted = CryptoJS.AES.encrypt(data, key, {
			iv: iv,
			mode: CryptoJS.mode.CBC,
			padding: CryptoJS.pad.Pkcs7
		});
		return encrypted.toString();
	}
	function encryptAES(data, _p1) {
		if (!_p1) {
			return data;
		}
		var encrypted = _gas(_rds(64) + data, _p1, _rds(16));
		return encrypted;
	}
	function _ep(p0, p1) {
		try {
			return encryptAES(p0, p1);
		} catch (e) {}
		return p0;
	}
	var $_chars = 'ABCDEFGHJKMNPQRSTWXYZabcdefhijkmnprstwxyz2345678';
	var _chars_len = $_chars.length;
	
	function _rds(len) {
		var i=0;
		var retStr = '';
		for (i = 0; i < len; i++) {
			retStr += $_chars.charAt(Math.floor(Math.random() * _chars_len));
		}
		return retStr;
	}
	
	Date.prototype.Format = function (fmt) {
	    var o = {
	        "M+": this.getMonth() + 1, //月份 
	        "d+": this.getDate(), //日 
	        "H+": this.getHours(), //小时 
	        "m+": this.getMinutes(), //分 
	        "s+": this.getSeconds(), //秒 
	        "q+": Math.floor((this.getMonth() + 3) / 3), //季度 
	        "S": this.getMilliseconds() //毫秒 
	    };
	    if (/(y+)/.test(fmt)) fmt = fmt.replace(RegExp.$1, (this.getFullYear() + "").substr(4 - RegExp.$1.length));
	    for (var k in o)
	    if (new RegExp("(" + k + ")").test(fmt)) fmt = fmt.replace(RegExp.$1, (RegExp.$1.length == 1) ? (o[k]) : (("00" + o[k]).substr(("" + o[k]).length)));
	    return fmt;
	}
	export default {
		data() {
			return {
				form: {
						username: uni.getStorageSync("username"),
						password: uni.getStorageSync("password"),
						method: uni.getStorageSync("method")
					},
				table:null,
				state:false,
			}
		},
		onLoad() {

		},
		methods: {
			task(){
				var username=uni.getStorageSync('username');
				var password=uni.getStorageSync('password');
				var method=uni.getStorageSync('method');
				var reg = /^[0-9]*$/;
				if(username == ""){
					this.$refs.uToast.show({
						title: '用户名为空',
						// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
						 type: 'error', 
						 position:'top'
						// 如果不需要图标，请设置为false
						// icon: false
					})

					return ;
				}
				if(password == ""){

					this.$refs.uToast.show({
						title: '密码为空',
						// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
						 type: 'error', 
						 position:'top'
						// 如果不需要图标，请设置为false
						// icon: false
					})
					
					return ;
				}
				if(method == ""){

					this.$refs.uToast.show({
						title: '打卡接口为空',
						// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
						 type: 'error', 
						 position:'top'
						// 如果不需要图标，请设置为false
						// icon: false
					})
					return
				}
				if(reg.test(password)){
					if(method == "myhhu"){
						uni.hideLoading()
						this.$refs.uToast.show({
							title: '弱密码无法使用信息门户打卡',
							// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
							 type: 'error', 
							 position:'top'
							// 如果不需要图标，请设置为false
							// icon: false
						})
						return ;
					}
				}
				timer.clear()
				timer.start(false)
				timer.chain(
				[
					{
					id:'daka0',
					func:this.dotask,
					time: '06:00:00'
					},
					{
					id:'daka1',
					func:this.dotask,
					time: '07:00:00'
					},
					{
					id:'daka2',
					func:this.dotask,
					time: '08:00:00'
					},
					{
					id:'daka3',
					func:this.dotask,
					time: '09:00:00'
					},
					{
					id:'daka4',
					func:this.dotask,
					time: '06:30:00'
					},
					{
					id:'daka5',
					func:this.dotask,
					time: '07:30:00'
					},
					{
					id:'daka6',
					func:this.dotask,
					time: '08:30:00'
					},
					{
					id:'daka7',
					func:this.dotask,
					time: '09:30:00'
					},
					{
					id:'daka8',
					func:this.dotask,
					time: '10:00:00'
					},
					{
					id:'daka9',
					func:this.dotask,
					time: '11:00:00'
					},
					{
					id:'daka10',
					func:this.dotask,
					time: '12:00:00'
					},
				]
				)
				this.$refs.uToast.show({
					title: '打卡任务创建成功！',
					// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
					 type: 'success', 
					 position:'top'
					// 如果不需要图标，请设置为false
					// icon: false
				})
				return ;
			},
			oldReport(cookieStr){
				let self = this
				uni.request({
					url:'http://form.hhu.edu.cn/pdc/form/list',
					header:{
						Cookie:cookieStr
					},
					success:function(res){
						if(res.data.indexOf("健康打卡") == -1){
							uni.hideLoading()
							self.$refs.uToast.show({
								title: '打卡网页异常',
								// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
								 type: 'error', 
								 position:'top'
								// 如果不需要图标，请设置为false
								// icon: false
							})
							var options = {cover:false};
							var str = "[定时任务] 打卡网页异常，打卡失败";
							plus.push.createMessage(str, "LocalMSG", options);
						}
						else{
							if(res.data.indexOf("本科生") != -1){
								uni.request({
									url:'http://form.hhu.edu.cn/pdc/formDesignApi/S/gUTwwojq',
									success:function(res){
										self.$refs.uToast.show({
											title: '开始读取打卡数据',
											// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
											 type: 'info', 
											 position:'top'
											// 如果不需要图标，请设置为false
											// icon: false
										})
										var pattern1=/<script type="text\/javascript">([\w\W]*)<\/script>/
										var javascript=pattern1.exec(res.data)[0]
										var wid = /(?<=_selfFormWid = \')(.*?)(?=\')/.exec(javascript)[0]
										var uid = /(?<=_userId = \')(.*?)(?=\')/.exec(javascript)[0]
										var fillDetail0=/(?<=fillDetail = )(.*?)(?=\;)/.exec(javascript)[0]
										var dataDetail0=/(?<=dataDetail = )(.*?)(?=\]\,)/.exec(javascript)[0]+"]"
										var fillDetail = eval('(' + fillDetail0 + ')');
										var dataDetail = eval('(' + dataDetail0 + ')');
										var result = Object.assign(dataDetail[0], fillDetail[0]);
										var postData={
											DATETIME_CYCLE:new Date().Format("yyyy/MM/dd")
										}
										var showData=[]
										var column = {
										            XGH_336526: "学号",
										            XM_1474: "姓名",
										            SFZJH_859173: "身份证号",
										            SELECT_941320: "学院",
										            SELECT_459666: "年级",
										            SELECT_814855: "专业",
										            SELECT_525884: "班级",
										            SELECT_125597: "宿舍楼",
										            TEXT_950231: "宿舍号",
										            TEXT_937296: "手机号",
										            RADIO_6555: "您的体温情况？",
										            RADIO_535015: "您今天是否在校？",
										            RADIO_891359: "本人健康情况？",
										            RADIO_372002: "同住人健康情况？",
										            RADIO_618691: "本人及同住人14天内是否有中高风险地区旅居史或接触过中高风险地区人员？"
										        }
										for(var key in column){
										  postData[key]=result[key]
										  showData.push({key:column[key],data:result[key]})
										}  
						
										console.log(postData)
										self.$refs.uToast.show({
											title: '打卡数据读取成功',
											// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
											 type: 'info', 
											 position:'top'
											// 如果不需要图标，请设置为false
											// icon: false
										})
										var postUrl = "http://form.hhu.edu.cn/pdc/formDesignApi/dataFormSave?wid=" + wid + "&userId=" + uid
										uni.request({
											url:postUrl,
											method:'POST',
											header: {
													 'Content-Type': 'application/x-www-form-urlencoded'  
													},
											data:postData,
											success:function(res){
												if(res.data.result == true){
													uni.hideLoading()
													self.$refs.uToast.show({
														title: '打卡成功',
														// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
														 type: 'success', 
														 position:'top'
														// 如果不需要图标，请设置为false
														// icon: false
													})
													var d =new Date()
													uni.setStorageSync('last',d.getFullYear().toString()+d.getMonth().toString()+d.getDate().toString());
													var options = {cover:false};
													var str = "[定时任务] 打卡成功";
													
													plus.push.createMessage(str, "LocalMSG", options);
													self.table = showData
												}
												else{
													uni.hideLoading()
													self.$refs.uToast.show({
														title: '打卡失败',
														// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
														 type: 'error', 
														 position:'top'
														// 如果不需要图标，请设置为false
														// icon: false
													})
													var options = {cover:false};
													var str = "[定时任务] 出现未知错误，打卡失败";
													plus.push.createMessage(str, "LocalMSG", options);
												}
											},
											fail:function(res){
												uni.hideLoading()
												self.$refs.uToast.show({
													title: '打卡失败',
													// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
													 type: 'error', 
													 position:'top'
													// 如果不需要图标，请设置为false
													// icon: false
												})
												var options = {cover:false};
												var str = "[定时任务] 出现未知错误，打卡失败";
												plus.push.createMessage(str, "LocalMSG", options);
											}
											})
									},
									fail:function(res){
										uni.hideLoading()
										self.$refs.uToast.show({
											title: '出现未知错误',
											// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
											 type: 'error', 
											 position:'top'
											// 如果不需要图标，请设置为false
											// icon: false
										})
										var options = {cover:false};
										var str = "[定时任务] 出现未知错误，打卡失败";
										plus.push.createMessage(str, "LocalMSG", options);
									},
									})
							}
							else if(res.data.indexOf("研究生") != -1){
								uni.request({
									url:'http://form.hhu.edu.cn/pdc/formDesignApi/S/xznuPIjG',
									success:function(res){
										self.$refs.uToast.show({
											title: '开始读取打卡数据',
											// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
											 type: 'info', 
											 position:'top'
											// 如果不需要图标，请设置为false
											// icon: false
										})
										var pattern1=/<script type="text\/javascript">([\w\W]*)<\/script>/
										var javascript=pattern1.exec(res.data)[0]
										var wid = /(?<=_selfFormWid = \')(.*?)(?=\')/.exec(javascript)[0]
										var uid = /(?<=_userId = \')(.*?)(?=\')/.exec(javascript)[0]
										var fillDetail0=/(?<=fillDetail = )(.*?)(?=\;)/.exec(javascript)[0]
										var dataDetail0=/(?<=dataDetail = )(.*?)(?=\]\,)/.exec(javascript)[0]+"]"
										var fillDetail = eval('(' + fillDetail0 + ')');
										var dataDetail = eval('(' + dataDetail0 + ')');
										var result = Object.assign(dataDetail[0], fillDetail[0]);
										var postData={
											DATETIME_CYCLE:new Date().Format("yyyy/MM/dd")
										}
										var showData=[]
										var column = {
										            "XGH_566872": "学号",
										                        "XM_140773": "姓名",
										                        "SFZJH_402404": "身份证号",
										                        "SZDW_439708": "学院",
										                        "ZY_878153": "专业",
										                        "GDXW_926421": "攻读学位",
										                        "DSNAME_606453":"导师",
										                        "PYLB_253720": "培养类别",
										                        "SELECT_172548": "宿舍楼",
										                        "TEXT_91454": "宿舍号",
										                        "TEXT_24613": "手机号",
										                        "TEXT_826040": "紧急联系人电话",
										                        "RADIO_799044": "您的体温情况？",
										                        "RADIO_384811": "您今天是否在校？",
										                        "RADIO_907280": "本人健康情况？",
										                        "RADIO_716001": "同住人健康情况？",
										                        "RADIO_248990": "本人及同住人14天内是否有中高风险地区旅居史或接触过中高风险地区人员？"
										        }
										for(var key in column){
										  postData[key]=result[key]
										  showData.push({key:column[key],data:result[key]})
										}  
										console.log(postData)
										self.$refs.uToast.show({
											title: '打卡数据读取成功',
											// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
											 type: 'info', 
											 position:'top'
											// 如果不需要图标，请设置为false
											// icon: false
										})
										var postUrl = "http://form.hhu.edu.cn/pdc/formDesignApi/dataFormSave?wid=" + wid + "&userId=" + uid
										uni.request({
											url:postUrl,
											method:'POST',
											header: {
													 'Content-Type': 'application/x-www-form-urlencoded'  
													},
											data:postData,
											success:function(res){
												if(res.data.result == true){
													uni.hideLoading()
													self.$refs.uToast.show({
														title: '打卡成功',
														// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
														 type: 'success', 
														 position:'top'
														// 如果不需要图标，请设置为false
														// icon: false
													})
													var d =new Date()
													uni.setStorageSync('last',d.getFullYear().toString()+d.getMonth().toString()+d.getDate().toString());
													var options = {cover:false};
													var str = "[定时任务] 打卡成功";
													plus.push.createMessage(str, "LocalMSG", options);
													self.table = showData
												}
												else{
													uni.hideLoading()
													self.$refs.uToast.show({
														title: '打卡失败',
														// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
														 type: 'error', 
														 position:'top'
														// 如果不需要图标，请设置为false
														// icon: false
													})
													var options = {cover:false};
													var str = "[定时任务] 出现未知错误，打卡失败";
													plus.push.createMessage(str, "LocalMSG", options);
												}
											},
											fail:function(res){
												uni.hideLoading()
												self.$refs.uToast.show({
													title: '打卡失败',
													// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
													 type: 'error', 
													 position:'top'
													// 如果不需要图标，请设置为false
													// icon: false
												})
												var options = {cover:false};
												var str = "[定时任务] 出现未知错误，打卡失败";
												plus.push.createMessage(str, "LocalMSG", options);
											}
											})
									},
									fail:function(res){
										uni.hideLoading()
										self.$refs.uToast.show({
											title: '出现未知错误',
											// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
											 type: 'error', 
											 position:'top'
											// 如果不需要图标，请设置为false
											// icon: false
										})
										var options = {cover:false};
										var str = "[定时任务] 出现未知错误，打卡失败";
										plus.push.createMessage(str, "LocalMSG", options);
									},
									})
							}
							else{
								uni.hideLoading()
								self.$refs.uToast.show({
									title: '身份识别异常',
									// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
									 type: 'error', 
									 position:'top'
									// 如果不需要图标，请设置为false
									// icon: false
								})
								var options = {cover:false};
								var str = "[定时任务] 身份识别异常，打卡失败";
								plus.push.createMessage(str, "LocalMSG", options);
							}
						}
					},
					fail:function(res){
						uni.hideLoading()
						self.$refs.uToast.show({
							title: '出现未知错误',
							// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
							 type: 'error', 
							 position:'top'
							// 如果不需要图标，请设置为false
							// icon: false
						})
						var options = {cover:false};
						var str = "[定时任务] 出现未知错误，打卡失败";
						plus.push.createMessage(str, "LocalMSG", options);
					},
				})
			},
			newReport(){
				let self = this
				uni.request({
					url:'http://dailyreport.hhu.edu.cn/pdc/form/list',
					success:function(res){
						if(res.data.indexOf("健康打卡") == -1){
							uni.hideLoading()
							self.$refs.uToast.show({
								title: '打卡网页异常',
								// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
								 type: 'error', 
								 position:'top'
								// 如果不需要图标，请设置为false
								// icon: false
							})
							var options = {cover:false};
							var str = "[定时任务] 打卡网页异常，打卡失败";
							plus.push.createMessage(str, "LocalMSG", options);
						}
						else{
							if(res.data.indexOf("本科生") != -1){
								uni.request({
									url:'http://dailyreport.hhu.edu.cn/pdc/formDesignApi/S/gUTwwojq',
									success:function(res){
										self.$refs.uToast.show({
											title: '开始读取打卡数据',
											// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
											 type: 'info', 
											 position:'top'
											// 如果不需要图标，请设置为false
											// icon: false
										})
										var pattern1=/<script type="text\/javascript">([\w\W]*)<\/script>/
										var javascript=pattern1.exec(res.data)[0]
										var wid = /(?<=_selfFormWid = \')(.*?)(?=\')/.exec(javascript)[0]
										var uid = /(?<=_userId = \')(.*?)(?=\')/.exec(javascript)[0]
										var fillDetail0=/(?<=fillDetail = )(.*?)(?=\;)/.exec(javascript)[0]
										var dataDetail0=/(?<=dataDetail = )(.*?)(?=\]\,)/.exec(javascript)[0]+"]"
										var fillDetail = eval('(' + fillDetail0 + ')');
										var dataDetail = eval('(' + dataDetail0 + ')');
										var result = Object.assign(dataDetail[0], fillDetail[0]);
										var postData={
											DATETIME_CYCLE:new Date().Format("yyyy/MM/dd")
										}
										var showData=[]
										var column = {
										            XGH_336526: "学号",
										            XM_1474: "姓名",
										            SFZJH_859173: "身份证号",
										            SELECT_941320: "学院",
										            SELECT_459666: "年级",
										            SELECT_814855: "专业",
										            SELECT_525884: "班级",
										            SELECT_125597: "宿舍楼",
										            TEXT_950231: "宿舍号",
										            TEXT_937296: "手机号",
										            RADIO_6555: "您的体温情况？",
										            RADIO_535015: "您今天是否在校？",
										            RADIO_891359: "本人健康情况？",
										            RADIO_372002: "同住人健康情况？",
										            RADIO_618691: "本人及同住人14天内是否有中高风险地区旅居史或接触过中高风险地区人员？"
										        }
										for(var key in column){
										  postData[key]=result[key]
										  showData.push({key:column[key],data:result[key]})
										}  
										console.log(postData)
										self.$refs.uToast.show({
											title: '打卡数据读取成功',
											// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
											 type: 'info', 
											 position:'top'
											// 如果不需要图标，请设置为false
											// icon: false
										})
										var postUrl = "http://dailyreport.hhu.edu.cn/pdc/formDesignApi/dataFormSave?wid=" + wid + "&userId=" + uid
										uni.request({
											url:postUrl,
											method:'POST',
											header: {
													 'Content-Type': 'application/x-www-form-urlencoded'  
													},
											data:postData,
											success:function(res){
												if(res.data.result == true){
													uni.hideLoading()
													self.$refs.uToast.show({
														title: '打卡成功',
														// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
														 type: 'success', 
														 position:'top'
														// 如果不需要图标，请设置为false
														// icon: false
													})
													var d =new Date()
													uni.setStorageSync('last',d.getFullYear().toString()+d.getMonth().toString()+d.getDate().toString());
													var options = {cover:false};
													var str = "[定时任务] 打卡成功";
													plus.push.createMessage(str, "LocalMSG", options);
													self.table = showData
												}
												else{
													uni.hideLoading()
													self.$refs.uToast.show({
														title: '打卡失败',
														// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
														 type: 'error', 
														 position:'top'
														// 如果不需要图标，请设置为false
														// icon: false
													})
													var options = {cover:false};
													var str = "[定时任务] 出现未知错误，打卡失败";
													plus.push.createMessage(str, "LocalMSG", options);
												}
											},
											fail:function(res){
												uni.hideLoading()
												self.$refs.uToast.show({
													title: '打卡失败',
													// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
													 type: 'error', 
													 position:'top'
													// 如果不需要图标，请设置为false
													// icon: false
												})
												var options = {cover:false};
												var str = "[定时任务] 出现未知错误，打卡失败";
												plus.push.createMessage(str, "LocalMSG", options);
											}
											})
									},
									fail:function(res){
										uni.hideLoading()
										self.$refs.uToast.show({
											title: '出现未知错误',
											// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
											 type: 'error', 
											 position:'top'
											// 如果不需要图标，请设置为false
											// icon: false
										})
										var options = {cover:false};
										var str = "[定时任务] 出现未知错误，打卡失败";
										plus.push.createMessage(str, "LocalMSG", options);
									},
									})
							}
							else if(res.data.indexOf("研究生") != -1){
								uni.request({
									url:'http://dailyreport.hhu.edu.cn/pdc/formDesignApi/S/xznuPIjG',
									success:function(res){
										self.$refs.uToast.show({
											title: '开始读取打卡数据',
											// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
											 type: 'info', 
											 position:'top'
											// 如果不需要图标，请设置为false
											// icon: false
										})
										var pattern1=/<script type="text\/javascript">([\w\W]*)<\/script>/
										var javascript=pattern1.exec(res.data)[0]
										var wid = /(?<=_selfFormWid = \')(.*?)(?=\')/.exec(javascript)[0]
										var uid = /(?<=_userId = \')(.*?)(?=\')/.exec(javascript)[0]
										var fillDetail0=/(?<=fillDetail = )(.*?)(?=\;)/.exec(javascript)[0]
										var dataDetail0=/(?<=dataDetail = )(.*?)(?=\]\,)/.exec(javascript)[0]+"]"
										var fillDetail = eval('(' + fillDetail0 + ')');
										var dataDetail = eval('(' + dataDetail0 + ')');
										var result = Object.assign(dataDetail[0], fillDetail[0]);
										var postData={
											DATETIME_CYCLE:new Date().Format("yyyy/MM/dd")
										}
										var showData=[]
										var column = {
										            "XGH_566872": "学号",
										                        "XM_140773": "姓名",
										                        "SFZJH_402404": "身份证号",
										                        "SZDW_439708": "学院",
										                        "ZY_878153": "专业",
										                        "GDXW_926421": "攻读学位",
										                        "DSNAME_606453":"导师",
										                        "PYLB_253720": "培养类别",
										                        "SELECT_172548": "宿舍楼",
										                        "TEXT_91454": "宿舍号",
										                        "TEXT_24613": "手机号",
										                        "TEXT_826040": "紧急联系人电话",
										                        "RADIO_799044": "您的体温情况？",
										                        "RADIO_384811": "您今天是否在校？",
										                        "RADIO_907280": "本人健康情况？",
										                        "RADIO_716001": "同住人健康情况？",
										                        "RADIO_248990": "本人及同住人14天内是否有中高风险地区旅居史或接触过中高风险地区人员？"
										        }
										for(var key in column){
										  postData[key]=result[key]
										  showData.push({key:column[key],data:result[key]})
										}  
										console.log(postData)
										self.$refs.uToast.show({
											title: '打卡数据读取成功',
											// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
											 type: 'info', 
											 position:'top'
											// 如果不需要图标，请设置为false
											// icon: false
										})
										var postUrl = "http://dailyreport.hhu.edu.cn/pdc/formDesignApi/dataFormSave?wid=" + wid + "&userId=" + uid
										uni.request({
											url:postUrl,
											method:'POST',
											header: {
													 'Content-Type': 'application/x-www-form-urlencoded'  
													},
											data:postData,
											success:function(res){
												if(res.data.result == true){
													uni.hideLoading()
													self.$refs.uToast.show({
														title: '打卡成功',
														// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
														 type: 'success', 
														 position:'top'
														// 如果不需要图标，请设置为false
														// icon: false
													})
													var d =new Date()
													uni.setStorageSync('last',d.getFullYear().toString()+d.getMonth().toString()+d.getDate().toString());
													var options = {cover:false};
													var str = "[定时任务] 今日打卡成功";
													plus.push.createMessage(str, "LocalMSG", options);
													self.table = showData
												}
												else{
													uni.hideLoading()
													self.$refs.uToast.show({
														title: '打卡失败',
														// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
														 type: 'error', 
														 position:'top'
														// 如果不需要图标，请设置为false
														// icon: false
													})
													var options = {cover:false};
													var str = "[定时任务] 出现未知错误，打卡失败";
													plus.push.createMessage(str, "LocalMSG", options);
												}
											},
											fail:function(res){
												uni.hideLoading()
												self.$refs.uToast.show({
													title: '打卡失败',
													// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
													 type: 'error', 
													 position:'top'
													// 如果不需要图标，请设置为false
													// icon: false
												})
												var options = {cover:false};
												var str = "[定时任务] 出现未知错误，打卡失败";
												plus.push.createMessage(str, "LocalMSG", options);
											}
											})
									},
									fail:function(res){
										uni.hideLoading()
										self.$refs.uToast.show({
											title: '出现未知错误',
											// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
											 type: 'error', 
											 position:'top'
											// 如果不需要图标，请设置为false
											// icon: false
										})
										var options = {cover:false};
										var str = "[定时任务] 出现未知错误，打卡失败";
										plus.push.createMessage(str, "LocalMSG", options);
									},
									})
							}
							else{
								uni.hideLoading()
								self.$refs.uToast.show({
									title: '身份识别异常',
									// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
									 type: 'error', 
									 position:'top'
									// 如果不需要图标，请设置为false
									// icon: false
								})
								var options = {cover:false};
								var str = "[定时任务] 身份识别异常，打卡失败";
								plus.push.createMessage(str, "LocalMSG", options);
							}
						}
					},
					fail:function(res){
						uni.hideLoading()
						self.$refs.uToast.show({
							title: '出现未知错误',
							// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
							 type: 'error', 
							 position:'top'
							// 如果不需要图标，请设置为false
							// icon: false
						})
						var options = {cover:false};
						var str = "[定时任务] 出现未知错误，打卡失败";
						plus.push.createMessage(str, "LocalMSG", options);
					},
				})
			},
			old(username,password){
				let self = this 
				uni.request({
				   url: 'http://mids.hhu.edu.cn/_ids_mobile/login18_9',
				   method:'POST',
				   header: {
				   		 'Content-Type': 'application/x-www-form-urlencoded'  
				   		},
				   data:{
				   	username:username,
				   	password:password,
		
				   },
				   success:function(res){
					   if(res.header.hasOwnProperty("loginErrCode")){
						   uni.hideLoading()
						   self.$refs.uToast.show({
						   	title: 'E河海登录失败',
						   	// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
						   	 type: 'error', 
						   	 position:'top'
						   	// 如果不需要图标，请设置为false
						   	// icon: false
						   })
						   var options = {cover:false};
						   var str = "[定时任务] E河海登陆失败，打卡失败";
						   plus.push.createMessage(str, "LocalMSG", options);
					   }
					   else{
						   var cookieArr=eval("("+res.header.ssoCookie+")")
						   var cookieStr=""
						   for(var key in cookieArr){
						     cookieStr+=cookieArr[key]["cookieName"]+"="+cookieArr[key]["cookieValue"]+";"
						   }
						   self.$refs.uToast.show({
						   	title: 'E河海登录成功',
						   	// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
						   	 type: 'info', 
						   	 position:'top'
						   	// 如果不需要图标，请设置为false
						   	// icon: false
						   })
						   self.oldReport(cookieStr)
					   }
					   
				   },
				   fail:function(res){
					   uni.hideLoading()
					   self.$refs.uToast.show({
					   	title: '出现未知错误',
					   	// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
					   	 type: 'error', 
					   	 position:'top'
					   	// 如果不需要图标，请设置为false
					   	// icon: false
					   })
					   var options = {cover:false};
					   var str = "[定时任务] 出现未知错误，打卡失败";
					   plus.push.createMessage(str, "LocalMSG", options);
				   }
				   })
			},
			new(username,password){
				let self = this
				 uni.request({
				    url: 'http://authserver.hhu.edu.cn/authserver/login',
					header:{
						Cookie:""
					},
					fail: function(res){
						uni.hideLoading()
						self.$refs.uToast.show({
							title: '出现未知错误',
							// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
							 type: 'error', 
							 position:'top'
							// 如果不需要图标，请设置为false
							// icon: false
						})
						var options = {cover:false};
						var str = "[定时任务] 出现未知错误，打卡失败";
						plus.push.createMessage(str, "LocalMSG", options);
					},
				    success: function(res){
						
						var pattern = /(?<=pwdDefaultEncryptSalt = ").+(?=";)/;
						var pattern1 =/(?<=<input type="hidden" name="lt" value=").+(?="\/>)/;
						var pattern2 =/(?<=<input type="hidden" name="execution" value=").+(?="\/>)/;
				        console.log(pattern.exec(res.data)[0]);
						console.log(encryptAES(password,pattern.exec(res.data)[0]))
						uni.request({
						    url: 'http://authserver.hhu.edu.cn/authserver/login',
							method:'POST',
							header: {
									 'Content-Type': 'application/x-www-form-urlencoded'  
									},
							data:{
								username:username,
								password:encryptAES(password,pattern.exec(res.data)[0]),
								lt:pattern1.exec(res.data)[0],
								execution:pattern2.exec(res.data)[0],
								dllt:"userNamePasswordLogin",
								_eventId:"submit",
								rmShown:"1"
							},
						    success: function(res){
						        if(res.data.indexOf("个人设置") != -1 ||res.data.indexOf("User setting") != -1){
									self.$refs.uToast.show({
										title: '信息门户登录成功',
										// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
										 type: 'info', 
										 position:'top'
										// 如果不需要图标，请设置为false
										// icon: false
									})
									self.newReport()
								}
								else{
									self.$refs.uToast.show({
										title: '信息门户登录失败',
										// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
										 type: 'error', 
										 position:'top'
										// 如果不需要图标，请设置为false
										// icon: false
									})
									uni.hideLoading()
									var options = {cover:false};
									var str = "[定时任务] 信息门户登陆失败，打卡失败";
									plus.push.createMessage(str, "LocalMSG", options);
								}
						    }
						});
				    }
				});
			},
			submit(){
				var d =new Date()
				if (d.getFullYear().toString()+d.getMonth().toString()+d.getDate().toString() == uni.getStorageSync("last")){
					this.$refs.uToast.show({
						title: '今日已打卡，无需重复打卡',
						// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
						 type: 'default', 
						 position:'top'
						// 如果不需要图标，请设置为false
						// icon: false
					})
					return
				}
				uni.setStorageSync('username', this.form.username);
				uni.setStorageSync('password', this.form.password);
				uni.setStorageSync('method', this.form.method);
				this.table=null
				let self = this
				uni.showLoading({title: '打卡中',mask:true});
				var reg = /^[0-9]*$/;
				if(this.form.username == ""){
					uni.hideLoading()
					this.$refs.uToast.show({
						title: '用户名为空',
						// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
						 type: 'error', 
						 position:'top'
						// 如果不需要图标，请设置为false
						// icon: false
					})
					return ;
				}
				if(this.form.password == ""){
					uni.hideLoading()
					this.$refs.uToast.show({
						title: '密码为空',
						// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
						 type: 'error', 
						 position:'top'
						// 如果不需要图标，请设置为false
						// icon: false
					})
					return ;
				}
				if(this.form.method == ""){
					uni.hideLoading()
					this.$refs.uToast.show({
						title: '打卡接口为空',
						// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
						 type: 'error', 
						 position:'top'
						// 如果不需要图标，请设置为false
						// icon: false
					})
					return ;
				}
				if(reg.test(this.form.password)){
					if(this.form.method == "myhhu"){
						uni.hideLoading()
						this.$refs.uToast.show({
							title: '弱密码无法使用信息门户打卡',
							// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
							 type: 'error', 
							 position:'top'
							// 如果不需要图标，请设置为false
							// icon: false
						})
						return ;
					}
					if(this.form.method == "auto"){
						this.$refs.uToast.show({
							title: '检查到弱密码，使用E河海接口',
							// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
							 type: 'info', 
							 position:'top'
							// 如果不需要图标，请设置为false
							// icon: false
						})
					}
					this.old(this.form.username,this.form.password)
					
				}
				else{
					if(this.form.method == "ehhu"){
						this.old(this.form.username,this.form.password)
					}
					else{
						uni.request({
							url:"http://authserver.hhu.edu.cn/authserver/needCaptcha.html?username=" + this.form.username + "&pwdEncrypt2=pwdEncryptSalt",
							success:function(res){
								if(res.data == "true"){
									self.$refs.uToast.show({
										title: '需要验证码，使用E河海接口',
										// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
										 type: 'info', 
										 position:'top'
										// 如果不需要图标，请设置为false
										// icon: false
									})
									self.old(self.form.username,self.form.password)
								}
								else{
									self.new(self.form.username,self.form.password)
								}
							},
							fail:function(res){
								uni.hideLoading()
								self.$refs.uToast.show({
									title: '系统异常',
									// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
									 type: 'error', 
									 position:'top'
									// 如果不需要图标，请设置为false
									// icon: false
								})
								return ;
							}
						})
						
					}
				}
			},
		dotask(){
				var d =new Date()
				if (d.getFullYear().toString()+d.getMonth().toString()+d.getDate().toString() == uni.getStorageSync("last")){
					return
				}
				var options = {cover:false};
				var str = "[定时任务] 开始打卡";
				plus.push.createMessage(str, "LocalMSG", options);
				
				var username=uni.getStorageSync('username');
				var password=uni.getStorageSync('password');
				var method=uni.getStorageSync('method');
				this.table=null
				let self = this
				uni.showLoading({title: '打卡中',mask:true});
				var reg = /^[0-9]*$/;
				if(username == ""){
					uni.hideLoading()
					this.$refs.uToast.show({
						title: '用户名为空',
						// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
						 type: 'error', 
						 position:'top'
						// 如果不需要图标，请设置为false
						// icon: false
					})
					var options = {cover:false};
					var str = "[定时任务] 用户名为空";
					plus.push.createMessage(str, "LocalMSG", options);
					return ;
				}
				if(password == ""){
					uni.hideLoading()
					this.$refs.uToast.show({
						title: '密码为空',
						// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
						 type: 'error', 
						 position:'top'
						// 如果不需要图标，请设置为false
						// icon: false
					})
					var options = {cover:false};
					var str = "[定时任务] 密码为空";
					plus.push.createMessage(str, "LocalMSG", options);
					return ;
				}
				if(method == ""){
					uni.hideLoading()
					this.$refs.uToast.show({
						title: '打卡接口为空',
						// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
						 type: 'error', 
						 position:'top'
						// 如果不需要图标，请设置为false
						// icon: false
					})
					var options = {cover:false};
					var str = "[定时任务] 打卡接口为空";
					plus.push.createMessage(str, "LocalMSG", options);
					return ;
				}
				if(reg.test(password)){
					if(method == "myhhu"){
						uni.hideLoading()
						this.$refs.uToast.show({
							title: '弱密码无法使用信息门户打卡',
							// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
							 type: 'error', 
							 position:'top'
							// 如果不需要图标，请设置为false
							// icon: false
						})
						return ;
					}
					if(method == "auto"){
						this.$refs.uToast.show({
							title: '检查到弱密码，使用E河海接口',
							// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
							 type: 'info', 
							 position:'top'
							// 如果不需要图标，请设置为false
							// icon: false
						})
					}
					this.old(username,password)
					
				}
				else{
					if(method == "ehhu"){
						self.old(username,password)
					}
					else{
						uni.request({
							url:"http://authserver.hhu.edu.cn/authserver/needCaptcha.html?username=" + username + "&pwdEncrypt2=pwdEncryptSalt",
							success:function(res){
								if(res.data == "true"){
									self.$refs.uToast.show({
										title: '需要验证码，使用E河海接口',
										// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
										 type: 'info', 
										 position:'top'
										// 如果不需要图标，请设置为false
										// icon: false
									})
									self.old(username,password)
								}
								else{
									self.new(username,password)
								}
							},
							fail:function(res){
								uni.hideLoading()
								self.$refs.uToast.show({
									title: '系统异常',
									// 如果不传此type参数，默认为default，也可以手动写上 type: 'default'
									 type: 'error', 
									 position:'top'
									// 如果不需要图标，请设置为false
									// icon: false
								})
								var options = {cover:false};
								var str = "[定时任务] 系统异常，打卡失败";
								plus.push.createMessage(str, "LocalMSG", options);
								return ;
							}
						})
						
					}
				}
			}
		},
	}
</script>

<style>
	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}
	.divider{
		 background: #E0E3DA;
		 width: 90%;
		 height: 1rpx;
		}
</style>
