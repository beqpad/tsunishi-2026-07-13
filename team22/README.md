# Google Classroom 機能改善プロジェクト

## 1. 研究の背景と目的

学校で日常的に使用しているGoogle Classroomには、生徒・教師の両方が不便に感じる機能上の制約が存在する。本研究では、使用者側の視点からGoogle Classroomの課題を洗い出し、GASや外部サービスとの連携によって機能不足を補う改善策を開発・検証する。

---

## 2. リサーチクエスチョンと仮説

**RQ：** 自分たちが既存のGoogle Classroomに対して、より便利にするには何ができるか。

**仮説：** 使用者側の視点から改善案を出し、外部ツール（GASまたは外部サービス）との連携によって機能不足を解消できる。

---

## 3. 前提知識・現状整理

### Google Classroomアカウントの種類と制限

Google Classroomは、個人用Gmailアカウント（無料）と学校・組織向けGoogle Workspace（有料）で利用可能な機能が大きく異なる。

| 機能項目 | 個人向け無料版（@gmail.com） | 学校・組織向け（Google Workspace） |
| :--- | :--- | :--- |
| **接続制限** | 個人アカウント同士のみ | 学校の独自ドメインで管理。外部アカウントの参入を制限可能 |
| **Google Meet 連携** | 毎回URLを発行して共有 | クラスごとに常設の参加リンクを固定設置可能 |
| **Classroomアドオン** | 利用不可 | 有料プランで利用可 |
| **演習セット・動画機能** | 利用不可 | AIがヒントを出すインタラクティブな課題が可能 |
| **独自性レポート** | 制限あり（確認回数が少ない） | 提出物の自動照合が可能 |
| **ストレージ容量** | 15GB（全サービス共有） | 組織全体で100TB以上（プランによる） |
| **管理者コントロール** | なし | 管理コンソールから一括制限が可能 |

### 外部ツールによる拡張の概要

機能不足を補う手段として、主に以下の2つのアプローチがある。

| アプローチ | 概要 | 向いているケース |
| :--- | :--- | :--- |
| **GAS（Google Apps Script）** | JavaScriptベース。サーバー不要で無料利用可能。Classroom APIと連携して自動化処理を実装できる | 繰り返し作業の自動化（課題の一括配布・集計など） |
| **外部サービス連携** | PythonなどからClassroom APIを呼び出し、独自のWebアプリや通知システムと組み合わせる | Classroomにない機能を別サービスで補う（通知、ダッシュボード、分析など） |

---

## 4. 研究デザインと実験計画

### 改善対象と課題の特定

**（生徒・教師へのヒアリング後に記入）**

> ヒアリング項目の例：
> * Google Classroomを使っていて「できない」「不便だ」と感じた具体的な場面は何か
> * 生徒として不便なこと・教師として不便なことはそれぞれ何か
> * もし1つだけ改善できるとしたら、最も解決したい問題は何か

### 改善アプローチの選定

ヒアリングで特定した課題の内容に応じて、GAS拡張・外部サービス連携のいずれか（または組み合わせ）を選定する。

---

## 5. 計測方法と評価指標

改善の成否は以下の3段階で評価する。

| 評価軸 | 測定方法 |
| :--- | :--- |
| **機能不足の解消**（最低条件） | 「以前はできなかったことが、改善後にできるようになったか」をチェックリストで確認 |
| **作業時間の短縮** | 改善前後で同じ作業にかかった時間を計測・比較 |
| **使用率・満足度の向上** | 改善後のアンケートで「使いやすくなったか」を5段階評価で回収 |

---

## 6. 研究プロセス（フェーズ別）

| フェーズ | 内容 |
| :--- | :--- |
| **現状調査フェーズ** | 生徒・教師へのヒアリングで不満・課題を収集し、優先度を整理 |
| **課題定義フェーズ** | 改善対象を1〜2つに絞り、RQと仮説を具体化 |
| **開発フェーズ** | GASまたは外部サービスとの連携を実装 |
| **検証フェーズ** | 実際の学校環境で動作させ、評価指標を計測 |
| **発表準備フェーズ** | 結果の整理・スライド作成・発表練習 |

---

## 7. 最終アウトプット（スライド発表構成案）

| スライド | 内容 |
| :--- | :--- |
| 表紙 | テーマ・班名 |
| 背景と目的 | Google Classroomを使っていて感じた課題 |
| RQと仮説 | 何を改善し、どう解決できると考えたか |
| 現状整理 | 無料版と有料版の機能差、外部ツールでできること |
| 改善アプローチ | GASまたは外部サービス連携の選定理由と設計 |
| 実装内容 | 開発したコード・仕組みのデモ |
| 検証結果 | 評価指標（機能解消・時間短縮・満足度）の計測結果 |
| 考察 | できたこと・できなかったこと・工夫した点 |
| 結論・提言 | 他の学校でも使えるか、次の改善アイデア |

---

## 付録：GASおよびPython APIの実装サンプル

### GAS：クラスの一括作成

Google スプレッドシートの「拡張機能」>「Apps Script」に貼り付けて実行する。
事前にGAS画面左側の「サービス」から **Classroom API** を追加すること。

```javascript
function createClassroomCourses() {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  const data = sheet.getDataRange().getValues(); // 1行目がヘッダー、2行目からデータ

  for (let i = 1; i < data.length; i++) {
    const className = data[i][0]; // A列：クラス名
    const section   = data[i][1]; // B列：セクション（例：1組、2組）

    if (!className) continue;

    const courseInfo = {
      name: className,
      section: section,
      ownerId: 'me',
      courseState: 'ACTIVE'
    };

    try {
      const createdCourse = Classroom.Courses.create(courseInfo);
      Logger.log('クラスを作成しました: ' + createdCourse.name + ' (ID: ' + createdCourse.id + ')');
    } catch (e) {
      Logger.log('エラーが発生しました: ' + e.toString());
    }
  }
}
```

### Python API：クラス一覧の取得（接続テスト）

事前準備：
1. Google Cloud Console でプロジェクトを作成し、**Google Classroom API** を有効化
2. OAuth 2.0 クライアント ID を作成し、`credentials.json` としてダウンロード
3. `pip install google-auth-oauthlib google-api-python-client` を実行

```python
import os.path
from google.auth.transport.requests import Request
from google.oauth2.credentials import Credentials
from google_auth_oauthlib.flow import InstalledAppFlow
from googleapiclient.discovery import build
from googleapiclient.errors import HttpError

SCOPES = ["https://www.googleapis.com/auth/classroom.courses.readonly"]

def main():
    creds = None
    if os.path.exists("token.json"):
        creds = Credentials.from_authorized_user_file("token.json", SCOPES)
    if not creds or not creds.valid:
        if creds and creds.expired and creds.refresh_token:
            creds.refresh(Request())
        else:
            flow = InstalledAppFlow.from_client_secrets_file("credentials.json", SCOPES)
            creds = flow.run_local_server(port=0)
        with open("token.json", "w") as token:
            token.write(creds.to_json())

    try:
        service = build("classroom", "v1", credentials=creds)
        result = service.courses().list(pageSize=10).execute()
        courses = result.get("courses", [])

        if not courses:
            print("有効なクラスが見つかりませんでした。")
            return

        for course in courses:
            print(f"クラス名: {course.get('name')} (ID: {course.get('id')})")

    except HttpError as error:
        print(f"APIエラーが発生しました: {error}")

if __name__ == "__main__":
    main()
```
