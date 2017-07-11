# RESTFul API_2

* 3 장의 내용 설명

## 클래스뷰 API
클래스뷰로 만들어 놓으면 상속받아서 사용할 수 있다. function base 뷰의 경우 decorator 다수를 사용해야한다.

클래스 뷰는 이름으로 지정하여 판단한다. 지금까지 request.post, request.get 으로 판단했었다.

```
class SnippetList(APIView):  
    """
    코드 조각을 모두 보여주거나 새 코드 조각을 만듭니다.
    """
    def get(self, request, format=None):
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return Response(serializer.data)

    def post(self, request, format=None):
        serializer = SnippetSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

앞서 1, 2 장과 다른점이 있다면 Renderer() 를 통한 변환 값을 사용하지 않고  
rest_framework 안에 있는 Response 를 직접 쓰고 있다는 것이다. 

* function base 뷰에서는 url 로 바로 넘겨서 처리해줬으나 class base 뷰를 쓸 때는 클래스를 통해서 as_view() 메서드를 사용해서 뷰함수를 생성해줘야 한다.

```
urls.py

urlpatterns=[
	url(r'^$', views.SnippetList.as_view())
]
```

하지만 지금의 코드도 get, post, delete, put 등의 요청을 매번 클래스 안에서 요청해줘야하는 불편함과 번거로움이 있다. 이를 개선하기 위해 **mixin** 을 사용한다. 

## MIXIN
 
mixin 은 필요한 기능을 상속받아 놓는데 하나씩만 정의되어 있다. 혼자 독립적으로 사용하지 않고 어딘가에 추가를 해서 추가적인 메서드를 구성하는데 사용된다. 즉, mixin 은 추가기능만 있기 때문애 전체를 구현할 수 있는 generics 가 필요하다.

```
class SnippetList(mixins.ListModelMixin,  
                  mixins.CreateModelMixin,
                  generics.GenericAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer

    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)

    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)

```

* 위의 처음 클래스뷰와 비교하여 get이 가지는 코드들이 가벼워졌다. 이는 SnippetList 라는 클래스가 mixin으로 필요한 기능을 하나씩 상속받았기 때문이다. 
* 더불어 queryset과 어떤 serializer 를 사용할 것인지 미리 밖에서 지정해준다.



## generic 클래스 뷰

mixin 을 사용하면서 상속받아 코드가 줄어들기는 했으나 하나에 한가지씩만 정의되어 있기 때문에 많은 작업을 할때는 길게 늘어나게 된다. 이를 효과적으로 개선하기 위해 generic 클래스 뷰를 사용한다.

상당부분이 감소하게 된다. generics 안에 다 정의되어있기 때문이다. get, patch, post 등

```
class SnippetList(generics.ListCreateAPIView):  
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer


class SnippetDetail(generics.RetrieveUpdateDestroyAPIView):  
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer
```

List 와 Detail 을 모두 써도 mixin 의 list 클래스의 코드정도 밖에 되지 않는다.  
여기서 단지 queryset 과 어떤 serializer 를 사용할 것인지에 대해서만 적어줄 뿐이다.

* update : 전체를 update 한다. PUT 을 사용한다.
* partial update : 부분적으로 update 한다. PATCH 를 사용한다. 기존의 내용을 그대로 쓰고 새롭게 추가된 내용만 업데이트를 해준다.
* serializer(partial=partial) 이 되어있다면 반드시 채워줘야할 것 같은 requier 값들이 없어도 된다.