# 지능화 기술 생태계 분석을 위한 데이터 수집 및 가공 (Data collection and processing for intelligent technology ecosystem analysis)

## Packages
1. sklearn(TfidfVectorizer, CountVectorizer, PCA)
2. KMeans
3. DBSCAN
4. 
5. 
6. 

## Index
- Research Proceduer
  * Item-based Technology Type Analysis
  * Topic-based Technology Type Analysis
- Research Result (Github's key Repository Analysis)
  * Star-based
  * Big Tech Company
  * Future Tech : Autonomous-vehicle, Metaverse


## Research
<img width="525" alt="스크린샷 2022-03-02 오후 3 49 56" src="https://user-images.githubusercontent.com/87521259/156309915-04dd1918-b0c1-4f0c-b8f3-0fd2284caf04.png">

### 1. 데이터 수집 및 전처리

   1) 깃허브 오픈소스 정보 및 API 분석

   - 깃허브는 대용량의 오픈소스 정보를 효과적으로 확인하기 위한 도구로 API를 제공하고 있으며, 개발자, 개발환경, 현황 등 여러 기술속성을 검색할 수 있도록 지원하고 있음.
   - 아래 url을 통해 "deep learning" 키워드 검색 결과에 대한 정보를 웹페이지 상에서 API 형태로 확인 가능
   https://api.github.com/search/repositories?q=deep%20learning&page,per_page,sort,order
  
  <img width="1273" alt="스크린샷 2022-03-02 오후 3 47 13" src="https://user-images.githubusercontent.com/87521259/156309578-501ba7dd-f673-44b4-8742-2ab95e4c2e00.png">

   2) API 이용하여 필요한 저장소 정보 Crawling test
  
   ```
   def topic(t):

     topic = t.replace(' ', '%20')
     response = urlopen('https://api.github.com/search/repositories?q={}&page,per_page,sort,order'.format(topic)).read().decode('utf-8')

     responseJson = json.loads(response)

     name_lst = []
     type_lst = []
     create_lst = []
     size_lst = []
     star_lst = []
     fork_lst = []
     login_lst = []

     items = responseJson.get('items')

     for lst in items:
         name = lst.get('name')
         typ = lst.get('owner').get('type')
         create = lst.get('created_at')
         size = lst.get('size')
         star = lst.get('stargazers_count')
         fork = lst.get('forks_count')
         login = lst.get('owner').get('login')

         name_lst.append(name)
         type_lst.append(typ)
         create_lst.append(create)
         size_lst.append(size)
         star_lst.append(star)
         fork_lst.append(fork)
         login_lst.append(login)

     df = pd.DataFrame([name_lst, type_lst, create_lst, size_lst, star_lst, fork_lst, login_lst])
     df = df.transpose()
     df.columns = ['name','type','created_at','size','stargazers_count','fork','login']
     return df

  # test
  topic('deep learning')
  ```
  <img width="852" alt="스크린샷 2022-03-02 오후 3 48 58" src="https://user-images.githubusercontent.com/87521259/156309815-3d802eca-173e-4956-a84f-0e02e350bb27.png">

   3) 전체 페이지 Json 형태로 response, Crawling 및 Excel 형태 저장
   - Github 자체에서 인터페이스 기반의 페이지 변화와 자동 크롤링 방지를 위한 요청시간 확인 등의 이슈사항 존재 : time.sleep() 함수를 통해 대기시간 발생시켜 반복적인 동적 Crawling
   ```
   def topic(t):

     topic = t.replace(' ', '%20')
     name_lst = []
     type_lst = []
     create_lst = []
     size_lst = []
     star_lst = []
     fork_lst = []
     login_lst = []

     for i in range(1,11):

         try:
             response = urlopen('https://api.github.com/search/repositories?q={}&sort=stars&per_page=100&page={}'.format(topic, i)).read().decode('utf-8')
         except:
             time.sleep(10)

         responseJson = json.loads(response)

         print(f'{i} response')

         items = responseJson.get('items')

         for lst in items:
             name = lst.get('name')
             typ = lst.get('owner').get('type')
             create = lst.get('created_at')
             size = lst.get('size')
             star = lst.get('stargazers_count')
             fork = lst.get('forks_count')
             login = lst.get('owner').get('login')

             name_lst.append(name)
             type_lst.append(typ)
             create_lst.append(create)
             size_lst.append(size)
             star_lst.append(star)
             fork_lst.append(fork)
             login_lst.append(login)

 #         print('{} / {} / {} / {} / {} / {} / {}'.format(name, typ, create, size, star, fork, login))

     df = pd.DataFrame([name_lst, type_lst, create_lst, size_lst, star_lst, fork_lst, login_lst])
     df = df.transpose()
     df.columns = ['name','type','created_at','size','stargazers_count','fork','login']
     return df

   # test
   topic('deep learning')
   ```
   <img width="817" alt="스크린샷 2022-03-02 오후 4 30 11" src="https://user-images.githubusercontent.com/87521259/156315060-db89266d-baa6-4450-b07c-eb6212785f3f.png">
   
   -> 위와 같은 Crawling 방식으로 상위 인지도(star), 빅테크 기업(Google, MS, Intel, Facebook, Apple, Amazon 등), 미래기술(자율주행차, 메타버스) 기준 Crawling 및 데이터베이스화
   
### 2. 기술 분석

  1) 인지도(star) 기반 깃허브 주요 저장소 분석

   - project name과 topic keyword을 column으로 한 Crawling 결과 데이터베이스 불러오기
   
   ```
    df = pd.read_excel("Topics_stars10000_project_keyword.xlsx") #star>10000개가 있는 Df
    # df = pd.read_excel("Topics_keywords.xlsx")   # vehicle Data가 들은 Df
    df.drop_duplicates(['project_name'])
    df.head(50)
   ```
    
   <img width="559" alt="스크린샷 2022-03-02 오후 6 33 11" src="https://user-images.githubusercontent.com/87521259/156334602-a5669b6b-ee2f-4366-aaa6-beb87f71f351.png">

   - 비슷한 토픽끼리 
