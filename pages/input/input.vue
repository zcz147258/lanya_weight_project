<template>
	<view class="person">
		<view class="center">
			<view class="custom">
				<view class="title">出生日期：</view>
				<view>
					 <view class="uni-list-cell-db" style="width: 100%;">
						 <picker mode="date" :value="date" :start="startDate" :end="endDate" @change="bindDateChange">
							 <view >{{ date?date:'请选择日期' }}</view>
						 </picker>
					 </view>
				</view>
			</view>
			<view class="custom">
				<view class="title">身高(cm)：</view>
				<view>
					<input type="number"  placeholder="请输入" v-model="height">
				</view>
			</view>
			<view class="custom">
				<view class="title">性别：</view>
				<view>
					<picker @change="bindPickerChange" style="width: 100%;" :value="multiIndex" :range="multiArray">
						<view class="uni-input">{{ multiArray[multiIndex]}} </view>
					</picker>
				</view>
				
			</view>
			<view class="custom">
				<view>目标体重(kg)：</view>
				<view>
					<input type="number" placeholder="请输入" v-model="targetWeight">
				</view>
			</view>
		</view>
		<!-- 固定提交 -->
		<view class="fixed">
			<view @click="Submit">保存信息</view>
		</view>
	</view>		
</template>

<script>
	export default {
		data() {
			const currentDate = this.getDate({
				format: true
			})
			return {
				date: '请选择日期',
				targetWeight:"",
				height:"",
				multiIndex:0,//男女选中的index
				multiArray:['男','女'],
			}
		},
		computed: {
			startDate() {
				return this.getDate('start');
			},
			endDate() {
				return this.getDate('end');
			}
		},
		methods: {
			Submit(){//保存信息
				uni.request({
					url:this.BaseUrl + 'fast-scale/archives',
					method:'POST',
					header:{
						'Authorization':'Bearer dsadasdasdasdasdjasldjalksdjalskjdla'
					},
					data:{
						genderDesc: this.multiArray[this.multiIndex],
						birthday: this.date,
						height: this.height,
						targetWeight: this.targetWeight
					},
					success: (res) => {
						console.log(res)
						if(res.data.data){
							uni.showToast({
								title:'上传数据成功',
								icon:'none'
							})
							setTimeout(function(){
								uni.navigateBack({
									delta:1
								})
							},500)
						}
					}
				})
				
			},
			getDate(type) {
				const date = new Date();
				let year = date.getFullYear();
				let month = date.getMonth() + 1;
				let day = date.getDate();
			
				if (type === 'start') {
					year = year - 60;
				} else if (type === 'end') {
					year = year + 2;
				}
				month = month > 9 ? month : '0' + month;;
				day = day > 9 ? day : '0' + day;
				return `${year}-${month}-${day}`;
			},
			bindDateChange: function(e) {
				this.date = e.target.value
			},
			bindPickerChange: function(e) {
				this.multiIndex = e.detail.value
				this.$forceUpdate()
			},
		}
	}
</script>

<style lang="less">
.person{
	padding-bottom: 150rpx;
	input{
		direction: rtl;
		height: 60rpx;
	}
	.custom2{
		color: #3B4144;
		display: flex;
		height: 92rpx;
		line-height: 92rpx;
		justify-content: space-between;
		align-items: center;
		padding: 0 50rpx 0 30rpx;
		font-size: 28rpx;
	}
	.center{
		.custom{
			color: #3B4144;
			display: flex;
			height: 92rpx;
			line-height: 92rpx;
			justify-content: space-between;
			align-items: center;
			padding: 0 50rpx 0 30rpx;
			font-size: 28rpx;
			input{
				text-align: right;
			}
		}
	}
	.bottom{
		padding: 30rpx 50rpx;
		display: flex;
		justify-content: space-around;
		align-items: center;
		.bottom_item{
			margin-bottom: 40rpx;
			color: #C2C2C2;
			display: flex;
			justify-content: center;
			align-items: center;
			flex-direction: column;
			font-size: 25rpx;
		}
	}
	.fixed{
		padding: 30rpx;
		position: fixed;
		bottom: 10rpx;
		width: 90%;
		height: 100rpx;
		z-index: 998;
		view{
			color: white;
			font-size: 35rpx;
			font-weight: 480;
			text-align: center;
			background:linear-gradient(#FFC747,#FE9105);
			height: 100rpx;
			line-height: 100rpx;
			border-radius: 60rpx;
		}
	}
}
</style>
