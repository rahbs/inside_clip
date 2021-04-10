# inside_clip
Video Production Project Using Face Swap Machine Learning Technology  
머신러닝을 통해 얼굴 합성 기술을 이용한 동영상 제작 프로젝트  
드라마나, 짧은 동영상에서 좋아하는 배우의 상대 배우를 자신의 얼굴로 바꿈으로써 연예인과 소통하는 기분을 느끼게 해주는 소프트웨어  
  
![face_swap](https://user-images.githubusercontent.com/35826556/96461070-e7cb0580-125e-11eb-9471-df8ce18eefc1.png)

## How it works

### calculating angles of face
- dlib (python 라이브러리) 사용 : 얼굴의 facial landmark 위치 반환하는 기능 제공
<img src="https://user-images.githubusercontent.com/35826556/114263059-4e66fa00-9a1e-11eb-89ef-63bdd9ebd339.png" width="400">  

- calculate angle of face  
좌우 180도를 총 10개의 구간으로 나누어 각도를 계산  

```python
def getFaceAngle(shapes2D):
    # 왼쪽, 오른쪽의 기준은 영상 기준이 아니라 영상을 보는 사람의 기준
    # 두 점 사이의 길이:root((x1-x2)^2+(y1-y2)^2)
    # 왼쪽 볼 길이 (32번 포인트 - 3번 포인트)
    leftCheekLen = math.sqrt(
        math.pow(shapes2D[0][0][31] - shapes2D[0][0][2], 2) + math.pow(shapes2D[0][1][31] - shapes2D[0][1][2], 2))
    # 오른쪽 볼 길이 (15번 포인트 - 36번 포인트)
    rightCheekLen = math.sqrt(
        math.pow(shapes2D[0][0][14] - shapes2D[0][0][35], 2) + math.pow(shapes2D[0][1][14] - shapes2D[0][1][35], 2))


    # 얼굴 수평 길이 (15번 포인트 - 3번 포인트)
    faceParallelLen = math.sqrt(
        math.pow(shapes2D[0][0][14] - shapes2D[0][0][2], 2) + math.pow(shapes2D[0][1][14] - shapes2D[0][1][2], 2))
    # (비율) 왼쪽 볼 길이 / 얼굴 수평 길이
    leftCheekRatio = leftCheekLen / faceParallelLen
    # (비율) 오른쪽 볼 길이 / 얼굴 수평 길이
    rightCheekRatio = rightCheekLen / faceParallelLen

    # 왼쪽 볼 비율- 오른 쪽 비율(정면 보고 있다고 가정, x축만)
    face_x_dif = leftCheekRatio - rightCheekRatio

    # face_x_dif
    ret = 0
    # ~-0.4: 왼쪽으로 0
    if (face_x_dif < -0.4):
        ret = 0

    # -0.4~-0.3: 왼쪽으로 1
    elif (-0.4 <= face_x_dif) & (face_x_dif < -0.3):
        ret = 1

    # -0.3~-0.2: 왼쪽으로 2
    elif (-0.3 <= face_x_dif) & (face_x_dif < -0.2):
        ret = 2

    # -0.2~-0.1: 왼쪽으로 3
    elif (-0.2 <= face_x_dif) & (face_x_dif < -0.1):
        ret = 3

    # -0.1~0.1: 정면 4
    elif (-0.1 <= face_x_dif) & (face_x_dif <= 0.1):
        ret = 4

    # 0.1 ~ 0.2: 오른쪽으로  5
    elif (0.1 <= face_x_dif) & (face_x_dif < 0.2):
        ret = 5

    # 0.2 ~ 0.3: 오른쪽으로 6
    elif (0.2 <= face_x_dif) & (face_x_dif < 0.3):
        ret = 6

    # 0.3 ~ 0.4: 오른쪽으로 7
    elif (0.3 <= face_x_dif) & (face_x_dif < 0.4):
        ret = 7

    # 0.4~: 오른쪽으로 8
    else:
        ret = 8
        
    return ret
```
### flow
1. 사용자의 얼굴이 좌우로 180도 나오는 동영상을 입력으로 받는다.
2. 총 9개 각도의 대표사진들 추출 (만약 모두 추출되지 않았다면, 다시 찍을 것 요청)
3. 합성할 동영상 선택
4. 합성하고자 하는 동영상에서 매 프레임 마다, 배우 얼굴 각도 추출 후, 대응되는 각도의 사용자 얼굴로 합성

### demo video
![out_1 (online-video-cutter com)](https://user-images.githubusercontent.com/35826556/113377941-b3807700-93b0-11eb-97e0-45f7474b715b.gif)
