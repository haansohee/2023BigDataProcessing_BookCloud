# BookCloud: 서울 도서관의 일일 인기 대출 도서를 알아보자!

# 빅데이터처리: 데이터 시각화

## 주제 선정

도서관에 가면 하루 전의 일일 인기 대출 도서가 궁금하진 않으신가요? 아니면, 인기 대출 도서의 줄거리를 한눈에 파악하고 싶진 않으신가요? 👀
<br> 이 정보들을 한눈에 파악해 봅시다!

<hr>

## 사용한 프로그래밍 언어와 라이브러리

**python 3.10, matplotlib, pandas, selenium, plotly, WordCloud**

## 사용한 데이터

<li> 도서 데이터: <a href="https://data.seoul.go.kr/dataList/OA-15475/S/1/datasetView.do"> 서울 열린 데이터 광장 </a></li>
<li> 도서 카테고리 정보: <a href="https://developers.naver.com/products/intro/plan/plan.md"> Naver Developers </a> </li>

<br>

## 결과

### 2023년 11월 5일의 Top 100 도서 카테고리 

![newplot-2](https://github.com/haansohee/2023BigDataProcessing_BookCloud/assets/90755590/34dbf60b-ab91-447b-8778-3f2c74ba9486)

카테고리가 **한국소설**인 책이 11월 5일 일자에 가장 많이 대출이 됐습니다. <br>
데이터를 조금 더 간추려서 봅시다. Top10의 도서는 무엇일까요?

![newplot-3](https://github.com/haansohee/2023BigDataProcessing_BookCloud/assets/90755590/e5e25f70-53d6-46b7-a906-b2e5538834d6)


**나미야 잡화점의 기적** 도서가 가장 인기가 많았네요.

![newplot-4](https://github.com/haansohee/2023BigDataProcessing_BookCloud/assets/90755590/178e1e61-8c10-4217-82ef-bacf4bd009bf)

<br>

Top100 도서에서 인기가 많았던 카테고리 3개는 **한국소설, 영미소설, 교양인문**, 그리고 Top10 도서에서 인기가 많았던 카테고리 3개는 **한국소설, 일본소설, 교양인문**이었어요.

나미야 잡화점의 기적(일본소설)이 가장 인기가 많았던 걸 다시 한번 알 수 있습니다.

<br>

### Top 10 도서 워드클라우드 결과

<img width="1243" alt="스크린샷 2023-11-06 오후 10 45 47" src="https://github.com/haansohee/2023BigDataProcessing_BookCloud/assets/90755590/5b31069b-73a0-40dd-b852-aeccd1c60b5a">




