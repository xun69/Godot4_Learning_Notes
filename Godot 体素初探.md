[godot 体素教程 1(概念，以及介绍godot voxel)_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1h3411271X/?spm_id_from=333.337.search-card.all.click&vd_source=2f49ad1693bc7945c7bf165c74c86263)
## 体素的性能问题
如果直接使用立方体网格堆叠的方式，则会存在很多不显示但是真实存在的面，立方体一多，就会加重计算的负担。
好在Godot提供了一个叫`SurfaceTool`的类，可以用十分底层（定义法线、UV和添加顶点）的方式创建3D网格资源，这就让程序化生成3D网格有了可能，并有了动态融并网格的算法可能。
## SurfaceTool的应用
### 搭建SurfaceTool简单测试场景
通过创建一个含有`MeshInstance3D`的简单3D场景+一个简单的`EditorScript`脚本，就可以搭建一个基础的`SurfaceTool`测试场景。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686909890989-1dcf180d-1386-42ae-8f54-a6388c7326e1.png#averageHue=%232b323d&clientId=u489c36a8-8b59-4&from=paste&height=98&id=u12a04dc5&originHeight=293&originWidth=544&originalType=binary&ratio=3&rotation=0&showTitle=false&size=21040&status=done&style=none&taskId=u91e330b9-0c66-4825-b886-93c480f74b0&title=&width=181.33333333333334)
框架代码如下：
```swift
@tool
extends EditorScript


func _run():
	var ins:MeshInstance3D = get_scene().get_node("MeshInstance3D")
	ins.mesh = create_mesh()
	pass

func create_mesh():
	var st = SurfaceTool.new()
	# ...
	return mesh
```
其中自定义函数create_mesh()用于SurfaceTool实例的创建和3D网格的返回。
`_run()`则在运行后，对场景中的`MeshInstance3D`节点的`mesh`属性赋值。
这样做的好处是，可以直接在编辑器中看到网格的样子。
比如下面的代码：
```swift
@tool
extends EditorScript


func _run():
	var ins:MeshInstance3D = get_scene().get_node("MeshInstance3D")
	ins.mesh = create_mesh()
	pass

func create_mesh():
	var st = SurfaceTool.new()

	st.begin(Mesh.PRIMITIVE_TRIANGLES)

	# Prepare attributes for set_vertex.
	st.set_normal(Vector3(0, 0, 1))
	st.set_uv(Vector2(0, 0))
	# Call last for each vertex, adds the above attributes.
	st.add_vertex(Vector3(-1, -1, 0))

	st.set_normal(Vector3(0, 0, 1))
	st.set_uv(Vector2(0, 1))
	st.add_vertex(Vector3(-1, 1, 0))

	st.set_normal(Vector3(0, 0, 1))
	st.set_uv(Vector2(1, 1))
	st.add_vertex(Vector3(1, 1, 0))

	# Commit to a mesh.
	var mesh = st.commit()
	return mesh

```
运行后，生成了如下的3D网格：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686910132814-9962d1dc-71d2-4078-afa8-7f7290a1f856.png#averageHue=%233d3727&clientId=u489c36a8-8b59-4&from=paste&height=235&id=uef7e2011&originHeight=882&originWidth=1129&originalType=binary&ratio=3&rotation=0&showTitle=false&size=461444&status=done&style=none&taskId=ue543c850-c79a-41be-adbf-5cd9ed40983&title=&width=300.3333435058594)
在后视图可以看到其实际的形状是一个等腰直角三角形的三角面。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686911960391-e4f05f3c-281b-43af-ba0b-640cc7708e24.png#averageHue=%237c7f7c&clientId=u489c36a8-8b59-4&from=paste&height=464&id=u8f0e1926&originHeight=1393&originWidth=2645&originalType=binary&ratio=3&rotation=0&showTitle=false&size=2700764&status=done&style=none&taskId=ucdccb26d-085c-4171-b007-b3b494295ea&title=&width=881.6666666666666)
法线都是（0,0,1）。而且UV坐标在垂直方向是反的。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686910825334-d71ef5a5-eb40-4be0-981e-49b237782b14.png#averageHue=%23959797&clientId=u489c36a8-8b59-4&from=paste&height=290&id=u412c35b6&originHeight=1054&originWidth=1131&originalType=binary&ratio=3&rotation=0&showTitle=false&size=492573&status=done&style=none&taskId=u7d0adfd8-f9c8-45fa-b13e-a4d6152f39e&title=&width=311)
## 生成方形网格
会生成三角面了，那么生成方形网格也就不难了，方形网格可以看做是两个三角面组成。
执行如下脚本：
```swift
@tool
extends EditorScript


func _run():
	var ins:MeshInstance3D = get_scene().get_node("MeshInstance3D")
	ins.mesh = create_mesh()
	pass

func create_mesh():
	var st = SurfaceTool.new()

	st.begin(Mesh.PRIMITIVE_TRIANGLES)
	
	st.set_normal(Vector3(0, 0, 1))
	st.set_uv(Vector2(0, 0))
	st.add_vertex(Vector3(-1, -1, 0))

	st.set_normal(Vector3(0, 0, 1))
	st.set_uv(Vector2(0, 1))
	st.add_vertex(Vector3(-1, 1, 0))

	st.set_normal(Vector3(0, 0, 1))
	st.set_uv(Vector2(1, 1))
	st.add_vertex(Vector3(1, 1, 0))
	
	
	st.set_normal(Vector3(0, 0, 1))
	st.set_uv(Vector2(1, 1))
	st.add_vertex(Vector3(1, 1, 0))
	
	st.set_normal(Vector3(0, 0, 1))
	st.set_uv(Vector2(1, 0))
	st.add_vertex(Vector3(1, -1, 0))
	
	st.set_normal(Vector3(0, 0, 1))
	st.set_uv(Vector2(0, 0))
	st.add_vertex(Vector3(-1, -1, 0))
	

	# Commit to a mesh.
	var mesh = st.commit()
	return mesh
```
生成的方形网格如下：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686911719665-a040b29f-65aa-4bed-984a-7d8dee0d9e4a.png#averageHue=%23949695&clientId=u489c36a8-8b59-4&from=paste&height=270&id=U0Mku&originHeight=1064&originWidth=994&originalType=binary&ratio=3&rotation=0&showTitle=false&size=450125&status=done&style=none&taskId=ub1ef8c78-52a5-46aa-b2f5-70387b13cd9&title=&width=252.33334350585938)
其顶点和UV以及三角面的示意图如下：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686913583243-da8d1e86-282b-42d7-906e-2addc943059c.png#averageHue=%23888e8f&clientId=u489c36a8-8b59-4&from=paste&height=249&id=u6b6a207a&originHeight=856&originWidth=1210&originalType=binary&ratio=3&rotation=0&showTitle=false&size=555457&status=done&style=none&taskId=u175896dd-f581-4742-ab39-5f1113446a8&title=&width=351.3333435058594)
注意：

- 添加顶点的顺序非常重要，如果不按首尾顺序添加，则法线可能相反。
- 每3个点组成一个三角面，如果顶点不够，就会报错。
## 创建立方体
在Godot中，可以使用一个`AABB`来表示一个基本的立方体。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686916039611-ef2c3767-3a4c-4d6e-80d4-37a591471727.png#averageHue=%23fcfbfb&clientId=u489c36a8-8b59-4&from=paste&height=282&id=u572674e3&originHeight=916&originWidth=1080&originalType=binary&ratio=3&rotation=0&showTitle=false&size=54514&status=done&style=none&taskId=ua3a95ee5-3cd4-4f3f-b4a9-4a3963f8603&title=&width=333)
通过计算可以获得`AABB`代表的立方体的所有顶点坐标。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686916930420-38d86ab8-6a96-47f8-88fc-8b79b9a900a3.png#averageHue=%23fbfbfb&clientId=u489c36a8-8b59-4&from=paste&height=297&id=pPiJq&originHeight=1280&originWidth=1546&originalType=binary&ratio=3&rotation=0&showTitle=false&size=106009&status=done&style=none&taskId=u2c8a324c-2934-47ae-80ba-d53ac2d6243&title=&width=358.3333740234375)
```swift
# 三条坐标轴方向的边的端点
var p_Z = (0,0,end.z) # Z轴
var p_Y = (0,end.y,0) # Y轴
var p_X = (end.x,0,0) # X轴
# 三个平面上的点的端点
var p_YZ = (0,end.y,end.z) # YZ平面
var p_XZ = (end.x,0,end.z) # XZ平面
var p_XY = (end.x,end.y,0) # XY平面
```
上面的点坐标是基于一个`position=Vector3.ZERO`，然后`size = Vector3.ONE`的`AABB`进行计算的。
其他位置的立方体可以通过`AABB`的变换得到。
如果想要获得以坐标原点为中心的矩形，可以设置`position=-Vector3.ONE/2`，然后`size = Vector3.ONE`的`AABB`。并将上述所有的0变成postion相应轴上的分量。也就是：
```swift
# 三条坐标轴方向的边的端点
var p_Z = (position.x,position.y,end.z) # Z轴
var p_Y = (position.x,end.y,position.z) # Y轴
var p_X = (end.x,position.y,position.z) # X轴
# 三个平面上的点的端点
var p_YZ = (position.x,end.y,end.z) # YZ平面
var p_XZ = (end.x,position.y,end.z) # XZ平面
var p_XY = (end.x,end.y,position.z) # XY平面
```
### 定义6个面的法线和顶点顺序
为六个面定义名称：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686920048925-90086742-4c0c-4f93-ab45-5d3d520d8f65.png#averageHue=%23fafafa&clientId=u489c36a8-8b59-4&from=paste&height=310&id=ua849301d&originHeight=1264&originWidth=1501&originalType=binary&ratio=3&rotation=0&showTitle=false&size=116266&status=done&style=none&taskId=ua262e35f-49a2-4135-9263-4463c41e74a&title=&width=368.3333435058594)
```swift
# UV坐标
var uvs = [Vector2.DOWN,Vector2.ZERO,Vector2.RIGHT,Vector2.ONE]
# 右侧面
var f_right = {
    normal = Vector3.RIGHT,
    vectors = [p_XZ,p_E,p_XY,p_XY,p_X,p_XZ]
}
```
实现后的完整代码如下：
```swift
@tool
extends EditorScript


func _run():
	var ins:MeshInstance3D = get_scene().get_node("MeshInstance3D")
	ins.mesh = create_cubemesh()

func create_cubemesh():
	var st = SurfaceTool.new()
	st.begin(Mesh.PRIMITIVE_TRIANGLES)
	# 构造AABB
	var position = Vector3.ZERO
	var size = Vector3.ONE
	var box:AABB = AABB(position,size)
	var end = box.end
	# ============= 获得立方体8个顶点的坐标
	# positon和and
	var p_P = position
	var p_E = end
	# 三条坐标轴方向的边的端点
	var p_Z = Vector3(position.x,position.y,end.z) # Z轴
	var p_Y = Vector3(position.x,end.y,position.z) # Y轴
	var p_X = Vector3(end.x,position.y,position.z) # X轴
	# 三个平面上的点的端点
	var p_YZ = Vector3(position.x,end.y,end.z) # YZ平面
	var p_XZ = Vector3(end.x,position.y,end.z) # XZ平面
	var p_XY = Vector3(end.x,end.y,position.z) # XY平面
	
	# 6个顶点的UV坐标顺序
	var uvs = [Vector2.DOWN,Vector2.ZERO,Vector2.RIGHT,Vector2.ONE]
	# ============= 定义立方体6个面的法线和顶点绘制顺序
	# 右侧面
	var faces = {
		f_right = { # 右侧面
			normal = Vector3.RIGHT,
			vectors = [p_XZ,p_E,p_XY,p_X]
		},
		f_up = { # 上面
			normal = Vector3.UP,
			vectors = [p_E,p_YZ,p_Y,p_XY]
		},
		f_left = { # 左侧面
			normal = Vector3.LEFT,
			vectors = [p_P,p_Y,p_YZ,p_Z]
		},
		f_bottom = { # 底面
			normal = Vector3.DOWN,
			vectors = [p_Z,p_XZ,p_X,p_P]
		},
		f_front = { # 前面
			normal = Vector3.FORWARD,
			vectors = [p_X,p_XY,p_Y,p_P]
		},
		f_bcak = { # 后面
			normal = Vector3.BACK,
			vectors = [p_Z,p_YZ,p_E,p_XZ]
		}
	}
	
	for face in faces:
		for i in [0,1,2,2,3,0]:
			st.set_normal(faces[face]["normal"])
			st.set_uv(uvs[i])
			st.add_vertex(faces[face]["vectors"][i])
	
	var mesh = st.commit()
	return mesh

```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686920135863-a02afca8-fdbf-47a4-b173-351dabad1e07.png#averageHue=%233c3625&clientId=u489c36a8-8b59-4&from=paste&height=250&id=ub91b5a52&originHeight=750&originWidth=798&originalType=binary&ratio=3&rotation=0&showTitle=false&size=363554&status=done&style=none&taskId=uca2641e5-558a-43ac-8bc1-5b526c45e77&title=&width=266)
当实现一个立方体网格的绘制后，就可以基于此创建融并的网格。
## 创建融并网格和区域
```swift
@tool
extends EditorScript


func _run():
	var ins:MeshInstance3D = get_scene().get_node("MeshInstance3D")
	ins.mesh = create_area([Vector3.ZERO,Vector3.ONE])

# 在area_pos_arr传入的所有3维空间位置创建一个16×16的立方体网格区域
func create_area(area_pos_arr:PackedVector3Array):
	var pos_arr:PackedVector3Array
	for area_pos in area_pos_arr:
		# 以16×16为一个小区域
		var area_start_pos = area_pos * 16
		var area_size = Vector3.ONE * 16
		var box:AABB = AABB(area_start_pos,area_size)
		# 随机生成box位置
		for x in range(area_start_pos.x,box.end.x):
			for y in range(area_start_pos.y,box.end.y):
				for z in range(area_start_pos.z,box.end.z):
					var is_empty = randi_range(0,1)
					if not is_empty:
						pos_arr.append(Vector3(x,y,z))
	return create_cubemesh(pos_arr)

# 在pos_arr传入的所有3维空间位置创建一个立方体网格
func create_cubemesh(pos_arr:PackedVector3Array):
	var st = SurfaceTool.new()
	st.begin(Mesh.PRIMITIVE_TRIANGLES)
	for pos in pos_arr:
		# 构造AABB
		var size = Vector3.ONE
		var box:AABB = AABB(pos,size)
		var end = box.end
		# ============= 获得立方体8个顶点的坐标
		# positon和and
		var p_P = pos
		var p_E = end
		# 三条坐标轴方向的边的端点
		var p_Z = Vector3(pos.x,pos.y,end.z) # Z轴
		var p_Y = Vector3(pos.x,end.y,pos.z) # Y轴
		var p_X = Vector3(end.x,pos.y,pos.z) # X轴
		# 三个平面上的点的端点
		var p_YZ = Vector3(pos.x,end.y,end.z) # YZ平面
		var p_XZ = Vector3(end.x,pos.y,end.z) # XZ平面
		var p_XY = Vector3(end.x,end.y,pos.z) # XY平面
		
		# 6个顶点的UV坐标顺序
		var uvs = [Vector2.DOWN,Vector2.ZERO,Vector2.RIGHT,Vector2.ONE]
		# ============= 定义立方体6个面的法线和顶点绘制顺序
		# 右侧面
		var faces = {
			f_right = { # 右侧面
				normal = Vector3.RIGHT,
				vectors = [p_XZ,p_E,p_XY,p_X]
			},
			f_up = { # 上面
				normal = Vector3.UP,
				vectors = [p_E,p_YZ,p_Y,p_XY]
			},
			f_left = { # 左侧面
				normal = Vector3.LEFT,
				vectors = [p_P,p_Y,p_YZ,p_Z]
			},
			f_bottom = { # 底面
				normal = Vector3.DOWN,
				vectors = [p_Z,p_XZ,p_X,p_P]
			},
			f_front = { # 前面
				normal = Vector3.FORWARD,
				vectors = [p_X,p_XY,p_Y,p_P]
			},
			f_bcak = { # 后面
				normal = Vector3.BACK,
				vectors = [p_Z,p_YZ,p_E,p_XZ]
			}
		}
		# 检测上下左右前后方向有无立方体
		# 有的话就删除相应的面
		for face in faces:
			if pos + faces[face]["normal"] not in pos_arr:
				for i in [0,1,2,2,3,0]:
					st.set_normal(faces[face]["normal"])
					st.set_uv(uvs[i])
					st.add_vertex(faces[face]["vectors"][i])
	
	var mesh = st.commit()
	return mesh

```
运行后，将基于一个`MeshInstance3D`节点生成如下复杂的网格：
但是后期最好的做法是一个`MeshInstance3D`节点只生成一个16×16小区域的网格。
![生成的两个区域的网格](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686922797178-6972ff4d-8558-4713-8d62-07add1dde5b1.png#averageHue=%233c3825&clientId=u489c36a8-8b59-4&from=paste&height=331&id=u87b57e5b&originHeight=1209&originWidth=1636&originalType=binary&ratio=3&rotation=0&showTitle=true&size=1729398&status=done&style=none&taskId=u7d402033-02ec-4953-8e74-9e9df0e2a5b&title=%E7%94%9F%E6%88%90%E7%9A%84%E4%B8%A4%E4%B8%AA%E5%8C%BA%E5%9F%9F%E7%9A%84%E7%BD%91%E6%A0%BC&width=448.3333435058594 "生成的两个区域的网格")
![未采用网格融并算法](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686924661151-bb602c7b-59b1-47e8-8911-f67a1b074dad.png#averageHue=%23354142&clientId=u67177426-be85-4&from=paste&height=403&id=ude7c89b9&originHeight=1420&originWidth=1544&originalType=binary&ratio=3&rotation=0&showTitle=true&size=335652&status=done&style=none&taskId=u4ca5f910-3877-4198-9814-fbcd048b10a&title=%E6%9C%AA%E9%87%87%E7%94%A8%E7%BD%91%E6%A0%BC%E8%9E%8D%E5%B9%B6%E7%AE%97%E6%B3%95&width=437.66668701171875 "未采用网格融并算法")![采用了网格融并算法](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686924692799-1f7f5566-7c56-42a4-b3f8-89bea780c4b2.png#averageHue=%23283535&clientId=u67177426-be85-4&from=paste&height=412&id=ub8da5572&originHeight=1463&originWidth=1569&originalType=binary&ratio=3&rotation=0&showTitle=true&size=297848&status=done&style=none&taskId=u7daddcf3-0a31-4796-9e8d-2f1777a22c5&title=%E9%87%87%E7%94%A8%E4%BA%86%E7%BD%91%E6%A0%BC%E8%9E%8D%E5%B9%B6%E7%AE%97%E6%B3%95&width=442 "采用了网格融并算法")
对比之后可以看到，采用了融并网格算法生成的三角面数比没有采用的少了将近7000左右。
## 总结
本篇文章简单实现了一个体素融并网格的生成算法。
如果要实现类似Minecraft那样效果，就需要在每个16×16小区域内判定对某个位置的方块进行删除或添加。所谓的删除或添加也就是像数组添加或删除一个位置信息。然后重新生成这个区域的网格就行了。实际可能会更复杂一些。
另外在地形生成上可以使用随机地图生成技术中的柏林算法，应该可以获得更自然的效果。

