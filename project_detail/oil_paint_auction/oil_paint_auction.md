# :pushpin:유화 경매 프로젝트
>유화 제작 경매 서비스  
 

</br>

## 1. 제작 기간 & 참여 인원 & 맡은 역할
- 2022년 11월 22일 ~ 11월 28일
- 팀 프로젝트 (5명)
<details>
<summary >맡은 역할</summary>
<div markdown="1">

- 유저 관리 및 추가 기능
- 포인트 적립, 사용 기능
- User 테스트코드 

</div>
</details>

</br>

## 2. 사용 기술
#### `Back-end`
  - Python 3.10.7
  - Django 4.1.3
  - DRF 3.14.0
  - Django simple JWT 5.2.2
#### `Database`
  - SQLite
#### `Front-end`
  - Vanilla JS
  - Element UI
#### `Management`
  - Notion
  - Github
  - Slack

</br>

## 3. 핵심 기능
- 사용자 환경(회원가입, 로그인, 회원정보 관리 등)
- 유화 작품 생성, 수정, 삭제 기능 구현(사진 업로드, 유화 스타일 선택/적용 등)
- 나의 유화 작품 경매 등록, 삭제 기능 구현
- 포인트 적립, 사용 기능 구현
- 댓글 생성, 수정, 삭제 기능 구현

<br>

## 4. [ERD 설계](https://www.erdcloud.com/d/pqrad25r55RSeqcYS)
![ex_screenshot](./img/erd.png)

<br>

## 5. API 설계 
<details>
<summary style="font-size: 18px;"><b>USER API</b></summary>
<div markdown="1">

![ex_screenshot](./img/API1.PNG)
![ex_screenshot](./img/API2.PNG)
![ex_screenshot](./img/API3.PNG)
![ex_screenshot](./img/API4.PNG)

</div>
</details>


<details>
<summary style="font-size: 18px;"><b>PAINTING API</b></summary>
<div markdown="1">

![ex_screenshot](./img/API5.PNG)
![ex_screenshot](./img/API6.PNG)

</div>
</details>

<details>
<summary style="font-size: 18px;"><b>AUCTION API</b></summary> 
<div markdown="1">

![ex_screenshot](./img/API7.PNG)
![ex_screenshot](./img/API8.PNG)
![ex_screenshot](./img/API9.PNG)
![ex_screenshot](./img/API10.PNG)
![ex_screenshot](./img/API11.PNG)

</div>
</details>

<br>

## 6. 트러블 슈팅
<details>
<summary style="font-size:18px"><b>6.1. Unicode Error</b></summary>
<div markdown="1">
- JSON으로 서버에있는 사진을 불러오려면 사진 파일을 불러오는 것인줄 알았습니다. image를 불러오려고 하니 unicode error가 발생했습니다

<br>

<details>
<summary><b>기존 코드</b></summary>
<div markdown="1">

~~~python
#serializer.py
def get_author_profile_image(self, obj):
        return obj.author.profile_image
~~~

</div>
</details>

- 서버에 저장되어있는 파일을 그대로 가져오는 것이 아닌 url을 가져오는 것을 알고 수정했습니다.

<br>

<details>
<summary><b>개선된 코드</b></summary>
<div markdown="1">

~~~python
def get_author_profile_image(self, obj):
        return obj.author.profile_image.url
~~~

</div>
</details>
</div>
</details>

<br>

<details>
<summary style="font-size:18px"><b>6.2. Byte error</b></summary>
<div markdown="1">
- 비밀번호 찾기를 기능 중에 uidb64와 token을 대조하여 구현하려고 했습니다. user id를 urlsafe_base64_encode 메소드를 사용하려고 했으나 byte type error가 발생했습니다.

<br>

<details>
<summary><b>기존 코드</b></summary>
<div markdown="1">

~~~python
uidb64 = urlsafe_base64_encode(user.id)
~~~

</div>
</details>

- byte로 변환하기 위해서 smart_byte라는 메소드를 사용하여 변환하였습니다. 또한 반대로 byte를 str값으로 변환하려면 decode한 다음 force_str 메소드를 사용해주었습니다.

<br>

<details>
<summary><b>개선된 코드</b></summary>
<div markdown="1">

~~~python
# encode
uidb64 = urlsafe_base64_encode(smart_bytes(user.id))

# decode
user_id = force_str(urlsafe_base64_decode(uidb64))
~~~

</div>
</details>

</div>
</details>
<br>

## 7. 회고 / 느낀점 / 현황판 / 그 외 트러블 슈팅
>프로젝트 개발 회고 글: https://bolder-starburst-a73.notion.site/221128-9950cd25bced4f6e8e71092ddf86f889
<br>
>프로젝트 현황판 / 그 외 트러블 슈팅: https://bolder-starburst-a73.notion.site/c4e68e7765b64fd3bef2fb0071fb8996