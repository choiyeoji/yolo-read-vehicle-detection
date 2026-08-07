1팀 VALHALA , YOLO 기반 차량 인식·분류 및 고속도로 1차선 트럭 단속·돌발 상황 감지 시스템

실행방법
1.  engine파일을 실행할 소스코드와 같은 디렉토리안에 저장을한다.( 만약, onnx를 사용하는 경우 engine으로 변형하여 사용)
	
변형 코드: trtexec --onnx=vehicle5_yolo11n_e250.onnx --saveEngine=vehicle5_yolo11n_e250.engine

2. 같은 디렉토리 안에 소스파일을 실행할 파일을 생성하고 소스파일을 실행 (python 명령어를 사용할 것)

3. 고속도로 소스파일

	1. xlauch를 통해 카메라가 나오는 경우, 도로이 차들을 찍으면서 Detect가 잘되는지 확인
	2. 도로위에 정차가 되는 상황이면,  Emergency box 생성 및 capture -> captures에 저장
	     3. 마우스를 화면에 클릭을 하게 되면, dot가 나오게 되면서, 이들이 연결이 되면서, zone으로 만들어진다.
	     4. zone으로 특정Class(truck)이 지나가게 되면, capture ->  truck_screenshots으로 저장

4. 국도 소스파일
	
	1. xlauch를 통해 카메라가 나오는 경우, 도로이 차들을 찍으면서 Detect가 잘되는지 확인
	2. 도로위에 정차가 되는 상황이면,  Emergency box 생성 및 capture -> captures에 저장
3. 마우스를 화면에 Drag를 하게 되면, 사각형 box(No Parking zone)가 생성
	4. zone안으로 정차하는 차량들은 illegal을 나오게 되며 Capture가 되며