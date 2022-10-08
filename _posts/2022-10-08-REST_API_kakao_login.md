---
title : 카카오 REST API 소셜 로그인 feat. Django
tags : python django kakao_social_login social_login
category : [Programming]
---
# REST API v2 카카오 소셜 로그인 가이드!(Feat. django)

출처 : [kakao developers doc](https://developers.kakao.com/docs/latest/ko/kakaologin/rest-api#before-you-begin-process)

[참고하면 괜찮을 블로그](https://velog.io/@devmin/kakao-login-basic)

## <span style= 'color : yellow;'>해야할 리스트(사전 준비)</span>
### 1. 카카오 developers에서 애플리케이션 추가하기
<img src="/images/kakao_login_step_1.png" width="600" />

### 2. 추가한 애플리케이션에 들어가서 앱키 받아놓기(앱키는 노출되지 않게 주의해 주세요!)
<img src="/images/kakao_login_step_2.png" width="600" />

### 3. 앱 설정의 플랫폼에 들어가 Web URL에 내 사이트 등록하기
<img src="/images/kakao_login_step_3.png" width="600" />

### 4. 필요한 정보를 동의항목에서 설정하기
<img src="/images/kakao_login_step_4.png" width="600" />

### 5. 카카오 로그인 활성화와 redirect URI 설정하기

<b style="font-size: 1.1em">5-1) 백 엔드에서 redierct uri를 받을 url설정하기
    - 처리를 원하는 url을 설정해 줍니다.</b>

```python
# urls.py
from django.urls import path
from user.views import kakao_social_login, kakao_social_login_callback

urlpatterns = [
    #로그인 요청을 보낼 url
    path('account/login/kakao/', kakao_social_login, name='kakao_login'),
    #받은 인가 코드로 접근 토근을 받아 유저의 정보를 가져올 url
    path('account/login/kakao/callback/', kakao_social_login_callback, name='kakao_login_callback'),
]


# views.py
def kakao_social_login(request):
    pass

def kakao_social_login_callback(request):
    pass

```

<b style="font-size: 1.1em">5-2) 프론트 엔드에서 버튼을 만들기!</b>

```html
<!--  로그인 버튼과 백엔드에서 설정해준 URL로 설정하기    -->
<a id="kakao-login-btn" href="/account/login/kakao">
    <img src="https://k.kakaocdn.net/14/dn/btroDszwNrM/I6efHub1SN5KCJqLm1Ovx1/o.jpg" width="222" alt="카카오 로그인 버튼"/>
</a>
```

<b style="font-size: 1.1em">5-3) 카카오 developers에서 로그인 활성화 & redirect uri 등록하기</b>

<img src="/images/kakao_login_step_5.png" width="600" />


    

## <span style= 'color : yellow;'>REST API로 인증 요청 및 접근 코드 받아서 사용자 정보 활용하기</span>
<img src="/images/kakao_login_step_6.png" width="600" />


<b style="font-size: 1.1em">저희가 처리해야할 부분은 4부분이 되겠습니다.</b>

### 1번 - /oauth/authorize 에 요청 보내기


```python
def kakao_social_login(request):
    """
    카카오톧에 나의 애플리케이션의 정보를 담아 사용자에게 카카오 로그인 요청
    """
    if request.method == 'GET':
        client_id = CLIENT_ID # 앱키 ex) b923j1i23k4io1l2k3ji1uaq2
        redirect_uri = 'http://127.0.0.1:8000/account/login/kakao/callback' # 인가 코드를 받을 uri
        return redirect(
            f'https://kauth.kakao.com/oauth/authorize?client_id={client_id}&redirect_uri={redirect_uri}&response_type=code'
        )
```

### 2번 Redirect URI로 전달된 인가 코드를 받기

`pip install requests`

```python
def kakao_social_login_callback(request):
    """
    받은 인가 코드, 애플리케이션 정보를 담아 /oath/token/에 post요청하여 접근코드를 받아 처리하는 함수
    """
    try:
        code = request.GET.get('code')
#         client_id = CLIENT_ID
#         redirect_uri = 'http://127.0.0.1:8000/account/login/kakao/callback' # 인가 코드를 받은 URI
#         token_request = requests.post(
#             'https://kauth.kakao.com/oauth/token', {'grant_type': 'authorization_code',
#                                                     'client_id': client_id, 'redierect_uri': redirect_uri, 'code': code}
#         )
        
#         token_json = token_request.json()

#         error = token_json.get('error', None)

#         if error is not None:
#             print(error)
#             return JsonResponse({"message": "INVALID_CODE"}, status=400)

#         access_token = token_json.get("access_token")

#     except KeyError:
#         return JsonResponse({"message": "INVALID_TOKEN"}, status=400)

#     except access_token.DoesNotExist:
#         return JsonResponse({"message": "INVALID_TOKEN"}, status=400)

#         #------get kakaotalk profile info------#

#     profile_request = requests.get(
#         "https://kapi.kakao.com/v2/user/me", headers={"Authorization": f"Bearer {access_token}"},
#     )
#     profile_json = profile_request.json()
#     kakao_id = profile_json.get('id')
#     username = profile_json['properties']['nickname']
```

### 3번 /oauth/token에 인가 코드를 담아 접근 토큰 받기

```python
def kakao_social_login_callback(request):
    """
    받은 인가 코드, 애플리케이션 정보를 담아 /oath/token/에 post요청하여 접근코드를 받아 처리하는 함수
    """
    try:
        code = request.GET.get('code')
        client_id = CLIENT_ID # 앱 키
        redirect_uri = 'http://127.0.0.1:8000/account/login/kakao/callback' # 인가 코드를 받은 URI, 이 부분은 크게 상관이 없어보입니다.
        token_request = requests.post(
            # grant_type은 authorization_code로 고정입니다.
            'https://kauth.kakao.com/oauth/token', {'grant_type': 'authorization_code',
                                                    'client_id': client_id, 'redierect_uri': redirect_uri, 'code': code}
        )
        
        token_json = token_request.json()

#         error = token_json.get('error', None)

#         if error is not None:
#             print(error)
#             return JsonResponse({"message": "INVALID_CODE"}, status=400)

#         access_token = token_json.get("access_token")

#     except KeyError:
#         return JsonResponse({"message": "INVALID_TOKEN"}, status=400)

#     except access_token.DoesNotExist:
#         return JsonResponse({"message": "INVALID_TOKEN"}, status=400)

#         #------get kakaotalk profile info------#

#     profile_request = requests.get(
#         "https://kapi.kakao.com/v2/user/me", headers={"Authorization": f"Bearer {access_token}"},
#     )
#     profile_json = profile_request.json()
#     kakao_id = profile_json.get('id')
#     username = profile_json['properties']['nickname']
```

### 4번 토큰 유효성 검증 후 토큰으로 사용자의 정보 조회와 서비스 내에서 처리

```python
def kakao_social_login_callback(request):
    """
    받은 인가 코드, 애플리케이션 정보를 담아 /oath/token/에 post요청하여 접근코드를 받아 처리하는 함수
    """
    try:
        code = request.GET.get('code')
        client_id = CLIENT_ID # 앱 키
        redirect_uri = 'http://127.0.0.1:8000/account/login/kakao/callback' # 인가 코드를 받은 URI
        token_request = requests.post(
            'https://kauth.kakao.com/oauth/token', {'grant_type': 'authorization_code',
                                                    'client_id': client_id, 'redierect_uri': redirect_uri, 'code': code}
        )
        
        token_json = token_request.json()
        #------------유효성 검증 --------------#
        error = token_json.get('error', None)

        if error is not None:
            print(error)
            return JsonResponse({"message": "INVALID_CODE"}, status=400)
        #-------------받은 토큰---------------#
        access_token = token_json.get("access_token")

    except KeyError:
        return JsonResponse({"message": "INVALID_TOKEN"}, status=400)

    except access_token.DoesNotExist:
        return JsonResponse({"message": "INVALID_TOKEN"}, status=400)

    #------토큰을 이용하여 사용자 정보 조회------#
    profile_request = requests.get(
        "https://kapi.kakao.com/v2/user/me", headers={"Authorization": f"Bearer {access_token}"},
    )
    #------사용자 정보를 활용---------------#
    profile_json = profile_request.json()
    print(profile_json)
    kakao_id = profile_json.get('id')
    username = profile_json['properties']['nickname']
```
