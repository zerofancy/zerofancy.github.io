title: android项目开发：Fragment
author: 归零幻想
publishDate: 2020-12-21
editDate: 2020-12-21
tags: [android, 第一行代码, Kotlin, Activity]

<!--config-->

# Fragment

## Fragment的使用方式

### 静态添加Fragment

Fragment的写法

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical" android:layout_width="match_parent"
              android:layout_height="match_parent">

    <Button
            android:id="@+id/button"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="button"
            android:layout_gravity="center_horizontal"/>

</LinearLayout>
```

```kotlin
package top.ntutn.fragmenttest

import android.os.Bundle
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import androidx.fragment.app.Fragment

class LeftFragment : Fragment() {
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        return inflater.inflate(R.layout.left_fragment, container, false)
    }
}
```

静态添加Fragment

```xml

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              xmlns:tools="http://schemas.android.com/tools"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="horizontal"
              tools:context=".MainActivity">

    <fragment
            android:id="@+id/leftFragment"
            android:name="top.ntutn.fragmenttest.LeftFragment"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"/>

    <fragment
            android:id="@+id/rightFragment"
            android:name="top.ntutn.fragmenttest.RightFragment"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"/>

</LinearLayout>
```

<!--summary-->

### 动态添加Fragment

```kotlin

val fragmentManager = supportFragmentManager
val transaction = fragmentManager.beginTransaction()
transaction.replace(R.id.rightLayout, fragment)
transaction.commit()
```

### 在Fragment中实现返回栈

```kotlin

val fragmentManager = supportFragmentManager
val transaction = fragmentManager.beginTransaction()
transaction.replace(R.id.rightLayout, fragment)
transaction.addToBackStack(null)
transaction.commit()
```

### Fragment与Activity交互

Activity中获取Fragment

```kotlin
val fragment = supportFragmentManager.findFragmentById(R.id.leftFragment) as LeftFragment
```

或使用`kotlin-android-extensions`

```kotlin
val fragment = leftFragment as LeftFragment
```

## Fragment的生命周期

和Activity的生命周期类似，重点是几个方法：

- onAttach() 当Fragment和Activity建立关联的时候调用
- onCreateView() 为Fragment创建视图时调用
- onActivityCreated() 确保与Fragment相关联的Activity创建完毕时调用。
- onDestroyView() 与Fragment关联的视图被移除时调用。
- onDetach() 当Fragment与Activity解除关联时调用
