# Gradle中projectFlavor的应用

## 需求分析
* 共主线
* 差异化

# 实际配置流程
* build.gradle(app)简易配置

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

*#参照developers中写法，将flavorDimensions()写在android标签下，报[Gradle DSL method not found:'lavorDimensions()']错误*
*#转移flavorDimensions()至defaultConfig标签下，代码成功运行。*

## flavor(风味)和demension(维度）

* 概述：
* 组合多种产品变种与变种维度

flavorDimensions ：

             /**
             * [google翻译]
             * 指定你所需要使用的维度。列出不同维度的顺序由高到低确定了Gradle合并变体源和配置时（?）不同维度的优先级。
             * 你需要确保你配置的每个风味都符合其中一个维度。
             * */
            flavorDimensions "api", "mode"


productFlavors：

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

* 过滤变体：过滤不需要的变体
       variantFilter { variant ->
            def names = variant.flavors*.name
            //要检查特定的构建类型，请使用variant.buildType.name ==“ <buildType>”（？）
            if (names.contains("minApi21") && names.contains("demo")) {
                // Gradle忽略满足上述条件的所有变体。
                setIgnore(true)
            }
        }

* 选择构建任意变体
Build > Select Build Variant

#### [参考文章]
[developers-Android Studio-配置编译版本-配置编译变体](https://developer.android.com/studio/build/build-variants "developers-Android Studio-配置编译版本-配置编译变体")