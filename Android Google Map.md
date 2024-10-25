

    ```java
    public static final int MAP_TYPE_NONE = 0;
    //普通地图
    public static final int MAP_TYPE_NORMAL = 1;
    //卫星地图
    public static final int MAP_TYPE_SATELLITE = 2;
    //地形地图
    public static final int MAP_TYPE_TERRAIN = 3;
    //混合地图
    public static final int MAP_TYPE_HYBRID = 4;
    ```

# LatLng 以纬度和经度表示的地理位置坐标


## latitudes 维度


## longitude 经度

## addMarker
- 在地图上添加一个标记
```kotlin
 val sydney = LatLng(-33.852, 151.211)
 googleMap.addMarker(
        MarkerOptions()
            .position(sydney)
            .title("Marker in Sydney")
    )

googleMap.moveCamera(CameraUpdateFactory.newLatLng(sydney))
```

## MarkerOptions

### position 位置的 LatLng 值

### alpha 透明度

### anchor

### draggable 是否允许移动

### flat 

### icon

### 

## moveCamera
