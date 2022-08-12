Da borgerne i Hong Kong i 2019 og 2020 gik på gaden i tusindvis i protest mod et nyt lovforslag om udlevering af kriminelle, opstod efterfølgende nogle af de mest langvarige og største demostrationer, der er set i verdenshistorien.

Mediernes repotager om massedemonstrationer, politibrutalitet, gadekampe og demonstrater, der kastede sig ud i kampen for frihedsværdier gjorde stort indtryk på alle.

Det undrer mig, hvilke spor protesterne har sat sig på Wikipidia? Og hvilke forskelle og ligheder der eksisterer på de wikipidiasider, som omhandler protesterne?

Hvilken metode vil jeg kunne anvende til at indsamle data, der ville kunne anvendes til en undersøgelse?

Eftersom jeg kender Python biblioteket BeautifulSoup, som man kan bruge til webscrape, så forestiller jeg mig, at jeg ville kunne bruge det til at indsamle tekstdata, som ville kunne bruges til at undersøge min hypotese.     

Før dataindsamlingen vælger jeg en hjemmeside på Wikipidia, der handler om demonstrationerne i Hong Kong i 2019-2020.

Jeg har fundet denne side: https://en.wikipedia.org/wiki/2019%E2%80%932020_Hong_Kong_protests

For at hente sidens data skal jeg bruge nogle biblioteker, som jeg henter nedenfor.


```python
# til manipulering af tekststrenge
import re

# Webscrape biblioteker
from random import randint
from time import sleep
from bs4 import BeautifulSoup
import requests

# til strukturering af datasættet
import pandas as pd
```

Jeg henter dataen ned på computeren og gemmer indholdet i variablen 'soup'.


```python
# gem url'en i en variabel
url = "https://en.wikipedia.org/wiki/2019%E2%80%932020_Hong_Kong_protests"

# hent data
page = requests.get(url)

# scrape websiden
soup = BeautifulSoup(page.content, 'html.parser')
 
# vis scrapped data
#print(soup.prettify())
```

Nu åbner jeg siden i min webbrowser (jeg bruger chrome) og vælger mit 'udviklerværktøj'.

<img src='udviklerværktøj3.png' width=100%>

Ved hjælp af udviklerværktøjet har jeg mulighed for at gennemse sidens html og identificere de dele, der indeholder de data, jeg vil bruge til min undersøgelse.

De data jeg skal bruge, består i links til sider, der omhandler samme emner, men er skrevet på andre sprog. Disse links ligger som 'li-tags' og har en 'class', der begynder med 'interlanguage-link'.

<img src='udviklerværktøj4.png' width=100%>

Jeg bruger Beautiful Soup til at hente alle 'li-tags', der har en 'class', hvis tekststreng indeholder 'interlanguage-link'. 


```python
li_tags = soup.find_all('li', class_= 'interlanguage-link')
```

Ovenfor fik jeg returneret en liste, og nedenfor jeg inspicerer det første og det sidste element i listen.


```python
print (li_tags[0])
print ('\n')
print (li_tags[-1])
```

    <li class="interlanguage-link interwiki-ar mw-list-item"><a class="interlanguage-link-target" href="https://ar.wikipedia.org/wiki/%D8%A7%D8%AD%D8%AA%D8%AC%D8%A7%D8%AC%D8%A7%D8%AA_%D9%87%D9%88%D9%86%D8%BA_%D9%83%D9%88%D9%86%D8%BA_2019-20" hreflang="ar" lang="ar" title="احتجاجات هونغ كونغ 2019-20 – Arabic"><span>العربية</span></a></li>
    
    
    <li class="interlanguage-link interwiki-zh mw-list-item"><a class="interlanguage-link-target" href="https://zh.wikipedia.org/wiki/%E5%8F%8D%E5%B0%8D%E9%80%83%E7%8A%AF%E6%A2%9D%E4%BE%8B%E4%BF%AE%E8%A8%82%E8%8D%89%E6%A1%88%E9%81%8B%E5%8B%95" hreflang="zh" lang="zh" title="反對逃犯條例修訂草案運動 – Chinese"><span>中文</span></a></li>
    

Se godt på dataen ovenfor. 

De værdier jeg skal bruge ligger i 'a-tags', som jeg kan finde med .find(), og trække ud med .get().

Jeg vil gerne bruge links ('href'), sprogkoden ('lang') og titler ('title'). 

Ud fra titlerne kan jeg lave en liste over forskellige sprog. Det kan jeg, fordi alle titlerne er kendetegnet ved en ens syntasks (f.eks. title="六四事件 – Chinese"). Jeg kan splitte tekststrengen på ' - ' og tage det sidste element i den liste, som .split() returnerer ( [-1] ).


```python
# Hent links, sprogkoder, titler og sprog
links = [i.find('a').get('href') for i in li_tags]
lang_codes = [i.find('a').get('lang') for i in li_tags]
titles = [i.find('a').get('title') for i in li_tags]
lang = [i.split(' – ')[-1] for i in titles]

# tilføj data om den engelske side
links.append('https://en.wikipedia.org/wiki/2019%E2%80%932020_Hong_Kong_protests')
lang_codes.append('en')
titles.append('2019–2020 Hong Kong protests')
lang.append('English')
```

For at hente sidernes html bruger jeg de url'er, som jeg har samlet i listen links.

Jeg skriver en funtion, der består af et 'loop', hvor url'en bliver åbnet, og html'en bliver gemt som et beautifulsoup element i variablen 'soup' og bliver returneret fra funktionen. Det tager lidt tid, så stå op og stræk ryggen ud.


```python
def get_html(url):
    
    # vent mellem et og fem sekunder
    sleep(randint(1,5))
    
    # hent data
    page = requests.get(url)

    # scrape websiden
    soup = BeautifulSoup(page.content, 'html.parser')
    
    return soup
```

Jeg bruger nu funktionen til at hente sidernes kode og gemme dem i variablen 'html'.


```python
html = [get_html(i) for i in links]
```

For at hente tekst fra siderne bruger de beautifulsoup elementer, som jeg har samlet i listen 'html'.

Jeg skriver en funtion, der består af et 'loop', hvor elementerne bliver åbnet og h3-tags (overskrifter) og teksten i p_tags bliver trukket ud.


```python
def get_text(bs_element):
    
    # hent p_tags
    p_tags_list = bs_element.find_all(['h3', 'p'])

    # hent teksten i p_tags
    text_list = [i.get_text().strip() for i in p_tags_list]

    # join listen til en tekststreng
    text_string = ' '.join(text_list)
    
    # rens teksten for henvisninger til reference. Henvisningerne ligger inden for to firkantede paranteser, f.eks. '[17]'
    text_string1 = re.sub('(\[.+?])', '', text_string)

    return text_string1
```

Jeg bruger nu funktionen til at hente teksterne og gemme dem i variablen 'text_strings'


```python
text_strings = [get_text(i) for i in html]
```


```python
len(text_strings)
```




    54




```python
print (text_strings[0][0:1000])
```

     اندلعت احتجاجات هونغ كونغ في عامي 2019 – 2020، والمعروفة أيضًا باسم الحركة المناهضة لمشروع تعديل قانون تسليم المطلوبين، إثر إصدار حكومة هونغ كونغ لمشروع قانون تسليم المجرمين الفارين. كان القانون ليسمح بتسليم المجرمين إلى السلطات القضائية التي لم توقع اتفاقيات تسليم مع هونغ كونغ، بما فيها بر الصين الرئيسي وتايوان. قاد هذا إلى مخاوف من أن يصير المقيمون في هونغ كونغ وزوارها عرضة للنظام القانوني خاصة بر الصين الرئيسي، وبالتالي ضعضعة حكم هونغ كونغ الذاتي وانتهاك الحريات المدنية. أطلق الأمر سلسلة من الأفعال الاحتجاجية التي بدأت باعتصام في مقرات الحكومة في 15 مارس 2019، وتظاهرة خرج فيها مئات الآلاف في 9 يونيو 2019، أعقبها احتشاد خارج مجمع المجلس التشريعي بغية تعطيل قراءة المشروع الثانية في 12 يونيو تصعّدَ إلى عنف جذب انتباه العالم. في 16 يونيو، بعد يوم واحد فقط من تعليق حكومة هونغ كونغ القانون، حدث احتجاج أكبر من سابقيه للضغط ناحية الإلغاء الكامل للمشروع وردًا على الاستخدام المفرط المحسوس للقوة من قبل الشرطة في 12 يونيو. بعد أن حققت الاحتجاجات تقدمًا، طرح الناشطون خمسة مطالب رئيسية، هي بالتح
    

Nu har jeg samlet en del data i fem forskellige lister. For at få disse data samlet i en variabel samler jeg dem i en pandas dataframe.


```python
df = pd.DataFrame({'Language': lang,
                  'Title': titles,
                  'Language_code': lang_codes,
                  'Link':links,
                  'Text': text_strings})
```

Jeg har indsamlet dataen og struktureret tekster og metadata i en dataframe.


```python
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Language</th>
      <th>Title</th>
      <th>Language_code</th>
      <th>Link</th>
      <th>Text</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Arabic</td>
      <td>احتجاجات هونغ كونغ 2019-20 – Arabic</td>
      <td>ar</td>
      <td>https://ar.wikipedia.org/wiki/%D8%A7%D8%AD%D8%...</td>
      <td>اندلعت احتجاجات هونغ كونغ في عامي 2019 – 2020...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Azerbaijani</td>
      <td>Honkonqda ekstradisiya qanun layihəsi əleyhinə...</td>
      <td>az</td>
      <td>https://az.wikipedia.org/wiki/Honkonqda_ekstra...</td>
      <td>2019-cu ildə Honkonqda ekstradisiya qanun layi...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Balinese</td>
      <td>Démonstrasi lawan ékstradisi ring Hong Kong du...</td>
      <td>ban</td>
      <td>https://ban.wikipedia.org/wiki/D%C3%A9monstras...</td>
      <td>Démonstrasi lawan ékstradisi di Hong Kong duk ...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Chinese (Min Nan)</td>
      <td>Hoán-tùi Tô-hoān Tiâu-lē Siu-tēng Chhó-àn Ūn-t...</td>
      <td>nan</td>
      <td>https://zh-min-nan.wikipedia.org/wiki/Ho%C3%A1...</td>
      <td>Hoán-tùi Tô-hoān Tiâu-lē Siu-tēng Chhó-àn Ūn-t...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Belarusian</td>
      <td>Пратэсты ў Ганконгу (2019—2020) – Belarusian</td>
      <td>be</td>
      <td>https://be.wikipedia.org/wiki/%D0%9F%D1%80%D0%...</td>
      <td>(няма цэнтралізаванага кіраўніцтва): «Демасiст...</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Catalan</td>
      <td>Protestes a Hong Kong de 2019 – Catalan</td>
      <td>ca</td>
      <td>https://ca.wikipedia.org/wiki/Protestes_a_Hong...</td>
      <td>Pàgines per a editors no registrats més inform...</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Czech</td>
      <td>Demonstrace v Hongkongu 2019–2020 – Czech</td>
      <td>cs</td>
      <td>https://cs.wikipedia.org/wiki/Demonstrace_v_Ho...</td>
      <td>Pokračující demonstrace v Hongkongu 2019–2020,...</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Danish</td>
      <td>Demonstrationerne i Hongkong 2019-2020 – Danish</td>
      <td>da</td>
      <td>https://da.wikipedia.org/wiki/Demonstrationern...</td>
      <td>Demonstrationerne i Hongkong 2019-2020 er en r...</td>
    </tr>
    <tr>
      <th>8</th>
      <td>German</td>
      <td>Proteste in Hongkong 2019/2020 – German</td>
      <td>de</td>
      <td>https://de.wikipedia.org/wiki/Proteste_in_Hong...</td>
      <td>Im Sommer 2019 brachen in der chinesischen Son...</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Estonian</td>
      <td>2019.–2020. aasta Hongkongi meeleavaldused – E...</td>
      <td>et</td>
      <td>https://et.wikipedia.org/wiki/2019.%E2%80%9320...</td>
      <td>Lokalistid Iseseisvuslased Pekingimeelne laage...</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Greek</td>
      <td>Διαδηλώσεις στο Χονγκ Κονγκ 2019-2020 – Greek</td>
      <td>el</td>
      <td>https://el.wikipedia.org/wiki/%CE%94%CE%B9%CE%...</td>
      <td>Οι διαδηλώσεις στο Χονγκ Κονγκ του 2019-2020 ή...</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Spanish</td>
      <td>Protestas en Hong Kong de 2019-2021 – Spanish</td>
      <td>es</td>
      <td>https://es.wikipedia.org/wiki/Protestas_en_Hon...</td>
      <td>Manifestantes (sin autoridad centralizada) Gru...</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Basque</td>
      <td>2019-2020ko Hong Kongeko protestak – Basque</td>
      <td>eu</td>
      <td>https://eu.wikipedia.org/wiki/2019-2020ko_Hong...</td>
      <td>Izena eman gabeko erabiltzaileentzako orrialde...</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Persian</td>
      <td>اعتراضات ۲۰۱۹–۲۰۲۰ هنگ کنگ – Persian</td>
      <td>fa</td>
      <td>https://fa.wikipedia.org/wiki/%D8%A7%D8%B9%D8%...</td>
      <td>صفحه‌هایی برای ویرایشگرانی که از سامانه خارج ش...</td>
    </tr>
    <tr>
      <th>14</th>
      <td>French</td>
      <td>Manifestations de 2019-2020 à Hong Kong – French</td>
      <td>fr</td>
      <td>https://fr.wikipedia.org/wiki/Manifestations_d...</td>
      <td>Pages pour les contributeurs déconnectés en sa...</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Irish</td>
      <td>2019 agóidí Hong Cong – Irish</td>
      <td>ga</td>
      <td>https://ga.wikipedia.org/wiki/2019_ag%C3%B3id%...</td>
      <td>Is sraith léirsithe ar bun anois i Hong Cong, ...</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Gan Chinese</td>
      <td>反修例運動 – Gan Chinese</td>
      <td>gan</td>
      <td>https://gan.wikipedia.org/wiki/%E5%8F%8D%E4%BF...</td>
      <td>反修例運動係香港嗰社會運動，自2019年3月15號開始，哈冇完嗰。 個人工具 空間名 望下 ...</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Hakka Chinese</td>
      <td>Fán Sung Chûng Yun-thung – Hakka Chinese</td>
      <td>hak</td>
      <td>https://hak.wikipedia.org/wiki/F%C3%A1n_Sung_C...</td>
      <td>Fán Sung Chûng Yun-thung (反送中運動) he chhṳ 2019-...</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Korean</td>
      <td>2019-2020년 홍콩 시위 – Korean</td>
      <td>ko</td>
      <td>https://ko.wikipedia.org/wiki/2019-2020%EB%85%...</td>
      <td>로그아웃한 편집자를 위한 문서 더 알아보기 둘러보기 사용자 모임 편집 안내 도구 인...</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Armenian</td>
      <td>Բողոքի ակցիաներ Հոնկոնգում – Armenian</td>
      <td>hy</td>
      <td>https://hy.wikipedia.org/wiki/%D4%B2%D5%B8%D5%...</td>
      <td>Բողոքի ակցիաներ Հոնկոնգում՝ արտահանձնման օրինա...</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Indonesian</td>
      <td>Unjuk rasa Hong Kong 2019–2020 – Indonesian</td>
      <td>id</td>
      <td>https://id.wikipedia.org/wiki/Unjuk_rasa_Hong_...</td>
      <td>Halaman penyunting yang telah keluar log pelaj...</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Icelandic</td>
      <td>Mótmælin í Hong Kong 2019–20 – Icelandic</td>
      <td>is</td>
      <td>https://is.wikipedia.org/wiki/M%C3%B3tm%C3%A6l...</td>
      <td>Mótmælin í Hong Kong voru fjöldamótmæli í kínv...</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Italian</td>
      <td>Proteste a Hong Kong del 2019-2020 – Italian</td>
      <td>it</td>
      <td>https://it.wikipedia.org/wiki/Proteste_a_Hong_...</td>
      <td>Supporto da: Cina Le proteste a Hong Kong del ...</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Hebrew</td>
      <td>המחאות בהונג קונג (2019–2020) – Hebrew</td>
      <td>he</td>
      <td>https://he.wikipedia.org/wiki/%D7%94%D7%9E%D7%...</td>
      <td>דפים לעורכים שלא נכנסו לחשבון מידע נוסף ניווט ...</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Georgian</td>
      <td>საპროტესტო გამოსვლები ჰონგ-კონგში (2019-2020) ...</td>
      <td>ka</td>
      <td>https://ka.wikipedia.org/wiki/%E1%83%A1%E1%83%...</td>
      <td>საპროტესტო გამოსვლები ჰონგ-კონგში (2019-2020) ...</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Hungarian</td>
      <td>2019-es hongkongi tüntetések – Hungarian</td>
      <td>hu</td>
      <td>https://hu.wikipedia.org/wiki/2019-es_hongkong...</td>
      <td>Pontosságellenőrzött A 2019-es hongkongi tünte...</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Marathi</td>
      <td>२०१९–२० हाँग काँग निषेध – Marathi</td>
      <td>mr</td>
      <td>https://mr.wikipedia.org/wiki/%E0%A5%A8%E0%A5%...</td>
      <td>नवीन सदस्यांना मार्गदर्शनहा साचा अशुद्धलेखन, ...</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Malay</td>
      <td>Tunjuk perasaan Hong Kong 2019–2020 – Malay</td>
      <td>ms</td>
      <td>https://ms.wikipedia.org/wiki/Tunjuk_perasaan_...</td>
      <td>Bantahan menentang rang undang-undang serah ba...</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Minangkabau</td>
      <td>Unjuak raso Hong Kong 2019 – Minangkabau</td>
      <td>min</td>
      <td>https://min.wikipedia.org/wiki/Unjuak_raso_Hon...</td>
      <td>Unjuak raso Hong Kong 2019, nan disabuik juo s...</td>
    </tr>
    <tr>
      <th>29</th>
      <td>Min Dong Chinese</td>
      <td>Huāng sáe̤ng Dṳ̆ng Ông-dông – Min Dong Chinese</td>
      <td>cdo</td>
      <td>https://cdo.wikipedia.org/wiki/Hu%C4%81ng_s%C3...</td>
      <td>Chăng-kō̤ Mìng-dĕ̤ng-ngṳ̄ Háng-cê gì bēng-buōn...</td>
    </tr>
    <tr>
      <th>30</th>
      <td>Dutch</td>
      <td>Protesten in Hongkong in 2019-2020 – Dutch</td>
      <td>nl</td>
      <td>https://nl.wikipedia.org/wiki/Protesten_in_Hon...</td>
      <td>De protesten in Hongkong in 2019-2020 zijn een...</td>
    </tr>
    <tr>
      <th>31</th>
      <td>Japanese</td>
      <td>2019年-2020年香港民主化デモ – Japanese</td>
      <td>ja</td>
      <td>https://ja.wikipedia.org/wiki/2019%E5%B9%B4-20...</td>
      <td>ログアウトした編集者のページ もっと詳しく 案内 ヘルプ ツール 印刷/書き出し 他のプロジ...</td>
    </tr>
    <tr>
      <th>32</th>
      <td>Occitan</td>
      <td>Manifestacions de 2019 a Hong Kong – Occitan</td>
      <td>oc</td>
      <td>https://oc.wikipedia.org/wiki/Manifestacions_d...</td>
      <td>De manifestacions contre l'emendament de la le...</td>
    </tr>
    <tr>
      <th>33</th>
      <td>Western Punjabi</td>
      <td>2019–20 ہانگ کانگ مظاہرے – Western Punjabi</td>
      <td>pnb</td>
      <td>https://pnb.wikipedia.org/wiki/2019%E2%80%9320...</td>
      <td>Thai protesters (solidarity support and online...</td>
    </tr>
    <tr>
      <th>34</th>
      <td>Polish</td>
      <td>Protesty w Hongkongu (2019–2020) – Polish</td>
      <td>pl</td>
      <td>https://pl.wikipedia.org/wiki/Protesty_w_Hongk...</td>
      <td>Chiny Hongkong 15 marca 2019 30 czerwca 2020 u...</td>
    </tr>
    <tr>
      <th>35</th>
      <td>Portuguese</td>
      <td>Protestos em Hong Kong em 2019–2020 – Portuguese</td>
      <td>pt</td>
      <td>https://pt.wikipedia.org/wiki/Protestos_em_Hon...</td>
      <td>Páginas para editores conectados saiba mais Na...</td>
    </tr>
    <tr>
      <th>36</th>
      <td>Romanian</td>
      <td>Protestele din Hong Kong din 2019 – Romanian</td>
      <td>ro</td>
      <td>https://ro.wikipedia.org/wiki/Protestele_din_H...</td>
      <td>Protestele din Hong Kong din 2019, cunoscute ș...</td>
    </tr>
    <tr>
      <th>37</th>
      <td>Russian</td>
      <td>Протесты в Гонконге (2019—2020) – Russian</td>
      <td>ru</td>
      <td>https://ru.wikipedia.org/wiki/%D0%9F%D1%80%D0%...</td>
      <td>(нет централизованного руководства) Поддержива...</td>
    </tr>
    <tr>
      <th>38</th>
      <td>Scots</td>
      <td>2019–20 Hong Kong protests – Scots</td>
      <td>sco</td>
      <td>https://sco.wikipedia.org/wiki/2019%E2%80%9320...</td>
      <td>Supportit bi: The ongaein 2019–20 Hong Kong pr...</td>
    </tr>
    <tr>
      <th>39</th>
      <td>Albanian</td>
      <td>Protestat në Hong Kong 2019 – Albanian</td>
      <td>sq</td>
      <td>https://sq.wikipedia.org/wiki/Protestat_n%C3%A...</td>
      <td>Protestat e Hong Kongut të vitit 2019 janë një...</td>
    </tr>
    <tr>
      <th>40</th>
      <td>Simple English</td>
      <td>2019–20 Hong Kong protests – Simple English</td>
      <td>en-simple</td>
      <td>https://simple.wikipedia.org/wiki/2019%E2%80%9...</td>
      <td>The 2019-20 Hong Kong anti-extradition bill pr...</td>
    </tr>
    <tr>
      <th>41</th>
      <td>Serbian</td>
      <td>Протести у Хонгконгу 2019/20. – Serbian</td>
      <td>sr</td>
      <td>https://sr.wikipedia.org/wiki/%D0%9F%D1%80%D0%...</td>
      <td>Странице за одјављене уреднике детаљније Навиг...</td>
    </tr>
    <tr>
      <th>42</th>
      <td>Finnish</td>
      <td>Hongkongin vuosien 2019–2020 mielenosoitukset ...</td>
      <td>fi</td>
      <td>https://fi.wikipedia.org/wiki/Hongkongin_vuosi...</td>
      <td>Hongkongissa on järjestetty vuosina 2019–2020 ...</td>
    </tr>
    <tr>
      <th>43</th>
      <td>Swedish</td>
      <td>Demonstrationerna i Hongkong 2019 – Swedish</td>
      <td>sv</td>
      <td>https://sv.wikipedia.org/wiki/Demonstrationern...</td>
      <td>Demonstrationerna i Hongkong 2019 är en rad åt...</td>
    </tr>
    <tr>
      <th>44</th>
      <td>Tamil</td>
      <td>ஹாங்காங் போராட்டம் 2019 – Tamil</td>
      <td>ta</td>
      <td>https://ta.wikipedia.org/wiki/%E0%AE%B9%E0%AE%...</td>
      <td>ஹாங்காங் போராட்டம் 2019 (2019 Hong Kong protes...</td>
    </tr>
    <tr>
      <th>45</th>
      <td>Thai</td>
      <td>การประท้วงในฮ่องกง พ.ศ. 2562–2563 – Thai</td>
      <td>th</td>
      <td>https://th.wikipedia.org/wiki/%E0%B8%81%E0%B8%...</td>
      <td>หน้าสำหรับผู้แก้ไขที่ออกจากระบบ เรียนรู้เพิ่มเ...</td>
    </tr>
    <tr>
      <th>46</th>
      <td>Turkish</td>
      <td>2019-2020 Hong Kong protestoları – Turkish</td>
      <td>tr</td>
      <td>https://tr.wikipedia.org/wiki/2019-2020_Hong_K...</td>
      <td>Çıkış yapmış editörler için sayfalar daha fazl...</td>
    </tr>
    <tr>
      <th>47</th>
      <td>Ukrainian</td>
      <td>Протести в Гонконзі – Ukrainian</td>
      <td>uk</td>
      <td>https://uk.wikipedia.org/wiki/%D0%9F%D1%80%D0%...</td>
      <td>(без лідера) Підтримані: Тріади (без лідерів) ...</td>
    </tr>
    <tr>
      <th>48</th>
      <td>Vietnamese</td>
      <td>Biểu tình tại Hồng Kông 2019–20 – Vietnamese</td>
      <td>vi</td>
      <td>https://vi.wikipedia.org/wiki/Bi%E1%BB%83u_t%C...</td>
      <td>Trang dành cho người dùng chưa đăng nhập tìm h...</td>
    </tr>
    <tr>
      <th>49</th>
      <td>Classical Chinese</td>
      <td>己亥香港民潮 – Classical Chinese</td>
      <td>lzh</td>
      <td>https://zh-classical.wikipedia.org/wiki/%E5%B7...</td>
      <td>質素尚可 己亥香港民潮，別稱反修例運動，或曰反送中運動、己亥事變，肇乎香港民眾反修《逃犯條例...</td>
    </tr>
    <tr>
      <th>50</th>
      <td>Wu Chinese</td>
      <td>反修例运动 – Wu Chinese</td>
      <td>wuu</td>
      <td>https://wuu.wikipedia.org/wiki/%E5%8F%8D%E4%BF...</td>
      <td>反修例运动，即反对逃犯条例修订草案运动，是囊括各种游行、集会、街道搭立法会个占领、行政部门搭...</td>
    </tr>
    <tr>
      <th>51</th>
      <td>Cantonese</td>
      <td>反逃犯條例修訂運動 – Cantonese</td>
      <td>yue</td>
      <td>https://zh-yue.wikipedia.org/wiki/%E5%8F%8D%E9...</td>
      <td>無統一組織，嚟自社會各界，後生仔比例高啲 主要支持組織 主要支持組織 中華人民共和國政府 無...</td>
    </tr>
    <tr>
      <th>52</th>
      <td>Chinese</td>
      <td>反對逃犯條例修訂草案運動 – Chinese</td>
      <td>zh</td>
      <td>https://zh.wikipedia.org/wiki/%E5%8F%8D%E5%B0%...</td>
      <td>其他訴求 海外反應 中央政府： 香港政府： 反對《逃犯條例修訂草案》運動（英語：Anti-E...</td>
    </tr>
    <tr>
      <th>53</th>
      <td>English</td>
      <td>2019–2020 Hong Kong protests</td>
      <td>en</td>
      <td>https://en.wikipedia.org/wiki/2019%E2%80%93202...</td>
      <td>The Anti-Extradition Law Amendment Bill Movem...</td>
    </tr>
  </tbody>
</table>
</div>




```python
print (df.at[52, 'Text'][0:1000])
```

    其他訴求 海外反應 中央政府： 香港政府： 反對《逃犯條例修訂草案》運動（英語：Anti-Extradition Law Amendment Bill Movement），是指香港自2019年3月15日開始、6月9日大規模爆發的社會運動，逾萬人被捕。此次運動並無統一的領導，主要以社交媒體號召的方式組織，運動支持者以遊行示威、集會、靜坐、唱歌、吶喊、「三罷」行動、設置連儂牆、不合作運動、堵塞道路幹道、「起底」、破壞商鋪、建築物、大學、及公共設施等一系列行為，向香港特別行政區政府抗議其提出《逃犯條例》修訂草案。根据示威者的观点，該草案容許將香港的犯罪嫌疑人引渡至中國內地受審；而反對者因不信任中國大陸的司法制度而擔憂將嫌疑人引渡至大陸會出現不公平審訊的情況，进而損害香港在「一國兩制」及《基本法》下所列明的獨立司法管轄權地位。 香港眾志早在2019年3月15日就已於政府總部發起要求撤回《逃犯條例》修訂的靜坐。在3月至4月期間，民間人權陣線兩度發起示威遊行。6月9日，民陣再度發起遊行，主辦方宣稱有超过100万名市民參與。但特区政府不顾强烈反对，继续强行推动修例；6月12日，香港立法會继续將條例恢復二讀辯論，触发了四万名市民围在了立法会大楼外示威。为了阻止条例在立法会通过，示威者與警方發生了激烈衝突。事後主辦方指责警方濫用職權及過度使用武力。此後示威者提出「完全撤回《逃犯條例》修訂草案、撤回暴動定性（612）、撤回所有示威者控罪、追究警隊濫權、行政長官林鄭月娥辭職下台」等「五大訴求」。6月16日，民間人權陣線發起更大規模的遊行，再次引发了大量市民参与这次游行。主辦方宣稱游行人数超过了200万人，是「香港有史以来参与人数最多的游行」。7月1日遊行期間，部分示威者佔領立法會綜合大樓，其後並將林鄭月娥下台的訴求更改為「立即實現行政長官和立法會的真雙普選」，实现真正的民主普选诉求。 之後，示威者幾乎每週發起常態抗議活動。在7月21日，遊行後發生了元朗襲擊事件，这次事件成为运动的标志性事件，也使运动发生转折，示威者与警方之间的衝突加劇。8月中旬，示威者兩度癱瘓香港國際機場。8月18日，民陣再度舉辦大規模和平集會，参与8·18集会的人数超过170万人。8月31日，太子站事件後令示威行動升級。9月4日下午，行政長官林鄭月娥宣佈四項行動，並動議撤回《逃犯條例》修訂草案，但拒绝成立独立调查委员调
    

Notebooken slutter her, hvor tekstdata er indsamlet, og jeg overvejer de fremtidige perspektiver for undersøgelsen.

Eftersom jeg har indsamlet data fra 54 forskellige sprog, som jeg ikke forstår, skal der gøres noget. Jeg ville have svært ved at analysere teksterne computationelt på deres orginalsprog. Derfor overvejer jeg at oversætte et udvalg af dem til engelsk vha. google translate. På den måde ville jeg få en mulighed for at få indblik i sidernes indhold.
