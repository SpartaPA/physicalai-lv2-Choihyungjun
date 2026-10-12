# 모듈 3 — 설정값에 따른 Mask와 검출 결과 변화 분석

## 코드 출처

| 구분 | 내용 |
|---|---|
| 저장소 | [chj1319/ROS-Vision-Tracker](https://github.com/chj1319/ROS-Vision-Tracker) (팀 프로젝트 1, Team Xmen) |
| 사용 노드 | `xmen_tracker` 패키지의 `tracker_node`(검출), `rviz_node`(Overlay) |

| 파일 | 변경 |
|---|---|
| `xmen_tracker/cube_tracker.py` | `detector.detect()`가 만든 `mask`를 `self.last_mask`에 보관 |
| `xmen_tracker/tracker_node.py` | `/tracking/mask`(`sensor_msgs/Image`, `mono8`) 발행 |

## 환경

| 구분 | 내용 |
|---|---|
| OS | Ubuntu 24.04.5 LTS (kernel 7.0.0-34-generic) |
| ROS 2 | Lyrical |
| Python | 3.12.3 |
| OpenCV | 4.6.0 |
| NumPy | 1.26.4 |

## 입력

**rosbag 재생** 2026-10-08 팀 프로젝트에서 RealSense D435로 녹화한 기록 사용

| 용도 | bag | 관찰 구간 (bag 시작 기준) |
|---|---|---|
| 실험 1·2 (목표 있음) | `p5_success` | 18.6 s부터 한 프레임씩 (stamp `1791422051.500585067`~) |
| 목표 없음 확인 | `p5_loss_return` | 5–8 s |

## 실행 방법

모든 터미널에서 먼저:

```bash
source /opt/ros/lyrical/setup.bash
source ~/git/ROS-Vision-Tracker/install/setup.bash
cd ~/git/physicalai-lv2-Choihyungjun/lv2_module3
```

1. 검출 노드 (`<설정>`은 아래 표의 yaml)
   ```bash
   ros2 run xmen_tracker tracker_node --ros-args \
     -p use_sim_time:=true -p detector_config:=<설정>
   ```

2. Overlay 노드 + RViz2 화면 (Image 디스플레이 `raw_Image`·`mask_Image` 추가)
   ```bash
   ros2 launch xmen_bringup rviz2_launch.py use_bag:=false preview_transport:=raw
   ```

3. 입력 재생 (멈춘 채 시작 → bag 터미널에서 → 키로 메시지 한 개씩)
   ```bash
   # 실험 1·2
   ros2 bag play ~/git/tp/ROS-Vision-Tracker/report/bags/p5_success --clock \
     --start-paused --start-offset 18.6 \
     --qos-profile-overrides-path ~/git/ROS-Vision-Tracker/xmen_bringup/param/bag_qos.yaml \
     --topics /camera/color/image_raw /camera/aligned_depth_to_color/image_raw /camera/color/camera_info

   # 목표 없음: bag을 p5_loss_return, --start-offset 5 로
   ```
   - `--clock` + `use_sim_time:=true`: bag 시각으로 프레임 나이 판단. 없으면 프레임이 모두 버려짐
   - `bag_qos.yaml`: `camera_info`를 transient_local로 재생해 `tracker_node`가 실제 초점거리를 받게 함
   - `--topics`: bag에 함께 기록된 당시 결과(`/target` 등)는 재생하지 않음

4. 수신 확인
   ```bash
   ros2 topic list -t
   ros2 topic info /tracking/mask --verbose
   ros2 topic echo /tracking/bbox
   ros2 topic echo /target
   ros2 topic echo /perception_status
   ```

## 설정값 변경·적용 방법

| 파일 | 원본과 다른 값 |
|---|---|
| [config/exp1_A_s200.yaml](config/exp1_A_s200.yaml) | 없음 (원본, `hsv_ranges` lower `[104, 200, 8]`) |
| [config/exp1_B_s100.yaml](config/exp1_B_s100.yaml) | `hsv_ranges` lower `[104, 100, 8]` |
| [config/exp2_A_minpx60.yaml](config/exp2_A_minpx60.yaml) | 없음 (원본, `depth.min_area_px: 60`) |
| [config/exp2_B_minpx300.yaml](config/exp2_B_minpx300.yaml) | `depth.min_area_px: 300` |

- `tracker_node`는 시작할 때 설정을 한 번만 읽음. 값을 바꾸면 **노드 재시작**(bag도 같은 명령으로 다시 시작해 같은 프레임부터 재생)
- 적용 확인: 시작 로그 `Detector config <경로>: HSV [...], ...` 줄. 각 결과 폴더의 `applied.txt`에 저장

## 결과 파일 위치

| 폴더 | 내용 |
|---|---|
| [results/exp1_A/](results/exp1_A/) | 실험 1-A (S 하한 200, 원본 설정): `exp1_A_raw.png`, `exp1_A_mask.png`, `exp1_A_tracking.png`, `echo_*.txt`, `applied.txt` |
| [results/exp1_B/](results/exp1_B/) | 실험 1-B (S 하한 100): 같은 구성 |
| [results/exp2_A/](results/exp2_A/) | 실험 2-A (min_area_px 60): 같은 구성 + 토픽 구성 증빙 `topic_list.txt`, `topic_info.txt` |
| [results/exp2_B/](results/exp2_B/) | 실험 2-B (min_area_px 300): 같은 구성 |
| [results/no_target/](results/no_target/) | 목표 없음 확인: 캡처, echo, `hz.txt` |
| [report.md](report.md) | 토픽 구성, 가설·통제 조건, A·B 비교, 코드 근거, 최종 선택값 |
