---
title: "zappa + django의 cold start 문제 해결기"
date: 2018-12-14 02:33:28 +0900
categories: AWS, ZAPPA, DJANGO, LAMBDA, COLDSTART
---

django와 django rest framework를 합쳐서

django를 REST API SERVER로 사용하고 있었다.

zappa를 통해 django 를 aws api gateway + aws lambda 조합으로 사용하고 있었는데,

계속 timeout error가 났었다.

그래서 aws x-ray 를 적용했었다.

(ZAPPA + django + AWS X-RAY 적용기 : https://minwookhyejin.github.io/aws,/zappa,/django,/x-ray,/api-gateway/aws-x-ray-zappa-django/)

적용한 후 log를 확인해보니, timeout error의 정체는 다름 아닌 lambda의 cold start 문제였다.

api gateway는 30초의 hard timeout을 사용하는데, 이 값은 수정할 수 없다

실제로 api gateway 뒤의 lambda는 34초만에 요청을 처리했고, 33초 정도가 zappa가 coldstart하는데 시간을 잡아먹었고

1초는 sql과 각종 view를 만드는 행위들이었다.

128mb lambda를 사용중이었고, 256mb, 512mb, 768mb, 1024mb에 대해서 각각 cold start의 delay를 실험해봤다.

```
128mb : 33초
256mb : 15초
512mb : 8초
768mb : 4초
1024mb : 3초
```

이런 결과를 보였고, 가격 상 합리적인 것은 768mb까지였다. lambda의 가격은 memory에 거의 정비례하게 올라간다.

즉, 메모리를 두배로 올려서 시간이 절반으로 줄면 요금은 똑같다는 것이다.

요금은 100ms 마다 부과된다.

여기서 문제는 cold start가 아닐 때이다. 그 때의 수행시간은 보통 다음과 같다.

```
128mb : 170ms
256mb : 71ms
512mb : 50ms
768mb : 20ms
```

128mb의 100ms당 가격을 1달러라고 해보자. 그럼 coldstart가 아닐 때 요청당 요금은

```
128mb : 200ms * 1달러 = 2달러
256mb : 100ms * 2달러 = 2달러
512mb : 100ms * 4달러 = 4달러
768mb : 100ms * 6달러 = 6달러
```
이런 결론이 나온다. 왜냐면 lambda는 사용 시간을 100ms 기준으로 올림하거든.


그래서 결국 256mb의 lambda로 결정했다. 사실 coldstart 문제를 해결한 것은 아니다. 다만 cold start deley를 줄인 것일 뿐.


근데 ec2나 다른 호스팅을 서비스를 쓰더라도, load balancing이 되려면 새로운 instance(컴퓨터)가 올라와야 한다.


결국 새로운 instance가 올라오는데 15초 보다 빠른 서비스가 있는가? 라고 생각해봤을 때, 오히려 lambda가 합리적이란 생각이 들었다.


어느 flatform을 사용하던 load balancing 문제는 있을 것이고, 가격 면에서 봤을 때 lambda가 제일 저렴하다고 생각한다. 


물론 적당히 계산기를 두드려본 것이라 정확하진 않다.


cloud front caching을 잘 이용하면 생각보다 괜찮게 사용할 수 있을거란 생각이 든다.


제목은 거창하게 해결기라고 적었지만, 시원하게 해결하진 못했다. 성능과 가격, 그 사이에서 golden mean을 찾아보았다.


* zappa_settings.json 에서 keep_warm 기능이 있는데, 이것을 켜놔도 트래픽이 많아지면 결국 새로운 instance가 올라오므로 cold start 문제는 동일하게 발생한다.

keep_warm을 true로 놓지 말고 keep_warm : 3 이런식으로 integer로 설정할 경우 n개의 lambda container를 유지시켜준다는 것을

zappa github PR 목록에서 본 적이 있다. 관련 사항은 검색하면 나올 것이다.

일단은 keep_warm도 3으로 설정해두었다. 서버가 뻗는 일이 없길 기대해보자.
