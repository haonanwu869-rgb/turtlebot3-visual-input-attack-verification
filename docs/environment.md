# 环境与验收约定（待全组确认）

本分支为 Wu Haonan 原协作骨架的整合提案，尚不代表组长发布或全组通过。技术路线遵循小组仓库 README：Ubuntu 24.04、ROS 2 Jazzy、Gazebo Harmonic、TurtleBot3 Waffle Pi。尚未完成安装、编译或运行验收。

## 统一目标

| 项目 | 目标 / 状态 |
| --- | --- |
| 操作系统 | Ubuntu 24.04 |
| ROS | ROS 2 Jazzy |
| 仿真器 | Gazebo Harmonic |
| 机器人 | TurtleBot3 Waffle Pi |
| 感知 | Python / OpenCV，HSV 阈值在干净场景标定后固定 |
| 官方依赖 | turtlebot3、turtlebot3_msgs、turtlebot3_simulations 的分支和完整 commit 均待实测确认 |
| 工作空间 | 建议官方依赖放 ~/turtlebot3_ws，自编包放独立 ~/attack_ws；实际 overlay 顺序随验收记录固定 |

不要把旧文档中的 Humble、gazebo11、gazebo_ros 安装命令直接替换版本名后执行。当前不提供未经验证的 Jazzy/Harmonic 安装和启动命令；首个环境 PR 应补齐可复现步骤、依赖完整 commit、启动命令及输出，再由全组确认。

## 环境登记建议

以下编号来自旧协作方案，待组员确认。每位成员只填写自己的真实验收结果。

| 成员 | 文件标识 | 建议 ROS_DOMAIN_ID | GitHub 账号 |
| --- | --- | ---: | --- |
| Zhang Shuyu | zhang-shuyu | 31 | 待本人确认 |
| Hou Meiqi | hou-meiqi | 32 | 待本人确认 |
| Wu Haonan | wu-haonan | 33 | 待本人确认 |
| Yang Mingzhi | yang-mingzhi | 34 | 待本人确认 |
| Li Qingchen | li-qingchen | 35 | 待本人确认 |

同一实验涉及的节点需使用相同 ROS_DOMAIN_ID，独立成员实验使用不同编号。计算资源参考 README：多核 CPU、16 GB 内存，独立 GPU 可选；具体虚拟机兼容性待实测。

## 个人验收

按 [记录要求](environment-checks/README.md) 提交系统与依赖版本、编译输出、实际启动命令，并验证：

- Gazebo 中能够加载 Waffle Pi。
- 相机有画面、LiDAR 有数据，并记录实际话题名、消息类型、频率和 QoS。
- 在独立环境验收中验证键盘控制；正式实验关闭 teleop，确保可信控制器是唯一速度发布者。
- 记录里程计、仿真时间和 use_sim_time 设置。
- 检查相机与 LiDAR 的坐标系、重叠视野、时间戳与同步容差。
- 提交截图或关键输出、失败项和已尝试操作；缺少证据时不标记通过。

/scan、/camera/image_raw、/cmd_vel、/odom 仅作旧方案的话题候选，实际名称和类型以运行输出为准。

## 历史方案

[旧 Humble 环境文档](legacy/environment-humble-original.md) 仅用于追溯原始贡献，不是本仓库的安装指南。该文档的“组长发布”“必须一致”等表述仅属于原稿，不代表本组已作出相应决定。
