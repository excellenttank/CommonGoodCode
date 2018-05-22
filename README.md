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
