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

`FOLLOWER_PORT`와 `LEADER_PORT`를 확인해서 아래 명령의 포트에 넣습니다.

## 2) 캘리브레이션
### follower arm
```bash
lerobot-calibrate \
    --robot.type=so101_follower \
    --robot.port=<FOLLOWER_PORT> \
    --robot.id=my_awesome_follower_arm
```

### leader arm
```bash
lerobot-calibrate \
    --teleop.type=so101_leader \
    --teleop.port=<LEADER_PORT> \
    --teleop.id=my_awesome_leader_arm
```

## 3) 카메라 경로 찾기
### 장치 확인
```bash
lerobot-find-cameras opencv
```

### 경로 확인 (`by-path` 권장)
```bash
ls -l /dev/v4l/by-path/
```

`TOP_CAM_PATH`, `WRIST_CAM_PATH`를 정합니다.

예시:
- `/dev/v4l/by-path/pci-0000:00:14.0-usb-0:10:1.0-video-index0`
- `/dev/v4l/by-path/pci-0000:00:14.0-usb-0:9.2:1.0-video-index0`

## 4) 텔레옵 (카메라 없음)
```bash
lerobot-teleoperate \
    --robot.type=so101_follower \
    --robot.port=<FOLLOWER_PORT> \
    --robot.id=my_awesome_follower_arm \
    --teleop.type=so101_leader \
    --teleop.port=<LEADER_PORT> \
    --teleop.id=my_awesome_leader_arm
```

## 5) 텔레옵 (카메라 2개)
```bash
lerobot-teleoperate \
    --robot.type=so101_follower \
    --robot.port=<FOLLOWER_PORT> \
    --robot.id=my_awesome_follower_arm \
    --robot.cameras="{
      top: {type: opencv, index_or_path: <TOP_CAM_PATH>, width: 640, height: 480, fps: 30},
      wrist: {type: opencv, index_or_path: <WRIST_CAM_PATH>, width: 640, height: 480, fps: 30}
    }" \
    --teleop.type=so101_leader \
    --teleop.port=<LEADER_PORT> \
    --teleop.id=my_awesome_leader_arm \
    --display_data=true
```

## 6) 텔레옵 (wrist 카메라만)
```bash
lerobot-teleoperate \
    --robot.type=so101_follower \
    --robot.port=<FOLLOWER_PORT> \
    --robot.id=my_awesome_follower_arm \
    --robot.cameras="{
      wrist: {type: opencv, index_or_path: <WRIST_CAM_PATH>, width: 640, height: 480, fps: 30}
    }" \
    --teleop.type=so101_leader \
    --teleop.port=<LEADER_PORT> \
    --teleop.id=my_awesome_leader_arm \
    --display_data=true
```

## 7) 자주 겪는 문제
- `Mismatch between calibration values...`:
  연결된 장치와 포트/ID 조합이 다를 때 자주 발생. 포트 매핑부터 다시 확인.
- `read failed (status=False)`:
  카메라 동시 스트림 실패. 허브 분산 연결, `fps: 15`로 하향, 카메라 단독 테스트로 확인.
- `gRPC transport error`:
  `display_data` 시각화 쪽 로그성 경고인 경우가 많고, 텔레옵이 실제 동작하면 치명적이지 않은 경우가 많음.
