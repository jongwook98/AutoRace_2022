# autorace_core

## autorace_core.launch

각 미션에 대한 core_mode_controller, core_mode_decider 두 개의 노드를 실행

> **mission_mode_decider**
> 

공통된 decider node에서 CurrentMode에 대해서 Enum()을 저장해 놓는데, traffic_mode_decider의 경우 다음과 같이 저장한다. 

```python
self.CurrrentMode = Enum('CurrentMode', 'idle lane_following traffic_light')
```

위 코드는 CurrentMode 에 대해서 함수형 타입으로 정의하여 사용하는 것이다.

```python
>>> list(CurrentMode)
[<CurrentMode.idle: 1>, <CurrentMode.lane_following: 2>, <CurrentMode.traffic_light: 3>]
```

current_mode = self.CurrentMode.idle,value ⇒ 이것은 1 이다...

> **traffic_light_core_node_decider**
> 

/detect/traffic_light 값 (1) (green_cout ≥ 3일 때) 을 cbInvokedByTrafficLight 함수에 넘겨줌

traffic_light_core_node_controller에서 /core/returned_mode 값을 cbReturnedMode 함수로 넘겨주고 → fnInitMode함수 실행 → current_mode 를 CurrentMode.lane_following.value 값을 받아옴

---

### cbInvokedByTrafficLight

subscriber 된 값을 가져오며 연결된 publisher에서는 traffic light 가 초록색이면 감지, 1을 받아옴

fnDecideMode 함수를 호출하며 (1, 1) 값을 전달 → fnDecideMode 에서 current_mode를 traffic_light.value (2)를 대입하며, fnPublishMode()를 통해서 상태변화를 publish 함

### fnDecideMode

이 함수는 current_mode를 lane_following → traffic_light, traffic_light → lane_following 으로 바꾸어주고, fnPublishMode() 함

### cbReturnedMode

이 함수는 아마 미션이 종료될 때 실행되는 것 같다. fnInitMode() 근데.. pub, sub은 만들어 놨지만 publish 하는 부분이 없음

### fnInitMode

처음 decider이 실행될 때, (미션이 종료될 때 → 이것은 확실하지 않음 pub, sub을 만들긴 하지만 publish 하지 않음! ) 시행되며, current_mode = 1 에 대입하고, fnPublishMode 실행, 1 publish 함

> **traffic_light_core_node_controller**
> 

current_mode ⇒ 2 가 된것을 cbReceiveMode 통해서 가져오며 이 node를 통해서 다음 미션에 대한 decider, controller 를 실행할 것 같다..

### **fnControlNode**

이 함수는 cbReceiveMode(self, mode_msg)에서 값을 받아오면 실행된다.

→ cbRecevieMode에서 값을 받아오고, while not rospy.is_shutdown(): 구조에 의해 fnControlNode()실행

### cbReceiveMode

decider fnInitMode 에 의해 실행되어 triggerd = True 시킨다.

current_mode 를 받아온 값으로 변경시킨다.

> **core_node_mission**
> 

reset_proxy = rospy.Serviceproxy(’gazebo/reset_simulation’, Empty) → gazebo/reset_simulation을 service 한다..?