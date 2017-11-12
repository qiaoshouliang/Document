# 1.  由于TextView,RelativeLayout(LinearLayout) 默认不具备可点击性的，要让background的selector 起作用有两种方法：
- 一、在代码中给其设置点击事件OnClickListener
- 二、在xml中设置clickable = true
# 2. selector写的不对，不点击时的默认效果应该放在最后面

```
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:drawable="@drawable/bg_smart_device_start_pre" android:state_pressed="true"/>
    <item android:drawable="@drawable/bg_smart_device_start_nol"/>
</selector>
```
