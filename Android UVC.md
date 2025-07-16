UVC 

 

 

## USB_CLASS_VIDEO  判断

```kotlin
private fun isUsbClassVideo(device: UsbDevice): Boolean {
        //UsbConstants.USB_CLASS_VIDEO 14
        if (device.deviceClass == UsbConstants.USB_CLASS_VIDEO) {
            return true
        }
        for (i in 0 until device.interfaceCount) {
            val deviceInterface = device.getInterface(i)
            if (deviceInterface.interfaceClass == UsbConstants.USB_CLASS_VIDEO) {
                return true
            }
        }
        return false
}
```



## usb 权限请求兼容处理

```kotlin
   fun requestUsbPermissionCompat(
        context: Context,
        usbDevice: UsbDevice,
        requestCameraPermission: () -> Unit,
        requestUsbPermission: () -> Unit
    ) {
        //ps:https://developer.android.google.cn/reference/android/hardware/usb/UsbInstace.html#requestPermission(android.hardware.usb.UsbDevice,%20android.app.PendingIntent)
        if (context.applicationInfo.targetSdkVersion < Build.VERSION_CODES.P) {
            //低版本直接请求usb
            requestUsbPermission.invoke()
            Log.e(TAG, "requestUsbPermissionCompat: 低版本直接请求usb")
        } else {
            //ps:在 targetSdkVersion >= 28 情况下,如果是 USB_CLASS_VIDEO 设备，需要有相机权限，否则会不弹 USB 访问请求对话框
            if (isUsbClassVideo(usbDevice)) {
                val isCameraPermissionGranted = ContextCompat.checkSelfPermission(
                    context,
                    Manifest.permission.CAMERA
                ) == PackageManager.PERMISSION_GRANTED
                if (isCameraPermissionGranted) {
                    Log.e(TAG, "requestUsbPermissionCompat: 高版本 已经有相机权限")
                    //此时可以继续 UsbManager#requestPermission 逻辑了
                    requestUsbPermission.invoke()
                } else {
                    Log.e(TAG, "requestUsbPermissionCompat: 高版本 还没有相机权限,需要去申请")
                    requestCameraPermission.invoke()
                }
            } else {
                Log.e(TAG, "requestUsbPermissionCompat: 高版本 不是 camera 设备")
                requestUsbPermission.invoke()
            }
        }
    }
```



## 获取 usb 信息并打开

```kotlin
private lateinit var usbManager: UsbManager
private lateinit var usbPermissionPI: PendingIntent
private val ACTION_USB_PERMISSION = "com.android.usb.USB_PERMISSION"
  
 private val usbReceiver = object : BroadcastReceiver() {
        override fun onReceive(context: Context, intent: Intent) {
            when (intent.action) {
                UsbManager.ACTION_USB_DEVICE_ATTACHED -> {
                   val device: UsbDevice = intent.getParcelableExtra(UsbManager.EXTRA_DEVICE)
                        ?: return
                    //
                }
                UsbManager.ACTION_USB_DEVICE_DETACHED -> {
                    val device: UsbDevice = intent.getParcelableExtra(UsbManager.EXTRA_DEVICE)
                        ?: return
 					//
                }
                ACTION_USB_PERMISSION -> {
                    synchronized(this) {
                        val device: UsbDevice =
                            intent.getParcelableExtra(UsbManager.EXTRA_DEVICE)
                                ?: return
                        if (intent.getBooleanExtra(UsbManager.EXTRA_PERMISSION_GRANTED, false)) {
                           //授予 usb 权限后
                           openUsbDevice(device)
                        }
                    }
                }
            }

        }
    }

override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
		//处理摄像头权限
        PermissionsManager.init(this) {
            //获取摄像头权限后，请求 usb 权限
            requestUsbPermission()
        }
		//
        usbManager = this.getSystemService(Context.USB_SERVICE) as UsbManager
        usbPermissionPI = PendingIntent.getBroadcast(this, 0, Intent(ACTION_USB_PERMISSION), 0)
		//
        val intentFilter = IntentFilter()
        intentFilter.addAction(UsbManager.ACTION_USB_DEVICE_ATTACHED)
        intentFilter.addAction(UsbManager.ACTION_USB_DEVICE_DETACHED)
        intentFilter.addAction(ACTION_USB_PERMISSION)
        registerReceiver(usbBroadcastReceiver, intentFilter)


    }

override fun onDestroy() {
        unregisterReceiver(usbReceiver)
        super.onDestroy()
}

private fun requestUsbPermission() {
        Log.e(TAG, "requestUsbPermission: ")
        val deviceMap = usbManager.deviceList
        val uvcDeviceList = deviceMap.values.filter { device -> isUVC(device) }
        if (uvcDeviceList.isNotEmpty()) {
            val device = uvcDeviceList[0]
            val vendorId = device.vendorId
            val manufacturerName = device.manufacturerName
            val productId = device.productId
            val productName = device.productName
            if (usbManager.hasPermission(device)) {
                openUsbDevice(device)
            } else {
                usbManager.requestPermission(device, usbPermissionPI)
            }
        }
}

private fun isUVC(device: UsbDevice): Boolean {
    //class="239" subclass="2"
	return device.deviceClass == 239 && device.deviceSubclass == 2
}

private fun openUsbDevice(usbDevice: UsbDevice) {
        Log.e(TAG, "openUsbDevice: ")
        val usbDeviceConnection = usbManager.openDevice(usbDevice)
        val fileDescriptor = usbDeviceConnection.fileDescriptor
        val serial = usbDeviceConnection.serial
        val serialNumber = usbDevice.serialNumber
        
}
```

