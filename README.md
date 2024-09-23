# Linux 시스템 부하 관련 학습 정리👀

## 목차
### 1. 서론
### 2. Linux 평균 부하의 개념
### 3. 적당한 평균 부하란?
### 4. 사례 연구
### 5. 요약

---

## 1. 서론

시스템 속도가 느려지는 것을 확인하기 위해서 보통 ***`uptime`*** 이라는 명령어를 통해 다음과 같은 결과를 확인하고는 합니다.

```bash
$ uptime
02:34:03 up 2 days, 20:14, 1 user, load average: 0.63, 0.83, 0.88
```

하지만 저는 각 열의 의미를 정확히는 모르고 있었습니다.

이는 `현재시간`, `시스템 가동 시간`, `로그인 사용자 수`, `1분간 평균 부하`, `5분간 평균 부하`, `15분간 평균 부하`를 나타냅니다.

## 2. Linux 평균 부하의 개념

앞에 있는 현재시간, 시스템 가동 시간, 로그인 사용자 수에 대한건 쉽게 알 수 있는 내용입니다.

**그렇다면 평균 부하라는 것은 과연 무엇을 의미하는 것일까요?**

```bash
평균부하 = 활성 프로세스의 평균 수
```

**만약 평균 부하가 2라면?**

- CPU가 2개만 있는 시스템에서는 모든 CPU가 완전히 사용 중이라는 것을 의미합니다.
- CPU가 4개 있는 시스템에서는 50%의 유휴 CPU 용량이 있다는 것을 의미합니다.
- CPU가 1개만 있는 시스템에서는 프로세스의 절반이 CPU 시간을 놓고 경쟁한다는 뜻입니다.


## 3. 적당한 평균 부하란?

**그렇다면 적당한 평균 부하는 어느 정도 일까요?**

우리는 평균 부하에 대한 이상적인 상황은 CPU 수와 같을 떄라는 것을 알고 있습니다.  
이 상황에서 저희는 먼저 CPU의 수를 알아야합니다.

```bash
grep 'Model Name' /proc/cpuinfo | wc-l 2
```

위와 같은 명령어를 통해 CPU의 수를 알 수 있습니다.  
이때 평균 부하가 CPU수 보다 크게 된다면 과부하 상태라는 것을 알 수 있습니다.

하지만 또 다른 의문이 있습니다.  

**`1분`, `5분`, `15분` 과 같은 세 가지 평균 하중 값 중 어느 것을 참조해야 하는걸까요?**

정답은 모두 고려해야 한다 입니다.  
세 가지 다른 시간 간격은 시스템 부하의 추세를 분석할 수 있는 데이터 소스를 제공합니다.

- 값이 비슷하거나 크게 다르지 않으면 시스템 부하가 안정적임을 나타냅니다.  
- 1분 값이 15분 값보다 훨씬 낮은 경우, 부하가 감소하고 있음을 나타냅니다.  
- 반대로 1분 값이 15분 값보다 훨씬 높으면 부하가 증가하고 있음을 나타냅니다.  
- 1분 평균 부하가 CPU 수에 근접하거나 초과하면 시스템에 과부하가 발생했음을 의미합니다.

*실제 사례에서는 70% 수준으로 유지해야한다고 말하고 있습니다.*

## 4. 사례 연구🥽



### 준비 사항🛠
```bash
$ sudo apt install stress, sysstat
```
```bash
Machine configuration: 2 CPUs, 8GB RAM.
```


### 실험 내용
```bash
root@myserver1:/home/username# uptime
14:20:17 up  5:13,  8 users,  load average: 1.24, 3.40, 2.46
```

#### 시나리오 1
```bash
root@myserver1:/home/username# stress --cpu 1 --timeout 600
stress: info: [7991] dispatching hogs: 1 cpu, 0 io, 0 vm, 0 hdd
```
```bash
# the `-d` parameter indicates highlighting the changed areas
root@myserver1:/home/username# watch -d uptime
14:23:06 up  5:17,  8 users,  load average: 0.65, 2.10, 2.10
```
![image](https://github.com/user-attachments/assets/286f4ca7-4791-4d8b-9f60-15dcd6f0d254)
![image](https://github.com/user-attachments/assets/f2adbeb6-6953-4ff8-a047-5bb3b6070c18)

#### 시나리오 2
```bash
root@myserver1:/home/username# stress -i 1 --timeout 600
stress: info: [8262] dispatching hogs: 0 cpu, 1 io, 0 vm, 0 hdd
```
```bash
root@myserver1:/home/username# watch -d uptime
Every 2.0s: uptime      myserver1: Mon Sep 23 14:31:14 2024
14:31:14 up  5:24,  8 users,  load average: 1.12, 1.90, 2.05
```
![image](https://github.com/user-attachments/assets/f641028e-6b22-4cf3-9d76-31b1babfbea9)
![image](https://github.com/user-attachments/assets/880fbd81-46a4-464d-bf5d-1045d446bcda)

#### 시나리오 3
```bash
root@myserver1:/home/username# stress -c 8 --timeout 600
stress: info: [8564] dispatching hogs: 8 cpu, 0 io, 0 vm, 0 hdd
```
```bash
root@myserver1:/home/username# uptime
14:35:57 up  5:29,  8 users,  load average: 4.67, 2.27, 2.08
```
![image](https://github.com/user-attachments/assets/418d31dd-cbfc-4676-b1c7-d904b32bd968)
![image](https://github.com/user-attachments/assets/d6dbffb3-4cbf-4902-9620-515657fa5d51)

## 5. 요약


요약 내용



> 참고자료: ["Deep Understanding of Average Load in Linux."](https://blog.devgenius.io/deep-understanding-of-average-load-in-linux-74822e1dbcb1)
