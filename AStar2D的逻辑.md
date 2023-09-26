## 概述
对于2D平台跳跃或飞机大战，以及一些直接用键盘方向键操控玩家的游戏，是根本用不到寻路的，因为只需要检测碰撞就可以了。
但是对于像RTS或战棋这样需要操控玩家到地图指定位置的移动方式，就绝对绕不开寻路了。
### 导航、碰撞与寻路
在Godot中**导航**(navigation)可以被理解为是可通行区域。
而**碰撞**(collision)是体积，指代障碍物，提示“不可通行”。
所以可以把导航和碰撞看做是反义的。也可以看做是0和1，true和false。有导航的地方就能通行，有碰撞的地方就不能通形。
而**寻路**(Pathfinding)则是指在可通行区域和不可通行区域中找出一条可以行走的路径，而且这条路径往往是最短的。
### 寻路算法
任何计算机问题，都会有多种不同的编程解法，计算机问题的编程解法就可以称为“算法”。而算法是跨语言的，同样的算法你可以用不同编程语言实现。
对于寻路问题，也会有不同的编程解法，也就是不同寻路算法。
A*(A-Star)就是前辈大神们创造的寻路算法之一。它的特点是基于网格，而且可以快速的求解某个点到另一个点的最短有效路径。
Godot的`AStar`是封装了A*(A-Star)算法的类，2D版本的`AStar2D`、`AStarGrid2D`也是如此。封装的好处是你不必从头实现算法，而是专注于使用。
![Godot4.0中A*算法相关的类](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686499459705-908afa1b-5e63-4472-a09b-a24d66674149.png#averageHue=%2329313c&clientId=ua755ab23-f3c8-4&from=paste&height=124&id=u0a760c53&originHeight=371&originWidth=957&originalType=binary&ratio=3&rotation=0&showTitle=true&size=33010&status=done&style=none&taskId=u9eaa479c-3064-4996-929c-3abc08feba1&title=Godot4.0%E4%B8%ADA%2A%E7%AE%97%E6%B3%95%E7%9B%B8%E5%85%B3%E7%9A%84%E7%B1%BB&width=319 "Godot4.0中A*算法相关的类")
## Godot的AStar2D
`AStar2D`的使用思路是：

- 添加可以到达的位置
- 将可以行走的点两两连接，形成路径
- 通过其方法直接求取某个位置到目标位置的最短路径
- 让玩家或其他角色按照路径上点的顺序依次前进，直到到达目标位置

![AStar2D的整体思路](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686404435357-a0cde4a8-ff15-407a-9fc7-7b4a27a2efac.png#averageHue=%23f0e7e2&clientId=u7e95a6d7-0a15-4&from=paste&height=240&id=u0496636d&originHeight=721&originWidth=2488&originalType=binary&ratio=3&rotation=0&showTitle=true&size=114839&status=done&style=none&taskId=u68a0d796-48c6-452d-981f-bc2f0762218&title=AStar2D%E7%9A%84%E6%95%B4%E4%BD%93%E6%80%9D%E8%B7%AF&width=829.3333333333334 "AStar2D的整体思路")
以下是对应上图的代码实现：
```swift
extends Node2D

var astar = AStar2D.new() # 实例化

func _ready():
	# 添加可以到达的位置
	astar.add_point(0,Vector2(0,0))
	astar.add_point(1, Vector2(0, 0))
	astar.add_point(2, Vector2(0, 1), 1) # 默认权重为 1
	astar.add_point(3, Vector2(1, 1))
	astar.add_point(4, Vector2(2, 0))
	# 在点之间创建连接，形成路径
	astar.connect_points(1, 2, false)
	astar.connect_points(2, 3, false)
	astar.connect_points(4, 3, false)
	astar.connect_points(1, 4, false)
	# 查询某两个位置之间的路径
	var res = astar.get_id_path(1, 4) # [1,4]
```
AStar2D就是如此简单。

- `add_point()`的时候传入了一个ID，可以将其想象为是一个唯一索引值，对点的标记。
- `get_id_path()`方法获取的是两个对应ID的点之间的最短路径，返回的是包含路径经过的所有点的ID所组成的数组。
- 你也可以用`get_point_path()`方法直接获取两个点之间的最短路径，返回的额是包含所有经过的点数组。
### 定义网格
你可以看到，如果是单纯的使用`Vector2(0,0)`到`Vector2(2,1)`这样的坐标是毫无意义的，因为它们只代表屏幕上一个很小的像素区域，根本无法实现移动。
回过头看看上面对A*(A-Star)算法的描述:
> A*(A-Star)就是前辈大神们创造的寻路算法之一。它的特点是**基于网格**，而且可以快速的求解某个点到另一个点的最短有效路径。

可以看到它是“基于网格”的。所以我们要使用`AStar2D`就需要基于网格。
这个网格可以是你自己用代码创建的，也可以是基于`TileMap`这样现成的网格体系。
比如如下代码，我们自定义了一个网格，并在屏幕上绘制。
```swift
extends Node2D

var astar = AStar2D.new() # 实例化
# 定义网格
var grid_size = Vector2i(32,32) # 尺寸 - 有多少行、多少列
var cell_size = Vector2i(32,32) # 单元格大小


func _ready():
	# 添加可以到达的位置
	astar.add_point(0,Vector2(0,0))
	astar.add_point(1, Vector2(0, 0))
	astar.add_point(2, Vector2(0, 1), 1) # 默认权重为 1
	astar.add_point(3, Vector2(1, 1))
	astar.add_point(4, Vector2(2, 0))
	# 在点之间创建连接，形成路径
	astar.connect_points(1, 2, false)
	astar.connect_points(2, 3, false)
	astar.connect_points(4, 3, false)
	astar.connect_points(1, 4, false)
	# 查询某两个位置之间的路径
	var res = astar.get_id_path(1, 4) # [1,4]

func _draw():
	# 绘制网格
	for x in grid_size.x:
		for y in grid_size.y:
			draw_rect(Rect2i(Vector2i(x,y) * cell_size,cell_size),Color.YELLOW,false,1)

```
运行效果：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686500508632-d63e168c-b223-4048-a250-88234cf2b5b5.png#averageHue=%234d4d4d&clientId=ua755ab23-f3c8-4&from=paste&height=527&id=u0f63faa4&originHeight=1581&originWidth=1562&originalType=binary&ratio=3&rotation=0&showTitle=false&size=14937&status=done&style=none&taskId=u564fa43b-535a-43b1-b563-1f19ce6985e&title=&width=520.6666666666666)
### 绘制Astar的点和路径到网格
```swift
extends Node2D

var astar = AStar2D.new() # 实例化
# 定义网格
var grid_size = Vector2i(32,32) # 尺寸 - 有多少行、多少列
var cell_size = Vector2i(32,32) # 单元格大小


func _ready():
	# 添加可以到达的位置
	astar.add_point(0,Vector2(0,0))
	astar.add_point(1, Vector2(0, 0))
	astar.add_point(2, Vector2(0, 1), 1) # 默认权重为 1
	astar.add_point(3, Vector2(1, 1))
	astar.add_point(4, Vector2(2, 0))
	# 在点之间创建连接，形成路径
	astar.connect_points(1, 2, false)
	astar.connect_points(2, 3, false)
	astar.connect_points(4, 3, false)
	astar.connect_points(1, 4, false)
	# 查询某两个位置之间的路径
	var res = astar.get_id_path(1, 4) # [1,4]

func _draw():
	# 绘制网格
	for x in grid_size.x:
		for y in grid_size.y:
			draw_rect(Rect2i(Vector2i(x,y) * cell_size,cell_size),Color.YELLOW,false,1)
	# 绘制点
	for i in range(astar.get_point_count()):
		var pos = astar.get_point_position(i) * Vector2(cell_size) + Vector2(cell_size/2)
		draw_circle(pos,10,Color.YELLOW)
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686500976321-3e750f6f-418a-464f-83a9-0e060dbcd878.png#averageHue=%234d4d4d&clientId=ua755ab23-f3c8-4&from=paste&height=527&id=u144a14e9&originHeight=1582&originWidth=1555&originalType=binary&ratio=3&rotation=0&showTitle=false&size=15260&status=done&style=none&taskId=u2e602729-9470-41b4-9c2f-c08d7ce5a83&title=&width=518.3333333333334)
```swift
extends Node2D

var astar = AStar2D.new() # 实例化
# 定义网格
var grid_size = Vector2i(32,32) # 尺寸 - 有多少行、多少列
var cell_size = Vector2i(32,32) # 单元格大小


func _ready():
	# 添加可以到达的位置
	astar.add_point(0,Vector2(0,0))
	astar.add_point(1, Vector2(0, 0))
	astar.add_point(2, Vector2(0, 1), 1) # 默认权重为 1
	astar.add_point(3, Vector2(1, 1))
	astar.add_point(4, Vector2(2, 0))
	# 在点之间创建连接，形成路径
	astar.connect_points(1, 2, false)
	astar.connect_points(2, 3, false)
	astar.connect_points(4, 3, false)
	astar.connect_points(1, 4, false)
	# 查询某两个位置之间的路径
	var res = astar.get_point_connections(1)
	print(res)

func _draw():
	# 绘制网格
	for x in grid_size.x:
		for y in grid_size.y:
			draw_rect(Rect2i(Vector2i(x,y) * cell_size,cell_size),Color.YELLOW,false,1)
	# 绘制点
	for i in range(astar.get_point_count()):
		var pos = get_grid_pos(astar.get_point_position(i))
		draw_circle(pos,5,Color.YELLOW)
	# 绘制所有路径
	for i in range(astar.get_point_count()):
		var pos = get_grid_pos(astar.get_point_position(i))
		if i+1 <= astar.get_point_count():
			var pos2 = get_grid_pos(astar.get_point_position(i+1))
			draw_line(pos,pos2,Color.GREEN_YELLOW,2)
		
# 返回屏幕中的点或路径中的点对应在网格中的坐标
func get_grid_pos(point_pos:Vector2):
	return point_pos * Vector2(cell_size) + Vector2(cell_size/2)

```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686501772273-7e13fd74-8c10-4048-8114-7b9be5d4e2e2.png#averageHue=%234d4d4d&clientId=ua755ab23-f3c8-4&from=paste&height=523&id=u7201ee1c&originHeight=1569&originWidth=1549&originalType=binary&ratio=3&rotation=0&showTitle=false&size=15130&status=done&style=none&taskId=uadfe4b8a-e7ed-4624-92c2-f7fc6e2a864&title=&width=516.3333333333334)
```swift
extends Node2D

var astar = AStar2D.new() # 实例化
# 定义网格
var grid_size = Vector2i(32,32) # 尺寸 - 有多少行、多少列
var cell_size = Vector2i(32,32) # 单元格大小


func _ready():
	# 添加可以到达的位置
	astar.add_point(0,Vector2(0,0))
	astar.add_point(1, Vector2(0, 0))
	astar.add_point(2, Vector2(0, 1), 1) # 默认权重为 1
	astar.add_point(3, Vector2(1, 1))
	astar.add_point(4, Vector2(2, 0))
	# 在点之间创建连接，形成路径
	astar.connect_points(1, 2, false)
	astar.connect_points(2, 3, false)
	astar.connect_points(4, 3, false)
	astar.connect_points(1, 4, false)
	# 查询某两个位置之间的路径
	var res = astar.get_point_connections(1)
	print(res)

func _draw():
	# 绘制网格
	for x in grid_size.x:
		for y in grid_size.y:
			draw_rect(Rect2i(Vector2i(x,y) * cell_size,cell_size),Color.YELLOW,false,1)
	# 绘制点
	for i in range(astar.get_point_count()):
		var pos = get_grid_pos(astar.get_point_position(i))
		draw_circle(pos,5,Color.YELLOW)
	# 绘制所有路径
	for i in range(astar.get_point_count()):
		var pos = get_grid_pos(astar.get_point_position(i))
		if i+1 <= astar.get_point_count():
			var pos2 = get_grid_pos(astar.get_point_position(i+1))
			draw_line(pos,pos2,Color.GREEN_YELLOW,2)
	# 绘制寻找到的路径
	var path = astar.get_point_path(1,4)
	for i in path.size()-1:
		var pos = get_grid_pos(path[i])
		if i+1 <= path.size():
			var pos2 = get_grid_pos(path[i+1])
			draw_line(pos,pos2,Color.RED,2)
	
	

# 返回屏幕中的点或路径中的点对应在网格中的坐标
func get_grid_pos(point_pos:Vector2):
	return point_pos * Vector2(cell_size) + Vector2(cell_size/2)

```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686502100111-bb628a37-ab09-4ada-8389-a0620ff73fb1.png#averageHue=%234d4d4d&clientId=ua755ab23-f3c8-4&from=paste&height=525&id=u42efa3a6&originHeight=1574&originWidth=1552&originalType=binary&ratio=3&rotation=0&showTitle=false&size=15287&status=done&style=none&taskId=ufc844773-7c76-4167-976c-51263b51176&title=&width=517.3333333333334)
## 建立障碍物

