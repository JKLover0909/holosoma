# Holosoma

Holosoma (tiếng Hy Lạp: "whole-body") là một khung công tác toàn diện cho robot hình người, dùng để huấn luyện và triển khai các chính sách học tăng cường (reinforcement learning) trên robot hình người, cũng như chuyển đổi chuyển động (motion retargeting). Dự án hỗ trợ các nhiệm vụ di chuyển (theo dõi vận tốc) và theo dõi toàn thân trên nhiều bộ mô phỏng khác nhau (IsaacGym, IsaacSim, MJWarp, MuJoCo) với các thuật toán như PPO và FastSAC.

## Tính năng

- **Hỗ trợ đa bộ mô phỏng**: IsaacGym, IsaacSim, MuJoCo Warp (MJWarp), và MuJoCo (chỉ cho inference)
- **Nhiều thuật toán RL**: PPO và FastSAC
- **Hỗ trợ robot**: Robot hình người Unitree G1 và Booster T1
- **Loại nhiệm vụ**: Di chuyển (theo dõi vận tốc) và theo dõi toàn thân
- **Triển khai từ mô phỏng sang mô phỏng / mô phỏng sang thực tế**: Dòng suy luận chung cho cả mô phỏng và điều khiển robot thực tế
- **Chuyển đổi chuyển động (Motion retargeting)**: Chuyển dữ liệu bắt chuyển động của con người sang chuyển động robot đồng thời bảo toàn tương tác với các vật thể và địa hình
- **Tích hợp Wandb**: Ghi video, tự động tải checkpoint ONNX lên Wandb, và tải checkpoint trực tiếp từ Wandb

## Cấu trúc kho

```
src/
├── holosoma/              # Khung huấn luyện lõi (di chuyển & theo dõi toàn thân)
├── holosoma_inference/    # Pipeline suy luận và triển khai
└── holosoma_retargeting/  # Chuyển đổi chuyển động từ dữ liệu con người sang robot
```

## Tài liệu

- **[Hướng dẫn huấn luyện](src/holosoma/README.md)** - Huấn luyện chính sách di chuyển và theo dõi toàn thân trong IsaacGym/IsaacSim
- **[Hướng dẫn suy luận & triển khai](src/holosoma_inference/README.md)** - Triển khai chính sách lên robot thực tế hoặc đánh giá trong mô phỏng MuJoCo
- **[Hướng dẫn Retargeting](src/holosoma_retargeting/holosoma_retargeting/README.md)** - Chuyển dữ liệu bắt chuyển động của con người thành chuyển động cho robot

## Bắt đầu nhanh

### Cài đặt

Chọn script cài đặt phù hợp với trường hợp sử dụng của bạn:

```bash
# Cho huấn luyện trên IsaacGym
bash scripts/setup_isaacgym.sh

# Cho huấn luyện trên IsaacSim
# Yêu cầu Ubuntu 22.04 trở lên do các phụ thuộc của IsaacSim
bash scripts/setup_isaacsim.sh

# Cho huấn luyện MJWarp và mô phỏng MuJoCo (inference) — conda
bash scripts/setup_mujoco.sh

# Cho huấn luyện MJWarp và mô phỏng MuJoCo (inference) — uv (thay thế)
bash scripts/setup_mujoco_via_uv.sh

# Cho inference/triển khai
bash scripts/setup_inference.sh

# Cho chuyển đổi chuyển động
bash scripts/setup_retargeting.sh
```

### Huấn luyện

Huấn luyện robot G1 với FastSAC trên IsaacGym:

```bash
source scripts/source_isaacgym_setup.sh
python src/holosoma/holosoma/train_agent.py \
		exp:g1-29dof-fast-sac \
		simulator:isaacgym \
		logger:wandb \
		--training.seed 1
```

> **Lưu ý:** Đối với máy chủ không có giao diện (headless), xem [hướng dẫn huấn luyện](src/holosoma/README.md#video-recording) để biết cấu hình ghi video.

Xem thêm ví dụ và các tùy chọn cấu hình trong [Hướng dẫn huấn luyện](src/holosoma/README.md).

### Demo nhanh

Chúng tôi cung cấp script chạy toàn bộ pipeline: (tải và xử lý dữ liệu cho LAFAN), retargeting, chuyển đổi dữ liệu, và huấn luyện chính sách theo dõi toàn thân.

```bash
# Chạy retargeting và huấn luyện chính sách theo dõi toàn thân sử dụng dữ liệu OMOMO
bash demo_scripts/demo_omomo_wb_tracking.sh

# Chạy retargeting và huấn luyện chính sách theo dõi toàn thân sử dụng dữ liệu LAFAN
bash demo_scripts/demo_lafan_wb_tracking.sh
```

### Triển khai & Đánh giá

Sau khi huấn luyện, triển khai chính sách của bạn:

- **Robot thực tế**: Xem [Real Robot Locomotion](src/holosoma_inference/docs/workflows/real-robot-locomotion.md) hoặc [Real Robot WBT](src/holosoma_inference/docs/workflows/real-robot-wbt.md)
- **Mô phỏng MuJoCo**: Xem [Sim-to-Sim Locomotion](src/holosoma_inference/docs/workflows/sim-to-sim-locomotion.md) hoặc [Sim-to-Sim WBT](src/holosoma_inference/docs/workflows/sim-to-sim-wbt.md)

Hoặc duyệt tất cả các tùy chọn triển khai trong [Hướng dẫn suy luận & triển khai](src/holosoma_inference/README.md).

### Video demo

Xem các triển khai thực tế của chính sách Holosoma *(nhấp vào ảnh thu nhỏ để phát)*

<table>
	<tr>
		<th>G1 Locomotion</th>
		<th>T1 Locomotion</th>
		<th>G1 Dancing</th>
	</tr>
	<tr>
		<td width="33%">
			<a href="https://youtu.be/YYMgj5BDIMI">
				<img src="https://img.youtube.com/vi/YYMgj5BDIMI/hqdefault.jpg" width="100%" alt="▶ G1 Locomotion">
			</a>
		</td>
		<td width="33%">
			<a href="https://youtu.be/Q6rNHJZ2a6Y">
				<img src="https://img.youtube.com/vi/Q6rNHJZ2a6Y/hqdefault.jpg" width="100%" alt="▶ T1 Locomotion">
			</a>
		</td>
		<td width="33%">
			<a href="https://youtu.be/ouPk69_eFfE">
				<img src="https://img.youtube.com/vi/ouPk69_eFfE/hqdefault.jpg" width="100%" alt="▶ G1 Dancing">
			</a>
		</td>
	</tr>
</table>


## Báo cáo vấn đề

Chúng tôi hoan nghênh phản hồi và các báo cáo vấn đề để cải thiện holosoma. Vui lòng sử dụng phần issues để:

- Báo lỗi và sự cố kỹ thuật
- Yêu cầu tính năng mới

## Hỗ trợ

Nếu bạn cần trợ giúp ngoài issues, hãy tham gia [discord server](https://discord.gg/TPupMvpqHc).

Sử dụng Discord để thảo luận các kế hoạch lớn hơn và các vấn đề phức tạp hơn.

## Bảo mật

Xem [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) để biết thêm thông tin.

## Trích dẫn

Nếu bạn sử dụng Holosoma trong nghiên cứu, vui lòng trích dẫn theo panel "Cite this repository" ở thanh bên phải của repo Github.

## Giấy phép

Dự án này được cấp phép theo Apache-2.0 License.

