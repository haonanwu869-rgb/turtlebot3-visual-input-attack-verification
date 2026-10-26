> 历史原稿，仅用于追溯；不是当前安装指南。以下关于组长发布、版本强制和截止时间的表述未被本次小组采纳。当前路线见 ../environment.md。

# TurtleBot3 Waffle Pi 仿真环境统一规范

DASE7508 小组项目 · 组长发布 · 所有成员必须按本文搭建，不要自行更换版本。

> 命令以 ROBOTIS e-Manual（https://emanual.robotis.com，选 Humble）和 ROS 2 官方文档为准。如果两者与本文冲突，先在群里说明，再统一修改本文，不要各自处理。

## 1. 统一版本（必须一致）

| 项目 | 统一选择 |
| --- | --- |
| 操作系统 | Ubuntu 22.04 LTS（64 位） |
| ROS | ROS 2 Humble |
| 仿真器 | Gazebo Classic（gazebo11，随 `ros-humble-gazebo-ros-pkgs` 安装） |
| 机器人型号 | `waffle_pi`（只有它带摄像头） |
| ROBOTIS 仓库分支 | `humble`（不要用 `humble-devel`、`main`、`jazzy`） |
| 编译工具 | colcon |
| Python | 系统自带 3.10，不要另装其他版本覆盖 |
| 工作空间 | `~/turtlebot3_ws` |
| 固定提交（commit） | 组长填写：turtlebot3 `____` / turtlebot3_msgs `____` / turtlebot3_simulations `____` |

三个 ROBOTIS 仓库必须同一分支，不得混用。

## 2. 机器要求与非 Ubuntu 成员的方案

建议配置：4 核以上 CPU，8 GB 以上内存，40 GB 以上空闲硬盘。

- **Ubuntu 22.04 原生或双系统**：最优先。
- **Windows**：用 VMware / VirtualBox 装 Ubuntu 22.04 虚拟机（分配 4 核、8 GB 内存、50 GB 硬盘）；WSL2 的图形界面可能不稳定，遇到问题请换虚拟机。
- **macOS（Apple 芯片）**：装 ARM 版 Ubuntu 22.04 虚拟机；Gazebo 可能较慢，先试，不行请告知组长。
- 无法满足的成员请尽早告知，由组长安排（例如共用一台机器跑仿真）。

## 3. 安装步骤

### 3.1 安装 ROS 2 Humble

严格按官方文档完成（设置 locale、添加软件源、安装）： [https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debians.html](https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debians.html)

```bash
sudo apt update
sudo apt install -y ros-humble-desktop
```

### 3.2 安装 Gazebo 接口与工具

```bash
sudo apt install -y ros-humble-gazebo-ros-pkgs \
  python3-colcon-common-extensions python3-rosdep python3-vcstool git \
  ros-humble-cv-bridge python3-opencv ros-humble-rqt-image-view \
  ros-humble-dynamixel-sdk
sudo rosdep init      # 如果提示已经初始化，可忽略
rosdep update
```

### 3.3 获取并编译 TurtleBot3（全部使用 humble 分支）

```bash
source /opt/ros/humble/setup.bash
mkdir -p ~/turtlebot3_ws/src && cd ~/turtlebot3_ws/src
git clone -b humble https://github.com/ROBOTIS-GIT/turtlebot3_msgs.git
git clone -b humble https://github.com/ROBOTIS-GIT/turtlebot3.git
git clone -b humble https://github.com/ROBOTIS-GIT/turtlebot3_simulations.git

cd ~/turtlebot3_ws
rosdep install --from-paths src --ignore-src -r -y
colcon build --symlink-install
```

编译完成后，记录三个仓库的 commit，填入第 1 节：

```bash
for r in turtlebot3_msgs turtlebot3 turtlebot3_simulations; do
  echo $r $(git -C ~/turtlebot3_ws/src/$r rev-parse --short HEAD)
done
```

### 3.4 环境变量（追加到 `~/.bashrc`）

```bash
source /opt/ros/humble/setup.bash
source ~/turtlebot3_ws/install/setup.bash
source /usr/share/gazebo/setup.sh
export TURTLEBOT3_MODEL=waffle_pi
export ROS_DOMAIN_ID=<你的编号>
```

`ROS_DOMAIN_ID` 约定：Zhang Shuyu=31，Hou Meiqi=32，Wu Haonan=33，Yang Mingzhi=34，Li Qingchen=35。同一局域网内这样可以避免互相干扰。 不要在 `.bashrc` 里 source 其他 ROS 版本，也不要自行修改 `RMW_IMPLEMENTATION`（统一使用默认设置）。

改完后执行 `source ~/.bashrc`，或重新打开终端。

## 4. 验收清单（每人必须通过并截图或贴输出到群里）

终端 1：启动仿真

```bash
ros2 launch turtlebot3_gazebo turtlebot3_world.launch.py
```

终端 2：检查

```bash
echo $TURTLEBOT3_MODEL         # 应输出 waffle_pi
ros2 topic list                # 记录相机、LiDAR、速度、里程计话题名
ros2 topic hz /scan            # LiDAR 有数据
ros2 topic hz /camera/image_raw   # 相机有数据（话题名以 topic list 为准）
ros2 run rqt_image_view rqt_image_view   # 能看到相机画面
ros2 run turtlebot3_teleop teleop_keyboard  # 键盘能让机器人移动
```

验收通过的标准：Gazebo 里能看到 Waffle Pi，相机画面正常，LiDAR 有数据，机器人能被控制。 话题名预期是 `/scan`、`/camera/image_raw`、`/cmd_vel`、`/odom`，但**以每个人 `ros2 topic list` 的实际输出为准**，如有差异请汇报，由组长统一写入配置。

## 5. 代码与 Git 约定

- 官方三个仓库只作为依赖使用，**不要直接修改**。我们的代码放在自己的 ROS 包和 GitHub 仓库里，使用单独的工作空间（如 `~/attack_ws`），叠加在 `turtlebot3_ws` 之上。
- 仓库目录：`src/`（ROS 包）、`worlds/`、`configs/`（攻击和实验参数）、`results/`（日志和数据）、`docs/`、`README.md`（复现说明）。
- `main` 分支只放能运行的版本；每人在 `feat/<姓名>-<主题>` 分支上开发，通过 Pull Request 合并，至少一名其他成员检查。
- 所有实验使用配置文件，固定随机种子，并保存日志，以便复现。
- 使用 AI 辅助编码或排错的部分，请在提交说明或文档里简要记录，报告里需要说明。

## 6. 常见问题

| 现象 | 处理 |
| --- | --- |
| Gazebo 首次启动很久、黑屏 | 首次可能需要下载模型，耐心等几分钟；仍然不行就重启 |
| Gazebo 崩溃或图形异常（虚拟机常见） | 尝试 `export LIBGL_ALWAYS_SOFTWARE=1` 后重启 |
| `ros2: command not found` | 没有 source，检查 `.bashrc` |
| `package 'turtlebot3_gazebo' not found` | 没有 source `~/turtlebot3_ws/install/setup.bash`，或编译失败 |
| 编译缺依赖 | 重新运行 `rosdep install ...`，仍失败请把完整报错发到群里 |
| 仿真残留进程 | `killall gzserver gzclient`，然后重新启动 |

遇到问题时请在群里贴：Ubuntu 版本、报错全文、你执行过的命令。

## 7. 成员环境登记表（每人填写后交给组长）

| 成员 | 系统/虚拟机 | ROS_DOMAIN_ID | 验收通过（是/否） | 备注 |
| --- | --- | --- | --- | --- |
| Zhang Shuyu |  | 31 |  |  |
| Hou Meiqi |  | 32 |  |  |
| Wu Haonan |  | 33 |  |  |
| Yang Mingzhi |  | 34 |  |  |
| Li Qingchen |  | 35 |  |  |

**截止时间：** 请在 Week 6 内完成并填写，完成后才能进入 Week 7 的攻击实现。
