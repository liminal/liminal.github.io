---
layout: post
title: "Minimum Viable Thing: Android Launcher"
category: "mvt"
tags: minimum-viable-thing, android
---
# What?

What is the least amount of things we need to do in order to make an android
launcher? 

Based on an empty android project with no preexisting activity.

First we need an activity for the home screen, so we add the following

**AndroidManifest.xml:**

```xml
<!-- This goes in AndroidManifest.xml -->

<activity
  android:name=".HomeActivity"
  android:label="Name Of Launcher Here"
  android:theme="@android:style/Theme.Wallpaper.NoTitleBar.Fullscreen"
  android:launchMode="singleTask"
  android:stateNotNeeded="true">
  <intent-filter>
    <action android:name="android.intent.action.MAIN" />
    <category android:name="android.intent.category.HOME" />
    <category android:name="android.intent.category.DEFAULT" />
  </intent-filter>
</activity>
```

## Quick rundown

`android:theme="@android:style/Theme.Wallpaper.NoTitleBar.Fullscreen"`
This makes sure our home screen looks like a proper home screen and not an app.
We get no title bar, no status bar and the current Wallpaper as background.

`android:launchMode="singleTask"`
We only ever want one copy of our launcher running at once

`android:stateNotNeeded="true"`
We don't need to keep track of state when we restart. The base state of the
launcher will always be the base state of the launcher

`<category android:name="android.intent.category.HOME" />`
This is the most important part of the intent-filter clause. It's what makes
a launcher a launcher

We also need a layout for the launcher. For this mvt we'll just use a basic gridview
hardcoded to a 96dp columnWidth

**layout/home_activity.xml:**

```xml
<?xml version="1.0" encoding="utf-8"?>
<GridView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/app_grid"
    android:columnWidth="96dp"
    android:numColumns="auto_fit"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

And for displaying the apps we go for a LinearLayout with an ImageView for the
icon and a TextView for the app name. 

**layout/app_item.xml:**

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >

    <ImageView
        android:id="@+id/app_item_icon"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />

    <TextView
        android:id="@+id/app_item_label"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />
</LinearLayout>
```

Now that we have all the resources we need the only thing left is the actual code :)

What we want to achieve is a grid off app icons that you can click on to start the apps.

First thing is we want an adapter for the grid. 

**ResolveInfoAdapter.java:**

```java
public class ResolveInfoAdapter extends ArrayAdapter<ResolveInfo> {

    private final PackageManager mPackageManager;

    public ResolveInfoAdapter(Context context) {
        super(context, R.layout.app_item, listApps(context));
        mPackageManager = context.getPackageManager();
    }

    private static List<ResolveInfo> listApps(Context context) {
        Intent intent = new Intent(Intent.ACTION_MAIN, null);
        intent.addCategory(Intent.CATEGORY_LAUNCHER);
        return context.getPackageManager().queryIntentActivities(intent, 0);
    }

    public void launchApp(Context launchContext, int position) {
        String pkgName      = getItem(position).activityInfo.packageName;
        Intent launchIntent = mPackageManager.getLaunchIntentForPackage(pkgName);

        launchContext.startActivity(launchIntent);
    }

    @Override public View getView(int position, View convertView, ViewGroup parent) {
        if (convertView == null) {
            convertView = LayoutInflater.from(getContext())
                                        .inflate(R.layout.app_item, parent, false);
        }

        ResolveInfo info = getItem(position);

        ImageView icon  = (ImageView) convertView.findViewById(R.id.app_item_icon);
        icon.setImageDrawable(info.activityInfo.loadIcon(mPackageManager));

        TextView  label = (TextView) convertView.findViewById(R.id.app_item_label);
        label.setText(info.loadLabel(mPackageManager));

        return convertView;
    }
}
```

The interesting parts in the above is `listApps(Context)` and `launchApp(Context, int)`
that actually gets the information about the available apps on the device
the rest is a standard oversimplified ArrayAdapter

And then at last we put it all together in 

**HomeActivity.java:**

```java
public class HomeActivity extends Activity {

    @Override protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.home_activity);

        final ResolveInfoAdapter infoAdapter = new ResolveInfoAdapter(this);

        final GridView appGrid = (GridView) findViewById(R.id.app_grid);
        appGrid.setAdapter(infoAdapter);
        appGrid.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
                infoAdapter.launchApp(getApplicationContext(), position);
            }
        });
    }
}
```

And with that we have an ugly but functional Minimum Viable Launcher. Hooray

