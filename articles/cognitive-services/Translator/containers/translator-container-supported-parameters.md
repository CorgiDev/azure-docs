---
title: "Container: Translator translate method"
titleSuffix: Azure Cognitive Services
description: Understand the parameters, headers, and body messages for the container Translate method of Azure Cognitive Services Translator to translate text.
services: cognitive-services
author: laujan
manager: nitinme

ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: reference
ms.date: 05/12/2021
ms.author: lajanuar
---

# Container: Translator translate method

Translate text.

## Request URL

Send a `POST` request to:

```HTTP
https://api.cognitive.microsofttranslator.com/translate?api-version=3.0
```

## Request parameters

Request parameters passed on the query string are:

### Required parameters

| Query parameter | Description |
| --- | --- |
| api-version | _Required parameter_.  <br>Version of the API requested by the client. Value must be `3.0`. |
| to  | _Required parameter_.  <br>Specifies the language of the output text. The target language must be one of the [supported languages](../reference/v3-0-languages.md) included in the `translation` scope. For example, use `to=de` to translate to German.  <br>It's possible to translate to multiple languages simultaneously by repeating the parameter in the query string. For example, use `to=de&to=it` to translate to German and Italian. |
| from | _Required parameter_.  <br>Specifies the language of the input text. Find which languages are available to translate from by looking up [supported languages](../reference/v3-0-languages.md) using the `translation` scope.|

### Optional parameters

| Query parameter | Description |
| --- | --- |
| textType | _Optional parameter_.  <br>Defines whether the text being translated is plain text or HTML text. Any HTML needs to be a well-formed, complete element. Possible values are: `plain` (default) or `html`. |
| includeAlignment | _Optional parameter_.  <br>Specifies whether to include alignment projection from source text to translated text. Possible values are: `true` or `false` (default). |
| includeSentenceLength | _Optional parameter_.  <br>Specifies whether to include sentence boundaries for the input text and the translated text. Possible values are: `true` or `false` (default). |

Request headers include:

| Headers | Description |
| --- | --- |
| Authentication header(s) | _Required request header_.  <br>See [available options for authentication](/azure/cognitive-services/translator/reference/v3-0-reference#authentication). |
| Content-Type | _Required request header_.  <br>Specifies the content type of the payload.  <br>Accepted value is `application/json; charset=UTF-8`. |
| Content-Length | _Required request header_.  <br>The length of the request body. |
| X-ClientTraceId | _Optional_.  <br>A client-generated GUID to uniquely identify the request. You can omit this header if you include the trace ID in the query string using a query parameter named `ClientTraceId`. |

## Request body

The body of the request is a JSON array. Each array element is a JSON object with a string property named `Text`, which represents the string to translate.

```json
[
    {"Text":"I would really like to drive your car around the block a few times."}
]
```

The following limitations apply:

* The array can have at most 100 elements.
* The entire text included in the request cannot exceed 10,000 characters including spaces.

## Response body

A successful response is a JSON array with one result for each string in the input array. A result object includes the following properties:

* `translations`: An array of translation results. The size of the array matches the number of target languages specified through the `to` query parameter. Each element in the array includes:

* `to`: A string representing the language code of the target language.

* `text`: A string giving the translated text.

* `sentLen`: An object returning sentence boundaries in the input and output texts.

* `srcSentLen`: An integer array representing the lengths of the sentences in the input text. The length of the array is the number of sentences, and the values are the length of each sentence.

* `transSentLen`:  An integer array representing the lengths of the sentences in the translated text. The length of the array is the number of sentences, and the values are the length of each sentence.

    Sentence boundaries are only included when the request parameter `includeSentenceLength` is `true`.

  * `sourceText`: An object with a single string property named `text`, which gives the input text in the default script of the source language. `sourceText` property is present only when the input is expressed in a script that's not the usual script for the language. For example, if the input were Arabic written in Latin script, then `sourceText.text` would be the same Arabic text converted into Arab script.

Example of JSON responses are provided in the [examples](#examples) section.

## Response headers

| Headers | Description |
| --- | --- |
| X-RequestId | Value generated by the service to identify the request. It is used for troubleshooting purposes. |
| X-MT-System | Specifies the system type that was used for translation for each ‘to’ language requested for translation. The value is a comma-separated list of strings. Each string indicates a type:  </br></br>&FilledVerySmallSquare; Custom - Request includes a custom system and at least one custom system was used during translation.</br>&FilledVerySmallSquare; Team - All other requests |

## Response status codes

If an error occurs, the request will also return a JSON error response. The error code is a 6-digit number combining the 3-digit HTTP status code followed by a 3-digit number to further categorize the error. Common error codes can be found on the [v3 Translator reference page](../reference/v3-0-reference.md#errors).

## Examples

### Translate a single input

This example shows how to translate a single sentence from English to Simplified Chinese.

```curl
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=zh-Hans" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json; charset=UTF-8" -d "[{'Text':'Hello, what is your name?'}]"
```

The response body is:

```
[
    {
        "translations":[
            {"text":"你好, 你叫什么名字？","to":"zh-Hans"}
        ]
    }
]
```

The `translations` array includes one element, which provides the translation of the single piece of text in the input.

### Translate multiple pieces of text

Translating multiple strings at once is simply a matter of specifying an array of strings in the request body.

```curl
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=zh-Hans" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json; charset=UTF-8" -d "[{'Text':'Hello, what is your name?'}, {'Text':'I am fine, thank you.'}]"
```

The response contains the translation of all pieces of text in the exact same order as in the request.
The response body is:

```
[
    {
        "translations":[
            {"text":"你好, 你叫什么名字？","to":"zh-Hans"}
        ]
    },
    {
        "translations":[
            {"text":"我很好，谢谢你。","to":"zh-Hans"}
        ]
    }
]
```

### Translate to multiple languages

This example shows how to translate the same input to several languages in one request.

```curl
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=zh-Hans&to=de" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json; charset=UTF-8" -d "[{'Text':'Hello, what is your name?'}]"
```

The response body is:

```
[
    {
        "translations":[
            {"text":"你好, 你叫什么名字？","to":"zh-Hans"},
            {"text":"Hallo, was ist dein Name?","to":"de"}
        ]
    }
]
```

### Translate content with markup and decide what's translated

It's common to translate content which includes markup such as content from an HTML page or content from an XML document. Include query parameter `textType=html` when translating content with tags. In addition, it's sometimes useful to exclude specific content from translation. You can use the attribute `class=notranslate` to specify content that should remain in its original language. In the following example, the content inside the first `div` element will not be translated, while the content in the second `div` element will be translated.

```
<div class="notranslate">This will not be translated.</div>
<div>This will be translated. </div>
```

Here is a sample request to illustrate.

```curl
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=zh-Hans&textType=html" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json; charset=UTF-8" -d "[{'Text':'<div class=\"notranslate\">This will not be translated.</div><div>This will be translated.</div>'}]"
```

The response is:

```
[
    {
        "translations":[
            {"text":"<div class=\"notranslate\">This will not be translated.</div><div>这将被翻译。</div>","to":"zh-Hans"}
        ]
    }
]
```

### Translate with dynamic dictionary

If you already know the translation you want to apply to a word or a phrase, you can supply it as markup within the request. The dynamic dictionary is only safe for proper nouns such as personal names and product names.

The markup to supply uses the following syntax.

```
<mstrans:dictionary translation="translation of phrase">phrase</mstrans:dictionary>
```

For example, consider the English sentence "The word wordomatic is a dictionary entry." To preserve the word _wordomatic_ in the translation, send the request:

```
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=de" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json; charset=UTF-8" -d "[{'Text':'The word <mstrans:dictionary translation=\"wordomatic\">word or phrase</mstrans:dictionary> is a dictionary entry.'}]"
```

The result is:

```
[
    {
        "translations":[
            {"text":"Das Wort \"wordomatic\" ist ein Wörterbucheintrag.","to":"de"}
        ]
    }
]
```

This feature works the same way with `textType=text` or with `textType=html`. The feature should be used sparingly. The appropriate and far better way of customizing translation is by using Custom Translator. Custom Translator makes full use of context and statistical probabilities. If you have or can afford to create training data that shows your work or phrase in context, you get much better results. [Learn more about Custom Translator](../customization.md).

## Request limits

Each translate request is limited to 10,000 characters, across all the target languages you are translating to. For example, sending a translate request of 3,000 characters to translate to three different languages results in a request size of 3000x3 = 9,000 characters, which satisfy the request limit. You're charged per character, not by the number of requests. It's recommended to send shorter requests.

The following table lists array element and character limits for the Translator **translation** operation.

| Operation | Maximum size of array element | Maximum number of array elements | Maximum request size (characters) |
|:----|:----|:----|:----|
| translate | 10,000 | 100 | 10,000 |

