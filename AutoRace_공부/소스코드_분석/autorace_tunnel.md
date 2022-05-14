# autorace_tunnel

## detect_tunnel

sub (/detect/tunnel_order (tunnel_core_mode_controller)) → cbTunnelOrder

## tunnel_core_node_controller

core/decided_mode에서 값을 받으면 cbReceiveMode에 의해 is_triggered = True → 

current_mode 변환

uuid = roslaunch.rlutil.get_or_generate_uuid(None, False) → 이 값으로 

detect/tunnel_order → tunnel_core_mode_controller에서 받아옴

한 사이클 실행 후 is_triggered = False로 변환 → 다음 is_triggered = Ture로 변환 하면 다음 플래그 실행

> **fnControlNode**
> 

current mode가 CurrentMode.tunnel.value(2) 일 때 터널 주행에 대한 값 찾는 듯

터널의 step은 표지판 찾기, 진입, 네비게이션, 터널 탈출, 종료 순이며 터널 표지판을 찾고 나선 진입 step이기 때문에 current mode가 tunnel 일 때 바로 진입으로 바꿔준다.

각 step에 대해서 동작하는 것을 fnLaunch를 이용하여 정해준다.

1. lane_following

camera, detect_lane, detect_sign, control_lane : True

1. searching_tunnel_sign

detect_tunnel, control_moving : True

1. go_in_to_tunnel

detect_tunnel, control_tunnel : True

1. navigation

camera, detect_tunnel, control_moving : True

1. go_out_from_tunnel

camera, detect_lane, detect_sign, control_lane : True

1. exit

camera, detect_lane, detect_sign, control_lane : True

> **extrinsic_camera_calibraion**
> 

> **detect_sign**
> 

> **detect_lane**
> 

> **detect_tunnel**
> 

> **control_lane**
> 

> **control_tunnel**
> 

> **control_moving**
> 

> **fnLaunch**
> 

각 launch를 실행하게 해주며 roslaunch.scriptapi.ROSLaunch()로 실행파일로 만들며, roslaunch.parent.ROSLaunchParent(uuid, path) launch 파일 저장,

camera_ex_calib ⇒ camera/launch/extrinsic_camera_calibration

detect_sign ⇒ detect/launch/detect_sign

detect_lane ⇒ detect/launch/detect_lane

detect_tunnel ⇒ detect/launch/detect_tunnel

control_lane ⇒ driving/launch/turtlebot3_autorace_control_lane

control_tunnel ⇒ driving/launch/turtlebot3_autorace_control_tunnel

control_moving ⇒ driving/launch/turtlebot3_autorace_control_moving