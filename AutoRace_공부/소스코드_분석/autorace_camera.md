# autorace_camera

- turtlebot3_autorace_camera
    
    intrinsic - ROS 의 카메라 동작을 실행하는 Launch 파일
    
    extrinsic - 카메라 값을 image_compensation, image_projection 의 nodes로 처리하여 재맵핑 함
    
    - intrinsic_camera_calibration
        
        
        remap from “in” to “camera/rgb/image_raw”
        
        remap from “out” to “camera/image”
        
        remap from “image_raw” to “image”
        
    
    - extrinsic_camera_calibration