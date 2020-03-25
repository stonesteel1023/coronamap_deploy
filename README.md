# 코로나맵서비스 클론 테스트
- by 최석철(stonesteel1023)
- 출처 : https://github.com/nero96in/coronamap_deploy.git
- 아직 안한 것 (API 추가)
    - 1. [공공 데이터 포털 공적 마스크 판매 현황 API](https://app.swaggerhub.com/apis-docs/Promptech/public-mask-info/20200307-oas3#/)
    - 2. [Kakao map API](http://apis.map.kakao.com/)
    - 3. 서비스 배포 방법 [AWS EC2 서비스를 통한 Django 배포 방법](https://nerogarret.tistory.com/45) 참고

## Django를 이용한 지역 코로나맵 제작 Template
[울산 코로나맵](https://coronamap-ulsan.site/) 개발진들이 **각 지역의 코로나맵 제작**을 좀 더 쉽고 빠르게 제작할 수 있도록 배포한 Django APP입니다. 아래의 가이드라인을 따라 프로젝트를 제작하여 배포하실 수 있습니다. 자신의 지역의 코로나맵을 제작하여 코로나 바이러스 방역에 힘써주세요!

#### Kakaomap API 설정 관련
1. `main/static/main/js/corona.js`와 `main/static/main/js/patient_admin.js` 파일에서 `defaultx`와 `defaulty` 변수를 자신의 **지역의 중심좌표**로 수정
2. `main/static/main/js/corona.js`의 `searchPlace` 함수 내에 진료소 검색을 위한 키워드(`울산 코로나 진료소`로 되어 있는 부분)를 수정
3. `main/templates/main/index.html`와  `main/templates/main/patient_admin.html`의 head 태그에 `{API키를 입력하세요}` 부분에 자신의 kakaomap API key를 입력

#### 확진자 정보 및 동선 정보 크롤링 관련 (`main/views.py`)
`main/views.py` 파일에 공적 마스크 판매 정보(`get_ulsan_mask_stores`), 확진자 현황(`get_ulsan_status`), 확진자 정보 및 동선(`get_ulsan_status`)을 10분에 한번씩 실행하여 데이터베이스에 저장하는 함수들이 있습니다. 이 함수들을 각 지역에 맞게 수정해 주시면 됩니다. API를 사용해 보셨거나 `bs4`를 이용한 크롤링을 해보셨다면 아래의 가이드라인을 따라 쉽게 만드실 수 있습니다. 아래의 가이드는 **각 함수 내에 주석**으로 수정해야 할 부분이 표시되어 있으니, 그 부분을 따라서 수정해주시면 좀 더 편합니다.  

함수를 모두 구현하셨다면, 서비스 배포 직전(혹은 테스트를 위한 `runserver` 직전에)에 각 함수들을 `python3 manage.py shell`에서 한 번씩 실행하여 **데이터베이스를 업데이트(초기화)** 해주셔야 합니다. 이에 대한 설명은 함수 설명 후, 자세히 다루도록 하겠습니다.

#### 1. `get_mask_stores`
1. 공적 마스크 현황 정보를 가져오고 데이터베이스(`Mask` 모델)에 저장하는 함수. 매 실행시 데이터가 업데이트되는 방식이니 데이터가 쌓이지 않음.
2. `gu_list` 변수에 자신의 지역의 모든 시, 군, 구를 리스트 형태로 입력. ex) 경상남도 창원시 마산합포구, 울산광역시 울주군

#### 2. `get_status`
1. 지역의 확진자 현황 정보를 가져오고 데이터베이스(`Statistics` 모델)에 저장하는 함수. 확진자 수, 완치자 수만 수합하도록 짜여있으나 추가 통계 자료는 커스터마이징 가능. 매 실행시 데이터가 추가되는 것이 아닌, 업데이트되는 방식이니 데이터가 쌓이지 않음.
2. 확진자 현황 정보를 보고하는 각 지자체 공식 홈페이지를 크롤링하여 사용용. **따라서, 별도의 크롤링 코드 필요.**
    1. `values` 변수에 `[<확진자 수>, <완치자 수>]` 형식의 리스트가 저장되도록 크롤링 진행.
    
#### 3. `get_patient`
1. 각 확진자에 대한 정보와 동선 정보를 가져오고 데이터베이스(`Patient` 모델)에 저장하는 함수. 매 실행시, **환자 번호를 기준으로** 업데이트되는 방식.
2. `patients` 변수가 아래의 형식으로 저장되도록 크롤링 진행. (이 변수의 `key`는 환자 번호입니다.)
~~~
patients = {
	1: {
		“ID”: 환자 식별자,
		“Gender”: 환자 성별,
		“Age”: 환자 나이,
		“Region”: 환자 지역,
		“Confirmed Date”:  확진 날짜,
		“Current Status”: 현재 상태(격리 장소, 퇴원 여부 등 기타 정보),
		“Paths”: raw 환자 동선 정보,
	},
	2: {
		“ID”: 환자 식별자,
		“Gender”: 환자 성별,
		“Age”: 환자 나이,
		“Region”: 환자 지역,
		“Confirmed Date”:  확진 날짜,
		“Current Status”: 현재 상태(격리 장소, 퇴원 여부 등 기타 정보),
		“Paths”: raw 환자 동선 정보,
	},
	…
}
~~~

### 공지사항
공지 사항은 index.html 파일의 `modal`요소에서 수정하실 수 있습니다.
소스코드 사용 시 공지사항에 출처를 남겨주세요 :)
