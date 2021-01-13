+++

authors = [
    "Lena"
]
title = " multipart/form-data 이용해서 사진/이미지 배열 업로드하기 "
date = 2021-01-13T14:41:02+09:00
description = "Uploading array of images using multipart form Data in swift"
tags = [
    "iOS", "HTTP", "multipart/form-data", "uploading images", "image", "URLSession"
]
categories = [
     "iOS", "Network"
]
series = ["Networking in iOS"]
images = [
  "/images/whatis-how_http_works_mobile.png"
]
draft = false

+++

[Networking in iOS] HTTP multipart/form-data에 대해 설명한 후 URLSession을 이용하여 이미지 배열과 텍스트 정보를 서버에 업로드 하는 방법에 대해 설명합니다. <br>

<br>

<!--more-->



##    <  📑 목차  >

* HTTP multipart/form-data
  * HTTP란?
  * 클라이언트 → 서버 파일 업로드하는 과정과 원리 이해하기
    * HTTP 메세지 구성과 multipart
    * MIME에서의 multipart & multipart/form-data
    * 파일 업로드할 때 알아야하는 HTTP 규약
* URLSession으로 이미지 업로드하기
  * 필요한 코드와 코드의 역할 및 주의사항

<br><br>

## <span style="color: #6666FF">HTTP multipart/form-data</span>

 먼저 **HTTP**, **multipart**, **multipart/form-data** 세 가지 키워드에 대해 알아봅시다.

#### <span style="color:orange">**HTTP란?**</span>

#### [HTTP(HyperText Transfer Protocol)](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol)

> 인터넷 상에서 클라이언트와 서버가 자원을 주고 받을 때 쓰는 통신 규약.
> (From [Tech Interview](https://gyoogle.dev/blog/computer-science/network/HTTP%20&%20HTTPS.html))

<br>

#### <span style="color:orange">**클라이언트 → 서버 파일 업로드하는 과정 이해하기**</span>

>  파일 업로드를 구현할 때, 클라이언트가 웹브라우저라면 폼을 통해서 파일을 등록해서 전송하게 됩니다. 이때 웹 브라우저가 보내는 **HTTP 메시지는 Content-Type 속성이 <span style="color:orange">multipart/form-data</span>>로 지정**되며, 정해진 형식에 따라 **메시지를 인코딩**하여 전송합니다. 이를 처리하기 위한 서버는 **_멀티파트 메시지에 대해서 각 파트별로 분리하여 개별 파일의 정보를 얻게 됩니다._**
> (From [Wireframe](https://soooprmx.com/archives/9626))

이미지 파일을 전송한다고 해서 이메일에 첨부파일을 붙여 메일을 보내는 것처럼 png나 jpg 파일 자체가 전송되는 것이 아닙니다. 이미지 파일도 문자로 이뤄져 있기 때문에 이미지 파일을 스펙에 맞게 문자로 생성하여 HTTP request body에 담아 서버로 전송하는 것입니다.

<br>

**<HTTP 메세지 구성과 multipart>**

<img src="https://user-images.githubusercontent.com/52783516/104418336-189aac80-55ba-11eb-8cb0-6a85ee6dd58d.png" alt="image" style="zoom:70%;" />

<span style="color:orange">HTTP(request와 response)</span>는 간단하게 위 이미지와 같이 4개의 파트로 나눌 수 있습니다. **여기서 Message Body에 들어가는 데이터 타입을 HTTP Header에 명시해줄 수 있습니다**. 이 때 명시할 수 있도록 해주는 필드가 바로 Content-type입니다. 추가적으로 Content-type 필드에 [MIME(Multipurpose Internet Mail Extensions)](https://en.wikipedia.org/wiki/MIME) 타입을 기술해줄 수 있는데, 여러 타입 중 하나가 바로 **<span style="color:orange">multipart</span>** 입니다. 

<br>

**< MIME에서의 <span style="color:orange">multipart & multipart/form-data</span> >**

multipart 타입을 통해 MIME은 트리 구조의 메세지 형식을 정의할 수 있습니다. 
ex) 어떤 것이 첨부된 텍스트(multipart/mixed) / 텍스트와 HTML과 같이 다른 포맷을 함께 보낸 메세지(multipart/ alternative) 등

* [ Multipart 메세지 ] 

  * **서로 붙어있는 여러 개의 메세지를 포함하여 하나의 복합 메세지**로 보내집니다.
  * MIME multipart 메세지는 **"Content-type:" 헤더**에 boundary 파라미터를 포함합니다.
  * **boundary는 메세지 파트를 구분하는 역할을 하며, 메세지의 시작과 끝 부분도 나타납니다.**
  * 첫번째 Boundary 전에 나오는 내용은 MIME을 지원하지 않는 클라이언트를 위해 제공됩니다.
  * **boundary 를 선택하는 것은 클라이언트의 몫**입니다. **보통 무작위의 문자를 선택**해 메세지의 본문과 충돌을 피합니다. Ex) UUID

  * 멀티파트 폼 제출: 
    1. HTTP form을 채워서 제출하면, 가변 길이 텍스트 필드와 업로드 될 객체는 각각 멀티파트 본문을 구성하는 하나의 파트가 되어 보내집니다. 멀티 파트 분몬은 여러 다른 종류와 길이의 값으로 채워진 form을 허용합니다.
    2. <span style="color:orange">`multipart/form-data`</span>: 사용자가 양식을 작성한 결과 값의 집합을 번들로 만드는데 사용합니다.

(출처: [기록은 기억을 이긴다](https://qssdev.tistory.com/47))

<br>

**<파일 업로드할 때 알아야하는 HTTP 규약>**

> First, there’s the `Content-Type` header. It contains information about the type of data you’re sending (`multipart/form-data;`) and a `boundary`. This boundary should always have a unique, somewhat random value. In the example above I used a `UUID`. Since multipart forms are not always sent to the server all at once but rather in chunks, the server needs some way to know when a certain part of the form you’re sending it ends or begins. This is what the `boundary` value is used for. This must be communicated in the headers since that’s the first thing the receiving server will be able to read.  
> (출처: [Uploading images and forms to a server using URLSession](https://www.donnywals.com/uploading-images-and-forms-to-a-server-using-urlsession/))

**<파일 업로드할 때 알아야하는 HTTP 규약>**

![img](https://t1.daumcdn.net/cfile/tistory/255E643653B0F89026)

(이미지 출처: [탁구치는 개발자](https://lng1982.tistory.com/209))

서버에 multipart/form-data로 데이터를 보낼때의 request header와 body는 위 이미지와 같이 구성되어있습니다. 

위 이미지과 함께 다음과 같은 HTTP 통신 규격을 확인해 볼 수 있습니다

1. Content-Type가 multipart/form-data로 지정 되어있어야 서버에서 정상적으로 데이터를 처리할 수 있습니다.
2. 전송되는 파일 데이터의 구분자로 boundary에 지정되어 있는 문자열을 이용합니다.
3. boundary의 문자열 중 마지막 `**------WebKitFormBoundaryQGvWeNAiOE4g2VM5--**` 값은 다른 값과 다르게 `--`가 마지막에 붙었는데, `--` 는 body의 끝을 알리는 의미를 가집니다.

이 규격에 맞게 http header와 body 데이터를 생성 한 후 HTTP server에 요청하게 되면 서버에서도 HTTP 통신 규격에 맞게 데이터를 파싱한 후 처리하게 됩니다.  아래는 HTTP Request Data,  HTTP Response Data 예시입니다.

```http
## HTTP Request Data

POST /file/upload HTTP/1.1[\r][\n]

Content-Length: 344[\r][\n]

Content-Type: multipart/form-data; boundary=Uee--r1_eDOWu7FpA0LJdLwCMLJQapQGu[\r][\n]

Host: localhost:8080[\r][\n]

Connection: Keep-Alive[\r][\n]

User-Agent: Apache-HttpClient/4.3.4 (java 1.5)[\r][\n]

Accept-Encoding: gzip,deflate[\r][\n]

[\r][\n]

--Uee--r1_eDOWu7FpA0LJdLwCMLJQapQGu[\r][\n]

Content-Disposition: form-data; name=files; filename=test.txt[\r][\n]

Content-Type: application/octet-stream[\r][\n]

[\r][\n]

aaaa

[\r][\n]

--Uee--r1_eDOWu7FpA0LJdLwCMLJQapQGu[\r][\n]

Content-Disposition: form-data; name=files; filename=test1.txt[\r][\n]

Content-Type: application/octet-stream[\r][\n]

[\r][\n]

1111

[\r][\n]

--Uee--r1_eDOWu7FpA0LJdLwCMLJQapQGu--[\r][\n]


## HTTP Response Data

HTTPHTTP/1.1 200 OK[\r][\n]

Server: Apache-Coyote/1.1[\r][\n]

Accept-Charset: big5, big5-hkscs, euc-jp, euc-kr...[\r][\n]

Content-Type: text/html;charset=UTF-8[\r][\n]

Content-Length: 7[\r][\n]

Date: Mon, 30 Jun 2014 01:28:19 GMT[\r][\n]

[\r][\n]

SUCCESS
```

(출처: [탁구치는 개발자](https://lng1982.tistory.com/209))

추가적으로 `header` 와  `header` 를 구분하는 것은 개행문자이고,  `header` 와 `body` 를 구분하는 것은 개행 문자 2개, `body` 에 포함되어있는  `file data` 를 구분하는 것은 boundary입니다.

> 엄밀하게는 (개행)바운더리문자열(개행)을 기준으로 구분하게 됩니다. 또, 이때의 개행은 플랫폼에 상관없이 CRLF로 `\r\n`을 사용해야 합니다.
> 출처: [Wireframe](https://soooprmx.com/archives/9626)

<br>

## <span style="color:#6666FF">**URLSession으로 이미지 배열 업로드하기**</span>

<br>

<span style="color:orange">**필요한 코드와 코드의 역할 및 주의사항**</span>

[Uploading images and forms to a server using URLSession](https://www.donnywals.com/uploading-images-and-forms-to-a-server-using-urlsession/) 에 있는 내용을 일부 발췌해 정리했습니다.

``` swift

let boundary = "Boundary-\(UUID().uuidString)"

var request = URLRequest(url: URL(string: "https://some-page-on-a-server")!)
request.httpMethod = "POST"
request.setValue("multipart/form-data; boundary=\(boundary)", forHTTPHeaderField: "Content-Type")
```

1. **create a URLRequest**, make it a POST request and set its Content-Type header.

<br>

``` swift
func convertFormField(named name: String, value: String, using boundary: String) -> String {
  var fieldString = "--\(boundary)\r\n"
  fieldString += "Content-Disposition: form-data; name=\"\(name)\"\r\n"
  fieldString += "\r\n"
  fieldString += "\(value)\r\n"

  return fieldString
}
```
2. create a **method that will output these chunks of body data.**<br>
⭐️ Note the \r\n that is added to the string after every line. This is needed to add a new line to the string so we get the output that we want.

<br>


```swift
func convertFileData(fieldName: String, fileName: String, mimeType: String, fileData: Data, using boundary: String) -> Data {
  let data = NSMutableData()

  // ⭐️ 이미지가 여러 장일 경우 for 문을 이용해 data에 append 해줍니다. 
  // (현재는 이미지 파일 한 개를 data에 추가하는 코드)
  data.appendString("--\(boundary)\r\n")
  data.appendString("Content-Disposition: form-data; name=\"\(fieldName)\"; filename=\"\(fileName)\"\r\n")
  data.appendString("Content-Type: \(mimeType)\r\n\r\n")
  data.append(fileData)
  data.appendString("\r\n")

  return data as Data
}

extension NSMutableData {
  func appendString(_ string: String) {
    if let data = string.data(using: .utf8) {
      self.append(data)
    }
  }
}
```
3. create a **body chunk** for the file.  
    ✍🏻 Instead of a String, we create Data this time. The reason for this is twofold.   
    One is that **we already have the file data**. Converting this to a String and then back to Data when we add it to the HTTP body is wasteful.   
    The second reason is that the **HTTP body itself must be created as Data rather than a String**. To make appending text to the Data object, we add an extension on NSMutableData that safely appends the given string as Data. From the structure of the method, you should be able to derive that it matches the HTTP body that was shown earlier.

<br>

**<span style="color:orange">이미지가 여러장일 위 예제 코드에서 ⭐️한 곳에 이미지 배열을 반복문을 통해 순회하며 data에 추가해주면 됩니다.</span>**

<br><br>

```swift
let httpBody = NSMutableData()

for (key, value) in formFields {
  httpBody.appendString(convertFormField(named: key, value: value, using: boundary))
}

httpBody.append(convertFileData(fieldName: "image_field",
                                fileName: "imagename.png",
                                mimeType: "image/png",
                                fileData: imageData,
                                using: boundary))

httpBody.appendString("--\(boundary)--")

request.httpBody = httpBody as Data

print(String(data: httpBody as Data, encoding: .utf8)!)
```
4. You use the methods you wrote earlier to construct the HTTP body.  
   After adding the form fields you add the **final boundary with   the two trailing dashes and the resulting data is set** as the request’s httpBody.  
   print 하면 나오는 데이터는 아래와 같습니다.   

 ```http
 // print 하면 나오는 데이터
--Boundary-3A42CBDB-01A2-4DDE-A9EE-425A344ABA13
Content-Disposition: form-data; name="family_name"

Wals
--Boundary-3A42CBDB-01A2-4DDE-A9EE-425A344ABA13
Content-Disposition: form-data; name="name"

Donny
--Boundary-3A42CBDB-01A2-4DDE-A9EE-425A344ABA13
Content-Disposition: form-data; name="file"; filename="somefilename.jpg"
Content-Type: image/png

-a long string of image data-
--Boundary-3A42CBDB-01A2-4DDE-A9EE-425A344ABA13—
 ```

 ```swift
 URLSession.shared.dataTask(with: request) { data, response, error in
  // handle the response here
}.resume()
 ```
5. **run your request** just like you would normally

<br><br>

#### <span style="color:orange">**⭐️추가적인 URLSession / Alamofire를 이용한 예제는**</span>

#### <span style="color:orange">**[이 블로그](https://nsios.tistory.com/39)를 참고하면 될 것 같습니다.⭐️**</span>



<br>

## <span style="color: #6666FF">참고</span>

* HTTP 이해하기 
  1. [multipart/form-data 타입의 HTTP 메시지 구성 방법](https://soooprmx.com/archives/9626)  
  2. [HTTP multipart/form-data raw 데이터는 어떤 형태일까?](https://lng1982.tistory.com/209) 
  3. [HTTP Multipart 와 MIME](https://qssdev.tistory.com/47)

* iOS 에서 이미지 업로드하기 
  1. [[Swift] - MultiPart통신 (멀티파트 이미지업로드)](https://nsios.tistory.com/39) 
  2. [Uploading images and forms to a server using URLSession](https://www.donnywals.com/uploading-images-and-forms-to-a-server-using-urlsession/) 
  3. [How to Upload Image to Imgur in iOS using Swift](https://johncodeos.com/how-to-upload-image-to-imgur-in-ios-using-swift/) 
  4. [Upload image to server using URLSessionUploadTask](https://fluffy.es/upload-image-to-server/)

