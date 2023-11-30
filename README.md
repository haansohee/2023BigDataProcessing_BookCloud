# BookCloud: 사서 추천 도서와 도서관 인기 도서 알아보기
## 빅데이터처리: 데이터 시각화

사서 추천 도서를 도서관 이용자들이 많이 찾을까요?

도서관 이용자들이 가장 많이 찾는 책은 무엇일까요?

한눈에 파악해 봅시다!

<hr>

## 사용한 프로그래밍 언어와 라이브러리

**python 3.10, matplotlib, pandas, selenium, plotly, WordCloud**

## 사용한 데이터

<li> 도서 데이터: <a href="https://data.seoul.go.kr/dataList/OA-15475/S/1/datasetView.do"> 서울 열린 데이터 광장 - 서울 도서관 </a></li>
<li> 도서 카테고리 정보: <a href="https://developers.naver.com/products/intro/plan/plan.md"> Naver Developers - 책 검색 API </a> </li>
<li> 사서 추천 도서: <a href="https://www.nl.go.kr/NL/contents/N31101030900.do"> 국립중앙도서관 </a></li>

<br>

## 분석 과정

### (1) 인기 도서 대출 csv 파일을 데이터프레임 형태로 가져와 ISBN 컬럼의 결측치를 제거합니다.

```python
book_data = pd.read_csv('/content/20231127_100Book.csv', engine='python', encoding='CP949')
book_data.dropna(subset=['ISBN번호'], how='any', axis=0, inplace=True)
```

<img width="1107" alt="스크린샷 2023-11-29 오전 10 46 20" src="https://github.com/haansohee/2023BigDataProcessing_BookCloud/assets/90755590/e6006c16-e9f1-41e0-a301-e5d9829cf3e7">

<br>

### (2) 사서 추천 도서 목록을 가져옵니다. (인기 도서 대출이 60일 누적된 자료이므로 9월~10일, 두 달 간의 데이터를 가져옵니다.
```python
response = requests.get(url, params=params)
results = BeautifulSoup(response.content, 'xml')
for result in results.findAll('item'):
  if any(char.isspace() for char in result.recomisbn.text):
    isbn = result.recomisbn.text.split()
    dict['제목'] = result.recomtitle.text
    dict['작가'] = result.recomauthor.text
    dict['ISBN'] = isbn[1]
    list.append(dict)
  else:
    dict['제목'] = result.recomtitle.text
    dict['작가'] = result.recomauthor.text
    dict['ISBN'] = result.recomisbn.text
    list.append(dict)

...

librarian_recommended_books = pd.DataFrame(list)
# 공백이 있는 행은 NaN으로 변경
librarian_recommended_books.replace('', pd.NA, inplace=True)
# ISBN 컬럼 결측치 제거
librarian_recommended_books.dropna(subset=['ISBN'], how='any', axis=0, inplace=True)
```

<img width="858" alt="스크린샷 2023-11-29 오전 10 46 26" src="https://github.com/haansohee/2023BigDataProcessing_BookCloud/assets/90755590/7b94b630-fe98-4e36-bc17-8c2d05693ade">

<br>

**사서 추천 도서의 대출 횟수 데이터 시각화**

![image](https://github.com/haansohee/2023BigDataProcessing_BookCloud/assets/90755590/5d63ff6e-f522-4dee-816e-5cd520761f92)

<Br>

### (3) (1)에서 가져온 인기 대출 도서 데이터를 Top100 인기 대출 도서 데이터로 조정합니다.

```python
top100_book_data = []
while True:
    index = book_data['대출횟수'].idxmax()
    top100_book_data.append(book_data.iloc[index])


    if len(top100_book_data) == 100:
        top100_book_data = pd.DataFrame(top100_book_data, columns=['제어번호', '제목', '저자', '발행처', '발행년도', 'ISBN번호', '분류기호', '대출횟수'])
        top100_book_data.reset_index(drop=True, inplace=True)
        break

    book_data.drop(index, inplace=True)
```

<img width="1183" alt="스크린샷 2023-11-29 오전 10 47 06" src="https://github.com/haansohee/2023BigDataProcessing_BookCloud/assets/90755590/67a3f43c-1517-44ff-a6b1-1e2ed8f2ed95">

<br>

### (4) Top100 인기 도서 대출 데이터에 카테고리, 설명 칼럼을 추가합니다. 그 후 카테고리, 설명 컬럼의 결측치를 제거합니다.

```python
...
for i in range(0, len(top100_book_data)):

  book_isbn = urllib.parse.quote(top100_book_data.iloc[i][5])
  url = 'https://openapi.naver.com/v1/search/book_adv.json?d_isbn=' + book_isbn
  request = urllib.request.Request(url)
  request.add_header("X-Naver-Client-Id", id)
  request.add_header("X-Naver-Client-Secret", secret)
  response = urllib.request.urlopen(request)
  response_code = response.getcode()

  if response_code == 200:
    response_body = response.read().decode('utf-8')
    result = json.loads(response_body)['items']

    if not result or len(result) == 0:
      result_description = '?'
      result_category = '?'
    else:
      result_description = result[0]['description']
      book_link = result[0]['link']
      driver.get(book_link)
      result_category = driver.find_element(By.XPATH,'//*[@id="book_section-info"]/div[2]/ul/li[1]/div/div[2]').text

    description.append(result_description)
    category.append(result_category)

  else:
    print("Error Code: ", reponse_code)

top100_book_data['설명'] = description
top100_book_data['카테고리'] = category

...

# 카테고리와 설명 컬럼 결측치 제거
top100_book_data.replace('?', pd.NA, inplace=True)
top100_book_data.dropna(subset=['카테고리'], how='any', axis=0, inplace=True)
top100_book_data.dropna(subset=['설명'], how='any', axis=0, inplace=True)
```

<img width="1655" alt="스크린샷 2023-11-29 오전 10 57 24" src="https://github.com/haansohee/2023BigDataProcessing_BookCloud/assets/90755590/8a425c51-ea9c-4bf9-a04a-47fcc1031368">

<br>

### (5) Top100 인기 대출 도서를 카테고리로 그룹화하여 카테고리 별 인기 대출 도서를 알아봅니다.

```python
category_group = top100_book_data.groupby(['카테고리'])['대출횟수'].sum()
category_group = pd.DataFrame(category_group, columns=['대출횟수'])
```

**인기 카테고리**
![image](https://github.com/haansohee/2023BigDataProcessing_BookCloud/assets/90755590/2845b44d-d30f-4b24-9458-9372e29c24af)

<br>

### (6) Top100 인기 대출 도서를 조금 더 간소화하여 Top10 인기 대출 도서 데이터로 조정합니다. 

```python
...
while True:
    index = temp_top100_book['대출횟수'].idxmax()
    top10bookData.append(temp_top100_book.iloc[index])

    if len(top10bookData) == 10:
        top10_book_data = pd.DataFrame(top10bookData, columns=['제어번호', '제목', '저자', '발행처', '발행년도', 'ISBN번호', '분류기호', '대출횟수', '카테고리', '설명'])
        top10_book_data.reset_index(drop=True, inplace=True)
        break

    temp_top100_book.drop(index, inplace=True)
```

<img width="1437" alt="스크린샷 2023-11-29 오전 11 03 52" src="https://github.com/haansohee/2023BigDataProcessing_BookCloud/assets/90755590/b56f85c4-2daa-4cde-b576-4e17df0476e9">

***op10 인기 도서 대출 데이터 시각화**

![image](https://github.com/haansohee/2023BigDataProcessing_BookCloud/assets/90755590/95b0703a-d019-45ef-b281-f09e4f878eeb)

**Top10 인기 도서 카테고리 별 대출 데이터 시각화**

![image](https://github.com/haansohee/2023BigDataProcessing_BookCloud/assets/90755590/474b7532-60bb-491d-a80b-7fed13b8e328)

### (7) 사서 추천 도서 데이터에 카테고리 칼럼을 추가합니다.

```python
...
for i in range(0, len(librarian_recommended_books)):

  book_isbn = urllib.parse.quote(librarian_recommended_books.iloc[i][2])
  url = 'https://openapi.naver.com/v1/search/book_adv.json?d_isbn=' + book_isbn
  request = urllib.request.Request(url)
  request.add_header("X-Naver-Client-Id", id)
  request.add_header("X-Naver-Client-Secret", secret)
  response = urllib.request.urlopen(request)
  response_code = response.getcode()

  print(i, ':', book_isbn)

  if response_code == 200:
    response_body = response.read().decode('utf-8')
    result = json.loads(response_body)['items']

    if not result or len(result) == 0:
      print("?")
      result_category = '?'
    else:
      print(result)
      book_link = result[0]['link']
      driver.get(book_link)
      result_category = driver.find_element(By.XPATH,'//*[@id="book_section-info"]/div[2]/ul/li[1]/div/div[2]').text

    description.append(result_description)
    category.append(result_category)
    print(result_category)

  else:
    print("Error Code: ", reponse_code)

librarian_recommended_books['카테고리'] = category
...
```
<img width="921" alt="스크린샷 2023-11-29 오전 11 10 32" src="https://github.com/haansohee/2023BigDataProcessing_BookCloud/assets/90755590/c50dc2f7-0ebb-4ad3-83bd-4a5d31fa76d6">

**사서가 추천한 카테고리 별 개수 데이터 시각화**

![image](https://github.com/haansohee/2023BigDataProcessing_BookCloud/assets/90755590/9440d836-6da2-4968-bd52-324490029ac5)


<br>

### (8) 카테고리 칼럼이 추가된 사서 추천 도서 데이터에서 대출 여부가 존재하는 것만 나오도록 수정합니다.

```python
merge_df2 = pd.merge(book_data, librarian_recommended_books, how = 'inner', left_on='ISBN번호', right_on='ISBN')
merge_df2 =  merge_df2.drop('ISBN', axis=1)
merge_df2 = merge_df2.drop('제목_y', axis=1)
merge_df2 = merge_df2.drop('작가', axis=1)
merge_df2 = merge_df2.drop('total', axis=1)
merge_df2 = merge_df2.rename(columns={'제목_x':'제목'})
```

<img width="1200" alt="스크린샷 2023-11-29 오전 11 11 18" src="https://github.com/haansohee/2023BigDataProcessing_BookCloud/assets/90755590/e02b303a-58bc-42b4-bf0c-e540d216b56f">

**사서 추천 도서의 대출 여부가 있는 도서들의 인기 카테고리, 책 제목 데이터 시각화**

<img width="1427" alt="스크린샷 2023-11-30 오전 11 29 16" src="https://github.com/haansohee/2023BigDataProcessing_BookCloud/assets/90755590/d82dcbdf-3378-491c-a409-3ad9289d723c">


<hr>

## 위 데이터 분석의 결과

사서가 추천한 도서들을 도서관 이용자들이 잘 찾지 않는다는 것을 확인할 수 있습니다. 한국 소설 카테고리가 압도적으로 인기 많은 도서 카테고리인 것, **소년이 온다**라는 책이 압도적\으로 인기가 많은 도서인 것을 알 수 있습니다. 사서가 자주 추천하는 도서 카테고리는 **성공/처세**지만, 인기 대출 도서 데이터(Top100, Top10 인기 대출 도서)를 통해 도서관 이용자들은 **한국 소설**을 많이 찾는다는 것 또한 알 수 있었습니다.

 <hr>

 ## Top10 인기 대출 도서들의 책 설명 워드 클라우드
✔️ 도서들의 설명글을 한눈에 파악하기 위한 데이터 시각화입니다.

### (1) {제목: 설명} 형태의 딕셔너리를 생성하여 도서 설명의 품사를 태깅 합니다. 필요 없는 단어는 stopword 파일에 추가하여 태깅 되지 않도록 설정합니다.

```python
...
for i in range(0, len(top10_book_data)):
  title = top10_book_data.iloc[i][1]
  description = top10_book_data.iloc[i][9]
  book_description[title] = description
...
stop_word_file = '/content/stopword.txt'
stop_file = open(stop_word_file, 'rt', encoding='utf-8')
stop_words = [ word.strip() for word in stop_file.readlines()]

for key, value in book_description.items():
  nouns = okt.nouns(value)
  filtered_nouns = [word for word in nouns if word not in stop_words]
  description_tag[key] = filtered_nouns
```

<br>

## (2) 단어 빈도를 탐색합니다.
```python
...
for key, value in description_tag.items():
  count = Counter(value)
  print('책 제목: ', key)
  word_count = {}
  for tag, counts in count.most_common(80):
    if(len(str(tag)) > 1):
      word_count[tag] = counts
      print("%s: %d" % (tag, counts))
  word_dict[key]=word_count
```

<br>

## (3) 임의로 만든 워드클라우드 마스크 이미지를 이용하여 워드클라우드를 생성합니다.

<img width="212" alt="book_mask_image" src="https://github.com/haansohee/2023BigDataProcessing_BookCloud/assets/90755590/4cea31e9-0c83-4806-b7ac-6a9036372026">

프로젝트 목적과 비슷하도록 책 모양으로 만들었습니다.


```python
book_image_path = '/content/book_mask_image.png'

word_cloud = WordCloud(font_path,
                       background_color = 'white',
                       width=800,
                       height=600,
                       colormap='Paired',
                       mask=np.array(Image.open(book_image_path))
                       )

for key, value in word_dict.items():
  print(key)
  cloud = word_cloud.generate_from_frequencies(value)
  plt.figure(figsize=(8,8))
  plt.imshow(cloud, interpolation='bilinear')
  plt.axis('off')
  plt.show()
```

### 워드클라우드 이미지 중 하나의 결과와 원본 설명글의 비교입니다. 

<img width="1341" alt="image" src="https://github.com/haansohee/2023BigDataProcessing_BookCloud/assets/90755590/d7c55f9c-11b6-4d08-8d1f-000777caa3d9">

