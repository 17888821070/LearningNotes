# 包

## 定义包

包名称应全部为小写

```scala
/*
第一种
在文件的头定义包名，后续所有代码都放在该包中
*/
package com.runoob

/*
第二种
可以在一个文件中定义多个包
*/
package com.runoob {
  class HelloWorld 
}
```

## 引用

`import` 语句可以出现在任何地方，而不是只能在文件顶部

`import` 的效果从开始延伸到语句块的结束，这可以大幅减少名称冲突的可能性

```scala
import java.awt.Color  // 引入Color
 
import java.awt._  // 引入包内所有成员
 
def handler(evt: event.ActionEvent) { // java.awt.event.ActionEvent
  ...  // 因为引入了java.awt，所以可以省去前面的部分
}

// 引入包中的几个成员
import java.awt.{Color, Font}

// 重命名成员
import java.util.{HashMap => JavaHashMap}

// 隐藏成员
import java.util.{HashMap => _, _} 
// 引入了 util 包的所有成员，但是 HashMap 被隐藏了
```

## 目录结构

包名和目录应采用以下的约定格式：`<top-level-domain>.<domain-name>.<project-name>`，如 `com.google.selfdrivingcar.camera`，对应的目录结构 `SelfDrivingCar/src/main/scala/com/google/selfdrivingcar/camera/Lens.scala`

