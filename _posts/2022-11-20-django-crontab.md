---
title : django-crontab 일정 주기로 함수 실행하기
use_math : true
tags : python django django-crontab
category : [Programming, Django]

---

## django-crontab이란?
- 서버를 실행하고 주기적으로 함수를 실행해야할 경우 필요한 라이브러리이다.

1. django-crontab
`pip install django-crontab`

2. settings.py 앱 추가
settings.py
```python
INSTALLED_APPS = [
    .
    .
    'django_crontab',
    .
    .
]
```

3. 앱 하위에 cron.py 생성 후 원하는 함수 구현
```python
def crontab_hello():
    print('hello')
```

4. settings.py 설정하기
```python
CRONJOBS = [
    ('* * * * *', 'study_group.cron.crontab_penalty_student', '>> '+os.path.join(BASE_DIR, 'stady/log/cron.log')),
]
```
- 매분 실행하고 프로젝트/폴드/로그파일 경로를 설정하여 로그를 확인할 수 있다.
(crontab 시간 설정 웹 사이트)[https://crontab.guru/#*_*_*_*_*]