## 利用Appium+python-androidviewclient简单搞一下微信

### 前言

本篇技术分享纯属娱乐，代码有风险，技术需谨慎，一切后果自己承担哦

### 实现效果

从官网下载的微信，登录微信后，给另外一个微信账号发一条消息，消息内容为iloveu，挺简单的逻辑，全自动代码实现，无人为参与。效果如下图：

<img src="https://ae01.alicdn.com/kf/U8fdf9a9193d148daa020ce45efe15214G.jpg" alt="1621326672_adb" style="zoom:50%;" />



### 实现

- 有java 环境（不介绍）

- 有adb 环境（不介绍）

- 下载并安装appium 地址：https://github.com/Oslanka/appium-desktop

- 配置关键字段并启动

  ```json
  {
    "platformName": "android",
    "deviceName": "MYVDU15629007350"
  }
  ```

- 抓取元素信息

  <img src="https://ae01.alicdn.com/kf/U5c0422afcc7346808f59898fae61117d4.jpg" alt="wechat5" style="zoom:50%;" />

- python借助[AndroidViewClient](https://github.com/dtmilano/AndroidViewClient/wiki)，新版本支持python3

- 安装

  ```bash
  $ mkdir avc-workdir
  $ cd avc-workdir
  $ pyenv install 3.7.3
  $ pyenv local 3.7.3
  $ pip3 install --pre androidviewclient --upgrade
  ```

- 最后上代码

  ```python
  import os
  from com.dtmilano.android.viewclient import ViewClient, View, ViewNotFoundException
  GLOBAL_phone_serialno = "MYVDU15629007350"
  GLOBAL_adb = "adb -s MYVDU15629007350 "
  
  def clickWhat(what):
      vc.dump()
      view = vc.findViewWithText(what)
      view.touch()
      print("点击了"+what)
  
  def sayWhat(what_key):
      adbstr = GLOBAL_adb + "shell input keyevent 'KEYCODE_'"+what_key+""
      os.system(adbstr)
      print(adbstr)
  
  if __name__ == '__main__':
      device, serialno = ViewClient.connectToDeviceOrExit(serialno=GLOBAL_phone_serialno)
      vc = ViewClient(device=device, serialno=serialno)
      clickWhat("通讯录")
      clickWhat("宁")
      clickWhat("发消息")
      # [130,1682][842,1790]
      ViewClient.sleep(2)
      device.touch(x=200, y=1700)
      ViewClient.sleep(3)
      # adb -s MYVDU15629007350 shell input keyevent "KEYCODE_0"
      adbstr = GLOBAL_adb + "shell input text '你好啊'"
      sayWhat("I")
      sayWhat("L")
      sayWhat("O")
      sayWhat("V")
      sayWhat("E")
      sayWhat("U")
      clickWhat("发送")
  ```

### 总结

- 自动化挺好玩的，可以实现很多繁琐的任务
- 作为开发的我们了解一下自动化相关知识，能帮助我们搞事情，至于搞啥，自己考虑哦
- 最后，再次强调一下哈，本篇纯属娱乐

### 问题

- ```shell
  adb -s MYVDU15629007350 shell input text '你好啊' 
  #这条命令执行完，微信输入框没效果，原因不明，其他app 成功过，没问题，不知道是不是微信限制
  ```

### 引用

- https://github.com/Oslanka/appium-desktop

- https://github.com/dtmilano/AndroidViewClient