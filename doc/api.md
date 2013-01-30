3du.tw API
==========

Status: draft

初步想法
--------
* 一律用 http GET，不用 POST，方便瀏覽器和 proxy 做 cache。
* 為每個資料來源提供一個 URL 入口，另外再用一個 URL 做跨資料源的搜尋。如 GET /dict 是辭典、GET /idiom 是成語典、GET /search 是跨資料源搜尋。
* 參數越少越好，不用參數的預設結果應該要很有道理。一開始不支援參數，以後可考慮 format=(json|xml)、language=(zh-hant|zh-hans|zh-hant-hk) 之類的參數。


辭典 API
--------

初步 URL 設計：
* Input: GET /dict/%E6%96%87 （%E6%96%87 是「文」的 UTF-8 的 RFC 2396 的 escape 結果）
* Ouptut: 「文」的辭典 JSON 資料，HTTP header 中，status = 200，Expires 設很長（debug 時可以關掉 caching）。

JSON 內容欄位以教育部《重編國語辭典修訂本》的內容為準設計，以下是以破音字「混」的查詢結果為例，混有五個音，《重編國語辭典修訂本》裡是分列，彼此之間沒有連結，無法一覽結果，JSON 資料則已將五個音的結果合併。

無結果時：

    {
      "dict": { "word": "要重", "error": "查無此辭" }
    }

有結果時：（註："related" 欄位留給未來實作）

    {
      "dict": {
        "word": "混",
        "radical": "水",
        "strokes": 11,
        "non_radical_strokes": 8,
        "heteronyms": [
          {
            "id": "h0",
            "bopomofo1": "ㄏㄨㄣˋ",
            "bopomofo2": "huèn",
            "pinyin": "hùn",
            "definitions": {
              "adj": [
                { "id": "h0-adj-0",
                  "def": "水勢盛大。",
                  "example": "說文解字：「混，豐流也。」文選˙司馬相如˙上林賦：「汩乎混流，順阿而下。」"
                },
                { "id": "h0-adj-1",
                  "def": "汙濁不清。",
                  "example": "史記˙卷八十四˙屈原賈生傳：「舉世混濁而我獨清，眾人皆醉而我獨醒。」文選˙班固˙典引：「五德初始，同於草昧玄混之中。」李善˙注：「混猶溷濁。」"
                }
              ],
              "verb": [
                { "id": "h0-verb-0",
                  "def": "摻雜。",
                  "example": "如：「混為一談」。文選˙班固˙幽通賦：「道混成而自然兮，術同原而分流。」"
                },
                { "id": "h0-verb-1",
                  "def": "蒙騙、冒充。",
                  "example": "如：「魚目混珠」。紅樓夢˙第四十七回：「姨太太的牌也生，咱們一處坐著，別叫鳳姐兒混了我們去。」"
                },
                { "id": "h0-verb-2",
                  "def": "胡亂、苟且的度過。",
                  "example": "如：「鬼混」、「混日子」。儒林外史˙第十二回：「思量房裡沒有別人，只是楊執中的蠢兒子在那裡混。」"
                }
              ],
              "adv": [
                { "id": "h0-adv-0",
                  "def": "胡亂。",
                  "example": "如：「混說」、「混搞」。儒林外史˙第十五回：「尋了錢又混用掉了，而今落得這一個收場。」紅樓夢˙第三十四回：「薛大哥哥從來不這樣的，你們別混猜度。」"
                }
              ]
            },
            "related": [ "白混", "噴射混凝土", "矇混", ...]
          },
          {
            "id": "h1",
            "bopomofo1": "ㄍㄨㄣˇ",
            "bopomofo2": "guěn",
            "pinyin": "gǔn",
            "link": "混混",
            "related": [ "混混" ]
          },
          {
            "id": "h2",
            "bopomofo1": "ㄎㄨㄣ",
            "bopomofo2": "kuēn",
            "pinyin": "kūn",
            "link": "混夷"
          },
          {
            "id": "h3",
            "bopomofo1": "ㄏㄨㄣˊ",
            "bopomofo2": "huén",
            "pinyin": "hún",
            "link": "#h0-adj-1",
            "related": [ "混頭混腦" ]
          },
          {
            "id": "h4",
            "bopomofo1": "ㄏㄨㄣˇ",
            "bopomofo2": "huěn",
            "pinyin": "hǔn",
            "link": "#h0"
          }
        ]
      }
    }

說明：
* 每個字有一個或多個音（破音字，heteronyms），每個音有一個或多個義（definitions），每個義有其詞性：名詞（noun）、助名詞（pron）、動詞（verb）、形容詞（adj）、副詞（adv）、介詞（prep）、連接詞（conj）、助詞（aux）、歎詞（int）、狀聲詞（TODO: 英文名待訂）、詞綴（affix TODO: 英文名待確認）。JSON 中同詞性的會接在一起成為 array，並用詞性當 key 放在 definitions 中。參考： http://dict.revised.moe.edu.tw/htm/ji/tf41c.htm。
* 每個字音和每個字義都有 id，在同個字的 JSON 輸出中唯一。
* 若一個破音字沒有 definitions 但有 link，如「混ㄍㄨㄣˇ」的 link 是「混混」，代表字義在 link 到的字詞條目中。有的 link 是指向同個字的不同音，則 link 會以 "#" 開頭。
* 字義中的 related 欄位尚未定義完備，先不實作。


成語 API
--------

初步 URL 設計：
* Input: GET /idiom/%E6%BA%AB%E6%95%85%E7%9F%A5%E6%96%B0 （%E6%BA%AB%E6%95%85%E7%9F%A5%E6%96%B0 是「溫故知新」的 UTF-8 的 RFC 2396 的 escape 結果）
* Ouptut: 「溫故知新」的成語典 JSON 資料，HTTP header 中，status = 200，Expires 設很長（debug 時可以關掉 caching）。

JSON 內容欄位以教育部《成語典》的內容為準設計，以下是以「溫故知新」的查詢結果為例。

無結果時：和 /dict 同法處理

有結果時：

    {
      "idiom": {
        "word": "溫故知新",
        "bopomofo1": "ㄨㄣ　ㄍㄨˋ　ㄓ　ㄒ｜ㄣ",
        "pinyin": "wēn gù zhī xīn",
        "def": "複習學過的課業，而能領悟出新的道理。語出《論語．為政》。",
        “origin": "《論語．為政》\n子曰：「溫故而知新，可以為師矣。」",
        "origin_explanation": "溫，溫習；故，舊的。「溫故知新」就是溫習過去所學，從中獲得新的知識和體會。在《論語．為政》中記載了孔子說的一句話：「溫故而知新，可以為師矣。」意思是說：如果能常常複習以往學過的課業，而能從中領悟出新的道理，這樣就可以為人師表了。」後來「溫故知新」這句成語就從這裡演變而出，用來說明反覆學習的功效。",
        "research": "01.《漢書．卷一○．成帝紀》：「儒林之官，四海淵原，宜皆明於古今，溫故知新，通達國體，故謂之博士。」\n02.晉．潘尼〈釋奠頌〉：「抽演微言，啟發道真，探幽窮賾，溫故知新。」\n...",
        "usages": {
          "def": "複習學過的課業，而能領悟出新的道理。",
          "type": "用在「溫舊得新」的表述上。",
          "examples": [
            "做學問應該反覆熟讀，才能溫故知新，日益精進。",
            "小明成績不斷進步，靠的是溫故知新的讀書方法。", ...
          ]
        }
        "synonyms": [ "數往知來" ],
        "antonyms": [ "抱殘守缺", "食而不化" ],
        “references": [ "知新溫故" ]
      }
    }
