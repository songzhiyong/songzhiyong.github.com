---
layout: post
title: "Android Launcher Notes 1"
date: 2014-07-15 21:14
comments: true
categories: android
---
####1. 入口

```java
 public void onReceive(Context context, Intent data) {
        //接受程序创建shortcut的广播
        if (!ACTION_INSTALL_SHORTCUT.equals(data.getAction())) {
            return;
        }
        Intent intent = data.getParcelableExtra(Intent.EXTRA_SHORTCUT_INTENT);
        if (intent == null) {
            return;
        }
        // This name is only used for comparisons and notifications, so fall back to activity name
        // if not supplied
        String name = data.getStringExtra(Intent.EXTRA_SHORTCUT_NAME);
        if (name == null) {
            try {
                PackageManager pm = context.getPackageManager();
                ActivityInfo info = pm.getActivityInfo(intent.getComponent(), 0);
                name = info.loadLabel(pm).toString();
            } catch (PackageManager.NameNotFoundException nnfe) {
                return;
            }
        }
        // Queue the item up for adding if launcher has not loaded properly yet
        boolean launcherNotLoaded = LauncherModel.getCellCountX() <= 0 ||
                LauncherModel.getCellCountY() <= 0;
        PendingInstallShortcutInfo info = new PendingInstallShortcutInfo(data, name, intent);
        if (mUseInstallQueue || launcherNotLoaded) {
            //进入队列
            mInstallQueue.add(info);
        } else {
            //创建
            processInstallShortcut(context, info);
        }
    }


```

####2. 处理快捷方式Info,并创建
```java

private static void processInstallShortcut(Context context,
            PendingInstallShortcutInfo pendingInfo) {
        String spKey = LauncherApplication.getSharedPreferencesKey();
        SharedPreferences sp = context.getSharedPreferences(spKey, Context.MODE_PRIVATE);
        final Intent data = pendingInfo.data;
        final Intent intent = pendingInfo.launchIntent;
        final String name = pendingInfo.name;
        // Lock on the app so that we don't try and get the items while apps are being added
        LauncherApplication app = (LauncherApplication) context.getApplicationContext();
        final int[] result = {INSTALL_SHORTCUT_SUCCESSFUL};
        boolean found = false;
        synchronized (app) {
            final ArrayList<ItemInfo> items = LauncherModel.getItemsInLocalCoordinates(context);
            final boolean exists = LauncherModel.shortcutExists(context, name, intent);
            // Try adding to the workspace screens incrementally, starting at the default or center
            // screen and alternating between +1, -1, +2, -2, etc. (using ~ ceil(i/2f)*(-1)^(i-1))
            final int screen = Launcher.DEFAULT_SCREEN;
            for (int i = 0; i < (2 * Launcher.SCREEN_COUNT) + 1 && !found; ++i) {
                int si = screen + (int) ((i / 2f) + 0.5f) * ((i % 2 == 1) ? 1 : -1);
                if (0 <= si && si < Launcher.SCREEN_COUNT) {
                    found = installShortcut(context, data, items, name, intent, si, exists, sp,
                            result);
                }
            }
        }
        // We only report error messages (duplicate shortcut or out of space) as the add-animation
        // will provide feedback otherwise
        if (!found) {
            if (result[0] == INSTALL_SHORTCUT_NO_SPACE) {
                Toast.makeText(context, context.getString(R.string.completely_out_of_space),
                        Toast.LENGTH_SHORT).show();
            } else if (result[0] == INSTALL_SHORTCUT_IS_DUPLICATE) {
                Toast.makeText(context, context.getString(R.string.shortcut_duplicate, name),
                        Toast.LENGTH_SHORT).show();
            }
        }
    }
    private static boolean installShortcut(Context context, Intent data, ArrayList<ItemInfo> items,
            String name, Intent intent, final int screen, boolean shortcutExists,
            final SharedPreferences sharedPrefs, int[] result) {
        //寻找位置
        ...
                    // Update the Launcher db更新数据库
                    LauncherApplication app = (LauncherApplication) context.getApplicationContext();
                    ShortcutInfo info = app.getModel().addShortcut(context, data,
                            LauncherSettings.Favorites.CONTAINER_DESKTOP, screen,
                            tmpCoordinates[0], tmpCoordinates[1], true);
        ...         
        return false;
    }
```

####3. LauncherModel.addShortcut(...) --->  addItemToDatabase(... notify=true)

####4. LauncherApplication   ContentObserver  onChange(...)
```java
        mModel.resetLoadedState(false, true);
        mModel.startLoaderFromBackground();
```
####5. LauncherModel
```java
 /**
   * When the launcher is in the background, it's possible for it to miss
   * paired configuration changes. So whenever we trigger the loader from the
   * background tell the launcher that it needs to re-run the loader when it
   * comes back instead of doing it now.
   */
  public void startLoaderFromBackground() {
    ...
  }
```

