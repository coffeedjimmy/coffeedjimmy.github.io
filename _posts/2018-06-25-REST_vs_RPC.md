---
title: REST와 RPC
date: 2018.06.25 12:00:00
categories:
- API
tags:
- REST
- RPC
---



이 글은 REST와 RPC를 비교한 [다음 글](https://apihandyman.io/do-you-really-know-why-you-prefer-rest-over-rpc/)을 번역하며 기술하였습니다.


일반적으로 알려져 있기론 `REST(Representational State Transfer)`와 `RPC(Remote Procedure Call)` 는 다음과 같은 특징을 가집니다.

> REST : resource oriented

> RPC : operation oriented

단순히, REST 는 자원 기반, RPC는 동작 기반이라는 말이 잘 이해가 안될 것이라 좀 더 자세히 설명해 보고자 합니다.

### HTTP Request

**RPC**와 **REST**는 둘 다 HTTP프로토콜을 이용해서 동작합니다. HTTP 요청은 기본적으로 `verb(method)` 와 `resource(endpoint)`로 구성되어 있습니다. 그 중에서도 `verb(동사)`는 다음과 같은 특징을 가지고 있습니다.

> 각각의 HTTP verb는...

- 의미를 가지고 있습니다.
- Idempotent or not : 같은 요청에 대해서 항상 같은 결과값을 리턴합니다. (POST는 idempotent하지 않음)
- Safe or not : Read-only이거나 아니거나.
- Cacheable or not : Caching이 가능하거나 아니거나.

| 동사   | 의미                                                         | 멱등원 | Safe | Cacheable                                                    |
| ------ | ------------------------------------------------------------ | ------ | ---- | ------------------------------------------------------------ |
| GET    | 자원을 읽어 들임                                             | Yes    | Yes  | Yes                                                          |
| POST   | 자원을 생성하거나, 데이터 처리 프로세스를 실행시킴.          | No     | No   | 결과값이 명시적으로 새로운 정보를 담고 있을 때에만 캐싱이 가능함. |
| PUT    | 이미 존재하는 자원을 업데이트(대체)하거나 새로운 자원을 생성함. | Yes    | No   | No                                                           |
| PATCH  | 자원을 일부만 업데이트.                                      | No     | No   | 결과값이 명시적으로 새로운 정보를 담고 있을 때에만 캐싱이 가능함. |
| DELETE | 자원을 삭제.                                                 | Yes    | No   | No                                                           |

> Idempotent(멱등원) : 수학용어로 어떤 과정을 몇 번이고 반복 수행하여도 결과가 동일함.

### RPC(Remote Procedure Call)

**The operation request style**

RPC라는 축약어는 많은 의미를 가지고 있고, Remote Procedure Call은 많은 형태를 가지고 있습니다. 이번 글에서는 RPC에 대해서 논할 때, _WYGOPIAO_ (What You GET Or POST Is An Operation)에 대해서 다루겠습니다.

이 경우에 RPC는 HTTP를 통신 프로토콜로하여 데이터를 조작하기 위한 동작들을 나타내는 것을 의미합니다. 이에 대해 특정하게 정해진 규칙은 없지만 일반적으로 다음과 같은 특징을 가집니다.

- Endpoint는 당신이 실행하고자 원하는 기능(동작)의 이름을 가지고 있습니다.
- GET / POST 만을 사용합니다.

```bash
# anId라는 데이터를 가지고 someoperation을 실행
GET /someoperation?data=anId

# 데이터 자체를 조작하는 anotheroperation이라는 동작을 수행
POST /anotheroperation
{
  "data":"anId";
  "anotherdata":"another value"
}
```

그렇다면 사람들은 어떤 기준으로 GET/POST를 사용할까요?

- HTTP 프로토콜에 대해 어느 정도 고려하는 사람들은 **GET**을 데이터를 수정하지 않는 동작에 사용하고, **POST**를 그 외의 상황에 사용합니다.
- HTTP 프로토콜에 대해 그다지 신경쓰지 않는 사람들은 **GET**을 너무 많은 파라미터들을 가지고 있지 않은 동작에 사용하고, **POST**를 그 외의 상황에 사용합니다.
- 이런 것들과 관련된 것에서 아예 고려하지 않는 사람들은 **GET**과 **POST**를 랜덤하게 사용하거나 오직 **POST**를 사용합니다.

### REST(Representational State Transfer)

**The resource request style**

*이 글에서는 REST에 대해 깊게 다루지는 않을 것입니다.*

REST는 데이터를 자원의 관점에서 표현하고 이를 조작하기 위해 HTTP 프로토콜을 사용합니다.

* Endpoint는 당신이 조작하고자 하는 자원을 포함합니다.
* 많은 사람들이 REST request 원리를 설명하기 위해 CRUD 아날로지를 사용합니다. HTTP 프로토콜의 동사들은 Create/Read/Update/Delete 등과 같이 우리가 무엇을 하고자 하는지를 그 뒤에 나오는 리소스(자원)과 함께 나타냅니다.

```bash
# someresource라는 endpoint 아래에 있는 anId라는 리소스를 가져옴.
GET /someresources/anId

# someresource라는 endpoint 아래에 있는 anId라는 리소스를 수정함.
PUT /someresources/anId
{"anotherdata":"another value"}
```



### REST / RPC 비교

| Operation                      | RPC (operation)                     | REST (resource)          |
| ------------------------------ | ----------------------------------- | ------------------------ |
| Signup                         | POST /signup                        | POST /persons            |
| Resign                         | POST /resign                        | DELETE /persons/1234     |
| Read a person                  | GET /readPerson?personid=1234       | GET /persons/1234        |
| Read a person’s items list     | GET /readUsersItemsList?userid=1234 | GET /persons/1234/items  |
| Add an item to a person’s list | POST /addItemToUsersItemsList       | POST /persons/1234/items |
| Update an item                 | POST /modifyItem                    | PUT /items/456           |
| Delete an item                 | POST /removeItem?itemId=456         | DELETE /items/456        |

| Item                                                      | Who wins? |
| :-------------------------------------------------------- | :-------- |
| Beauty (얼마나 깔끔하게 구성할 수 있는가)                 | Draw      |
| Designability (Endpoint를 구성하기에 어떤 것이 더 편한가) | Draw      |
| API definition language (문서화)                          | Draw      |
| Predictability and semantic (예상 가능성)                 | REST      |
| Hypermediability (Hypermedia API)                         | Draw      |
| Cachability (캐싱 가능성)                                 | Draw      |
| Usability (사용성)                                        | Draw      |

그렇다면 **Predictability and semantic** 의 측면에서 REST가 더 낫기 때문에 REST가 RPC에 비해 우월하다고 말할 수 있을까요? 답은 아니라는 것입니다.

RPC와 REST는 서로 다른 접근법을 가지고 있고, 그에 따라 장점과 단점을 수반하고 있습니다. 그리고 이는 각각의 상황에 따라서 달라지게 됩니다. 심지어 하나의 API에 대해서 두 접근법을 섞어서 사용할 수도 있습니다.

**Context** 가 바로 이를 결정하는 키이고, 그 어디에도 모든 경우를 케어하는 만병통치약은 없습니다. 따라서 단순히 트랜드를 따르는 것이 아니라 그 상황(context)에 맞게 실용적으로 판단해서 솔루션을 찾아내야 합니다.

적어도 이번 글을 통해 제가 REST를 더 선호하는 이유는 **Predictability and semantic** 때문이라는 것을 알게 되었지만, 당신은 다를 수 있습니다.



...
