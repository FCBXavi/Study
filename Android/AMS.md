ActivityManagerService
==================================
在SystemServer执行run方法时被创建，并运行在独立的进程中，SystemServer管理Android中所有的系统服务，这些系统服务的生命周期回调由SystemServer负责。          
    
创建AMS之前，在SystemServer的run方法中，调用了createSystemContext()方法。
    
	//Initialize the system context.
	createSystemContext();
	//Create the system service manager.
	mSystemServiceManager = new SystemServiceManager(mSystemContext);
	
	private void createSystemContext() {
		 //创建ActivityThread的实例，会创建出系统对应的Context对象
        ActivityThread activityThread = ActivityThread.systemMain();
        //取出上面创建的Context对象，保存在mSystemContext中
        mSystemContext = activityThread.getSystemContext();
        //设置系统主题
        mSystemContext.setTheme(DEFAULT_SYSTEM_THEME);
    }
    
    public static ActivityThread systemMain() {
        // The system process on low-memory devices do not get to use hardware
        // accelerated drawing, since this can add too much overhead to the
        // process.
        if (!ActivityManager.isHighEndGfx()) {
            ThreadedRenderer.disable(true);
        } else {
            ThreadedRenderer.enableForegroundTrimming();
        }
        //创建ActivityThread对象
        ActivityThread thread = new ActivityThread();
        thread.attach(true);
        return thread;
    }
    //ActivityThread成员变量
    //定义了AMS与应用通信的接口
    final ApplicationThread mAppThread = new ApplicationThread();
    final Looper mLooper = Looper.myLooper();
    final H mH = new H();
    final ArrayMap<IBinder, ActivityClientRecord> mActivities = new ArrayMap<>();
    final ArrayMap<IBinder, Service> mServices = new ArrayMap<>();
    final ArrayList<Application> mAllApplications = new ArrayList<Application>();
    
ApplicationThread
------------------------------------------
ApplicationThread是ActivityThread的一个成员变量，该类继承自ApplicationThreadNative，ApplicationThreadNative继承自Binder，实现了IApplicationThread接口，内部有一个ApplicationThreadProxy类，也实现了IApplicationThread接口。IApplicationThread接口定义了AMS和应用进程之间的交互函数。        当AMS和应用进程通信时，ApplicationThread将作为Binder通信的服务端，AMS通过ApplicationThreadNative获取应用进程对应的ApplicationThreadProxy对象，通过该对象，将调用信息通过Binder传递到ActivityThread中的ApplicationThread对象。这个后面再分析。

attach方法
------------------------------------------
上面初始化完成ActivityThread后，调用了thread.attach(boolean system)方法，传入true表示该ActivityThread是系统进程的ActivityThread。
		
	private void attach(boolean system) {
		 //用一个静态变量保存该ActivityThread，AMS中的大量操作依赖这个ActivityThread
        sCurrentActivityThread = this;
        mSystemThread = system;
        if (!system) {
        	//应用进程的处理逻辑
        } else {
        	// 系统进程的处理逻辑
            // Don't set application object here -- if the system crashes,
            // we can't display an alert, we just want to die die die.
            // 设置DDMS中看到的SystemServer的进程名为"system_process"
            android.ddm.DdmHandleAppName.setAppName("system_process",
                    UserHandle.myUserId());
            try {
            	//创建ActivityThread中的重要成员，Instrumation、Context和Application
                mInstrumentation = new Instrumentation();
                ContextImpl context = ContextImpl.createAppContext(
                        this, getSystemContext().mPackageInfo);
                mInitialApplication = context.mPackageInfo.makeApplication(true, null);
                mInitialApplication.onCreate();
            } catch (Exception e) {
                throw new RuntimeException(
                        "Unable to instantiate Application():" + e.toString(), e);
            }
        }

		//系统进程和非系统进程都会执行的逻辑
        // add dropbox logging to libcore
        DropBox.setReporter(new DropBoxReporter());
		//注册Configuration发生变化的回调
        ViewRootImpl.addConfigCallback(new ComponentCallbacks2() {
            @Override
            public void onConfigurationChanged(Configuration newConfig) {
            }
            @Override
            public void onLowMemory() {
            }
            @Override
            public void onTrimMemory(int level) {
            }
        });
    }

对于系统进程而言，attach方法的重要作用就是创建了Instrumation、Context和Application          

至此，createSystemContext执行完毕，执行完毕后：      
1. 得到了一个ActivityThread对象，代表当前进程(系统进程)的主线程。      
2. 得到了一个Context对象，对于SystemServer而言，它包含的Application运行环境与framework-res.apk有关。      


AMS初始化
-------------------------------------------
创建完Android运行环境之后，调用了startBootstrapServices()方法，在其中通过反射的方法创建了AMS

	private void startBootstrapServices() {
        Installer installer = mSystemServiceManager.startService(Installer.class);

        // Activity manager runs the show.
        // 启动AMS的内部类Lifecycle，因为AMS没有继承SystemService，通过Lifecycle让AMS可以像SystemService一样被SystemServiceManager通过反射启动。
        mActivityManagerService = mSystemServiceManager.startService(
                ActivityManagerService.Lifecycle.class).getService();
        mActivityManagerService.setSystemServiceManager(mSystemServiceManager);
        mActivityManagerService.setInstaller(installer);

        ...
    }
    
    public static final class Lifecycle extends SystemService {
        private final ActivityManagerService mService;
		//SystemServer的SystemContext
        public Lifecycle(Context context) {
            super(context);
            mService = new ActivityManagerService(context);
        }

        @Override
        public void onStart() {
            mService.start();
        }

        public ActivityManagerService getService() {
            return mService;
        }
    }
    
    //AMS的构造函数
    public ActivityManagerService(Context systemContext) {
    	//上下文与SystemServer一致
        mContext = systemContext;
        mFactoryTest = FactoryTest.getMode();
        //取出ActivityThread的静态变量sCurrentActivityThread
        //此时为SystemServer中创建的ActivityThread
        mSystemThread = ActivityThread.currentActivityThread();

        mHandlerThread = new ServiceThread(TAG,
                android.os.Process.THREAD_PRIORITY_FOREGROUND, false /*allowIo*/);
        mHandlerThread.start();
        //处理AMS中消息的主力
        mHandler = new MainHandler(mHandlerThread.getLooper());
        //对应Android中的UIHandler
        mUiHandler = new UiHandler();

        /* static; one-time init here */
        if (sKillHandler == null) {
            sKillThread = new ServiceThread(TAG + ":kill",
                    android.os.Process.THREAD_PRIORITY_BACKGROUND, true /* allowIo */);
            sKillThread.start();
            //用于接收消息，杀死进程
            sKillHandler = new KillHandler(sKillThread.getLooper());
        }

		 //创建两个BroadcastQueue前台的超时时长是10s，后台的是60s
        mFgBroadcastQueue = new BroadcastQueue(this, mHandler,
                "foreground", BROADCAST_FG_TIMEOUT, false);
        mBgBroadcastQueue = new BroadcastQueue(this, mHandler,
                "background", BROADCAST_BG_TIMEOUT, true);
        mBroadcastQueues[0] = mFgBroadcastQueue;
        mBroadcastQueues[1] = mBgBroadcastQueue;
        
		 //创建变量，用于存储信息
        mServices = new ActiveServices(this);
        mProviderMap = new ProviderMap(this);
        mAppErrors = new AppErrors(mContext, this);

        // TODO: Move creation of battery stats service outside of activity manager service.
        //BBS初始化
        File dataDir = Environment.getDataDirectory();
        File systemDir = new File(dataDir, "system");
        systemDir.mkdirs();
        mBatteryStatsService = new BatteryStatsService(systemDir, mHandler);
        mBatteryStatsService.getActiveStatistics().readLocked();
        mBatteryStatsService.scheduleWriteToDisk();
        mOnBattery = DEBUG_POWER ? true
                : mBatteryStatsService.getActiveStatistics().getIsOnBattery();
        mBatteryStatsService.getActiveStatistics().setCallback(this);

        mProcessStats = new ProcessStatsService(this, new File(systemDir, "procstats"));
        //启动Android权限检查服务，并注册对应的回调接口。
        mAppOpsService = new AppOpsService(new File(systemDir, "appops.xml"), mHandler);
        mAppOpsService.startWatchingMode(AppOpsManager.OP_RUN_IN_BACKGROUND, null,
                new IAppOpsCallback.Stub() {
                    @Override public void opChanged(int op, int uid, String packageName) {
                        if (op == AppOpsManager.OP_RUN_IN_BACKGROUND && packageName != null) {
                            if (mAppOpsService.checkOperation(op, uid, packageName)
                                    != AppOpsManager.MODE_ALLOWED) {
                                runInBackgroundDisabled(uid);
                            }
                        }
                    }
                });

        mGrantFile = new AtomicFile(new File(systemDir, "urigrants.xml"));

        mUserController = new UserController(this);

        GL_ES_VERSION = SystemProperties.getInt("ro.opengles.version",
            ConfigurationInfo.GL_ES_VERSION_UNDEFINED);

        if (SystemProperties.getInt("sys.use_fifo_ui", 0) != 0) {
            mUseFifoUiScheduling = true;
        }

        mTrackingAssociations = "1".equals(SystemProperties.get("debug.track-associations"));

        mConfiguration.setToDefaults();
        mConfiguration.setLocales(LocaleList.getDefault());

        mConfigurationSeq = mConfiguration.seq = 1;
        mProcessCpuTracker.init();

        mCompatModePackages = new CompatModePackages(this, systemDir, mHandler);
        mIntentFirewall = new IntentFirewall(new IntentFirewallInterface(), mHandler);
        mStackSupervisor = new ActivityStackSupervisor(this);
        mActivityStarter = new ActivityStarter(this, mStackSupervisor);
        mRecentTasks = new RecentTasks(this, mStackSupervisor);

		 //创建线程统计CPU使用情况
        mProcessCpuThread = new Thread("CpuTracker") {
            @Override
            public void run() {
                while (true) {
                    try {
                        try {
                            synchronized(this) {
                                final long now = SystemClock.uptimeMillis();
                                long nextCpuDelay = (mLastCpuTime.get()+MONITOR_CPU_MAX_TIME)-now;
                                long nextWriteDelay = (mLastWriteTime+BATTERY_STATS_TIME)-now;
                                //Slog.i(TAG, "Cpu delay=" + nextCpuDelay
                                //        + ", write delay=" + nextWriteDelay);
                                if (nextWriteDelay < nextCpuDelay) {
                                    nextCpuDelay = nextWriteDelay;
                                }
                                if (nextCpuDelay > 0) {
                                    mProcessCpuMutexFree.set(true);
                                    this.wait(nextCpuDelay);
                                }
                            }
                        } catch (InterruptedException e) {
                        }
                        updateCpuStatsNow();
                    } catch (Exception e) {
                        Slog.e(TAG, "Unexpected exception collecting process stats", e);
                    }
                }
            }
        };
		 //加入Watchdog监控。
        Watchdog.getInstance().addMonitor(this);
        Watchdog.getInstance().addThread(mHandler);
    }

主要是初始化了一些变量

AMS的start方法
----------------------------------------

	private void start() {
		//统计前的抚慰工作
        Process.removeAllProcessGroups();
        //开始监控CPU的使用情况
        mProcessCpuThread.start();

		//注册服务
        mBatteryStatsService.publish(mContext);
        mAppOpsService.publish(mContext);
        Slog.d("AppOps", "AppOpsService published");
        LocalServices.addService(ActivityManagerInternal.class, new LocalService());
    }
    
主要是启动CPU监控线程，统计不同进程使用CPU的情况         
发布一些服务，如BatteryStatsService、AppOpsService(权限管理相关)和本地实现的继承ActivityManagerInternal的服务。

将SystemServer纳入AMS管理体系
----------------------------------------
startBootstrapServices()方法创建了AMS实例后

		//执行了该方法
		// Set up the Application instance for the system process and get started.
        mActivityManagerService.setSystemProcess();
        
        
        public void setSystemProcess() {
        try {
        	//向ServiceManager注册服务
            ServiceManager.addService(Context.ACTIVITY_SERVICE, this, true);
            //注册进程统计信息服务
            ServiceManager.addService(ProcessStats.SERVICE_NAME, mProcessStats);
            //用于打印内存信息用的
            ServiceManager.addService("meminfo", new MemBinder(this));
            //用于输出进程使用硬件渲染方面的信息
            ServiceManager.addService("gfxinfo", new GraphicsBinder(this));
            //用于输出数据库相关信息
            ServiceManager.addService("dbinfo", new DbBinder(this));
            if (MONITOR_CPU_USAGE) {
            //用于输出进程的CPU使用情况
                ServiceManager.addService("cpuinfo", new CpuBinder(this));
            }
            //注册权限管理服务
            ServiceManager.addService("permission", new PermissionController(this));
            //注册获取进程信息的服务
            ServiceManager.addService("processinfo", new ProcessInfoService(this));
            //1.向PKMS查询package名为"android"应用的ApplicationInfo
            ApplicationInfo info = mContext.getPackageManager().getApplicationInfo(
                    "android", STOCK_PM_FLAGS | MATCH_SYSTEM_ONLY);
            //2.调用installSystemApplicationInfo
            mSystemThread.installSystemApplicationInfo(info, getClass().getClassLoader());
				
			//以下与AMS进程管理有关	
            synchronized (this) {
                ProcessRecord app = newProcessRecordLocked(info, info.processName, false, 0);
                app.persistent = true;
                app.pid = MY_PID;
                app.maxAdj = ProcessList.SYSTEM_ADJ;
                app.makeActive(mSystemThread.getApplicationThread(), mProcessStats);
                synchronized (mPidsSelfLocked) {
                    mPidsSelfLocked.put(app.pid, app);
                }
                updateLruProcessLocked(app, false, null);
                updateOomAdjLocked();
            }
        } catch (PackageManager.NameNotFoundException e) {
            throw new RuntimeException(
                    "Unable to find android system package", e);
        }
    }
    
该方法主要的功能：         
1. 注册一些服务        
2. 获取package名为"android"的应用的ApplicationInfo        
3. 调用ActivityThread的installSystemApplicationInfo         
4. AMS进程管理相关操作          

先分析到这里，看不下去了。。。。后续继续从SystemServer的run方法往下走，看一看startCoreServices()方法和startOtherServices()方法。  


总结一下AMS的启动过程
------------------------------------------
1. 创建出SystemServer进程的Android运行环境。       
主要是创建出对应的ActivityThread和ContextImpl，AMS后续的操作依赖于SystemServer在此创建出的运行环境。      
2. 完成AMS的初始化和启动。     
调用AMS的构造函数和start函数，完成初始化工作。       
3. 将SystemServer进程纳入到AMS的管理体系中。      
AMS加载了SystemServer中framework-res.apk的信息，并启动和注册了SettingsProvider.apk        
4. 开始执行AMS启动完毕后才能执行的工作。     
AMS通过调用systemReady函数，通知系统中的其它服务和进程，可以进行对应工作了。 
在这个过程中，值得我们关注的是：Home Activity被启动了。当该Activity被加载完成后，最终会触发ACTION_BOOT_COMPLETED广播。        

AMS和ActivityManager通信
----------------------------------------
ActivityManagerNative继承自Binder类，有远程通信的条件，实现了IActivityManager接口，能够得到ActivityManager管理关于内存、任务等内部信息，AMS作为AMN的子类，也有这些特性。        

ActivityManager中的方法被调用时，调用了ActivityManagerNative.getDefault()的方法。

	/**
     * Retrieve the system's default/global activity manager.
     */
    static public IActivityManager getDefault() {
        return gDefault.get();
    }
    //ServiceManager是系统提供的服务管理类，所有Service都通过他被注册和管理，通过getService()方法能获取到ActivityManager与AMS远程通信Binder对象
    private static final Singleton<IActivityManager> gDefault = new Singleton<IActivityManager>() {
        protected IActivityManager create() {
            IBinder b = ServiceManager.getService("activity");
            if (false) {
                Log.v("ActivityManager", "default service binder = " + b);
            }
            //看一下这个方法
            IActivityManager am = asInterface(b);
            if (false) {
                Log.v("ActivityManager", "default service = " + am);
            }
            return am;
        }
    };
    
    //拿到AMS的Binder对象以后，交给ActivityManagerProxy来代理，处理Activity和AMS的通信
    static public IActivityManager asInterface(IBinder obj) {
        if (obj == null) {
            return null;
        }
        IActivityManager in =
            (IActivityManager)obj.queryLocalInterface(descriptor);
        if (in != null) {
            return in;
        }

        return new ActivityManagerProxy(obj);
    }
    
    //也实现了IActivityManager接口，当客户端（ActivityManager）发起向服务端(AMS)的请求时，会把请求数据打包，然后由ActivitytManager的远程通信binder对象通过transact()方法提交数据，然后再把数据写出返回给binder对象。AMS在自己的进程中获得ActivityManager发来的数据信息，从而完成对Android系统组件生命周期的调用。
    class ActivityManagerProxy implements IActivityManager {
    		public ActivityManagerProxy(IBinder remote) {
        		mRemote = remote;
    		}

    		public IBinder asBinder() {
        		return mRemote;
    		}
    	}
    	
    	
    	
    	
    	
Activity启动过程
--------------------------------------------
调用Activity的startActivity方法后，最终调用了其startActivityForResult方法(requestCode是-1)，startActivityForResult调用了mInstrumentation.execStartActivity方法。     

	public ActivityResult execStartActivity(
            Context who, IBinder contextThread, IBinder token, Activity target,
            Intent intent, int requestCode, Bundle options) {
        ...
        try {
            intent.migrateExtraStreamToClipData();
            intent.prepareToLeaveProcess(who);
            //调用了ActivityManagerNative.getDefault()的startActivity方法
            int result = ActivityManagerNative.getDefault()
                .startActivity(whoThread, who.getBasePackageName(), intent,
                        intent.resolveTypeIfNeeded(who.getContentResolver()),
                        token, target != null ? target.mEmbeddedID : null,
                        requestCode, 0, null, options);
            checkStartActivityResult(result, intent);
        } catch (RemoteException e) {
            throw new RuntimeException("Failure from system", e);
        }
        return null;
    }
    
内部调用了ActivityManagerNative.getDefault()的startActivity方法，ActivityManagerNative.getDefault()返回的是一个ActivityManagerProxy对象。继承自IActivityManager，ActivityManagerNative也实现了该接口，并继承自Binder。       

ActivityManagerNative相当于Binder客户端，而AMS相当于服务端，调用其方法的时候，底层通过Binder driver将请求传递给server端，在server端执行具体的接口逻辑，Binder机制是单向的。          

这里涉及到了Binder的数据传输机制，后续再研究一下这块，数据传输完毕后调用了SystemServer进程的AMS的startActivity方法            

AMS中的startActivity方法   
首先让任务栈中的Activity执行onPause方法，通过一个IApplicationThread对象执行schedulePauseActivity方法，这里又是使用的Binder传输数据，这里的IApplicationThread 相当于Binder的Client端，ActivityThread中的ApplicationThread相当于Binder的Server端，通过执行IApplicationThread的schedulePauseActivity方法，实际上执行了ApplicationThread的schedulePauseActivity方法。ApplicationThread中的此方法通过Handler发送一个Message，Handler收到此消息后调用了栈顶Activity的onPause方法

	public final int startActivity(IApplicationThread caller, String callingPackage,
            Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode,
            int startFlags, ProfilerInfo profilerInfo, Bundle bOptions) {
        return startActivityAsUser(caller, callingPackage, intent, resolvedType, resultTo,
                resultWho, requestCode, startFlags, profilerInfo, bOptions,
                UserHandle.getCallingUserId());
    }
    
    //调用了startActivityAsUser方法
    @Override
    public final int startActivityAsUser(IApplicationThread caller, String callingPackage,
            Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode,
            int startFlags, ProfilerInfo profilerInfo, Bundle bOptions, int userId) {
        enforceNotIsolatedCaller("startActivity");
        userId = mUserController.handleIncomingUser(Binder.getCallingPid(), Binder.getCallingUid(),
                userId, false, ALLOW_FULL_ONLY, "startActivity", null);
        // TODO: Switch to user app stacks here.
        return mActivityStarter.startActivityMayWait(caller, -1, callingPackage, intent,
                resolvedType, null, null, resultTo, resultWho, requestCode, startFlags,
                profilerInfo, null, null, bOptions, false, userId, null, null);
    }
    //调用了mActivityStarter.startActivityMayWait方法
     final int startActivityMayWait(IApplicationThread caller, int callingUid,
            String callingPackage, Intent intent, String resolvedType,
            IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
            IBinder resultTo, String resultWho, int requestCode, int startFlags,
            ProfilerInfo profilerInfo, IActivityManager.WaitResult outResult, Configuration config,
            Bundle bOptions, boolean ignoreTargetSecurity, int userId,
            IActivityContainer iContainer, TaskRecord inTask) {
				...
            int res = startActivityLocked(caller, intent, ephemeralIntent, resolvedType,
                    aInfo, rInfo, voiceSession, voiceInteractor,
                    resultTo, resultWho, requestCode, callingPid,
                    callingUid, callingPackage, realCallingPid, realCallingUid, startFlags,
                    options, ignoreTargetSecurity, componentSpecified, outRecord, container,
                    inTask);
            return res;
    }
    //调用了startActivityLocked方法
    final int startActivityLocked(IApplicationThread caller, Intent intent, Intent ephemeralIntent,
            String resolvedType, ActivityInfo aInfo, ResolveInfo rInfo,
            IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
            IBinder resultTo, String resultWho, int requestCode, int callingPid, int callingUid,
            String callingPackage, int realCallingPid, int realCallingUid, int startFlags,
            ActivityOptions options, boolean ignoreTargetSecurity, boolean componentSpecified,
            ActivityRecord[] outActivity, ActivityStackSupervisor.ActivityContainer container,
            TaskRecord inTask) {
        int err = ActivityManager.START_SUCCESS;
        err = startActivityUnchecked(r, sourceRecord, voiceSession, voiceInteractor, startFlags,
                    true, options, inTask);
        return err;
    }
    //这个方法构造了AMS端的Activity对象ActivityRecord，根据Activity的启动模式执行相关方法，最后调用了startActivityUnchecked方法
    private int startActivityUnchecked(final ActivityRecord r, ActivityRecord sourceRecord,
            IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
            int startFlags, boolean doResume, ActivityOptions options, TaskRecord inTask) {
             ActivityStack.logStartActivity(
                EventLogTags.AM_CREATE_ACTIVITY, mStartActivity, mStartActivity.task);
        mTargetStack.mLastPausedActivity = null;
        mTargetStack.startActivityLocked(mStartActivity, newTask, mKeepCurTransition, mOptions);
    }
    //这个方法执行了不同启动模式不同栈的处理，最后调用startActivityLocked方法
    