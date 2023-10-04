## 概述
Godot4.0改进了2D部分的导航，`NavigationRegion2D`基本可以看做是3.X的`Navigation2D`的进化版本。
它的基本用法就是先绘制`navigation_polygon`，也就是导航网格，或者直白点就是“可通行区域”。然后将带有碰撞形状和`NavigationAgent2D`子节点的玩家放置到这个可通行区域，然后在`_physics_process`中判断并移动就可以了。
## 案例
### 创建地图
从网上随便找一了一张2D游戏的地形素材。（仅做学习使用）
![map.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/8438332/1686487947542-491a5ece-1efe-4e72-88a7-d5e885ac16ec.jpeg#averageHue=%232b3f29&clientId=u50fdf842-7e80-4&from=drop&height=348&id=uf0438def&originHeight=640&originWidth=640&originalType=binary&ratio=3&rotation=0&showTitle=false&size=48930&status=done&style=none&taskId=u71fd4b46-dc4c-432a-9e1a-eb8ee98eba8&title=&width=348)
我们创建一个2D场景，将地形素材拖入场景，根节点下自动创建`Sprite2D`节点，更名为map，然后继续添加一个`NavigationRegion2D`节点。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686487848671-82e8329e-690e-43a0-86f0-d24d259c4229.png#averageHue=%232a2f39&clientId=u50fdf842-7e80-4&from=paste&height=112&id=ue326a8e0&originHeight=337&originWidth=558&originalType=binary&ratio=3&rotation=0&showTitle=false&size=25218&status=done&style=none&taskId=u751f3240-55db-4ae6-ade0-6ecca1c0f44&title=&width=186)![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686487872122-b286495c-d2be-475a-80d7-f7cfe520f8b3.png#averageHue=%2331452b&clientId=u50fdf842-7e80-4&from=paste&height=356&id=uf412d094&originHeight=1068&originWidth=1074&originalType=binary&ratio=3&rotation=0&showTitle=false&size=1077449&status=done&style=none&taskId=u219a3ac3-9675-4ef4-9251-7b8e62b09df&title=&width=358)
选中`NavigationRegion2D`节点，为我们导入的地形素材手工绘制可通行区域的多边形。
![绘制可通行区域（也就是导航区域）](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686488168856-c030b6d5-ec38-4a76-9538-4a7285345db7.png#averageHue=%232e422a&clientId=u50fdf842-7e80-4&from=paste&height=421&id=uc77674f8&originHeight=1263&originWidth=1074&originalType=binary&ratio=3&rotation=0&showTitle=true&size=1139845&status=done&style=none&taskId=uc14d4ee6-d311-4e51-853a-46e338a192b&title=%E7%BB%98%E5%88%B6%E5%8F%AF%E9%80%9A%E8%A1%8C%E5%8C%BA%E5%9F%9F%EF%BC%88%E4%B9%9F%E5%B0%B1%E6%98%AF%E5%AF%BC%E8%88%AA%E5%8C%BA%E5%9F%9F%EF%BC%89&width=358 "绘制可通行区域（也就是导航区域）")
:::danger
**注意**

- 多个闭合多边形虽然可以被画出来，但即便有重合区域，却不被认为是可互相通行的；

因此，最好是一气呵成的绘制。
:::
### 创建玩家
用`CharacterBody2D`类型创建玩家场景，用工程自带的`icon.svg`作为玩家样貌(可以适当缩放)，添加圆形碰撞形状。
除此之外，添加一个`NavigationAgent2D`。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686485151635-a90d7c98-014f-4226-ac4a-92d600263716.png#averageHue=%232a7185&clientId=u50fdf842-7e80-4&from=paste&height=183&id=u3134b194&originHeight=550&originWidth=1272&originalType=binary&ratio=3&rotation=0&showTitle=false&size=74357&status=done&style=none&taskId=u2bcd6003-1842-4e36-9f5a-7d85941fb1f&title=&width=424)
为玩家创建如下代码：
```swift
# player.gd
extends CharacterBody2D

var move_speed = 200.0

var target_pos:Vector2:
	set(val):
		target_pos = val
		nav.target_position = val

@onready var nav = $NavigationAgent2D

func _physics_process(delta):
	if nav.is_navigation_finished():
		return
	if nav.is_target_reachable():
		var next_pos = nav.get_next_path_position()
		velocity = global_position.direction_to(next_pos) * move_speed
		move_and_slide()

```
其中：

- 第6行：定义`target_pos`变量，并通过Setter函数，传递给`NavigationAgent2D`的`target_position`属性。
- 在`_physics_process`中我们通过`NavigationAgent2D`的`get_next_path_position()`方法获取下一个可以到达的位置，并通过获取当前位置（也就是玩家的global_position）到下一位置的方向乘以移动速度来获取玩家的速度向量，然后执行move_and_slide()进行移动。
- 当玩家到达指定位置后，`NavigationAgent2D`的`is_navigation_finished()`就会变为true，直接return就可以了。
- 通过`is_target_reachable()`可以判断，多指定的点是否可以到达。但是设置后，玩家不会尽量像目标位置移动，而是直接不走了。

为world2场景添加Palyer场景实例，并将玩家放置在可通行区域内。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/8438332/1686489101275-4aac234a-6e34-4fe0-ae0b-03dd8041e32e.png#averageHue=%23315e53&clientId=u50fdf842-7e80-4&from=paste&height=427&id=uf6940829&originHeight=1280&originWidth=1723&originalType=binary&ratio=3&rotation=0&showTitle=false&size=1238954&status=done&style=none&taskId=u40fea39e-8de5-465a-80d0-311377bd33c&title=&width=574.3333333333334)
为world2场景根节点添加如下代码：
```swift
# world2.gd
extends Node2D

@onready var player = $Player

func _input(event):
	if event is InputEventMouseButton:
		if event.button_index == MOUSE_BUTTON_LEFT and event.is_pressed():
			player.target_pos = get_global_mouse_position()
```
其核心是通过鼠标左键点击，将鼠标的全局位置传递给player的`target_pos`属性，也就是间接的传递给`Player`下`NavigationAgent2D`的`target_position`属性。
## 运行效果
运行后的效果如下：
[![基于NavigationRegion2D的寻路.mp4 (29.04MB)](https://gw.alipayobjects.com/mdn/prod_resou/afts/img/A*NNs6TKOR3isAAAAAAAAAAABkARQnAQ)]()## 总结
可以看到，这种导航形式非常简单，通过在地图上绘制可通行区域，在角色身上添加导航代理，然后将想要到达的位置传递给导航代理，再在物理帧处理函数中不断地获取下一个位置，并动态计算速度向量，然后`move_and_slide()`的就可以了。
## 参考

- [【Godot 4】2D寻路教程-2D Pathfinding Tutorial_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1GP411Q7x9/?spm_id_from=333.337.search-card.all.click&vd_source=2f49ad1693bc7945c7bf165c74c86263)
