레일스에서 자바스크립트로 작업하기
================================

본 가이드는 레일스의 내장 Ajax/JavaScript 기능(그 이상)을 다룹니다; 당신이 손쉽게 풍부하고 동적인 Ajax 응용프로그램을 작성할 수 있도록 해 줄 것입니다.

본 가이드를 읽은 후, 다음의 내용들을 알게 될 것입니다.

* Ajax의 기초.
* 겸손한 자바스크립트(Unobtrusive JavaScript).
* 어떻게 레일스의 내장 헬퍼가 당신을 돕는가.
* 서버측에서 Ajax를 다루는 법.
* Turbolinks gem.

-------------------------------------------------------------------------------

[An Introduction to Ajax] Ajax 소개
------------------------

Ajax를 이해하기 위해, 먼저 웹브라우저가 보통 무엇을 하는지 이해해야 합니다.

웹브라우저의 주소 막대에 `http://localhost:3000`를 입력하고 'Go'를 누르면, 브라우저('클라이언트')는 서버로 요청을 보냅니다.
브라우저는 서버로부터의 응답을 분석하고, 자바스크립트 파일들, 스타일시트들 그리고 이미지들과 같은 연관된 모든 자산들을 불러옵니다.
그리고 나서 페이지들을 조합합니다. 만약 링크를 클릭하면, 브라우저는 같은 절차를 수행합니다. 
페이지를 불러오고, 자산들을 불러오고, 그것들을 조합하여 결과를 보여줍니다.
이것을 '요청 응답 주기(request response cycle)'이라 합니다.

자바스크립트도 서버로의 요청을 보내고, 응답을 분석할 수 있습니다. 그리고 페이지에 정보를 업데이트할 수 있습니다.
이 두가지 능력을 조합하여 자바스크립트 작성자는 서버로부터 전체 페이지 데이터를 받아올 필요 없이, 페이지의 일부만을 갱신하는 웹 페이지를 작성할 수 있습니다.
이것이 Ajax라 부르는 강력한 기술입니다.

레일스는 커피스크립트(CoffeeScript)를 기본으로 탑재하고 있고 본 가이드의 나머지 예제들은 커피스크립트로 만들어질 것입니다.
예제 강좌는 모두 평범한 자바스크립트에도 적용됨은 물론입니다.

여기 jQuery 라이브러리를 이용하여 Ajax 요청을 만드는 커피스크립트 코드 에제가 있습니다.

```coffeescript
$.ajax(url: "/test").done (html) ->
  $("#results").append html
```

이 코드는 "/test"로부터 데이터를 받아온 후, `results` 아이디를 가진 `div`에 그 결과를 덧붙입니다.

레일스에는 이 기술을 이용하여 웹페이지를 만드는데 필요한 많은 내장 지원이 있습니다. 당신은 이 코드를 직접 작성할 필요가 거의 없습니다.
이후 본 가이드는 당신이 이 방식으로 웹사이트를 만드는데 레일스가 어떻게 도움을 주는지 보여줄 것입니다. 
하지만 이 모든 것은 매우 간단한 기술 위에 만들어져 있습니다.

[Unobtrusive JavaScript] 겸손한(Unobtrusive) 자바스크립트 
-------------------------------------

레일스는 DOM에 연결된 자바스크립트를 다루기 위해 "겸손한 자바스크립트"라 불리는 기술을 사용합니다.
이것은 일반적으로 프론트엔드 커뮤니티에서 모범사례로 간주됩니다. 
하지마 당신은 간혹 다른 방식으로 보여주는 튜토리얼을 볼 수 있습니다.

여기 자바스크립트를 작성하는 가장 간단한 방법이 있습니다. 이것은 'inline JavaScript'라 불리는 것입니다.

```html
<a href="#" onclick="this.style.backgroundColor='#990000'">Paint it red</a>
```

링크가 클릭될 때, 배경색이 붉은색으로 바뀔 것입니다. 여기 문제가 있습니다. 
클릭했을 때 실행되기를 원하는 자바스크립트가 아주 많이 있다면 어떤 일이 생길까요?

```html
<a href="#" onclick="this.style.backgroundColor='#009900';this.style.color='#FFFFFF';">Paint it green</a>
```

불편하지 않습니까? 우리는 클릭 핸들러 밖으로 함수 정의를 끌어내어 커피스크립트로 바꿀 수 있습니다.

```coffeescript
paintIt = (element, backgroundColor, textColor) ->
  element.style.backgroundColor = backgroundColor
  if textColor?
    element.style.color = textColor
```

이렇게 하면 페이지는 다음과 같이 됩니다.

```html
<a href="#" onclick="paintIt(this, '#990000')">Paint it red</a>
```

좀 더 나아졌습니다만, 같은 효과를 가진 여러 링크가 있다면 어떻게 될까요?

```html
<a href="#" onclick="paintIt(this, '#990000')">Paint it red</a>
<a href="#" onclick="paintIt(this, '#009900', '#FFFFFF')">Paint it green</a>
<a href="#" onclick="paintIt(this, '#000099', '#FFFFFF')">Paint it blue</a>
```

그닥 DRY하지 않지요? 우리는 이벤트를 이용하여 이 문제를 해결할 수 있습니다. 
링크에 `data-*` 속성을 추가하고 이 속성을 가진 모든 링크의 클릭 이벤트에 핸들러를 연결할 것입니다.

```coffeescript
paintIt = (element, backgroundColor, textColor) ->
  element.style.backgroundColor = backgroundColor
  if textColor?
    element.style.color = textColor

$ ->
  $("a[data-background-color]").click ->
    backgroundColor = $(this).data("background-color")
    textColor = $(this).data("text-color")
    paintIt(this, backgroundColor, textColor)
```
```html
<a href="#" data-background-color="#990000">Paint it red</a>
<a href="#" data-background-color="#009900" data-text-color="#FFFFFF">Paint it green</a>
<a href="#" data-background-color="#000099" data-text-color="#FFFFFF">Paint it blue</a>
```

우리는 이것을 '겸손한' 자바스크립트라고 부릅니다. 더이상 자바스크립트를 HTML 안에 섞지 않기 때문입니다.
우리는 앞으로 있을 변경을 쉽게 하기 위해 적절하게 우리 고려사항을 분리했습니다.
우리는 data 속성을 추가하는 것만으로 어떤 링크에든 손쉽게 동작을 추가할 수 있습니다.
우리는 미니마이저와 연결연산자를 통해 모든 우리의 자바스크립트를 실행할 수 있습니다.
우리는 전체 자바스크립트 묶음을 모든 페이지에 제공할 수 있는데, 이는 전체 자바스크립트가 첫 번째 페이지 로드할 때 다운로드되고,
이후 모든 페이지에서 캐시됨을 뜻합니다. 수많은 작은 혜택들이 늘어날 것입니다.

레일스 팀은 이런 스타일로 당신의 커피스크립트(자바스크립트 역시)를 작성할 것을 강력 권장합니다. 
그리고 많은 라이브러리들이 이 패턴을 따를 것을 당신은 기대할 수 있습니다.

[Built-in Helpers] 내장 헬퍼들
----------------------

레일스는 HTML을 생성함에 있어 당신을 돕기 위해 루비로 작성된 많은 뷰 헬퍼 메서드를 갖고 있습니다.
간혹 당신은 요소들에 약간의 Ajax를 추가하기를 원하고, 그러한 경우 레일스는 당신을 도와줄 것입니다.

겸손한 자바스크립트 때문에, 레일스의 "Ajax Helpers"는 사실 두 부분으로 되어 있습니다. 자바스크립트 부분과 루비 부분입니다.

[rails.js](https://github.com/rails/jquery-ujs/blob/master/src/rails.js)는 자바스크립트 부분을 제공하고,
표준 루비 뷰 헬퍼는 적절한 태그를 당신의 DOM에 추가합니다. rails.js 안의 CoffeeScript는 이들 속성을 수신하고 적절한 핸들러를 연결합니다.

### form_for

[`form_for`](http://api.rubyonrails.org/classes/ActionView/Helpers/FormHelper.html#method-i-form_for)는 form을 작성하는 것을 도와주는 헬퍼입니다.
`form_for`는 `:remote` 옵션을 가집니다. 이것은 다음과 같이 작동합니다.

```erb
<%= form_for(@post, remote: true) do |f| %>
  ...
<% end %>
```

이 코드는 다음과 같은 HTML을 생성합니다.

```html
<form accept-charset="UTF-8" action="/posts" class="new_post" data-remote="true" id="new_post" method="post">
  ...
</form>
```

`data-remote='true'` 부분을 참고하십시오. 이제 폼은 브라우저의 일반적은 전송 메커니즘 대신 Ajax에 의해 전송될 것입니다.

하지만 어쩌면 당신은 완성된 `<form>`을 앉아서 구경만 하고 싶지 않을지 모릅니다.
당신은 전송 성공시 뭔가를 하고 싶을 수 있습니다. 그렇게 하려면 `ajax:success` 이벤트를 연결하십시오.
실패시에는 `ajax:error`를 사용하십시오. 다음을 확인해 보십시오.

```coffeescript
$(document).ready ->
  $("#new_post").on("ajax:success", (e, data, status, xhr) ->
    $("#new_post").append xhr.responseText
  ).bind "ajax:error", (e, xhr, status, error) ->
    $("#new_post").append "<p>ERROR</p>"
```

분명 당신은 그보다 좀더 정교해지기를 원할 것입니다. 그러나 이것이 시작입니다.

### form_tag

[`form_tag`](http://api.rubyonrails.org/classes/ActionView/Helpers/FormTagHelper.html#method-i-form_tag)는 `form_for`와 아주 유사합니다.
이는 `:remote` 옵션을 가지고 있는데, 이것은 다음과 같이 사용할 수 있습니다.

```erb
<%= form_tag('/posts', remote: true) %>
```

다른 것들은 `form_for`와 같습니다. 세부사항 확인을 위해서는 문서를 확인하십시오.

### link_to

[`link_to`](http://api.rubyonrails.org/classes/ActionView/Helpers/UrlHelper.html#method-i-link_to)는 링크를 생성하는 것을 돕는 헬퍼입니다.
이것은 `:remote` 옵션을 가지고 있는데, 다음과 같이 사용할 수 있습니다.

```erb
<%= link_to "a post", @post, remote: true %>
```

이것은 다음 코드를 생성합니다.

```html
<a href="/posts/1" data-remote="true">a post</a>
```

당신은 `form_for`에서와 같은 Ajax 이벤트를 연결할 수 있습니다. 여기 예제가 있습니다. 
단 한번의 클릭으로 삭제될 수 있는 포스트의 목록이 있다고 가정해 봅시다. 다음과 같이 HTML을 생성합니다.

```erb
<%= link_to "Delete post", @post, remote: true, method: :delete %>
```

그리고 약간의 커피스크립트를 다음과 같이 작성합니다.

```coffeescript
$ ->
  $("a[data-remote]").on "ajax:success", (e, data, status, xhr) ->
    alert "The post was deleted."
```

### button_to

[`button_to`](http://api.rubyonrails.org/classes/ActionView/Helpers/UrlHelper.html#method-i-button_to)는 버튼을 생성하는 것을 돕는 헬퍼입니다.
이것은 `:remote` 옵션을 갖고 있으며 다음과 같이 호출할 수 있습니다.

```erb
<%= button_to "A post", @post, remote: true %>
```

이것은 다음 코드를 생성합니다.

```html
<form action="/posts/1" class="button_to" data-remote="true" method="post">
  <div><input type="submit" value="A post"></div>
</form>
```

이것은 단지 `<form>`이기 때문에, `form_for`에 있는 모든 정보 역시 적용됩니다.

[Server-Side Concerns] 서버측 고려사항들
--------------------

Ajax는 단지 클라이언트측 코드가 아닙니다. 당신은 Ajax를 지원하기 위해 서버측에도 몇 가지 작업을 해야 합니다.
사람들은 간혹 Ajax 요청을 하면서 HTML보다는 JSON을 돌려받기를 원합니다. 그렇게 하기 위해 필요한 것을 논의해 보겠습니다.

### 간단한 예제

당신이 보여주고자 하는 일련의 사용자 목록을 갖고 있으며 같은 페이지에서 새로운 사용자를 만드는 폼을 제공한다고 해 보겠습니다.
당신의 컨트롤러의 인덱스 액션은 다음과 같을 것입니다.

```ruby
class UsersController < ApplicationController
  def index
    @users = User.all
    @user = User.new
  end
  # ...
```

인덱스 뷰 (`app/views/users/index.html.erb`)는 다음 내용을 포함합니다.

```erb
<b>Users</b>

<ul id="users">
<% @users.each do |user| %>
  <%= render user %>
<% end %>
</ul>

<br>

<%= form_for(@user, remote: true) do |f| %>
  <%= f.label :name %><br>
  <%= f.text_field :name %>
  <%= f.submit %>
<% end %>
```

`app/views/users/_user.html.erb` 파셜은 다음 내용을 포함합니다.

```erb
<li><%= user.name %></li>
```

인덱스 페이지의 상단 부분은 사용자를 표시합니다. 하단 부분은 새로운 사용자를 생성하는 폼을 제공합니다.

하단 폼은 Users 컨트롤러에 있는 create 액션을 호출할 것입니다.
폼의 remote 옵션이 true로 되어 있기 때문에, 요청은 자바스크립트를 찾아 Ajax 요청으로 사용자 컨트롤러에 전송될 것입니다.
그 요청을 처리하기 위한 컨트롤러의 create 액션은 다음과 같을 것입니다.

```ruby
  # app/controllers/users_controller.rb
  # ......
  def create
    @user = User.new(params[:user])

    respond_to do |format|
      if @user.save
        format.html { redirect_to @user, notice: 'User was successfully created.' }
        format.js   {}
        format.json { render json: @user, status: :created, location: @user }
      else
        format.html { render action: "new" }
        format.json { render json: @user.errors, status: :unprocessable_entity }
      end
    end
  end
```

`respond_to` 블록에 있는 format.js를 주목하십시오. 이것은 컨트롤러가 당신의 Ajax 요청에 응답할 수 있도록 합니다.
다음으로 그에 상응하는 뷰파일 `app/views/users/create.js.erb`이 있는데, 이것은 클라이언트측에 전송되고 실행될 실제 자바스크립트 코드를 생성합니다.

```erb
$("<%= escape_javascript(render @user) %>").appendTo("#users");
```

Turbolinks
----------

레일스 4는 [Turbolinks gem](https://github.com/rails/turbolinks)를 포함하여 배포됩니다.
이 젬은 대부분의 응용프로그램에서 페이지 렌더링 속도를 높이기 위해 Ajax를 사용합니다.

### Turbolinks는 어떻게 작동하는가

Turbolinks는 페이지에 있는 모든 `<a>`에 클릭 핸들러를 연결합니다. 만약 당신의 브라우저가 [PushState](https://developer.mozilla.org/en-US/docs/DOM/Manipulating_the_browser_history#The_pushState(\).C2.A0method)를 지원하는 것이라면,
Turbolinks는 Ajax 요청을 만들고, 응답을 분석하고, 응답에 있는 `<body>` 내용으로 페이지상의 `<body>` 전체를 바꿔줍니다.
그 다음으로, PushState를 이용하여 URL을 올바른 것으로 변경하는데, 이는 새로고침 의미를 유지하고 예쁜 URL을 제공하기 위함입니다.

Turbolinks를 사용하기 위해 해야할 일은 당신의 Gemfile에 Turbolinks를 포함하고, 커피스크립트 manifest에 `//= require turbolinks`를 넣는 것입니다.
커피스크립트 manifest는 보통 `app/assets/javascripts/application.js` 안에 있습니다.

만약 특정 링크에 Turbolinks 지원을 하지 않으려면 태그의 속성에 `data-no-turbolink`를 추가하십시오.

```html
<a href="..." data-no-turbolink>No turbolinks here</a>.
```

### 페이지 변경 이벤트(Page Change Events)

커피스크립트를 작성할 때, 페이지 로드시 몇 가지 절차를 수행하고 싶을 때가 있습니다. jQuery로 다음과 같은 코드를 작성할 수 있습니다.

```coffeescript
$(document).ready ->
  alert "page has loaded!"
```

하지만, Turbolinks는 일반적인 페이지의 로딩 절차를 오버라이드하기 때문에 이에 의존하는 이벤트가 발생하지 않을 것입니다.
만일 그러한 코드가 있아면, 아래와 같이 코드를 변경해야 합니다. 

```coffeescript
$(document).on "page:change", ->
  alert "page has loaded!"
```

연결하고자 하는 다른 이벤트들을 포함한 보다 상세한 내용은 [the Turbolinks README](https://github.com/rails/turbolinks/blob/master/README.md)를 참고하십시오.

[Other Resources] 기타 리소스
---------------

여기 더 많은 내용 학습에 유용한 링크들이 있습니다.

* [jquery-ujs wiki](https://github.com/rails/jquery-ujs/wiki)
* [jquery-ujs list of external articles](https://github.com/rails/jquery-ujs/wiki/External-articles)
* [Rails 3 Remote Links and Forms: A Definitive Guide](http://www.alfajango.com/blog/rails-3-remote-links-and-forms/)
* [Railscasts: Unobtrusive JavaScript](http://railscasts.com/episodes/205-unobtrusive-javascript)
* [Railscasts: Turbolinks](http://railscasts.com/episodes/390-turbolinks)