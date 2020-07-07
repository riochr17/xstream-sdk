# XStream SDK

XStream is live streaming SDK for Android.

### Installation

Minimum SDK version: 18

Target SDK version: 25

On Project Root Build Gradle
```gradle
allprojects {
    repositories {
        ...
        maven { url "https://jitpack.io" }
    }
}
```

On App Module Build Gradle
```gradle
dependencies {
    ...
    implementation 'com.github.riochr17:xstream-sdk:1.0.4'
}
```

### Usage
1. Create GLSurfaceView on layout
```xml
<android.opengl.GLSurfaceView
    android:id="@+id/cameraPreview_surfaceView"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:visibility="gone"
    android:layout_gravity="center" />
```
2. Bind GLSurfaceView
```java
private GLSurfaceView mGLView;
{
    // Configure the GLSurfaceView. This will start the Renderer thread,
    // with an appropriate EGL activity.
    mGLView = (GLSurfaceView) findViewById(R.id.cameraPreview_surfaceView);
    if (mGLView != null) {
        mGLView.setEGLContextClientVersion(2);     // select GLES 2.0
    }
}
```
3. Prepare Service Connection Callback
```java
import android.content.ComponentName;
import android.os.IBinder;
import android.content.ServiceConnection;
import android.hardware.Camera;
import id.xstream.android.broadcaster.LiveVideoBroadcaster;
import id.xstream.android.broadcaster.ILiveVideoBroadcaster;

private ILiveVideoBroadcaster mLiveVideoBroadcaster;
private ServiceConnection mConnection = new ServiceConnection() {
    @Override
    public void onServiceConnected(ComponentName className,
                                   IBinder service) {
        // We've bound to LocalService, cast the IBinder and get LocalService instance
        LiveVideoBroadcaster.LocalBinder binder = (LiveVideoBroadcaster.LocalBinder) service;
        if (mLiveVideoBroadcaster == null) {
            mLiveVideoBroadcaster = binder.getService();
            mLiveVideoBroadcaster.init(YourActivity.this, mGLView);
            mLiveVideoBroadcaster.setAdaptiveStreaming(true);
        }
        mLiveVideoBroadcaster.openCamera(Camera.CameraInfo.CAMERA_FACING_FRONT);
    }

    @Override
    public void onServiceDisconnected(ComponentName arg0) {
        mLiveVideoBroadcaster = null;
    }
};
```
4. Start Service & Bind Connection
```java
import android.opengl.GLSurfaceView;
import id.xstream.android.broadcaster.LiveVideoBroadcaster;

{
    Intent mService = new Intent(this, LiveVideoBroadcaster.class);
    startService(mService);
    bindService(mService, mConnection, 0);
}
```
5. Start Broadcasting
```java
private void startBroadcasting() {
    String STREAMING_URL = "rtmp://your-streaming-url";
    if (mLiveVideoBroadcaster != null) {
        if (!mLiveVideoBroadcaster.isConnected()) {
            new AsyncTask<String, String, Boolean>() {
                @Override
                protected Boolean doInBackground(String... url) {
                    return mLiveVideoBroadcaster.startBroadcasting(url[0]);
                }

                @Override
                protected void onPostExecute(Boolean result) {
                    if (!result) {
                        // something error
                    }
                }
            }.execute(STREAMING_URL);
        }
        else {
            // previous streaming still alive usually caused by network problem
        }
    } else {
        // this should not be happened
    }
}
```
6. Stop Broadcasting
```java
mLiveVideoBroadcaster.stopBroadcasting();
```
7. Unbind Connection
```java
unbindService(mConnection);
```