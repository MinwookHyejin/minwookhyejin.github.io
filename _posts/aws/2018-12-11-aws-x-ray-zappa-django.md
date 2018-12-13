---
title: "AWS X-RAY와 zappa django 연동하기"
date: 2018-12-11 21:33:28 +0900
categories: AWS, ZAPPA, DJANGO, X-RAY, API-GATEWAY
---

django와 django rest framework를 합쳐서

django를 REST API SERVER로 사용하고 있었다.

zappa를 통해 django 를 aws api gateway + aws lambda 조합으로 사용하고 있었는데,

api gateway를 통한 https request가 간혈적으로 timeout error를 띄우는 사태가 발생했다.

그래서 django(lambda)의 결과를 URL 별로 모니터링 할 도구가 필요했고, AWS X-RAY를 사용하기로 했다.


  1. zappa

`zappa_settings.json` 파일에서, 

```
"xray_tracing": true
```
로 설정하면 된다고 zappa github page에 나와있다. 

추가해주었다.

출처 : https://github.com/Miserlou/Zappa








  2. django
  
django 의 경우, AWS 에서 공식 지원하는 python x-ray sdk가 있다.

settings.py

```
MIDDLEWARE = [
    'aws_xray_sdk.ext.django.middleware.XRayMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
]
```
실패한 요청까지 포함해 모든 요청을 로깅하기 위해 middleware 최상단에 넣어준다.





```
INSTALLED_APPS = [
    ...
    'aws_xray_sdk.ext.django',
]
```
installed_apps에 추가




```
XRAY_RECORDER = {
    'AWS_XRAY_DAEMON_ADDRESS': '127.0.0.1:2000',
    'AUTO_INSTRUMENT': True,  # If turned on built-in database queries and template rendering will be recorded as subsegments
    'AWS_XRAY_CONTEXT_MISSING': 'RUNTIME_ERROR',
    'PLUGINS': (),
    'SAMPLING': True,
    'SAMPLING_RULES': None,
    'AWS_XRAY_TRACING_NAME': None, # the segment name for segments generated from incoming requests
    'DYNAMIC_NAMING': None, # defines a pattern that host names should match
    'STREAMING_THRESHOLD': None, # defines when a segment starts to stream out its children subsegments
}
```
레코더 설정이다. 뭐가 뭔지 잘 모르겠어서 그냥 적용시키자는 생각에 그대로 입력했다.

daemon address :  x-ray를 logging 할 때 마다 http로 업로드할 경우 부하가 커지니까 deamon process란 것을 실행시켜 두는데, daemon process가 log를 수집했다가 어느정도 쌓이면 한번에 업로드한다.

그 daemon process가 실행되는 ip주소이다. 

zappa를 통해 업로드 된 aws lambda 환경에서 구동될 것이기 때문에 localhost:2000 을 사용한다. 기본 값이다.

출처 : (https://docs.aws.amazon.com/xray-sdk-for-python/latest/reference/)







그대로 적용한 결과 zappa scheduling(일정한 주기마다 django 내의 함수를 실행시켜준다) 을 통한 사용은 log 가 잘 남지만,

api gateway를 통한 https request들은 logging이 안되는 문제가 발생했다.

이것저것 몇 시간의 구글링 삽질 끝에, `api gateway에서 x-ray tracing`을 켜야 전달되는 것을 확인하였다.

(이 문서 참조 : https://docs.aws.amazon.com/ko_kr/xray/latest/devguide/xray-services-apigateway.html)

해당 옵션을 키면 https 요청을 받은 api gateway가 http header를 보고 x-ray에 필요한 tracing header를 만들어서 넘겨주게 된다.

옵션 적용 후 http 요청을 한 뒤 x-ray를 확인해본 결과 URL별로 이쁘게 logging이 되는 것을 확인했다.
