---
layout: post
title:  "텔레그램 봇을 만들어보자"
description: An example post which shows code rendering.
date:   2020-07-09 14:47:36 +0530
categories: Python Telegrambot
---

이런저런 배치와 데몬을 만들다보면 어느순간 잘 돌았는지 체크하는 것 자체가 부담이 되는 순간이 옵니다.

스케쥴표를 만들어서 특정 시간마다 로그를 들여다 볼 수도 없고, 창 하나에 로그 파일을 tail -f 해놓으면 화면이 부족해지고..

이럴 때 실행 결과나 에러가 발생했을 때 push를 받으면 큰 도움이 됩니다.

이메일로 받을 수도 있지만 요즘은 광고성 메일이 너무 많다보니 종종 놓치는 경우도 생깁니다.

이럴때는 텔레그램이 좋은 대안이 됩니다. 배치의 실행결과 플랫폼의 에러 등이 바로 채팅으로 오니 실시간으로 대응도 가능하고요.

이번 포스팅에서는 텔레그램봇을 만들고 어떻게 모니터링에 활용 하는 방법에 대해서 공유해보겠습니다.


## 텔레그램 봇 만들기

텔레그램봇은 모든 텔레그램봇의 아버지 봇 @BotFather 를 이용해서 만들 수 있습니다.

![posting-creating-telegrambot-1.PNG](../assets/images/posting-creating-telegrambot-1.PNG)

BotFather 와 채팅창에서 봇생성, 봇이름 변경, command 추가 등의 작업을 할 수 있는데요, 일단 봇을 만들어보겠습니다.

![posting-creating-telegrambot-2.PNG](../assets/images/posting-creating-telegrambot-2.PNG)

위의 박스로 가려놓은 부분에 bot_token이 생성이 되고 텔레그램 REST API를 사용할때 이 bot_token을 사용하게 됩니다.

그리고 @mc_ex_bot 을 검색을 해보면 봇이 생성된것을 확인할 수 있습니다.

![posting-creating-telegrambot-2.PNG](../assets/images/posting-creating-telegrambot-3.PNG)


## 텔레그램 봇으로 메세지를 보내기

저희의 궁극적인 목표는 텔레그램으로 메세지를 보내려면 봇이 참여하고 있는 방이 필요하겠죠?

그리고 어떤 채팅방인지도 알아내야 합니다. 텔레그램에서는 chat_id 라고 합니다.

한번 봇과 단둘이 오붓하게 대화할 수 있는 개인방을 열어서 시작해봅시다.

![posting-creating-telegrambot-2.PNG](../assets/images/posting-creating-telegrambot-4.PNG)

이렇게 봇이 참가한 방에 메세지는 [getUpdates](https://core.telegram.org/bots/api#getupdates)라는 REST API로 가져올수 있습니다.

텔레그램 봇 REST API의 기본 URL은

https://api.telegram.org/bot 여기 뒤에 바로 위의 봇 토큰을 붙여서 호출하면 됩니다.

getUpdates는 아래와 같이 호출하면 되겠죠??

https://api.telegram.org/bot예제지만그래도봇토큰은가려야지/getUpdates

별다른 파라미터가 없어도 되니까 그냥 브라우저에 넣어서 콜을 해봤습니다.

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

메세지를 3개를 보내서 3개가 뜰텐데 하나만 골라서 눈을 부릅뜨고 보면 "chat"이라는 필드를 발견할 수 있습니다.

이게 바로 채팅방의 정보이고요, 요 dict 안에 "id" 필드가 채팅방의 ID가 되겠습니다. (짧게 앞으론 chat_id라고 할게요)

예제에서 방은 53395910 인가 봅니다.

자 이제 메세지를 한번 보내봅시다.

[sendMessage API](https://core.telegram.org/bots/api#sendmessage)를 이용하면 되는데요. 일단 브라우저에서 바로 한번 콜을 해보겠습니다.

![posting-creating-telegrambot-5.PNG](../assets/images/posting-creating-telegrambot-5.PNG)

chat_id에는 53395910을 넣고 text에는 보낼 메세지를 넣어 보냈고, 메세지가 잘 날아왔군요.

먼길을 왔군요. 이제는 코드에 적용할 일만 남았습니다!


## Python에서 텔레그램 메세지 보내기

일단 뛰기전에 먼저 걸어봐야겠죠?? 메세지를 보내는 기본 코드입니다. POST로 json 형태로 데이터를 보내는 예제입니다

```python
import json

# pip installed packages
import requests

_BOT_TOKEN = '1009737758:AAEZXTKSsjTJ2oVofMt4Q-P92l1P4UVu3rY'

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
