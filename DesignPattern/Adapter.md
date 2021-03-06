# Adapter模式（适配器模式）
## 意图
将一个类的接口转换成客户希望的另外一个接口。Adapter模式使原本由于接口不兼容而不能一起工作的类可以一起工作。

## 实现
将原有类包含在另一个类之中。让包含类与需要的接口匹配，调用被包含类的方法。

## Facade模式与Adapter模式不同
|  | Facade模式 | Adapter模式 |
| :---： | :----: | ：----: |
| 是否存在既有的类？ | 是 | 是 |
| 是否必须按某个接口设计？ | 否     | 是     |
| 对象需要多态行为吗？ | 否 | 可能 | 
| 需要更简单的接口？ | 是 | 否


## 实例
联合国的一个翻译要使来自不同国家的外交官能够用各自的语言阐述和辩论各自国家的立场。
翻译采用了一种语言到另一种语言的“动态等价”表示法，这样各种概念将以接受者所期望和需要的方式进行交流。
