<template>
	<view class="content">
		<view class="text-area">
			<view v-for="(item, index) in list" :key="index">
				<view style="display: block;height: 100rpx;line-height: 100rpx;" @click="createBLEConnection(item)">{{ item }}</view>
			</view>
			<!-- <view style="text-align: center;margin-top: 50rpx;" @click="toget">获取响应</view> -->
		</view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			list: [],
			deviceId: '',
			serviceId: '',
			characteristics: '',
			characteristicId: ''
		};
	},
	onLoad() {

		uni.openBluetoothAdapter({
			success: res => {
				this.startBluetoothDeviceDiscovery();
			}
		});
	},
	methods: {
		// 发现外围设备
		onBluetoothDeviceFound() {
			// console.log("zhixing")
			uni.onBluetoothDeviceFound(res => {
				// ["name", "deviceId"]
				// 吧搜索到的设备存储起来，方便我们在页面上展示
				if (res.devices[0].name.indexOf('HM') != -1) {
					this.list.push(res.devices[0].deviceId);
				}
				console.log(this.list)
			});
		},
		// 开始搜索蓝牙设备
		startBluetoothDeviceDiscovery() {
			uni.startBluetoothDevicesDiscovery({
				success: res => {
					// 发现外围设备
					this.onBluetoothDeviceFound();
				},
				fail: err => {
					console.log(err, '错误信息');
				}
			});
		},
		// 停止搜寻蓝牙设备
		stopBluetoothDevicesDiscovery() {
			uni.stopBluetoothDevicesDiscovery({
				success: e => {
					this.loading = false;
					console.log('停止搜索蓝牙设备:' + e.errMsg);
					this.getBLEDeviceServices();
				},
				fail: e => {
					console.log('停止搜索蓝牙设备失败，错误码：' + e.errCode);
				}
			});
		},
		//选择设备连接吧deviceId传进来
		createBLEConnection(deviceId) {
			console.log(deviceId);
			let thit = this;
			//data里面建立一个deviceId，存储起来
			this.deviceId = deviceId;
			//连接蓝牙
			uni.createBLEConnection({
				// 这里的 deviceId 需要已经通过 createBLEConnection 与对应设备建立链接
				deviceId: this.deviceId,
				success(res) {
					//防止在这里面取不到this，古在外面用thit存储了this
					thit.stopBluetoothDevicesDiscovery();
					console.log(res);
					console.log('蓝牙连接成功');
				},
				fail(res) {
					console.log('蓝牙连接失败', res);
				}
			});
		},
		//获取蓝牙的所有服务
		getBLEDeviceServices() {
			console.log('获取蓝牙服务' + this.deviceId);
			setTimeout(() => {
				uni.getBLEDeviceServices({
					// 这里的 deviceId 需要已经通过 createBLEConnection 与对应设备建立链接
					deviceId: this.deviceId,
					success: res => {
						// console.log("成功",res)
						//这里会获取到好多个services  uuid  我们只存储我们需要用到的就行，这个uuid一般硬件厂家会给我们提供
						res.services.forEach(item => {
							// if(item.uuid.indexOf("00001800")!=-1){
							if (item.uuid.indexOf('0783B03E-8535-B5A0-7140-A304D2495CB7') != -1) {
								this.serviceId = item.uuid;
								//进入特征
								setTimeout(()=>{
									this.getBLEDeviceCharacteristics();
								},1000)
							}
						});
					}
				});
			}, 1500);
		},
		//获取蓝牙特征
		getBLEDeviceCharacteristics() {
			console.log('进入特征');
			console.log('获取蓝牙服务' + this.deviceId + '服务' + this.serviceId);
			// setTimeout(() => {
				plus.bluetooth.onBLECharacteristicValueChange(function(e) {
					console.log(e);
					console.log(that.ab2hex(e.value));
					var array = new Uint8Array(e.value); // 这是监听蓝牙返回的数据
				});
				uni.getBLEDeviceCharacteristics({
					// 这里的 deviceId 需要已经通过 createBLEConnection 与对应设备建立链接
					deviceId: this.deviceId,
					// 这里的 serviceId 需要在 getBLEDeviceServices 接口中获取
					serviceId: this.serviceId,
					success: res => {
						this.characteristics = res.characteristics;
						//循环所有的uuid

						// for(let i=0;i<3;i++){
						// 	this.notifyBLECharacteristicValueChange(res.characteristics[i].uuid)
						// 	console.log(res.characteristics[i].uuid)
						// 	console.log(i,'i')
						// }
						console.log(res)
						res.characteristics.forEach(item => {
							if (item.uuid.indexOf('0783B03E-8535-B5A0-7140-A304D2495CB8') != -1) {
								// if(item.uuid.indexOf("00002A00-0000-1000-8000-00805F9B34FB") != -1){
								//利用传参的形势传给下面的notify，这里的uuid如果都需要用到，就不用做判断了，建议使用setTimeout进行间隔性的调用此方法
								this.notifyBLECharacteristicValueChange(item.uuid);
							}
							// if (item.uuid.indexOf('0783B03E-8535-B5A0-7140-A304D2495CBA') != -1) {
							// 	setTimeout(()=>{
							// 		this.writeBLECharacteristicValue(item.uuid);
							// 	},3000)
							// 	// this.notifyBLECharacteristicValueChange(item.uuid)
							// }
						});
					},
					fail: res => {
						console.log(res);
					}
				});
			// }, 1000);
		},
		toget() {},
		hexStringToArrayBuffer(str) {
			if (!str) {
			return new ArrayBuffer(0);
			}
			var buffer = new ArrayBuffer(str.length);
			let dataView = new DataView(buffer)
			let ind = 0;
			for (var i = 0, len = str.length; i < len; i += 2) {
				let code = parseInt(str.substr(i, 2), 16)
				dataView.setUint8(ind, code)
				ind++
			}
				return buffer;
			},
			notifyBLECharacteristicValueChange(characteristicId) {
			this.characteristicId = characteristicId;
			let that = this;
			
			setTimeout(() => {
				
				uni.notifyBLECharacteristicValueChange({
					state: true, // 启用 notify 功能
					// 这里的 deviceId 需要已经通过 createBLEConnection 与对应设备建立链接
					deviceId: this.deviceId,
					// 这里的 serviceId 需要在 getBLEDeviceServices 接口中获取
					serviceId: this.serviceId,
					// 这里的 characteristicId 需要在 getBLEDeviceCharacteristics 接口中获取
					characteristicId: this.characteristicId,
					success: res => {
						console.log(res);
						console.log('开启监听')
						uni.onBLECharacteristicValueChange(function(e){
								console.log('onBLECharacteristicValueChange: '+JSON.stringify(e));
								var value = buffer2hex(e.value);
								console.log('value(hex) = '+value);
								if(characteristicId == e.characteristicId){
									// 更新到页面显示
									alert('特征值变化: '+value);
								}
						});
						// this.notifyBLECharacteristicValueChange(characteristicId);
					},
					fail: res => {
						console.log(res);
					}
				});
				
			}, 1500);

		},
		ab2hex(buffer) {
			const hexArr = Array.prototype.map.call(new Uint8Array(buffer), function(bit) {
				return ('00' + bit.toString(16)).slice(-2);
			});
			return hexArr.join('');
		},
		writeBLECharacteristicValue(characteristicId) {
			let buffer =this.hexStringToArrayBuffer("0xF50x03AABBCCDDEEFFGGHHJJ00000000CRC")
			uni.writeBLECharacteristicValue({
				// 这里的 deviceId 需要在 getBluetoothDevices 或 onBluetoothDeviceFound 接口中获取
				deviceId: this.deviceId,
				// 这里的 serviceId 需要在 getBLEDeviceServices 接口中获取
				serviceId: this.serviceId,
				// 这里的 characteristicId 需要在 getBLEDeviceCharacteristics 接口中获取
				characteristicId: characteristicId,
				// 这里的value是ArrayBuffer类型
				value: buffer,
				success(res) {
					console.log(res);
				},
				fail: (err) => {
					console.log(err)
				}
			});
		},
		strToHexCharCode() {
			var hex = '0xF5 0x03 AA BB CC DD EE FF GG HH JJ 00 00 00 00 CRC'; //要发送的信息
			var typedArray = new Uint8Array(
				hex.match(/[\da-f]{2}/gi).map(function(h) {
					return parseInt(h, 16);
				})
			);
			return typedArray.buffer;
		},
		// byte 转 16进制 用：间隔模式
		Bytes2Str(arrBytes) {
			var str = '';
			for (var i = 0; i < arrBytes.length; i++) {
				var tmp;
				var num = arrBytes[i];
				if (num < 0) {
					//此处填坑，当byte因为符合位导致数值为负时候，需要对数据进行处理
					tmp = (255 + num + 1).toString(16);
				} else {
					tmp = num.toString(16);
				}
				if (tmp.length == 1) {
					tmp = '0' + tmp;
				}
				if (i > 0) {
					str += ':' + tmp; // 可以修改 : 成你需要的间隔符号，如果不需要间隔符号那就空着
				} else {
					str += tmp;
				}
			}
			return str;
		},
		//比如 [12,5,8,41,2]
		// 返回的是 12:5:8:41:2

		// byte转纯字符串
		BytesToStr(arrBytes) {
			var str = '';
			for (var i = 0; i < arrBytes.length; i++) {
				var tmp;
				var num = arrBytes[i];
				if (num < 0) {
					//此处填坑，当byte因为符合位导致数值为负时候，需要对数据进行处理
					tmp = (255 + num + 1).toString(16);
				} else {
					tmp = num.toString(16);
				}
				if (tmp.length == 1) {
					tmp = '0' + tmp;
				}
				if (i > 0) {
					str += tmp;
				} else {
					str += tmp;
				}
			}
			return str;
		},

		// ！！！！重点！！！！  很多时候因为你返回的16进制 是 2个数字为 1 字节
		// 比如 442288C7 这种，用简单的字符转byte 会把字符串 44 分开编译成两个Byte数组
		// 以下方法是 44 22 88 C7 这样为一组编译
		// 字符 转 16进制Byte // 两个为一组
		bin2Str(str) {
			let a = [];
			let arr = [];
			for (var i = 0; i < str.length; i += 2) {
				a.push('0x' + str.substr(i, 2));
				a.join(' ');
			}
			if (a.length) {
				a.forEach(item => {
					arr.push(parseInt(item, 16));
				});
			} else {
				return [];
			}
			return arr;
		},
		// Hex字符 转换成 Byte
		HexString2Bytes(str) {
			var pos = 0;
			var len = str.length;
			if (len % 2 != 0) {
				return null;
			}
			len /= 2;
			var arrBytes = new Array();
			for (var i = 0; i < len; i++) {
				var s = str.substr(pos, 2);
				var v = parseInt(s, 16);
				arrBytes.push(v);
				pos += 2;
			}
			return arrBytes;
		},
		//CRC蓝牙校验
				crc16(buffer) {
				  var crc = 0xF5; // 这是 设置初始值的 可以根据自己的CRC16协议进行调整
				  var odd;
				  for(var i = 0; i < buffer.length; i++) {
				    crc ^= (buffer[i] << 8)
				    for(var j = 0; j < 8; j++) {
				      odd = crc & 0x8000;
				      crc = crc << 1;
				      if(odd) {
				        crc = crc ^ 0x1021
				      }
				    }
				  }
				  var hi = ((crc & 0xFF00) >> 8); //高位置
				  var lo = (crc & 0x00FF); //低位置
				  var crcArr = []
				  crcArr.push(hi)
				  crcArr.push(lo)
				  crc &= 0xFFFF // 偏差值
				  return this.crcToString(crcArr, false) // 传true false 是看你参数是高位在前 还是 低位在前，各人情况不同定
				},
				//转为大写String
				crcToString(arr, isReverse) {
				  if(typeof isReverse == 'undefined') {
				    isReverse = true;
				  }
				  var hi = arr[0],
				    lo = arr[1];
				  return this.padLeft((isReverse ? hi + lo * 0x100 : hi * 0x100 + lo).toString(16).toUpperCase(), 4, '0');
				},
				padLeft(s, w, pc) {
				    if (pc == undefined) {
				        pc = '0';
				    }
				    for (var i = 0, c = w - s.length; i < c; i++) {
				        s = pc + s;
				    }
				    return s;
				}
	}
};
</script>

<style>
.content {
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
}

.logo {
	height: 200rpx;
	width: 200rpx;
	margin-top: 200rpx;
	margin-left: auto;
	margin-right: auto;
	margin-bottom: 50rpx;
}

.title {
	font-size: 36rpx;
	color: #8f8f94;
}
</style>
