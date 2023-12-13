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

## Part 1  Controller のテストを作成する
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

Spring の機能を使わない純粋な Java コードはそのままテストが作成されます。

![pojo1.png](pojo1.png)

![pojo2.png](pojo2.png)

ただ、テストが通るように生成されるため、バグが有っても生成されたテストそのままではバグが発見できません。

![pojo3.png](pojo3.png)

![pojo4.png](pojo4.png)

## Part 3 Service のテスト

サービスも簡単にテストを作ることが出来、
リポジトリのMock化もしてくれます。

![service1.png](service1.png)

![service2.png](service2.png)

```Java
@ContextConfiguration(classes = {DummyService.class})
@ExtendWith(SpringExtension.class)
class DummyServiceDiffblueTest {
    @MockBean
    private ChildRepository childRepository;

    @MockBean
    private DummyRepository dummyRepository;

    @Autowired
    private DummyService dummyService;

    /**
     * Method under test: {@link DummyService#getDummies()}
     */
    @Test
    void testGetDummies() {
        ArrayList<Dummy> dummyList = new ArrayList<>();
        when(dummyRepository.findAll()).thenReturn(dummyList);
        List<Dummy> actualDummies = dummyService.getDummies();
        verify(dummyRepository).findAll();
        assertTrue(actualDummies.isEmpty());
        assertSame(dummyList, actualDummies);
    }

    /**
     * Method under test: {@link DummyService#getChildren(Long)}
     */
    @Test
    void testGetChildren() {
        ArrayList<Child> childList = new ArrayList<>();
        when(childRepository.findByDummyId(Mockito.<Long>any())).thenReturn(childList);
        List<Child> actualChildren = dummyService.getChildren(1L);
        verify(childRepository).findByDummyId(Mockito.<Long>any());
        assertTrue(actualChildren.isEmpty());
        assertSame(childList, actualChildren);
    }

    /**
     * Method under test: {@link DummyService#saveDummy(Dummy)}
     */
    @Test
    void testSaveDummy() {
        Dummy dummy = new Dummy();
        ArrayList<Child> childList = new ArrayList<>();
        dummy.setChildList(childList);
        dummy.setComment("Comment");
        dummy.setDummyId(1L);
        dummy.setName("Name");
        dummy.setText("Text");
        when(dummyRepository.save(Mockito.<Dummy>any())).thenReturn(dummy);

        Dummy dummy2 = new Dummy();
        dummy2.setChildList(new ArrayList<>());
        dummy2.setComment("Comment");
        dummy2.setDummyId(1L);
        dummy2.setName("Name");
        dummy2.setText("Text");
        dummyService.saveDummy(dummy2);
        verify(dummyRepository).save(Mockito.<Dummy>any());
        assertEquals("Comment", dummy2.getComment());
        assertEquals("Name", dummy2.getName());
        assertEquals("Text", dummy2.getText());
        assertEquals(1L, dummy2.getDummyId().longValue());
        assertEquals(childList, dummy2.getChildList());
    }

    /**
     * Method under test: {@link DummyService#deleteDummy(Long)}
     */
    @Test
    void testDeleteDummy() {
        doNothing().when(dummyRepository).deleteById(Mockito.<Long>any());
        dummyService.deleteDummy(1L);
        verify(dummyRepository).deleteById(Mockito.<Long>any());
    }
}
```
##  Part 4 Repository のテスト

Repository のテストも作成できますが、DB接続設定からテストコードを作るので設定完了後である必要があります。
![repository1.png](repository1.png)

生成されたテストコードは最初コメントアウトされています。これはテストコードが埋め込みDB（H2SQL）にデフォルトでアクセスしようとするからで、
接続に失敗したり接続したは良いもののテーブル情報が無いためどうしてもエラーになります。（デフォルトで既存DBにアクセスできれば無事に生成できそうですが…）

![repository2.png](repository2.png)

@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)を追加することでテストが動くようになります。

```Java
@ContextConfiguration(classes = {ChildRepository.class})
@EnableAutoConfiguration
@EntityScan(basePackages = {"com.example.testing.entity"})
@DataJpaTest(properties = {"spring.main.allow-bean-definition-overriding=true"})
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE) // ←自動生成時はこの行が無いのでDBアクセスに失敗して動かない。デフォルトの挙動をこれにする方法があればそのまま動くと思うのだが…
class ChildRepositoryDiffblueTest {
    @Autowired
    private ChildRepository childRepository;

    @Test
//    @Disabled("TODO: Complete this test") // ←この行が生成されていてテストから除外されるため、この行は削除する。
    void testFindByDummyId() {
        // TODO: Complete this test. // ここからDB接続に失敗した時の StackTrace がコメントとして残されている。
```

DB 接続失敗の StackTrace がコメントとして残されているので削除すると以下のようになります。
　Child.dummyId は外部キーなので flyway などを用いてテスト用データのマイグレーションをするべきでしょう。

```Java
@ContextConfiguration(classes = {ChildRepository.class})
@EnableAutoConfiguration
@EntityScan(basePackages = {"com.example.testing.entity"})
@DataJpaTest(properties = {"spring.main.allow-bean-definition-overriding=true"})
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE) // 自動生成時はこの行が無いのでDBアクセスに失敗して動かない。デフォルトの挙動をこれにする方法があればそのまま動くと思うのだが…
class ChildRepositoryDiffblueTest {
    @Autowired
    private ChildRepository childRepository;

    /**
     * Method under test: {@link ChildRepository#findByDummyId(Long)}
     */
    @Test
    void testFindByDummyId() {
        Child child = new Child();
        child.setDummyId(1L);
        child.setName("Name");
        child.setText("Text");

        Child child2 = new Child();
        child2.setDummyId(2L);
        child2.setName("com.example.testing.entity.Child");
        child2.setText("com.example.testing.entity.Child");
        childRepository.save(child);
        childRepository.save(child2);
        childRepository.findByDummyId(1L);
    }
}

