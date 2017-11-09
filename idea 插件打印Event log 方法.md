
# 打印 Event log 的方式

## 方法如下：
```
public class EventLogger {
    private static final String GROUP_ID = "Log";//The unique group id where "Event Log" could use to group your messages together.
    private static final String TITLE = "Log Event Log";//The title on Balloon
    /**
     * Print log to "Event Log"
     */
    public static void log(String msg) {
        Notification notification = new Notification(GROUP_ID, TITLE, msg, NotificationType.INFORMATION);//build a notification
        //notification.hideBalloon();//didn't work
        Notifications.Bus.notify(notification);//use the default bus to notify (application level)
        Balloon balloon = notification.getBalloon();
        if (balloon != null) {//fix: #20 潜在的NPE
            balloon.hide(true);//try to hide the balloon immediately.
        }
    }
}
```
## 使用方式
` EventLogger.log("String log = \"adsdf \";");`

如下图在Event Log中就打印出`Log Event Log: String log = "adsdf ";`

![](https://ws4.sinaimg.cn/large/006tNc79ly1fhy6kqc37nj30wm079t9u.jpg)
