# 记录自己遇到的问题，及解决问题的方式（android）
### 1.应用中如果保存图片需要在系统图库中显示，发送ACTION_MEDIA_SCANNER_SCAN_FILE广播没有效果的话，不妨试一下以下方式：
  新建一个SingleMediaScanner类继承自MediaScannerConnection.MediaScannerConnectionClient
  
    package xx.xx.xx;
    import android.content.Context;
    import android.media.MediaScannerConnection;
    import android.net.Uri;
    import java.io.File;
    public class SingleMediaScanner implements MediaScannerConnection.MediaScannerConnectionClient {
        private MediaScannerConnection mMs;
        private File mFile;
        public SingleMediaScanner(Context context, File f) {
            mFile = f;
            mMs = new MediaScannerConnection(context, this);
            mMs.connect();
        }
        @Override
        public void onMediaScannerConnected() {
            mMs.scanFile(mFile.getAbsolutePath(), null);
        }
        @Override
        public void onScanCompleted(String path, Uri uri) {
            mMs.disconnect();
        }
    }
    
  然后在保存完图片之后以如下方式调用
  
    new SingleMediaScanner(context, new File(imagePath));
    
### 2.记录一次某些手机在4G网络下获取不到mac地址，无法看海康平台视频直播的解决方式，当时的解决方式是随机生成mac地址
    
    public static String randomMACAddress() {
        Random rand = new Random();
        byte[] macAddr = new byte[6];
        rand.nextBytes(macAddr);

        macAddr[0] = (byte) (macAddr[0] & (byte) 254);

        StringBuilder sb = new StringBuilder(18);
        for (byte b : macAddr) {

            if (sb.length() > 0)
                sb.append(":");

            sb.append(String.format("%02x", b));
        }
        return sb.toString();
    }
  这样4G网络下无法看视频的问题就解决了:):):)
  并且附上检测Mac地址格式的代码：
  
    public static boolean checkMacAvailable(String macAddress) {
        String patternMac = "^[a-f0-9]{2}(:[a-f0-9]{2}){5}$";
        return Pattern.compile(patternMac).matcher(macAddress).find();
    }
### 3.对360加固后的apk反编译成功的参考文章
   http://www.cnblogs.com/AsionTang/p/7390591.html
### 4.将bitmap保存到手机内存
    public void saveBitmap(Bitmap bitmap) {
        // 首先保存图片
        File appDir = new File(Environment.getExternalStorageDirectory(), "asave_image");
        if (!appDir.exists()) {
            appDir.mkdirs();
        }
        String fileName = "temp_image" + System.currentTimeMillis() + ".png";
        File file = new File(appDir, fileName);
        try {
            FileOutputStream fos = new FileOutputStream(file);
            bitmap.compress(Bitmap.CompressFormat.JPEG, 100, fos);
            fos.flush();
            fos.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
### 5.将字节数组转为Bitmap
    private Bitmap byte2bitmap(byte[] bytes, int width, int height) {
        Bitmap bitmap = null;
        final int w = width; // 宽度
        final int h = height;
        final YuvImage image = new YuvImage(bytes, ImageFormat.NV21, w, h, null);
        ByteArrayOutputStream os = new ByteArrayOutputStream(bytes.length);
        if (!image.compressToJpeg(new Rect(0, 0, w, h), 100, os)) {
            return null;
        }
        byte[] tmp = os.toByteArray();
        bitmap = BitmapFactory.decodeByteArray(tmp, 0, tmp.length);

        Matrix matrix = new Matrix();
        matrix.setRotate(0);
        bitmap = Bitmap.createBitmap(bitmap, 0, 0, bitmap.getWidth(), bitmap.getHeight(), matrix, true);

        return bitmap;
    }
### 6.EditText点击不弹出系统键盘,显示光标
    public void hideSoftInputMethod(EditText ed){
        
        int currentVersion = android.os.Build.VERSION.SDK_INT;
        String methodName = null;
        if(currentVersion >= 16){
            // 4.2
            methodName = "setShowSoftInputOnFocus";
        }
        else if(currentVersion >= 14){
            // 4.0
            methodName = "setSoftInputShownOnFocus";
        }

        if(methodName == null){
            ed.setInputType(InputType.TYPE_NULL);
        }
        else{
            Class<EditText> cls = EditText.class;
            Method setShowSoftInputOnFocus;
            try {
                setShowSoftInputOnFocus = cls.getMethod(methodName, boolean.class);
                setShowSoftInputOnFocus.setAccessible(true);
                setShowSoftInputOnFocus.invoke(ed, false);
            } catch (NoSuchMethodException e) {
                ed.setInputType(InputType.TYPE_NULL);
                e.printStackTrace();
            } catch (InvocationTargetException e) {
                e.printStackTrace();
            } catch (IllegalAccessException e) {
                e.printStackTrace();
            }
        }
    }
### 6.qq分享返回的时候出现两个一样的应用程序让选择
    出现这种问题的原因可能是因为配置了两次com.tencent.tauth.AuthActivity，如果确认只调用了一次，确认是否调用了某个shareSdk
### 7.TabLayout设置Indicator线的padding
    public void wrapTabIndicatorToTitle(TabLayout tabLayout) {
        View tabStrip = tabLayout.getChildAt(0);
        if (tabStrip instanceof ViewGroup) {
            ViewGroup tabStripGroup = (ViewGroup) tabStrip;
            int childCount = ((ViewGroup) tabStrip).getChildCount();
            for (int i = 0; i < childCount; i++) {
                View tabView = tabStripGroup.getChildAt(i);
                //set minimum width to 0 for instead for small texts, indicator is not wrapped as expected
                tabView.setMinimumWidth(0);
                // set padding to 0 for wrapping indicator as title
                tabView.setPadding(0, tabView.getPaddingTop(), 0, tabView.getPaddingBottom());
                // setting custom margin between tabs
                if (tabView.getLayoutParams() instanceof ViewGroup.MarginLayoutParams) {
                    ViewGroup.MarginLayoutParams layoutParams = (ViewGroup.MarginLayoutParams) tabView.getLayoutParams();
                    if (i == 0) {
                        // left
                        settingMargin(layoutParams);
                    } else if (i == childCount - 1) {
                        // right
                        settingMargin(layoutParams);
                    } else {
                        // internal
                        settingMargin(layoutParams);
                    }
                }
            }

            tabLayout.requestLayout();
        }
    }

    private void settingMargin(ViewGroup.MarginLayoutParams layoutParams) {

        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN_MR1) {
            layoutParams.setMarginStart(SizeUtils.dp2px(30));
            layoutParams.setMarginEnd(SizeUtils.dp2px(30));
        } else {
            layoutParams.leftMargin = SizeUtils.dp2px(30);
            layoutParams.rightMargin = SizeUtils.dp2px(30);
        }
    }
### 8.EditText默认不弹出键盘
    android:focusable="true"
    android:focusableInTouchMode="true"
### 9.Gradle project sync 下载某个jar包长时间下载不下来

   经常会遇到Gradle下载jar包很慢，热别是jcenter中的包，如果发现某个jar包或文件很长时间下载不下来，我们可以使用其他工具(比如迅雷)先现在让后手动放到    gradle中这样就可以不用漫长的等待下载了。
    
   具体操作方法比如kotlin-compiler-embeddable-1.2.71.jar这个jar包有27.7M大小，当时在gradle中的下载速度是30-50K，10分钟都下载不完，我们现在想着    要手动下载，我们去jcenter的官网根据gradle中的下载链接我们找到他的下载地址是“jcenter.bintray.com/org/jetbrains/kotlin/kotlin-compiler-embeddable/1.2.71/kotlin-compiler-embeddable-1.2.71.jar”
    
   我们用迅雷下载，下载速度非常快，我们拿到jar包后放在C:\Users\你的电脑名称\.gradle\caches\modules-2\files-2.1\org.jetbrains.kotlin\kotlin-compiler-embeddable\1.2.71\b394ac31590bff78aea6619b8dc0e2c0958aa599，后面的b394ac31590bff78aea6619b8dc0e2c0958aa599是当初gradle尝试下载时建立的下载目录，直接把jar文件放进去即可，再次sync项目就可以了，这个方法同样适用于其他jar包。

### 10.com.blankj:utilcode中PermissionUtils用法记录
    PermissionUtils.permission(PermissionConstants.CAMERA, PermissionConstants.STORAGE, PermissionConstants.LOCATION,               PermissionConstants.PHONE).rationale(new PermissionUtils.OnRationaleListener() {
                        @Override
                        public void rationale(final ShouldRequest shouldRequest) {
                        }
                    }).callback(new PermissionUtils.FullCallback() {
                        @Override
                        public void onGranted(List<String> permissionsGranted) {
                            LogUtils.d(permissionsGranted);
                            afterRequestPermission();
                        }

                        @Override
                        public void onDenied(List<String> permissionsDeniedForever, List<String> permissionsDenied) {
                            LogUtils.d(permissionsDeniedForever, permissionsDenied);
                            afterRequestPermission();
                        }
                    }).request();
### 11.Vultr VPS搭建SS教程
  https://www.hellojava.club/article/27
### 12.优化启动白屏
  item name="android:windowDisablePreview">true</item
  
  item name="android:windowIsTranslucent">true</item
### 13.配置VPS及加速
    1.
    wget --no-check-certificate -O shadowsocks.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh

    2.
    chmod +x shadowsocks.sh
    ./shadowsocks.sh 2>&1 | tee shadowsocks.log

    3.
    systemctl stop firewalld.service

    4.
    ssserver -c /etc/shadowsocks.json -d start

    5.
    wget –no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh
     chmod +x bbr.sh
     ./bbr.sh

    6.
    lsmod | grep bbr
### 14.静默安装后用AlarmManager重启应用
      private void alarmRestart() {
          Intent intent = Utils.getApp().getPackageManager().getLaunchIntentForPackage(AppUtils.getAppPackageName());
          PendingIntent restartIntent = PendingIntent.getActivity(Utils.getApp(), 0, intent, PendingIntent.FLAG_ONE_SHOT);
          AlarmManager mgr = (AlarmManager) Utils.getApp().getSystemService(Context.ALARM_SERVICE);
          if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {// 6.0及以上
              mgr.setExactAndAllowWhileIdle(AlarmManager.RTC_WAKEUP, System.currentTimeMillis() + 20000, restartIntent);
  
          } else if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {// 4.4及以上
              mgr.setExact(AlarmManager.RTC_WAKEUP, System.currentTimeMillis() + 20000, restartIntent);
          }
      }
### 15.开源接口地址
    https://www.apiopen.top/
### 16.通过onPreviewFrame获取摄像头明暗程度
      //上次记录的时间戳
      long lastRecordTime = System.currentTimeMillis();
      //上次记录的索引
      int darkIndex = 0;
      //一个历史记录的数组，255是代表亮度最大值
      long[] darkList = new long[]{255, 255, 255, 255};
      //扫描间隔
      int waitScanTime = 300;
      //亮度低的阀值
      int darkValue = 80;
      private boolean isCameraPreviewDark(byte[] data) {
          boolean isDarkEnv = true;
          long cameraLight = 0;
          long currentTime = System.currentTimeMillis();
          if (currentTime - lastRecordTime < waitScanTime) {
              return false;
          }
          lastRecordTime = currentTime;

          int width = camera.getParameters().getPreviewSize().width;
          int height = camera.getParameters().getPreviewSize().height;
          //像素点的总亮度
          long pixelLightCount = 0L;
          //像素点的总数
          long pixeCount = width * height;
          //采集步长，因为没有必要每个像素点都采集，可以跨一段采集一个，减少计算负担，必须大于等于1。
          int step = 10;
          //data.length - allCount * 1.5f的目的是判断图像格式是不是YUV420格式，只有是这种格式才相等
          //因为int整形与float浮点直接比较会出问题，所以这么比
          if (Math.abs(data.length - pixeCount * 1.5f) < 0.00001f) {
              for (int i = 0; i < pixeCount; i += step) {
                  //如果直接加是不行的，因为data[i]记录的是色值并不是数值，byte的范围是+127到—128，
                  // 而亮度FFFFFF是11111111是-127，所以这里需要先转为无符号unsigned long参考Byte.toUnsignedLong()
                  pixelLightCount += ((long) data[i]) & 0xffL;
              }
              //平均亮度
              cameraLight = pixelLightCount / (pixeCount / step);
              //更新历史记录
              int lightSize = darkList.length;
              darkList[darkIndex = darkIndex % lightSize] = cameraLight;
              darkIndex++;

              //判断在时间范围waitScanTime * lightSize内是不是亮度过暗
              for (int i = 0; i < lightSize; i++) {
                  if (darkList[i] > darkValue) {
                      isDarkEnv = false;
                  }
              }

          }
          System.out.println("摄像头环境亮度为 ： " + cameraLight + ",是否是亮度过暗:" + isDarkEnv);

          return isDarkEnv;

      }
  
  

