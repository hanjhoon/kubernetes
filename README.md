# kubernetes


# 쿠버네티스 배포 전략(RollingUpdate, Blue/Green, Canary) 및 롤백(Rollback) 개념과 설정

## 1. 개념
+ 쿠버네티스는 서비스의 무중단 업데이트를 위해 3가지 배포 방식을 지원
  - 롤링 업데이트 : 정해진 비율만큼의 파드만 점진적으로 배포
![image](https://github.com/hanjhoon/kubernetes/assets/121271030/f3783404-d1ba-4ef4-b02c-972f847d0a6d)

  - 블루/그린 : ver 1.0과 ver 2.0을 구성해놓고, 트래픽을 ver 2.0으로 전환
    
    ![image](https://github.com/hanjhoon/kubernetes/assets/121271030/05d2cdbc-b6fb-42a9-be14-52ab6f17d178)
    ![image](https://github.com/hanjhoon/kubernetes/assets/121271030/5b02bd17-f915-41fc-8f0b-f68e525a725f)

  - 카나리 : ver 2.0을 일부만 배포하고, 트래픽도 일부만 ver 2.0으로 전환. 배포에 문제가 없을 경우 ver 2.0을 점진적으로 배포 및 트래픽 전환
    
    ![image](https://github.com/hanjhoon/kubernetes/assets/121271030/8c9a68a3-7325-4723-9213-4c77c0b7af6e)

+ 쿠버네티스는 롤링 업데이트를 디폴트 배포 전략으로 설정
+ 또한 배포 이후 장애 시 복구를 위해 이전 버전으로 되돌리는 롤백 지원

## 2. 롤링 업데이트 옵션
+ maxSurge

  - 롤링 업데이트를 위해 최대로 생성할 수 있는 파드 갯수
  - maxSurge를 높게 설정하면 롤링 배포를 빠르게 적용 가능
  - % 단위 또는 갯수 단위로 지정 가능
  - 설정하지 않을 경우 기본 값은 25%

+ maxUnavailable

  - 롤링 업데이트 중 최대로 삭제할 파드 갯수
  - maxUnavailable를 높게 설정하면 롤링 배포를 빠르게 적용 가능
  - 다만 한번에 많은 파드를 삭제할 경우 트래픽이 남아있는 소수의 파드로 집중될 수 있어 값을 적절히 설정 필요
  - % 단위 또는 갯수 단위로 지정 가능
  - 설정하지 않을 경우 기본 값은 25%

## 3. YAML을 활용한 디플로이먼트 생성
디플로이먼트에 롤링 배포 적용

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: deploy-test
  name: deploy-test
spec:
  replicas: 6
  selector:
    matchLabels:
      app: deploy-test
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: deploy-test
    spec:
      containers:
      - image: nginx:1.9.0
        name: nginx

```

## 4. 명령어를 활용한 롤링 업데이트 핸들링
+ 이미지 업데이트
```
	kubectl set image deployment/[디플로이먼트_이름] [컨테이너_이름]=[이미지]:[버전]
	ex) kubectl set image deployment/deploy-test nginx=nginx:1.9.2
```

+ 롤링 업데이트 상태 확인
```
kubectl rollout status deployment [디플로이먼트_이름]
ex) kubectl rollout status deployment deploy-test

or

kubectl describe deployments.apps [디플로이먼트_이름]
ex) kubectl describe deployments.apps deploy-test
```

+ 롤백 적용

```
kubectl rollout undo deployment [디플로이먼트_이름]
ex) kubectl rollout undo deployment deploy-test
```

+ 롤링 업데이트/롤백 히스토리 확인
```
kubectl rollout history deployment [디플로이먼트_이름]
ex) kubectl rollout history deployment deploy-test

and

kubectl rollout history deployment [디플로이먼트_이름] --revision=[리비전_버전]
ex) kubectl rollout history deployment deploy-test --revision=3
```

## 5. 롤링 업데이트 적용
+ 컨테이너 이름과 이미지 버전 확인

![image](https://github.com/hanjhoon/kubernetes/assets/121271030/a8a93339-7c4b-4874-b4b1-edc13fec980b)

+ 디플로이먼트 이미지 업데이트

![image](https://github.com/hanjhoon/kubernetes/assets/121271030/bb752dff-fc3d-4237-aaf8-b92f20c9d0ad)

+ 롤링 업데이트 적용 상태 확인

![image](https://github.com/hanjhoon/kubernetes/assets/121271030/0c8fc2c4-95ea-44b7-8f34-8b1ea387d4eb)

+ 업데이트 적용 확인

![image](https://github.com/hanjhoon/kubernetes/assets/121271030/21548249-fd5c-4639-a2c4-915a0b6ffaa0)

+ 롤백 적용

![image](https://github.com/hanjhoon/kubernetes/assets/121271030/0cd3318e-97eb-4b0d-8b88-eb816ae0d3cc)

+ 롤백 적용 확인

![image](https://github.com/hanjhoon/kubernetes/assets/121271030/01f45ffa-06cb-48de-b50c-d132e801a1ba)

+ 롤링 업데이트/롤백 히스토리 확인

![image](https://github.com/hanjhoon/kubernetes/assets/121271030/29e5d335-2767-450c-a299-e4499d2930ca)

## 6. 참고
+ https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/

+ https://digitalvarys.com/canary-vs-blue-green-vs-rolling-deployment/

+ https://dev.to/mostlyjason/intro-to-deployment-strategies-blue-green-canary-and-more-3a3

+ https://ooeunz.tistory.com/124

+ https://gracefullight.dev/2019/12/27/kubernetes-rolling-update/


