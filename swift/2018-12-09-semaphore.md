---
title: "swift 4에서 Semaphore 사용"
date: 2018-12-09 01:25:28 +0900
categories: jekyll update
---


s = DispatchSemaphore(value: 1)

value 는 동시에 접근할 수 있는 thread의 수 라고 생각하면 되는 듯 싶다.

wait()
  semaphore의 value 를 1씩 낮추고, value가 음수가 되면 value가 0 이상이 될 때 까지 busy waiting을 한다.
  
signal()
  semaphore의 value 를 1씩 올린다
  
위의 경우 value가 1인 상태로 선언했으므로, 처음으로 s.wait()을 사용한 thread는 바로 value를 0으로 낮추고 멈추지 않는다.

그 후로 n개의 thread가 wait()을 불러도 value가 0이므로 busy waiting을 하며

s.signal()이 불리는 순간 waiting queue 에서 대기중이던 thread가 다음 권한을 얻는다.



< modipie에서의 사용 >

REST api call을 위해 access token에 접근하는 시점에서 semaphore를 이용해 항상 한개의 http call만 access token 값에 관여할 수 있게 해두었다.

access token 이 손상되거나 만료되었거나 여타 이유로 http 401 auth error를 띄우면 access token에 lock을 걸고 refresh token call을 하며,

그 외의 thread는 모두 access token 값이 변경될 때 까지 wait 한다.


출처 : https://developer.apple.com/documentation/dispatch/dispatchsemaphore
