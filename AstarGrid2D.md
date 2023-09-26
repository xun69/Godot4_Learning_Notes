## 概述
`AstarGrid2D`是Godot4.0新增的A*寻路辅助类型。可以看做是`Astar2D`的加强版。
它允许你通过设置其`size`和`cell_size`属性来创建一个虚拟的网格。
并使用`set_point_solid()`这样的方法来在指定位置创建障碍物。
`AstarGrid2D`的好处是你不再需要手动的添加点以及点与点之间的连接，而是直接用get_point_path()这样的方法来获取最短路径（也就是一个包含了最短路径经过的点的数组）。
通过遍历这个数组，就可以实现路径移动了。
```swift
extends Control

var astar_grid = AStarGrid2D.new()

var cell_size = Vector2.ONE * 100

func _ready():
	astar_grid.size = Vector2i.ONE * 32
	astar_grid.cell_size = Vector2i.ONE * 32
	astar_grid.update()
	


func _draw():
	# 绘制网格
	var grid_width = astar_grid.size.x * astar_grid.cell_size.x
	var cell_width = astar_grid.cell_size.x
	var cell_height = astar_grid.cell_size.y
	
	for i in range(astar_grid.size.x):
		draw_line(i * Vector2i(0,cell_height),i * Vector2i(grid_width,cell_height),Color.DARK_OLIVE_GREEN,2)
	for j in range(astar_grid.size.y):
		draw_line(j * Vector2i(cell_height,0),j * Vector2i(cell_height,grid_width),Color.DARK_OLIVE_GREEN,2)
	# 绘制路径和其上的点
	var path  = astar_grid.get_point_path(Vector2i(0,0),Vector2i(10,10))
	for pot in path:
		draw_circle(pot,5,Color.YELLOW)
	draw_polyline(path,Color.YELLOW,2)

```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686405993933-7e9f3e46-6e2c-4d9e-8e13-a5c17d0cc707.png#averageHue=%23585858&clientId=u1635b161-9f36-4&from=paste&height=334&id=u39e28146&originHeight=1094&originWidth=1013&originalType=binary&ratio=3&rotation=0&showTitle=false&size=17474&status=done&style=none&taskId=u6d4f957a-ad35-4864-9466-8d566367555&title=&width=309.66668701171875)
```swift
extends Control

var astar_grid = AStarGrid2D.new()

var path:PackedVector2Array

var solids = []

func _ready():
	randomize()
	astar_grid.size = Vector2i.ONE * 32
	astar_grid.cell_size = Vector2i.ONE * 32
	astar_grid.offset = astar_grid.cell_size/2
	astar_grid.update()
	
	# 随机生成障碍
	for i in range(50):
		var solid_point = Vector2i(randi_range(0,astar_grid.size.x),randi_range(0,astar_grid.size.y))
		astar_grid.set_point_solid(solid_point,true)
		solids.append(solid_point)
		
	


func _draw():
	var grid_width = astar_grid.size.x * astar_grid.cell_size.x
	var cell_width = astar_grid.cell_size.x
	var cell_height = astar_grid.cell_size.y
	# 绘制网格
	for i in range(astar_grid.size.x):
		draw_line(i * Vector2i(0,cell_height),i * Vector2i(grid_width,cell_height),Color.DARK_OLIVE_GREEN,2)
	for j in range(astar_grid.size.y):
		draw_line(j * Vector2i(cell_height,0),j * Vector2i(cell_height,grid_width),Color.DARK_OLIVE_GREEN,2)
	
	# 绘制路径和其上的点
	if path.size() > 0:
		for pot in path:
			draw_circle(pot,5,Color.YELLOW)
				
		draw_polyline(path,Color.YELLOW,2)
	for p in solids:
		draw_rect(Rect2(p * Vector2i(astar_grid.cell_size),astar_grid.cell_size),Color.GRAY)

func _on_gui_input(event):
	if event is InputEventMouseButton:
		if event.button_index == MOUSE_BUTTON_LEFT:
			if event.is_pressed():
				path = astar_grid.get_point_path(Vector2i(0,0),floor(get_global_mouse_position()/astar_grid.cell_size))
				queue_redraw()

```

- 13行：将`AstarGrid`的`offset` 设为 `astar_grid.cell_size/2`,也就实现了整体的坐标偏移。
- 48行：`floor(get_global_mouse_position()/astar_grid.cell_size)`获得的就是鼠标点击所在的单元格的坐标。

![AstarGrid寻路2.gif](https://cdn.nlark.com/yuque/0/2023/gif/8438332/1686408190219-c861942c-6968-4cec-8097-85adc49a8b99.gif#averageHue=%235b5b5b&clientId=ub109aed3-0d12-4&from=drop&height=350&id=u9b477ede&originHeight=1205&originWidth=1268&originalType=binary&ratio=3&rotation=0&showTitle=false&size=338879&status=done&style=none&taskId=u30ec7f11-9818-462d-836c-0c0378a79d3&title=&width=368)
目前为止，`AstarGrid`显示对角线形式的连接。
如果我们需要设置其只显示横平竖直的连接，则可以设置`AstarGrid`的`diagonal_mode`为`DIAGONAL_MODE_NEVER`。
```swift
astar_grid.diagonal_mode = AStarGrid2D.DIAGONAL_MODE_NEVER
```
![AstarGrid寻路3.gif](https://cdn.nlark.com/yuque/0/2023/gif/8438332/1686408407721-1824f56d-4530-4363-b68b-a97359fc5079.gif#averageHue=%235b5b5b&clientId=ub109aed3-0d12-4&from=drop&height=415&id=uf6786afa&originHeight=1205&originWidth=1268&originalType=binary&ratio=3&rotation=0&showTitle=false&size=370769&status=done&style=none&taskId=uc687ca98-e098-415e-9c7d-8e92cc56369&title=&width=437)

| DiagonalMode | 值 | 说明 |
| --- | --- | --- |
| DIAGONAL_MODE_ALWAYS  | 0 | 该寻路算法将忽略目标单元格周围的实体邻居，并允许沿对角线通过。 |
| DIAGONAL_MODE_NEVER  | 1 | 该寻路算法将**忽略所有对角线，并且路径始终是正交的。** |
| DIAGONAL_MODE_AT_LEAST_ONE_WALKABLE | 2 | 如果在特定路径段的相邻单元格周围放置了至少两个障碍物，则该寻路算法将避免使用对角线。 |
| DIAGONAL_MODE_ONLY_IF_NO_OBSTACLES  | 3 | 如果在特定路径段的相邻单元格周围放置了任意障碍物，则该寻路算法将避免使用对角线。 |
| DIAGONAL_MODE_MAX  | 4 | 代表 DiagonalMode 枚举的大小。 |

## 实现玩家基于AstarGrid2D的移动
用一个简单`Sprite2D`作为玩家。将其缩放为原始尺寸`128×128`的四分之一，也就是`32×32`。刚好可以填入`AstarGrid2D`的单元格中。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686410355536-6d898801-d65e-408e-afbf-3c007be09ac0.png#averageHue=%2328303b&clientId=u6e4379d4-9442-4&from=paste&height=54&id=u743e29c4&originHeight=162&originWidth=555&originalType=binary&ratio=3&rotation=0&showTitle=false&size=9402&status=done&style=none&taskId=u43418aaf-6cb7-4535-a123-fd74a2ef2dc&title=&width=185)![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686410394261-b5ed8b3d-d331-452e-a1c5-5874df03ec89.png#averageHue=%232a2e36&clientId=u6e4379d4-9442-4&from=paste&height=163&id=ubcec1235&originHeight=490&originWidth=419&originalType=binary&ratio=3&rotation=0&showTitle=false&size=31954&status=done&style=none&taskId=u54b372df-d4d3-4a43-b8a7-8b94adf2a57&title=&width=139.66666666666666)
```swift
extends Control

var astar_grid = AStarGrid2D.new()
var path:PackedVector2Array
var solids = [] # 障碍物列表

@onready var icon = $Icon
var speed = 200.0
var can_walk = false
var target_pos:Vector2

func _process(delta):
	if can_walk:
		if path.size()>0:
			target_pos = path[0]
			if icon.position.distance_to(target_pos)>0:
				icon.position = icon.position.move_toward(target_pos,speed * delta)
			else:
				path.remove_at(0)
				queue_redraw()
		else:
			can_walk = false

func _ready():
	randomize()
	astar_grid.size = Vector2i.ONE * 32
	astar_grid.cell_size = Vector2i.ONE * 32
	astar_grid.offset = astar_grid.cell_size/2
	icon.position = astar_grid.cell_size/2
	astar_grid.diagonal_mode = AStarGrid2D.DIAGONAL_MODE_NEVER
	astar_grid.default_compute_heuristic  = AStarGrid2D.HEURISTIC_MANHATTAN

	astar_grid.update()
	
	# 随机生成障碍
	for i in range(150):
		var solid_point = Vector2i(randi_range(0,astar_grid.size.x),randi_range(0,astar_grid.size.y))
		astar_grid.set_point_solid(solid_point,true)
		solids.append(solid_point)
		
	


func _draw():
	var grid_width = astar_grid.size.x * astar_grid.cell_size.x
	var cell_width = astar_grid.cell_size.x
	var cell_height = astar_grid.cell_size.y
	# 绘制网格
	for i in range(astar_grid.size.x):
		draw_line(i * Vector2i(0,cell_height),i * Vector2i(grid_width,cell_height),Color.DARK_OLIVE_GREEN,2)
	for j in range(astar_grid.size.y):
		draw_line(j * Vector2i(cell_height,0),j * Vector2i(cell_height,grid_width),Color.DARK_OLIVE_GREEN,2)
	
	# 绘制路径和其上的点
	if path.size() > 0:
		for pot in path:
			draw_circle(pot,5,Color.YELLOW)
				
		draw_polyline(path,Color.YELLOW,2)
	for p in solids:
		draw_rect(Rect2(p * Vector2i(astar_grid.cell_size),astar_grid.cell_size),Color.GRAY)

func _on_gui_input(event):
	if event is InputEventMouseButton:
		if event.button_index == MOUSE_BUTTON_LEFT:
			if event.is_pressed():
				can_walk = true
				path = astar_grid.get_point_path(floor(icon.position/astar_grid.cell_size),floor(get_global_mouse_position()/astar_grid.cell_size))
				queue_redraw()

```
每次获取路径的第一个点，利用简单的距离判断和移动，到达后，从路径中删除该点。继续获取路径第一个，如此循环，知道路径中的点删除完毕。
![基于AstarGrid2D的简单玩家移动](https://cdn.nlark.com/yuque/0/2023/gif/8438332/1686410646757-1ca26e38-70aa-449b-9f09-782c845274e2.gif#averageHue=%23595959&clientId=u6e4379d4-9442-4&from=drop&height=439&id=u59dfa256&originHeight=1672&originWidth=1660&originalType=binary&ratio=3&rotation=0&showTitle=true&size=1130029&status=done&style=none&taskId=u4eb7d8bf-3d61-4b64-ba70-19ef369d094&title=%E5%9F%BA%E4%BA%8EAstarGrid2D%E7%9A%84%E7%AE%80%E5%8D%95%E7%8E%A9%E5%AE%B6%E7%A7%BB%E5%8A%A8&width=436 "基于AstarGrid2D的简单玩家移动")
## 可行动范围的获取
矩形范围查找法：
先不考虑对角线移动的问题，我们假设玩家可行动点数是4，那么我们只需要遍历一个8×8矩阵的每个点到玩家的是否有可行走的路径，如果有且路径的点数<=4，则标记为可到达的点，否则，标记为不能到达的点。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686413029794-89b78ab5-7266-415b-8064-e381c23e849f.png#averageHue=%23f1f1f1&clientId=uacdde385-0694-4&from=paste&height=263&id=u0240a49e&originHeight=1286&originWidth=1342&originalType=binary&ratio=3&rotation=0&showTitle=false&size=25915&status=done&style=none&taskId=ub3d2d82a-e1a8-4aeb-a4c4-4247ff5b175&title=&width=274.3333435058594)
```swift
extends Control

var astar_grid = AStarGrid2D.new()
var path:PackedVector2Array
var solids = [] # 障碍物列表
var max_step:int = 4 # 玩家单次的最大行动点数
var can_walk_rect:Rect2i
var can_walk_points:PackedVector2Array 

@onready var icon = $Icon
var speed = 200.0
var can_walk = false
var target_pos:Vector2

func _process(delta):
	if can_walk:
		if path.size()>0:
			target_pos = path[0]
			if icon.position.distance_to(target_pos)>0:
				icon.position = icon.position.move_toward(target_pos,speed * delta)
			else:
				path.remove_at(0)
				queue_redraw()
		else:
			can_walk = false
			queue_redraw()

func _ready():
	randomize()
	astar_grid.size = Vector2i.ONE * 32
	astar_grid.cell_size = Vector2i.ONE * 32
	astar_grid.offset = astar_grid.cell_size/2
	icon.position = astar_grid.cell_size/2
	astar_grid.diagonal_mode = AStarGrid2D.DIAGONAL_MODE_NEVER

	astar_grid.update()
	
	# 随机生成障碍
	for i in range(150):
		var solid_point = Vector2i(randi_range(0,astar_grid.size.x),randi_range(0,astar_grid.size.y))
		astar_grid.set_point_solid(solid_point,true)
		solids.append(solid_point)

	
	

func _draw():
	var grid_width = astar_grid.size.x * astar_grid.cell_size.x
	var cell_width = astar_grid.cell_size.x
	var cell_height = astar_grid.cell_size.y
	# 绘制网格
	for i in range(astar_grid.size.x):
		draw_line(i * Vector2i(0,cell_height),i * Vector2i(grid_width,cell_height),Color.DARK_OLIVE_GREEN,2)
	for j in range(astar_grid.size.y):
		draw_line(j * Vector2i(cell_height,0),j * Vector2i(cell_height,grid_width),Color.DARK_OLIVE_GREEN,2)
	
	# 绘制路径和其上的点
	if path.size() > 0:
		for pot in path:
			draw_circle(pot,5,Color.YELLOW)
				
		draw_polyline(path,Color.YELLOW,2)
	# 绘制障碍物
	for p in solids:
		draw_rect(Rect2(p * Vector2i(astar_grid.cell_size),astar_grid.cell_size),Color.GRAY)
	# 绘制可行走范围
	# 遍历矩形
	if !can_walk:
		var player_pos = floor(icon.position/astar_grid.cell_size)
		var top_left = clamp(player_pos - Vector2.ONE * max_step,Vector2.ZERO,player_pos)
		var end =  clamp(player_pos + Vector2.ONE * max_step,player_pos,Vector2(astar_grid.size))
		can_walk_rect = Rect2(top_left,end-top_left) # 获取矩形
		for i in range(can_walk_rect.position.x,can_walk_rect.end.x + 1):
			for j in range(can_walk_rect.position.y,can_walk_rect.end.y + 1):
				var v = Vector2(i,j)
				if astar_grid.get_point_path(player_pos,v).size() <= max_step+1:
					if !astar_grid.is_point_solid(v):
						can_walk_points.append(v)
						draw_rect(Rect2(v * astar_grid.cell_size,astar_grid.cell_size),Color.YELLOW_GREEN,false,2)
	
	
	
func _on_gui_input(event):
	if event is InputEventMouseButton:
		if event.button_index == MOUSE_BUTTON_LEFT:
			if event.is_pressed():
				can_walk = true
				var player_pos = floor(icon.position/astar_grid.cell_size)
				var targ_pos = floor(get_global_mouse_position()/astar_grid.cell_size)
				if targ_pos in can_walk_points: # 如果在可行走的范围内
					path = astar_grid.get_point_path(player_pos,targ_pos)
					can_walk_points.clear() # 清空原来的可行走范围
					queue_redraw()

```
![AstarGrid寻路-显示和限定行走范围.gif](https://cdn.nlark.com/yuque/0/2023/gif/8438332/1686416068496-e22c9cad-422c-4aa0-975b-2257214b89d8.gif#averageHue=%23606060&clientId=uacdde385-0694-4&from=drop&height=477&id=u977ac3ae&originHeight=1670&originWidth=1660&originalType=binary&ratio=3&rotation=0&showTitle=false&size=655530&status=done&style=none&taskId=u56d476b7-6b10-481a-9365-5cc9f23fc00&title=&width=474)
![AstarGrid寻路-显示和限定行走范围2.gif](https://cdn.nlark.com/yuque/0/2023/gif/8438332/1686416500499-820fb133-7276-4fb8-b624-aad026c53d3c.gif#averageHue=%23626262&clientId=uacdde385-0694-4&from=drop&height=457&id=u9756b7a1&originHeight=1670&originWidth=1660&originalType=binary&ratio=3&rotation=0&showTitle=false&size=1151846&status=done&style=none&taskId=u013e20fd-d832-4db0-ba26-29adf3d4ea7&title=&width=454)
