# 基本概念

## 语言版本

### 语义版本

#### 格式

__主版本号.次版本号.修订号__

* 主版本号
    做了不兼容的API修改

* 次版本号
    做了向下兼容的API修改

* 修订号
    做了向下兼容的API问题修正



### 发行版本

* master -> Nightly

* beta   -> Beta

* stable -> Stable

### Edition

* 2015

* 2018

* 2021

## 面向表达式

一切皆类型

* 属性 `#![...]`

属性设置

* 行分割 `;`

遇到 时`;` 进行求值 产生一个 `()` (Unit Type) 

即分号表达式返回值永远是自身的单元类型, 但只有在块表达式的最后一行才进行求值, 其他情况作为
连接符

* 块分割 `{...}`

只对最后一行进行求值返回

## 编译期计算

* 过程宏 + Build脚本


* CTFE (CPP 中 constexpr)

