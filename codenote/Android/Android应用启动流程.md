---
title: Android 应用启动流程梳理
date: 2021-1-17 21:16:01
categories: codenote
tags: Android
---

Android 应用启动流程梳理（基于Android7.1.1）
<!--more-->

![aaa](Android应用启动流程.assets/aaa.png)

### 涉及的几个重要类

- ActivityThread.java

  ```java
  /**
   * This manages the execution of the main thread in an
   * application process, scheduling and executing activities,
   * broadcasts, and other operations on it as the activity
   * manager requests.
   *
   * {@hide}
   */
   
   //关键类成员
   final ApplicationThread mAppThread = new ApplicationThread();
   final H mH = new H();
   Instrumentation mInstrumentation;
   final ArrayMap<String, WeakReference<LoadedApk>> mPackages = new ArrayMap<String, WeakReference<LoadedApk>>();
   //关键方法
   public static void main(String[] args){}
   private void attach(boolean system) {}
   public final void bindApplication(String processName, ApplicationInfo appInfo,
                  List<ProviderInfo> providers, ComponentName instrumentationName,
                  ProfilerInfo profilerInfo, Bundle instrumentationArgs,
                  IInstrumentationWatcher instrumentationWatcher,
                  IUiAutomationConnection instrumentationUiConnection, int debugMode,
                  boolean enableBinderTracking, boolean trackAllocation,
                  boolean isRestrictedBackupMode, boolean persistent, Configuration config,
                  CompatibilityInfo compatInfo, Map<String, IBinder> services, Bundle coreSettings) {
              AppBindData data = new AppBindData();
              data.processName = processName;
              data.appInfo = appInfo;
              data.providers = providers;
              data.instrumentationName = instrumentationName;
              data.instrumentationArgs = instrumentationArgs;
              data.instrumentationWatcher = instrumentationWatcher;
              data.instrumentationUiAutomationConnection = instrumentationUiConnection;
              data.debugMode = debugMode;
              data.enableBinderTracking = enableBinderTracking;
              data.trackAllocation = trackAllocation;
              data.restrictedBackupMode = isRestrictedBackupMode;
              data.persistent = persistent;
              data.config = config;
              data.compatInfo = compatInfo;
              data.initProfilerInfo = profilerInfo;
              sendMessage(H.BIND_APPLICATION, data); 
   }
   private void handleBindApplication(AppBindData data) {}
  ```

- ActivityManagerService.java

  ```java
  //关键方法
  public final void attachApplication(IApplicationThread thread) 
  ```

- ApplicationLoaders.java

  ```java
   //关键变量
   private final ArrayMap<String, ClassLoader> mLoaders = new ArrayMap<String, ClassLoader>();//默认是PathClassLoader
   private static final ApplicationLoaders gApplicationLoaders = new ApplicationLoaders();
   //关键方法
   public static ApplicationLoaders getDefault() //被LoaderApk的createOrUpdateClassLoaderLocked方法调用
  ```

  

- LoadedApk.java

  ```java
  /**
   * Local state maintained about a currently loaded .apk.
   *一个应用程序对应一个LoadedApk对象。它用来保存当前加载的APK包的各种信息，包括app安装路径、资源路径、用户数据保存路径、使用的类加载器、Application信  息等。
   * @hide
   */
  //关键变量
   private ClassLoader mClassLoader;
   private Application mApplication;
   private final ActivityThread mActivityThread
   private final ClassLoader mBaseClassLoader;
   //关键方法
   public Application makeApplication(boolean forceDefaultAppClass,Instrumentation instrumentation) {}
   public ClassLoader getClassLoader()
   private void createOrUpdateClassLoaderLocked(List<String> addedPaths)//被getClassLoader调用
  ```

  

- Instrumentation类

  ```java
  /**
   * Base class for implementing application instrumentation code.  When running
   * with instrumentation turned on, this class will be instantiated for you
   * before any of the application code, allowing you to monitor all of the
   * interaction the system has with the application.  An Instrumentation
   * implementation is described to the system through an AndroidManifest.xml's
   * &lt;instrumentation&gt; tag.
   */
  
  //关键变量
  private ActivityThread mThread = null;
  private Context mInstrContext;
  private Context mAppContext;
  //关键方法
  public Application newApplication(ClassLoader cl, String className, Context context) //被LoadedApk的makeApplication方法调用
  static public Application newApplication(Class<?> clazz, Context context)
  ```

  

参考：https://shuwoom.com/?p=142