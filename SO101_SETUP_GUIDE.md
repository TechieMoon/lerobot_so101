# SO101 Setup Guide (Portable)

다른 컴퓨터로 옮겨도 그대로 따라할 수 있게, 포트/카메라 경로를 매번 확인해서 넣는 방식입니다.

## 0) 연결 원칙
- `follower arm`과 `leader arm`의 `/dev/ttyACM*` 번호는 PC/재부팅/USB 순서에 따라 바뀔 수 있습니다.
- 카메라는 동일 모델 2개면 `by-id`로 구분이 안 될 수 있으니 `by-path` 사용을 권장합니다.
- 카메라 경로는 `...video-index0`를 우선 사용합니다.

## 1) 팔 포트 찾기 (`/dev/ttyACM*`)
```bash
lerobot-find-port
```

`FOLLOWER_PORT`와 `LEADER_PORT`를 확인해서 아래 환경변수에 넣습니다.

```bash
export FOLLOWER_PORT=/dev/ttyACM0
export LEADER_PORT=/dev/ttyACM1
```

새 컴퓨터에 포트 접근 권한을 줘야 합니다.
```bash
sudo chmod 666 /dev/ttyACM0 /dev/ttyACM1
```

## 2) 캘리브레이션
### follower arm
```bash
lerobot-calibrate \
    --robot.type=so101_follower \
    --robot.port="$FOLLOWER_PORT" \
    --robot.id=my_awesome_follower_arm
```

### leader arm
```bash
lerobot-calibrate \
    --teleop.type=so101_leader \
    --teleop.port="$LEADER_PORT" \
    --teleop.id=my_awesome_leader_arm
```

## 3) 카메라 경로 찾기
### 장치 확인
```bash
lerobot-find-cameras opencv
```

### 카메라 켜보기
```bash
ffplay /dev/video0
```

### 새 컴퓨터에 비디오 장치 접근 권한을 줘야 합니다.
```bash
sudo chmod 666 /dev/video12 /dev/video14
```

### 경로 확인 (`by-path` 권장)
```bash
ls -l /dev/v4l/by-path/
```

`TOP_CAM_PATH`, `WRIST_CAM_PATH`를 정합니다.

예시:
- `/dev/v4l/by-path/pci-0000:00:14.0-usb-0:10:1.0-video-index0`
- `/dev/v4l/by-path/pci-0000:00:14.0-usb-0:9.2:1.0-video-index0`

```bash
export TOP_CAM_PATH=/dev/video14
export WRIST_CAM_PATH=/dev/video12
```

## 4) 텔레옵 (카메라 없음)
```bash
lerobot-teleoperate \
    --robot.type=so101_follower \
    --robot.port="$FOLLOWER_PORT" \
    --robot.id=my_awesome_follower_arm \
    --teleop.type=so101_leader \
    --teleop.port="$LEADER_PORT" \
    --teleop.id=my_awesome_leader_arm
```

## 5) 텔레옵 (카메라 2개)
```bash
lerobot-teleoperate \
    --robot.type=so101_follower \
    --robot.port="$FOLLOWER_PORT" \
    --robot.id=my_awesome_follower_arm \
    --robot.cameras="{
      top: {type: opencv, index_or_path: $TOP_CAM_PATH, width: 640, height: 480, fps: 30},
      wrist: {type: opencv, index_or_path: $WRIST_CAM_PATH, width: 640, height: 480, fps: 30}
    }" \
    --teleop.type=so101_leader \
    --teleop.port="$LEADER_PORT" \
    --teleop.id=my_awesome_leader_arm \
    --display_data=true
```

## 6) 텔레옵 (wrist 카메라만)
```bash
lerobot-teleoperate \
    --robot.type=so101_follower \
    --robot.port="$FOLLOWER_PORT" \
    --robot.id=my_awesome_follower_arm \
    --robot.cameras="{
      wrist: {type: opencv, index_or_path: $WRIST_CAM_PATH, width: 640, height: 480, fps: 30}
    }" \
    --teleop.type=so101_leader \
    --teleop.port="$LEADER_PORT" \
    --teleop.id=my_awesome_leader_arm \
    --display_data=true
```

## 7) 레코딩 (카메라 2개)
```bash
lerobot-record \
  --robot.type=so101_follower \
  --robot.port="$FOLLOWER_PORT" \
  --robot.id=my_awesome_follower_arm \
  --robot.cameras="{
    top: {type: opencv, index_or_path: $TOP_CAM_PATH, width: 640, height: 480, fps: 30},
    wrist: {type: opencv, index_or_path: $WRIST_CAM_PATH, width: 640, height: 480, fps: 30}
  }" \
  --teleop.type=so101_leader \
  --teleop.port="$LEADER_PORT" \
  --teleop.id=my_awesome_leader_arm \
  --display_data=true \
  --dataset.repo_id "${HF_USER}/record-test-1" \
  --dataset.num_episodes=5 \
  --dataset.single_task="Bring the pen into hole of the toilet paper"
```

# 레코딩 시간 조절:

```bash
lerobot-record \
  --robot.type=so101_follower \
  --robot.port="$FOLLOWER_PORT" \
  --robot.id=my_awesome_follower_arm \
  --robot.cameras="{
    top: {type: opencv, index_or_path: $TOP_CAM_PATH, width: 640, height: 480, fps: 30},
    wrist: {type: opencv, index_or_path: $WRIST_CAM_PATH, width: 640, height: 480, fps: 30}
  }" \
  --teleop.type=so101_leader \
  --teleop.port="$LEADER_PORT" \
  --teleop.id=my_awesome_leader_arm \
  --display_data=true \
  --dataset.repo_id "${HF_USER}/so-101" \
  --dataset.num_episodes=50 \
  --dataset.single_task="pick up the pen and put into the cup" \
  --dataset.episode_time_s=30 \
  --dataset.reset_time_s=7 \
  --dataset.push_to_hub=true 
```

### 데이터셋 저장하기

```bash
lerobot-record \
  --robot.type=so101_follower \
  --robot.port="$FOLLOWER_PORT" \
  --robot.id=my_awesome_follower_arm \
  --robot.cameras="{
    top: {type: opencv, index_or_path: $TOP_CAM_PATH, width: 640, height: 480, fps: 30},
    wrist: {type: opencv, index_or_path: $WRIST_CAM_PATH, width: 640, height: 480, fps: 30}
  }" \
  --teleop.type=so101_leader \
  --teleop.port="$LEADER_PORT" \
  --teleop.id=my_awesome_leader_arm \
  --display_data=true \
  --dataset.repo_id "${HF_USER}/so-101" \
  --dataset.num_episodes=50 \
  --dataset.single_task="pick up the pen and put into the cup" \
  --dataset.episode_time_s=30 \
  --dataset.reset_time_s=7 \
  --dataset.push_to_hub=true \
  --resume=true \
  --dataset.num_episodes=3 
```

--resume=true추가하고 --dataset.num_episodes에는 추가로 학습할 에피소드 개수를 적는다.(총 개수가 아니다.)

# 자동 업로드 비활성화

```bash
lerobot-record \
  --robot.type=so101_follower \
  --robot.port="$FOLLOWER_PORT" \
  --robot.id=my_awesome_follower_arm \
  --robot.cameras="{
    top: {type: opencv, index_or_path: $TOP_CAM_PATH, width: 640, height: 480, fps: 30},
    wrist: {type: opencv, index_or_path: $WRIST_CAM_PATH, width: 640, height: 480, fps: 30}
  }" \
  --teleop.type=so101_leader \
  --teleop.port="$LEADER_PORT" \
  --teleop.id=my_awesome_leader_arm \
  --display_data=true \
  --dataset.repo_id "${HF_USER}/record-test-1" \
  --dataset.num_episodes=5 \
  --dataset.single_task="pick up the pen and put into the cup" \
  --dataset.episode_time_s=15 \
  --dataset.reset_time_s=5 \
  --dataset.push_to_hub=false
```

# 로컬에 저장되어 있는 거 Huggingface에 직접 올리기

```bash
huggingface-cli upload ${HF_USER}/record-test ~/.cache/huggingface/lerobot/{repo-id} --repo-type dataset
```

레코딩 실패 시:
- `FileExistsError`: 같은 경로가 이미 있음. `record-test-2`처럼 새 이름 사용.
- 카메라 경로는 `by-path`의 `video-index0`를 사용.

## 8) 자주 겪는 문제
- `Mismatch between calibration values...`:
  연결된 장치와 포트/ID 조합이 다를 때 자주 발생. 포트 매핑부터 다시 확인.
- `read failed (status=False)`:
  카메라 동시 스트림 실패. 허브 분산 연결, `fps: 15`로 하향, 카메라 단독 테스트로 확인.
- `gRPC transport error`:
  `display_data` 시각화 쪽 로그성 경고인 경우가 많고, 텔레옵이 실제 동작하면 치명적이지 않은 경우가 많음.


# 데이터셋 보기
- 동영상으로 보기: https://huggingface.co/spaces/lerobot/visualize_dataset
- 수치로 보기: https://huggingface.co/TechieMoon

## Replay an episode
```bash
lerobot-replay \
    --robot.type=so101_follower \
    --robot.port="$FOLLOWER_PORT" \
    --robot.id=my_awesome_follower_arm \
    --dataset.repo_id=${HF_USER}/record-test-1 \
    --dataset.episode=0
```

# 데이터셋으로 학습하기

```bash
lerobot-train \
  --dataset.repo_id=${HF_USER}/so-101 \
  --policy.type=act \
  --output_dir=outputs/train/act_so101_test \
  --job_name=act_so101_test \
  --policy.device=cuda \
  --wandb.enable=true \
  --policy.repo_id=${HF_USER}/my_policy
```

## 이어서 학습하기(중간에 끊길 때)

```bash
lerobot-train \
  --config_path=outputs/train/act_so101_test/checkpoints/last/pretrained_model/train_config.json \
  --resume=true
```

# Upload policy checkpoints

## Once training is done, upload the latest checkpoint with:

```bash
huggingface-cli upload ${HF_USER}/act_so101_test \
  outputs/train/act_so101_test/checkpoints/last/pretrained_model
```

You can also upload intermediate checkpoints with:

```bash
huggingface-cli upload ${HF_USER}/act_so101_test${CKPT} \
  outputs/train/act_so101_test/checkpoints/${CKPT}/pretrained_model
```

# Run inference and evaluate your policy

You can use the record script from lerobot-record with a policy checkpoint as input, to run inference and evaluate your policy. For instance, run this command or API example to run inference and record 10 evaluation episodes:

```bash
lerobot-record \
  --robot.type=so101_follower \
  --robot.port="$FOLLOWER_PORT" \
  --robot.id=my_awesome_follower_arm \
  --robot.cameras="{
    top: {type: opencv, index_or_path: $TOP_CAM_PATH, width: 640, height: 480, fps: 30},
    wrist: {type: opencv, index_or_path: $WRIST_CAM_PATH, width: 640, height: 480, fps: 30}
  }" \
  --display_data=false \
  --dataset.repo_id=${HF_USER}/eval_so101 \
  --dataset.single_task="Put lego brick into the transparent box" \
  # <- Teleop optional if you want to teleoperate in between episodes \
  # --teleop.type=so101_leader \
  # --teleop.port="$LEADER_PORT" \
  # --teleop.id=my_awesome_leader_arm \
  --policy.path=${HF_USER}/my_policy
```