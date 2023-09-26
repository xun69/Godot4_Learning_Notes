文档生成时间：2023-07-02 00:40:30
## 基本信息
| 项目 | 信息 |
| --- | --- |
| 文件名 | Grid.gd |
| Godot版本 | 4.0.3-stable (official) |
| is_tool | true |
| extend | Node2D |
| class_name | Grid2D |
| 所在项目名称 | 音频可视化 |
| 文件路径 | res://Grid.gd |
| 绝对路径 | E:/Godot4.x - 软件/音频可视化/Grid.gd |
| 作者 | 巽星石 |

## 概述
2D参数化网格节点Grid2D。基于Node2D扩展，可以实时参数化的设定和显示网格。目前版本还非常基础。
## 变量
| 变量 | 类型 | 描述 |
| --- | --- | --- |
| grid_size | Vector2i | 网格尺寸 - 有多少行、多少列 |
| cell_size | Vector2i | 单元格大小 |
| grid_color | Color | 网格颜色 |
| grid_border_width | int | 网格边框宽度 |

## 方法
### get_cell_rect(cell_pos:Vector2i) -> void
网格坐标 -> 单元格的矩形。
### get_cell_center(cell_pos:Vector2) -> void
网格坐标 -> 单元格中心点。
### get_grid_pos(screen_pos:Vector2) -> void
屏幕坐标 -> 网格坐标。
## 源代码
以下是完整源代码：
```swift
# ========================================================
# 名称：Grid2D
# 类型：Node2D扩展
# 简介：扩展自Node2D的网格节点，提供基本的网格绘制和坐标转换。
#      可以在不用TileMap的情况下实现一些网格的效果。
# 作者：巽星石
# Godot版本：4.0.3-stable (official)
# 创建时间：2023-07-02 00:32:48
# 最后修改时间：2023-07-02 00:32:48
# ========================================================
@tool
extends Node2D
class_name Grid2D
# =================================== 参数 ===================================
## 网格尺寸 - 有多少行、多少列
@export var grid_size = Vector2i(32,32):
	set(val):
		grid_size = val
		queue_redraw()

## 单元格大小
@export var cell_size = Vector2i(32,32):
	set(val):
		cell_size = val
		queue_redraw()

## 网格颜色
@export var grid_color:Color = Color("#a49b7858"):
	set(val):
		grid_color = val
		queue_redraw()

## 网格边框宽度
@export var grid_border_width:int = 1:
	set(val):
		grid_border_width = val
		queue_redraw()

# =================================== 虚函数 ===================================
func _draw():
	# 绘制网格
	for x in grid_size.x:
		for y in grid_size.y:
			draw_rect(get_cell_rect(Vector2(x,y)),grid_color,false,grid_border_width)

# =================================== 方法 ===================================
# 网格坐标 -> 单元格的矩形
func get_cell_rect(cell_pos:Vector2i):
	return Rect2(cell_pos * cell_size,cell_size)

# 网格坐标 -> 单元格中心点
func get_cell_center(cell_pos:Vector2):
	return cell_pos * Vector2(cell_size) + Vector2(cell_size/2)
	
# 屏幕坐标 -> 网格坐标
func get_grid_pos(screen_pos:Vector2):
	return floor(screen_pos / Vector2(cell_size))
```
