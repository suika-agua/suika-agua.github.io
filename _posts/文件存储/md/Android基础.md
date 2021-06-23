# Android基础

## 1、ANR

### 6条原因

### 4条修正

### 1条解决方案

- 将所有耗时操作，比如访问网络，Socket通信，查询大 量SQL 语句，复杂逻辑计算等都放在子线程中去，然 后通过handler.sendMessage、runonUIThread、AsyncTask、RxJava等方式更新UI。无论如何都要确保用户界面的流畅 度。如果耗时操作需要让用户等待，那么可以在界面上显示度条

## 2、Activity和Fragment生命周期

### 画图

## 3、横竖屏切换时Activity生命周期

### android:configChanges

- 大于3.2

	- 各生命周期执行一次

- 小于3.2

	- 切横屏执行一次，切竖屏执行两次

### android:configChanges=”orientation”

- 全版本

	- 各生命周期执行一次

### android:configChanges=”orientation |keyboardHidden”

- 大于3.2

	- 各生命周期执行一次

- 小于3.2

	- 来回切换都只调用 onConfigurationChanged

### android:configChanges="orientation|screenSize"

- 来回切换都只调用 onConfigurationChanged

### 各生命周期执行一次->

- onPause->onSaveInstancceState->onStop->onDestroy->onCreate->onStart->onRestoreInstanceState->onResume

## 4、AsyncTask

### 是什么

### 存在问题

- 生命周期

	- 一直执行知道doinbackgroound执行完毕，可手动销毁，必须在activity销毁之前被销毁

- 内存泄漏

	- 若AsyncTask被声明为Activity的非静态内部类，Activity销毁AsyncTask仍在执行，会导致Activity无法回收，内存泄漏

- 结果丢失

	- 若Activity重新创建，之前AsyncTask中关于Activity的引用已失效

- 并行还是串行

	- 1.6前，A是串行，1.6后采用线程池处理。且3.0开始，又采用串行

### 原理

- 有两线程池，THREAD_POOL_EXECUTOR用于执行任务，InternalHandler用于将执行环境由线程切换到主线程
- 含静态Handler sHandler必须在主线程创建，则AsyncTask必须在主线程创建

## 5、onSaveInatanceState与onRestoreInatanceState

### 存在于Activity生命周期中，遇到意外需系统销毁Activity时才调用，用于保存临时性数据

## 6、android中进程优先级

### 前台进程>可见进程>服务进程>后台进程>空进程

## 9、Context相关

### 5

## 12、解析xml类横向对比

### DOM

### SAX

### Xmlpull

## 13、Jar和aar区别

### jar包只有代码，aar中有代码和资源文件

## 14、android程序内存

## 17、Thread、AsyncTask、IntentService*

### Thread：Activity被finish后，没有执行完或主动停止时会 一直执行下去

### AsyncTask：用于网络请求或简单数据处理

### IntentService：处理异步请求，实现多线程，多个耗时任务依次执行

## 18、Merge、ViewStub作用

### Merge：减少视图层级，删除多余层级

### ViewStub：按需加载，减少内存使用量、加快渲染速度，不支持merge标签

## 19、activity的startActivity和context的starrtActivity的区别

## 20、在service中创建dialog对话框

### 获取到dialog对象后先设置TYPE_SYSTEM_ALERT类型

### 在Manifext中添加android.permission.SYSTEM_ALERT_WINOW"权限

*XMind - Evaluation Version*