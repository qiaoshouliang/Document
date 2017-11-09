
当Androi Studio编译的时候，遇到如下错误
```
Error:(1, 0) The android gradle plugin version 2.4.0-alpha7 is too old, please update to the latest version.
To override this check from the command line please set the ANDROID_DAILY_OVERRIDE environment variable to "a151637df19e9dbfff417b406bc6cf9cbcb8a8f6"
```

使用命令，并重启AS
> launchctl setenv ANDROID_DAILY_OVERRIDE a151637df19e9dbfff417b406bc6cf9cbcb8a8f6