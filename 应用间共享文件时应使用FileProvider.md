# 应用间共享文件时，不要通过放宽文件系统权限的方式去实现，而应使用FileProvider。

应用间共享文件时，不要通过放宽文件系统权限的方式去实现，而应使用
FileProvider。
正例：
​	

```java
<!-- AndroidManifest.xml -->

<manifest>

...

<application>

...

<provider

android:name="android.support.v4.content.FileProvider"

android:authorities="com.example.fileprovider"

android:exported="false"

android:grantUriPermissions="true">

<meta-data

android:name="android.support.FILE_PROVIDER_PATHS"

android:resource="@xml/provider_paths" />

</provider>

...

</application>

</manifest>

<!-- res/xml/provider_paths.xml -->

<paths>

<files-path path="album/" name="myimages" />

</paths>

void getAlbumImage(String imagePath) {

File image = new File(imagePath);

Intent getAlbumImageIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);

Uri imageUri = FileProvider.getUriForFile(

this,

"com.example.provider",

image);

getAlbumImageIntent.putExtra(MediaStore.EXTRA_OUTPUT, imageUri);



startActivityForResult(takePhotoIntent, REQUEST_GET_ALBUMIMAGE);

}

```



反例：

```java
void getAlbumImage(String imagePath) {

File image = new File(imagePath);

Intent getAlbumImageIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);

//不要使用 file://的 URI 分享文件给别的应用，包括但不限于 Intent

getAlbumImageIntent.putExtra(MediaStore.EXTRA_OUTPUT, Uri.fromFile(image));

startActivityForResult(takePhotoIntent, REQUEST_GET_ALBUMIMAGE);

}

```

