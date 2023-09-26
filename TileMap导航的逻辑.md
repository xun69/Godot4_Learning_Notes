## 基于NavigationRegion2D
我们基于[NavigationRegion2D的逻辑](https://www.yuque.com/xunwukong/frdo3b/um8gbtdhap0pgtfz?view=doc_embed)一文的场景结构，但是将`NavigationRegion2D`删除，更改为TileMap节点。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686490067627-deaf8771-9fad-423b-9a33-110170457c3a.png#averageHue=%232c333e&clientId=u68f59145-4fe8-4&from=paste&height=116&id=u28ff984c&originHeight=347&originWidth=545&originalType=binary&ratio=3&rotation=0&showTitle=false&size=24221&status=done&style=none&taskId=u2874f877-95aa-43cf-860e-901e37d07d6&title=&width=181.66666666666666)
为TileMap创建Tileset，并创建一个导航层。在TileSet面板中，为草地和黄色泥土地面图块绘制可通行区域（整个矩形）。这样做的含义是只有这连个图块允许通行，其余图块全是障碍物。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686490033517-98198e85-65bf-442f-8f76-c84838529980.png#averageHue=%232f3743&clientId=u68f59145-4fe8-4&from=paste&height=351&id=u3ae2b272&originHeight=1272&originWidth=626&originalType=binary&ratio=3&rotation=0&showTitle=false&size=101976&status=done&style=none&taskId=u6084b7f2-ff36-4030-a2b2-7bc17548a65&title=&width=172.6666717529297)![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686490054302-2df6012b-62dd-4505-9e45-755916d78d89.png#averageHue=%23363d49&clientId=u68f59145-4fe8-4&from=paste&height=373&id=uf2ebf391&originHeight=1351&originWidth=1950&originalType=binary&ratio=3&rotation=0&showTitle=false&size=125594&status=done&style=none&taskId=u0f15a379-068a-4a7a-8069-ce2cacc0929&title=&width=539)
切换至TileMap面板，利用矩形和直线绘制方式绘制一个简单的有出口的地图。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686490298827-1b3c60e2-bbc7-412e-9551-d111669a2a83.png#averageHue=%2383c569&clientId=u68f59145-4fe8-4&from=paste&height=637&id=u74947c60&originHeight=1912&originWidth=2082&originalType=binary&ratio=3&rotation=0&showTitle=false&size=235211&status=done&style=none&taskId=uf8525554-30e0-495a-a380-ca03e0b38a8&title=&width=694)
运行场景，会发现基于`NavigationAgent2D`的导航在TileMap上依旧是可行的。
[![TileMap导航.mp4 (25.9MB)](https://gw.alipayobjects.com/mdn/prod_resou/afts/img/A*NNs6TKOR3isAAAAAAAAAAABkARQnAQ)]()## 基于AStarGrid2D
可以看出基于`NavigationRegion2D`的导航总还是有些不尽如人意的地方，偶尔就卡在了半路上。
`AStarGrid2D`本身基于网格，那么它与`TileMap`组合可以说是“天作之合”。
删除Player的`NavigationRegion2D`以及脚本。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686494523744-50c859a5-8f16-474e-82b4-38a94a00f33d.png#averageHue=%232d353f&clientId=u8c252fda-317e-4&from=paste&height=111&id=u562c8b42&originHeight=332&originWidth=436&originalType=binary&ratio=3&rotation=0&showTitle=false&size=23500&status=done&style=none&taskId=u6a2bf49f-3385-4e16-b873-b0ea9176452&title=&width=145.33333333333334)![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686494200039-73f3380e-d761-437f-a22f-4b38f20443f5.png#averageHue=%23384658&clientId=u8c252fda-317e-4&from=paste&height=331&id=uc15a34e3&originHeight=994&originWidth=597&originalType=binary&ratio=3&rotation=0&showTitle=false&size=72806&status=done&style=none&taskId=u4d425605-448b-4f02-8a0c-72094c67d8a&title=&width=199)
查看icon.svg可以发现其图片大小为128×128，而我们的tileset的cell_size为16×16，所以为了将玩家显示在一个网格里，我们需要将icon.svg缩放为16×16。
128/16=8，所以我们需要将其设为原来的1/8，也就是缩小到原来的0.125倍。相应调整碰撞形状的大小。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686494723207-d0e30419-9548-4395-aab7-7853ab1c857c.png#averageHue=%232d343e&clientId=u8c252fda-317e-4&from=paste&height=348&id=u590b2306&originHeight=1043&originWidth=594&originalType=binary&ratio=3&rotation=0&showTitle=false&size=72716&status=done&style=none&taskId=u45af352b-24fb-4095-a1ef-b7f53e2300d&title=&width=198)![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686494731866-65fe5eeb-3941-437a-9e67-6e12dac4c0ae.png#averageHue=%2370aa50&clientId=u8c252fda-317e-4&from=paste&height=167&id=ue40240e0&originHeight=500&originWidth=491&originalType=binary&ratio=3&rotation=0&showTitle=false&size=40909&status=done&style=none&taskId=u181af4d9-420c-432a-820f-38d0eb1976e&title=&width=163.66666666666666)
移动玩家到某个单元格内。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686494270302-230d0418-fb25-4fa1-9fec-eacc59275078.png#averageHue=%2384c669&clientId=u8c252fda-317e-4&from=paste&height=177&id=ua921a408&originHeight=531&originWidth=731&originalType=binary&ratio=3&rotation=0&showTitle=false&size=64004&status=done&style=none&taskId=u88622cd3-4c4e-4ab0-a0ed-009c7ec380e&title=&width=243.66666666666666)
将world2的脚本改成如下：
```swift
# world2.gd
extends Node2D

@onready var player = $Player
@onready var tile_map = $TileMap
var astar_grid = AStarGrid2D.new()
var move_speed = 200.0
var path:PackedVector2Array
var solids:PackedVector2Array

func _ready():
	# 获取tile_map包含所有已绘制图块的矩形
	var rect = tile_map.get_used_rect()
	# 构建AStarGrid2D
	astar_grid.size = rect.size
	astar_grid.cell_size = tile_map.tile_set.tile_size
	astar_grid.offset = astar_grid.cell_size/2
	astar_grid.update()
	# 获取所有已经绘制的图块的数组
	var cells = tile_map.get_used_cells(0)
	for cell in cells:
    	# TileData包含图块的信息
		var data:TileData = tile_map.get_cell_tile_data(0,cell)
		if !data.get_navigation_polygon(0): # 如果图块没有设置导航
			solids.append(cell)
			astar_grid.set_point_solid(cell,true) # 设为障碍物

# 左键点击，获取导航目标位置
func _input(event):
	if event is InputEventMouseButton:
		if event.button_index == MOUSE_BUTTON_LEFT and event.is_pressed():
			# TileMap的local_to_map()用于将屏幕坐标转化为TileMap的网格坐标
			var p1 = tile_map.local_to_map(player.position)
			var p2 = tile_map.local_to_map(get_global_mouse_position())
			path = astar_grid.get_point_path(p1,p2)
			queue_redraw()

# 实现沿着路径行走
func _process(delta):
	if path.size() > 0:
		var target_pos = path[0]
		if target_pos.distance_to(player.position)>2:
			player.velocity = player.position.direction_to(target_pos) * move_speed
			player.move_and_slide()
		else:
			path.remove_at(0)
		queue_redraw()

# 绘制路径信息
func _draw():
	var grid_width = astar_grid.size.x * astar_grid.cell_size.x
	var cell_width = astar_grid.cell_size.x
	var cell_height = astar_grid.cell_size.y
	
	# 绘制路径和其上的点
	if path.size() > 0:
		for pot in path:
			draw_circle(pot,5,Color.YELLOW)
				
		draw_polyline(path,Color.YELLOW,2)

```
开启显示碰撞区域和导航。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686495181594-92f46767-aa8f-411c-8d77-d6e10240e15a.png#averageHue=%23353c48&clientId=u8c252fda-317e-4&from=paste&height=245&id=ude37c14d&originHeight=735&originWidth=756&originalType=binary&ratio=3&rotation=0&showTitle=false&size=100787&status=done&style=none&taskId=u1790851f-8c2f-4760-bb1c-0666076dee7&title=&width=252)
运行后的效果如下：
[![2023-06-11_22-40-24.mp4 (5.06MB)](https://gw.alipayobjects.com/mdn/prod_resou/afts/img/A*NNs6TKOR3isAAAAAAAAAAABkARQnAQ)]()基本的导航还是可以的。
:::success
**提示**
默认由于TileMap是world2的子节点，所以world2上用_draw绘制的内容会被TileMap绘制的地图覆盖而无法显示。解决方法是将TileMap的z_index设为-1。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686495297931-3271b0c8-5127-4580-baf0-343dce4d06d1.png#averageHue=%232b313a&clientId=u8c252fda-317e-4&from=paste&height=107&id=u9d41fd72&originHeight=321&originWidth=448&originalType=binary&ratio=3&rotation=0&showTitle=false&size=22968&status=done&style=none&taskId=udc794694-5615-4f96-8758-95ce068fded&title=&width=149.33333333333334)![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686495392510-76cef762-aa48-463a-a5ed-8c23e636dba3.png#averageHue=%232f343c&clientId=u8c252fda-317e-4&from=paste&height=434&id=uf96ef272&originHeight=1301&originWidth=616&originalType=binary&ratio=3&rotation=0&showTitle=false&size=109342&status=done&style=none&taskId=u46530754-e679-485e-b089-b71e5c2e372&title=&width=205.33333333333334)
:::
