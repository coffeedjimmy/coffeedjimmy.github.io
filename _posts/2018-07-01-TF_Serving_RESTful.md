---
title: TensorFlow Serving RESTful API
date: 2018.07.01 12:00:00
categories:
- TensorFlow Serving
tags:
- TensorFlow
---

.

# Tensorflow Serving RESTful API

`gRPC` API에 더불어 Tensorflow ModelServer는 Tensorflow 모델들에 대한 classification, regression 그리고 prediction을 위해 `RESTful API` 또한 제공합니다. 이번 글은 이 `RESTful API`를 사용하기 위해 어떠한 API 엔드포인트가 있는지와 request/response가 어떻게 구성되는 지를 설명합니다.

특정 `host:port` 에서 실행되고 있는 Tensorflow ModelServer는 다음과 같은 REST API request를 받아들입니다.

```bash
POST http://host:port/<URI>:<VERB>

URI: /v1/models/${MODEL_NAME}[/versions/${MODEL_VERSION}]
VERB: classify|regress|predict
```

`/versions/${MODEL_VERSION}`은 선택입니다. 생략되는 경우에는 최신 버전을 사용합니다.

이 RESTful API는 gRPC 버전의 [PredictionService](https://github.com/tensorflow/serving/blob/5369880e9143aa00d586ee536c12b04e945a977c/tensorflow_serving/apis/prediction_service.proto#L15) API와 밀접하게 연관이 있습니다.

Request URL들의 예시는 다음과 같습니다.

```bash
http://host:port/v1/models/iris:classify
http://host:port/v1/models/mnist/versions/314:predict
```

Request와 Response는 `JSON` 객체입니다. 이 객체의 구성은 요청의 타입과 verb에 따라 달라집니다.

에러가 발생하는 경우, 모든 API들은 `error`를 키로 가지고 error message를 value로 가지는 JSON 객체를 반환합니다.

```bash
{
  "error": <error message string>
}
```

## Classify and Regress API

### Request format

`classify`와 `regress` API의 request body는 다음과 같은 JSON 객체여야 합니다.

```bash
{
  // Optional: serving signature to use.
  // If unspecifed default serving signature is used.
  "signature_name": <string>,

  // Optional: Common context shared by all examples.
  // Features that appear here MUST NOT appear in examples (below).
  "context": {
    "<feature_name3>": <value>|<list>
    "<feature_name4>": <value>|<list>
  },

  // List of Example objects
  "examples": [
    {
      // Example 1
      "<feature_name1>": <value>|<list>,
      "<feature_name2>": <value>|<list>,
      ...
    },
    {
      // Example 2
      "<feature_name1>": <value>|<list>,
      "<feature_name2>": <value>|<list>,
      ...
    }
    ...
  ]
}
```

`<value>`는 JSON 번호나 string이고 `<list>` 은 value들의 리스트입니다. 아래에 있는 `Encoding binary values` 섹션을 참고하세요. 이 포맷은 gRPC의 `ClassificatinRequest`와 `RegressionRequest` protos와 유사합니다. 두 버전 모두 Example 객체의 리스트를 수용합니다.

### Response format

`classify` 요청은 다음과 같은 응답 객체를 반환합니다.

```bash
{
  "result": [
    // List of class label/score pairs for first Example (in request)
    [ [<label1>, <score1>], [<label2>, <score2>], ... ],

    // List of class label/score pairs for next Example (in request)
    [ [<label1>, <score1>], [<label2>, <score2>], ... ],
    ...
  ]
}
```

`<label>`은 비어있는 `""`가 될 수도 있는 string이며 `<score>`는 10진수 (소수점 포함) 숫자입니다.

`regress` 요청은 다음과 같은 응답 객체를 반환합니다.

```bash
{
  // One regression value for each example in the request in the same order.
  "result": [ <value1>, <value2>, <value3>, ...]
}
```

`<value>`는 10진수의 숫자입니다.

gRPC API 사용자들은 `ClassificationResponse`와 `RegressionResponse` proto 가 유사한 포맷을 가지고 있으므로 참고하면 됩니다.

## Predict API

### Request format

`predict` API의 요청은 다음과 같은 JSON 객체 형태여야 합니다.

```bash
{
  // Optional: serving signature to use.
  // If unspecifed default serving signature is used.
  "signature_name": <string>,

  // List of tensors (each element must be of same shape and type)
  "instances": [ <value>|<(nested)list>|<object>, ... ]
}
```

이 형태는 gRPC의 `PredictRequest` proto와 [CMLE predict API](https://cloud.google.com/ml-engine/docs/v1/predict-request)와 유사합니다.

만약에 input이 하나만 존재한다면 리스트에 있는 아이템들은 스칼라 값이 됩니다. (number/string)

```bash
{
  "instances": [ "foo", "bar", "baz" ]
}
```

또는 기본 타입들의 리스트입니다.

```bash
{
  // List of 2 tensors each of [1, 2] shape
  "instances": [ [[1, 2]], [[3, 4]] ]
}
```

텐서들은 기본적으로 리스트를 인위적으로 flatten 할 필요가 없기 때문에 보통 nested된 형태로 표기합니다.

Input이 하나 이상일 경우에 각각의 아이템들은 input name/tensor value 쌍을 포함하고 있는 객체로 표현됩니다. 예를 들어 아래에 나오는 요청은 두 개의 인스턴스를 포함하고 각각 3개의 input tensor를 가지는 경우입니다.

```bash
{
 "instances": [
   {
     "tag": ["foo"]
     "signal": [1, 2, 3, 4, 5]
     "sensor": [[1, 2], [3, 4]]
   },
   {
     "tag": ["bar"]
     "signal": [3, 4, 1, 2, 5]]
     "sensor": [[4, 5], [6, 8]]
   },
 ]
}
```

더 자세한 내용은 아래의 `Encoding binary values`섹션을 참고해주세요.

### Response format

`predict` 요청은 다음과 같은 형태의 JSON 객체를 반환합니다.

```bash
{
  "predictions": [ <value>|<(nested)list>|<object>, ...]
}
```

만약 모델의 결과물이 하나의 명기된 텐서를 포함하고 있을 경우 이름과 `prediction` 키맵을 생략합니다. 만약 모델의 결과물이 하나 이상의 명기된 텐서들을 포함하고 있을 경우에 앞선 경우와는 다르게 위에서 언급한 요청의 포맷과 유사한 객체 리스트를 반환합니다.

명기된 텐서들 중에 `_bytes`라는 접미어를 가지고 있는 것들은 바이너리 값을 가지고 있는 것으로 간주합니다. 자세한 내용은 아래의 `encoding binary values` 섹션을 참고하세요.

## JSON mapping

RESTful API JSON 형태로 이루어진 표준 인코딩을 지원하고 시스템간에 데이터 교환을 쉽게 해줍니다. 지원하는 타입들은 아래의 표를 참고하세요. 아래 표에 없는 타입들은 지원되지 않음을 의미합니다.

| [TF Data Type](https://www.tensorflow.org/versions/r1.1/programmers_guide/dims_types#data_types) | [JSON Value](http://json.org/) | JSON example                   | Notes                                                        |
| ------------------------------------------------------------ | ------------------------------ | ------------------------------ | ------------------------------------------------------------ |
| DT_BOOL                                                      | true, false                    | *true, false*                  |                                                              |
| DT_STRING                                                    | string                         | *"Hello World!"*               |                                                              |
| DT_INT8, DT_UINT8, DT_INT16, DT_INT32, DT_UINT32, DT_INT64, DT_UINT64 | number                         | *1, -10, 0*                    | JSON value will be a decimal number.                         |
| DT_FLOAT, DT_DOUBLE                                          | number                         | *1.1, -10.0, 0, NaN, Infinity* | JSON value will be a number or one of the special token values - `NaN`, `Infinity`, and `-Infinity`. See [JSON conformance](https://www.tensorflow.org/serving/api_rest#json_conformance) for more info. Exponent notation is also accepted. |

## Encoding binary Values

JSON은 UTF-8 인코딩을 사용합니다. 만약 이미지 bytes와 같이 바이너리여야 하는 input feature나 tensor가 있는 경우, **반드시** 다음과 같이 `b64`를 키로 가지는 방식으로 JSON 객체를 `Base64`인코딩해야합니다.

```bash
{ "b64": <base64 encoded string> }
```

또한, input feature와 tensor에 대한 값의 형태로 이 객체들을 명세할 수도 있습니다. 같은 포맷이 결과를 인코딩하는데 사용됩니다.

아래의 예시는 image(binary data)와 caption 피쳐를 포함한 classification 요청을 나타낸 것입니다.

```bash
{
  "signature_name": "classify_objects",
  "examples": [
    {
      "image": { "b64": "aW1hZ2UgYnl0ZXM=" },
      "caption": "seaside"
    },
    {
      "image": { "b64": "YXdlc29tZSBpbWFnZSBieXRlcw==" },
      "caption": "mountains"
    }
  }
}
```

## JSON conformance

많은 피쳐와 텐서들이 부동 소수점 수입니다. 유한한 값들 (e.g. 3.14, 1.0 etc)과는 다르게 이는 `NaN`이나 non-finite(`Infinity` and `-Infinity`)값이 될 수 있습니다. 아쉽게도 JSON 명세는 이러한 값들을 인식할 수 없습니다.

이 페이지에서 설명한 REST API들은 요청/응답에 있어 이러한 값들을 허용합니다. 이는 아래와 같은 요청들이 유효하다는 것을 뜻합니다.

```bash
{
  "example": [
    {
      "sensor_readings": [ 1.0, -3.14, Nan, Infinity ]
    }
  ]
}
```

위의 예시는 `NaN`과 `Infinity` 토큰들이 실 숫자들과 섞여있기 때문에 엄격한 표준 JSON 파서들은 파싱 에러를 냅니다. 이와 같은 요청/응답을 적절히 다루기 위해서는 반드시 다음과 같은 토큰들을 허용하는 JSON 파서를 사용해야 합니다.

`NaN`, `Infinity`, `-Infinity` 토큰들은 **proto3**, Python의 **JSON** 모듈 그리고 **Javascript**에서 인식합니다.

...
