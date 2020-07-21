类似于LeakCanary，打开app的同时在桌面上再打开一个页面

	<activity
            android:name=".DisplayTrackActivity"
            android:icon="@drawable/icon_track"
            android:label="Horadric"
            android:launchMode="singleTask"
            android:screenOrientation="portrait"
            android:taskAffinity="com.quxiaopeng.track"
            android:theme="@style/Theme.AppCompat.DayNight.DarkActionBar"/>

        <activity-alias
            android:name="com.example.quxiaopeng.testlib.DisplayTrackLaunchActivity"
            android:icon="@drawable/icon_track"
            android:label="Horadric"
            android:launchMode="singleTask"
            android:screenOrientation="portrait"
            android:targetActivity=".DisplayTrackActivity"
            android:taskAffinity="com.kuaikan.track"
            android:theme="@style/Theme.AppCompat.DayNight.DarkActionBar"
            android:enabled="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity-alias>
        
使用activity-alias，打开app的同时会在桌面上打开targetActivity
