# autorace_driving

## autorace_control_lane.launch

control_lane node 실행, detect/lane 값을 받기 위해 remap

---

### controal_lane

sub_lane : ‘/control/lane’ == ‘/detect/lane’ 값을 받아오며 해당 값은 라인 추종값이다.

max_vel : ‘/control/max_vel’ 값을 받아오는데 어디서 받아오는지 못 찾음

pub_cmd_vel : cbFollowLane 함수를 통해서 계산함 → desired_center은 detect/lane의 목표 방향이며, x 축의 절반인 500에서 desired_center 값을 빼서 error PD 제어를 통해 모터에 가하는 힘을 계산한다.

Twist msg 타입을 이용하여 터틀봇의 이동방향을 정하며, Twist msg 타입은 3차원의 linear, angular 값이 저장되어 있으며, twist.linear.x, twist.angular.z 두 값만 사용하여 계산한다.

추측 : twist.linear.x 로 속도조절, twist.angular.z 로 각 조절을 하는 것 같음.

속도값은 이동방향이 화면으로부터 +-372 까지는 0.05 * Max_vel 로 조정되며 0.05를 곱하는 이유는 소스코드의 주기가 20Hz인것 같다. → 아닐 수도 있다.