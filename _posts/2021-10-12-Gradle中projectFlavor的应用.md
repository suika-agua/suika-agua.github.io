# Gradle中projectFlavor的应用

## 需求分析
* 共主线
* 差异化

# 实际应用流程

## 配置部分

* build.gradle(app)简易配置


```
    android {
        compileSdkVersion 30
        buildToolsVersion "30.0.3"
        defaultConfig {
            applicationId "com.example.FlavorDemo"
            minSdkVersion 15
            targetSdkVersion 30
            versionCode 1
            versionName "1.0"
            testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
            flavorDimensions("version")
        }
        buildTypes {
            release {
                minifyEnabled false
                proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
            }
        }
        productFlavors{
            demo {
                /**
                 * [google翻译]
                 * 将次产品flavor(风味)分配给[版本]产品dimension(维度)。
                 * 如果仅适用一个dimension，则此属性为可选属性，
                 * 然后插件自动将所有模块的flavor分配给该dimension。
                 * */
                dimension "version"
                applicationIdSuffix ".demo"
                versionNameSuffix "-demo"
            }
            full {
                dimension "version"
                applicationIdSuffix ".full"
                versionNameSuffix "-full"
            }
        }
    }
```

*#参照developers中写法，将flavorDimensions()写在android标签下，报[Gradle DSL method not found:'lavorDimensions()']错误*
*#转移flavorDimensions()至defaultConfig标签下，代码成功运行。*

## flavor(风味)和demension(维度）

* 概述：
* 组合多种产品变种与变种维度

flavorDimensions ：

```java
         /**
         * [google翻译]
         * 指定你所需要使用的维度。列出不同维度的顺序由高到低确定了Gradle合并变体源和配置时（?）不同维度的优先级。
         * 你需要确保你配置的每个风味都符合其中一个维度。
         * */
        flavorDimensions "api", "mode"
```

productFlavors：

```java
        productFlavors{
                demo {
                    dimension "mode"
                }
                full {
                    dimension "mode"
                }
                /**
                 * 由于在flavorDimensions中的优先级，[api]风味中的配置会覆盖[mode]中的配置以及defaultConfig中的配置。
                 * 靠近法则：Gradle根据到flavorDimensions的顺序判定不同维度优先级（近>远）
                 * */
                minApi24 {
                    dimension "api"
                    minSdkVersion 24
                    /**
                     * 确保目标设备获取最高API可兼容等级的的版本号，以API等级递增的顺序来分配版本号。
                     * 要了解有关将版本代码分配给的更多信息来支持App的更新与上传，请至Google Play阅读-Multiple APK Support。
                     * */
                    versionCode 30000 + android.defaultConfig.versionCode
                    versionNameSuffix "-minApi24"
                }
                minApi23 {
                    dimension "api"
                    minSdkVersion 23
                    versionCode 20000  + android.defaultConfig.versionCode
                    versionNameSuffix "-minApi23"
                }
                minApi21 {
                    dimension "api"
                    minSdkVersion 21
                    versionCode 10000  + android.defaultConfig.versionCode
                    versionNameSuffix "-minApi21"
                }
            }
```

* 过滤变体：过滤不需要的变体


```java
       variantFilter { variant ->
            def names = variant.flavors*.name
            //要检查特定的构建类型，请使用variant.buildType.name ==“ <buildType>”（？）
            if (names.contains("minApi21") && names.contains("demo")) {
                // Gradle忽略满足上述条件的所有变体。
                setIgnore(true)
            }
        }
```

* 选择构建任意变体
Build > Select Build Variant

## 差异代码位置

* 查看差异代码点击 IDE 窗口右侧的 Gradle 图标 。

1.  依次转到 MyApplication > Tasks > android，然后双击 sourceSets。
2.  Gradle 执行该任务后，系统应该会打开 Run 窗口以显示输出。
3.  如果显示内容不是处于如上所示的文本模式，请点击 Run 窗口左侧的 Toggle view 图标位置


* Java类文件

1. 打开 Project 窗格，然后从窗格顶部的下拉菜单中选择 Project 视图。
2. 转到 MyProject/app/src/。
3. 右键点击 src 目录，然后依次选择 New > Folder > Java Folder。
4. 从 Target Source Set 旁边的下拉菜单中，选择 debug。
5. 点击 Finish。

* xml文件

1. 在同一 Project 窗格中，右键点击 src 目录，然后依次选择 New > XML > Values XML File。
2. 输入 XML 文件的名称或保留默认名称。
3. 从 Target Source Set 旁边的下拉菜单中，选择 debug。
4. 点击 Finish。

**eg.xml文件差异使用**

debug\...\strings.xml

    <resources>
            <string name="app_name">FlavorDemo</string>
            <string name="test_string">这里是debug App</string>
    </resources>

main\...\strings.xml

    <resources>
        <string name="app_name">FlavorDemo</string>
        <string name="test_string">这里是普通 App</string>
    </resources>

*#此处插入xml位置图、运行结果图*

## 更改默认源代码集配置（?）

```java
android {
  ...
  sourceSets {
    // 封装main source set的配置
    main {
      // 更改Java源的目录。 默认目录是'src/main/java'
      java.srcDirs = ['other/java']

      /**[google翻译]
       *如果列出多个目录，Gradle会使用所有目录来收集来源。 
       *因为Gradle赋予这些目录相同的优先级，如果您在多个目录中定义了相同的资源，
       *合并资源时出错。 默认目录为“ src / main / res”。
       */
      res.srcDirs = ['other/res1', 'other/res2']

      /**[google翻译]
      *注意：您应该避免指定一个或多个其他目录的父目录。 
      *例如，避免以下操作：res.srcDirs = ['other / res1'，'other / res1 / layouts'，'other /      		*res1 / strings']
      *您应仅指定根目录“ other / res1”或仅 嵌套的“ other / res1 / layouts”和“ other / res1 / 		  *strings”目录。
      */
          
      /**[google翻译]
       *对于每个source set(源集)，您只能指定一个Android清单。
       *默认情况下，Android Studio为您的创建 main source set
       *在src / main /目录中设置。
       */
      manifest.srcFile 'other/AndroidManifest.xml'
      ...
    }

	//创建其他块以配置其他源集。
    androidTest {

       /**[google翻译]
       *如果源集的所有文件都位于单个根目录下，您可以使用setRoot属性指定该目录。
       *收集源集的源时，Gradle仅在位置中查找相对于您指定的根目录。
       *例如，应用androidTest源集的以下配置，Gradle寻找Java仅在src / tests / java /目录中提供源。
       */
      setRoot 'src/tests'
      ...
    }
  }
}
...

```



#### [参考文章]

[developers-Android Studio-配置编译版本-配置编译变体](https://developer.android.com/studio/build/build-variants "developers-Android Studio-配置编译版本-配置编译变体")