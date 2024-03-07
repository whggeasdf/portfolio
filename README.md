# :pushpin: Carboard
> 대시보드기반 탄소중립 실천 서비스  

</br>

## 1. 제작 기간 & 참여 인원
- 2023-11-28~2024-1-5
- 최고봉
- 박은선
- 이경근
- 선준혁
- 김종찬
  
</br>

## 2. 사용 기술
#### `Back-end`
  - java 
  - python
  - Apache Tomcat 9.0
  - Oracle DateBase
#### `Front-end`
  - HTML
  - CSS
  - JS

</br>

## 3. ERD 설계
![](https://zuminternet.github.io/images/portal/post/2019-04-22-ZUM-Pilot-integer/final_erd.png)
![](https://github.com/JungHyung2/gitio.io/blob/master/assets/images/portfolio/p1.jpg)


## 4. 핵심 기능

이 서비스의 핵심 기능은 데이터 시각화입니다. <br/>
탄소 중립과 관련된 데이터를 공공데이터포털 오픈API에서 가져와 Chart.js 라이브러리를 사용하여 시각적으로 표현하였습니다.<br/>

<details>
<summary><b>핵심 기능 설명 펼치기</b></summary>
<div markdown="1">

### 4-1. 공공데이터 API를 활용하여 탄소배출량, 평균기온, 폭염일수 등 데이터 시각화<br/>

#### 국가 온실가스 인벤토리 배출량 공공데이터 API 활용<br/>
- 인증키 발급 : 공공데이터포탈(https://www.data.go.kr)에서 오픈API 서비스의 [SERVICE KEY]를 발급 받아야 함<br/><br/>

#### 기타 데이터 출처<br/>
- 평균기온 (지역별,계절별)<br/>
(https://data.kma.go.kr/climate/RankState/selectRankStatisticsDivisionList.do)<br/>
- 폭염 일수<br/>
(https://data.kma.go.kr/climate/heatWave/selectHeatWaveChart.do)<br/>
- 탄소배출량 (지역별), 배출원별, 온실가스 비중<br/>
(https://www.data.go.kr/data/15049589/fileData.do)<br/>

</div>
</details>

</br>

## 5. 핵심 트러블 슈팅
### 5.1. 탄소배출량 계산 문제
- 로그인, 회원가입 기능처럼 JSP를 활용하고자 하였으나 데이터베이스 안의 데이터를 자바로 꺼내와서 계산 후 다시 DB에 넣는 로직이 비효율적이고 유지, 보수에 불리하다고 생각하였습니다.
- 다른 효율적인 방법을 찾던 중  DB멘토링을 통해 트리거라는 것을 배웠습니다.  데이터베이스 이벤트에 반응하여 실행되는 프로그램 단위인 오라클 트리거를 활용해 DB에 연료의 종류와 연료량이 입력되기 전에 트리거가 실행되어 DB내에서 탄소배출량을 산정하여 DB에 바로 저장할 수 있도록 하였습니다. 

<details>
<summary><b>개선된 코드</b></summary>
<div markdown="1">

~~~SQL
CREATE OR REPLACE TRIGGER TRG_CARBON_CAL
BEFORE INSERT ON TBL_MEMBER_CO2
REFERENCING NEW AS NEW FOR EACH ROW
DECLARE 
    
    V_NET  NUMBER(12,4); -- TBL_CO2_REF.NET_CALORIFIC_VAL%TYPE; 
    V_CO2  NUMBER(12,4);
    V_CH4  NUMBER(12,4);
    V_N2O  NUMBER(12,4);
    
    RESULT_CO2 NUMBER(12,4);
    RESULT_CH4 NUMBER(12,4);
    RESULT_N2O NUMBER(12,4);
    

BEGIN
    SELECT NET_CALORIFIC_VAL, CO2_E_FACTOR, CH4_E_FACTOR, N2O_E_FACTOR
    INTO V_NET, V_CO2, V_CH4, V_N2O
    FROM TBL_CO2_REF
    WHERE FUEL_NAME = :NEW.MEM_FUEL_NAME AND TRANSPORTATION = :NEW.TRANSPORTATION;
    
    RESULT_CO2 := :NEW.FUEL_AMOUNT * V_NET * V_CO2 * 0.000001;
    RESULT_CH4 := :NEW.FUEL_AMOUNT * V_NET * V_CH4 * 0.000021;
    RESULT_N2O := :NEW.FUEL_AMOUNT * V_NET * V_N2O * 0.000310;
    
    :NEW.CO2_EMISSION := RESULT_CO2 * 0.001;
    :NEW.CH4_EMISSION := RESULT_CH4 * 0.001;
    :NEW.N2O_EMISSION := RESULT_N2O * 0.001;
    :NEW.TOTAL_EMISSION := :NEW.CO2_EMISSION + :NEW.CH4_EMISSION + :NEW.N2O_EMISSION;

   
END;
~~~

</div>
</details>

</br>
