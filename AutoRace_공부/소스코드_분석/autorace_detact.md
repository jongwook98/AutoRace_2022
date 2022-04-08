# autorace_detect

## detect_lane

HSV param 값 yaml 파일에서 변경

### launch

lane.yaml에서 rosparam 값을 가져온다.
detect/image_input을 camer/image_projected_compensated로 받아옴.

→ extrinsic 카메라 launch 파일에서 변환된 값

### Init

get_param으로 lane.yaml 파일에서 가져온다
sub, pub 이미지 타입을 정할 수 있다 raw, compressed
calibration_mode : action, calibration 두 가지 모드가 있는데 calibration 모드는 자동으로 param 값을 조정한다

★ **cbFindLane** 에게 카메라 데이터 값을 넘겨준다

publisher 들을 만든다

- cbFindLane

중앙으로 향하게 하는 조향값(pub_lane), 조향라인 이미지(pub_image_lane)

- maskYellowLane, maskWhiteLane

하얀선(pub_image_white_lane), 신뢰성(pub_white_line_reliability)

노란선(pub_image_yellow_lane), 신뢰성(pub_yellow_line_reliability) 

### cbFindLane

1/3 으로 실행

maskWhiteLane, maskYellowLane 함수를 불러와서 각각의 검출 크기, inrange 된 이미지를 받아옴

검출된 영역의 크기로부터 try: fit_from_lines 함수, except: sliding_windown 함수를 실행

(left, right)fit, (left, right)fitx, mov_avg_(left, right) 값을 만든다

mov_avg_(left, right)는 최근 5 개의 이동방향인듯?

이미지와 white fraction, yellow fraction 인자를 통해서 make_lane을 실행한다.

### make(White, Yellow)Lane

영상에 대한 처리 inRange, bitwise_and 를 하고 HSV 색 영역의 크기를 계산한다.

신뢰성에 대한 값은 inRange 처리된 이미지의 데이터에서 높이를 기준으로 0이 아닌 값이 많으면 신뢰도가 높아진다. → 해당 높이 이미지에서 inRange 범위의 값이 없다면 how_much_short 증가

영상 처리된 하얀선, 노란선 이미지를 publish하고, 검출된 영역의 크기, 영상처리된 이미지를 return

### fit_from_lines

처음에는 실행하지 않다가 except 의 sliding_windwon 함수에서 fitx, fit 값을 초기화 하고 나서 실행 되는듯

### sliding_windown

histogram : 받아온 높이를 반으로 나눈 inRange 단색 이미지를 세로축으로 더하여 각 세로축에서 감지된 픽셀 수 직선 배열로 나타냄

out_img : 이미지를 그릴 공간 만듦, midpoint : 영상의 중앙값

lane_base : 왼쪽 차선(노란선), 오른쪽 차선(하얀선) 에서 가장 많이 검출된 세로축을 기준으로 만듦

good_lane_inds : ??

lane_fit, lane_fitx : ??

### make_lane

warp_zero : 이미지의 크기 만큼의 0 배열을 만든다

color_warp, color_warp_lines : 3채널의 컬러 0 배열을 만든다

ploty : 0 부터 영상의 세로 길이 만큼의 배열을 만든다

검출된 영역의 크기, 색 영역에 따른 각각의 신뢰도에 따라서 선을 만드는 방법이 달라진다.

centrex.item(350)