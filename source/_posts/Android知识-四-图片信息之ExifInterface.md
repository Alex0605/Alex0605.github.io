---
title: Android知识(四)--图片信息之ExifInterface
date: 2018-03-01 16:49:14
tags: [Android]
---

### 1、Exif-ExifInterface简介 

​	Exif是一种图像文件格式，它的数据存储与JPEG格式是完全相同的。实际上Exif格式就是在JPEG格式头部插入了数码照片的信息，包括拍摄时的光圈、快门、白平衡、ISO、焦距、日期时间等各种和拍摄条件以及相机品牌、型号、色彩编码、拍摄时录制的声音以及GPS全球定位系统数据、缩略图等。你可以利用任何可以查看JPEG文件的看图软件浏览Exif格式的照片，但并不是所有的图形程序都能处理Exif信息。今天这篇文章就来讲讲Android中操作Exif。

### 2、ExifInterface中的功能简单进行介绍 

​	Android开发中，在对图片进行展示、编辑、发送等操作时经常会涉及Exif的操作，Android中操作Exif主要是通过ExifInterface，ExifInterface看上去是一个接口，其实是一个类，位于Android.media.ExifInterface的位置。进入ExifInterface类，发现方法很少，主要就是三个方面：读取、写入、缩略图。

### 3、ExifInterface进行读取操作

​	Exif信息在文件头中是以二进制的形式存储的，存储的字段名称和字段值格式都是固定的。我测试的Android23（6.0）版本中，总共有26个Exif字段，其中TAG_SUBSECTIME被加上了@hide注解，也就是还剩25个，我写了个demo，获取这25个字段的值，看看都是什么样的格式。

```java
ExifInterface exifInterface = new ExifInterface(filePath);

String orientation = exifInterface.getAttribute(ExifInterface.TAG_ORIENTATION);
String dateTime = exifInterface.getAttribute(ExifInterface.TAG_DATETIME);
String make = exifInterface.getAttribute(ExifInterface.TAG_MAKE);
String model = exifInterface.getAttribute(ExifInterface.TAG_MODEL);
String flash = exifInterface.getAttribute(ExifInterface.TAG_FLASH);
String imageLength = exifInterface.getAttribute(ExifInterface.TAG_IMAGE_LENGTH);
String imageWidth = exifInterface.getAttribute(ExifInterface.TAG_IMAGE_WIDTH);
String latitude = exifInterface.getAttribute(ExifInterface.TAG_GPS_LATITUDE);
String longitude = exifInterface.getAttribute(ExifInterface.TAG_GPS_LONGITUDE);
String latitudeRef = exifInterface.getAttribute(ExifInterface.TAG_GPS_LATITUDE_REF);
String longitudeRef = exifInterface.getAttribute(ExifInterface.TAG_GPS_LONGITUDE_REF);
String exposureTime = exifInterface.getAttribute(ExifInterface.TAG_EXPOSURE_TIME);
String aperture = exifInterface.getAttribute(ExifInterface.TAG_APERTURE);
String isoSpeedRatings = exifInterface.getAttribute(ExifInterface.TAG_ISO);
String dateTimeDigitized = exifInterface.getAttribute(ExifInterface.TAG_DATETIME_DIGITIZED);
String subSecTime = exifInterface.getAttribute(ExifInterface.TAG_SUBSEC_TIME);
String subSecTimeOrig = exifInterface.getAttribute(ExifInterface.TAG_SUBSEC_TIME_ORIG);
String subSecTimeDig = exifInterface.getAttribute(ExifInterface.TAG_SUBSEC_TIME_DIG);
String altitude = exifInterface.getAttribute(ExifInterface.TAG_GPS_ALTITUDE);
String altitudeRef = exifInterface.getAttribute(ExifInterface.TAG_GPS_ALTITUDE_REF);
String gpsTimeStamp = exifInterface.getAttribute(ExifInterface.TAG_GPS_TIMESTAMP);
String gpsDateStamp = exifInterface.getAttribute(ExifInterface.TAG_GPS_DATESTAMP);
String whiteBalance = exifInterface.getAttribute(ExifInterface.TAG_WHITE_BALANCE);
String focalLength = exifInterface.getAttribute(ExifInterface.TAG_FOCAL_LENGTH);
String processingMethod = exifInterface.getAttribute(ExifInterface.TAG_GPS_PROCESSING_METHOD);

Log.e("TAG", "## orientation=" + orientation);
Log.e("TAG", "## dateTime=" + dateTime);
Log.e("TAG", "## make=" + make);
Log.e("TAG", "## model=" + model);
Log.e("TAG", "## flash=" + flash);
Log.e("TAG", "## imageLength=" + imageLength);
Log.e("TAG", "## imageWidth=" + imageWidth);
Log.e("TAG", "## latitude=" + latitude);
Log.e("TAG", "## longitude=" + longitude);
Log.e("TAG", "## latitudeRef=" + latitudeRef);
Log.e("TAG", "## longitudeRef=" + longitudeRef);
Log.e("TAG", "## exposureTime=" + exposureTime);
Log.e("TAG", "## aperture=" + aperture);
Log.e("TAG", "## isoSpeedRatings=" + isoSpeedRatings);
Log.e("TAG", "## dateTimeDigitized=" + dateTimeDigitized);
Log.e("TAG", "## subSecTime=" + subSecTime);
Log.e("TAG", "## subSecTimeOrig=" + subSecTimeOrig);
Log.e("TAG", "## subSecTimeDig=" + subSecTimeDig);
Log.e("TAG", "## altitude=" + altitude);
Log.e("TAG", "## altitudeRef=" + altitudeRef);
Log.e("TAG", "## gpsTimeStamp=" + gpsTimeStamp);
Log.e("TAG", "## gpsDateStamp=" + gpsDateStamp);
Log.e("TAG", "## whiteBalance=" + whiteBalance);
Log.e("TAG", "## focalLength=" + focalLength);
Log.e("TAG", "## processingMethod=" + processingMethod);
```

打印的结果

```java
E/TAG: ## orientation=0
E/TAG: ## dateTime=2016:05:23 17:30:11
E/TAG: ## make=Xiaomi
E/TAG: ## model=Mi-4c
E/TAG: ## flash=16
E/TAG: ## imageLength=4160
E/TAG: ## imageWidth=3120
E/TAG: ## latitude=31/1,58/1,253560/10000
E/TAG: ## longitude=118/1,44/1,491207/10000
E/TAG: ## latitudeRef=N
E/TAG: ## longitudeRef=E
E/TAG: ## exposureTime=0.050
E/TAG: ## aperture=2.0
E/TAG: ## iso=636
E/TAG: ## dateTimeDigitized=2016:05:23 17:30:11
E/TAG: ## subSecTime=379693
E/TAG: ## subSecTimeOrig=379693
E/TAG: ## subSecTimeDig=379693
E/TAG: ## altitude=0/1000
E/TAG: ## altitudeRef=0
E/TAG: ## gpsTimeStamp=9:30:8
E/TAG: ## gpsDateStamp=2016:05:23
E/TAG: ## whiteBalance=0
E/TAG: ## focalLength=412/100
E/TAG: ## processingMethod=NETWORK
```

这25个字段分别是代表什么呢？

```java
ExifInterface.TAG_ORIENTATION //旋转角度，整形表示，在ExifInterface中有常量对应表示
ExifInterface.TAG_DATETIME //拍摄时间，取决于设备设置的时间
ExifInterface.TAG_MAKE //设备品牌
ExifInterface.TAG_MODEL //设备型号，整形表示，在ExifInterface中有常量对应表示
ExifInterface.TAG_FLASH //闪光灯
ExifInterface.TAG_IMAGE_LENGTH //图片高度
ExifInterface.TAG_IMAGE_WIDTH //图片宽度
ExifInterface.TAG_GPS_LATITUDE //纬度
ExifInterface.TAG_GPS_LONGITUDE //经度
ExifInterface.TAG_GPS_LATITUDE_REF //纬度名（N or S）
ExifInterface.TAG_GPS_LONGITUDE_REF //经度名（E or W）
ExifInterface.TAG_EXPOSURE_TIME //曝光时间
ExifInterface.TAG_APERTURE //光圈值
ExifInterface.TAG_ISO //ISO感光度
ExifInterface.TAG_DATETIME_DIGITIZED //数字化时间
ExifInterface.TAG_SUBSEC_TIME //
ExifInterface.TAG_SUBSEC_TIME_ORIG //
ExifInterface.TAG_SUBSEC_TIME_DIG //
ExifInterface.TAG_GPS_ALTITUDE //海拔高度
ExifInterface.TAG_GPS_ALTITUDE_REF //海拔高度
ExifInterface.TAG_GPS_TIMESTAMP //时间戳
ExifInterface.TAG_GPS_DATESTAMP //日期戳
ExifInterface.TAG_WHITE_BALANCE //白平衡
ExifInterface.TAG_FOCAL_LENGTH //焦距
ExifInterface.TAG_GPS_PROCESSING_METHOD //用于定位查找的全球定位系统处理方法。
```

其中TAG_SUBSEC_TIME 、TAG_SUBSEC_TIME_ORIG 、TAG_SUBSEC_TIME_DIG 没有加注释，我也没查清楚具体是什么意思，但是看log，三个值是一样的。有知道的朋友可以跟我说下，谢谢！

细心的朋友会发现，上面所有的取值都是用的一个方法：exifInterface.getAttribute(String tag)，其实ExifInterface还提供了其它方法。

```java
exifInterface.getAltitude(long default); //返回海拔高度，单位米，如果exif的tag不存在，返回默认值。 
exifInterface.getAttributeDouble(String tag, Double default) //返回double值，传入默认值 
exifInterface.getAttributeInt(String tag, int default) //返回int值，传入默认值 
exifInterface.getLatLong(float[] value) //返回纬度和经度，数组第一个是纬度，第二个是经度
```

### 4、ExifInterface进行写入操作

相对读取，写入就简单很多了。

```java
ExifInterface exifInterface = new ExifInterface(filePath);
exifInterface.setAttribute(ExifInterface.TAG_GPS_ALTITUDE,"1/1000");
exifInterface.setAttribute(ExifInterface.TAG_ORIENTATION,"6");
exifInterface.setAttribute(ExifInterface.TAG_IMAGE_WIDTH,"2000");
exifInterface.saveAttributes();
```

代码很简答，但有一点需要说下，setAttributes只是设置属性值，没有保存，saveAttributes才真正保存，但是这个方法比较耗时，不要每set一次都save，全部set完后，再统一save一次。 

有一点很尴尬，saveAttributes()这个方法内部会遍历保存所有的值，哪怕你只改变了其中一个值。

### 5、ExifInterface获取缩略图

getThumbnail()这个方法可以生成一个缩略图，返回一个字节数组，得到字节数组就可以轻松生成Bitmap。

但是在调用这个方法前，最好先调用exifInterface.hasThumbnail()判断一下是否有缩略图。

getThumbnail()这个方法调用的是native方法，所以具体的实现就看不到了，我也不知道生成的缩略图的分辨率是多少。到这里，ExifInterface的使用介绍以及完毕。

### 6、在ExifInterface获取的信息中进行添加到图片

#### 步骤一

我们在布局文件中activity_main.xml中的代码，创建一个控件

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
	tools:context="com.yoyoyt.exifinterfacedemo.MainActivity">
    <ImageView
        android:id="@+id/imageview"
        android:layout_width="500dp"
        android:layout_height="300dp"/>
</RelativeLayout>
```
#### 步骤二

我们在MainActivity中找的控制，并获取信息，将信息添加到图片上面。并显示在控件上，具体代码如下：

```java
package com.zalex.exifinterfacedemo;

import android.graphics.Bitmap;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Paint;
import android.graphics.Rect;
import android.graphics.Typeface;
import android.graphics.drawable.BitmapDrawable;
import android.graphics.drawable.Drawable;
import android.media.ExifInterface;
import android.os.Bundle;
import android.os.Environment;
import android.support.v7.app.AppCompatActivity;
import android.util.Log;
import android.widget.ImageView;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity {
   
        //获取加载图片的路径
String path = Environment.getExternalStorageDirectory().getPath() + "/DCIM/Camera/IMG_20160927_135402.jpg";
private String width;
private String height;
private Bitmap bitmap;
private Bitmap imgTemp;

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    //找到控件的id
    ImageView iv= (ImageView) findViewById(R.id.imageview);
    bitmap = BitmapFactory.decodeFile(path);//        iv.setImageBitmap(bitmap);
         //android读取图片EXIF信息
    try {
        ExifInterface exifInterface=new ExifInterface(path);
      //         执行保存
            exifInterface.saveAttributes();
            //获取图片的方向
            String orientation = exifInterface.getAttribute(ExifInterface.TAG_ORIENTATION);
            //获取图片的时间
            String dateTime = exifInterface.getAttribute(ExifInterface.TAG_DATETIME);
            String make = exifInterface.getAttribute(ExifInterface.TAG_MAKE);
            String model = exifInterface.getAttribute(ExifInterface.TAG_MODEL);
            String flash = exifInterface.getAttribute(ExifInterface.TAG_FLASH);
             height = exifInterface.getAttribute(ExifInterface.TAG_IMAGE_LENGTH);
             width = exifInterface.getAttribute(ExifInterface.TAG_IMAGE_WIDTH);
            String latitude = exifInterface.getAttribute(ExifInterface.TAG_GPS_LATITUDE);
            String longitude = exifInterface.getAttribute(ExifInterface.TAG_GPS_LONGITUDE);
        StringBuilder sb = new StringBuilder();
        sb.append(longitude)
                .append(latitude);

        Log.e("TAG", "## orientation=" + orientation);
        Log.e("TAG", "## dateTime=" + dateTime);
        Log.e("TAG", "## make=" + make);
        Log.e("TAG", "## model=" + model);
        Log.e("TAG", "## flash=" + flash);
        Log.e("TAG", "## imageLength=" + height);
        Log.e("TAG", "## imageWidth=" + width);
        Log.e("TAG", "## latitude=" + latitude);
        Log.e("TAG", "## longitude=" + longitude);
        String driving_name="驾校名称:东方时尚驾校";
        String coach="教练员姓名:张三";
        String lerner="学员姓名:李四";
        String time="采集时间:2016";
        String car_number="车牌号:京RF8900786";
        String longt="102.00";
        String lat="35.5";
        String car_speed="车辆行驶速度:20km/h";

        Toast.makeText(MainActivity.this,sb, Toast.LENGTH_LONG).show();
        Drawable drawable = createDrawable(driving_name,coach,lerner,time,car_number,longt,lat,car_speed);
        iv.setBackgroundDrawable(drawable);

    } catch (Exception e) {
        e.printStackTrace();
    }

}

// 穿件带字母的标记图片
private Drawable createDrawable(String driving_name, String coach, String lerner, String time, String car_number, String longt, String lat, String car_speed) {
    imgTemp = Bitmap.createBitmap(Integer.valueOf(width), Integer.valueOf(height), Bitmap.Config.ARGB_8888);
    Canvas canvas = new Canvas(imgTemp);
    Paint paint = new Paint(); // 建立画笔
    paint.setDither(true);
    paint.setFilterBitmap(true);
    Rect src = new Rect(0, 0,Integer.valueOf(width), Integer.valueOf(height));
    Rect dst = new Rect(0, 0, Integer.valueOf(width), Integer.valueOf(height));
    canvas.drawBitmap(bitmap, src, dst, paint);

    Paint textPaint = new Paint(Paint.ANTI_ALIAS_FLAG
            | Paint.DEV_KERN_TEXT_FLAG);
    textPaint.setTextSize(100.0f);
    textPaint.setTypeface(Typeface.DEFAULT_BOLD); // 采用默认的宽度
    textPaint.setColor(Color.WHITE);

    canvas.drawText(driving_name,Integer.valueOf(width)/100, Integer.valueOf(height)/8,
            textPaint);
    canvas.drawText(coach,0, (Integer.valueOf(height)/8)+100,textPaint);
    canvas.drawText(lerner,0,(Integer.valueOf(height)/8)+200,textPaint);
    canvas.drawText(time,0,(Integer.valueOf(height)/8)+300,textPaint);
    canvas.drawText(car_number,0,(Integer.valueOf(height)/8)+400,textPaint);
    canvas.drawText(longt,0,(Integer.valueOf(height)/8)+500,textPaint);
    canvas.drawText(lat,0,(Integer.valueOf(height)/8)+600,textPaint);
    canvas.drawText(car_speed,0,(Integer.valueOf(height)/8)+700,textPaint);
    canvas.save(Canvas.ALL_SAVE_FLAG);
    canvas.restore();

    return (Drawable) new BitmapDrawable(getResources(), imgTemp);

}
}
```

#### 步骤三

最后，我们在配置文件中进行添加权限

```xml
 <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
 <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
 <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
 <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
```

