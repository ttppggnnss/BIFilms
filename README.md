# BI Films

## 🚩 배포 서버 URL

[BI Films](http://deploy-vue.s3-website.ap-northeast-2.amazonaws.com/) <= 클릭하면 사이트로 이동합니다.



## 🤵​ 팀원 정보 및 업무 분담 내역

### 김성민(Bradley) - 기본적으로 장고, 뷰 함께 구현 + 모델링 관련

### 김세훈(Isaac) - 기본적으로 장고, 뷰 함께 구현 + 구현 관련



## :bulb: 목표 서비스 구현 및 실제 구현 정도

### 필수기능 제외한 목표 서비스

​		[개인적인 점수]

- [9점] 페이지네이션

  데이터를 분할해서 가져오지 않고 한번에 가져오는 것이 아쉬웠다. 그래도 방법을 찾아서 구현했다.

- [8점] 프로필 페이지 구현

  유저 팔로우 기능을 미구현 한 것이 아쉽다. 또 보여줄 데이터가 많은데 페이지네이션을 어떻게 활용해야할 지 고민이 더 필요하다

- [8점] 유저 관리 페이지 구현

  관리자가 회원 탈퇴 까지는 좋은데 다 구현하고 나서 경고 카드 (Yellow Card) 같은 기능을 구현하지 못한 것이 아쉬웠다.

- [8점] 영화 좋아요 구현

  로그인 하지 않은 경우 좋아요가 보이지 않는 것이 아쉽다.




## 🛠 데이터베이스 모델링(ERD)

영화 / 평점  <=> 유저 <=> 게시글 / 댓글

```python
### movies/models.py
from django.db import models
from django.conf import settings
# Create your models here.

class Genre(models.Model):
    name = models.CharField(max_length=100)

class Movie(models.Model):
    title = models.CharField(max_length=100)
    original_title = models.CharField(max_length=100, null=True)
    release_date = models.DateField()
    popularity = models.FloatField(null=True)
    vote_count = models.IntegerField(null=True)
    vote_average = models.FloatField()
    adult = models.BooleanField(null=True)
    overview = models.TextField()
    original_language = models.CharField(max_length=100, null=True)
    poster_path = models.CharField(max_length=100, null=True)
    backdrop_path = models.CharField(max_length=100, null=True)
    genres = models.ManyToManyField(Genre, related_name = 'movies')
    like_users = models.ManyToManyField(settings.AUTH_USER_MODEL, related_name='like_movies', blank=True)

class Rank(models.Model):
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    movie = models.ForeignKey(Movie, related_name='ranks', on_delete = models.CASCADE)
    score = models.IntegerField()
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

```python
## accounts/models.py
from django.db import models
from django.contrib.auth.models import AbstractUser
from movies.models import Movie

class User(AbstractUser):
    rank_movie = models.ManyToManyField(
      Movie,
      related_name='rank_user',
    )
```

```python
## articles/models.py
from django.db import models
from django.conf import settings


class Article(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)

class Comment(models.Model):
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    article = models.ForeignKey(Article, related_name='comments', on_delete = models.CASCADE)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```



## ⚙ 필수 기능

### 1. 관리자 뷰

관리자로 로그인 하는 경우에만 영화 등록 / 수정 / 삭제 버튼이 보이게 했다. 또 키값을 확인해서 관리자 키인 경우에만 장고에서 실행되도록 구현했다.

관리자로 로그인 하는 경우에만 유저 관리 페이지에서 유저 탈퇴 권한을 갖는다

### 2. 영화 정보

djangp_pjt5 에서 사용한 영화 데이터를 활용하여 500개의 데이터를 구성했다.

로그인 한 유저만 평점 등록 / 수정 /삭제 및 좋아요를 구현했다.

### 3. 추천 알고리즘

평점 기반으로 자신이 평가한 평균 평점 이상의 영화 중 좋아요한 영화에서 하나를 뽑고 그 장르 중에서 하나를 뽑아서 평가하지 않고 좋아요 하지 않은 영화 5개를 골라준다.

### 4. 커뮤니티

로그인 한 사용자만 글을 조회 / 생성 할 수 있고 본인만 글을 수정 / 삭제 할 수 있다.

로그인 한 사용자만 게시글에 댓글을 작성할 수 있고 작성자 본인만 댓글을 수정 / 삭제 할 수 있다.

생성 및 수정 시각 정보를 포함한다.

게시글에도 페이지네이션을 구성했다.

### 5. 기타

5개 넘는 URL 페이지를 구성했다.

에러 페이지는 alert 를 활용해서 구현했다.

비동기 방식을 사용했다. 그런데 데이터가 500개가 넘고 한번에 가져오려하니 살짝 버벅거리는 것이 느껴진다.



## 🌈 기타 (느낀점)

크게 3 가지 정도로 느낀점을 요약할 수 있는 것 같다. 

1. 생각보다 시간이 많이 부족하다.

   평소에 하루 프로젝트에서도 많은 기능을 구현해서 많은 것을 할 수 있을 줄 알았는데 장고와 뷰 모두 하려니 생각하지도 못한 오류도 많았다. 처음에는 욕심을 부려서 이것도 넣어보고 저것도 넣어보고 했지만 이걸 구현하면 저게 오류가 생기고 저거 오류를 해결하면 이게 다시 문제가 되는 상황들이 많았다. 또 영화 데이터도 시간 남으면 api 활용해서 데이터를 가져와 보려 했지만 기존 데이터를 사용할 수 밖에 없어서 아쉬웠다.

   도메인(bifilms.xyz)을 구매했는데 배포할 때 등록된 도메인으로 바꾸는 것이 생각처럼 잘 안되었다 가격이 싸다고 호이스팅어를 사용했는데 자료가 부족했다. 다음번에는 자료를 찾기 쉬운 사이트를 활용해보아야겠다.

2. live share 사용 후기.

   live share 가 협업하기 좋은 것 같긴 하다. 하지만 둘이 같은 페이지에서 수정하다 보면 알 수 없는 에러가 간혹 발생한다. 원인을 찾아서 계속 해결하려 해도 해결 되지 않는데 live share 를 끄니 해결 되는 경우가 종종 있었다. 더 좋은 협업 툴이 있으면 좋을 것 같다.

3. 어디에나 방법은 있다.

   vue 에서는 페이지네이션을 따로 배우지 않아서 이걸 할 까 말까 고민을 많이 했다. 그런데 vue 에서도 pagination 을 썼을 거라고 생각했고 여러가지 방법을 찾아서 시도해보았지만 실패하다가 vuetify 라는 것을 추가해서 시도하여 성공할 수 있었다. 배우지 않은 것들을 찾아서 도전하는 경험이 (실패했으면 별로였겠지만 성공했으니까) 좋았던 것 같다.

git branch 를 활용하지 못한 것이 조금 아쉬웠다. 그래도 배포까지 성공할 수 있어서 좋은 경험이 된 것 같다.