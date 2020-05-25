the ros driver for aimibotV2 robot. including the robot diff and odometry driver, etc...


## 7.3 视觉建图
### 7.3.1 安装驱动与依赖
安装realsensor驱动： 
注意先不要插上realsensor设备！！！安装方式可参考官方。  
- 注册服务器的公钥:  
`sudo apt-key adv --keyserver keys.gnupg.net --recv-key F6E65AC044F831AC80A06380C8B3A55A6F3EFCDE || sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-key F6E65AC044F831AC80A06380C8B3A55A6F3EFCDE`
如果仍然无法检索公钥，请检查并指定代理设置: `export http_proxy="http://<proxy>:<port>"`  , 并重新运行该命令。 请参阅以下其他方法[link](https://unix.stackexchange.com/questions/361213/unable-to-add-gpg-key-with-apt-key-behind-a-proxy).  

- 将服务器添加到存储库列表中:  
  Ubuntu 16 LTS:  
`sudo add-apt-repository "deb http://realsense-hw-public.s3.amazonaws.com/Debian/apt-repo xenial main" -u`  
  Ubuntu 18 LTS:  
`sudo add-apt-repository "deb http://realsense-hw-public.s3.amazonaws.com/Debian/apt-repo bionic main" -u`

- 安装库（如果升级软件包，请参阅下面的部分）:  
  `sudo apt-get install librealsense2-dkms`  
  `sudo apt-get install librealsense2-utils`  
  以上两行将部署librealsense2 udev规则，构建并激活内核模块，运行时库以及可执行的演示和工具。

- （可选）安装开发人员和调试软件包:  
  `sudo apt-get install librealsense2-dev`  
  `sudo apt-get install librealsense2-dbg`  
  安装了`dev`软件包后，您可以使用`g ++ -std = c ++ 11 filename.cpp -lrealsense2`或您选择的IDE编译具有librealsense的应用程序。

重新连接英特尔实感深度摄像头并运行：`realsense-viewer`以验证安装。

验证内核是否已更新：    
`modinfo uvcvideo | grep "version:"` 应该会出现 `realsense` 字符串

安装realsensor ros功能包  
`sudo apt-get install ros-kinetic-realsense2-camera`

安装rtabmap ros功能包  
`sudo apt-get install ros-kinetic-rtabmap-ros`
### 7.3.2 确定realsensor的位置
默认是安装了支架的，位置与aimibotV2_description的urdf文件中描述的一致，可以通过  
`roslaunch aimibot2_description display.launch`  
来查看realsensor安装位置是否与实际安装位置一致。且坐标系应如图所示    
[![rostf.md.png](https://wx1.sbimg.cn/2020/05/25/rostf.md.png)](https://sbimg.cn/image/pbtjm)  
相机坐标系x轴与相机方向一致。若需要人为更改相机姿态，则需要自己设置aimibotV2_description的urdf描述文件。在aimibotV2_description/urdf/aimibot.urdf文件中找到如下设置：  
``` xml
<joint
  name="realsense_joint"
  type="fixed">
  <origin
    xyz="0.22476 4.8928E-05 0.5145 "
    rpy="0 0 0" />  <!--这里设置相机姿态  -->
  <parent
    link="base_footprint" />
  <child
    link="realsense_link" />
  <axis
    xyz="0 0 0" />
</joint>
```
rpy="0 0 0"处可以设置姿态，如果位置有变动，则需要以base_footprint底盘坐标系作为参考，来变化  
[![rostf2.th.png](https://wx2.sbimg.cn/2020/05/25/rostf2.th.png)](https://sbimg.cn/image/pbNdj)

### 7.3.3 建图
建图会同时通过视觉slam建立点云地图，以及激光slam建立栅格地图，导航最后通过视觉点云地图来定位，通过激光的栅格地图来导航，通过激光来避障。首先进行建图，建图的期间尽量保持环境是静态的，目前的rtabmap无法剔除动态点云。建图之后保存地图文件，然后依赖地图文件进行定位与导航。需要注意的是，如果环境的视觉特征过于少（例如全是白色的墙壁，相机拍摄的画面全是白色），则会导致视觉建图实效。  
**建图**  
`roslaunch aimibot rtabmap_local.launch rviz:=true`   
使用手柄控制机器人建图，建好的地图保存在~/.ros中rtabmap.db。将rtabmap.db文件放入~/map中 自己命名好名字  
**定位与导航**  
`roslaunch aimibot rtabmap_local.launch rviz:=true localization:=true database_path:=~/map/yourmapname.db`
载入后机器人会自动定位，然后可以设定目标点，机器人会导航到目标点。




----
## 给开发者的话 

该分支以2020.3.21的主分支独立。  
* 添加了一些文件  
* 改动的地方对rtabmap建图无影响。视情况合并分支即可。  
* aimibotV2_description的urdf文件更改对rtabmap建图是有必要的，否则会造成建立的地图畸形。

5月25号更新
---
* 改动了aimibotV2_description的aimibot.urdf，改变了坐标系依赖关系。
* 增加技术文档。



