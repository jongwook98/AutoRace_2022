# ROS

ROS – launch
remap from AA to BB : 노드에서 사용 중인 ROS 변수 요소의 이름을 변경한다 AA → BB

ROS - nodes

rospy.spin() 노드가 종료될 때까지 노드가 종료되지 않도록 한다.
rospy.on_shutdown(??) 종료신호를 받고 종료되기 전 ?? 함수를 실행한다.

- rospy.Subscriber
    
    __ init __(self, name, data_class, callback=None, callback_args=None, queue_size=None, buff_size=65536, tcp_nodelay=False)
    

- rospy.Subscriber(“/토픽”, 메시지 종류, calback 함수)
    
    callback 함수 -> 메시지 변수를 각각 대입
    

- rospy.Publisher
__ init __(self, name, data_class, subscriber_listener=None, tcp_nodelay=False, latch=False, headers=None, queue_size=None)

- ROS – msgs
--sensor_msgs--
-Image-
    
    
    std_msgs/Header header -> uint32 seq, time stamp, string frame_id
    uint32 height
    uint32 width
    
    string encoding
    uint8 is_bigendian
    uint32 step
    uint8[] data
    

-CompressedImage-
std_msgs/Header header
string format
uint8[] data