title: 用Jetpack Compose重构我的连连看游戏
author: 归零幻想
publishDate: 2021-09-21
editDate: 2021-09-21
tags: [Jetpack Compose, Kotlin, 安卓, Flow]

<!--config-->

此前的文章中提到，我闲暇时写了个连连看游戏。因为比较闲，正好又对Jetpack Compose比较感兴趣，于是我想用Compose重构界面，学习下Compose的使用。
然后没有花费太大力气就完成了。

[GameViewModelCompose.kt](https://github.com/zerofancy/match/blob/main/app/src/main/java/top/ntutn/match/GameViewModelCompose.kt)

于是我想，既然用上Compose了，是不是可以直接迁移到桌面平台？

这篇文章实际上没什么内容，就是对于折腾过程的一个简单记录，最后的成品在github，直接[去看代码](https://github.com/zerofancy/match-m)就可以了。

<!--summary-->

## 依赖版本

虽然安卓上的Compose已经是正式发布了，甚至在公司里面某些业务线都在调研引入的可行性了，但桌面上的Compose目前（2021年9月20日）还在alpha3，很多API还是不稳定状态，依赖版本也有点混乱——我用idea创建的默认工程竟然不能在所有支持平台打出Hello World包！

1. 确定依赖版本。默认创建的工程不能直接跑起来，我花费了一些时间去查每个依赖最新版本是多少，稳定版本是多少，去StackOverflow查错误对应解决方案。
2. 删除有关自动测试的依赖。虽然大家都说自动测试很重要，但我们团队实际开发中是基本不写的，我暂时也没有学习的计划。当然更重要的原因是这个依赖留着会报错，我试了好久也没解决。
3. 最坑爹的是这东西限制JDK版本很死，不能低于15不能高于16。低于15会不能正常打包，高于16有些依赖会报错。

## ViewModel移植

我此前的连连看游戏遵循了MVVM结构，只要将VM部分移植过来，那主要的游戏逻辑就移植过来了。

然而我还是太天真了：ViewModel和LiveData都是安卓上的库，需要找个替代品。

LiveData就是个被观察对象，特点是拥有生命周期感知能力，不用手动取消注册。说实话我们游戏界面比较简单，其实没有用到这个特性，完全可以手撸一个。不过我还是找到了比较好的替代品：[Flow](https://developer.android.com/kotlin/flow/stateflow-and-sharedflow?hl=zh-cn)。

基本用StateFlow完全替换掉LiveData即可。

对于ViewModel，安卓上我仍然想androidx库中的ViewModel，为了和桌面上共用逻辑，需要定义接口IViewModel。

```kt
package top.ntutn.common

import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow

interface IViewModel {
    var mahjongArea: Array<Array<MutableStateFlow<MahjongType>>>
    val gameState: StateFlow<GameState>
    val gameTime: StateFlow<Int>
    val rows: StateFlow<Int>
    val cols: StateFlow<Int>
    fun init(
        rows: Int,
        cols: Int,
        itemCount: Int,
        maxGameTime: Int,
        stepGameTime: Int,
        selectableItemCount: Int = 0
    )

    fun start()
    fun pause()
    fun resume()
    fun timeTick()
    fun itemClick(row: Int, col: Int)

    enum class GameState {
        PENDING,
        RUNNING,
        PAUSE,
        SUCCEEDED,
        FAILED
    }
}
```

游戏逻辑实现在GameViewModel中，而给安卓用的VM就是个Wrapper，持有一个GameViewModel即可。当然有Kotlin的类委托代码可以写的更简单点：

```kt
package top.ntutn.common

import androidx.lifecycle.ViewModel

class AndroidGameViewModel : ViewModel(), IViewModel by GameViewModel()
```

这样安卓的VM仍然可以按照我们熟悉的方式管理。

## UI界面移植

桌面版的Compose和安卓版的实际上没有多大变化，但加载图片的方式显然是不同的。桌面上可没有通过`R.drawable.xxx`去加载资源的道理。因而根据id去绘图的代码需要抽离出来：

```kt
@Composable
fun GamePage(time: Int, gameViewModel: IViewModel, getPainterById: @Composable (Int) -> Painter) {
    Box(
        modifier = Modifier.fillMaxSize()
    ) {
        Column {
            Spacer(modifier = Modifier.height(48.dp))
            TimerArea(
                time = time
            )
        }

        Surface(
            modifier = Modifier
                .align(Alignment.Center)
                .padding(16.dp)
        ) {
            Board(gameViewModel, getPainterById)
        }
    }
}
```

然后安卓版的绘图通过R文件
```kt
            Surface(color = MaterialTheme.colors.background) {
                    GamePage(
                        time = gameTimeState,
                        gameViewModel
                    ) {
                        painterResource(MahjongAndroid.front[it])
                    }
                }
            }
```

桌面版则去加载资源文件

```kt
                Surface(color = MaterialTheme.colors.background) {
                    GamePage(
                        time = gameTimeState,
                        gameViewModel
                    ) {
                        BitmapPainter(imageFromResource("drawable/${MahjongDesktop.all[it]}.png"))
                    }
                }
```

## 计时逻辑变更

桌面上计时不是通过Handler每秒去发送一次事件，不过大差不差

```kt
        Timer(1000, object : ActionListener {
            override fun actionPerformed(e: ActionEvent?) {
                gameViewModel.timeTick()
            }
        }).start()
```

## 弹窗逻辑变更

```kt
        GlobalScope.launch {
            gameViewModel.gameState.collect {
                println("游戏状态：$it")
                when (it) {
                    IViewModel.GameState.SUCCEEDED -> {
                        EventQueue.invokeLater {
                            val option = JOptionPane.showConfirmDialog(
                                null,
                                "你赢了。是否再来一局？",
                                "你赢了",
                                JOptionPane.OK_OPTION,
                                JOptionPane.QUESTION_MESSAGE
                            )
                            if (option == JOptionPane.OK_OPTION) {
                                startGame()
                            } else {
                                System.exit(0)
                            }
                        }
                    }
                    IViewModel.GameState.FAILED -> {
                        ...
                    }
                    ...
                }
            }
        }
```

就是调用相关函数弹个窗而已。但这里我还是踩了俩坑的：

1. 不要阻塞事件分发线程，这货对应安卓的主线程。
2. 不要把弹窗相关代码写在Compose微件里。因为Compose函数可能相当频繁地重组，我们用的弹窗不是Compose封装过的特殊版本，重复执行显然不符合预期。

## 打包发布

打包这里也有坑：

1. JDK版本。打包是JDK15提供的特性，要换到JDK15。然而你换到15打包还会出错，说你用的JDK11。这个是Android Studio设置太混乱了，有好几处关于JDK版本的设置。项目的、gradle的，IDE自身的。你可以只设置下系统的环境变量，然后用命令打包./gradlew package。
2. 图标。经过测试不设置一个图标在macOS上无法完成打包。
3. 软件包名。目前经过测试，Windows对于这点容忍度反而是最高的，macOS和Linux上带了小数点打出的包不能运行。另外Linux上也不能用中文包名啥的。不知道具体规则，建议直接试试。

