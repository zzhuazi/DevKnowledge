[toc]
## Activity定义
&emsp;&emsp;Activity是一个应用组件，用于可以与其提供的屏幕进行交互，，以执行拨打电话、拍摄照片、发送电子邮件或查看地图等操作。 每个 Activity 都会获得一个用于绘制其用户界面的窗口。窗口通常会充满屏幕，但也可小于屏幕并浮动在其他窗口之上。

&emsp;&emsp;一个应用通常由多个彼此松散联系的Activity 组成。一般会指定应用中的某个Activity为主Activity，即首次启动应用时呈现给用户的那个Activity。而且每个Activity均可启动另一个Activity，以便执行不同的操作。每次新Activity启动时，前一Activity便会停止，但系统会在堆栈（“返回栈”）中保留该Activity。当新Activity启动时，系统会将其推送到返回栈上，并取得用户焦点。返回栈遵循基本的“后进先出”堆栈机制，因此，当用户完成当前Activity并按“返回”按钮时，系统会从堆栈中将其弹出（并销毁），然后恢复前一 Activity。

&emsp;&emsp;当一个Activity因为某个新Activity的启动而停止时，系统会通过该Activity的生命周期回调方法通知其这一状态变化，Activity会因状态变化而创建、停止、恢复、销毁Activity，从而调用其对应的回调方法，每种方法都会提供与该状态变化相应的特定操作的机会。

## 创建Activity
&emsp;&emsp;要创建Activity，必须创建Activity的子类，需要在子类中实现Activity在其生命周期的各种状态之间转变时，系统所调用的回调方法，其中两个最为重要的回调方法是：

1. onCreate()
该方法必须要实现，系统会在创建Activity时调用此方法，开发者需要在该方法中初始化Activity的必须组件，最重要的时调用setContentView()来定义Activity界面的布局。

2. onPause()
系统将此方法作为用户离开Activity的第一个信号进行调用，通过在该方法中确认用户会话结束后仍然有效的任何更改。

### 实现用户界面
&emsp;&emsp;Activity的用户界面是由层级式视图提供的（衍生自View类的对象），每个视图都控制着Activity窗口内的特定矩形空间。
&emsp;&emsp;可以利用 Android 提供的许多现成视图设计和组织您的布局。
- “小部件”是提供按钮、文本字段、复选框或仅仅是一幅图像等屏幕视觉（交互式）元素的视图。 
- “布局”是衍生自 ViewGroup 的视图，为其子视图提供唯一布局模型，例如线性布局、网格布局或相对布局。

&emsp;&emsp;还可以为 View 类和 ViewGroup 类创建子类（或使用其现有子类）来自行创建小部件和布局，然后将它们应用于您的 Activity 布局。

&emsp;&emsp;利用视图定义布局的最常见方法是借助保存在应用资源内的 XML 布局文件。这样就可以将用户界面的设计与定义Activity 行为的源代码分开维护。 通过setContentView()将布局设置为Activity的UI，从而传递布局的资源ID。不过，您也可以在Activity代码中创建新 View，并通过将新View插入ViewGroup来创建视图层次，然后通过将根ViewGroup传递到setContentView()来使用该布局。

### 在清单文件中声明Activity
必须在ApplicationMinifest.xml清单文件中声明Activity，这样系统才能够访问它。在清单文件中，将<activity>元素添加到<application>元素中，如：
```xml
<manifest ...>   
    <application  ...>        
        <activity android:name=".view.activity.BillDetailActivity"></activity>        
        <activity android:name=".view.activity.AddBillActivity" />       
        <activity android:name=".view.activity.MainActivity">            
            <intent-filter>               
                <action android:name="android.intent.action.MAIN" />        
                <category android:name="android.intent.category.LAUNCHER" />    
            </intent-filter>
        </activity>
    </application>
</manifest>
```
可以在<activity>元素中加入其他特性，用于定义Activity标签、Activity图标或者风格主题等用于设置activity UI风格的属性。同时，必不可少的的属性是android:name，该属性用于指定Activity的类名，具有唯一性。

<activity> 元素还可指定各种Intent过滤器—使用<intent-filter>元素—以声明其他应用组件激活它的方法。
当使用Android SDK工具创建新应用时，系统自动为您创建的存根Activity 包含一个 Intent 过滤器，其中声明了该 Activity 响应“主”操作且应置于“launcher”类别内。
- <action> 元素指定这是应用的“主”入口点。
- <category> 元素指定此 Activity 应列入系统的应用启动器内（以便用户启动该 Activity）。

如果打算让应用成为独立应用，不允许其他应用激活其Activity，则不需要任何其他Intent过滤器。 正如前例所示，只应有一个Activity具有“主”操作和“launcher”类别。 如果不想提供给其他应用的Activity不应有任何 Intent 过滤器，您可以利用显式Intent自行启动它们

不过，如果您想让Activity对衍生自其他应用（以及您的自有应用）的隐式 Intent 作出响应，则必须为 Activity 定义其他 Intent 过滤器。 对于您想要作出响应的每一个 Intent 类型，您都必须加入相应的 <intent-filter>，其中包括一个 <action> 元素，还可选择性地包括一个 <category> 元素和/或一个 <data> 元素。这些元素指定您的 Activity 可以响应的 Intent 类型。

## 启动Activity
可以通过调用startActivity()和startActivityForResult()两个方法，并将传递给描述需要启动的Intent来启动另外一个Activity。Intent对象可以指定想要启动的Activity或者想要执行的操作类型，同时可以携带少量数据，如int、float、string和bundle对象等。

1. 显式启动
按照名称指定要启动的Activity，通常在本应用内使用显式intent来启动Activity，因为在本应用内知道具体的activity类名。当使用显示启动时，系统会立即启动Intent对象中执行的Activity。
```java
Intent intent = new Intent(getContext(), BillDetailActivity.class);
startActivity(intent);
```

2. 隐式启动
通常不会指定特定的activity类名，而是声明要执行的常规操作，从而允许其他应用中的组件来处理它，比如：发送邮件时，附上邮件的地址，当电子邮件应用响应该Intent时，就会读取extra中提供的字符串，作为收件人字段。当使用该Intent时，系统将Intent的内容与在设备上的其他应用的清单文件中声明的intent过滤器进行比较，从而找到需要启动的组件，如果存在多个符合要求的组件，则弹出一个对话框，让用户选择。
```java
Intent intent = new Intent(Intent.ACTION_SEND);
intent.putExtra(Intent.EXTRA_EMAIL， recipientArray);
startActivity(intent);
```
上述是通过调用startActivity()方法启动Activity，如果需要从启动的Activity获取结果，那么就需要调用startActivityForResult()来启动Activity。想要在随后收到新Activity返回的结果，就需要实现onActivityResult()回调。当新Activity完成时，会使用Intent向onActivityResult()方法返回结果。
```java
@Override    
public void onActivityResult(int requestCode, int resultCode, Intent data) {        
    switch (requestCode){            
        case 1:     //请求添加Bill的请求码                
            if(resultCode == RESULT_OK) {       
                //添加Bill成功                    
                Log.d(TAG, "onActivityResult: 添加账单成功");                    
                //重新加载bills                    
                mPresenter.loadBills(DateUtil.getYearAndMonth(new Date()));                
            }else if(resultCode == RESULT_CANCELED) {                   
                //没有添加账单                    
                Log.d(TAG, "onActivityResult: 没有添加账单");               
            }               
            break;                 
    }    
}
//新Activity需要返回处
//设置返回码为取消添加。
this.setResult(RESULT_CANCELED, new Intent());
this.finish()
```

### ! Activity A启动Activity B会调用哪些方法？如果B是透明主题或者DialogActivity呢？
### ! 启动其他应用的Activity
## 结束Activity
## Activity生命周期
### 声明周期回调
### 保存Activity状态
### 处理配置变更
### 协调Activity
### ! 横竖切屏Activity生命周期变化
## Activity启动模式、应用场景
### 启动模式介绍
### 任务相关性（TaskAffinity）
## Activity标志位

## ! Activity启动过程