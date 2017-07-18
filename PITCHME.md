# JavaでDeep Learning（コーディング編）

---
## 前回まで
### JavaでもDeepLearningはできる。
必要なライブラリ(jar)はmavenで  
Gradle, SBT, Leiningenでも可  
-> pom.xmlを書き換えたら、CPU/GPUを簡単に切り替え。

---
## 必要な処理手順
1. データの用意
2. 学習
3. 評価・適用

---
## 教師あり学習

##### Irisの例（Rより抜粋）
|Sepal.Length|Sepal.Width|Petal.Length|Petal.Width|Species|
|--:|--:|--:|--:|--|
|5.1|3.5|1.4|0.2|setosa|
|4.9|3.0|1.4|0.2|setosa|
|7.0|3.2|4.7|1.4|versicolor|
|6.4|3.2|4.5|1.5|versicolor|
|6.3|3.3|6.0|2.5|virginica|
|5.8|2.7|5.1|1.9|virginica|
  
-> こんな形のデータを用意する必要がある。

---
## 畳み込み

