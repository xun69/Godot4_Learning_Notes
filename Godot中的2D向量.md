# Godot中的2D向量

巽星石 2022年11月26日10:56:12

## 概述
在Godot中，乃至一切游戏编程中，你应该都躲不开向量。这是每一个初学者都应该知道和掌握的内容，否则你将很难理解和实现某些其实原理非常简单的东西。
我也是好几次想尝试完全总结这一部分内容了，这次就趁着要写公众号文章和星球剪报，尝试一次性总下。让新手们不再和我当初初学时那样迷茫。本篇汇总了我两年时间里写过的一些关于向量的东西。
## 概念
Vector2D，二维向量，是指有两个分量的向量。
## 万能的向量
向量可以表示屏幕二维坐标系上的点的位置。可以表示方向，可以存储矩形的尺寸。也可以用一堆点表示一条折线路径。
## Node2D的朝向
![2D节点自身的朝向](https://cdn.nlark.com/yuque/0/2022/png/8438332/1669432081117-49f99a2b-c0fa-4a9d-b52e-7be60e23cbb4.png#averageHue=%23f3efdf&clientId=u911347b9-5360-4&from=paste&height=81&id=u3fc053ca&originHeight=195&originWidth=894&originalType=binary&ratio=1&rotation=0&showTitle=true&size=20138&status=done&style=none&taskId=u065b1bd5-f087-4db5-8193-5fead16a85b&title=2D%E8%8A%82%E7%82%B9%E8%87%AA%E8%BA%AB%E7%9A%84%E6%9C%9D%E5%90%91&width=369.6666259765625 "2D节点自身的朝向")
**每一个2D物体其实有两个方向**：

- 第一个是它自身的**朝向**，也就是由它的`rotation_degree`属性所定义的方向
- 第二个是在移动过程中，从自己的位置到目标位置的方向，也就是**移动方向**

不能只知道移动的方向，却忽略物体自身的朝向。在实际的设计中两者配合，才能做出更好的效果。
在Godot中确实没有“朝向”这个概念，只有`rotation_degree`属性，但是却的的确确向我们展示了“朝向”的存在。`look_at()`方法和经典的“Sprite随鼠标定位旋转”示例就表明了这一点。
### Sprite随鼠标定位旋转
创建如下的场景，将icon.png拖进来，放到视口矩形范围的中间位置。
![image.png](https://cdn.nlark.com/yuque/0/2022/png/8438332/1652604769461-d0ce7c85-152b-43e0-9887-8f63ea79ecde.png#averageHue=%23e1e1e1&clientId=u93f03e54-7400-4&from=paste&height=120&id=ysMgO&originHeight=287&originWidth=494&originalType=binary&ratio=1&rotation=0&showTitle=false&size=18314&status=done&style=none&taskId=u22c3c041-733f-4b50-a786-754e80d4838&title=&width=205.83332515425184)![image.png](https://cdn.nlark.com/yuque/0/2022/png/8438332/1669434064577-63b75c84-d228-48e0-8d07-5926e450899f.png#averageHue=%234d4d4d&clientId=uc8cdf873-bfb6-4&from=paste&height=96&id=u27d22be4&originHeight=172&originWidth=231&originalType=binary&ratio=1&rotation=0&showTitle=false&size=4900&status=done&style=none&taskId=u78dc07b1-3a67-4a2f-8117-ef2f5d12444&title=&width=128.33332823382503)
为`Icon`节点添加如下代码：
```swift
extends Sprite


func _process(delta):
	look_at(get_global_mouse_position())
```
运行后就可以看到一个始终朝向鼠标位置的效果。而这里可以看到，始终是右边朝向鼠标位置。
## ![Sprite节点始终“看”向鼠标位置](https://cdn.nlark.com/yuque/0/2022/gif/8438332/1652605265788-9471bbc9-7565-4b95-97ea-415f83969adc.gif#averageHue=%235b5b5b&clientId=u93f03e54-7400-4&from=drop&height=321&id=ud947c267&originHeight=678&originWidth=1032&originalType=binary&ratio=1&rotation=0&showTitle=true&size=818358&status=done&style=none&taskId=u58be99c7-712a-4080-81ea-de204deac61&title=Sprite%E8%8A%82%E7%82%B9%E5%A7%8B%E7%BB%88%E2%80%9C%E7%9C%8B%E2%80%9D%E5%90%91%E9%BC%A0%E6%A0%87%E4%BD%8D%E7%BD%AE&width=488.44268798828125 "Sprite节点始终“看”向鼠标位置")
如果不够直观，我们可以给Sprite节点添加一个箭头，与其“朝向”保持一致。
![image.png](https://cdn.nlark.com/yuque/0/2022/png/8438332/1669432847476-aefe6366-0867-4404-8ca1-4b2237c7adc7.png#averageHue=%234f555c&clientId=uc8cdf873-bfb6-4&from=paste&height=81&id=ubc12a693&originHeight=145&originWidth=201&originalType=binary&ratio=1&rotation=0&showTitle=false&size=7496&status=done&style=none&taskId=u95ea6159-b93b-42c7-a1cc-b24d5683b67&title=&width=111.66666222943218)
你就可以看到更清晰的效果。
![用红色箭头指示Sprite节点的朝向](https://cdn.nlark.com/yuque/0/2022/gif/8438332/1669433018009-03e374c1-70c6-4f34-9771-bac673fb6341.gif#averageHue=%234b4b4d&clientId=uc8cdf873-bfb6-4&from=drop&id=u03dfe2a6&originHeight=368&originWidth=532&originalType=binary&ratio=1&rotation=0&showTitle=true&size=311343&status=done&style=none&taskId=u5fd3af5c-8011-488a-9a30-d04a6eeeef8&title=%E7%94%A8%E7%BA%A2%E8%89%B2%E7%AE%AD%E5%A4%B4%E6%8C%87%E7%A4%BASprite%E8%8A%82%E7%82%B9%E7%9A%84%E6%9C%9D%E5%90%91 "用红色箭头指示Sprite节点的朝向")
如果看到上面的示例，你能够想到经典游戏《祖玛》，说明你很聪明。
![经典游戏《祖玛》中随鼠标定位旋转的效果](https://cdn.nlark.com/yuque/0/2022/gif/8438332/1669433581384-ca130812-9eea-486b-9df3-04592c0bab76.gif#averageHue=%2368767f&clientId=uc8cdf873-bfb6-4&from=drop&height=404&id=ufb92e53f&originHeight=661&originWidth=927&originalType=binary&ratio=1&rotation=0&showTitle=true&size=6793553&status=done&style=none&taskId=ub77269c0-2af3-4d26-8a2d-bcac0950236&title=%E7%BB%8F%E5%85%B8%E6%B8%B8%E6%88%8F%E3%80%8A%E7%A5%96%E7%8E%9B%E3%80%8B%E4%B8%AD%E9%9A%8F%E9%BC%A0%E6%A0%87%E5%AE%9A%E4%BD%8D%E6%97%8B%E8%BD%AC%E7%9A%84%E6%95%88%E6%9E%9C&width=566.9982299804688 "经典游戏《祖玛》中随鼠标定位旋转的效果")
这里我们不做《祖玛》的示例，换为两张坦克素材图片，组成一个坦克。
![image.png](https://cdn.nlark.com/yuque/0/2022/png/8438332/1669433723676-ba972452-4afa-44cf-b154-8b838cb1129d.png#averageHue=%23e5e5e5&clientId=uc8cdf873-bfb6-4&from=paste&height=154&id=u5cf105d4&originHeight=278&originWidth=387&originalType=binary&ratio=1&rotation=0&showTitle=false&size=20199&status=done&style=none&taskId=uc46fe34a-8f9c-4d86-a2f5-332ca24ff16&title=&width=214.9999914566679)![image.png](https://cdn.nlark.com/yuque/0/2022/png/8438332/1669433313354-3f6b8abd-9eb2-4fff-9bbf-e027fdda99d3.png#averageHue=%2364594f&clientId=uc8cdf873-bfb6-4&from=paste&height=102&id=u169d7c07&originHeight=184&originWidth=402&originalType=binary&ratio=1&rotation=0&showTitle=false&size=36550&status=done&style=none&taskId=u173fc49f-8b76-4582-a077-af36e48e0e2&title=&width=223.33332445886435)
我们将之前的代码放到坦克的“炮身”也就是`cannon`节点上。
你立马就得到了一个可以随鼠标位置瞄准的坦克。
![坦克随鼠标定位瞄准](https://cdn.nlark.com/yuque/0/2022/gif/8438332/1669433892378-e85c981f-4897-42c0-a8c0-bb31389572c3.gif#averageHue=%235b5a58&clientId=uc8cdf873-bfb6-4&from=drop&height=389&id=ufcf14c8d&originHeight=660&originWidth=1041&originalType=binary&ratio=1&rotation=0&showTitle=true&size=711918&status=done&style=none&taskId=u365ff173-0a71-4b70-ab4d-9860ce681b4&title=%E5%9D%A6%E5%85%8B%E9%9A%8F%E9%BC%A0%E6%A0%87%E5%AE%9A%E4%BD%8D%E7%9E%84%E5%87%86&width=612.9982299804688 "坦克随鼠标定位瞄准")
## 向量与位置
### 屏幕坐标系
在二维平面上表示一个点，最简单的方式就是定义一个平面坐标系，然后用一对(x,y)坐标来表示。Godot采用的坐标系与大多数计算机可视化编程所采用的是一致的，也就是以左上角为原点（0,0），X轴正方向向右，Y轴正方向向下的“屏幕坐标系”。
![Godot的屏幕坐标系](https://cdn.nlark.com/yuque/0/2022/png/8438332/1669434587309-883844f5-8550-4d55-af00-ed2b2db62f2e.png#averageHue=%23f5f5f5&clientId=uc8cdf873-bfb6-4&from=paste&height=418&id=u347ae09c&originHeight=982&originWidth=1335&originalType=binary&ratio=1&rotation=0&showTitle=true&size=13949&status=done&style=none&taskId=u6338f7b3-aa25-4f11-8d56-8534d685e1b&title=Godot%E7%9A%84%E5%B1%8F%E5%B9%95%E5%9D%90%E6%A0%87%E7%B3%BB&width=568.6666259765625 "Godot的屏幕坐标系")
在Godot的2D工作区视口中，你可以看到Y轴用绿色，X轴用紫色，左上角原点也会有特殊的标记。
![实际视口中的XY轴、原点和视口可视区域矩形](https://cdn.nlark.com/yuque/0/2022/png/8438332/1669434857617-657caec6-ad0d-438d-824e-5171dec8fd46.png#averageHue=%234d4d4d&clientId=uc8cdf873-bfb6-4&from=paste&height=351&id=u6e18725e&originHeight=631&originWidth=1030&originalType=binary&ratio=1&rotation=0&showTitle=true&size=5278&status=done&style=none&taskId=ub0359d8e-43f0-4cd3-9e01-8dc6fcd573b&title=%E5%AE%9E%E9%99%85%E8%A7%86%E5%8F%A3%E4%B8%AD%E7%9A%84XY%E8%BD%B4%E3%80%81%E5%8E%9F%E7%82%B9%E5%92%8C%E8%A7%86%E5%8F%A3%E5%8F%AF%E8%A7%86%E5%8C%BA%E5%9F%9F%E7%9F%A9%E5%BD%A2&width=572.2221994841549 "实际视口中的XY轴、原点和视口可视区域矩形")
## 视口矩形
游戏窗口的尺寸是有限的，但是游戏地图或者游戏的世界可以比游戏窗口大得多。我们运行场景或项目时，窗口首先展示的屏幕范围，我们可以将其称为“第一屏”，这和网页设计中的“第一屏”概念是类似的。
“第一屏”是一个矩形区域，你可以用`get_tree().root.get_visible_rect()`获得包含其信息的`Rect2`数据。它与我们在项目设置中设置的窗口大小可能一致，也可能不一致，因为游戏窗口可以在运行过程中改变大小，并且还受缩放模式等设置的影响。
![项目设置中的窗口大小](https://cdn.nlark.com/yuque/0/2022/png/8438332/1669435037169-de4690bc-0fcc-434d-be47-4f835a587f6a.png#averageHue=%23e2e1e1&clientId=uc8cdf873-bfb6-4&from=paste&height=435&id=u07e1fd1f&originHeight=1055&originWidth=1417&originalType=binary&ratio=1&rotation=0&showTitle=true&size=135930&status=done&style=none&taskId=u9fc6c57d-b075-49a1-92ef-0dfe56bafe4&title=%E9%A1%B9%E7%9B%AE%E8%AE%BE%E7%BD%AE%E4%B8%AD%E7%9A%84%E7%AA%97%E5%8F%A3%E5%A4%A7%E5%B0%8F&width=583.9982299804688 "项目设置中的窗口大小")
游戏中，我们通常不会始终待在“第一屏”，除非你的设计就是每个关卡地图就是一屏的大小。通过摄像机我们可以看到关卡地图的其他地方。
### 位置
好了，讲了必须讲的前置知识后，回到正题——“向量与位置”。在平面坐标系中，表示一个点的位置，就是用它在X轴和Y轴上的投影处的值组成的坐标。
![点的坐标表示](https://cdn.nlark.com/yuque/0/2022/png/8438332/1669436011723-a13f1ad5-80db-4c7b-bcba-a8a809166983.png#averageHue=%23ebebeb&clientId=uc8cdf873-bfb6-4&from=paste&height=240&id=u5b643d53&originHeight=604&originWidth=735&originalType=binary&ratio=1&rotation=0&showTitle=true&size=19809&status=done&style=none&taskId=ueb423c09-9c1a-49d8-b308-511cd4f4ee7&title=%E7%82%B9%E7%9A%84%E5%9D%90%E6%A0%87%E8%A1%A8%E7%A4%BA&width=292.33331298828125 "点的坐标表示")
**那这又怎么跟向量扯上关系了呢？**
向量其实可以理解为“相对移动”或“相对位置”，这种“相对性”其实很让人迷糊。但是举个例子你就明白了：
已知我们在地球上，已经通过地磁场确定了东西南北（废话），假设你的家乡在某个小镇A，但是你大学毕业后到了某个省的省会城市B工作，那么描述你的家乡小镇A和你工作的城市B之间的相对位置关系，你会怎么描述呢？
![image.png](https://cdn.nlark.com/yuque/0/2022/png/8438332/1669437233957-43b50607-8794-475f-a938-3e027d780730.png#averageHue=%2385a942&clientId=uc8cdf873-bfb6-4&from=paste&height=395&id=ude852438&originHeight=711&originWidth=885&originalType=binary&ratio=1&rotation=0&showTitle=false&size=123420&status=done&style=none&taskId=u9b189b44-3963-44fb-9b4c-4027a123779&title=&width=491.6666471295894)
你会说你的家乡小镇A在你工作所在城市B的西北方向1200公里（数据胡诌），而你工作的城市B在你老家小镇A的东南方向1200公里公里。
你会发现两个地点的相对位置，可以用一个“方向+距离”的形式轻松的描述清楚。而“**方向+距离”就是向量**。
**在相对中引入绝对**
世上本没有东西南北，人类用某种科学规律和约定俗成规定了东西南北的方向，地球也本没有经纬，同样是人类用某种科学规律和科学家之间的约定俗成创建了一个可以定位地球某个点的方法。同样的在二维平面上，本没有坐标系，为了方便描述位置，才引入了平面坐标系。类比《圣经·创世纪》中：世上本没有光，“神说“要有光”，就有了光”，科学家们何尝不是分开混沌，理清世界的“神”。
在屏幕坐标系中，左上角是坐标系原点`（0,0）`，那么屏幕上的任何一点都可以被理解为“基于坐标系原点`（0,0）`在某个方向上偏移多少距离”，或者“相对坐标系原点的某个方向和距离的点”。这也就与我们上面所说的两个地点的相对位置描述一致了，只不过其中的一个点固定成了坐标系原点`（0,0）`。
那么以下图为例，点`（120,80）`的向量含义就是“相对坐标系原点`（0,0）`的XX方向上移动YY距离的点”。而这就是屏幕坐标点与向量关系的由来。
![平面坐标点的向量表示](https://cdn.nlark.com/yuque/0/2022/png/8438332/1669436248391-a73414bc-6ef3-4e37-9922-b0026b34ab09.png#averageHue=%23efefee&clientId=uc8cdf873-bfb6-4&from=paste&height=263&id=vwHMA&originHeight=731&originWidth=842&originalType=binary&ratio=1&rotation=0&showTitle=true&size=29231&status=done&style=none&taskId=u2a7509d2-76b3-45b8-a1d6-0ca7c95cf1f&title=%E5%B9%B3%E9%9D%A2%E5%9D%90%E6%A0%87%E7%82%B9%E7%9A%84%E5%90%91%E9%87%8F%E8%A1%A8%E7%A4%BA&width=302.77081298828125 "平面坐标点的向量表示")
那么这里的方向和距离到底是什么呢，如何计算？
![从平面坐标点的向量表示到直角三角形](https://cdn.nlark.com/yuque/0/2022/png/8438332/1669442306805-f9af65ce-c3e6-453d-b766-eca405bca319.png#averageHue=%23fafafa&clientId=uc8cdf873-bfb6-4&from=paste&height=312&id=u6100df07&originHeight=561&originWidth=1250&originalType=binary&ratio=1&rotation=0&showTitle=true&size=33676&status=done&style=none&taskId=uc094563f-8d11-44dc-aeb8-2cd4cd1f0fc&title=%E4%BB%8E%E5%B9%B3%E9%9D%A2%E5%9D%90%E6%A0%87%E7%82%B9%E7%9A%84%E5%90%91%E9%87%8F%E8%A1%A8%E7%A4%BA%E5%88%B0%E7%9B%B4%E8%A7%92%E4%B8%89%E8%A7%92%E5%BD%A2&width=694.4444168497026 "从平面坐标点的向量表示到直角三角形")
我们先说距离，平面坐标系上的任何一个点，它在X轴、Y轴投影与向量之间围成一个直角三角形（如上图右）。那么根据勾股定律，**向量的长度**![](https://cdn.nlark.com/yuque/__latex/397ffa6a9d85dedb91e738b303f859a0.svg#card=math&code=d%3D%5Csqrt%7Bx%5E2%20%2B%20y%5E2%7D&id=hNHld)。
在GDScript中我们可以直接使用`Vector2`的`length()`方法获取向量的长度，如果是屏幕点的位置的话，它所求的就是从屏幕坐标系原点到这个点的距离。
```swift
get_global_mouse_position().length()
```
除此之外，你可以用`Vector2`的`distance_to()`求屏幕上任意两个点之间的**距离**。
```swift
Vector2(100,100).distance_to(Vector2(200,200))
```
那么方向呢？方向我们**单位向量**来表示，单位向量是指长度为1的向量，计算的话就是`**单位向量 = **向量/向量长度`。在Godot中你同样可以省下这种计算，只用一个方法搞定，`Vector2`的`normalized()`就是求一个向量的单位向量。
```swift
Vector2(100,100).normalized()
```

![单位向量 = 长度为1的向量](https://cdn.nlark.com/yuque/0/2022/png/8438332/1669442863096-10a4f9d7-bc11-48fd-8ef8-c277139dee0d.png#averageHue=%23f7f7f7&clientId=uc8cdf873-bfb6-4&from=paste&height=315&id=uf28962da&originHeight=815&originWidth=917&originalType=binary&ratio=1&rotation=0&showTitle=true&size=40998&status=done&style=none&taskId=u372c58d5-cda4-4be3-a03c-daf9502fc03&title=%E5%8D%95%E4%BD%8D%E5%90%91%E9%87%8F%20%3D%20%E9%95%BF%E5%BA%A6%E4%B8%BA1%E7%9A%84%E5%90%91%E9%87%8F&width=354.4444274902344 "单位向量 = 长度为1的向量")
单位向量的好处是，它的长度是1，1乘以任何数就是这个数本身，同时它又保存了向量的“方向”。那么我们就可以用单位向量乘以任何标量来“缩放向量”，同时也可以用**单位向量×长度（或叫距离）**来表示一个向量。也就回到了“在某个方向上偏移多少距离”的含义。
进一步的考虑在一个二维平面上，所有可能的方向都包含在一个中心点为坐标原点，半径为1的圆里。想想游戏手柄的摇杆和手机游戏的虚拟摇杆，你就应该明白，为什么它们可以控制你的游戏角色或视角移动了吧。
![image.png](https://cdn.nlark.com/yuque/0/2022/png/8438332/1669444307835-03840d7d-e44e-431f-9e14-a5aacb5057db.png#averageHue=%23f8f8f8&clientId=uc8cdf873-bfb6-4&from=paste&height=271&id=u7d055c75&originHeight=557&originWidth=674&originalType=binary&ratio=1&rotation=0&showTitle=false&size=27967&status=done&style=none&taskId=u20d791f2-da1b-4752-9174-a842407e66d&title=&width=327.4444274902344)
当然手柄的摇杆和手机游戏的虚拟摇杆还进一步检测了你拖动摇杆的力度，所以它不再是只包含圆周上的那些点，而是包含了圆周和圆内所有的点。
## 向量与移动
有了上面的基础，那么就应该研究物体在二维平面上的移动了。从A点到B点的移动可以理解为单纯的距离缩短。也可以描述为“A不停的向B移动，直到A和B重合（距离=0）”。
![基于向量移动的基础示意图](https://cdn.nlark.com/yuque/0/2022/png/8438332/1669445796388-421292b4-eb98-409e-b305-2ad2725c5eaf.png#averageHue=%23f8f7f7&clientId=uc8cdf873-bfb6-4&from=paste&height=328&id=Vuixb&originHeight=591&originWidth=684&originalType=binary&ratio=1&rotation=0&showTitle=true&size=50733&status=done&style=none&taskId=u9d06230b-aa51-4426-942d-b46d67e965a&title=%E5%9F%BA%E4%BA%8E%E5%90%91%E9%87%8F%E7%A7%BB%E5%8A%A8%E7%9A%84%E5%9F%BA%E7%A1%80%E7%A4%BA%E6%84%8F%E5%9B%BE&width=379.99998490015724 "基于向量移动的基础示意图")
那么体现在代码上就是需要知道A到B的方向和距离，然后定义单位时间内移动的速度，然后就可以移动了。
A到B的方向`direction`可以用`A.direction_to(B)`求得，A到B的距离`distance`可以用`A.distance_to(B)`。
定义单位时间的移动距离，也就是速度`speed`，那么速度向量`velocity = direction * speed`，也就是`方向*移动距离`。
不要懵了，`direction`、`distance`和`velocity`是初学者学习基础移动必须学会的三个单词：

- `direction`：方向，申明变量时可以简写为`dir`
- `distance`：距离，申明变量时可以简写为`dis`或`d`
- `velocity`：(沿某一方向的)速度，申明变量时可以简写为`vec`或`v`

无论是申明变量还是使用`Vector2`的方法你都会遇到这三个单词。
整个原理就是先判断起始点到目标点的距离是否大于0，然后将A的位置加上速度向量`velocity`，移动一段距离，然后循环，直到距离=0。
具体可以参阅相关的示例。
这是用纯向量方法移动物体的形式，Godot中移动物体和实现碰撞需要用物理体那套，但是基础的基于向量的移动是必学的基础，它在某些时候会有用处。
## 局部坐标系
就像人类曾经经历了地心说和日心说，再到现在的宏大宇宙观，中心与原点有相似性，它也可以因为不同的认识和参照定义而发生变化。
你可以将二维平面的任意一点作为原点构造一个平面坐标系。但是你或者别人完全可以选择除了这一个点之外的任何一个点的位置重新建立一个坐标系。同一个点会因为你设立的坐标系不同而有不同的坐标值表示。
同时在二维平面的某个局部，你又可以创建一个局部坐标系。局部坐标系的好处是，它可以只描述局部范围内的内容，而忽略其他的东西。
相对的你可以将它的上一级坐标系称为“全局坐标系”，当然这很违心，因为“全局坐标系”本质上也是一个“局部坐标系”，因为它的外面可以有更大的坐标系。
就像地球的经纬度系统就可以看做是地球的全局坐标系，但是在太阳系乃至更大的银河系和宇宙来说，坐标系还可以随着讨论范围逐渐扩大。
但是在Godot里，一般情况下你就可以认为屏幕坐标系就是“最大”的坐标系，是“全局坐标系”，一个场景的根节点其“局部坐标系”默认与“全局坐标系”重合，除非你不移动它。而每一层的子节点，都有一个自己的“局部坐标系”。
在Godot的API中也体现了全局位置、缩放、旋转和局部位置、缩放、旋转的概念。
![全局坐标系与局部坐标系](https://cdn.nlark.com/yuque/0/2022/png/8438332/1669441650310-db5162c4-5b85-43be-82d1-ae75fbeefe76.png#averageHue=%23efeeee&clientId=uc8cdf873-bfb6-4&from=paste&height=474&id=u187be044&originHeight=1040&originWidth=1248&originalType=binary&ratio=1&rotation=0&showTitle=true&size=40582&status=done&style=none&taskId=u46201323-c46f-4a96-9b0d-3f306fb35dd&title=%E5%85%A8%E5%B1%80%E5%9D%90%E6%A0%87%E7%B3%BB%E4%B8%8E%E5%B1%80%E9%83%A8%E5%9D%90%E6%A0%87%E7%B3%BB&width=569.3263549804688 "全局坐标系与局部坐标系")
## 极坐标系
大致说完平面坐标系和二维向量，再加入一点极坐标和三角函数的内容。在一个平面上表示一个点的位置，不止有平面坐标系法和向量法，还有极坐标系法。
极坐标系（polar coordinates）是指在平面内由极点、极轴和极径组成的坐标系。在平面上取定一点O，称为极点。从O出发引一条射线Ox，称为极轴。再取定一个单位长度，通常规定角度取逆时针方向为正。这样，平面上任一点P的位置就可以用线段OP的长度ρ以及从Ox到OP的角度θ来确定，有序数对（ρ，θ）就称为P点的极坐标，记为P（ρ，θ）；ρ称为P点的极径，θ称为P点的极角。
![点的极坐标表示](https://cdn.nlark.com/yuque/0/2022/png/8438332/1669448378965-22de84fb-87a8-41a6-b444-619de38de27f.png#averageHue=%23fcfcfc&clientId=uc8cdf873-bfb6-4&from=paste&height=381&id=u8d5e7d7c&originHeight=686&originWidth=705&originalType=binary&ratio=1&rotation=0&showTitle=true&size=78632&status=done&style=none&taskId=u38eb9c72-8586-45b6-82fe-02cec531dac&title=%E7%82%B9%E7%9A%84%E6%9E%81%E5%9D%90%E6%A0%87%E8%A1%A8%E7%A4%BA&width=391.66665110323225 "点的极坐标表示")
它的本质是规定一个正方向，表示0度，然后平面内的某个点就可以表示为方向为与这个正方向的夹角，距离为原点到这个点的距离，两个参数确定一个点。依然是方向和距离，但是用了角度和长度。
在Godot中可以理解为，2D节点的朝向，也就是`rotation_degree`属性，遵循了这种坐标系，但是依然是反的，它的角度顺时针方向取正，逆时针方向取负。并且`rotation_degree`属性的单位是度，而不是弧度。
### 向量的夹角
在GDScript中，你可以用`Vector2`的`angle()`求某个向量与X轴正方向的夹角。
```swift
print(Vector2(100,100).angle()) # 0.785398，单位：弧度
```
因此你完全可以将一个二维坐标系点的位置，通过封装`（到原点的距离，与X轴正方向夹角（弧度））`来获得其极坐标表示。
你也可以用`A.angle_to(B)`求两个向量之间的夹角。
![angle()和angle_to()](https://cdn.nlark.com/yuque/0/2022/png/8438332/1669450055103-6c4b15a1-bca3-4628-96bf-33461d64695e.png#averageHue=%23f7f5f4&clientId=uc8cdf873-bfb6-4&from=paste&height=437&id=u00920ce5&originHeight=786&originWidth=970&originalType=binary&ratio=1&rotation=0&showTitle=true&size=62340&status=done&style=none&taskId=u3e5a2cf6-ed51-49a1-86fd-f48158bb492&title=angle%28%29%E5%92%8Cangle_to%28%29&width=538.8888674753692 "angle()和angle_to()")
用`A.angle_to_point(B)`求**由B到A的向量**与X轴正方向的夹角。
![A.angle_to_point(B)](https://cdn.nlark.com/yuque/0/2022/png/8438332/1669450442955-f527dc94-08cc-46c6-866f-6e317f9d9d2f.png#averageHue=%23faf8f7&clientId=u73bb21fe-7baf-4&from=paste&height=264&id=UM4F7&originHeight=593&originWidth=906&originalType=binary&ratio=1&rotation=0&showTitle=true&size=33089&status=done&style=none&taskId=u8ec289dd-eba7-4e24-ab7c-1d3e7c75b77&title=A.angle_to_point%28B%29&width=403.33331298828125 "A.angle_to_point(B)")
`B.angle_to_point(A)`求由**A到B的向量**与X轴正方向的夹角。
![B.angle_to_point(A)](https://cdn.nlark.com/yuque/0/2022/png/8438332/1669450432671-f81285e7-6035-4988-93ff-2fb568ee0d17.png#averageHue=%23f9f7f6&clientId=u73bb21fe-7baf-4&from=paste&height=361&id=u2f512838&originHeight=650&originWidth=733&originalType=binary&ratio=1&rotation=0&showTitle=true&size=31970&status=done&style=none&taskId=u3e82b670-b4ac-4f88-986d-21a3e2537a5&title=B.angle_to_point%28A%29&width=407.2222060406656 "B.angle_to_point(A)")
在三者中，`angle_to_point()`应该是最让人费解的，但是平常也用不到，初学时不必深究。
### 向量的旋转
说完夹角再说旋转。向量除了加减乘除运算之外，还可以旋转。通过将一个向量旋转一定角度，可以获得一个新的向量。
向量的旋转有时候也很有用，比如在用`Line2D`节点绘制多边形或圆时，就会用到。
### Line2D和PoolVector2Array
`Line2D`节点用于绘制路径，它的`points`属性存储构成路径的所有顶点，是`PoolVector2Array`类型。`PoolVector2Array`你可以简单理解为只存储`Vector2`类型的特殊数组。
![image.png](https://cdn.nlark.com/yuque/0/2022/png/8438332/1669451567688-188f06b8-89e7-46a5-96dd-d99feab2f647.png#averageHue=%23d9d9d9&clientId=uc8cdf873-bfb6-4&from=paste&height=56&id=uaf3ea4ef&originHeight=100&originWidth=746&originalType=binary&ratio=1&rotation=0&showTitle=false&size=9766&status=done&style=none&taskId=ub979de2f-f156-42ed-b4f0-d522c6593b9&title=&width=414.4444279759025)
你可以直接在编辑器中手动绘制路径的顶点。注意它记录的是全局坐标，也就是基于屏幕左上角的位置，与自身的层级和局部坐标无关。
![在编辑器中手动绘制Line2D的路径顶点](https://cdn.nlark.com/yuque/0/2022/png/8438332/1669451697996-403d0127-60c0-4330-bf0b-29af92017124.png#averageHue=%23676565&clientId=uc8cdf873-bfb6-4&from=paste&height=364&id=u6c369f30&originHeight=833&originWidth=1092&originalType=binary&ratio=1&rotation=0&showTitle=true&size=34172&status=done&style=none&taskId=u31ba6dcb-6a8c-474d-81ed-99eee8be492&title=%E5%9C%A8%E7%BC%96%E8%BE%91%E5%99%A8%E4%B8%AD%E6%89%8B%E5%8A%A8%E7%BB%98%E5%88%B6Line2D%E7%9A%84%E8%B7%AF%E5%BE%84%E9%A1%B6%E7%82%B9&width=477.6666259765625 "在编辑器中手动绘制Line2D的路径顶点")
绘制后你可以在“检视器”面板展开`points`属性，看到其中记录的顶点坐标。
![points中记录的顶点坐标](https://cdn.nlark.com/yuque/0/2022/png/8438332/1669451795333-e24e17af-4644-43ed-9990-f5443c0dea32.png#averageHue=%23e3e3e3&clientId=uc8cdf873-bfb6-4&from=paste&height=319&id=u9ad7b90f&originHeight=786&originWidth=749&originalType=binary&ratio=1&rotation=0&showTitle=true&size=47730&status=done&style=none&taskId=u55b28898-975c-4465-8f0d-9e3d75ab3d8&title=points%E4%B8%AD%E8%AE%B0%E5%BD%95%E7%9A%84%E9%A1%B6%E7%82%B9%E5%9D%90%E6%A0%87&width=304.111083984375 "points中记录的顶点坐标")
同样的你可以用代码生成这些顶点坐标，而通过旋转向量的方法，我们可以用`Line2D`绘制圆、圆弧等等。
### 绘制多边形和圆
策略很简单：

- 指定**细分数**，或者多边形的边数，然后我们用`360/边数`获得单次要旋转的角度
- 指定**旋转半径**，我们将X轴正方向的单位向量`Vector2.RIGHT`乘以旋转半径就获得了我们初始要旋转的向量
- 然后我们将它旋转指定的角度，并将旋转得到的新的点的位置加入到一个`PoolVector2Array`中，通过多次旋转就得到了多个点
- 最后我们赋值`Line2D`的`points`属性为这个`PoolVector2Array`，搞定！
```swift
extends Line2D

var subdivision = 6 # 细分数
var r = 100 # 旋转半径
var center = Vector2(400,200) # 旋转中心


func _ready():
	width = 2
	
	var pots:PoolVector2Array = []
	var uint = Vector2.RIGHT # 向右的单位向量
	var per_angle = (2 * PI) / subdivision # 单次旋转角度
	var basic_vec = uint * r  # 要旋转基础向量
	pots.append(basic_vec + center)
	for i in range(1,subdivision+1):
		pots.append(basic_vec.rotated(per_angle * i)  + center)
	pots.append(basic_vec + center) # 回到第一个点，闭合
	points = pots
	pass
```
上面的代码运行后绘制的就是一个中心点在`（400,200）`，中心点到每个顶点的距离是`100`像素的正六边形。
![细分数 = 6](https://cdn.nlark.com/yuque/0/2022/png/8438332/1669453333414-0f688dca-1d55-4cc9-911a-9b12b0f9d3ae.png#averageHue=%235a5a58&clientId=uc8cdf873-bfb6-4&from=paste&height=364&id=ud3b79337&originHeight=656&originWidth=1046&originalType=binary&ratio=1&rotation=0&showTitle=true&size=54829&status=done&style=none&taskId=u190227a4-1825-4633-b77b-f6d410c2b96&title=%E7%BB%86%E5%88%86%E6%95%B0%20%3D%206&width=581.1110880198311 "细分数 = 6")
![细分数 = 3](https://cdn.nlark.com/yuque/0/2022/png/8438332/1669453435598-b2ddd606-894b-45c4-a7c6-1df64a81e860.png#averageHue=%234c4c4c&clientId=uc8cdf873-bfb6-4&from=paste&height=116&id=udba363e1&originHeight=209&originWidth=209&originalType=binary&ratio=1&rotation=0&showTitle=true&size=1256&status=done&style=none&taskId=u24c47508-590c-4a4a-9fce-6ac4cfb2daa&title=%E7%BB%86%E5%88%86%E6%95%B0%20%3D%203&width=116.11110649727027 "细分数 = 3")![细分数 = 4](https://cdn.nlark.com/yuque/0/2022/png/8438332/1669453453054-f15ecd4c-9c0f-4573-83e4-616fe414bdc5.png#averageHue=%234c4c4c&clientId=uc8cdf873-bfb6-4&from=paste&height=117&id=ub53e8305&originHeight=245&originWidth=249&originalType=binary&ratio=1&rotation=0&showTitle=true&size=1969&status=done&style=none&taskId=u38ea35d7-7d4d-48fa-9884-0a01a96f5c2&title=%E7%BB%86%E5%88%86%E6%95%B0%20%3D%204&width=119.33332824707031 "细分数 = 4")![细分数 = 5](https://cdn.nlark.com/yuque/0/2022/png/8438332/1669453471361-c0dd7266-7b8f-4f2d-8221-e21f0dabaddb.png#averageHue=%234c4c4c&clientId=uc8cdf873-bfb6-4&from=paste&height=116&id=u7791db32&originHeight=222&originWidth=218&originalType=binary&ratio=1&rotation=0&showTitle=true&size=1731&status=done&style=none&taskId=uf78a3d39-f7d0-404a-b51d-ce20f60a964&title=%E7%BB%86%E5%88%86%E6%95%B0%20%3D%205&width=114.1111068725586 "细分数 = 5")
圆和正多边形的唯一区别就是细分数，只要细分数达到一定的数量，就会“以直求曲”，从折线变成近似曲线的效果，这也是很多计算机软件里绘制圆形的奥秘。
![细分数=20](https://cdn.nlark.com/yuque/0/2022/png/8438332/1669453599090-321a0757-d752-43b2-9772-86a232627cc8.png#averageHue=%234c4c4c&clientId=uc8cdf873-bfb6-4&from=paste&height=131&id=u4254cf4c&originHeight=236&originWidth=232&originalType=binary&ratio=1&rotation=0&showTitle=true&size=1735&status=done&style=none&taskId=ub1c89d9e-b8cf-423d-8d39-21aa95d611c&title=%E7%BB%86%E5%88%86%E6%95%B0%3D20&width=128.8888837673048 "细分数=20")![细分数=100](https://cdn.nlark.com/yuque/0/2022/png/8438332/1669453558746-b0820c36-785a-4114-bd18-8c6953a02262.png#averageHue=%234c4c4c&clientId=uc8cdf873-bfb6-4&from=paste&height=129&id=uf8aaa6d6&originHeight=233&originWidth=236&originalType=binary&ratio=1&rotation=0&showTitle=true&size=1811&status=done&style=none&taskId=u514cac55-aab2-4861-93d1-ffd6a9f4475&title=%E7%BB%86%E5%88%86%E6%95%B0%3D100&width=131.11110590122385 "细分数=100")
## 绘制圆弧和扇形
学会了画圆，那么圆弧和扇形也就没有什么困难了。
```swift
extends Line2D

var subdivision = 5 # 细分数
var start_angle = deg2rad(45) # 起始角度
var end_angle = deg2rad(90) # 起始角度
var r = 100 # 旋转半径
var center = Vector2(400,200) # 选中中心


func _ready():
	width = 2
	
	var pots:PoolVector2Array = []
	var uint = Vector2.RIGHT # 向右的单位向量
	var d_angle = end_angle - start_angle if end_angle - start_angle >=0 else start_angle - end_angle
	var per_angle = d_angle / subdivision # 单次旋转角度
	var basic_vec = uint * r  # 要旋转基础向量
	
	print(basic_vec.rotated(start_angle))
	pots.append(center) # 添加中心点
	var start_point = basic_vec.rotated(start_angle) + center # 起始角度点
	pots.append(start_point) # 添加起始点
	
	for i in range(1,subdivision+1):
		pots.append(basic_vec.rotated(start_angle + per_angle * i)  + center)
	pots.append(center) # 回到中心点，闭合
	points = pots
	pass
```
![扇形](https://cdn.nlark.com/yuque/0/2022/png/8438332/1669454696635-c2b716f6-fbca-4bc7-a718-ab8401b45f18.png#averageHue=%234c4c4c&clientId=uc8cdf873-bfb6-4&from=paste&height=129&id=u102d14d0&originHeight=274&originWidth=288&originalType=binary&ratio=1&rotation=0&showTitle=true&size=1313&status=done&style=none&taskId=u582b46fd-57b0-4329-b0c5-3c364e69a73&title=%E6%89%87%E5%BD%A2&width=136 "扇形")![圆弧](https://cdn.nlark.com/yuque/0/2022/png/8438332/1669454767643-6ec38110-09dd-4909-b237-8a3c20426757.png#averageHue=%234c4c4c&clientId=uc8cdf873-bfb6-4&from=paste&height=143&id=u0a965d27&originHeight=257&originWidth=224&originalType=binary&ratio=1&rotation=0&showTitle=true&size=972&status=done&style=none&taskId=u0b72be46-ea5d-4e9f-9e10-3b5a3ace2bb&title=%E5%9C%86%E5%BC%A7&width=124.44443949946671 "圆弧")
上面的代码如果首尾都不加中心点，就变成了圆弧。
## 螺旋线
上面举例的都是只旋转，但是旋转半径不变的情况，但你完全可以尝试一下按参数规律变化半径的螺旋线之类的。
## 三角函数
涉及平面坐标系、位置和角度，那么三角函数也是躲不过去的一个话题，这里我只简单的说一下，知道半径和角度，如何计算坐标。其实一张图就够了，你可以自己思考一下为什么。
![image.png](https://cdn.nlark.com/yuque/0/2022/png/8438332/1652863966572-ad9fb38f-60c6-4675-a95e-303a68be99f6.png#averageHue=%23fafafa&clientId=ubbf94ead-30f2-4&from=paste&height=214&id=uc6be2bf0&originHeight=513&originWidth=758&originalType=binary&ratio=1&rotation=0&showTitle=false&size=20649&status=done&style=none&taskId=u14e26e5f-87b4-432c-8caa-82a4ef0154b&title=&width=315.83332078324474)
关于向量其实想说的还很多，但是限于篇幅和经历，这次只说这么多，希望对新手有所帮助。
