
## Activity定义
&emsp;&emsp;Activity是一个应用组件，用于可以与其提供的屏幕进行交互，，以执行拨打电话、拍摄照片、发送电子邮件或查看地图等操作。 每个 Activity 都会获得一个用于绘制其用户界面的窗口。窗口通常会充满屏幕，但也可小于屏幕并浮动在其他窗口之上。

&emsp;&emsp;一个应用通常由多个彼此松散联系的Activity 组成。一般会指定应用中的某个Activity为主Activity，即首次启动应用时呈现给用户的那个Activity。而且每个Activity均可启动另一个Activity，以便执行不同的操作。每次新Activity启动时，前一Activity便会停止，但系统会在堆栈（“返回栈”）中保留该Activity。当新Activity启动时，系统会将其推送到返回栈上，并取得用户焦点。返回栈遵循基本的“后进先出”堆栈机制，因此，当用户完成当前Activity并按“返回”按钮时，系统会从堆栈中将其弹出（并销毁），然后恢复前一 Activity。

&emsp;&emsp;当一个Activity因为某个新Activity的启动而停止时，系统会通过该Activity的生命周期回调方法通知其这一状态变化，Activity会因状态变化而创建、停止、恢复、销毁Activity，从而调用其对应的回调方法，每种方法都会提供与该状态变化相应的特定操作的机会。

## 创建Activity
### 实现用户界面
### 在清单文件中声明Activity
## 启动Activity
### 启动Activity获取结果
### ！启动其他应用的Activity
## 结束Activity
## Activity生命周期
### 声明周期回调
### 保存Activity状态
### 处理配置变更
### 协调Activity
### ！横竖切屏Activity生命周期变化
## Activity启动模式、应用场景
### 启动模式介绍
### 任务相关性（TaskAffinity）
## Activity标志位

## ！Activity启动过程