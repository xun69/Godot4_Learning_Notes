## 测试素材下载
[Hexagon Pack · Kenney](https://www.kenney.nl/assets/hexagon-pack)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686945407410-73071b85-7228-4cec-a630-2da1933ca530.png#averageHue=%234684ae&clientId=uf58dd381-1131-4&from=paste&id=u4ac64a34&originHeight=515&originWidth=918&originalType=url&ratio=3&rotation=0&showTitle=false&size=373799&status=done&style=none&taskId=u3267fa52-f0dc-4861-b315-4e17baaa394&title=)
## 简单绘制
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686940877950-8e23e81e-a832-4405-bf87-63046db28226.png#averageHue=%2329a65e&clientId=uf58dd381-1131-4&from=paste&height=632&id=u538e99ec&originHeight=1895&originWidth=2828&originalType=binary&ratio=3&rotation=0&showTitle=false&size=681127&status=done&style=none&taskId=ub4399e9e-7d6a-4754-a9dd-a3504c47d6c&title=&width=942.6666666666666)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686940931261-450f6d40-d4a0-4242-99fe-502dea583e5d.png#averageHue=%233c4a50&clientId=uf58dd381-1131-4&from=paste&height=388&id=u38fbc9ad&originHeight=1164&originWidth=2346&originalType=binary&ratio=3&rotation=0&showTitle=false&size=381612&status=done&style=none&taskId=u0c5b370b-f7b0-40c4-b843-c591c985cdf&title=&width=782)
## 显示网格坐标
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686941765327-278d1745-c46e-42af-ba00-1d804a08c558.png#averageHue=%23504d4b&clientId=uf58dd381-1131-4&from=paste&height=323&id=u5fef311a&originHeight=969&originWidth=1341&originalType=binary&ratio=3&rotation=0&showTitle=false&size=133890&status=done&style=none&taskId=u619adbdc-08cd-4afe-aa21-5b2fc28c3c9&title=&width=447)
```swift
extends TileMap
@onready var label = $Label


func _ready():
	# 显示网格的坐标
	var cells = get_used_cells(0)
	for cell in cells:
		var lab = label.duplicate()
		lab.text = str(cell)
		lab.position = map_to_local(cell) - Vector2(30,20)
		lab.show()
		add_child(lab)

func _input(event):
	if event is InputEventMouseMotion:
		label.text = str(local_to_map(get_global_mouse_position()))
```

- 通过`local_to_map()`可以将屏幕坐标转化为`TileMap`的网格坐标。
- 而`map_to_local()`则正好是相反的操作，可以将`TileMap`的网格坐标转化为屏幕坐标(单元格中心点的位置)。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686941780376-ff3c68b2-bc10-489b-9598-818546b36293.png#averageHue=%23605550&clientId=uf58dd381-1131-4&from=paste&height=248&id=ueba74780&originHeight=743&originWidth=786&originalType=binary&ratio=3&rotation=0&showTitle=false&size=124246&status=done&style=none&taskId=u73f45a23-4fc1-4401-a9cf-56bfc4773d9&title=&width=262)
## 获取单元格周边的网格
通过`get_surrounding_cells()`可以获取指定单元格周边的所有单元格坐标。
```swift
extends TileMap
@onready var label = $Label


func _ready():
	# 显示网格的坐标
	var cells = get_surrounding_cells(Vector2i(8,5))
	for cell in cells:
		var lab = label.duplicate()
		lab.text = str(cell)
		lab.position = map_to_local(cell) - Vector2(30,20)
		lab.show()
		add_child(lab)
```
![获取中心单元格周边的单元格](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686942344456-33d2ae2c-95bd-4043-bdf9-e9e6c4db63bf.png#averageHue=%23e3d4b0&clientId=uf58dd381-1131-4&from=paste&height=212&id=u75d796db&originHeight=636&originWidth=693&originalType=binary&ratio=3&rotation=0&showTitle=true&size=90989&status=done&style=none&taskId=ub684cc06-80e6-4eda-93e6-1f0e4eb2900&title=%E8%8E%B7%E5%8F%96%E4%B8%AD%E5%BF%83%E5%8D%95%E5%85%83%E6%A0%BC%E5%91%A8%E8%BE%B9%E7%9A%84%E5%8D%95%E5%85%83%E6%A0%BC&width=231 "获取中心单元格周边的单元格")
### 获取几步内的所有单元格坐标
```swift
func _ready():
	var pos := Vector2i(8,5)
	# 获取指定单元格周围几步内的所有单元格坐标
	var cells:Array[Vector2i] 
	cells = get_surrounds(cells,pos,2)
	# 显示网格的坐标
	for cell in cells:
		var lab = label.duplicate()
		lab.text = str(cell)
		lab.position = map_to_local(cell) - Vector2(30,20)
		lab.show()
		add_child(lab)

# 返回周围几步内的单元格坐标
func get_surrounds(cells:Array[Vector2i],pos:Vector2i,range:int,deep:int = 0) -> Array[Vector2i]:
	deep += 1
	var cells_1 = get_surrounding_cells(pos)
	for cell in cells_1:
		if cell not in cells and cell != pos:
			cells.append(cell)
		if deep < range:
			cells = get_surrounds(cells,cell,range,deep)
	return cells
```
通过简单的递归，获取几步内的所有单元格坐标。后期通过判断不添加不能通行的单元格就可以了。
![步幅 = 2](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686942887118-4c0633cb-9df1-49de-9d46-c932cc690804.png#averageHue=%23bf7656&clientId=uf58dd381-1131-4&from=paste&height=224&id=uca889071&originHeight=672&originWidth=720&originalType=binary&ratio=3&rotation=0&showTitle=true&size=119566&status=done&style=none&taskId=u4a1e0007-947c-47d8-b986-e45484f0383&title=%E6%AD%A5%E5%B9%85%20%3D%202&width=240 "步幅 = 2")![步幅 = 3](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686944535151-70a75cd5-9d9b-46cc-8769-a754276e5290.png#averageHue=%23605653&clientId=uf58dd381-1131-4&from=paste&height=252&id=u246b662d&originHeight=757&originWidth=880&originalType=binary&ratio=3&rotation=0&showTitle=true&size=151606&status=done&style=none&taskId=ufc9ccab6-23c9-492a-af93-25a5c2d361d&title=%E6%AD%A5%E5%B9%85%20%3D%203&width=293.3333333333333 "步幅 = 3")
## 在六边形地图中使用AStar2D
`AStarGrid2D`显然不太适合六边形网格。所以只能退而求其次，选择`AStar2D`。
因此需要遍历整个地图来创建可以到达的位置和连线。可以判断单元格的图块有没有导航区域，如果有就可以通行，如果没有，或者设置了碰撞，就视为障碍物。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686945157368-e92e1fe3-173d-4a7a-86eb-c7ce83771a93.png#averageHue=%23bb7a58&clientId=uf58dd381-1131-4&from=paste&height=350&id=u29c80bf5&originHeight=1318&originWidth=1363&originalType=binary&ratio=3&rotation=0&showTitle=false&size=634865&status=done&style=none&taskId=uc84997dd-2908-4c0a-b3c5-dd6fa8ac5fd&title=&width=362.3333435058594)
为一些无法通行的图块设置物理层。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686945545665-6b86c516-7604-4c71-941b-4f5096383dcc.png#averageHue=%233a484e&clientId=uf58dd381-1131-4&from=paste&height=329&id=u5fe956a7&originHeight=986&originWidth=2236&originalType=binary&ratio=3&rotation=0&showTitle=false&size=352933&status=done&style=none&taskId=ue829dd0d-fede-4fff-9912-c029101a5a3&title=&width=745.3333333333334)
然后为另一些可通行图块，添加导航形状。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686945655486-3f00d4b1-3f81-47e4-8515-dc99c7505b61.png#averageHue=%233b434c&clientId=uf58dd381-1131-4&from=paste&height=360&id=u1ffa1138&originHeight=1081&originWidth=2363&originalType=binary&ratio=3&rotation=0&showTitle=false&size=331267&status=done&style=none&taskId=ue9e17a93-4c3e-4199-8a92-237216f47dd&title=&width=787.6666666666666)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1687000708958-e5fe2f57-59f1-4b14-953e-3f7aec614ab8.png#averageHue=%232d343f&clientId=ue20e433b-e8b9-4&from=paste&height=104&id=ub583f23d&originHeight=312&originWidth=473&originalType=binary&ratio=3&rotation=0&showTitle=false&size=22324&status=done&style=none&taskId=u69fc51c4-f5a4-430d-8fcd-8ce7271208c&title=&width=157.66666666666666)
```swift
# TileMap.gd
extends TileMap
@onready var label = $Label

var astar = AStar2D.new()
@onready var debug = $debug

func _ready():
	
	var pos := Vector2i(8,5)
	# 获取所有已经绘制的单元格
	var used_cells:Array[Vector2i] = get_used_cells(0)
	# 遍历所有网格，将带有导航区域的单元格位置添加为AStar的可通行点
	for cell in used_cells:
		var data = get_cell_tile_data(0,cell)
		if data:
			if data.get_navigation_polygon(0): # 有导航区域
				# 设为可通行点
				var id = astar.get_available_point_id()
				astar.add_point(id,cell)
				show_cell_id(cell,id)
		
	# 遍历已经添加了的可通行点,进行连线
	var points_ids = astar.get_point_ids()
	for id in points_ids:
		if astar.is_point_disabled(id):
			return
		var cel_pos = astar.get_point_position(id) # 转为坐标
		var surrd = get_surrounding_cells(cel_pos) # 获取周边6个位置
		for cel in surrd:
			var cel_id = astar.get_closest_point(cel)
			if Vector2i(astar.get_point_position(cel_id)) in surrd:
				if cel_id in points_ids:
					if !astar.are_points_connected(cel_id,id):
						astar.connect_points(cel_id,id)
	debug.queue_redraw()


func show_cell_lab(cell:Vector2):
	var lab = label.duplicate()
	lab.text = str(cell)
	lab.position = map_to_local(cell) - Vector2(30,20)
	lab.show()
	add_child(lab)

func show_cell_id(cell:Vector2,id:int):
	var lab = label.duplicate()
	lab.text = str(id)
	lab.position = map_to_local(cell) - Vector2(30,20)
	lab.show()
	add_child(lab)


# 返回周围几步内的单元格坐标
func get_surrounds(cells:Array[Vector2i],pos:Vector2i,range:int,deep:int = 0) -> Array[Vector2i]:
	deep += 1
	var cells_1 = get_surrounding_cells(pos)
	for cell in cells_1:
		if cell not in cells and cell != pos:
			cells.append(cell)
		if deep < range:
			cells = get_surrounds(cells,cell,range,deep)
	return cells


func _input(event):
	if event is InputEventMouseMotion:
		label.text = str(local_to_map(get_global_mouse_position()))

```
```swift
# debug.gd
extends Control

var astar:AStar2D
@export var tileMap:NodePath

func _ready():
	astar = get_parent().astar

func _draw():
	var map:TileMap = get_node(tileMap)
	var points_ids = astar.get_point_ids()
	for point_id in points_ids:
		var cel_pos = astar.get_point_position(point_id) # 转为单元格位置坐标
		var cel_screen_pos = map.map_to_local(cel_pos)   # 转为屏幕坐标
		draw_circle(map.map_to_local(cel_pos),5,Color.YELLOW_GREEN)
		# 获取连接信息
		var connects = astar.get_point_connections(point_id)
		for cnt_point_id in connects:
			var cnt_pos = astar.get_point_position(cnt_point_id) # 转为单元格位置坐标
			var cnt_screen_pos = map.map_to_local(cnt_pos)   # 转为屏幕坐标
			draw_line(cel_screen_pos,cnt_screen_pos,Color.YELLOW_GREEN,1)
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1687000661835-1a748559-2a78-49ef-b1e6-22b665587215.png#averageHue=%23bec977&clientId=ue20e433b-e8b9-4&from=paste&height=394&id=u3de83f5d&originHeight=1181&originWidth=1012&originalType=binary&ratio=3&rotation=0&showTitle=false&size=195960&status=done&style=none&taskId=u5893bf10-78de-46ad-958b-8dbe390dd87&title=&width=337.3333333333333)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1687007569213-40364e68-ed7f-4247-97dc-966bf0cc209d.png#averageHue=%23b87247&clientId=ue20e433b-e8b9-4&from=paste&height=388&id=u04f72502&originHeight=1164&originWidth=1136&originalType=binary&ratio=3&rotation=0&showTitle=false&size=263180&status=done&style=none&taskId=uf90455ad-d30a-4cca-b4b4-44a057d5f86&title=&width=378.6666666666667)

