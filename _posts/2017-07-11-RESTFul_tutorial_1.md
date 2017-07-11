# RESTFul

Django object <- Serializer -> Json(또는 이외의 data)  
결국 의미는 serializer(직렬화)를 통해 django object를 변환하여 반환!!

* Tutorial 1장에 해당하는 내용을 설명했다.

* postman 으로 확인작업이 중요하다. 팀프로젝트(협업) 시에 API로 변환하여 postman 으로 동작을 확인

> Serializer 의 역할
> 
> 더불어 form 갖은 역할이라고 생각하자.

## Serialize

```
views.py

class JSONResponse(HttpResponse)
	def __init__(self, data, **kwargs):
		JSONRenderer().render(data)
		kwargs['content_type'] = 'application/json'
		super().__init__(content, **kwargs)
```

* JSONRenderer() 는 특정 python 객체가 왔을때 JSON문자로 변환해준다.
* JSON 은 우리가 보다 쉽게 확인할 수 있는 data 값을 나타낸다.

> JSON object는 name/value 쌍들의 비순서화된 SET이다.  

```
views.py

@csrf_exempt
def snippet_list(request):
	if request.method =='GET':
		snippets = Snippet.object.all()
		serializer = SnippetSerializer(snippets, many=True)
		return JSONResponse(serializer.data)
```

* GET 요청일때 Snippet 을 전체 다 가져와서 이를 JSON 형식으로 변환한다.
* 여러 object 를 갖을때 Serializer의 속성으로 many=True 를 줘야한다.

> snippets 는 윗줄에서 Snippet.object.all() 로 Snippet 전체를 가져오기 때문에 many=True 가 필요하다.


이처럼 작업이 끝났다면 url 로의 연결이 필수적이다. 아직 우리는 snippet을 수정만 했지 무엇인지 나타내고 있지 않다.

> snippet_list 로 정의한 부분에 대한 url 의 접근이 필요.

```
elif request.method == 'POST':
        serializer = SnippetSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

```

* POST 요청일때 request를 data로 받아서 Serializer 시킨다. 이를 유효한지 판단하여 유효할 때 생성하고 그렇지 않다면 에러를 반환한다. 
* pk 를 받지 않기 때문에 반드시 post 요청에 대해 생성된다.

> \.is\_valid() : 인스턴스를 사용하여 유효성 검사를 하여 부울값으로 나타낸다.  
> 폼에서 자동으로 clean() 을 호출한다. (is\_valid()는 views 에서 사용하능하고, clean()은 form class에서 사용가능하다.)
> clean\_data = form\.is\_valid() 할 필요가 없다. is\_valid()가 cleaned 된 form object 에 대해 clean을 호출하고 overwrite data 하기 때문이다.      
> .errors : 에러의 속성에 접근하여 에러 메세지를 사전형으로 가져온다.


```
views.py

@api_view(['GET', 'PUT', 'DELETE'])
def snippet_detail(request, pk):  
    """
    코드 조각 조회, 업데이트, 삭제
    """
    try:
        snippet = Snippet.objects.get(pk=pk)
    except Snippet.DoesNotExist:
        return Response(status=status.HTTP_404_NOT_FOUND)

    if request.method == 'GET':
        serializer = SnippetSerializer(snippet)
        return Response(serializer.data)

    elif request.method == 'PUT':
        serializer = SnippetSerializer(snippet, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    elif request.method == 'DELETE':
        snippet.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)

```

* list 의 코드와 유사하지만 추가사항으로 put 에 대한 update 와 delete 의 삭제가 추가되었다.
* url 을 받을때 주의할 것은 pk를 받고 있기 때문에 url에 이를 명시해야 한다.


## ModelSerialize 
직접 구현하지 않고 ModelSerialize 를 이용한다.  
modelserialize 이기 때문에 create, update 부분이 자동이다. (snippet 모델을 이용하는 형태로)

```
 class SnippetSerializer(serializers.ModelSerializer):  
    class Meta:
        model = Snippet
        fields = ('id', 'title', 'code', 'linenos', 'language', 'style')
```
아래의 코드가 위처럼 간단하게 쓸 수 있기 때문에 사용한다.

```

class SnippetSerializer(serializers.Serializer):  
    pk = serializers.IntegerField(read_only=True)
    title = serializers.CharField(required=False, allow_blank=True, max_length=100)
    code = serializers.CharField(style={'base_template': 'textarea.html'})
    linenos = serializers.BooleanField(required=False)
    language = serializers.ChoiceField(choices=LANGUAGE_CHOICES, default='python')
    style = serializers.ChoiceField(choices=STYLE_CHOICES, default='friendly')

    def create(self, validated_data):
        """
        검증한 데이터로 새 `Snippet` 인스턴스를 생성하여 리턴합니다.
        """
        return Snippet.objects.create(**validated_data)

    def update(self, instance, validated_data):
        """
        검증한 데이터로 기존 `Snippet` 인스턴스를 업데이트한 후 리턴합니다.
        """
        instance.title = validated_data.get('title', instance.title)
        instance.code = validated_data.get('code', instance.code)
        instance.linenos = validated_data.get('linenos', instance.linenos)
        instance.language = validated_data.get('language', instance.language)
        instance.style = validated_data.get('style', instance.style)
        instance.save()
        return instance
```


## Resquest & Response

### Request
rest_framework 의 Request 는 함수의 request 의 확장형태라 보자.  
 그전까지 request.POST, reqeust.GET 의 형태로 form 데이터에 제한되어서 사용했지만 Request.data 로 통합하여 아무 데이터나 다룰 수 있다.

### Response
Response 의 경우도 Response(data) 로 사용하면 알아서 클라이언트에 맞춰 변환한다.



## postman 

조건에 맞춰 post, get, patch 를 설정하고 해당 url 로 접근. 이때, 필요 값들을 줘서 확인한다.