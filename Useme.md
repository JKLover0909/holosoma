# Holosoma G1 29-DOF Quick Use

Ghi chú nhanh để train policy locomotion cho Unitree G1 29-DOF bằng MJWarp, export ONNX, rồi chạy sim-to-sim trong MuJoCo.

## 1. Train Policy

Kích hoạt môi trường MuJoCo/MJWarp:

```bash
unset CONDA_ENV_NAME
unset LD_LIBRARY_PATH
source scripts/source_mujoco_setup.sh
```

Train không dùng W&B:

```bash
python src/holosoma/holosoma/train_agent.py \
  exp:g1-29dof \
  simulator:mjwarp \
  logger:disabled
```

Hoặc train với W&B offline:

```bash
python src/holosoma/holosoma/train_agent.py \
  exp:g1-29dof \
  simulator:mjwarp \
  logger:wandb-offline
```

Checkpoint và ONNX được lưu trong:

```text
logs/hv-g1-manager/<timestamp>-g1_29dof_manager-locomotion/
```

Tìm ONNX mới nhất:

```bash
find logs/hv-g1-manager -name '*.onnx' | sort | tail
```

## 2. Chạy MuJoCo Simulator

Mở terminal thứ nhất:

```bash
unset CONDA_ENV_NAME
unset LD_LIBRARY_PATH
source scripts/source_mujoco_setup.sh

python src/holosoma/holosoma/run_sim.py robot:g1-29dof \
  --simulator.config.bridge.enabled=True \
  --simulator.config.bridge.interface eno1
```

Nếu máy đang dùng Wi-Fi thay vì LAN, đổi `eno1` thành `wlp131s0` ở cả terminal simulator và terminal policy.

## 3. Chạy ONNX Policy

Mở terminal thứ hai:

```bash
unset CONDA_ENV_NAME
unset LD_LIBRARY_PATH
source scripts/source_inference_setup.sh

python3 src/holosoma_inference/holosoma_inference/run_policy.py inference:g1-29dof-loco \
  --task.model-path logs/hv-g1-manager/20260527_090254-g1_29dof_manager-locomotion/model_24999.onnx \
  --task.no-use-joystick \
  --task.interface eno1
```

Thay `--task.model-path` bằng file `.onnx` bạn muốn chạy.

## 4. Trình Tự Điều Khiển

Trong cửa sổ MuJoCo:

| Phím | Tác dụng |
| --- | --- |
| `8` | Hạ robot xuống đất |
| `9` | Bỏ gantry/giá treo |
| `Backspace` | Reset simulation |

Trong terminal chạy `run_policy.py`:

| Phím | Tác dụng |
| --- | --- |
| `]` | Start policy |
| `o` | Stop policy |
| `i` | Đưa robot về default pose |
| `=` | Chuyển standing/walking mode |
| `w` | Tiến |
| `s` | Lùi |
| `a` | Sang trái |
| `d` | Sang phải |
| `q` | Xoay trái |
| `e` | Xoay phải |

## 5. Lưu Ý Quan Trọng

- Luôn `unset LD_LIBRARY_PATH` trước khi source môi trường MuJoCo hoặc inference để tránh conflict thư viện DDS/IsaacSim.
- Bridge simulator và policy phải dùng cùng interface, ví dụ cùng là `eno1` hoặc cùng là `wlp131s0`.
- Không dùng placeholder kiểu `<path-to-model>.onnx` trong shell. Hãy thay bằng path thật và bỏ dấu `< >`.
- Nếu `source scripts/source_inference_setup.sh` báo thiếu env `hsinference`, chạy:

```bash
bash scripts/setup_inference.sh
```

## 6. Deploy Lên Robot Thật

Khi deploy lên robot thật, không chạy `run_sim.py`. Chỉ chạy `run_policy.py`; policy sẽ giao tiếp trực tiếp với robot qua Ethernet.

```text
Robot thật <--> run_policy.py
```

### Chuẩn Bị Unitree G1

- Treo robot lên gantry khi test lần đầu.
- Bật robot và tay điều khiển.
- Cắm Ethernet từ robot vào laptop.
- Đưa robot về damping mode.
- Nhấn `L2 + R2` trên controller để vào development mode.

### Cấu Hình Mạng

Interface laptop nối với G1 cần cấu hình:

```text
IP Address: 192.168.123.224
Netmask:    255.255.255.0
```

Tìm interface đang nối robot:

```bash
ip -br addr
```

Nếu cắm LAN, thường dùng `eno1`. Nếu tên interface khác, thay `eno1` trong command bên dưới.

### Chạy Policy Trên Robot Thật

Kích hoạt inference env:

```bash
unset CONDA_ENV_NAME
unset LD_LIBRARY_PATH
source scripts/source_inference_setup.sh
```

Chạy bằng joystick:

```bash
python3 src/holosoma_inference/holosoma_inference/run_policy.py inference:g1-29dof-loco \
  --task.model-path logs/hv-g1-manager/20260527_090254-g1_29dof_manager-locomotion/model_24999.onnx \
  --task.use-joystick \
  --task.interface eno1
```

Chạy bằng bàn phím trong terminal:

```bash
python3 src/holosoma_inference/holosoma_inference/run_policy.py inference:g1-29dof-loco \
  --task.model-path logs/hv-g1-manager/20260527_090254-g1_29dof_manager-locomotion/model_24999.onnx \
  --task.no-use-joystick \
  --task.interface eno1
```

### Phím Điều Khiển Robot Thật

Joystick:

| Nút | Tác dụng |
| --- | --- |
| `A` | Start policy |
| `B` | Stop policy |
| `Y` | Đưa robot về default pose |
| `Start` | Chuyển standing/walking mode |
| `L1 + R1` | Kill controller |
| Left stick | Đi tới/lùi/trái/phải |
| Right stick | Xoay trái/phải |

Keyboard trong terminal policy:

| Phím | Tác dụng |
| --- | --- |
| `]` | Start policy |
| `o` | Stop policy |
| `i` | Đưa robot về default pose |
| `=` | Chuyển standing/walking mode |
| `w` `a` `s` `d` | Đi tới/lùi/trái/phải |
| `q` `e` | Xoay trái/phải |

### An Toàn

- Luôn test policy mới khi robot đang treo gantry.
- Chỉ start policy khi robot đã vào đúng mode và network đã ổn định.
- Luôn sẵn sàng emergency stop hoặc `L1 + R1`.
- Không đứng gần robot khi chuyển sang walking mode.


## 7. Cấu Hình SSH Server + RTX 5060 Ti 16GB + No Display

Máy này chạy qua SSH, không có màn hình/X11. Ưu tiên chạy IsaacSim ở headless mode, tắt video, và không dùng lệnh preview GUI.

### Kiểm Tra GPU/Driver

```bash
nvidia-smi
```

Nếu không thấy RTX 5060 Ti hoặc CUDA driver báo lỗi, xử lý driver trước rồi mới setup IsaacSim.

### Setup IsaacSim

```bash
cd /home/jkl0909/Code/rl/holosoma
OMNI_KIT_ACCEPT_EULA=1 bash scripts/setup_isaacsim.sh
```

### Kích Hoạt Môi Trường

```bash
cd /home/jkl0909/Code/rl/holosoma
unset CONDA_ENV_NAME
unset LD_LIBRARY_PATH
unset DISPLAY
source scripts/source_isaacsim_setup.sh
```

`unset DISPLAY` giúp IsaacSim không cố bám vào display ảo/cũ khi chạy qua SSH.

### Train IsaacSim Headless

RTX 5060 Ti 16GB nên bắt đầu với `1024` hoặc `2048` env. Nếu VRAM còn dư thì tăng dần lên `4096`.

```bash
python src/holosoma/holosoma/train_agent.py \
  exp:g1-29dof-wbt \
  simulator:isaacsim \
  logger:disabled \
  --training.headless=True \
  --training.num-envs=1024 \
  --logger.video.enabled=False
```

Nếu muốn train locomotion thay vì WBT:

```bash
python src/holosoma/holosoma/train_agent.py \
  exp:g1-29dof \
  simulator:isaacsim \
  logger:disabled \
  --training.headless=True \
  --training.num-envs=1024 \
  --logger.video.enabled=False
```

### Chạy Qua SSH Bền Hơn

Dùng `tmux` để job không chết khi mất SSH:

```bash
tmux new -s holosoma
cd /home/jkl0909/Code/rl/holosoma
unset CONDA_ENV_NAME
unset LD_LIBRARY_PATH
unset DISPLAY
source scripts/source_isaacsim_setup.sh
python src/holosoma/holosoma/train_agent.py \
  exp:g1-29dof-wbt \
  simulator:isaacsim \
  logger:disabled \
  --training.headless=True \
  --training.num-envs=1024 \
  --logger.video.enabled=False
```

Thoát khỏi tmux nhưng giữ job chạy: `Ctrl-b`, rồi `d`.

Quay lại session:

```bash
tmux attach -t holosoma
```

### Tăng/Giảm Theo VRAM

- Nếu `nvidia-smi` báo gần hết VRAM hoặc bị OOM: giảm `--training.num-envs=512`.
- Nếu còn nhiều VRAM: thử `--training.num-envs=2048`, sau đó `4096`.
- Không chạy `--training.headless=False` trên SSH server no display, trừ khi đã cấu hình X server/VNC/VirtualGL.
- Không dùng `demo_scripts/train_g1_armhold_isaacsim.sh` trong repo hiện tại vì file này chưa tồn tại.
