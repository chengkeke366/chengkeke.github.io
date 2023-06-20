```mermaid
classDiagram
class A
A : +String name
A : -int age
A : List~Object~ child    //带泛型的变量
A : +eat()
A : +sleep(time)          //有参数的方法
A : +getAge() int         //有返回值的方法

class B {
    +String name
    -int age
    List~Object~ child
    +eat()
    +sleep(time)
    +getAge() int
}

```



``` mermaid
graph BT;
A-->B
A-->C
B-->D;
C-->D;
```

```mermaid
graph LR
emperor((朱八八))-.子.->朱五四-.子.->朱四九-.子.->朱百六

朱雄英--长子-->朱标--长子-->emperor
emperor2((朱允炆))--次子-->朱标
朱樉--次子-->emperor
朱棡--三子-->emperor
emperor3((朱棣))--四子-->emperor
emperor4((朱高炽))--长子-->emperor3
```





```mermaid
graph TB
A-->B0
A---B1
A-.-B2
A-->|Link1|B3
A---|Link2|B4
A==>B5
A===B6
```



```mermaid
graph LR
A-->B
B-->C
D-->C

subgraph Sub1
B-->E
end

subgraph Sub2
B-->F
end
```



```mermaid
graph LR
style A fill:#f9f,stroke:#333,stroke-width:4px,fill-opacity:0.5
style B fill:#ccf,stroke:#333,stroke-width:2px,fill-opacity:1,stroke-dasharray:5,5

A-->B
```



```mermaid
graph LR
    A[Start]--> B{Is it?};
    B-- Yes--> C[OK];
    C --> D[Rethink];
    D --> B;
    B-- No -->E[End];
```

```mermaid
graph BT
默认方形
id1[方形]
id2(圆边矩形)
id3([体育场形])
id4[[子程序形]]
id5[(圆柱形)]
id6((圆形))
id7{菱形}
id8{{六角形}}
id9[/平行四边形/]
id10[\反向平行四边形\]
id11[/梯形\]
id12[\反向梯形/]
```





饼状图



```mermaid
pie
    title 为什么总是宅在家里？
    "喜欢宅" : 15
    "天气太热或太冷" : 20
    "穷" : 500
```





```mermaid
pie
    title 2023财年重点工作
    "RTC" : 60
    "播放器" : 25
    "ab转mp4" : 15
```

RTC主要KPI

* RTC SDK API行为对齐声网，满足云教室业务端需求

* 解决通信过程中看不见、听不见

* 黑屏、绿屏问题、声音异常

* 设备兼容性问题处理

  

  

  * room模块健壮性
    * 未正常退出，进入不了教室
    * 异步通信，lambda回调崩溃问题
  * 设备兼容性问题
    * 音频初始化失败
    * 采集过程中异常
    * 视频采集数据大小异常
    * win7 loopback兼容

* 解决声音异常问题

  * 屏幕共享回音及音频

* 解决黑屏、绿屏问题

  

