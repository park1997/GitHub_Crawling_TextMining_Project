# 지능화 기술 생태계 분석을 위한 데이터 수집 및 가공 (Data collection and processing for intelligent technology ecosystem analysis)

## Packages
1. sklearn(TfidfVectorizer, CountVectorizer, PCA)
2. KMeans
3. DBSCAN

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

  #### 1) 인지도(star) 기반 깃허브 주요 저장소 분석

   - project name, topic keyword, star number를 column으로 Crawling 실행
   
   ```
    stars = input("스타수 입력 : ")
    url = "https://github.com/search?p=1&q=stars%3A%3E{}&type=Repositories".format(stars)

    def crawling_func(url):
        print(url)
        try:
            res = requests.get(url)
            res.raise_for_status()
            soup = BeautifulSoup(res.text,"lxml")
            p = soup.find_all("a",attrs={"class":"v-align-middle"}) 
        except:
            time.sleep(1)
            crawling_func(url) # 오류를 대비하기위해 재귀함수 호출
            pass
        finally:
            return p

    topic_ads = []
    pages = int(input("검색할 페이지 수(Ex : 10) : "))
    print("{}개 이상의 star수를 가진 Repository의 상위 {}페이지를 크롤링합니다. ".format(stars,pages))
    print()
    for i in range(1,pages+1):# x 페이지까지 탐색
        url = "https://github.com/search?p={}&q=stars%3A%3E10000&type=Repositories".format(i) # 페이지 formatting
        time.sleep(15)  # 10초를 쉬어도 오류가 떴었음
        for j in crawling_func(url):
            topic_ads.append(j.get_text())
            print(j.get_text())


    topics_dic = {}
    topics_list = []
    for ad in topic_ads:
        url_topic = "https://github.com/" + ad
        res_topic = requests.get(url_topic)
        res_topic.raise_for_status()
        soup_topic = BeautifulSoup(res_topic.text,"lxml")

        topic = soup_topic.find("div",attrs={"class":"BorderGrid-cell"}).find_all("a",attrs={"class":"topic-tag topic-tag-link"})
        project_topics=[i.get_text().replace("\n","").replace("\t","").strip() for i in topic]

        # 스타 수(북마크 수, 인기도)
        star_num = soup_topic.find("ul",attrs={"class":"pagehead-actions flex-shrink-0 d-none d-md-inline"}).find("a",attrs={"class":"social-count js-social-count"}).get_text()
        star_num = star_num.replace('\t', '').replace('\n', '').strip()

        topics_list.append([ad,project_topics,star_num])

        for t in project_topics:
            if t in topics_dic:
                topics_dic[t] += 1
            else:
                topics_dic[t] = 1

    # value로 내림차순 정렬
    topics_dic= sorted(topics_dic.items(), key=lambda x: x[1], reverse=True)
    df_topic2 = pd.DataFrame(topics_list,columns=['project_name','topic_keyword','star_number'])
    # 엑셀로 저장하기

    df_topic2.to_excel('Topics_stars{}_project_keyword.xlsx'.format(stars),index=False)
    print()
    print("star 수가 {} 개 이상인 프로젝트 상위 {}페이지에대한 Data가 저장되었습니다.".format(stars,pages))
    print("종료 .")

   ```
    
   <img width="559" alt="스크린샷 2022-03-02 오후 6 33 11" src="https://user-images.githubusercontent.com/87521259/156334602-a5669b6b-ee2f-4366-aaa6-beb87f71f351.png">

   - 각 Project별 Topic Keyword 벡터화 진행 (TF-IDF) 
      -> 유의어끼리 서로 묶는 작업 및 Topic의 누적합을 딕셔너리 형태로 만드는 작업 후 진행

   ```
   vectorize = TfidfVectorizer(
       min_df=5    # 예제로 보기 좋게 1번 정도만 노출되는 단어들은 무시하기로 했다
                   # min_df = 0.01 : 문서의 1% 미만으로 나타나는 단어 무시
                   # min_df = 10 : 문서에 10개 미만으로 나타나는 단어 무시
                   # max_df = 0.80 : 문서의 80% 이상에 나타나는 단어 무시
                   # max_df = 10 : 10개 이상의 문서에 나타나는 단어 무시
   )
   X = vectorize.fit_transform(df['topic_keyword_str'])
   print('fit_transform, (sentence {}, feature {})'.format(X.shape[0], X.shape[1]))

   # 문장에서 뽑아낸 feature 들의 배열
   features = vectorize.get_feature_names()

   X.toarray()
   ```
   <img width="631" alt="스크린샷 2022-05-07 오후 8 06 00" src="https://user-images.githubusercontent.com/87521259/167251598-da88a3c8-2924-4783-bc96-c47e8f8e5fe8.png">


   - 비슷한 Topic끼리 1차 DBSCAN Clustering
       -> PCA로 차원을 축소한 후 진행. 
       -> 여기서는 정보량의 손실이 5%만큼만 발생하도록 216차원에서 155차원으로 줄임

   ```
# 차원축소를 하지 않고도 PCA를 돌려봐보기

# 정보량이 95% 인 만큼의 칼럼수가 155임
pca = PCA(n_components=155)
df_pca = pca.fit_transform(tfidf_vector_df)
df_pca = pd.DataFrame(df_pca, index=tfidf_vector_df.index,
                      columns=[f"pca{num+1}" for num in range(df_pca.shape[1])])

for i in df_dbscan_cluster['clusters']model = DBSCAN(eps=0.4, min_samples=5, metric='cosine')

result = model.fit_predict(df_pca)
set(result)

df_result = df.copy()
df_result['result'] = result

j = 0
keyword = []
topic = []
num = []

for cluster_num in set(result):
    
    if(cluster_num == -1 or cluster_num == 0):
        continue # 노이즈 데이터들은 버림
    else:
        print('cluster num : {}'.format(cluster_num))
        temp_df = df_result[df_result['result'] == cluster_num]       
        
        i = 0
        
        for k in temp_df['topic_keyword_str']:
            keyword.append(k)
            num.append(cluster_num)
            i = i + 1
        # print('군집 내 데이터 개수: ',i)
        
        for t in temp_df['project_name']:
            topic.append(t)
    j += i
    print()
dic_cluster = {}
dic_cluster['topic'] = topic
dic_cluster['keyword'] = keyword
dic_cluster['number'] = num

clusters = {}
keywords = {}

num = []
for i in set(df_cluster['number']):
    n = 0
    cluster = []
    keyword = []
    for j in df_cluster.values:
        if j[2] == i:
            cluster.append(j[0])
            clusters[i] = cluster
            keyword += j[1].split(' ')
            n += 1
        else:
            pass
        keywords[i] = keyword
    num.append(n)

count_items = []

for i in keywords.values():
    count = {}
    for j in i:
        try:
            count[j] += 1
        except:
            count[j] = 1
    val = sorted(count.items(), key=lambda x: x[1], reverse=True)
    count_items.append(val[:15]) 

df_cluster_ = pd.DataFrame()
df_cluster_['clusters'] = clusters.values()
df_cluster_['cluster_num'] = clusters.keys()
df_cluster_['count'] = num
df_cluster_['top_15_topics'] = count_items
df_cluster_
   ```
   <img width="762" alt="스크린샷 2022-05-07 오후 8 15 35" src="https://user-images.githubusercontent.com/87521259/167251931-8b2a024f-c082-4ef3-82ac-58949a1d8fbe.png">

   - cluster_num = 1인 Topic끼리 2차 DBSCAN Clustering

```
num_list = []
for idx, num in enumerate(result):
    if num == 1:
        num_list.append(df_pca.iloc[idx])
    df_result2 = df_result[df_result['result'] == 1]
df_pca2 = pd.DataFrame(num_list)
df_pca2 = df_pca2.loc[~df_pca2.index.duplicated(keep='first')]
tfidf_vector_df2 = pd.merge(tfidf_vector_df, df_pca2, left_index=True, right_index=True, how='inner', sort=False)
tfidf_vector_df2.drop(tfidf_vector_df2.iloc[:,217:], axis=1, inplace=True)
tfidf_vector_df2.loc[~tfidf_vector_df2.index.duplicated(keep='first')]

ca = PCA(n_components=155)
df_pca2 = pca.fit_transform(tfidf_vector_df2)
df_pca2 = pd.DataFrame(df_pca2, index=tfidf_vector_df2.index,
                      columns=[f"pca{num+1}" for num in range(df_pca2.shape[1])])
                      
model2 = DBSCAN(eps=0.3, min_samples=5, metric='cosine') # parameter 값 재설정 필요

result2 = model2.fit_predict(df_pca2)
set(result2)

# 이후로는 위 클러스터링 방법과 동일
```
<img width="719" alt="스크린샷 2022-05-07 오후 8 18 03" src="https://user-images.githubusercontent.com/87521259/167251997-2faaf97c-5c72-4eb8-9e5c-b5e823f4ad23.png">


  #### 2) 빅테크 기업 저장소 분석
  

   - 검색할 기업이름 기준 정보 수집

```
# org = ["aws","facebook","google","naver","kakao","apple","alibaba","tencent","baidu","microsoft","samsung"]
org = list(input("검색하실 기업이름을 영어로 입력해주세요.(여러개 인경우 띄어쓰기로 구분하여 입력) : ").split())
org_dic={}
for o in org:
    url = "https://github.com/orgs/{}/repositories".format(o)
    print("{} 에 대한 정보 수집 시작.".format(url))
    res= requests.get(url)
    try:
        res.raise_for_status()
    except:
        print("입력하신 기업\"{}\" 에 대한 정보가 존재하지 않습니다.\n".format(o))
        continue
    soup=BeautifulSoup(res.text,"lxml")
    try:
        max_page = int(soup.find("div",attrs={"role":"navigation"}).find_all("a")[-2].get_text())
    except:
        max_page = 1
    item_temp = []
    for p in range(1,max_page+1):
        time.sleep(1)
        url = "https://github.com/orgs/{}/repositories?page={}".format(o,p)
        res= requests.get(url)
        res.raise_for_status()
        soup=BeautifulSoup(res.text,"lxml")
        print("{}{}수집시작{}".format("*"*10,o,"*"*10))
        for item in soup.find("div",attrs={"class":"org-repos repo-list"}).find_all("li",attrs={"class":"Box-row"}):
            print(item.a.get_text().strip())
            item_temp.append(item.a.get_text().strip())
    org_dic[o]=item_temp
    print()
```
<img width="598" alt="스크린샷 2022-05-08 오후 11 02 37" src="https://user-images.githubusercontent.com/87521259/167299903-fc425718-b5fd-414c-92f8-02632874ef2a.png">


   - DBSCAN Clustering

```
for o in org:
    print("{} {} DBSCAN 클러스터링 시작 {}".format("*"*10,o,"*"*10))
    excel_name = "{}_vectors.xlsx".format(o)
    df_org = pd.read_excel("{}.xlsx".format(o))
    df_vector = pd.read_excel(excel_name)
    # eps 값을 조정해나가면서 클러스터링을 해야 더 정확한 결과가 나온다
    dbscan = DBSCAN(eps = 0.3)
    dbscan_cluster = dbscan.fit_predict(df_vector)
    dbscan_cluster
    dbscan_clustered_dic = {}
    dbscan_clustered_list = []
    dbscan_cluster_num = len(set(dbscan_cluster))
    
    
    for idx,i in enumerate(dbscan_cluster):
        if i not in dbscan_clustered_dic:
            dbscan_clustered_dic[i] = [df_org['ProjectName'][idx]]
        else:
            dbscan_clustered_dic[i].append(df_org['ProjectName'][idx])

    # 클러스터링이 된 패키지들
    # 20개의 군집으로 생성

    dbscan_clustered_dic = sorted(dbscan_clustered_dic.items(), key=lambda x: x[0])

    df_dbscan_cluster = pd.DataFrame(dbscan_clustered_dic,columns=['num','clusters'])
    dbscan_cluster_num = [len(i) for i in df_dbscan_cluster['clusters']]
    df_dbscan_cluster['cluster_num'] = dbscan_cluster_num
    topic_dbscan_clustered_list = []
    for i in df_dbscan_cluster['clusters']:
        temp_dic = {}
        for j in i:
            topics = df_org[df_org['ProjectName']==j]['Topics'].values[0].replace("[","").replace("]","").replace("'","").strip().split(",")
            for i in topics:
                if len(i)==0:
                    continue
                i = same_things(i)
                if i not in temp_dic:
                    temp_dic[i] = 1
                else:
                    temp_dic[i] += 1
        temp_dic = sorted(temp_dic.items(), key=lambda x: x[1], reverse=True)
    #     print(temp_dic[:15]) # 상위 15개만 보여줌
    #     print()
        topic_dbscan_clustered_list.append(temp_dic[:15])
    df_dbscan_cluster['top_15_topics'] = topic_dbscan_clustered_list
    df_dbscan_cluster.to_excel("{}_DBSCAN_clusters.xlsx".format(o),index=False)
    print(df_dbscan_cluster)
    print("{}_DBSCAN_clusters.xlsx".format(o),"저장완료")
    print("*"*50)
    
```
<img width="677" alt="스크린샷 2022-05-08 오후 11 04 03" src="https://user-images.githubusercontent.com/87521259/167299961-e43f224c-8c38-4cac-b4cd-a59f396d91ea.png">


  #### 3) 미래기술 저장소 분석
  

   - 검색할 기술 이름 기준 정보 수집

```
topic_name = input("기술명을 입력해 주세요. : ")
url = "https://github.com/topics/{}?o=desc&s=stars".format(topic_name)
# 내 컴퓨터의 User_Agent
res= requests.get(url)
res.raise_for_status()
soup=BeautifulSoup(res.text,"lxml")
soup
options = webdriver.ChromeOptions()
# headless option임

options.add_argument("headless")
browser = webdriver.Chrome("./chromedriver",options=options)
# browser = webdriver.Chrome("./chromedriver")
browser.get(url)
soup = BeautifulSoup(browser.page_source,'lxml')
# Load more 을 몇번 누를것인지??
# 임의로 100번을 했지만 star수가 예를들어 x개 이상일때까지 Load_more버튼을 누르는 식으로도 가능
Load_more_times = 100
for _ in range(Load_more_times):
    prev = len(soup.find_all("article",attrs={"class":"border rounded color-shadow-small color-bg-subtle my-4"}))
    try:
        browser.find_element_by_xpath("//*[@id=\"js-pjax-container\"]/div[2]/div[2]/div/div[1]/form/button").click()
    except:
        print("End")
        break
    while 1:
        soup = BeautifulSoup(browser.page_source,'lxml')
        if prev < len(soup.find_all("article",attrs={"class":"border rounded color-shadow-small color-bg-subtle my-4"})):
            prev = len(soup.find_all("article",attrs={"class":"border rounded color-shadow-small color-bg-subtle my-4"}))
            break
    print(prev,"개 load 완료.")
    
# Topic들을 Crawling후 나열 
soup = BeautifulSoup(browser.page_source,'lxml')
topics = soup.find_all("h3",attrs={"class":"f3 color-fg-muted text-normal lh-condensed"})
topic_ads = []
for i in topics:
    topic_ad = "".join(i.get_text().strip().replace("\n","").split()) 
    topic_ads.append(topic_ad)
```
<img width="303" alt="스크린샷 2022-05-08 오후 11 06 24" src="https://user-images.githubusercontent.com/87521259/167300051-915c5e4c-811d-434e-89a1-a3aa5e4eefb8.png">

   - DBSCAN Clustering

```
print("{} {} DBSCAN 클러스터링 시작 {}".format("*"*10,topic_name,"*"*10))
excel_name = "keyword({})_vectors.xlsx".format(topic_name)
df_vector = pd.read_excel(excel_name)
# eps 값을 조정해나가면서 클러스터링을 해야 더 정확한 결과가 나온다
dbscan = DBSCAN(eps = 0.3)
dbscan_cluster = dbscan.fit_predict(df_vector)
dbscan_cluster
dbscan_clustered_dic = {}
dbscan_clustered_list = []
dbscan_cluster_num = len(set(dbscan_cluster))


for idx,i in enumerate(dbscan_cluster):
    if i not in dbscan_clustered_dic:
        dbscan_clustered_dic[i] = [df_topic['project_name'][idx]]
    else:
        dbscan_clustered_dic[i].append(df_topic['project_name'][idx])

# 클러스터링이 된 패키지들
# 20개의 군집으로 생성

dbscan_clustered_dic = sorted(dbscan_clustered_dic.items(), key=lambda x: x[0])

df_dbscan_cluster = pd.DataFrame(dbscan_clustered_dic,columns=['num','clusters'])
dbscan_cluster_num = [len(i) for i in df_dbscan_cluster['clusters']]
df_dbscan_cluster['cluster_num'] = dbscan_cluster_num
topic_dbscan_clustered_list = []
for i in df_dbscan_cluster['clusters']:
    temp_dic = {}
    for j in i:
        topics = df_topic[df_topic['project_name']==j]['topic_keyword'].values[0]
        for i in topics:
            if len(i)==0:
                continue
            i = same_things(i)
            if i not in temp_dic:
                temp_dic[i] = 1
            else:
                temp_dic[i] += 1
    temp_dic = sorted(temp_dic.items(), key=lambda x: x[1], reverse=True)
#     print(temp_dic[:15]) # 상위 15개만 보여줌
#     print()
    topic_dbscan_clustered_list.append(temp_dic[:15])
df_dbscan_cluster['top_15_topics'] = topic_dbscan_clustered_list
df_dbscan_cluster.to_excel("{}_DBSCAN_clusters.xlsx".format(topic_name),index=False)
print(df_dbscan_cluster)
print("{}_DBSCAN_clusters.xlsx".format(topic_name),"저장완료")
print("*"*50)

```
<img width="656" alt="스크린샷 2022-05-08 오후 11 07 04" src="https://user-images.githubusercontent.com/87521259/167300066-9ebf66ca-9e55-4c78-9faf-c74878e81999.png">

