---
layout: post
title:  "텔레그램 봇을 만들어보자"
description: An example post which shows code rendering.
date:   2020-07-09 14:47:36 +0530
categories: Python Telegrambot
---

## 모니터링, 어떻게 하고 계신가요?

이런저런 배치와 데몬을 만들다보면 어느순간 잘 돌았는지 체크하는 것 자체가 부담이 되는 순간이 옵니다.

알람을 맞춰놓고 로그를 들여다 보자니 로그 파일 여는 것도 번거로와지고

창 하나에 로그 파일을 tail -f 해놓자니 화면이 부족해지고..

뭔가 뭔가 뭔가 내가 봐야 하는거만 딱 봤으면 좋겠는데 하는 생각이 들게 되죠.



이럴때는 텔레그램이 좋은 대안이 됩니다. 가령 뭔가 돌리고 있는데 에러가 나서 프로그램이 죽는다던지 했을때 바로 텔레그램이 오면 실시간으로 대응도 가능하죠.

이번 포스팅에서는 텔레그램봇을 모니터링에 활용 하는 방법을 공유해보겠습니다.


## 텔레그램 봇 만들기

텔레그램봇은 모든 텔레그램봇의 아버지 봇 @BotFather 를 이용해서 만들 수 있습니다.

![posting-creating-telegrambot-1.PNG](../assets/images/posting-creating-telegrambot-1.PNG)

BotFather 와 채팅창에서 봇생성, 봇이름 변경, command 추가 등의 작업을 할 수 있는데요, 일단 봇을 만들어보겠습니다.

![posting-creating-telegrambot-2.PNG](../assets/images/posting-creating-telegrambot-2.PNG)

위의 박스로 가려놓은 부분에 bot_token이 생성이 되고 텔레그램 REST API를 사용할때 이 bot_token을 사용하게 됩니다.

그리고 @mc_ex_bot 을 검색을 해보면 봇이 생성된것을 확인할 수 있습니다.

![posting-creating-telegrambot-2.PNG](../assets/images/posting-creating-telegrambot-3.PNG)


## 텔레그램 봇이 메세지를 보낼 방 알아내기

저희의 궁극적인 목표는 텔레그램으로 메세지를 보내려면 봇이 참여하고 있는 방이 필요하겠죠?

그리고 어떤 채팅방인지도 알아내야 합니다. 텔레그램에서는 chat_id 라고 합니다.

한번 봇과 단둘이 오붓하게 대화할 수 있는 개인방을 열어서 시작해봅시다.

![posting-creating-telegrambot-2.PNG](../assets/images/posting-creating-telegrambot-4.PNG)

이렇게 봇이 참가한 방에 오고가는 메세지는 [getUpdates](https://core.telegram.org/bots/api#getupdates)라는 REST API로 가져올수 있습니다.

텔레그램 봇 REST API의 기본 URL은 https://api.telegram.org/bot 뒤에 바로 위의 봇 토큰을 붙이시면 됩니다.

가령 봇 토큰이 "ABCD123456"라면 https://api.telegram.org/botabcd123456 이 되겠죠?

그리고 getUpdates API의 URL은 아래와 같이 됩니다.

https://api.telegram.org/bot예제지만그래도봇토큰은가려야지/getUpdates

뭐 API 문서를 보면 이런저런 파라미터가 필요하지만 지금은 그냥 파라미터 없이 한번 호출을 해보았습니다. 브라우저에 그냥 입력하시면 되요 ~

결과는 아래와 같이 나오네요.

```json
{
    "ok":true,
    "result":[
        ...
        {"update_id":129349147,
         "message":{
             "message_id":3,
             "from":{"id":53395910,"is_bot":false,"first_name":"\ubbfc\ucca0","last_name":"\uc2e0","username":"제텔레그램아이디가여기나와요","language_code":"ko"},
             "chat":{"id":53395910,"first_name":"\ubbfc\ucca0","last_name":"\uc2e0","username":"smanioso","type":"private"},
             "date":1594787860,
             "text":"where am I?"}
        },
        ...
]}
```

메세지를 3개를 보내서 3개가 떴는데, 가독성이 떨어지니까 하나만 뽑아왔습니다.

눈을 부릅뜨고 보면 "chat"이라는 key와 dict를 발견할 수 있습니다.

이 dict 바로 우리가 알고싶어하는 채팅방의 정보가 담겨 있습니다!

dict 안에 "id" 필드가 채팅방의 ID가 되겠습니다. (짧게 앞으론 chat_id라고 할게요)

예제에서 사용한 방의 ID는 53395910 이군요


## 텔레그램으로 메세지 보내기

자 이제 메세지를 한번 보내봅시다.

[sendMessage API](https://core.telegram.org/bots/api#sendmessage)를 이용하면 되는데요. 일단 브라우저에서 바로 한번 콜을 해보겠습니다.

![posting-creating-telegrambot-5.PNG](../assets/images/posting-creating-telegrambot-5.PNG)

chat_id에는 53395910을 넣고 text에는 보낼 메세지를 넣어 보냈고, 메세지가 잘 날아왔군요.

먼길을 왔군요. 이제는 코드에 적용할 일만 남았습니다!

메세지를 보내는 기본 코드를 후루룩 짜봤습니다. POST로 json 형태로 데이터를 보내는 예제입니다.

```python
import json

# pip installed packages
import requests

_BOT_TOKEN = '여기에는봇토큰이들어갑니다.'

_BASE_URL = 'https://api.telegram.org/bot{}'.format(_BOT_TOKEN)


HEADERS = {
    'Content-Type': 'application/json'
}


def send_message(chat_id, text):
    url = '/'.join([_BASE_URL, 'sendMessage'])
    data = {
        'chat_id': chat_id,
        'text': text,
    }

    res = requests.post(url, data=json.dumps(data), headers=HEADERS)
    print(res)
    res_json = json.loads(res.text)
    print(res_json)


if __name__ == '__main__':
    send_message(53395910, '파이썬 코드에서 보냄')
```

참 쉽죠?

## 모니터링시 쓸만한 몇가지 패턴

텔레그램을 사용해서 이런저런 메세지를 많이 받고 있는데요,

이것도 너무 많이 받다보면 무감각해져서 놓치는 경우가 생깁니다.

그래서 개인적으로는 대략적인 기준을 세워서 텔레그램 메세지를 꼭 필요한 경우에만 받습니다.

1. Exception이 발생하는 경우
1. 에러가 발생했는데 수동으로 작업이 필요한 경우
1. 실행결과를 반드시 확인 해야 하는경우

이 중 1번 Exception의 경우 try-except 문을 사용하여 보통 처리를 합니다.

```python
import json
import traceback

# pip installed packages
import requests

_BOT_TOKEN = '여기에는봇토큰이들어갑니다.'

_BASE_URL = 'https://api.telegram.org/bot{}'.format(_BOT_TOKEN)


HEADERS = {
    'Content-Type': 'application/json'
}


def send_message(chat_id, text):
    url = '/'.join([_BASE_URL, 'sendMessage'])
    data = {
        'chat_id': chat_id,
        'text': text,
    }

    res = requests.post(url, data=json.dumps(data), headers=HEADERS)
    print(res)
    res_json = json.loads(res.text)
    print(res_json)


def divide(divider):
    try:
        return 1000 / divider
    except Exception:
        send_message(53395910, 'divide function has error\n{}'.format(
            traceback.format_exc()
        ))
        raise


if __name__ == '__main__':
    divide(0)
```

divide 함수에 try-except를 추가하고 traceback.format_exc()로 Traceback을 텔레그램으로 받았습니다.

문제는 git과 같은 형상관리 툴에서 기존 코드를 저렇게 묶어버리면 기존 코드 전체가 제가 작성한걸로 간주가 됩니다.

그리고 괜히 쓸데없이 코드가 길어지기도 하고요

코드 수정은 예쁘고 잘 정리하면 좋으니까 이런경우에는 아래와 같이 decorator를 자주 씁니다.

```python
import json
import traceback

# pip installed packages
import requests

_BOT_TOKEN = '여기에는봇토큰이들어갑니다.'

_BASE_URL = 'https://api.telegram.org/bot{}'.format(_BOT_TOKEN)


HEADERS = {
    'Content-Type': 'application/json'
}


def send_message(chat_id, text):
    url = '/'.join([_BASE_URL, 'sendMessage'])
    data = {
        'chat_id': chat_id,
        'text': text,
    }

    res = requests.post(url, data=json.dumps(data), headers=HEADERS)
    print(res)
    res_json = json.loads(res.text)
    print(res_json)


def send_exc(original_func):
    def deco(*args, **kwargs):
        try:
            return original_func(*args, **kwargs)
        except Exception:
            send_message(53395910, 'divide function has error\n{}'.format(
                traceback.format_exc()
            ))
            raise
    return deco


def divide(divider):
    try:
        return 1000 / divider
    except Exception:
        send_message(53395910, 'divide function has error\n{}'.format(
            traceback.format_exc()
        ))
        raise


@send_exc
def divide2(divider):
    return 1000 / divider


if __name__ == '__main__':
    divide2(0)
```
