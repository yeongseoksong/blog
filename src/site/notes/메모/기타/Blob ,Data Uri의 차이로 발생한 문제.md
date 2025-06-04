---
{"dg-publish":true,"permalink":"///blob-data-uri/","title":"Bob 이란","tags":["javascript","python"]}
---


React-WebCam 라이브러리 를 사용해 Data Uri를 OCR 서버 에 이미지를 전송하는 기능을 작업하는중 계속해서 422 에러 를 반환 했다 .

canvas 를 활용해 화면을 캡쳐할때 Data Uri 형식으로 캡쳐되는데 
이를 서버에서 이미지로 인식하지 못하여 에러가 발생 했던것이다.
이를 해결 하기 위해서는 두가지 방법이 있다.

1.  Data Uri 를 Blob 으로 변환후에 서버에 전달
2.  서버에서 Data Uri 의 실제 이미지 부분을 디코딩 (base64)

[[메모/JavasScript/Blob , Data Uri 란 무엇일까\|Blob , Data Uri 란 무엇일까]]


#### 1. 클라이언트에서 해결법  (js)
---

```javascript
function data_uri_to_blob(dataURI) {
  const byteString = atob(dataURI.split(",")[1]);
  const mimeString = dataURI.split(",")[0].split(":")[1].split(";")[0];

  const ab = new ArrayBuffer(byteString.length);
  const ia = new Uint8Array(ab);

  for (let i = 0; i < byteString.length; i++) {
    ia[i] = byteString.charCodeAt(i);
  }

  return new Blob([ab], { type: mimeString });

}

export { data_uri_to_blob };
```


#### 2. 서버에서 해결법 (python)
---
```python
def save_image_from_data_uri(data_uri, file_path): 
	_, encoded_data = data_uri.split(',', 1) 
	image_binary = base64.b64decode(encoded_data) 
	image = Image.open(BytesIO(image_binary)) 
	image.save(file_path)
```

```python
@app.post("/upload/") async def create_upload_file(file: UploadFile = File(None), data_uri: str = None): 
	try: 
		if file: 
		# 추가적으로 파일 확장자 명을 검사 가능.
		#allow_extensions= ['png','jpg'...]
		#if file.filename in allow_estensions:
			# next code  . . .
			with open(file.filename, "wb") as buffer: 
				buffer.write(file.file.read()) 
		if data_uri: 
			save_image_from_data_uri(data_uri, "decoded_image.jpg") 
		return JSONResponse(content={"message": "File uploaded successfully"}, status_code=200)
	 except Exception as e: 
		 return JSONResponse(content={"message": f"Error: {str(e)}"}, status_code=500)
```

DataUri 를 분해하여 코딩후 저장하는 함수를 선언한후에 파일의 형식에 따라 다르게 처리하는 로직이다.  Blob 형식 , data uri 형식 모두 에러 없이 수신 가능하다.