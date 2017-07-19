JavaでDeepLearning  
（コーディング編）

---
<table border="1">
  <tr>
    <td>
      <img src="https://photos-3.dropbox.com/t/2/AACs8LWHQmxEUqb5qkLV3SKUlx4ZBxkffqmnDgY2s3D9uA/12/172232240/jpeg/32x32/3/1500454800/0/2/2017-07-15%2015.51.17.jpg/EIifm4MBGOD-ZyACKAI/Qxtr24nIuf2wFq9xIqifA9RSWxiMXQBS3BwLrsp0ksc?dl=0&size=2048x1536&size_mode=3" height="200">
    </td>
    <td>
      Yusuke Shinohara<br>
      twitter : @allgiz<br>
      (株)ベガコーポレーション<br>
      DS/SE
    </td>

  </tr>
</table>

---
### 前回まで
JavaでもDeepLearningはできる。  
必要なライブラリ(jar)はmavenで用意できる。  
※ Gradle, SBT, Leiningenでも可能   




-> pom.xmlで、CPU/GPUを簡単に切替可能。  

---
### 必要な処理手順
1. データの用意
2. 学習
3. 評価
4. 適用

---
### データの用意
Irisの例（Rより抜粋）

|Sepal.Length|Sepal.Width|Petal.Length|Petal.Width|Species|
|--:|--:|--:|--:|--|
|5.1|3.5|1.4|0.2|setosa|
|7.0|3.2|4.7|1.4|versicolor|
|6.3|3.3|6.0|2.5|virginica|
|5.8|2.7|5.1|1.9|virginica|

-> こんな形のデータを用意する必要がある。

---
### データの用意
メモリ上で処理してもよいが、   
CSVを一度出力する。  




Stream API (Java8)が便利。   
(DL以外でも機械学習用のデータ編集にオススメ)

+++
##### Stream APIメソッド
|Method|用途|
|--|--|
|distinct()|重複削除|
|filter(Predicate<? super T> predicate)|検索・結合|
|min/max(Comparator<? super T> comparator)|最小値/最大値|
|reduce(BinaryOperator<T> accumulator)|合計や累積など|

+++
CSVを用意したら↓の要領で読込
```
int numLinesToSkip = 1;// ヘッダなどでスキップする行数
String delimiter = "\t";// CSVの区切り文字

int labelIndex = 60;// 入力層の数（説明変数のカラム数）
int numClasses = 7;// 出力層の数（出力のクラス数）
int batchSize = 150;

try (RecordReader recordReader = new CSVRecordReader(numLinesToSkip, delimiter);) {

  recordReader.initialize(new FileSplit(teacher));

  DataSetIterator iterator = new RecordReaderDataSetIterator(recordReader, batchSize, labelIndex, numClasses);

```

---
### 学習 (設定)
<img src="https://github.com/complex-alpha/DLwJavaLT4/blob/master/genericDNN.png?raw=true" width="400">

+++
```
MultiLayerConfiguration conf = new NeuralNetConfiguration.Builder()
    .seed(seed)
    .iterations(iterations)
    .activation(Activation.RELU)
    .weightInit(WeightInit.XAVIER)
    .learningRate(0.1)
    .regularization(true)
    .l2(1e-4)
    .list()
    .layer(layerCnt++, new DenseLayer.Builder().activation(Activation.RELU).nIn(numInputs).nOut(600).build())
    .layer(layerCnt++, new DenseLayer.Builder().activation(Activation.RELU).nIn(600).nOut(600).build())
    .layer(layerCnt++, new DenseLayer.Builder().activation(Activation.RELU).nIn(600).nOut(7).build())
    .layer(layerCnt++, new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD).activation(Activation.SOFTMAX).nIn(7).nOut(outputNum).build())
    .backprop(true)
    .pretrain(false)
    .build();
```
+++
```
DataSet allData = iterator.next();
allData.shuffle();
SplitTestAndTrain testAndTrain = allData.splitTestAndTrain(0.65); // トレーニングに使用するデータの割合

DataSet trainingData = testAndTrain.getTrain();// トレーニング用データ
DataSet testData = testAndTrain.getTest();// テスト用データ

// データを正規化する。NormalizeStandardizeクラスを使用（平均0、単位分散を与える）
DataNormalization normalizer = new NormalizerStandardize();
normalizer.fit(trainingData); // トレーニングデータから統計情報（平均/標準偏差）を収集
normalizer.transform(trainingData); // トレーニングデータを正規化
normalizer.transform(testData); // テストデータを正規化 (trainingから計算された統計を使用)
```

---
### 学習
```
// run the model
final int numInputs = labelIndex;
int outputNum = numClasses;
int iterations = 10000;
long seed = 123;
MultiLayerNetwork model = new MultiLayerNetwork(conf);
model.init();
model.setListeners(new ScoreIterationListener(100));

model.fit(trainingData);
```
---
### 評価
```
// evaluate the model on the test set
Evaluation eval = new Evaluation(3);
INDArray output = model.output(testData.getFeatureMatrix());
eval.eval(testData.getLabels(), output);
System.out.println(eval.stats());
```
+++
```
==========================Scores========================================
 Accuracy:        0.3333
 Precision:       0.8333
 Recall:          0.5778
 F1 Score:        0.6824
========================================================================
```
|用語|意味|
|--|--|
|Accuracy|正解率|
|Precision|適合率|
|Recall|再現率|
|F1 Score|F値（適合率と再現率の調和平均）|

+++
#### 学習したモデルを保存
```
ModelSerializer.writeModel(model, modelFile, true);
```
---
### 適用
```
INDArray input = testData.get(i).getFeatureMatrix();// 入力
INDArray answer = testData.get(i).getLabels();// 答え
INDArray predict = model.output(input, false);// 予測
```

---
### まとめ
結構手間がかかる。  
データ編集にはJava8で導入されたStream APIがオススメ。  
