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
series = ["Network"]
images = [
  "/images/send_file_to_server.png"
]
draft = false

+++

[Networking in iOS] URLSession을 이용하여 이미지 배열과 텍스트 정보를 서버에 업로드 하는 방법에 대해 설명합니다. <br>

<br>

<!--more-->



##    <  📑 목차  >

* URLSession으로 이미지와 텍스트 업로드하기
  * 필요한 코드와 코드의 역할 및 주의사항
  * 동작하는 코드
<br><br>

<br>



이전 포스트 [HTTP multipart/form-data 이해하기](https://lena-chamna.netlify.app/post/http_multipart_form-data/) 에 iOS에서 서버에 파일을 보낼 때 필요한 HTTP multipart/form-data 에 대한 설명이 있습니다! 이전 포스트를 보고 오시면 이해와 응용에 훨씬 도움됩니다 😆 

## <span style="color:#6666FF">**URLSession으로 이미지 배열 업로드하기**</span>

<br>

#### <span style="color:orange">필요한 코드와 코드의 역할 및 주의사항</span>

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

#### <span style="color:orange">⭐️동작하는 코드</span>

서버에 보내는 HTTP request body에 담아 보낼 데이터 형식은 이렇습니다.

<img src="/Users/keunnalee/Library/Application Support/typora-user-images/image-20210214180555051.png" alt="image-20210214180555051" style="zoom:50%;" />


```swift
import Foundation
import Combine
import UIKit.UIImage

final class NoteNetworkingManager {

typealias NoteImage = UIImage
typealias NoteText = String
    
    private let decoder: JSONDecoder = .init()
    
    // MARK: - create note
    func createNote(with text: NoteText,
                    images: [NoteImage],
                    completion: @escaping(Bool) -> Void) {
        
        let boundary = generateBoundaryString()
        guard let endpoint = Endpoint(path: .createNote).url else { return }
        var request = URLRequest(urlWithToken: endpoint, method: .post)
        request.setValue("multipart/form-data; boundary=\(boundary)", forHTTPHeaderField: "Content-Type")
        
        // memo Text Data
        let rawMemo = ["rawMemo": text]
        let encoder = JSONEncoder()
        guard let jsonData = try? encoder.encode(rawMemo) else { return }
        guard let jsonString = String(data: jsonData, encoding: .utf8) else { return }
        let textData: [String: String] = ["request": jsonString]
        
        var httpBody = NSMutableData()
        for (key, value) in textData {
            httpBody.appendString(convertFormField(named: key, value: value, using: boundary))
        }

        // photo Image Data
        for image in images {
            guard let imageData = image.jpegData(compressionQuality: 0.1) else { return }
            httpBody.append(convertFileData(fieldName: "file", fileName: "\(Date().millisecondsSince1970)_photo.jpg", mimeType: "multipart/form-data", fileData: imageData, using: boundary))
        }
        httpBody.appendString("--\(boundary)--")  // add final boundary with the two trailing dashes
        request.httpBody = httpBody as Data

        // request
        UseCase.shared
            .request(request: request)
            .receive(subscriber: Subscribers.Sink(receiveCompletion: { [weak self] in
            guard case let .failure(error) = $0 else { return }
            debugPrint(error.message)
        }, receiveValue: { [weak self] response in
            guard let httpResponse = response as? HTTPURLResponse, (200...299).contains(httpResponse.statusCode) else {
                completion(false)
                return
            }
           completion(true)
        }))
    }
}

extension NoteNetworkingManager {
    private func convertFormField(named name: String,
                                  value: String,
                                  using boundary: String) -> String {
        let mimeType = "application/json"
        var fieldString = "--\(boundary)\r\n"
        fieldString += "Content-Disposition: form-data; name=\"\(name)\"\r\n"
        fieldString += "Content-Type: \(mimeType)\r\n\r\n"
        fieldString += "\r\n"
        fieldString += "\(value)\r\n"
        
        return fieldString
    }
    
    private func convertFileData(fieldName: String,
                                 fileName: String,
                                 mimeType: String,
                                 fileData: Data,
                                 using boundary: String) -> Data {
        let data = NSMutableData()
        
        data.appendString("--\(boundary)\r\n")
        data.appendString("Content-Disposition: form-data; name=\"\(fieldName)\"; filename=\"\(fileName)\"\r\n")
        data.appendString("Content-Type: \(mimeType)\r\n\r\n")
        data.append(fileData)
        data.appendString("\r\n")
        
        return data as Data
    }
    
    private func generateBoundaryString() -> String {
        return "Boundary-\(UUID().uuidString)"
    }
}

```

<br>

## <span style="color: #6666FF">참고</span>

* iOS 에서 이미지 업로드하기 
  1. [[Swift] - MultiPart통신 (멀티파트 이미지업로드)](https://nsios.tistory.com/39) 
  2. [Uploading images and forms to a server using URLSession](https://www.donnywals.com/uploading-images-and-forms-to-a-server-using-urlsession/) 
  3. [How to Upload Image to Imgur in iOS using Swift](https://johncodeos.com/how-to-upload-image-to-imgur-in-ios-using-swift/) 
  4. [Upload image to server using URLSessionUploadTask](https://fluffy.es/upload-image-to-server/)
  5. [[Swift] - MultiPart통신 (멀티파트 이미지업로드)](https://nsios.tistory.com/39)

