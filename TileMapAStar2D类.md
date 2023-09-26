## 概述
Godot提供的`AStar2D`和`AStarGrid2D`基本可以解决所有2D的A*寻路问题：

- 前者提供了基础的A*寻路支持，但是需要手动处理很多内容
- 后者针对基于方形图块的A*寻路，进行了很多自动化的工作，用起来十分简便。但是不使用于六边形、isometric之类的图块

所以针对`AStar2D`和`AStarGrid2D`各自特性和现况，好的办法就是自己基于`AStar2D`扩展一个类似于`AStarGrid2D`自动化完成添加点和连线，又可以不限于仅在方形图块下使用的类型。
## 源代码
```swift
# =====================================================
# TileMapAStar2D
# AStar2D扩展类型，用于为TileMap自动化的创建A*导航网格
# 作者：巽星石
# Godot版本：v4.0.3.stable.official [5222a99f5]
# 创建时间：2023年6月17日18:33:28
# 最后修改时间：2023年6月17日21:13:44
# ====================================================

extends AStar2D
class_name TileMapAStar2D

var _tile_map:TileMap
var _layer_id:int

var _pos_labs:Array[Label] = []
var _id_labs:Array[Label] = []

# 是否用Label显示所有可通行位置的单元格坐标
@export var show_navigation_cells_pos:bool = false:
	set(val):
		show_navigation_cells_pos = val
		if val:
			for cell in get_has_navigation_cells():
				show_cell_pos(cell)
		else:
			for lab in _pos_labs:
				lab.queue_free()
			_pos_labs.clear()


# 是否用Label显示所有可通行位置的单元格坐标
@export var show_navigation_cells_id:bool = false:
	set(val):
		show_navigation_cells_id = val
		if val:
			for point_id in get_point_ids():
				var cell = Vector2i(get_point_position(point_id))
				if cell in get_has_navigation_cells():
					show_cell_id(cell,point_id)
		else:
			for lab in _id_labs:
				lab.queue_free()
			_id_labs.clear()


# 基于TileMap，添加可通行点和连接线，创建Astar网格,并返回一个AStar2D实例
func _init(tile_map:TileMap,layer_id:int):
	# 存储TileMap对象和layer_id
	_tile_map = tile_map
	_layer_id = layer_id
	
	# 添加可通行点
	for cell in get_has_navigation_cells():
		var id = get_available_point_id()
		add_point(id,cell)
	
	# 遍历已经添加了的可通行点,进行连线
	var points_ids = get_point_ids()
	for point_id in points_ids:
		var point_cell_pos = get_point_position(point_id) # id转为单元格坐标
		var surround_cell_pos_arr = tile_map.get_surrounding_cells(point_cell_pos)# 获取周边6个位置
		for cel in surround_cell_pos_arr:
			var cel_id = get_closest_point(cel)# 尝试获取最接近的点的id
			if Vector2i(get_point_position(cel_id)) in surround_cell_pos_arr: # 验证获取的点的ID在周围6个位置内
				if cel_id in points_ids: # 验证点在已经添加的位置内
					if !are_points_connected(cel_id,point_id): # 验证两个点之间不存在连接
						connect_points(cel_id,point_id)

# 判断单元格是否存在导航多边形
func is_cell_has_navigation_polygon(cell:Vector2i):
	var bol = false
	var data = _tile_map.get_cell_tile_data(_layer_id,cell)
	if data:
		if data.get_navigation_polygon(_layer_id): # 有导航区域
			bol = true
	return bol

# 获取TileMap所有拥有导航网格的单元格
# 查找是在get_used_cells基础上进行的
func get_has_navigation_cells() -> Array[Vector2i]:
	var cells:Array[Vector2i] = []
	# 获取所有已经绘制的单元格
	var used_cells:Array[Vector2i] = _tile_map.get_used_cells(_layer_id)
	# 遍历所有网格，将带有导航区域的单元格位置添加为AStar的可通行点
	for cell in used_cells:
		if is_cell_has_navigation_polygon(cell): # 格子有导航多边形
			cells.append(cell)
	return cells


# =================================== 底层辅助函数 ===================================

func create_lab(content:String,font_color:Color = Color("#ffffff")):
	var lab = Label.new()
	lab.horizontal_alignment = HORIZONTAL_ALIGNMENT_CENTER
	lab.vertical_alignment = VERTICAL_ALIGNMENT_CENTER
	# 构造LabelSettings
	var lab_setting = LabelSettings.new()
	lab_setting.font_color = font_color
	lab_setting.font_size = _tile_map.tile_set.tile_size.x /8
	lab_setting.outline_color = Color("#444444")
	lab_setting.outline_size = lab_setting.font_size/3
	lab_setting.shadow_color = Color("#3333339e")
	lab_setting.shadow_size = 1
	lab_setting.shadow_offset = Vector2(2,2)
	# 其他配置
	lab.label_settings = lab_setting
	lab.text = content
	return lab

# 显示TileMap对应单元格的位置信息
func show_cell_pos(cell:Vector2i):
	var lab = create_lab(str(cell))
	lab.position = _tile_map.map_to_local(cell)
	_tile_map.add_child(lab)
	_pos_labs.append(lab)

# 显示单元格对应的AStar2D的点ID
func show_cell_id(cell:Vector2i,id:int):
	var lab = create_lab(str(id),Color.ORANGE)
	lab.position = _tile_map.map_to_local(cell) - Vector2(_tile_map.tile_set.tile_size /5.5)
	_tile_map.add_child(lab)
	_id_labs.append(lab)

```
## 使用方法
在一个使用TileMap的场景中，比如这里直接有一个以TileMap为根节点的场景。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1687007865990-ffeef9ab-72da-4a6f-b4bc-a29b1d44d5eb.png#averageHue=%232d3540&clientId=u7638af01-0caf-4&from=paste&height=73&id=u7a99386d&originHeight=218&originWidth=507&originalType=binary&ratio=3&rotation=0&showTitle=false&size=13775&status=done&style=none&taskId=ub46a4b50-f608-421b-87da-9e552c1c609&title=&width=169)
创建`TileSet`，并为可以通行的图块绘制导航多边形（默认形状即可）。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1687007933834-36f7d40f-1e97-450e-ad51-483366436c0e.png#averageHue=%23373f49&clientId=u7638af01-0caf-4&from=paste&height=391&id=u08fee0e7&originHeight=1174&originWidth=2830&originalType=binary&ratio=3&rotation=0&showTitle=false&size=400617&status=done&style=none&taskId=ud0229434-0f40-45c6-9a52-f6100465602&title=&width=943.3333333333334)
然后用可通行和不可通行的图块在TileMap上任意绘制一块地图区域。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1687007913380-65e2f29c-2cf8-4d96-a7bf-b6cf5f7ebd04.png#averageHue=%23534e4c&clientId=u7638af01-0caf-4&from=paste&height=309&id=u56e224fb&originHeight=928&originWidth=1391&originalType=binary&ratio=3&rotation=0&showTitle=false&size=181855&status=done&style=none&taskId=u2c529afd-a5e2-45df-bd3a-a4548527b80&title=&width=463.6666666666667)
为TileMap添加如下脚本：
```swift
extends TileMap

var astar = TileMapAStar2D.new(self,0)

func _ready():
	astar.show_navigation_cells_pos = true # 显示单元格的坐标
	astar.show_navigation_cells_id = true  # 显示TileMapAStar2D为单元个创建的位置ID
```
运行后就可以看到如下效果：
![显示所有可通行位置的AStar2D的ID和对应TileMap的单元格坐标](https://cdn.nlark.com/yuque/0/2023/png/8438332/1687008036532-ef5892ce-cc71-4bf6-beac-d0a6b7760c4a.png#averageHue=%2356504d&clientId=u7638af01-0caf-4&from=paste&height=369&id=u4f45ea9b&originHeight=1228&originWidth=1703&originalType=binary&ratio=3&rotation=0&showTitle=true&size=269727&status=done&style=none&taskId=u9ba864e0-1d98-42af-b079-866f06b460d&title=%E6%98%BE%E7%A4%BA%E6%89%80%E6%9C%89%E5%8F%AF%E9%80%9A%E8%A1%8C%E4%BD%8D%E7%BD%AE%E7%9A%84AStar2D%E7%9A%84ID%E5%92%8C%E5%AF%B9%E5%BA%94TileMap%E7%9A%84%E5%8D%95%E5%85%83%E6%A0%BC%E5%9D%90%E6%A0%87&width=511.66668701171875 "显示所有可通行位置的AStar2D的ID和对应TileMap的单元格坐标")
### 显示导航网格
为TileMap添加一个Control类型的节点，起名叫debug。用来实现TileMapAStar2D自动生成的导航网格。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1687008743651-4b3ab678-be0c-4e5a-a629-62a1462e5e0e.png#averageHue=%232c323b&clientId=u7638af01-0caf-4&from=paste&height=82&id=u86f49375&originHeight=246&originWidth=435&originalType=binary&ratio=3&rotation=0&showTitle=false&size=18156&status=done&style=none&taskId=ubfbbde24-6743-444b-97ac-6b1e7db05a2&title=&width=145)
```swift
extends Control

var astar:AStar2D
@export var path_color:Color = Color.YELLOW_GREEN
@export var point_color:Color = Color.YELLOW_GREEN


func _ready():
	astar = get_parent().astar

func _draw():
	if astar:
		var map:TileMap = get_parent()
		var points_ids = astar.get_point_ids()
		# 先画线
		for point_id in points_ids:
			var cel_pos = astar.get_point_position(point_id) # 转为单元格位置坐标
			var cel_screen_pos = map.map_to_local(cel_pos)   # 转为屏幕坐标
			# 获取连接信息
			var connects = astar.get_point_connections(point_id)
			for cnt_point_id in connects:
				var cnt_pos = astar.get_point_position(cnt_point_id) # 转为单元格位置坐标
				var cnt_screen_pos = map.map_to_local(cnt_pos)   # 转为屏幕坐标
				draw_line(cel_screen_pos,cnt_screen_pos,path_color,1)
		# 后画点
		for point_id in points_ids:
			var cel_pos = astar.get_point_position(point_id) # 转为单元格位置坐标
			draw_circle(map.map_to_local(cel_pos),5,point_color)
```
将TileMap的脚本改写为：
```swift
extends TileMap

var astar = TileMapAStar2D.new(self,0)

@onready var debug = $debug

func _ready():
	debug.astar = astar
	astar.show_navigation_cells_pos = true
	astar.show_navigation_cells_id = true
	debug.queue_redraw()
```
运行后就可以看到绘制出的A*导航网格。
![利用debug层显示AStar网格](https://cdn.nlark.com/yuque/0/2023/png/8438332/1687008998680-8a253a21-84e9-41ee-876b-58a6cea1f073.png#averageHue=%23bcb056&clientId=u7638af01-0caf-4&from=paste&height=389&id=u2b702116&originHeight=1167&originWidth=1027&originalType=binary&ratio=3&rotation=0&showTitle=true&size=260663&status=done&style=none&taskId=u08aebbd1-e57f-4af1-959f-51c7c777676&title=%E5%88%A9%E7%94%A8debug%E5%B1%82%E6%98%BE%E7%A4%BAAStar%E7%BD%91%E6%A0%BC&width=342.3333333333333 "利用debug层显示AStar网格")
有了 `TileMapAStar2D`为`TileMap`自动化的生成导航网格，以及利用其`show_navigation_cells_pos`和`show_navigation_cells_id`属性，还有用`debug`层绘制导航网格，则基于TileMap进行A*导航就有了非常好的视觉基础。无论是学习还是使用都更直观了。
当然对于一般四边形TileSet应该也是适用的，只不过`Godot4`已经提供了`AStarGrid2D`专用于四边形`TileSet`，但是它只适用于四边形网格，而`TileMapAStar2D`则更具有宽泛的适用性。
![TileMapAStar2D用于方形TileSet的效果](https://cdn.nlark.com/yuque/0/2023/png/8438332/1687009770092-1f9b8493-498a-4b88-9702-36c80ec14251.png#averageHue=%23c97e48&clientId=u7638af01-0caf-4&from=paste&height=416&id=u297277f4&originHeight=1247&originWidth=2248&originalType=binary&ratio=3&rotation=0&showTitle=true&size=297132&status=done&style=none&taskId=u883318af-1531-41cd-aad0-c7a4d5e4a1d&title=TileMapAStar2D%E7%94%A8%E4%BA%8E%E6%96%B9%E5%BD%A2TileSet%E7%9A%84%E6%95%88%E6%9E%9C&width=749.3333333333334 "TileMapAStar2D用于方形TileSet的效果")
![在Half-Offset下的验证](https://cdn.nlark.com/yuque/0/2023/png/8438332/1687010513159-f52a0be1-84b5-4f7a-a55b-44b36aec3579.png#averageHue=%23c97f48&clientId=u7638af01-0caf-4&from=paste&height=417&id=uc7e3793a&originHeight=1252&originWidth=2358&originalType=binary&ratio=3&rotation=0&showTitle=true&size=302955&status=done&style=none&taskId=u5b3e13fb-a650-42ae-a731-a610a7a3b98&title=%E5%9C%A8Half-Offset%E4%B8%8B%E7%9A%84%E9%AA%8C%E8%AF%81&width=786 "在Half-Offset下的验证")
![在isometric图块下的验证](https://cdn.nlark.com/yuque/0/2023/png/8438332/1687012384818-901939dc-6f1b-41bf-90d8-d6bb9ae887f6.png#averageHue=%23b8976b&clientId=u7638af01-0caf-4&from=paste&height=506&id=ue3274716&originHeight=1517&originWidth=2834&originalType=binary&ratio=3&rotation=0&showTitle=true&size=1112931&status=done&style=none&taskId=u4d7a79e5-557a-4cc4-9ad9-be36f03d448&title=%E5%9C%A8isometric%E5%9B%BE%E5%9D%97%E4%B8%8B%E7%9A%84%E9%AA%8C%E8%AF%81&width=944.6666666666666 "在isometric图块下的验证")
## 孤岛验证
验证算法在任何孤立的导航区域都能正确生成A*导航网格。
![六边形图块的孤岛验证](https://cdn.nlark.com/yuque/0/2023/png/8438332/1687013463689-4ec63d40-0315-442f-a382-b9da2d0a5025.png#averageHue=%23b87148&clientId=u7638af01-0caf-4&from=paste&height=651&id=u7d683da5&originHeight=1953&originWidth=2516&originalType=binary&ratio=3&rotation=0&showTitle=true&size=621576&status=done&style=none&taskId=u2e2e19dc-310e-4a6f-9906-4009705b1d1&title=%E5%85%AD%E8%BE%B9%E5%BD%A2%E5%9B%BE%E5%9D%97%E7%9A%84%E5%AD%A4%E5%B2%9B%E9%AA%8C%E8%AF%81&width=838.6666666666666 "六边形图块的孤岛验证")
