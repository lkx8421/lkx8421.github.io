---
title: 一份基于solidworks导出urdf文件应用于webots仿真的说明文档
date: 2025-02-05 15:06:02
tags: webots
categories: 机器人仿真
description: 这是一份将solidworks模型导出为urdf文件，并应用于webots仿真环境的说明文档
---
# 使用环境

系统：Window 11

建模软件：solidworks 2021

仿真软件：matlab R2022a 以及 webots R2023a

参考文档：

# 一、通过solidworks导出URDF文件配置

## 1、建模软件插件安装

默认已经安装好solidworks，需要在solidworks上安装导出urdf文件的插件，ROS目前提供了SolidWorks转URDF的插件，叫做sw\_urdf\_exporter。官网下载地址：<http://wiki.ros.org/sw_urdf_exporter>。根据对应的Solidworks版本下载双击安装即可（安装前关闭solidworks，如果你的Solidworks很新没有对应版本默认选择最新版本的插件来安装）。

![](图片-13.png)

安装好SolidWorks后，再安装这个插件，重新打开SW，点击菜单栏上方：工具-插件中会发现里面多了一个叫做SW2URDF的插件，勾选启用此插件。&#x20;

![](图片-11.png)

接着在上方菜单栏：工具-Tools-Export as URDF中对装配体配置urdf文件

![](图片-12.png)

## 2、URDF文件导出前配置坐标系信息：

在导出URDF文件前需要配置坐标系。目前我们机器人基于[MDH参数法](https://blog.csdn.net/inghoG/article/details/117881500)来建立机器人模型，对于现有的A1.1模型，对于左腿，建立坐标系情况如下：

![](图片-10.png)

![](图片-9.png)



坐标系0表示body坐标系，建立在两个电机轴线连线的中心上，以上方向为z轴，朝向前脸的方向为x轴，电机轴线从右电机到左电机的连接方向为y轴。

杆件的z轴方向均为其关节轴线方向朝外，x轴方向为从当前关节指向下一个关节。

对于关节1 2 4 5 6 其坐标系xoy平面均在同一平面，如下所示均建立在杆1的外表面上，对于坐标系3其xoy平面则建立在轮子两个表面的中间，这样做是保证在受力分析时轮子的支持力均在xoy平面内便于分析。

![](图片.png)

对于右腿由于镜像关系如下所示，不再赘述。

![](图片-1.png)

在建立好坐标系后，则可以开始后续的urdf文件导出操作

## 3、导出URDF文件

> 在导出URDF文件前，务必将各个杆件，关节的质量属性设置正确

在solidworks上方菜单栏中：工具-Tools-Export as URDF对模型导出URDF文件。

![](图片-2.png)

此界面是配置基座的相关信息：

1为定义基座名称，此处为base\_link

2为定义基座坐标，这里未配置，需配置为**坐标系0**

3处选择基座模型，在solidworks工作界面选择对应对应零件则高亮出

4为child links，表示在这个link下有几个子link，对应于此图有2个，分别为left\_motor和right\_motor。

5为此URDF结构，表示一个机器人的杆件之间的关系，是一个树形结构。

![](图片-3.png)

此界面是配置childlink的相关信息：

1为定义childlink名称，此处为**left\_leg1**

2为定义关节名称，必须定义否则导出前报错，此处命名为**left\_j1**

3为定义chlidlink坐标，这里配置为**坐标系1**，表示left\_leg1是以此坐标系为参考坐标系

4处为参考轴，表示旋转或平移副的旋转或平移的方向，这里选择**基准轴1**，表示绕基准轴1来进行旋转。

5处定义关节类型，continuous表示旋转副，无关节限制，而revolute也表示旋转副，但有角度限制，其中也有移动副prismatic和固定副fixed，此处选择continuous

下表为[A1.1模型](https://ehooj45utb.feishu.cn/file/boxcnoTqaWbTdOlckbDksZhYyQh)在建模时的坐标系，转轴和关节配置选择。

|            |              |             |             |              | **joint\_name** | **Reference Coordinate System** | **Reference Axis** | **Joint Type** |
| ---------- | ------------ | ----------- | ----------- | ------------ | --------------- | ------------------------------- | ------------------ | -------------- |
| base\_link |              |             |             |              | \\              | 坐标系0                            | \\                 | \\             |
|            | left\_motor  |             |             |              | left\_j4        | 坐标系4                            | 基准轴1               | **continuous** |
|            |              | left\_leg1  |             |              | left\_j1        | 坐标系1                            | 基准轴1               | **continuous** |
|            |              |             | left\_leg2  |              | left\_j2        | 坐标系2                            | 基准轴2               | **continuous** |
|            |              |             |             | left\_wheel  | left\_j3        | 坐标系3                            | 基准轴4               | **continuous** |
|            |              | left\_leg3  |             |              | left\_j5        | 坐标系5                            | 基准轴3               | **continuous** |
|            | right\_motor |             |             |              | right\_j4       | 坐标系14                           | 基准轴1               | **continuous** |
|            |              | right\_leg1 |             |              | right\_j1       | 坐标系11                           | 基准轴1               | **continuous** |
|            |              |             | right\_leg2 |              | right\_j2       | 坐标系12                           | 基准轴2               | **continuous** |
|            |              |             |             | right\_wheel | right\_j3       | 坐标系13                           | 基准轴4               | **continuous** |
|            |              | right\_leg3 |             |              | right\_j5       | 坐标系15                           | 基准轴3               | **continuous** |



由于urdf无法生成并联结构，后续在proto文件进行修改。因此此处建立每个关节之间的连接关系，如下图所示：

![](图片-4.png)

在配置好坐标系，零件以及关节副等相关信息后，在URDF Exporter界面点击Preview and Export...按钮来导出URDF文件，如下界面：

![](图片-5.png)

主要检查导出前的Joint Type是否正确，Coordinates和Axis是否如设置一样，Axis是否有数值（0，0，1或者0，0，-1，**如果0，0，-1改为0，0，1**）。检查无误后点击右下角Next。

![](图片-6.png)

此处来配置杆件的相关质量信息，一般需要提前在solidworks中设置好质量属性后，此处页面不用做修改。确认完毕后点击Export URDF and Meshes，选择导出路径以及导出文件名字diablo\_robot后，点击**保存**即导出完毕。导出文件结构如下：

![](图片-7.png)

meshes文件夹中包括各个模型的STL文件，urdf中则为导出的.urdf文件

![](图片-8.png)

# 二、检查导出urdf文件是否正确

## 1、在线urdf可视化

https://gkjohnson.github.io/urdf-loaders/javascript/example/bundle/

![](image.png)

将整个导出的文件夹拖拽到此网站，查看关节关系是否正确，如果转不动选择

![](image-1.png)



## ~~1、通过matlab中的simscape来检查模型是否正确（非必须）（删除）~~

为了快速检查下建立的urdf文件是否正确，通过matlab导入urdf模型的方式来检查，首先打开matlab，将导出的diablo\_robot添加到路径，如下所示

![](图片-14.png)

转到urdf所在文件目录，输入

```matlab
smimport("diablo_robot.urdf")
```

即可弹出simlink界面，点击运行弹出模型

![](图片-15.png)

此处就是根据urdf在matlab中生成的模型，可以看到各个关节由于重力作用下是按照我们所配置的坐标系来进行旋转的。检查发现**左腿**的方向错误，因此需要修改，分析是由于从body到左腿的motor的坐标转换的方向不对所导致的（在图形显示界面点击对应零件可显示对应零件的坐标系，检查此处是因为绕x轴的旋转角度错误）。

打开diablo\_robot.urdf文件，找到对应left\_j4，如下所示

```xml
  </link>
  <joint
    name="left_j4"
    type="continuous">
    <origin
      xyz="0 0.18755 0"
      rpy="1.5708 1.5708 0" />
    <parent
      link="base_link" />
    <child
      link="left_motor" />
    <axis
      xyz="0 0 -1" />
  </joint>
```

将其中的rpy中的第一项修改，表示绕x旋转角度，改为如下所示

```xml
  </link>
  <joint
    name="left_j4"
    type="continuous">
    <origin
      xyz="0 0.18755 0"
      rpy="-1.5708 1.5708 0" />
    <parent
      link="base_link" />
    <child
      link="left_motor" />
    <axis
      xyz="0 0 -1" />
  </joint>
```

再打开matlab导入urdf模型，检查看问题解决\~

![](图片-16.png)

# 三、新建一个Webots工程

## 1、webots安装

首先介绍webots的安装，直接在[官网下载](https://www.cyberbotics.com/)。点击后弹出页面向下拉选择webots-R2023.setup.exe，下载后直接双击安装，选择对应安装路径即可。

![](图片-17.png)

![](图片-18.png)

## 2、新建一个webots工程

（1）双击打开安装好的webots，在Tools-Preferences...中设置界面语言为简体中文。点击ok后重启软件。

![](图片-19.png)

![](图片-20.png)

（2）点击"文件-New-新项目目录..."，创建新项目目录

![](图片-21.png)

选择新项目目录所在文件夹

![](图片-22.png)

设置世界的名称，以及初始化时所带的特征，**全选**，点击下一步。

![](图片-23.png)

点击完成，等待一下即可弹出一个地板，（鼠标左键放在地板上按住拖动鼠标即可改变视角方向）。

![](图片-24.png)

![](图片-25.png)

* 注意要保存（Ctrl+Shift+S），默认建立工程会启动仿真，点击上方的暂停按钮来暂停仿真

  ![](图片-26.png)

## 3、将生成的urdf文件导入在webots工程文件夹下

首先先将通过solidworks生成了一个名为**diablo\_robot**的文件夹，里面包括了mesh文件和urdf文件。将整个**diablo\_robot**文件夹剪切到新建webots工程目录my\_project下，如下所示，为后续生成proto文件做铺垫。

![](图片-27.png)

# 四、生成用于webots仿真中的proto模型

## 1、安装urdf2webots



（1）安装python：https://blog.csdn.net/weixin\_49237144/article/details/122915089

（2）urdf2webots安装：

```bash
# webots版本>R2023b
pip install urdf2webots==2023.1.0 # 参照官网教程：https://github.com/cyberbotics/urdf2webots
#如果webots版本为R2023a
pip install urdf2webots==2.0.3
```

默认webots版本为R2023a，如果版本不相同，下载下方源码，解压后使用下列命令安装

```c++
pip install --upgrade --editable urdf2webots
```

[urdf2webots.zip](files/urdf2webots.zip)



## 2、生成proto文件

打开webtos工程下urdf文件所在目录.\diablo\_robot\urdf，右键点击在终端中输入

```bash
python -m urdf2webots.importer --input=diablo_robot.urdf
```

![](图片-28.png)

即可在对应文件夹内生成proto文件

![](图片-29.png)

# 五、proto文件修改

官网教程：https://cyberbotics.com/doc/reference/proto-design-guidelines?version=R2023a

可以通过记事本打开proto文件，不过在VScode中有对应的proto文件查看插件，可以实现语法高亮。

![](图片-30.png)

## 1、修改mesh文件路径

在windows环境下导出proto文件中，mesh文件路径（也就是对应的STL文件）会存在错误，如下所示，将对应url修改为相对路径解决

```yaml
      Shape {
        appearance DEF base_link_material PBRAppearance {
          baseColor 0.627450 0.627450 0.627450
          roughness 1.000000
          metalness 0
        }
        geometry DEF base_link Mesh {
          url "E:\Projects\Webots\diablo_A1\diablo_robot/diablo_robot2/meshes/base_link.STL"
        }
      }
```

改为：

```yaml
url "../diablo_robot/diablo_robot2/meshes/base_link.STL"
```

对于其他杆件的url以同样的方式进行处理

## 2、设置并联属性

在webots中，通过设置SolidReference属性来设置并联关系，在本案例中，对于left\_leg3而言是通过转动副（HingeJoint）与left\_leg2相连接的。铰接的位置相对于left\_leg3的坐标系用anchor表示，方向用axis表示。在本案例中axis为0 0 1，anchor为0.14.

因此设置SolidReference中solidName 为 left\_leg2，表示铰接在left\_leg2上。

```yaml
HingeJoint {
    jointParameters HingeJointParameters {
        axis 0.000000 0.000000 1.000000
        anchor 0.140000 0.000000 0.00000
        }
endPoint SolidReference {
    solidName "left_leg2"
    }        
} 
```

编辑好上述信息后，打开DiabloRobot.proto，将其复制在endPoint Solid的Shape节点为left\_leg3的同级节点下，如下图所示

![](图片-31.png)

对于right\_leg3也同样处理，只不过需要注意在SolidName中改为 right\_leg2。

## 3、设置关节

由于默认导出urdf文件时，除了会赋予Joint的位置信息（JointParameters），还会赋予关节的device信息，device一般包括电机（Motor）和位置传感器（PositionSensor），如下图所示

![](图片-32.png)

对于我们的模型而言，只有关节left\_j1、left\_j4、left\_j3和right\_j1、right\_j4、right\_j3是**主动关节**，其他的关节属于**被动关节**。因此需要对于我们的关节信息做一定修改，即将对应device信息中的Motor信息删除或者注释掉，如下所示：

![](图片-33.png)

通过这种方式来设置出我们需要的**主动关节**。

## 4、配置传感器

```bash
    GPS {
    }
    Gyro {
    }
    InertialUnit {
      name "imu"
    }
    Accelerometer {
      name "accl"
    }
```



### （1）配置altimeter

在Robot的children节点下插入

```yaml
    Altimeter {
      translation 0 0 0
      rotation 0 0 1 0 #定义altimeter相对机器人位置及姿态
      name "altimeter" #定义altimeter名字
    }
```

### （2）配置accelerometer

在Robot的children节点下插入

```yaml
    Accelerometer {
      translation 0 0 0
      rotation 0 0 1 0 #定义altimeter相对机器人位置及姿态
      name "accelerometer" #定义accelerometer名字
    }
```

### （3）配置inetial unit

在Robot的children节点下插入

```yaml
    InertialUnit {
      translation 0 0 0
      rotation 0 0 1 0 #定义inetial unit相对机器人位置及姿态
      name "inetial unit" #定义inetial unit名字
    }
```

## 5、proto导入在webots中

将编辑好的proto文件复制在我们创建好的webots工程文件夹中的/protos中，双击打开/worlds中我们创建好的世界new\_world，按照1，2，3，4的顺序依次点击，即将我们的proto文件导入在了webots环境中。

![](图片-34.png)

如果在webots中的命令行中出现这个，表示我们的proto文件的存在不合理字符，打开文件则发现设置的路径有问题

![](图片-35.png)

接着导入模型到我们的仿真界面，在**暂停**仿真状态下，依次点击1，2，3即可看到模型导入在了环境中

![](图片-36.png)

![](图片-37.png)

如果导入进来啥都没有，多半是由于设置的mesh路径错误，修改.proto文件中的url即可。

## 6\*、修改bounding属性（非必须，待完善）

通过urdf导出的proto文件默认bounding属性默认是将三维软件模型直接mesh来作为边界，如下所示，白色的线就是这个机器人的接触边界。但是这种接触边界很复杂，会降低我们的仿真求解速度，因此需要一点的简化。

![](图片-38.png)

```yaml
boundingObject USE right_wheel #默认在proto文件中，bounding为对应杆件三维模型的mesh形状
```

（1）修改轮子

将right\_wheel修改为圆柱体，如下所示：

```yaml
boundingObject Cylinder{
    height 0.052
    radius 0.0935
    subdivision 32 # in range [16,1000] 
}
```

修改后，点击菜单栏刷新，查看diablo\_robot

![]()

![](图片-40.png)

可以看到接触边界已经修改，调整subdivision来改变圆柱面精度，这里设置为32。

（2）修改杆件

对于较为复杂的杆件结构，我们通过建立group以及transform，例如，对于right\_leg1，其bounding属性修改为：

```yaml
boundingObject Group {
    children [
        Transform {
            translation 0 0 0.015
            children [
                Cylinder {
                    height 0.016
                    radius 0.0305
                }
            ]
        }
        Transform {
            translation 0 0 0
#            rotation 0 0 1 0
            children [
                Box {
                    size 0.12 0.020 0.016
                }
            ]
        }
    ]
}
```
