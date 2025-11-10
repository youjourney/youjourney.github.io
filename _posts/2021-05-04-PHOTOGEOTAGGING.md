---
title: "사진(이미지)에 직접 위치기록(geotagging) 하기"
excerpt: "사진(jpg, dng, raw...)에 GPS 정보를 입력하는 방법"
categories:
  - Development
tags:
  - GPS
  - GeoTag
  - Image
  - Metadata
  - Photography
last_modified_at: 2021-05-04T12:00:00+0900
---
### 개요

<p>GPS가 내장되지 않은 디지털카메라에 지오태깅을 위한 보조수단으로 앱을 사용하여 위치를 삽입할 때, 실수로 GPS track log를 기록하지 않았을 경우를 GPS기록을 임의(직접)로 넣는 방법을 설명합니다.</p>






### 시스템 환경

<h1>macOS 11.1 (Intel Mac) 기준</h1>

> ExifTool(Ver. 12.25 for macOS)

> TextEditor(1 of Atom, VSCode, TextEditor, VI, Nano, ..)






### ExifTool 참고 문서

ExifTool 다운로드는 [여기]( https://exiftool.org/ )<br/>
ExifTool Geotag Document는 [여기]( https://exiftool.org/geotag.html )<br/>
GPS 위도(latitude), 경도(longitude)는 [Google Map]( https://map.google.com/ )<br/>






### Geotagging(지오태깅 하기)

#### ExifTool 설치
<p>ExifTool을 위 [다운로드]( /#ExifTool-참고-문서 )에서 다운 받아 설치합니다.</p>







#### GPS 기록이 없는 사진 준비

<p>촬영한 사진이나 임의의 GPS 메타데이터가 없는 사진을 준비하고 사진의 타임스탬프를 확인합니다.<br/></p>

![서울시청]( {{ site.baseurl }}/images/2021-05-04-TECH1/seoul_cityhall.jpg ) 

![서울시청_이미지_meta]( {{ site.baseurl }}/images/2021-05-04-TECH1/image1.png )






#### GPS Track Log 만들기

<p>GPS TRACK LOG를 만들기 위해서는 사진의 타임스탬프, 해당 위치의 위도와 경도가 필요합니다.<br/></p>

<p>타임스탬프는 사진 파일 정보를 통해 확인할 수 있으며, 기록하려는 곳(주소)의 위치는 Google Map을 통해 확인할 수 있습니다.<br/></p>
![서울시청_위치_GMap]( {{ site.baseurl }}/images/2021-05-04-TECH1/image3.png )

<p>타임스탬프와 위치를 확인했다면,<br/>
이제 GPS TRACK LOG를 임의로 생성해야 합니다.<br/>
log는 메모장이나 텍스트에디터로 만들 수 있습니다. 텍스트에디터에서 빈 문서를 생성 후 아래 코드와 같이 기록한 뒤 "gps.txt"로 사진이 저장된 폴더에 함께 저장합니다.<br/></p>
```
GPSDateTime,GPSLatitude,GPSLongitude
2021:05:04 10:54:00,37.5662952,126.9757511
```
![서울시청_위치_log_gps.txt]( {{ site.baseurl }}/images/2021-05-04-TECH1/image2.png )




#### GPS Track Log를 사진에 기록하기

<p>터미널(Terminal) App을 실행한 뒤, 포커스를 사진과 gps.txt가 저장된 폴더로 이동시킵니다.</p>

> cd <사진과 gps.txt기 저장된 폴더의 경로>

![terminal_포커스_이동]( {{ site.baseurl }}/images/2021-05-04-TECH1/image4.png )



<p>exiftool -기능 "gps트랙파일" "사진파일"</p>

> exiftool -geotag gps.txt -geosync=+3600 '-geotime<${DateTimeOriginal}+00:00' seoul_cityhall.jpg







### 참고 문헌

서울 시청 이미지 [여기]( http://data.si.re.kr/collection/view/363 )<br>