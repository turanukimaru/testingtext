# diff blue Tutorial

diff blue はJavaのテストをコードから自動生成するツールです。
導入することで
* エディタのフラスコのマークをクリックすると
![image1.png](image1.png)

* テストが自動生成されます
![image2.png](image2.png)

## 準備

List the prerequisites that are required or recommended.

Make sure that:
- diff blue プラグインを IntelliJ にインストールする
- Java のプロジェクトを用意する

## diff blue プラグインを IntelliJ にインストールする

1. IntelliJ の設定を開いて
![image3.png](image3.png)
2. プラグインのマーケットプレイスで diff blue を探してインストールします
![image4.png](image4.png)

## Java のプロジェクトを用意する

1. 自動テストを作成したい Java プロジェクトを用意します
このとき、Spring などの設定を読み取ってテストを作成するため、
DB の準備などは一通りしておきます。
今回はサンプルプロジェクトとして以下のリポジトリを使います。
[https://github.com/turanukimaru/testing.git]
2. IntelliJ で新規＞バージョン管理からを選択し
![image5.png](image5.png)
3. URL からプロジェクトをダウンロードします
![image6.png](image6.png)
このとき、最新環境だとコンパイルが通らないことがあります。
Gradle のコンパイラと Gradlew が利用する JAVA_HOME を Java17 にします。
![image7.png](image7.png)
4. Execute the following command in the terminal:

## Controller のテストを作成する
Controller でテストを作成すると、Service を mock にしたテストが生成されます。
![controller3.png](controller3.png)
![controller4.png](controller4.png)
   ```Java
@ContextConfiguration(classes = {DummyController.class})
@ExtendWith(SpringExtension.class)
class DummyControllerDiffblueTest {
    @Autowired
    private DummyController dummyController;

    @MockBean
    private DummyUseCase dummyUseCase;

    /**
     * Method under test: {@link DummyController#add()}
     */
    @Test
    void testAdd() throws Exception {
        when(dummyUseCase.allDummies()).thenReturn(new ArrayList<>());
        MockHttpServletRequestBuilder requestBuilder = MockMvcRequestBuilders.get("/dummies");
        MockMvcBuilders.standaloneSetup(dummyController)
                .build()
                .perform(requestBuilder)
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andExpect(MockMvcResultMatchers.content().contentType("application/json"))
                .andExpect(MockMvcResultMatchers.content().string("[]"));
    }

    /**
     * Method under test: {@link DummyController#add(DummyAddRequest)}
     */
    @Test
    void testAdd2() throws Exception {
        when(dummyUseCase.allDummies()).thenReturn(new ArrayList<>());

        DummyAddRequest dummyAddRequest = new DummyAddRequest();
        dummyAddRequest.setComment("Comment");
        dummyAddRequest.setName("Name");
        dummyAddRequest.setText("Text");
        String content = (new ObjectMapper()).writeValueAsString(dummyAddRequest);
        MockHttpServletRequestBuilder requestBuilder = MockMvcRequestBuilders.get("/dummies")
                .contentType(MediaType.APPLICATION_JSON)
                .content(content);
        MockMvcBuilders.standaloneSetup(dummyController)
                .build()
                .perform(requestBuilder)
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andExpect(MockMvcResultMatchers.content().contentType("application/json"))
                .andExpect(MockMvcResultMatchers.content().string("[]"));
    }

    /**
     * Method under test: {@link DummyController#children(Long)}
     */
    @Test
    void testChildren() throws Exception {
        when(dummyUseCase.findChildren(Mockito.<Long>any())).thenReturn(new ArrayList<>());
        MockHttpServletRequestBuilder requestBuilder = MockMvcRequestBuilders.get("/dummies/{id}/children", 1L);
        MockMvcBuilders.standaloneSetup(dummyController)
                .build()
                .perform(requestBuilder)
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andExpect(MockMvcResultMatchers.content().contentType("application/json"))
                .andExpect(MockMvcResultMatchers.content().string("[]"));
    }

    /**
     * Method under test: {@link DummyController#delete(Long)}
     */
    @Test
    void testDelete() throws Exception {
        doNothing().when(dummyUseCase).deleteDummy(Mockito.<Long>any());
        MockHttpServletRequestBuilder requestBuilder = MockMvcRequestBuilders.delete("/dummies/{id}", 1L);
        MockMvcBuilders.standaloneSetup(dummyController)
                .build()
                .perform(requestBuilder)
                .andExpect(MockMvcResultMatchers.status().isOk());
    }

    /**
     * Method under test: {@link DummyController#delete(Long)}
     */
    @Test
    void testDelete2() throws Exception {
        doNothing().when(dummyUseCase).deleteDummy(Mockito.<Long>any());
        MockHttpServletRequestBuilder requestBuilder = MockMvcRequestBuilders.delete("/dummies/{id}", 1L);
        requestBuilder.contentType("https://example.org/example");
        MockMvcBuilders.standaloneSetup(dummyController)
                .build()
                .perform(requestBuilder)
                .andExpect(MockMvcResultMatchers.status().isOk());
    }
```
一覧(list())のテストが何故か testAdd() になってしまいましたが、無事テストが生成されました。
validation が機能しているかテストしてみましょう。
testAdd2を書き換えます。（実際にはコピーしてテストを増やしましょう）
```Java
    @Test
    void testAdd2() throws Exception {
        when(dummyUseCase.allDummies()).thenReturn(new ArrayList<>());

        DummyAddRequest dummyAddRequest = new DummyAddRequest();
        dummyAddRequest.setComment("");

```
とコメントを空にしてテストを実行すると…なぜか成功してしまいます。
よく見るとリクエストが Get になっているので Post に直します。
```Java
        MockHttpServletRequestBuilder requestBuilder = MockMvcRequestBuilders.post("/dummies")
```
実行すると無事テストは失敗しました。
![validation1.png](validation1.png)
どこが期待と違ったのかが画面に表示されるため、テストを直すかプログラムを直します。
![validation2.png](validation2.png)
今回はバリデーションに失敗するテストなので badRequest を期待します。
また、エラー処理をまだ書いていないのでエラーメッセージが帰ってこないのでレスポンスボディのチェックを削除します。
```Java
        MockMvcBuilders.standaloneSetup(dummyController)
                .build()
                .perform(requestBuilder)
                .andExpect(MockMvcResultMatchers.status().isBadRequest());
```
これでテストが成功するようになりました！

## Part 2 特に Spring に関係ないコードのテスト

This is the second part of the tutorial:

1. Step 1
2. Step 2
3. Step n

## What you've learned {id="what-learned"}

Summarize what the reader achieved by completing this tutorial.

<seealso>
<!--Give some related links to how-to articles-->
</seealso>
