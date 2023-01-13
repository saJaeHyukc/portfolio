# :pushpin: 가까? 마까?
>제주도 맛집 추천 플랫폼  
>https://www.gaggamagga.shop  

</br>

## 1. 제작 기간 & 참여 인원 & 맡은 역할
- 2022년 12월 1일 ~ 12월 29일
- 팀 프로젝트 (팀장)
<details>
<summary >맡은 역할</summary>
<div markdown="1">

- 유저관리, 프로필, 개인설정 및 추가기능
- User 테스트 코드
- Docker, AWS 배포
- CI/CD 구축
- 코드 리팩토링 / Swagger 적용
- 맡은 기능 프론트 연동

</div>
</details>

</br>

## 2. 사용 기술
#### `Back-end`
  - Python 3.10.8
  - Django 4.1.3
  - DRF 3.14.0
  - Django simple JWT 5.2.2
  - Django Channel 4.0.0
  - Celery 5.2.7
#### `Database`
  - PostgreSQL 14.5
#### `Infra`
  - AWS EC2
  - AWS Route 53
  - AWS CloudFront
  - AWS S3
  - Docker 20.10.12
  - Docker Compose 2.11.2
  - Gunicorn
  - Nginx 1.23.2
  - Daphne
  - Redis 7.0.7
  - Rabbit MQ 3.11
  - Github Action
#### `Front-end`
  - Vanilla JS
  - Element UI
#### `Management`
  - Notion
  - Github
  - Slack

</br>

## 3. 핵심 기능
- 사용자 환경(회원가입, 로그인, 회원정보 관리, 팔로우, 비활성화, 아이디/비밀번호 찾기 등등)
- 맛집 후기(리뷰) 작성/수정/삭제, 조회수 카운트, 좋아요, 검색 기능
- 후기 댓글 작성/수정/삭제
- 후기 댓글의 대댓글 작성/수정/삭제 기능
- 유저간 댓글 알림 기능

<br>

## 4. [ERD 설계](https://www.erdcloud.com/d/RvXb4PCLq3t3CPb3e)
![ex_screenshot](./img/erd.png)

<br>

## 5. API 설계 | [Swagger API Docs](https://www.back-gaggamagga.shop)
<details>
<summary style="font-size: 15px;"><b>USER API</b></summary>
<div markdown="1">

![ex_screenshot](./img/user_api.png)

</div>
</details>


<details>
<summary style="font-size: 15px;"><b>PLACE API</b></summary>
<div markdown="1">

![ex_screenshot](./img/place_api.png)

</div>
</details>

<details>
<summary style="font-size: 15px;"><b>REVIEW API</b></summary> 
<div markdown="1">

![ex_screenshot](./img/review_api.png)

</div>
</details>


<details>
<summary style="font-size: 15px;"><b>NOTIFICATION API</b></summary>
<div markdown="1">

![ex_screenshot](./img/notification_api.png)

</div>
</details>
<br>

## 6. Architecture
![ex_screenshot](./img/arch.png)

</br>

## 7. 핵심 트러블 슈팅
### 7.1. Email 전송 속도 향상(비동기 처리)
- 문제: 인증 이메일 전송 속도가 느림
- 문제의 원인: 이메일 전송은 동기로 처리하기에 요청을 보내고 응답을 받을 때까지 기다림

<details>
<summary><b>기존 코드</b></summary>
<div markdown="1">

~~~python
def send_email(message):
    email = EmailMessage(subject=message["email_subject"], body=message["email_body"], to=[message["to_email"]])
    email.send()
~~~
</div>
</details>

- 해결: 메시지 브로커 Rabbit MQ 와 Celery 활용으로 이메일 전송 비동기 처리 4.3s -> 4 ms 개선 

<details>
<summary><b>개선된 코드</b></summary>
<div markdown="1">

~~~python
# users/utils.py
message = {
          "email_body": email_body,
          "to_email": user.email,
          "email_subject": "이메일 인증",
       }
send_email.delay(message)

# users/task.py
from __future__ import absolute_import, unicode_literals

from celery import shared_task

from django.core.mail.message import EmailMessage

@shared_task
def send_email(message):
    email = EmailMessage(subject=message["email_subject"], body=message["email_body"], to=[message["to_email"]])
    email.send()

~~~

</div>
</details>

<br>

### 7.2. 네이버 SMS 401 에러
- 문제: 아이디 찾기 기능 구현 중 naver sms api에 요청을 보냈을 때 401 에러 발생

![ex_screenshot](./img/sms_error.png)
- 문제의 원인: 
1. 요청했을 때 콘솔창으로 네트워크 부분을 확인. 
2. signature-v2부분이 암호화가 되어 값이 바뀜. 
3. 요청보냈을 때 암호화를 하지않고 보낸 것이 원인

<details>
<summary><b>기존 코드</b></summary>
<div markdown="1">

~~~python
def send_sms(self):
    timestamp = str(int(time.time() * 1000))
    access_key =  get_secret("NAVER_ACCESS_KEY_ID")
    secret_key = get_secret("NAVER_SECRET_KEY")
    service_id = get_secret("SERVICE_ID")
    method = "POST"
    uri = f"/sms/v2/services/{service_id}/messages"
    message = method + " " + uri + "\n" + timestamp + "\n" + access_key
    message = bytes(message, "UTF-8")


headers = {
            "Content-Type": "application/json; charset=utf-8",
            "x-ncp-apigw-timestamp": timestamp,
            "x-ncp-iam-access-key": access_key,
            "x-ncp-apigw-signature-v2": secret_key,
        }
~~~

</div>
</details>

-  해결: nabver sms api docs를 확인 후 x-ncp-apigw-signature-v2에서 HMAC 암호화 알고리즘은 HmacSHA256 사용을 파악하여 암호화가 된 시크릿키를 보내어 개선

<details>
<summary><b>개선된 코드</b></summary>
<div markdown="1">

~~~python
def send_sms(self):
    timestamp = str(int(time.time() * 1000))
    access_key =  get_secret("NAVER_ACCESS_KEY_ID")
    secret_key = bytes( get_secret("NAVER_SECRET_KEY"), "UTF-8")
    service_id = get_secret("SERVICE_ID")
    method = "POST"
    uri = f"/sms/v2/services/{service_id}/messages"
    message = method + " " + uri + "\n" + timestamp + "\n" + access_key
    message = bytes(message, "UTF-8")
    signing_key = base64.b64encode(
        hmac.new(secret_key, message, digestmod=hashlib.sha256).digest()
    )

headers = {
            "Content-Type": "application/json; charset=utf-8",
            "x-ncp-apigw-timestamp": timestamp,
            "x-ncp-iam-access-key": access_key,
            "x-ncp-apigw-signature-v2": signing_key,
        }
~~~

</div>
</details>

<br>

### 7.3. 토큰 인증 에러
- 문제: client에서 서비스 이용 시 일정 시간이 지나면 개인 정보 undefined이 뜸
- 문제의 원인: access token이 만료되어 데이터베이스 접근이 불가한 것을 파악
- 해결: 
1. refresh token으로 access token을 발급 하지만 refresh 토큰이 유효한지 확인해주는 로직 필요 
2. simple jwt에서 verify token 로직 존재 유효한 토큰일 경우 200 유효하지 않을 경우 401 반환
3. status code를 기준으로 프론트에서 요청보내어 로직 구현

<details>
<summary><b>개선된 코드</b></summary>
<div markdown="1">

~~~python
#urls.py
    path("api/token/", views.CustomTokenObtainPairView.as_view(), name="token_obtain_pair_view"),
    path("api/token/refresh/", TokenRefreshView.as_view(), name="token_refresh_view"),
    path("api/token/verify/", TokenVerifyView.as_view(), name="token_verify"),
~~~

~~~ javascript
// Access token verify Logic
async function access_verify_token() {

    const response = await fetch(
        `${backendBaseUrl}/users/api/token/verify/`,
        { 
            headers: {
                'content-type': 'application/json'
            },
            method: 'POST',
            body: JSON.stringify({"token": localStorage.getItem("access")})
        }
    )
    if (response.status === 200) { 

    }
    if (response.status === 401){
        refresh_verify_token()
        
    }
}

// Refresh token verify Logic
async function refresh_verify_token() {

    const response = await fetch(
        `${backendBaseUrl}/users/api/token/verify/`,
        { 
            headers: {
                'content-type': 'application/json'
            },
            method: 'POST',
            body: JSON.stringify({"token": localStorage.getItem("refresh")})
        }
    )
    if (response.status === 200) { 
        access_token_get()
    }
    if (response.status === 401){
        localStorage.clear()
        window.location.reload()
    }
}

// Access token get Logic
async function access_token_get() {

    const response = await fetch(
        `${backendBaseUrl}/users/api/token/refresh/`,
        { 
            headers: {
                'content-type': 'application/json'
            },
            method: 'POST',
            body: JSON.stringify({"refresh": localStorage.getItem("refresh")})
        }
    )

    const response_json = await response.json()

    if (response.status === 200) {

    localStorage.removeItem("access")
    localStorage.removeItem("payload")
    localStorage.setItem("access", response_json.access); 

    const base64Url = response_json.access.split('.')[1];
    const base64 = base64Url.replace(/-/g, '+').replace(/_/g, '/');
    const jsonPayload = decodeURIComponent(
        atob(base64).split('').map(function (c) {
            return '%' + (
                '00' + c.charCodeAt(0).toString(16)
            ).slice(-2);
        }).join('')
    );
    localStorage.setItem("payload", jsonPayload);
    window.location.reload()
}}
~~~

</div>
</details>
</br>


## 8. 피드백 반영
### 8.1. 비밀번호 변경 시 인증
- 피드백 내용: 비밀번호 변경 시 개인정보가 보호받지 못하는 느낌입니다. 기존 비밀번호를 입력받아 확인하는 절차가 추가되면 좋을 것 같습니다.

<details>
<summary><b>기존 코드</b></summary>
<div markdown="1">

~~~python
#serializer.py
def validate(self, data):
    password = data.get("password")
    repassword = data.get("repassword")
~~~

</div>
</details>

- 피드백 반영: 해쉬 값을 확인하는 check_password 메소드를 활용 후 기능 구현.

<details>
<summary><b>개선된 코드</b></summary>
<div markdown="1">

~~~python
#serializer.py
confirm_password = serializers.CharField(
    error_messages={
        "required": "비밀번호를 입력해주세요.",
        "blank": "비밀번호를 입력해주세요.",
        "write_only": True,
    }
)

def validate(self, data):
    current_password = self.context.get("request").user.password
    confirm_password = data.get("confirm_password")
    password = data.get("password")
    repassword = data.get("repassword")

    # 현재 비밀번호 예외 처리
    if not check_password(confirm_password, current_password):
        raise serializers.ValidationError(detail={"password": "현재 비밀번호가 일치하지 않습니다."})
~~~

</div>
</details>

<br>


### 8.2. 자신이 작성한 게시글 신고됨
- 피드백 내용: 작성자 게시글에 작성자가 신고할 수 있어요

<details>
<summary><b>기존 코드</b></summary>
<div markdown="1">

~~~python
#views.py
def post(self, request, place_id, review_id):
    review_author = get_object_or_404(Review, id=review_id).author
    try:
        Report.objects.get(author=request.user.id, review=review_id)
        return Response({"message": "이미 신고를 한 리뷰입니다."}, status=status.HTTP_208_ALREADY_REPORTED)

    except Report.DoesNotExist:
        serializer = ReportSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save(author=request.user, review_id=review_id)
            return Response({"message": "신고가 완료되었습니다."}, status=status.HTTP_200_OK)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
~~~

</div>
</details>

- 피드백 반영: 요청 들어오는 유저와 작성자와 비교 후 400 status code 반환으로 해결

<details>
<summary><b>개선된 코드</b></summary>
<div markdown="1">

~~~python
#views.py
def post(self, request, place_id, review_id):
    review_author = get_object_or_404(Review, id=review_id).author
    if review_author == request.user:
        return Response({"message": "작성자는 신고를 할 수 없습니다."}, status=status.HTTP_400_BAD_REQUEST)

    try:
        Report.objects.get(author=request.user.id, review=review_id)
        return Response({"message": "이미 신고를 한 리뷰입니다."}, status=status.HTTP_208_ALREADY_REPORTED)

    except Report.DoesNotExist:
        serializer = ReportSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save(author=request.user, review_id=review_id)
            return Response({"message": "신고가 완료되었습니다."}, status=status.HTTP_200_OK)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
~~~

</div>
</details>

<br>

### 8.3. IP 주소 차단 기능
- 피드백 내용: IP 주소 차단같은 기능이 있으면 좋을 것 같아요
- 피드백 반영: IP 정보를 알 수 있는 API를 활용하여 해당 나라 IP일 경우 차단되도록 기능 구현

<details>
<summary><b>개선된 코드</b></summary>
<div markdown="1">

~~~python
#utils.py
def get_client_ip(request):
    x_forwarded_for = request.META.get("HTTP_X_FORWARDED_FOR")
    if x_forwarded_for:
        ip = x_forwarded_for.split(",")[0]
    else:
        ip = request.META.get("REMOTE_ADDR")
    return ip

def find_ip_country(user_ip):
    serviceKey = get_secret("WHOIS_KEY")
    url = "http://apis.data.go.kr/B551505/whois/ip_address?serviceKey=" + serviceKey + "&query=" + user_ip + "&answer=json"
    request = urllib.request.urlopen(url).read().decode("utf-8")
    return dict(eval(request))["response"]["whois"]["countryCode"]

#jwt_claim_serializer.py
user_ip = Util.get_client_ip(self.context.get("request"))
country = Util.find_ip_country(user_ip)
if BlockedCountryIP.objects.filter(user=self.target_user, country=country).exists():
    raise serializers.ValidationError(detail={"error": "해당 IP를 차단한 계정입니다."})
~~~

</div>
</details>

<br>

## 9. 회고 / 느낀점 / 현황판 / 그 외 트러블 슈팅
>프로젝트 개발 회고 글: https://bolder-starburst-a73.notion.site/221229-509674920cc44056b5a06ab88d2c4f73
<br>

>프로젝트 현황판 / 그 외 트러블 슈팅: https://bolder-starburst-a73.notion.site/060c8fd4af5845df8770441ef69bdaf5
