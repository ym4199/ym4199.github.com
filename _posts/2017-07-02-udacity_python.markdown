# udacity_python

* Udacity's Programming Foundations with Python 을 수강하고 정리한 것입니다. 정보를 제공하기 위한 목적이 아닙니다.

### Class, Object, Constructor

강의에서 설계도의 개념으로 설명을 한다.  
하나의 class 를 통해 다양한 Objects 들이 생성 될 수 있기 때문이다.  

쉽게 생각해서 하나의 장난감 도면(설명서)만 가지고 있다면 다른 색상의 동일한 장난감을 만들 수 있는 것과 같은 맥락이다.

```
class Movie():
	def __init__(self, title, storyline):
		self.title = title
		self.storyline = storyline
		
	def show_trailer(self):
		webbrowser.open(self.storyline)
		
Toy_story = media.Movie('title','storyline')
```
위의 코드에서 구분을 해보자.  

class | object | constructor | Instance Variable | Instance Method
:---: | :---:  | :---: | :---: | :---: | 
Movie | Toy_story | \_\_init_\_ | self.title | def show_trailer(self)

> constructor 는 첫 params 로 self 를 갖는다.


### Parent, Child

단어 그대로 부모와 자식의 관계를 설명한다.  
현실 속의 생물학적 유전을 생각해보자. 자식은 부모의 특성을 상속받을 수 있지만 부모는 자식의 특성을 갖을 수 없다.  
컴퓨터 코드 상에서도 동일한 이치를 갖는다. 

```
class Parent():
	def __init__(self, last_name, eye_color):
	self.last_name = last_name
	
class Child(parent):
	def __init__(self, last_name, eye_color, height):
	Parent.__init__(self, last_name, eye_color)
	self.height = height	
```

> Parent.\_\_init\_\_(self, last\_name, eye\_color) 로 부모의 constructor 를 상속받아왔다.

Parent 의 클래스에서 받아온 값 이외의 height 에 해당하는 instance variable 만 나타내었다.



