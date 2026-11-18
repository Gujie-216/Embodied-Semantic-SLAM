# 第 1 周：跑通 RGB-D SLAM 基线

本周交付：由你独立完成环境准备、数据回放、轨迹评估和复跑。先用原版 ORB-SLAM3 在一段公开 RGB-D 数据上输出相机轨迹，把它与数据集真值轨迹对齐，记录位置误差和轨迹覆盖。把原版基线记为 G0，之后用同一套流程检验动态观测筛选和轻量化有没有帮助。

## 今天先做两件事

1. **确定运行机器。** 先向孙超老师确认是否能使用实验室的 Ubuntu 电脑。当前 Windows 电脑未检测到已安装的 WSL 发行版，C 盘检查时约剩 10 GiB；有实验室机器就先用它。若只用本机，先按 [Microsoft 的 WSL 指南](https://learn.microsoft.com/zh-cn/windows/wsl/install) 检查安装条件和磁盘位置。相机借用等需要实景采集时再确认。
2. **下载第一段数据。** 在 [TUM RGB-D 官方下载页](https://cvg.cit.tum.de/data/datasets/rgbd-dataset/download) 找到 **fr1/xyz**，下载 **tgz**，解压后应包含 `rgb/`、`depth/`、`rgb.txt`、`depth.txt` 和 `groundtruth.txt`。这段约 30 秒，专用于查通路。原版 ORB-SLAM3 已提供它的 RGB 与深度时间戳配对表。

这一阶段用公开数据离线回放，不需要先购置设备，也不需要接入 ROS、SAM 2 或 CLIP。

## 在 Ubuntu 上完成第一次运行

按 [ORB-SLAM3 官方说明](https://github.com/UZ-SLAMLab/ORB_SLAM3#2-prerequisites) 准备 C++ 编译环境、OpenCV、Eigen 和 Pangolin；该示例会打开 Pangolin 窗口，首跑优先选有桌面的 Ubuntu 环境。把源码和数据放在用户目录下；下列命令假定数据已经解压到 `~/slam-data/rgbd_dataset_freiburg1_xyz`。

```bash
git clone https://github.com/UZ-SLAMLab/ORB_SLAM3.git ~/ORB_SLAM3
cd ~/ORB_SLAM3
git rev-parse HEAD
bash build.sh
```

成功编译后，在独立目录运行。自己复跑时，把下面的运行编号从 `run01` 改为 `run02`，避免覆盖首跑的轨迹：

```bash
orb_dir="$HOME/ORB_SLAM3"
sequence="$HOME/slam-data/rgbd_dataset_freiburg1_xyz"
run_id="fr1_xyz-g0-run01"
mkdir -p "$HOME/slam-runs/$run_id"
cd "$HOME/slam-runs/$run_id"
set -o pipefail
"$orb_dir/Examples/RGB-D/rgbd_tum" "$orb_dir/Vocabulary/ORBvoc.txt" \
  "$orb_dir/Examples/RGB-D/TUM1.yaml" "$sequence" \
  "$orb_dir/Examples/RGB-D/associations/fr1_xyz.txt" 2>&1 | tee run.log
test -s CameraTrajectory.txt
```

示例会导出 `CameraTrajectory.txt` 和 `KeyFrameTrajectory.txt`。评估覆盖时使用前者；关键帧文件只记部分帧。运行失败时先保留 `run.log`，核查依赖、相机配置和数据路径，一次只解决一个问题。

按 [evo 官方说明](https://michaelgrupp.github.io/evo/#installation--upgrade) 在独立 Python 环境安装 evo，安装后在同一运行目录执行：

```bash
evo_ape tum "$sequence/groundtruth.txt" CameraTrajectory.txt \
  --align --save_plot trajectory.png --save_results ape.zip
wc -l "$orb_dir/Examples/RGB-D/associations/fr1_xyz.txt" CameraTrajectory.txt
```

记录输出的 RMSE（米）。RGB-D 有尺度信息，只对齐旋转和平移；不启用尺度校正。首跑同时记录输入配对帧数、导出相机位姿数及两者比例；这是排查用的**轨迹覆盖率**，正式跟踪完整率后续以逐帧跟踪状态日志核对。不能只凭误差变小判定算法改进。

## 个人任务与本周验收

1. **第 1–2 天：准备。** 确认运行机器，下载并解压 fr1/xyz，编译原版 ORB-SLAM3，记录环境和代码版本。
2. **第 3–4 天：跑通并评估。** 用 fr1/xyz 导出相机轨迹，再用 evo 生成轨迹图，填写位置误差和轨迹覆盖率。出错时先保留日志，解决后再试。
3. **第 5–7 天：独立复跑。** 按自己留下的命令和配置，在新目录再跑一次同一序列。比较两次的误差、覆盖率与异常，并记录差异。

首跑和复跑都完成后，再换用 **fr3/sitting_xyz** 和 **fr3/walking_xyz** 作为开发数据，并更换对应的相机配置和配对表。沿用相同评价方法记录误差与覆盖；如果人员穿行附近出现异常，记下对应时间。正式测试序列先留出，不用于调参。现场走廊照片先用作布置参考，等能借到相机后再采集同步的 RGB 与深度序列，设计“通道中央→靠墙”的物体搬移对照。

本周至少收齐：代码版本、配置与配对表、两次运行日志、相机轨迹、评估图和填写好的 [实验记录](实验记录.md)。自己能再次走完轨迹导出和评估流程后，再新增第一项功能：用人员掩膜过滤潜在动态特征，与原版作对照。
