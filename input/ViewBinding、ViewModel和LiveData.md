title: ViewBinding、ViewModel和LiveData
author: 归零幻想
publishDate: 2021-02-22
editDate: 2021-02-22
tags: [tag1, tag2]

<!--config-->

毕设项目没有历史包袱，我可以尽量向best practice努力。

## ViewBinding

无数人痛恨findViewById，并且为了干掉它做了许多尝试，比如ButterKnife、kotlin-android-extensions。

现在，有了ViewBinding，项目中真的可以不写findViewById了。至少目前为止我的毕设项目还没有一个findViewById。

其实与ViewBinding相似的，还有一个DataBinding，但我不太喜欢，感觉在xml里面写代码不是一个好主意。

### 使用

首先在build.gradle（或build.gradle.kts）中的android块添加

```groovy
buildFeatures {
    viewBinding = true
}
```

在xml中正常定义你的布局

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <TextView
        android:id="@+id/text_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Hello World!" />

    <Button
        android:id="@+id/test_button"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="toast" />

    <Button
        android:id="@+id/change_button"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="change" />
</LinearLayout>
```

接下来就可以愉快使用了

```kotlin
package top.ntutn.viewmodeldemo

import android.os.Bundle
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import top.ntutn.viewmodeldemo.databinding.ActivityMainBinding

class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        initView()
    }

    private fun initView() {
        binding.apply {
            changeButton.setOnClickListener { textView.text = "Changed!" }
            testButton.setOnClickListener {
                Toast.makeText(
                    this@MainActivity,
                    "Test",
                    Toast.LENGTH_LONG
                ).show()
            }
        }
    }
}
```

我们在xml里面用下划线分隔的id在这里就直接变成了binding里面的字段，相当舒服。

### 在RecyclerView中的使用

在RecyclerView中代码要稍微发生一点变化，因为我们是在onCreateViewHolder时反射创建布局的。

```kotlin
package top.ntutn.viewmodeldemo

import android.view.LayoutInflater
import android.view.ViewGroup
import androidx.recyclerview.widget.RecyclerView
import top.ntutn.viewmodeldemo.databinding.ItemTestBinding

class MyAdapter : RecyclerView.Adapter<MyAdapter.ViewHolder>() {
    class ViewHolder(val binding: ItemTestBinding) : RecyclerView.ViewHolder(binding.root)

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        return ViewHolder(
            ItemTestBinding.inflate(LayoutInflater.from(parent.context), parent, false)
        )
    }

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        holder.binding.textView.text = TODO("Not yet implemented")
    }

    override fun getItemCount(): Int {
        TODO("Not yet implemented")
    }
}
```

仍然不需要findViewById！

<!--summary-->

## ViewModel和LiveData

应用MVVM模式更好管理代码，使之便于拓展和测试，好处不多说。

首先引入ViewModel、LiveData和Kotlin Coroutines的拓展包和Retrofit的依赖包。

```groovy
implementation('org.jetbrains.kotlinx:kotlinx-coroutines-android:1.4.1')
implementation('androidx.core:core-ktx:1.3.2')
implementation('androidx.lifecycle:lifecycle-viewmodel-ktx:2.3.0')
implementation('androidx.lifecycle:lifecycle-livedata-ktx:2.3.0')
implementation('com.squareup.retrofit2:retrofit:2.9.0')
implementation "androidx.fragment:fragment-ktx:1.3.0"
```

准备ViewModel

```kotlin
package top.ntutn.viewmodeldemo

import androidx.lifecycle.LiveData
import androidx.lifecycle.MutableLiveData
import androidx.lifecycle.ViewModel

class MainViewModel : ViewModel() {
    private val _text = MutableLiveData<String>().apply {
        value = "我的"
    }
    val text: LiveData<String> = _text

    fun changeText() {
        _text.value = "修改过的"
    }
}
```

在MainActivity中

```kotlin
private val mainViewModel by viewModels<MainViewModel>()

private fun initView() {
    binding.apply {
        changeButton.setOnClickListener { mainViewModel.changeText() }
        testButton.setOnClickListener {
            Toast.makeText(
                this@MainActivity,
                "Test",
                Toast.LENGTH_LONG
            ).show()
        }
    }
    mainViewModel.text.observe(this) {
        binding.textView.text = it
    }
}
```

这样，VC负责视图层的展示逻辑，VM负责业务逻辑，清晰了许多。

LiveData具有生命周期感知功能，随时监听修改并更新视图。它有setValue()和postValue()两个方法设置值，相对于前者，后者将setValue放到了下一个消息循环，可以在非UI线程调用。

### InitedLiveData

然而自带的LiveData和MutableLiveData用多了总感觉难受，因为getValue()返回的是一个可空的数据，当我写了n多次`_field.value?.key?:""`后，我爆发了。很多场景下这个value都可以定义成不可空的，所以还是简单封装一个不需要判空的更好。

```kotlin
package top.ntutn.viewmodeldemo

import androidx.lifecycle.LiveData

/**
 * 解决LiveData的空安全问题
 *
 */
abstract class CheckedLiveData<T> : LiveData<T>() {
    override fun getValue(): T {
        return super.getValue() ?: run {
            val res = initValue()
            value = res
            initValue()
        }
    }

    protected abstract fun initValue(): T
}

class InitedLiveData<E>(private val initBlock: () -> E) : CheckedLiveData<E>() {
    override fun initValue() = initBlock.invoke()

    public override fun setValue(value: E) {
        super.setValue(value)
    }

    public override fun postValue(value: E) {
        super.postValue(value)
    }
}
```

如此这般，上面的代码就可以这样表示了

```kotlin
package top.ntutn.viewmodeldemo

import androidx.lifecycle.ViewModel

class MainViewModel : ViewModel() {
    private val _text = InitedLiveData { "我的" }
    val text: CheckedLiveData<String> = _text

    fun changeText() {
        _text.value = "修改过的"
    }
}
```

### Kotlin协程与Retrofit更搭

首先Retrofit也值得我们封装一个工具类来创建Service：

```kotlin
object RetrofitUtil {
    ...
    private val retrofit by lazy {
        Retrofit.Builder()
            .baseUrl(retrofitConfig?.baseUrl ?: BASE_URL)
            .addConverterFactory(GsonConverterFactory.create())
            .client(okHttpClient)
            .build()
    }
    
    fun <T> create(serviceClass: Class<T>): T = retrofit.create(serviceClass)
    
    inline fun <reified T> create(): T = create(T::class.java)
}
```

前文提到，我们主要业务逻辑写到VM中，所以

```kotlin
package top.ntutn.novelrecommend.ui.viewmodel.main

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.launch
import kotlinx.coroutines.withContext
import retrofit2.await
import timber.log.Timber
import top.ntutn.novelrecommend.NovelService
import top.ntutn.novelrecommend.arch.CheckedLiveData
import top.ntutn.novelrecommend.arch.InitedLiveData
import top.ntutn.novelrecommend.model.NovelModel
import top.ntutn.novelrecommend.utils.RetrofitUtil

class DiscoverViewModel : ViewModel() {
    private val _novelList =
        InitedLiveData<MutableList<NovelModel>> { mutableListOf() }

    val novelList: CheckedLiveData<MutableList<NovelModel>> = _novelList

    private suspend fun getNovel(): List<NovelModel> {
        return RetrofitUtil.create<NovelService>()
            .getNovel(deviceInfo = DeviceUtil.getDeviceInfoMap())
            .await()
            .map { it.copy(localId = (0..Long.MAX_VALUE).random()) }
    }

    fun loadMore() {
        viewModelScope.launch {
            _novelList.value = withContext(Dispatchers.IO) {
                try {
                    _novelList.value.addAll(getNovel())
                } catch (e: Exception) {
                    Timber.e(e, "获取小说失败")
                }
                _novelList.value
            }
        }
    }
}
```

是不是很舒服？
