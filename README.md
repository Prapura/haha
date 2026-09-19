# Ubuntu 20.04 下安装 ROS Noetic 与小海龟测试

> 作者：Prapura
> 虚拟机：VMware Workstation 17.5
> 操作系统：Ubuntu 20.04.6 LTS (Focal Fossa)
> ROS 版本：ROS Noetic Ninjemys（ros-noetic-desktop-full 1.5.0）

---

## 一、环境说明

| 项目 | 版本 |
|---|---|
| 操作系统 | Ubuntu 20.04.6 LTS (focal) |
| 架构 | x86_64 / amd64 |
| ROS 发行版 | Noetic Ninjemys |
| 安装方式 | apt（USTC 中科大镜像） |

ROS Noetic 官方仅支持 Ubuntu 20.04，二者版本正好匹配。

---

## 二、安装步骤

### 1. 配置 ROS apt 源

新建 `/etc/apt/sources.list.d/ros.list`，写入（使用中科大镜像，国内下载更快）：

```bash
sudo sh -c 'echo "deb [arch=amd64] https://mirrors.ustc.edu.cn/ros/ubuntu/ focal main" > /etc/apt/sources.list.d/ros.list'
```

### 2. 导入 ROS GPG Key

官方 key 托管在 `raw.githubusercontent.com`，国内直连会超时，改用 gitee 镜像下载：

```bash
curl -sSL -o /tmp/ros.key https://gitee.com/zhao-xuzuo/rosdistro/raw/master/ros.key
sudo apt-key add /tmp/ros.key
```

### 3. 更新并安装桌面完整版

```bash
sudo apt update
sudo apt install -y ros-noetic-desktop-full
```

`desktop-full` 包含 roscpp / rospy、RViz、rqt、Gazebo、turtlesim、cv_bridge 等常用组件。

### 4. 安装 rosdep 工具链

```bash
sudo apt install -y python3-rosdep python3-rosinstall python3-rosinstall-generator python3-wstool build-essential
```

### 5. 初始化 rosdep

```bash
sudo rosdep init
rosdep update
```

### 6. 配置环境变量

把 ROS 环境写入 `~/.bashrc`，新开终端自动生效：

```bash
echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

### 7. 验证安装

```bash
rosversion -d      # 应输出 noetic
roscore            # 能正常启动即成功
```

---

## 三、遇到的问题与解决办法

### 问题 1：多余的 ros2 源 404，导致 apt 直接失败

**现象**：`sudo apt install ros-noetic-desktop-full` 刚下载几个包就报
`E: 无法下载 .../ros2/ubuntu/... 404 Not Found`，退出码 100。

**原因**：系统里残留了一条失效的 ros2 focal 源
`deb [arch=amd64] https://mirrors.ustc.edu.cn/ros2/ubuntu/ focal main`，
该路径下的包已被镜像方移除。

**解决**：注释掉这条失效源，只保留 ROS1 源：

```bash
sudo sed -i 's|^deb .*ros2/ubuntu.*|#&|' /etc/apt/sources.list.d/ros-fish.list
sudo apt update
```

### 问题 2：raw.githubusercontent.com 被墙，key 下载与 rosdep update 超时

**现象**：
- 下载官方 `ros.asc` / `ros.key` 超时；
- `sudo rosdep init` 报 `The read operation timed out`；
- `rosdep update` 卡在
  `Query rosdistro index https://raw.githubusercontent.com/ros/rosdistro/master/index-v4.yaml`。

**原因**：ROS 官方 key 和 rosdep 数据源都托管在 GitHub raw，国内直连不通。

**解决**：统一改用 gitee 上的 rosdistro 镜像 `gitee.com/zhao-xuzuo/rosdistro`：

```bash
# 1) key 从 gitee 下载（见步骤 2）

# 2) rosdep init 之前，先把源码里的 GitHub URL 替换成 gitee
sudo sed -i 's|raw.githubusercontent.com/ros/rosdistro|gitee.com/zhao-xuzuo/rosdistro/raw|g' \
  /usr/lib/python3/dist-packages/rosdep2/sources_list.py \
  /usr/lib/python3/dist-packages/rosdep2/gbpdistro_support.py \
  /usr/lib/python3/dist-packages/rosdistro/__init__.py

# 3) 再执行
sudo rosdep init
rosdep update
```

替换后 `rosdep update` 正常输出 `Add distro "noetic"` 并完成缓存。

---

## 四、运行小海龟测试

新开三个终端：

```bash
# 终端 1：启动核心
roscore

# 终端 2：打开小海龟仿真窗口
rosrun turtlesim turtlesim_node

# 终端 3：键盘控制（方向键控制移动）
rosrun turtlesim turtle_teleop_key
```

把鼠标点到终端 3 的窗口上，按方向键即可控制小海龟移动。

---

## 五、仓库文件说明

| 文件 | 说明 |
|---|---|
| `README.md` | 本文件 |
| `assets/version.png` | 版本截图：Ubuntu 20.04 与 ROS Noetic 版本信息 |
| `assets/turtle-demo.mp4` | 20–30 秒录屏：键盘控制小海龟移动 |
