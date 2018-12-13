---
title: "swift4 UICollectionView에서 Seperator view 야매로 구현하기"
date: 2018-12-14 02:13:28 +0900
categories: AWS, ZAPPA, DJANGO, X-RAY, API-GATEWAY
---


UICollectionView는 android의 recyclerView에 대응되는 뷰이다.


layout 객체를 통해서 vertical, horizontal 방향을 선택해 줄 수 있으며, RecyclerView과는 살짝 다르게 동작한다.


RecyclerView는 전부 다 한줄로 나오고, 각 child view가 자기 크기를 parent에게 알려주는 방식인데


UICollectionView는 사이즈에 맞춰서 여러줄로 나오고 부족하면 scroll 하는 방식이며,


childView(UICollectionViewCell)에게 크기를 parent(UICollectionView)가 미리 정해주는 방식이다.


childview를 child의 content 크기에 맞추려면 recyclerview는 `wrap_content`로 간단하게 끝났지만


UICollectionView는 auto layout을 쓰고 그 크기를 parent에 연결해야되서 코드로 설정하는게 좀 더 복잡했다.


대신 storyboard를 활용하면 android xml보단 훨씬 직관적인 장단점이 있다. code로 설정하는 편리함은 android쪽이 더 나은 느낌.

    




링크로 시작한다 : (https://developer.apple.com/documentation/uikit/uicollectionview)


위 링크를 보면 UICollectionView에서는 Supplementary Views 라는게 있지만 간단하게 해결하려고 했다.


구글링 결과 각 CollectionViewCell의 하단이나 상단에 뷰를 붙여서 Seperator를 구현하라는 얘기가 대부분이었으나, 


나는 CollectionViewCell 사이에만 Seperator가 필요한 상황이라(위 방법대로라면 하단에 뷰를 붙이면 맨 마지막 뷰는 하단에도 Seperator가 붙는다)


Section을 활용한 트릭으로 넘겼다.




1. UICollectionViewDataSource

```
datas = ["1", "2", "3", "4", "5", "6"]

func numberOfSections(in collectionView: UICollectionView) -> Int{
  return datas.count // 한개의 section 당 한개의 data를 사용함.
}

func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int{
  let sections = numberOfSections(in: collectionView)
  if section == sections - 1{
    //맨 마지막 섹션은 collectionViewCell이 한개다
    return 1
  }
  // 그 외의 섹션은 2개의 collectionViewCell을 가짐. 0번은 진짜 viewCell, 1번은 Seperator
  return 2
}


func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell{
  if indexPath.row == 0{
    let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "viewcell", for: IndexPath)
    let data = d[indexPath.section] // section 번호를 사용함.
    // do something
    // content가 들어간 collectionViewCell에 필요한 설정들을 해준다
    return cell
  }else{
    let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "seperator", for: IndexPath)
    // seperator 설정을 해준다. 길이라던지.. 색상이라던지.. 자기 입맛에 맞게
    // setting for seperator
    return cell
  }
}

```


이제 ```UICollectionView.register()``` 함수를 통해,


만들어둔 CollectionViewCell을 `"viewcell"`, `"seperator"`에 대해서 등록해주면 설정은 끝이 난다.


나머지는 layout 배치와 data binding 작업을 해주면 된다.




* 여담이지만,

recyclerView와는 동작하는 방식이 다른데, 당연히 같은 방식일거라고 생각하고 접근했다가 쓸 데 없이 시간을 날렸다.

android는 몇 년 다뤄본거에 비해 swift는 이제 겨우 한달이 채 되지 않아서 android식으로 생각하는게 익숙한 듯.

차라리 android를 모르는 상태에서 swift를 배웠다면 더 쉽게 했었을 것 같다.

그래도 차차 나아지는 것 같은 느낌.

편견이란게 참 무섭다.
