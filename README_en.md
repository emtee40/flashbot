# flashbot

 ## Source
 * The earliest idea of ​​flashbot comes from the "flash brush robot" (searchable on Taobao). This kind of hardware sells for more than 1,000 on Taobao.  It is used to quickly flash the apk to Android phones, and is often used for app promotion and pre-installation.
 * For batch installation, this 8-port and 16-port equipment is more efficient, but if it is scattered sharing, it is too big and inconvenient to use.
 * There are some softwares on the market with sharing functions, such as Kuaiya, pea pods, etc., but usually two mobile phones are equipped with Kuaiya for sharing. If this software is not installed on the target machine, how can I share it in the past?  It is a very typical chicken-and-egg problem.

 ## solution
 * During development, we generally install apk through adb install.  And the android system itself is based on linux, can we use an android phone to perform adb install for another phone?
 * otg allows our android phone to act as a PC-like host, which can be connected to other devices (such as keyboard, mouse, USB flash drive), of course, can also be connected to other android phones, most mobile phones currently support this technology.
 *   Program

   The following needs to use two android phones, one is the host H and the other is the target D. The host H is required to support OTG.
    * Imperfect solution
        * In the system after android 4.0, the system already carries the adb command, can the command `adb install` be used directly?
        * After connecting the two mobile phones with the otg cable in the field, and then execute `adb devices` in the command line, it is found that there is no target machine in the result. The reason is that `adb devices` actually scans the underlying code in the underlying code  Device handle, and ordinary users do not have permission to access these contents, unless you are running as root (requires the device to be rooted).
        * To root the host, you can run `adb install` to install the apk of host H to target D.
        * At present, some flashing manufacturers in the China Mobile/Unicom business hall use this solution. They usually hold an android pad that supports OTG. This device has been rooted. Use the above solution to install the corresponding operator on your mobile phone.  Assistant or other software.
        * Disadvantages: **root** is required. This is a flaw. Root is both complicated and security risks for ordinary people.
       
    * Perfect solution
      * Android's api actually provides the api of the underlying device. If the call is made through the corresponding api, the system is allowed. At most, an authorization box will pop up automatically, just confirm it.
      * Then the goal is very clear, rewrite adb with the api supported by android, so that you can run `adb install` without permission issues.  Part of the underlying work can be implemented in github, see [cgutman's adblib](https://github.com/cgutman/AdbLib/tree/master/src/com/cgutman/adblib).
      * Modify the above code. In cgutman's adblib, the data is interacted through Socket, abstract it as AdbChannel, and modify the original Socket implementation to TcpChannel, and write UsbChannel (its internal use UsbDeviceConnection, UsbEndpoint,  Standard APIs such as UsbInterface).
      * Then refer to the data format of adb to implement the `adb push` and `adb install` commands.
      * Finally set the app shell, the whole work is completed.
      * work process:
           * Open the adb option on the target machine D (basic cities where domestic users have used computer assistants, please search by yourself)
           * Host H and target D are connected through OTG cable
           * A prompt may pop up on the target machine D, to the effect of whether to trust the host machine H, etc., confirm.  (This process is exactly the same as connecting a computer to an android phone).
           * Host H can choose to be on its own machine
           * Installed apk (if the apk is not cleaned up after installation, the original apk is actually kept somewhere in the system directory),
           * Other apk files on sdcard
           
            Installed on the target machine D (actually call and rewrite `adb push` and `adb install`), and you're done.
      * [Video effects here](http://v.youku.com/v_show/id_XNjg3MzAxOTQ4.html?from=s1.8-1-1.2)
      * Advantages: Only required to support OTG, no root is required.  A 5 yuan otg line can solve the needs of vendors for scattered flashing or occasional sharing, which is excellent value for money.
