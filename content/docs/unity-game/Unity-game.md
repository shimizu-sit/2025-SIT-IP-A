---
title: "ゲームづくり"
weight: 1
bookMenu: main
---

# UnityでC#プログラミングをしてみよう

---

# 授業内容

## 今回の目標

- 「キー入力」のスクリプトをアタッチする
- ボールを「跳ね返す」スクリプトを書く
- 「失敗」（取り損ねた）ボールの消去
- 「リプレイ」をつくりましょう
- ボールを跳ね返すオブジェクト
- 回転するオブジェクト
- 課題

# 「キー入力」のスクリプトをアタッチする

## Ballオブジェクトの設定

- Ballに「**Rigidbody**」と「**Physic Material**」をつける

### 「Rigidbody」を追加

- 「**Hierarchy**」から「**Ball**」オブジェクトを選択した状態で右側にある「**Inspector**」をみて下までスクロールします

Unity-001.png

- 「**Inspector**」内の一番下にある「**Add Component**」をクリックして「**Physics**」を選択します

Unity-002.png

- 「**Physics**」内にある「**Rigidbody**」をクリックします

2024-04-26_15h28_27.png

- 「**Ball**」オブジェクトの「**Inspector**」内に「**Rigidbody**」コンポーネントが追加されている事を確認してください

Unity-004.png

### 「Physic material」を追加

- オブジェクトにセットする「**材質**」によって跳ね返り方が変化します
- **Asset** > **Create** > **Physic Material**を選択します

Unity-005.png

- 「**Assets**」内に「**New Physic Material**」が作成されますので，それを右クリックして「**Rename**」を選択して名前を「**Ball**」に変更
- します

Unity-006.png

- 「**Ball**」フィジクマテリアルを選択すると「**Inspector**」に**Physic Material**のパラメータが表示されます

Unity-007.png

- 「**Physic Material**」の設定内容

|       **項目**       |                 **説明**                 |          **値の範囲など**          |
| -------------------- | ---------------------------------------- | ---------------------------------- |
| **Dynamic Friction** | 移動するオブジェクトにかかる摩擦         | 0 = 摩擦がない　1 = すぐに止まる   |
| **Static Friction**  | 静止してるオブジェクトにかかる摩擦       | 0 = 摩擦がない　1 = 動きにくい     |
| **Bounciness**       | 跳ね返りの大きさ                         | 0 = 跳ね返らない　1 = よく跳ね返る |
| **Friction Combine** | 衝突したオブジェクト間の摩擦             | 今回は初期値のAverageのまま        |
| **Bounce Combine**   | 衝突したオブジェクト間の跳ね返りの大きさ | 今回は初期値のAverageのまま        |

今回はスーパーボールのような挙動にしたいので**Dynamic Friction**と**Static Friction**は**0**に近い値に設定します．さらに**Bounciness**は**1**に設定します．

- 作った「**Ball**」フィジックマテリアルを「**Ball**」オブジェクトに**attach**（追加）します
  - **attach**する方法は3つあります
    1. 「**Hierarchy**」上の「**Ball**」にattachする方法
    2. 「**Scene**」ビュー上の「**Ball**」にattachする方法
    3. 「**Ball**」の「**Inspector**」内の「**Sphere Collider**」の「**Material**」にattachする方法

Unity-008.png

- 「**Play**」を押して実行してみましょう
  - Ballの跳ね返り具合などを調整してください

- キーボードの右左の方向キーで操作する「**PlayerMove**」スクリプトを「**Paddle**」オブジェクトにattachします
  - attachする方法は3つあります
    1. 「**Hierarchy**」上の「**Paddle**」にattachする方法（説明省略）
    2. 「**Scene**」ビュー上の「**Paddle**」にattachする方法（説明省略）
    3. 「**Paddle**」の「**Inspector**」内の「**Add Componet**」の「**Scripts**」で「**PlayerMove**」をattachする方法

Unity-009.png

- PlayerMoveスクリプトの中身は以下の様になっています

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerMove : MonoBehaviour
{
    // float型でspeed変数を定義し15を代入
    public float speed = 15f;

    void FixedUpdate()
    {
        // キー入力の取得と速度の計算
        var velox = speed * Input.GetAxisRaw("Horizontal");

        // Rigidbodyコンポーネントを取得して速度を設定
        GetComponent<Rigidbody>().velocity = new Vector3(velox, 0f, 0f);
    }
}
```

- 「**Paddle**」オブジェクトに「**Rigidbody**」を追加します
- 「**Paddle**」オブジェクトの「**Rigidbody**」に調整を行います

Unity-010.png

- 「**Rigidbody**」内にある「**Constrains**」でPlayerの移動方向，回転方向を決めることができます
- 今回はPlayerが**「X軸方向」のみ**に移動すればよいので「**Freeze PositionのX以外**」にチェックをいれます

Unity-011.png

- 「**Play**」を押して実行してみましょう
  - 方向キーに左右を押して「**Paddle**」が左右に移動することを確認してください
  - 左右の移動速度を変更したい場合は，「**Paddle**」オブジェクトの「**PlayerMove**」スクリプト内にある「**Speed**」の値を変更してみていください

# ボールを「跳ね返す」スクリプトを書く

## よく跳ねるためにはどうすればよいか？

Unity-012.png

- 物理シミュレーションでは「**Paddle**」に当たった後はいずれ止まってしまいます
- 当たった時に，擬似的な「**はね返す力**」を上方向に加えることで，はね続けるボールをつくることができます

## 「跳ね返す」スクリプトを書く

- 「跳ね返す」スクリプト＝「**Bumper**」スクリプト
- 当たった相手に上向きの力を加えるスクリプトです

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Bumper : MonoBehaviour
{
  // float型でbounce変数を宣言し10を代入
  public float bounce = 10f;
  
  void OnCollisionEnter(Collision other)
  {
    // 衝突したオブジェクトのRigidbodyに力を加える
    // x軸方向には0の力を，Y軸方向にはbounce/6の力を，z軸方向にはbounceの力を与える
    // 力は瞬間的に与えるのでForceModeをImpulseに設定
    other.rigidbody.AddForce(0f, bounce / 6, bounce, ForceMode.Impulse);
  }
}
```

### 「AddForce」の中身について

- `AddForce`の第1引数がX軸方向，第2引数がY軸方向，第3引数がZ軸方向となっています
- Floorの傾き加減によってY軸方向の力の大きさを調整する必要があります

Unity-013.png

- 「**Bumper**」スクリプトを「**Paddle**」オブジェクトにattachする
  - attachする方法は3つあります
    1. 「**Hierarchy**」上の「**Paddle**」にattachする方法（説明省略）
    2. 「**Scene**」ビュー上の「**Paddle**」にattachする方法（説明省略）
    3. 「**Paddle**」の「**Inspector**」内の「**Add Componet**」でattachする方法（説明省略）

- 「**Play**」を押して実行してみましょう
  - 「**Ball**」が「**Paddle**」に当たったときの跳ね返りを調整してください
  - 跳ね返すを調整したい場合は，「**Paddle**」オブジェクトの「**Bumper**」スクリプト内にある「**Bounce**」の値を変更してみていください

## 「Tag」の役目

- 「**Tag**」はゲーム内のオブジェクトを区別するために使います
- 例えば，シューテュングゲームの場合，エネミーの弾にあたるのは「**プレイヤーのTag**」を持ったオブジェクトだけにできます
- エネミーの弾は「**エネミーのTag**」のオブジェクトには当たらない（味方は破壊できない）という処理がわかりやすく簡単にできます

Unity-014.png

- 「**Ball**」オブジェクトを選びます
- 「**Ball**」のInspectorの「**Tag**」の「Untagged」をクリック
- ドロップダウンリストから一番下の「**Add Tag…**」を選択します

Unity-015.png

- Inspectorに「**Tags & Layers**」が表示されます
- 右側にある「**＋**」をクリックします
- 「**New Tag Name**」が表示されます
- 「**Ball**」と入力してSaveします

Unity-016.png

Unity-017.png

Unity-018.png

- 再び「**Ball**」オブジェクトを選択してInspectorの「**Tag**」から「**Ball**」タグを追加します

Unity-019.png

- 「Bumper」スクリプトにコードを追加します

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Bumper : MonoBehaviour
{
  // float型でbounce変数を宣言し10を代入
  public float bounce = 10f;

  void OnCollisionEnter(Collision other)
  {
    if(other.gameObject.tag == "Ball") { // 追加するコード
      // 衝突したオブジェクトのRigidbodyに力を加える
      // x軸方向には0の力を，Y軸方向にはbounce/6の力を，z軸方向にはbounceの力を与える
      // 力は瞬間的に与えるのでForceModeをImpulseに設定
      other.rigidbody.AddForce(0f, bounce / 6, bounce, ForceMode.Impulse);
    }  // 閉じカッコも忘れずに！**
  }
}
```

- 「**Play**」を押して実行してみましょう
  - 「**Ball**」が「**Paddle**」に当たったときの跳ね返りを調整してください
  - 跳ね返すを調整したい場合は，「**Paddle**」オブジェクトの「**Bumper**」スクリプト内にある「**Bounce**」の値を変更してみていください

# 「失敗」（取り損ねた）ボールの消去

## 「Kill」スクリプトの中身を編集

- 「**Kill**」スクリプトを中身を以下のように編集します

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Kill : MonoBehaviour {

  void OnCollisionEnter(Collision other) {
    // killWallオブジェクトに当たったオブジェクトのタグが「Ball」の場合以下の処理をおこなう
    if(other.gameObject.tag == "Ball") {
      // 当たったオブジェクトを0.1秒後に削除する
      Destroy(other.gameObject, 0.1f);
    }
  }
}
```

- 「**kill**」スクリプトを「**KillWall**」オブジェクトにattachします

- 「**Play**」を押して実行してみましょう
  - 「**Ball**」が「KillWall」に当たったときに「Ball」が消えるか確認してください

## ボールの再セットする仕組み

- 「失敗」（取りそこねる）とボールを消すことをします
- 次にボールを再セットする仕組み（リプレイ）をつくります
- これで繰り返しゲームを遊ぶことができます
- 「ゲームの進行工程（シーケンス）」

## 「GameManager」を作りゲームを管理する

- C#スクリプトの作成とファイルに名前をつけます
- スクリプトの名前はとても大切です（後から変更するのが大変です）
- 名前は大文字から始まる名前にします（記号などは使えません）
- ここでは「**GameManager**」という名前にします

- 「**Project**」の「**Assets**」内にある「**Script**」フォルダを選択します
- Scriptフォルダ内が表示されるのでそこで「**右クリック**」をします
- 表示されたリストの一番上にある「**Create**」を選択します
- 更に表示されたリストの上から二番目にある「**C# Script**」を選択します

Unity-020.png

- スクリプトアイコンが表示されます
- すぐにファイル名を「**GameManager**」に変更してください

Unity-021.png

Unity-022.png

- 「Empty（空っぽ）」のゲームオブジェクトをつくります
- 「**GameObject**」を選択します
- 表示されたリストから「**Create Empty**」を選択します

Unity-023.png

- 作成した空っぽの「**GameObject**」オブジェクトを選択します
- Inspectorからオブジェクト名を「**GameManager**」とします
- 「**GameManager**」スクリプトをattachします

Unity-024.png

- 「GameManager」スクリプトの中身を編集します

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
public class GameManager : MonoBehaviour
{
  // GameObject型の「ballPrefab」という名前の変数を作成
  public GameObject ballPrefab;
  
  void Update()
  {
    // オブジェクト名が「Ball」のオブジェクトを探して，GameObject型の変数「ballObj」にいれる
    GameObject ballObj = GameObject.Find("Ball");
    // 「ballObj」の中身がScene内に存在しなくなったら以下の処理を行う
    if (ballObj == null)
    {
      // Inspector欄でセットされたオブジェクト「ballPrefab」を作成してGameObject型の変数「newBall」にいれる
      GameObject newBall = Instantiate(ballPrefab);
      // 「ballPrafab」の名前を「newBall」の名前にいれる
      newBall.name = ballPrefab.name;
    }
  }
}
```

## 「Ball」のPrefabを作成します

- Hierarchy内の「**Ball**」オブジェクトをドラッグします
- そのまま「**Assets**」にドロップします
- Hierarchy内の「**Ball**」オブジェクトが青くなります

Unity-025.png

- Hierarchyの「**GameManager**」オブジェクトを選択します
- Inspectorの「**GameManager**」コンポーネントを確認します
- 先ほど作成したAssets内の「**Ball**」プレファブを「**GameManager**」コンポーネントの「**Ball Prefab**」にドラッグアンドドロップします

Unity-026.png

- 「**Play**」を押して実行してみましょう
  - 「**Ball**」が「**KillWall**」に当たったときに「**Ball**」が消え再度「**Ball**」が表示されるか確認してください

# ボールを跳ね返すオブジェクト

- Hierarchy内の「BoundObject」を選択してください
- 「BoundObject」に「Bumper」スクリプトをattachしてください

- 「**Play**」を押して実行してみましょう
  - 「**Ball**」が「**BoundObject**」に当たったときに跳ね返ることを確認してください

# 回転するオブジェクト

## 「Rotation」スクリプトを作成してattachします

- C#スクリプトの作成とファイルに名前をつけます
- スクリプトの名前はとても大切です（後から変更するのが大変です）
- 名前は大文字から始まる名前にします（記号などは使えません）
- ここでは「Rotation」という名前にします

- 「**Project**」の「**Assets**」内にある「**Script**」フォルダを選択します
- Scriptフォルダ内が表示されるのでそこで「**右クリック**」をします
- 表示されたリストの一番上にある「**Create**」を選択します
- 更に表示されたリストの上から二番目にある「**C# Script**」を選択します

Unity-020.png

- Rotationスクリプトを中身を編集します

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.AI;

public class Rotation : MonoBehaviour
{
  // float型のrotAngle変数に4.0を入れる
  public float rotAngle = 4.0f;
  
  void FixedUpdate()
  {
    // Inspector欄のtransformのRotateのY軸の回転にrotAngleの角度だけ回転させる
    transform.Rotate(0f, -rotAngle, 0f);
  }
}
```

- Hierarchy欄の「**RotateObject**」オブジェクトに「**Rotation**」スクリプトをattachしてください

- 「**Play**」を押して実行してみましょう
  - 「**RotateObject**」が回転していることを確認してください
  - 回転速度を調整したい場合は，「**RotateObject**」オブジェクトの「**Rotation**」スクリプト内にある「**Rot Angle**」の値を変更してみてください

---

# 課題

- 完成したゲームを更に改造してください
- 改造したゲームのプレゼン資料を作成してください
- プレゼン資料はスライド形式（パワポやGoogle Slideなど）で作成してください
- プレゼン資料に掲載する内容は以下の通りです（掲載順番は自由です）
  - アピールポイント
  - プレイ時のスクショ
  - 今回の授業を通して楽しかったこと
  - 今回の授業を通して大変だった，難しかったこと
  もっとやってみたいこと
- 提出はスライド形式をPDFに変換してMoodleに提出してください
- 提出期限：**8月5日(月)終日**