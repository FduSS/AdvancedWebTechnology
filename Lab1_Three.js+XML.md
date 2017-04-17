#  高级Web技术 Lab 1：Three.js 与 XML

2017.4

TA: 

李逢双 邮箱：13302010002@fudan.edu.cn，微信号：lfs941102

童仲毅 邮箱：13302010039@fudan.edu.cn，微信号：ZHONGYI_TONG

关于 Lab 的问题推荐询问李逢双助教，关于 PJ 的问题推荐询问童仲毅助教。

[TOC]

## 概述

Lab 1 的主要内容包括：

- 使用 three.js 构建 Web3D 场景
- 使用 XML 与 XML Schema 定义 Web3D 场景的配置信息
- 使用 Angular 2 框架开发 Web 前端



## Part 1: Three.js

Three.js 是一个 WebGL 库，对 WebGL API 进行了很好的封装。它库函数丰富，上手容易，非常适合 WebGL 开发。

> Three.js 的官方网址： https://threejs.org
>
> 首页左侧的 documentation 中是 three.js 的官方文档。
>
> 文档下方的 examples 中有许多经典的例子。
>
> 同学们学习 three.js 和开发 PJ 可以参考这两个。

### 一、场景（Scene）

首先，创建如下 HTML 文件（即 three.html）。

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset=utf-8>
		<title>My first three.js app</title>
		<style>
			body { margin: 0; }
			canvas { width: 100%; height: 100% }
		</style>
	</head>
	<body>
		<script src="three.js"></script>
		<script>
			// Our Javascript will go here.
		</script>
	</body>
</html>
```

接下来，在`<script>`标签内创建场景。

```javascript
var scene = new THREE.Scene();
```

场景是一个很重要的概念，每一个场景中都包含有摄像机、地形、天空盒子等对象，渲染时可以在不同的场景间进行切换。在资源紧张的应用中，场景还要负责游戏对象的创建和销毁工作。一般来说，游戏中的每一个关卡都是一个场景。

### 二、摄像机（Camera）

场景创建完毕后，需要向场景中添加摄像机。 

每个场景都至少应包含一个摄像机。这里选择创建的摄像机为透视摄像机，摄像机捕获的内容占满整个页面，其视角为 45°，坐标为(0, 20, 50)，并将摄像机的镜头指向点(0, 15, 0)。 最后，将摄像机添加到场景中。

```javascript
var SCREEN_WIDTH = window.innerWidth, SCREEN_HEIGHT = window.innerHeight;
var VIEW_ANGLE = 45, ASPECT = SCREEN_WIDTH / SCREEN_HEIGHT, NEAR = 0.3, FAR = 1000;
var camera = new THREE.PerspectiveCamera(VIEW_ANGLE, ASPECT, NEAR, FAR);
camera.position.set(0, 20, 50);
camera.lookAt(new THREE.Vector3(0, 15, 0));
scene.add(camera);
```

### 三、渲染器（Renderer）

摄像机创建好后，就应该创建渲染器了。我们选择创建的渲染器为 WebGLRenderer，并设定为抗锯齿，渲染器渲染的内容同样占满整个页面。最后将渲染器内部的`<canvas>`对象添加到 body 中。

```javascript
var renderer = new THREE.WebGLRenderer({antialias: true});
renderer.setSize(SCREEN_WIDTH, SCREEN_HEIGHT);
document.body.appendChild(renderer.domElement);
```

抗锯齿 `antialias` 是创建 WebGLRenderer 时的一个可选参数，更多可选参数可以查看 https://threejs.org/docs/index.html#Reference/Renderers/WebGLRenderer。在学习 Three.js 时勤看文档是一个好习惯。

然后我们需要创建渲染回调函数。requestAnimationFrame 是浏览器提供的 JavaScript API，传递回调函数为参数。

```javascript
function render() {
  requestAnimationFrame(render);
  renderer.render(scene, camera);
}
render();
```

至此，一个场景所必备的基本要素已经完成，打开浏览器访问可以看到整个页面变成了全黑，接下来我们需要往场景中添加各种实体对象。

### 四、天空盒子（Sky Box）

天空盒子和普通的几何物体并无不同。但天空盒子作为一种技术，可以将天空效果简单有效地表示出来，所以单独拿出来讲解。天空盒子，就是一个立方体对象。在实际应用中，用户视角只在盒子内部活动，所以只需要渲染盒子内部表面。值得注意的是，天空盒子应当足够大，使得摄像机在移动时看天空仍然觉得足够远。但是，天空盒子不能超出摄像机最远可视范围。

首先创建一个 CubeGeometry，材质选择 MeshBasicMaterial，设定为只渲染背面（对立方体来说，从外面看到的是正面，从内部看到的是背面），并将其添加到场景中。

```javascript
var skyBoxGeometry = new THREE.BoxGeometry(500, 500, 500);
var skyBoxMaterial = new THREE.MeshBasicMaterial({
      color: 0x9999ff, 
      side: THREE.BackSide
    });
var skyBox = new THREE.Mesh(skyBoxGeometry, skyBoxMaterial);
scene.add(skyBox);
```

刷新页面，可以观察到背景颜色变成了蓝色。

### 五、几何形状（Geometry）

> Geometry 建议配合文档学习
>
> 如 https://threejs.org/docs/index.html#Reference/Geometries/BoxGeometry

Three.js 中提供了许多预设的几何形状，如立方体（BoxGeometry），平面（PlaneGeometry），球体（SphereGeometry），立体文字（TextGeometry）等。使用 Geometry 可以方便地新建所需形状的物体。

比如天空盒子使用的就是立方体形状（BoxGeometry）。

### 六、材质（Material）

Material 对象定义了物体的材质，包括颜色、透明度、材质等等。Three.js 提供了一些预设材质，如 MeshBasicMaterial ，MeshPhongMaterial，MeshLambertMaterial 等，具体的 Material 参数与预设材质的定义请参考文档。

### 七、纹理（Texture）

Three.js 中调用材质一般使用 TextureLoader。使用方式如下：

```javascript
// instantiate a loader
var loader = new THREE.TextureLoader();

// load a resource
loader.load(
	// resource URL
	'textures/land_ocean_ice_cloud_2048.jpg',
	// Function when resource is loaded
	function ( texture ) {
		// Texture is loaded. Do something with the texture
		var material = new THREE.MeshBasicMaterial( {
			map: texture
		 } );
	},
	// Function called when download progresses
	function ( xhr ) {
		console.log( (xhr.loaded / xhr.total * 100) + '% loaded' );
	},
	// Function called when download errors
	function ( xhr ) {
		console.log( 'An error happened' );
	}
);
```

load 方法的第一个参数为材质所在路径或者 URL，第二个参数为材质加载成功之后的回调函数，第三个参数为下载过程中的回调函数，第四个参数为下载失败的回调函数。后两个参数如不需要可以不写。

由于 loader 使用了 XMLHttpRequest 来获取内容，我们需要部署我们的网页。推荐使用 Intellij 自带的网页部署功能。安装了 python 的同学也可以通过在 lab 目录运行 `python -m SimpleHTTPServer 8000` 部署服务器，通过 `localhost:8000` 访问。

### 八、网格（Mesh）

Mesh 在 three.js 中是由许多三角形平面组成的立体对象，大部分情况下，我们都使用 Mesh 来构建三维物体。

构造函数：Mesh (geometry, material)

网格样例：

![](https://upload.wikimedia.org/wikipedia/commons/f/fb/Dolphin_triangle_mesh.png)

### 九、添加物体

简单了解了相关概念后，让我们向我们的场景中添加一些物体。

#### 1. 地板

地板是一个平面，导入 image/hardwood2_diffuse.jpg 作为纹理，且纹理设为横向、纵向都重复 4 次。最后经过位移和旋转，添加到场景中。

添加如下代码：

```javascript
var textureLoader = new THREE.TextureLoader();

textureLoader.load("image/hardwood2_diffuse.jpg", function (texture) {
  texture.wrapS = texture.wrapT = THREE.RepeatWrapping;
  texture.repeat.set(4, 4);
  var floorMaterial = new THREE.MeshBasicMaterial({
      map: texture,
      side: THREE.DoubleSide
  });
  var floorGeometry = new THREE.PlaneGeometry(100, 100, 5, 5);
  var floor = new THREE.Mesh(floorGeometry, floorMaterial);
  floor.position.y = -0.5;
  floor.rotation.x = Math.PI / 2;
  scene.add(floor);
})
```

#### 2. 木箱

木箱使用 BoxGeometry，导入 image/crate.gif 作为纹理。

添加代码如下：

```javascript
textureLoader.load("image/crate.gif", function (texture) {
  var cubeGeometry = new THREE.BoxGeometry(10, 10, 10);
  var crateMaterial = new THREE.MeshBasicMaterial({map: texture});
  var cube = new THREE.Mesh(cubeGeometry, crateMaterial);
  cube.position.set(-20, 8, 0);
  cube.castShadow = true;
  scene.add(cube);
});
```

#### 3. 旋转的地球

地球是球体形状，导入 image/earth_atmos_4096.jpg 作为纹理，并修改渲染函数实现旋转效果：

添加如下代码：

```javascript
textureLoader.load("image/earth_atmos_4096.jpg", function (texture) {
  var sphereGeometry = new THREE.SphereGeometry(8, 32, 16);
  var sphereMaterial = new THREE.MeshBasicMaterial({map: texture});
  window.sphere = new THREE.Mesh(sphereGeometry, sphereMaterial);
  sphere.position.set(20, 10, 0);
  scene.add(sphere);
});
```

修改 render 函数：

```javascript
function render() {
  requestAnimationFrame(render);
  if (window.sphere != undefined) {
    window.sphere.rotation.y -= 0.01;
  }
  renderer.render(scene, camera);
}
```

> 在 window 的全局命名空间中定义变量是一个很不好的做法， 这里只是在代码量很小的情况下图个方便。

修改后，刷新浏览器，将看到如下界面，其中地球会在转动： ![Screen Shot 2017-04-09 at 11.21.24 PM](https://cloud.githubusercontent.com/assets/6532225/25094183/8257129e-23c8-11e7-9055-3afc74c29cb6.png)

嗯？你问为什么地球会在一个地板上？嘛，这就是个 Tutorial，别管那么多~(～o￣▽￣)～o

### 十、光源（Light）

我们现在添加的物体都是使用的 MeshBasicMaterial ，直接渲染材质颜色，无视光照。下面我们来添加一个使用 MeshPhongMaterial 的蓝紫色球体，并加入两种光源。

添加如下代码：

```javascript
var phongSphereGeometry = new THREE.SphereGeometry(8, 32, 16);
var phongSphereMaterial = new THREE.MeshPhongMaterial({color: 0x8888ff});
var phongSphere = new THREE.Mesh(phongSphereGeometry, phongSphereMaterial);
phongSphere.position.set(0, 10, 0);
scene.add(phongSphere);
```

刷新页面后可以观察到一个黑色的球体，这是因为我们还没添加光源。

首先创建一个白色的，位于(0, 100, 0)的点光源：

```javascript
var light = new THREE.PointLight(0xffffff);
light.position.set(0, 100, 0);
scene.add(light);
```

刷新页面可以看到球体上半部已经显现出蓝紫色的材质颜色。

然后我们再创建一个聚光灯，颜色为 0xffff00，位于(-60, 150, -30)处：

```javascript
var spotlight = new THREE.SpotLight(0xffff00, 2);
spotlight.position.set(-60, 150, -30);
scene.add(spotlight);
```

最终结果如下图所示： ![Screen Shot 2017-04-09 at 11.50.31 PM](https://cloud.githubusercontent.com/assets/6532225/25094184/8259ecee-23c8-11e7-872a-d6299f935f6c.png)

### 十一、运行官方样例

访问 car 目录下的 webgl_materials_cars.html 文件，或者访问 https://threejs.org/examples/#webgl_materials_cars。

阅读源代码，理解这个样例是如何搭建这个场景的。

大部分代码都是场景配置代码，大家可以从 574 行和 604 行的 load 函数入手，理解场景的构建过程。

### 十二、继续学习 three.js

选择第一个和第二个 PJ 选题的小组需要继续学习 three.js 来实现 PJ 效果，推荐同学们以 three.js 官网为参考，首先浏览官方 examples ，查看 examples 的源代码，碰到问题时查阅 documentation。

> 相关链接：
>
> examples: https://threejs.org/examples/
>
> documentation: https://threejs.org/docs/index.html

## Part 2: XML

接下去，我们使用 XML 与 XML Schema 定义 car 样例场景的配置信息。Lab 中为简单起见，仅通过 XML 来控制官方样例中车辆的一种材质。

> 关于 XML 相关的知识，同学们可以查看戴老师的 PPT，或者通过 w3school 学习：https://www.w3schools.com/xml/default.asp

### 一、使用 XML 定义官方样例中车辆的材质

查看 `car.xml` 文件，我们定义了车辆的一种材质：

```xml
<?xml version="1.0"?>
<materials
        xmlns="advanced-web-lab1"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="advanced-web-lab1 car.xsd">
    <material>
        <name>Pure Blue</name>
        <renderType>MeshPhongMaterial</renderType>
        <params>
            <color>0000ff</color>
            <specular>0000ff</specular>
            <shininess>60</shininess>
            <combine>THREE.MultiplyOperation</combine>
        </params>
    </material>
</materials>
```

材质的名称为 `Light Blue` ，渲染方式为 `MeshPhongMaterial` ，然后是材质的一些具体配置。

### 二、使用 XSD 来定义 XML 的 Schema

XSD 是 XML Schema Definition 的意思。XSD 被用来描述 XML 文件的结构。

> 参考网站：https://www.w3schools.com/xml/schema_intro.asp

查看 `car.xsd` 文件，这个文件描述了 `car.xml` 文件的结构。

```xml
<?xml version="1.0"?>
<xs:schema
        xmlns:xs="http://www.w3.org/2001/XMLSchema"
        targetNamespace="advanced-web-lab1"
        xmlns="advanced-web-lab1"
        elementFormDefault="qualified">
    <xs:element name="materials">
        <xs:complexType>
            <xs:sequence>
                <xs:element name="material">
                    <xs:complexType>
                        <xs:all>
                            <xs:element name="name" type="xs:string"/>
                            <xs:element name="renderType" type="xs:string"/>
                            <xs:element name="params">
                                <xs:complexType>
                                    <xs:all>
                                        <xs:element name="color" type="xs:hexBinary"/>
                                        <xs:element name="specular" type="xs:hexBinary"/>
                                        <xs:element name="shininess" type="xs:integer"/>
                                        <xs:element name="combine" type="xs:string"/>
                                    </xs:all>
                                </xs:complexType>
                            </xs:element>
                        </xs:all>
                    </xs:complexType>
                </xs:element>
            </xs:sequence>
        </xs:complexType>
    </xs:element>
</xs:schema>
```

现在的 `car.xml` 是符合 `car.xsd` 描述的标准的。尝试修改 `car.xml` 里的标签名称，如果同学们使用的是 IntelliJ 的话，可以看到报错信息 `Element xxxxxx is not allowed here` 。尝试作其他 xsd 中文件没有描述的修改，IntelliJ 也会对应地报错。通过使用 `car.xsd` ，我们精确地定义了 `car.xml` 的结构，保证我们在编写时不会出现不标准的代码。

举例来说， xsd 文件中的 materials 里是 `xs:sequence` 标签，这代表着我们可以在 materials 里定义多个 material，而 params 标签里是 `<xs:all>` 标签，所以我们不能定义多个 color 元素或者 specular 元素。同学们可以尝试自己再新添加一种 Material。

### 三、在网页中加载 XML 的配置信息

接下去我们在官方样例源码中添加代码，让其加载我们的 XML 文档，并添加为车辆的一种材质。

#### Step 1. 导入 XML 文件数据

首先使用 `XMLHttpRequest` 来导入 `car.xml` 中的数据，将下面的代码插入到 582 行 `window.addEventListener('resize', onWindowResize, false);` 的下方：

```javascript
var xhr = new XMLHttpRequest();
xhr.open('get', 'car.xml', true);
xhr.send(null);
xhr.onload = function () {
	console.log(xhr.responseXML);
};
```

在浏览器中刷新页面，可以看到 console 中打印了 `car.xml` 的内容。

#### Step 2. 获取 XML 中的数据并加载到配置代码中

我们可以像查找 HTML DOM 树一样来获取 XML 中的数据，在 onload 的回调函数中插入下面的代码：

```javascript
var materials = xhr.responseXML.getElementsByTagName("material");
for (var i = 0; i < materials.length; i++) {
    var materialConfig = materials[i];
    var name = materialConfig.getElementsByTagName("name")[0].innerHTML;
    var params = materialConfig.getElementsByTagName("params")[0];
    var color = params.getElementsByTagName("color")[0].innerHTML;
    var specular = params.getElementsByTagName("specular")[0].innerHTML;
    var shininess = params.getElementsByTagName("shininess")[0].innerHTML;
    var combine = null;
    switch (params.getElementsByTagName("combine")[0].innerHTML) {
        case 'THREE.MixOperation':
            combine = THREE.MixOperation;
            break;
        case 'THREE.MultiplyOperation':
            combine = THREE.MultiplyOperation;
            break;
        default:
            combine = '';
    }
    var renderType = null;
    var material = null;
    switch (materialConfig.getElementsByTagName("renderType")[0].innerHTML) {
        case 'MeshPhongMaterial':
            material = new THREE.MeshPhongMaterial({
                color: parseInt(color, 16),
                specular: parseInt(specular, 16),
                shininess: parseInt(shininess),
                envMap: textureCube,
                combine: combine
            });
            break;
        case 'MeshLambertMaterial':
            material = new THREE.MeshLambertMaterial({
                color: parseInt(color, 16),
                specular: parseInt(specular, 16),
                shininess: parseInt(shininess),
                envMap: textureCube,
                combine: combine
            });
            break;
    }
    CARS['veyron'].materials.body.push([name, material]);
}
```

这段代码通过 `getElementsByTagName` 方法获取了各个元素的值，并加入到 `Bugatti Veyron` 这辆车的材质列表中。刷新页面可以看到材质列表的最后多了个 `Pure Blue` ，点击查看我们新添加的材质的效果，如下图： ![Screen Shot 2017-04-11 at 1.56.33 AM](https://cloud.githubusercontent.com/assets/6532225/25094185/825b8a04-23c8-11e7-8979-d79241fabc89.png)

> three.js 对 JSON 的支持非常好，推荐同学们使用 JSON 进行配置，使用 XML 配置 three.js 场景会比较麻烦。做 PJ 时可以仅将 XML 存储于后端，前后端交互推荐使用 JSON。Lab 是为了结合课程内容加入了 XML 板块的教学，不代表前端应该像这样写代码。
