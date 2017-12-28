--- 
layout: post
title: "Go语言基础入门 - 语法篇"
comments: true
categories:
 - golang
---

<pre class="brush: Go" line="1">
// Go是采用语法解析器自动在每行末尾增加分号，所以在写代码的时候可以把分号省略。
// Go编程中只有几个地方需要手工增加分号，如: for循环使用分号把初始化，条件和遍历元素分开。在一行中有多条语句时，需要增加分号。
// 不能把控制语句(if, for, switch, or select)、函数、方法 的左大括号单独放在一行， 如果你这样作了语法解析器会在大括号之前插入一个分号，导致编译错误。

// 引用包名与导入路径的最后一个目录一致
import "fmt"
import "math/rand"
fmt.Println(rand.Intn(10))    // 0到10之间的非负伪随机数

// 用圆括号组合导入包，这是“factored”导入语句
import ("fmt"; "math")
import (
    "fmt"
    "math"
)
// 导入包可以定义别名，防止同名称的包冲突
import n "net/http"
import (
    控制台 "fmt"
    m "math"
)
控制台.Println(m.Pi)

// 首字母大写的名称是被导出的, 首字母小写的名称只能在同一包内访问(同包跨文件也能访问)
var In int // In is public
var in byte // in is private
var 看不见 string // 看不见 is private
const Com bool = false // Com is public
const 还是看不见 uint8 = 1 // 还是看不见 is private
type Integer int // Integer is public
type ブーリアン *bool // ブーリアン is private
func Export() {} // Export is public
func 导入() {} // 导入 is private
func (me *Integer) valueOf(s string) int {} // valueOf is private
func (i ブーリアン) String() string {} // String is public

// Go 的基本类型：
┌──────┬─────────┬────────┬─────────┬───────────┬────────────┐
│ bool │ string  │        │         │           │            │
├──────┼─────────┼────────┼─────────┼───────────┼────────────┤
│ int  │ int8    │ int16  │ int32   │ int64     │            │
│      │         │        │ rune    │           │            │
├──────┼─────────┼────────┼─────────┼───────────┼────────────┤
│ uint │ uint8   │ uint16 │ uint32  │ uint64    │ uintptr    │
│      │ byte    │        │         │           │            │
├──────┼─────────┼────────┼─────────┼───────────┼────────────┤
│      │         │        │ float32 │ float64   │            │
├──────┼─────────┼────────┼─────────┼───────────┼────────────┤
│      │         │        │         │ complex64 │ complex128 │
└──────┴─────────┴────────┴─────────┴───────────┴────────────┘
// byte 是 uint8 的别名
// rune 是 int32 的别名，代表一个Unicode码点
</pre>
