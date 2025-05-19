# YOLOv8 기반 실시간 객체 인식 웹 애플리케이션

## 1. 문제 정의

실시간으로 카메라를 통해 객체를 인식하는 기능은 보안, 분석, 접근성 등 다양한 영역에서 중요합니다. 그러나 일반 사용자들은 객체 인식 기술에 쉽게 접근하기 어려운 상황입니다. 이 프로젝트는 다음과 같은 문제를 해결하고자 합니다:

- 고급 객체 인식 기술을 웹 기반으로 누구나 접근 가능하게 만들기
- 사용자 기기의 카메라를 활용하여 실시간으로 객체를 감지하고 시각화
- 경량화된 모델을 통한 빠른 처리 속도 제공
- 다양한 환경과 디바이스에서 동작 가능한 크로스 플랫폼 호환성 확보

## 2. 요구사항 분석

### 기능적 요구사항

1. **카메라 액세스 및 제어**
   - 사용자 기기의 전면/후면 카메라에 접근할 수 있어야 함
   - 카메라 전환 기능 제공

2. **객체 인식 기능**
   - 실시간 비디오 스트림에서 객체 감지
   - 감지된 객체에 경계 상자(bounding box) 표시
   - 객체 이름과 인식 정확도 표시

3. **결과 시각화**
   - 감지된 객체 목록 표시
   - 객체 유형별 개수 통계 제공
   - 시각적으로 명확한 UI 제공

4. **사용자 인터페이스**
   - 직관적인 버튼 배치와 기능
   - 반응형 디자인으로 다양한 화면 크기에 대응
   - 실시간 피드백 제공

### 비기능적 요구사항

1. **성능**
   - 낮은 지연시간으로 객체 인식 처리
   - 모바일 환경에서도 효율적인 동작

2. **보안**
   - 카메라 접근 권한 요청 및 안전한 처리
   - 업로드된 이미지 적절한 관리

3. **확장성**
   - 다양한 YOLO 모델 지원 가능한 구조
   - 추가 기능 확장이 용이한 모듈식 설계

4. **사용성**
   - 복잡한 설정 없이 즉시 사용 가능
   - 직관적인 UI/UX

## 3. 기술 스택 및 아키텍처

### 기술 스택

- **백엔드**
  - Flask: 파이썬 기반 경량 웹 프레임워크
  - Flask-CORS: 크로스 오리진 리소스 공유 지원
  - Ultralytics YOLOv8: 객체 인식 모델
  - OpenCV (cv2): 이미지 처리
  - NumPy: 배열 및 행렬 처리
  - Pyngrok: 로컬 개발 서버를 인터넷에 공개

- **프론트엔드**
  - React: 사용자 인터페이스 구축 라이브러리
  - Bootstrap: 반응형 디자인 프레임워크
  - MediaDevices API: 카메라 접근 및 스트림 처리
  - Canvas API: 비디오 스트림에 경계 상자 표시

### 아키텍처

이 프로젝트는 클라이언트-서버 아키텍처를 따르며, 주요 구성 요소는 다음과 같습니다:

1. **클라이언트 (프론트엔드)**
   - 사용자 카메라 액세스 및 비디오 스트림 처리
   - 주기적으로 비디오 프레임 캡처
   - 캡처된 이미지를 Base64로 인코딩하여 서버에 전송
   - 서버로부터 받은 인식 결과를 시각화

2. **서버 (백엔드)**
   - Flask 웹 서버로 API 엔드포인트 제공
   - YOLOv8 모델을 통한 객체 감지 처리
   - 감지 결과를 JSON 형태로 클라이언트에 반환

3. **데이터 흐름**
   - 카메라 → 프론트엔드(캡처) → 서버(감지) → 프론트엔드(시각화)

4. **배포**
   - ngrok을 통한 로컬 서버의 인터넷 공개
   - 별도의 서버 구성 없이 테스트 가능

## 4. 핵심 알고리즘 및 처리 메커니즘

### YOLOv8 객체 인식 알고리즘

YOLOv8(You Only Look Once)는 단일 신경망을 사용하여 이미지에서 객체를 감지하는 실시간 객체 인식 알고리즘입니다. 기존 객체 감지 방법들이 여러 단계를 거치는 것과 달리, YOLO는 전체 이미지를 한 번에 처리하여 속도와 정확성을 모두 확보합니다.

**YOLOv8의 주요 특징:**
- 단일 단계 감지기(Single-stage detector)로 빠른 처리 속도
- 이미지를 그리드로 분할하여 각 셀에서 객체 감지
- 경계 상자(bounding box), 신뢰도 점수, 클래스 확률을 동시에 예측
- Ultralytics의 최적화된 구현으로 이전 버전보다 향상된 성능

### 이미지 처리 메커니즘

1. **프론트엔드에서의 이미지 캡처:**
   ```javascript
   const captureImage = () => {
     const canvas = document.createElement('canvas');
     canvas.width = videoRef.current.videoWidth;
     canvas.height = videoRef.current.videoHeight;
     const ctx = canvas.getContext('2d');
     ctx.drawImage(videoRef.current, 0, 0, canvas.width, canvas.height);
     return canvas.toDataURL('image/jpeg');
   };
   ```

2. **Base64 인코딩 및 서버 전송:**
   ```javascript
   const imageData = captureImage();
   const response = await fetch('/api/detect', {
     method: 'POST',
     headers: {'Content-Type': 'application/json'},
     body: JSON.stringify({ image: imageData }),
   });
   ```

3. **서버에서의 이미지 디코딩:**
   ```python
   img_bytes = base64.b64decode(img_data)
   nparr = np.frombuffer(img_bytes, np.uint8)
   image = cv2.imdecode(nparr, cv2.IMREAD_COLOR)
   ```

4. **YOLOv8 모델을 통한 객체 감지:**
   ```python
   results = model(image)
   detections = []
   for r in results:
       boxes = r.boxes
       for box in boxes:
           x1, y1, x2, y2 = box.xyxy[0].tolist()
           conf = float(box.conf[0])
           cls = int(box.cls[0])
           name = model.names[cls]
           detections.append({
               'bbox': [x1, y1, x2, y2],
               'confidence': conf,
               'class': cls,
               'name': name
           })
   ```

5. **감지 결과 시각화:**
   ```javascript
   const drawDetections = (detectionResults) => {
     // Canvas에 경계 상자와 레이블 그리기
     detectionResults.forEach(detection => {
       const [x1, y1, x2, y2] = detection.bbox;
       // 경계 상자 그리기
       ctx.strokeRect(scaledX1, scaledY1, scaledWidth, scaledHeight);
       // 레이블 표시
       ctx.fillText(label, scaledX1 + 5, scaledY1 - 5);
     });
   };
   ```

## 5. 구현 과정 및 주요 코드 설명

### 5.1 서버 사이드 구현

Flask 애플리케이션은 다음과 같은 주요 기능을 담당합니다:

1. **애플리케이션 초기화 및 설정:**
   ```python
   app = Flask(__name__, template_folder='./www', static_folder='./www', static_url_path='/')
   CORS(app)
   model = YOLO('yolov8n.pt') # 경량화된 YOLOv8 모델 로드
   ```

2. **API 엔드포인트 구현:**
   ```python
   @app.route('/api/detect', methods=['POST'])
   def detect_objects():
       # Base64 이미지 데이터 처리
       # 객체 감지 수행
       # 결과 반환
   ```

3. **이미지 처리 및 저장:**
   ```python
   timestamp = int(time.time())
   img_path = f"{UPLOAD_FOLDER}/image_{timestamp}.jpg"
   cv2.imwrite(img_path, image)
   ```

### 5.2 클라이언트 사이드 구현

React 애플리케이션은 다음과 같은 주요 기능을 담당합니다:

1. **카메라 관리:**
   - 카메라 접근 및 스트림 설정
   - 전/후면 카메라 전환
   - 스트림 정리 및 해제

2. **주기적 객체 감지:**
   ```javascript
   timerRef.current = setInterval(() => {
     if (!isProcessing) {
       detectObjects();
     }
   }, 10000); // 10초마다 객체 감지
   ```

3. **감지 결과 시각화:**
   - Canvas를 사용한 경계 상자 그리기
   - 객체 이름 및 정확도 표시
   - 감지된 객체 목록과 통계 제공

4. **사용자 인터페이스:**
   - Bootstrap을 활용한 반응형 디자인
   - 카메라 전환 및 감지 실행 버튼
   - 감지 결과 목록 표시

### 5.3 서버 배포 구현

ngrok을 활용한 로컬 서버를 인터넷에 공개하는 과정:

```python
import subprocess
import time
from pyngrok import ngrok

server_process = subprocess.Popen(["python", "app.py"])
http_tunnel = ngrok.connect(3000)
print(f"ngrok 터널이 생성되었습니다: {http_tunnel.public_url}")
```

## 6. 해결 과정에서 발생한 문제점과 해결 방법

### 문제 1: 카메라 접근 권한 및 호환성

**문제점**: 다양한 브라우저와 기기에서 카메라 접근 방식이 다르며, 사용자 권한 요청이 일관되지 않음

**해결 방법**: 
- MediaDevices API의 getUserMedia 메서드를 사용하여 표준화된 방식으로 카메라 접근
- 에러 처리를 통해 권한 거부나 장치 미지원 상황에 대응
- 카메라 설정을 위한 제약 조건(constraints)을 환경에 맞게 조정

### 문제 2: 이미지 전송 지연과 성능 최적화

**문제점**: 지속적인 이미지 캡처와 전송으로 인한 네트워크 부하와 지연 발생

**해결 방법**:
- 일정 간격으로 객체 감지를 수행하여 연속 요청 방지
- Base64 인코딩 이전에 이미지 크기 조정 고려
- 처리 중 상태 플래그를 사용하여 중복 요청 방지

### 문제 3: 경계 상자 위치 정확도

**문제점**: 비디오 요소와 캔버스 사이의 크기 차이로 경계 상자 위치가 부정확함

**해결 방법**:
- 비디오와 캔버스의 비율을 맞추기 위한 스케일링 적용
- 비디오 로드 완료 후 캔버스 크기를 동기화

## 7. 습득한 기술 및 인사이트

### 기술적 습득

1. **딥러닝 모델 통합**
   - YOLOv8 모델을 웹 애플리케이션에 통합하는 과정 이해
   - 클라이언트-서버 간 효율적인 데이터 교환 방식

2. **실시간 비디오 처리**
   - 브라우저에서 카메라 스트림을 다루는 기술
   - Canvas API를 활용한 실시간 오버레이 구현

3. **Flask와 React 연동**
   - RESTful API를 통한 백엔드와 프론트엔드 통신
   - Flask의 정적 파일 서빙 방식

4. **ngrok을 활용한 임시 배포**
   - 로컬 개발 서버를 인터넷에 노출하는 방법
   - 터널링 서비스의 활용

### 인사이트

1. **사용자 경험과 성능의 균형**
   - 실시간성과 정확도 사이의 트레이드오프 관계 이해
   - 제한된 리소스에서 최적의 사용자 경험 제공 방안

2. **모바일 우선 설계의 중요성**
   - 다양한 디바이스에서의 일관된 경험 제공 필요성
   - 모바일 환경을 고려한 UI/UX 디자인

3. **프라이버시와 보안 고려사항**
   - 카메라 접근과 데이터 처리에 관한 사용자 신뢰 구축
   - 민감한 정보의 안전한 처리 방법

## 8. 결론

이 프로젝트는 최신 객체 인식 기술인 YOLOv8을 웹 애플리케이션으로 성공적으로 통합하여, 사용자가 브라우저에서 실시간으로 객체를 감지하고 분석할 수 있는 플랫폼을 구축했습니다. 제한된 리소스와 환경에서도 효율적으로 동작하는 솔루션을 개발함으로써, 고급 컴퓨터 비전 기술을 일반 사용자가 쉽게 접근하고 활용할 수 있는 가능성을 보여주었습니다.

구현 과정에서 다양한 기술적 도전과제를 해결하며, 웹 기반 머신러닝 애플리케이션 개발에 필요한 핵심 지식과 경험을 습득할 수 있었습니다. 특히 클라이언트-서버 아키텍처, 비디오 스트림 처리, 딥러닝 모델 통합, 사용자 인터페이스 설계 등 다양한 영역에서의 실전 경험이 큰 자산이 되었습니다.
