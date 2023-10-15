# GDScript中的进制转化

在 GDScript 中，你可以使用`0b`字面量书写二进制值，使用`0x`字面量书写十六进制值，使用`_`符号分隔较长的数字，提升可读性。

```swift
var x = 0b1001     # x 为 9
var y = 0xF5       # y 为 245
var z = 10_000_000 # z 为 10000000
```

Godot4的`String`提供了两个新的方法`num_int64`和`num_uint64`可以实现将任意数字转化为二、八、十六进制字符串形式的功能。
```swift
var num = 10
	
print(String.num_int64(num))    # "10"
print(String.num_int64(num,2))  # "1010"
print(String.num_int64(num,8))  # "12"
print(String.num_int64(num,16)) # "a"
```
## 函数封装
```swift
# 10进制整数 --> 2进制
func int10_to2(num10:int) -> String:
	return "%08d" % int(String.num_int64(num10,2))

# 10进制整数 --> 2进制
func f10_to2(f10:float) -> String:
	return "%s" % String.num_int64(f10,2)
```
