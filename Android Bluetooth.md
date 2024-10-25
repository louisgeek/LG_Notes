
# Android Bluetooth
- 传统蓝牙（标准蓝牙）
- Bluetooth Low Energy 低功耗蓝牙（低能耗蓝牙）
- 可以同时使用传统蓝牙和低功耗蓝牙，但是不能同时扫描（搜索、发现、查询）低功耗蓝牙设备和传统蓝牙设备
- Android 6 以上开启蓝牙还额外需要定位权限
- Android 8 以上可选 CompanionDeviceManager 


## 打开蓝牙
1. 通过获取 BLUETOOTH_SERVICE 系统服务获取到 BluetoothManager
2. 通过 bluetoothManager.adapter 获取 BluetoothAdapter 蓝牙适配器，如果为 null 代表该设备不支持蓝牙
3. 如果 bluetoothAdapter.isEnabled = false 代表没打开，就可以利用 startActivityForResult(Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE), REQUEST_ENABLE_BT) 去打开，然后可以从 onActivityResult 里获取到结果


## 传统蓝牙
传统蓝牙

### 查找
#### 查找已配对设备
- 通过 bluetoothAdapter?.bondedDevices 获取得到 BluetoothDevice 的 Set 集合，可以用列表展示出来

#### 设备发现
- 可以利用 bluetoothAdapter.startDiscovery 进行设备发现操作（大概持续 12 秒），也可以通过 bluetoothAdapter.isDiscovering 判断是否需要先执行 bluetoothAdapter.cancelDiscovery 操作，由于设备发现比较消耗蓝牙适配器的资源，所以需要及时执行 cancelDiscovery 操作 
- 通过监听 BluetoothDevice.ACTION_FOUND 广播利用 BluetoothDevice.EXTRA_DEVICE 获取到 BluetoothDevice
- 通过监听 BluetoothAdapter.ACTION_DISCOVERY_FINISHED 广播知晓发现操作执行完毕

#### 设置可被检测
可以通过执行 startActivityForResult 传入带 BluetoothAdapter.EXTRA_DISCOVERABLE_DURATION 参数的 Intent(BluetoothAdapter.ACTION_REQUEST_DISCOVERABLE) 使得本地设备可被检测（发现），BluetoothAdapter.EXTRA_DISCOVERABLE_DURATION 支持传入秒数，默认不传是 120 秒，0 秒代表一直可被检测到（官方不建议使用），数值最长支持 3600 秒

### 连接
#### 服务端

#### 客户端

### 数据传输



## 低功耗蓝牙
- 在智能穿戴设备、近程传感器、心率监测器、健身设备和车载系统上比较广泛
- 搜索、连接的速度更快，不过传输的速度慢，传输的数据量也很小，GATT 协议规定每次最多传 20 个字节，超过就需要分批多次进行传输了
- 中央设备与外围设备，中央设备处于中心角色会进行扫描并查找通告，处于外围设备角色的设备会发出通告
- GATT 服务器与 GATT 客户端，比如安卓设备充当客户端，心率监测器充当服务端

- ATT （Attribute Protocol 属性协议），每个属性均由 UUID 通用唯一标识符进行唯一标识的
- GATT （Generic Attribute Profile 通用属性配置文件）协议，GATT 是基于属性协议的，也称为 GATT/ATT，属性代表一小段数据
- Profiles Bluetooth SIG 一个设备可以实现多个配置文件，比如设备包含心率监测器和电池电量检测器
- Characteristic 特征
- Descriptor 描述符
- Service 服务


### 设备扫描
- 通过 bluetoothAdapter.bluetoothLeScanner 获取 BLE 扫描器，通过 bluetoothLeScanner.startScan 传入 ScanCallback 进行开始扫描操作，扫描支持传入 ScanFilter 列表进行过滤，通过 bluetoothLeScanner.stopScan 停止扫描，可以通过 Handler.postDelayed 延迟 10 秒左右主动去停止扫描
- 通过 ScanCallback 回调里返回的设备可以存起来，存入列表之前可能需要先判断去重
- 通过 bluetoothDevice.connectGatt 传入 BluetoothGattCallback  和 TRANSPORT_LE 参数去连接设备的 Gatt 服务，
