# 记录自己遇到的问题，及解决问题的方式
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
