# autorace_detect

## detect_lane

HSV param 값 yaml 파일에서 변경

---

### launch

lane.yaml에서 rosparam 값을 가져온다.
detect/image_input을 camer/image_projected_compensated로 받아옴.

→ extrinsic 카메라 launch 파일에서 변환된 값

---

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

---

### cbFindLane

1/3 으로 실행

maskWhiteLane, maskYellowLane 함수를 불러와서 각각의 검출 크기, inrange 된 이미지를 받아옴

검출된 영역의 크기로부터 try: fit_from_lines 함수, except: sliding_windown 함수를 실행

(left, right)fit, (left, right)fitx, mov_avg_(left, right) 값을 만든다

mov_avg_(left, right)는 최근 5 개의 이동방향인듯?

이미지와 white fraction, yellow fraction 인자를 통해서 make_lane을 실행한다.

---

### make(White, Yellow)Lane

영상에 대한 처리 inRange, bitwise_and 를 하고 HSV 색 영역의 크기를 계산한다.

신뢰성에 대한 값은 inRange 처리된 이미지의 데이터에서 높이를 기준으로 0이 아닌 값이 많으면 신뢰도가 높아진다. → 해당 높이 이미지에서 inRange 범위의 값이 없다면 how_much_short 증가

영상 처리된 하얀선, 노란선 이미지를 publish하고, 검출된 영역의 크기, 영상처리된 이미지를 return

---

### fit_from_lines

처음에는 실행하지 않다가 except 의 sliding_windwon 함수에서 fitx, fit 값을 초기화 하고 나서 실행하며 sliding_windown과 다르게 x 축의 좌표를 재조정하는 과정은 없고 이전의 곡선 방정식에서 값을 수정하는 방식이다.

---

### sliding_windown

왼쪽 차선과 오른쪽 차선 각각 이 함수를 실행한다.

histogram : 받아온 높이를 반으로 나눈 inRange 단색 이미지를 세로축으로 더하여 각 세로축에서 감지된 픽셀 수 직선 배열로 나타냄

out_img : 이미지를 그릴 공간 만듦, midpoint : 영상의 중앙값

lane_base : 왼쪽 차선(노란선), 오른쪽 차선(하얀선) 에서 가장 많이 검출된 세로축을 기준으로 만듦

nwindows = sliding_window 의 분해능 설정? 현재는 20

색이 검출된 x, y 좌표를 각각의 nonzeroy, nonzerox에 numpy.array 속성으로 저장해놓음

lane_base를 축으로 +- 50의 수직 영역의 ROI를 설정 → ROI의 x 축은 피드백으로 조정됨

good_lane_inds : 설정한 window 에서 색이 검출된 y 좌표의 배열을 (append)추가함 window의 분해능 만큼 반복시킨 만큼 감지된 픽셀의 y 좌표를 계속 추가하며 인덱스의 개수가 50이 넘어가는 순간 부터 window의 x 축을 인덱스의 x 좌표 평균으로 재조정함.

lane_fit : 이는 good_lane_inds 의 window의 x 축을 재조정하면서 얻은 픽셀들의 배열값을 합친 전체 배열이며 검출된 좌표로부터 얻어낸 직선의 2차 방정식의 y계수이다.

lane_fitx : y을 기준으로 2차 방정식의 x의 결과값들을 배열로 저장한 것

---

### make_lane

warp_zero : 이미지의 크기 만큼의 0 배열을 만든다

color_warp, color_warp_lines : 3채널의 컬러 0 배열을 만든다

ploty : 0 부터 영상의 세로 길이 만큼의 배열을 만든다

pts_left(right) : 2차 방정식의 x 결과값과 이미지 크기 만큼의 y 배열을 합치고 행과 열을 바꾸고, 위아래로 바꾼 값 → 화면으로 보여지는 값과 곡선의 위상을 맞추는 작업인 것 같다.

검출된 영역의 크기, 색 영역에 따른 각각의 신뢰도에 따라서 선을 만드는 방법이 달라진다.

centerx : 로봇의 생성 경로인듯

- 양측의 신뢰도 50 이상일 때
    1. 양측 영역 크기 3000 이상일 때
        
        2차 방정식의 x 결과값들의 평균으로 centerx 만듦
        
    2. 한 쪽 영역의 크기만 3000 이상일 때
        
        오른쪽 차선의 경우 x 결과값들에서 320을 뺀 배열 값으로 centerx를, 왼쪽 차선의 경우 x 결과값들에서 320을 더한 값으로 centerx를 만듦
        
- 한 쪽의 신뢰도만 50 이상일 때
    
    오른쪽 차선의 경우 x 결과값들에서 320을 뺀 배열 값으로 centerx를, 왼쪽 차선의 경우 x 결과값들에서 320을 더한 값으로 centerx를 만듦
    

- 320으로 조정하는 이유 : 영상의 가로폭은 1000이며, 중앙값은 500일 때, 중앙에서 본 왼쪽과 오른쪽 x 값이 보통 180, 820 이지 않을까 생각

centrex.item(350) : 이 값으로 차량의 이동 방향을 정하는데, 이는 2차 방정식에서 y가 350일 때의 x 결과값이며, 그 값으로 조향하는 것 같다.

---

## detect_sign

표지판을 읽어오는 launch 파일

launch 파일에서 mission 변수로 intersection, construction, parking, level_crossing, tunnel의 변수가 있다. 각각의 변수명에 따른 _sign node가 존재한다.

---

## detect_intersection

> **detect_intersection_sign**
> 

### __init__

fnPreproc 함수 실행, 이미지 타입 설정, 카메라로 부터 이미지를 받아 cbFindTrafficSign 함수에 넘겨줌, traffic_sign에 대해서 publisher을 만듦 TrafficSign = Enum(’TrafficSign’, ‘intersection left rigght’) 을 통해 0, 1 로 넘겨줌

counter = 1로 초기화

---

### fnPreproc

검출된 이미지와 비교하기전, 사전에 비교대상을 미리 준비하는 함수

cv2.SFIT_create() : 이미지끼리 서로 매칭이 되는지 확인할 때 각 이미지에서의 특징이 되는 부분끼리 비교하는 함수

detectAndCompute() : 특징점과 특징 디스크럽터를 동시에 계산함.

FLANN 모든 디스크립터를 전수 조사하기에는 속도가 느려지므로 FLANN을 사용하여 이웃하는 디스크립터끼리 비교하는 함수 → algorithm 선택키가 0 이면 BFMatcher와 동일한데 이것은 모든 디스크립터를 전수 조사하는 경우임

---

### cbFindTrafficSign

1/5 fram drop을 한다고 되어있지만 code 상으로 1/3 한 듯

→ 이 코드 전체를 분석하는 것은 의미가 없어 보인다. 어쨌든 특징점을 비교하여 폴더내에 있는 이미지와 현재 들어오는 이미지에서 같은 것을 찾는 것

---

> **detect_intersection**
> 

launch 파일에는 별것 없이 단순히 detect_intersection nodes 실행

### __init__

detect_intersection_sign nodes에서 /detect/traffic_sign 값을 cbInvokeByTrafficSign 함수로 넘겨줌, 

/detect/intersection_order 값을 cbIntersectionOrder 함수로 넘겨주는데 intersction_order값이 어디서 들어오는지 못 찾음 /control/moving/complete cbMovingComplete 함수로 넘겨줌