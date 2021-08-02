# デザインパターン

Gang Of Fourデザインパターンは『オブジェクト指向における再利用のためのデザインパターン』において4人の著者によって提案された"23種類の機能要件に適したコード設計"のこと

- **オブジェクトの生成に関するデザインパターン**
    <img width="964" alt="スクリーンショット 2021-05-19 19.57.55.png (64.4 kB)" src="https://img.esa.io/uploads/production/attachments/9489/2021/05/19/90193/3adb310f-167c-4deb-b3d8-4a3827156329.png">

- **プログラムの構造に関するデザインパターン**
    <img width="938" alt="スクリーンショット 2021-05-19 19.58.03.png (72.4 kB)" src="https://img.esa.io/uploads/production/attachments/9489/2021/05/19/90193/28ce2b1a-128e-49c2-b6dc-fc923912a50d.png">

 - **オブジェクトの振る舞いに関するデザインパターン**
    <img width="982" alt="スクリーンショット 2021-05-19 19.58.11.png (124.4 kB)" src="https://img.esa.io/uploads/production/attachments/9489/2021/05/19/90193/6391f3e8-3b1d-45f6-8815-c5a9184bd974.png">

## Template Method編
アルゴリズムの共通部分を抽象化して、振る舞いが異なる部分を具象クラスで実装するパターン
特に、<u>処理の流れが一緒で振る舞いだけが異なる場合に有効らしい</u>

<u>処理の流れを親クラス</u>=><u>順次呼ばれるメソッドをサブクラス</u>として定義する


<img width="300" alt="スクリーンショット 2021-05-19 20.18.49.png (134.4 kB)" src="https://img.esa.io/uploads/production/attachments/9489/2021/05/19/90193/281477e2-dc11-4da4-b3dd-f3a8e8fe0530.png">

サブクラス側で親クラスをオーバーライド(ECMAの場合はextendsして同名メソッドを再定義すること)によって実現する。


```
public abstract class 社員 {

    public void 一日の行動() {

        身支度をする();
        通勤電車に乗る();
        現場で仕事をする();
    }

    protected abstract void 身支度をする();

    protected abstract void 通勤電車に乗る();

    protected abstract void 現場で仕事をする();
}
```



```
public class 鈴木 extends 社員 {

    @Override
    protected void 身支度をする() {
        // 洗面所で顔を洗う
    }

    @Override
    protected void 通勤電車に乗る() {
        // 京急線を使う
    }

    @Override
    protected void 現場で仕事をする() {
        // テスト計画書を作る
    }
}


public class 佐藤 extends 社員 {

    @Override
    protected void 身支度をする() {
        // シャワーを浴びる
    }

    @Override
    protected void 通勤電車に乗る() {
        // 山手線を使う
    }

    @Override
    protected void 現場で仕事をする() {
        // プログラムを作る
    }
}


public class 高橋 extends 社員 {

    @Override
    protected void 身支度をする() {
        // 朝風呂に入る
    }

    @Override
    protected void 通勤電車に乗る() {
        // 総武線を使う
    }

    @Override
    protected void 現場で仕事をする() {
        // 設計書を作る
    }
}
```

**まとめ**
- 処理の内容は違っても処理の流れは一緒のような状況は割と起こるので便利っぽい


## Singleton編
生成するインスタンスの数を1つに制限するデザインパターン

**Singletonはインスタンスを一つしか作らないことを「保証する」ための仕組み**
**=> データの不整合を避けられる**



---


**Singletonの特徴**
- publicのコンストラクタを持たず、privateのコンストラクタを持つ。
- private変数として、自身のクラスインスタンスを持つ。
- public関数として、自身のクラスインスタンスを返すgetterメソッドを持つ
この3点がsingletonである条件である


#### 実際のSKのコードから探してみた！
```sls-body-karte-cloud-api/src/domain/Token/Token.ts
export class HearingSheetToken extends Token {
  static VALID_HOURS = 6;

  readonly sheetID: SheetID;

  constructor(params: IHearingSheetToken) {
    super(params);
    this.sheetID = params.sheetID;
  }

  canAccess(sheet: HearingSheet<any>): boolean {
    return this.sheetID === sheet.id;
  }

  static create(
    sheet: HearingSheet<any>,
    currentTime: Date = new Date()
  ): HearingSheetToken {
    return new HearingSheetToken({
      id: v4() as TokenID,
      expirationTime: dayjs(currentTime)
        .add(HearingSheetToken.VALID_HOURS, "hours")
        .toDate(),
      sheetID: sheet.id,
      branchID: sheet.branchID,
      companyID: sheet.companyID,
      createdAt: currentTime,
      updatedAt: currentTime
    });
  }
}
```

#### 調べると出てくる依存性注入(なんだこいつは)
- この記事などで比較されている(https://ichi.pro/swift-de-no-shinguru-ton-tai-izonsei-chunyu-151790763576968)
- strategyパターンに類似するらしい。


## 依存性注入(DI)  & Stratagy編
### 依存性注入(Dependency injection)とは
　<u>**クラス間の依存関係を薄くすること**</u>を目的としたクラスの書き方
　テストの書きやすさや、汎用性を上げるためにクラス同士の依存関係を極力減らすテクニック

### 種類
- **Interface injection(Strategyパターン)**
 =>注入用のインタフェースを定義して注入を行う方法
- **Constructor injection**
 =>コンストラクタを定義して注入を行う方法
- **Setter injection**
 =>setter メソッドを定義して注入を行う方法

###  <u>①Interface injection(Strategyパターン)</u>
interfaceにメソッド群を定義し、作成するクラスにそのインターフェースを使用することで依存性を注入するパターン

```sample.ts
// Interface injectionの例
interface Animal {
  cry: () => string;
}

class Dog implements Animal {
  cry() {
    return "ワン";
  }
}

class Cat implements Animal {
  cry() {
    return "ニャオ";
  }
}
```
---
#### 実際のコード内の例
```Slack.ts
interface AdminNotification {
  created(
    obj: { entityName: string; id: string; name: string },
    meta: Readonly<{
      adminDetails: AdminDetails;
      company?: Company;
      branch?: Branch;
    }>
  ): Promise<void>;

  updated(
    obj: { entityName: string; id: string; name: string },
    meta: Readonly<{
      adminDetails: AdminDetails;
      company?: Company;
      branch?: Branch;
    }>
  ): Promise<void>;

  read(
    obj: { id: string; name: string },
    meta: Readonly<{
      adminDetails: AdminDetails;
    }>
  ): Promise<void>;
}
```

```Slack.ts
// このClass(AdminEmptyNotification)とAdminSlackNotificationで同じinterfaceを使用
class AdminEmptyNotification implements AdminNotification {
  async created(
    _obj: { entityName: string; id: string; name: string },
    _meta: Readonly<{
      adminDetails: AdminDetails;
      company?: Company;
      branch?: Branch;
    }>
  ) {
    // Do nothing
  }

  async updated(
    _obj: { entityName: string; id: string; name: string },
    _meta: Readonly<{
      adminDetails: AdminDetails;
      company?: Company;
      branch?: Branch;
    }>
  ) {
    // Do nothing
  }

  async read(
    _obj: { id: string; name: string },
    _meta: Readonly<{
      adminDetails: AdminDetails;
    }>
  ) {
    // Do nothing
  }
}
```

###  <u>②Constructor injection</u>
コンストラクタに使用したいクラスを渡せるようにすることで依存性を注入できるようにするパターン
```sample.ts
// Constructor injectionの例
class Animal {
  cry(voice: string) {
    return `${voice}`;
  }
}

class Dog {
  private animal;

  constructor(animal: Animal) {
    this.animal = animal;
  }

  cry() {
    this.animal.cry("ワン");
  }
}

class Cat {
  private animal;

  constructor(animal: Animal) {
    this.animal = animal;
  }

  cry() {
    this.animal.cry("ニャオ");
  }
}
```


#### 実際のコード内の例
```StaffAuth0Factory.ts
export class StaffAuth0Factory implements StaffFactory {
  private readonly client: ManagementClient;

  constructor(client: ManagementClient) {
    this.client = client;
  }

  async createStaff(payload: InitialStaff & StaffCredentials): Promise<Staff> {
    try {
      const { user_id } = await this.client.createUser({
        email: payload.loginID,
        connection: "Username-Password-Authentication",
        password: payload.password
      });

      const providerID = user_id!.split("|")[1]; // auth0|

      return new Staff({
        providerName: "Auth0" as StaffProviderName,
        providerID: providerID as StaffProviderID,
        branchID: payload.branchID as BranchID,
        companyID: payload.companyID as CompanyID,
        name: payload.name,
        isDisplayScore: payload.isDisplayScore,
        isDisplayRank: payload.isDisplayRank,
        isDisplayRecommend: payload.isDisplayRecommend,
        karteMode: payload.karteMode,
        remoteKarteMode: payload.remoteKarteMode,
        version: 0,
        isActive: true,
        allowed: payload.allowed
      });
    } catch (error) {
      if (typeof error.message === "string") {
        throw new ValidationErrors().add(
          ValidationErrorType.INVALID,
          error.message
        );
      }
      throw error;
    }
  }
}
```



###  <u>③Setter injection</u>
https://qiita.com/1000k/items/aef6aed46b0fc34cc15e

## 依存性逆転の原則

### SOLID原則
1. Single Responsibility Principle：単一責任の原則
1. Open/closed principle：オープン/クロースドの原則
1. Liskov substitution principle：リスコフの置換原則
1. Interface segregation principle：インターフェース分離の原則
1. **Dependency inversion principle：依存性逆転の原則**⬅️これ

簡単に。。

### 1. Single Responsibility Principle：単一責任の原則
クラスは1つのことだけ責任を負うべき(例: repository)

### 2. Open/closed principle：オープン/クロースドの原則
新機能を追加するときに、拡張はしやすく(オープン)、修正はする必要がない(クローズ)ようにする

### 3. Liskov substitution principle：リスコフの置換原則
拡張されたオブジェクトを利用している箇所では、拡張元のオブジェクトも使えるべき(派生オブジェクト)

### 4. Interface segregation principle：インターフェース分離の原則
インターフェースに特定の場合しか使わない値を追加するべきでない、そういった場合は別のインターフェースに分離するべき

### 5. Dependency inversion principle：依存性逆転の原則
抽象に依存するべき
DDDのアーキテクチャ=>Repository(抽象)とRepositoryImpl(詳細)のように
直接RepositoryImpl(詳細)に依存するのではなく、共通な部分のみを抽出したRepository(抽象)を使った方が、影響が少なくなり修正などでの変更が少なくなる


## ポリモーフィズム
また今度。。

## DIを踏まえてStrategyパターンとは何なのか
- 依存性逆転の原則より、抽象クラスに依存するべきである
- 依存性注入Interface injectionより、interfaceを使って共通のクラス型を定義するべきである(インターフェース分離の原則を守る)



 ### DDDのRepositoryで理解を深める

#### *抽象クラス(RepositoryImplBase)*
```RepositoryImplBase.ts
export abstract class RepositoryImplBase<T> {
  protected tableName: string;
  protected dynamoClient: DocumentClient;

  constructor(tableName: string, dynamoClient: DocumentClient) {
    this.tableName = tableName;
    this.dynamoClient = dynamoClient;
  }
  // ~省略 
  async scanSafely(
    consumer: (list: T[]) => Promise<void>,
    options: Partial<DocumentClient.QueryInput> = {}
  ): Promise<number> {
    const tableName = this.tableName;
    const dynamoClient = this.dynamoClient;
    // ~省略
  }
}
```

#### *interface(Repository)*

```ExerciseRepository.ts
export interface ExerciseRepository {
  find(branch: Branch): Promise<ExerciseSettings | undefined>;

  create(
    exerciseSettings: ExerciseSettings,
    currentTime?: Date
  ): Promise<ExerciseSettings>;

  update(
    exerciseSettings: ExerciseSettings
  ): Promise<ExerciseSettings | undefined>;
}
```

#### *詳細クラス(RepositoryImpl)*
```ExerciseRepositoryImpl.ts
export class ExerciseRepositoryImpl
  extends RepositoryImplBase<ExerciseSettingsItem>
  implements ExerciseRepository {
  constructor(dynamoClient: DocumentClient) {
    super(config.dynamo.tables.corp, dynamoClient);
  }
  
  async find(
    exerciseSettings: ExerciseSettings,
    currentTime: Date = new Date()
  ): Promise<ExerciseSettings> {
  // ~省略
  }
  
  async create(
    exerciseSettings: ExerciseSettings,
    currentTime: Date = new Date()
  ): Promise<ExerciseSettings> {
  // ~省略
  }
  
  async update(
    exerciseSettings: ExerciseSettings,
    currentTime: Date = new Date()
  ): Promise<ExerciseSettings> {
  // ~省略
  }
 
}
```

この構成になっていることにより
既存ロジックの変更を行いたいときは、RepositoryImplだけ変更すれば良いし
新しいロジックを追加したいときは、Repositoryにメソッドを追加して、詳細をRepositoryImplに追記すれば良い

このように最小限の変更で実装可能にするのがStrategyパターンおよび、DIやSOLID原則に則ったオブジェクト指向プログラミングの強み。

(Repositoryを分けているのはここに異なるためインターフェイス分離の原則?)


## Composite編
初めての**プログラムの構造に関するデザインパターン**
Compositeパターンはツリー構造(再帰的な構造)を表現する際に役立つデザインパターン である。
<img width="300" alt="スクリーンショット 2021-06-18 17.01.37.png (65.9 kB)" src="https://img.esa.io/uploads/production/attachments/9489/2021/06/18/90193/72c7427f-c479-4f18-be78-5b1845e99c85.png">

### Sample
代表例としてOSのディレクトリ構造がよく使われている

```Entry.ts
abstract class Entry {
  public abstract getName():string;
  public abstract getSize():number;
}
```

```File.ts
class File extends Entry {
  name:string;
  size:number;
  
  constructor(name:string, size:number) {
    this.name = name;
    this.size = size;
  }
  public getName():string {
  }
  public getSize():number {
  }
}
```

```Directory.ts
class Directory extends Entry {
  name:string;
  directory: Entry[];

  constructor(name:string, size:number) {
    super();
    this.name = name;
    this.size = size;
  }
  public getName():string {
  }
  public getSize():number {
  }
  public add(entry:Entry):Entry {
    this.directory.push(entry);
    return this;
  }
}
```

同じ抽象クラスを継承して二つのクラスが作られ
片方のクラスでは抽象クラスを引数とするメソッドが作られているのが特徴的

### 実際の実装
```Entry.ts
abstract class Entry {
  public abstract getName():string;
  public abstract getStaffNum():number;
}
```

```Branch.ts
class Branch extends Entry {
  name:string;
  staffNum:number;
  
  constructor(name:string, staffNum:number) {
    this.name = name;
    this.staffNum = staffNum;
  }
  public getName():string {
  }
  public getStaffNum():number {
  }
}
```

```Area.ts
class Area extends Entry {
  name:string;
  staffNum:number;
  areaAndBranch: Entry[]
  
  constructor(name:string, staffNum:number) {
    this.name = name;
    this.staffNum = staffNum;
  }
  public getName():string {
  }
  public getStaffNum():number {
  }
  public add(entry:Entry):Entry {
    this.areaAndBranch.push(entry);
    return this;
  }
}
```

Composite とは、英語で「複合物」という意味で
容器(DirectoryやArea)と内容物(FileやBranch)を抽象クラスにより同一のものとして扱い
再帰的な構造を作るのに役立つデザインパターンである。


## Proxy編
Proxyは代理人の意
**忙しくて仕事ができない本人オブジェクトの代わりに、代理人オブジェクトが一部の仕事をこなす方式**

- 共通のinterfaceから本人クラスと代理人クラス(窓口)を作成
- 本人クラスは代理人クラスを通してのみ呼び出すことができる。

### SAMPLE

```OfficeWorker.ts
// 会社員
interface OfficeWorker {
  // 連絡
  contact: () => void;
  // ゴルフ
  golf: () => void;
}
```

```Secretary.ts
// 秘書
class Secretary implements OfficeWorker {
  president: President;
  
  constructor() {
    this.president = new President();
  }

  contact() {
    console.log("連絡する");
  }
  
  golf() {
    this.president.golf();
  }
}
```

```President.ts
// 社長
class President implements OfficeWorker {
  constructor() {}
  
  contact() {
    console.log("連絡する");
  }
  
  golf() {
    console.log("ゴルフをする");
  }
}
```

### 実際の実装
見つからなかったので参考URLから実用例を抜粋
- http://marupeke296.com/DP_Proxy.html
- http://capm-network.com/?tag=%E3%83%87%E3%82%B6%E3%82%A4%E3%83%B3%E3%83%91%E3%82%BF%E3%83%BC%E3%83%B3-Proxy

①ゲーム内のオブジェクト(vertual proxy)
<img width="200" alt="image.png (828.4 kB)" src="https://img.esa.io/uploads/production/attachments/9489/2021/06/29/90193/f284876d-daae-43b5-aa91-ec8b3345a22f.png">

RPGゲームなどで常にフィールド上の全オブジェクトを読み込むのは処理が重くなってしまうため
画面に表示する分のオブジェクトだけ読み込むようにすれば処理が早くなる。
このようにオブジェクトを生成したり描画したりする事にコストがかかるとき、オブジェクトに仮想的になりすましてそれを軽減させるproxyの書き方をvertual proxyと呼ぶ

②権限チェック(protection proxy)
proxy(代理人)に権限チェックをさせてから、本人を呼び出すようにすることで権限チェックを行うことができる。


## Abstract Factory & Factory Method 編

- Abstract Factoryは工場まるごと、FactoryMethodは工場機能を一部変更したいときに使われる?
- FactoryMethodはTemplateMethodに近いデザインパターン

### 1. Abstract Factory
https://www.techscore.com/tech/DesignPattern/AbstractFactory.html/
直訳「抽象的な工場」

抽象クラスを用いて、具象クラスの実装を気にせずに呼び出す側の整合性を保てる。

#### SAMPLE

```Burger.ts
export abstract class Burger {
  abstract getBans(): Bans;
  abstract getMeat(): Meat;
  abstract getVegetable(): Vegetable;
  abstract getOther(): Other;
}
```

```HumBurger.ts
export class HumBurger extends Burger{
   getBans(): Bans;
   getMeat(): Meat;
   getVegetable(): Vegetable;
   getOther(): Other;
}
```

```CheeseBurger.ts
export class CheeseBurger extends Burger{
   getBans(): Bans;
   getMeat(): Meat;
   getVegetable(): Vegetable;
   getOther(): Other;
}
```

```BurgerFactory.ts
export abstract class BurgerFactory {
  abstract createBurger(): Burger; 
}
```

```HamburgerFactory.ts
export class HamburgerFactory extends BurgerFactory{
  createBurger(): Burger{
    return new HumBurger();
  };
}
```

```CheeseBurgerFactory.ts
export class CheeseBurgerFactory extends BurgerFactory{
  createBurger(): Burger{
    return new CheeseBurger();
  };
}
```

```Main.ts
export class Main {
  getFactory(withCheese: boolean): BurgerFactory {
    if (withCheese) {
      return new CheeseburgerFactory();
    } else {
      return new HamburgerFactory();
    }
  }
  
   main(withCheese: boolean): void {
    const factory = this.createBurger(withCheese);
    const burger = factory.createBurger();
    burger.sand()
  }
}
```

共通の処理を抽象クラスで定義し、具象クラスで共通処理の具体内容を記述、呼び出す側では抽象クラスを期待するので、具象クラスの中身どうなっていようと関係なく扱える。

### 2. Factory Method
http://www.itsenka.com/contents/development/designpattern/factory_method.html
Template methodの応用版で、部品生成用のメソッドを用意し、それをオーバーライドする形で実装する

```Burger.ts
export abstract class Burger {
  abstract getBans(): Bans;
  abstract getMeat(): Meat;
  abstract getVegetable(): Vegetable;
  abstract getOther(): Other;
  abstract sand(): Burder;
  
  createBurger(): Burger {
    this.getBans();
    this.getMeat();
    this.getVegetable();
    this.getOther();
    return this.sand();
  }
}
```

処理の順序が決まっている場合はFactoryMethodが使える。
逆に、それ以外の場合はAbstractFactoryの方が汎用性が高い。

行いたい処理が同じだが、処理の一部を複数パターン用意したい時に役立つデザインパターン。











https://www.macky-studio.com/entry/2019/06/19/234617


## デザインパターン復習編
https://www.amazon.co.jp/%E5%A2%97%E8%A3%9C%E6%94%B9%E8%A8%82%E7%89%88Java%E8%A8%80%E8%AA%9E%E3%81%A7%E5%AD%A6%E3%81%B6%E3%83%87%E3%82%B6%E3%82%A4%E3%83%B3%E3%83%91%E3%82%BF%E3%83%BC%E3%83%B3%E5%85%A5%E9%96%80-%E7%B5%90%E5%9F%8E-%E6%B5%A9/dp/4797327030

## Builderパターン
- 構造をもったインスタンスを組み上げていくパターンです。
この章での重要なメッセージは”交換可能性”です。
”交換可能性”とはインスタンスの入れ替えが可能な作りにコードがなっていることを指します。

### 抽象クラス


### 詳細クラス①


### 詳細クラス②

とあったとして
詳細クラス①と詳細②のどちらも受け入れられる形に作って置くことが大事、これができていれば未来で新しく詳細クラスを定義することになったとしても、難なく追加することができるはず。