# Google Classroom 検証・開発仕様書

Google Classroom を検証目的で使用するためのアカウント選定、機能比較、および各種 API / 外部連携（GAS・Python）の実装仕様をまとめたドキュメントです。

---

## 1. 検証用サインアップと環境選定

Google Classroom は、特別な審査や契約手続きをすることなく、**個人の通常アカウント（Gmail アカウント）で今すぐ無料でサインアップして検証目的で利用可能**です。

### 💡 検証のアプローチ
1. **手軽に基本機能をテストしたい場合（個人アカウント）**
   * 日常的に使用している `〜@gmail.com` のアカウントで [Google Classroom 公式サイト](https://classroom.google.com/) にアクセスするだけで開始可能。
   * 無料の Gmail アカウントを2つ（教師役1つ、生徒役1つ）用意すれば、課題の配布・提出・採点の一連のフローを1台のPCやスマホで100%体験・検証できます。
2. **本番環境に近い管理機能をテストしたい場合（Workspace）**
   * 組織全体の管理、セキュリティポリシー、高度な連携機能を検証したい場合は、企業向け・教育機関向けの `Google Workspace` の無料トライアル環境や、テスト用サブドメインの構築が必要です。

---

## 2. 無料（個人向け）と有料・組織向け（Workspace）の機能比較

生徒側および管理者側での主な仕様・制限の違いは以下の通りです。

| 機能項目 | 個人向け無料版 (`@gmail.com`) | 学校・組織向け (Google Workspace) |
| :--- | :--- | :--- |
| **接続制限** | 個人アカウント同士でのみやり取り可能 | 学校の独自ドメイン（`@school.ed.jp` 等）で管理。外部アカウントの参入を制限可能 |
| **Google Meet 連携** | 会議ごとに毎回URLを発行して共有 | クラスごとに「常設の参加リンク」を固定設置可能 |
| **Classroom アドオン** | 利用不可 | 有料プランで利用可。他社製学習アプリを Classroom 内に統合可能 |
| **演習セット・動画機能**| 利用不可 | 有料プランで利用可。AIがヒントを出すインタラクティブな課題 |
| **独自性レポート** | 制限あり（確認回数が少ない） | 生徒の提出物が Web 上の文章のコピペでないか自動照合 |
| **ストレージ容量** | 15 GB（ドライブやメール全体で共有） | 組織全体で 100 TB 以上の膨大な共有ストレージ（プランによる） |
| **管理者コントロール** | なし（各ユーザーが管理） | 管理コンソールから一括で機能制限（チャット禁止等）が可能 |

---

## 3. Google Apps Script (GAS) による拡張
JavaScript ベースで、サーバーを用意することなく Classroom の自動化ができる環境です。個人無料アカウントで今すぐ完全無料で開発・テストが可能です。

### 📜 サンプルコード（クラスの一括作成）
Google スプレッドシートの「拡張機能」>「Apps Script」に貼り付けて実行します。
※事前に GAS 画面左側の「サービス」から **Classroom API** を追加してください。

```javascript
function createClassroomCourses() {
  // スプレッドシートのアクティブシートからデータを取得
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  const data = sheet.getDataRange().getValues(); // 1行目がヘッダー、2行目からデータと想定
  
  // 2行目から順番に処理
  for (let i = 1; i < data.length; i++) {
    const className = data[i][0]; // A列：クラス名
    const section   = data[i][1]; // B列：セクション（例：1組、2組）
    
    if (!className) continue;
    
    // Classroom APIの仕様に沿ったオブジェクトの作成
    const courseInfo = {
      name: className,
      section: section,
      ownerId: 'me', // 実行している本人のアカウントが教師になる
      courseState: 'ACTIVE'
    };
    
    try {
      // クラスの作成実行
      const createdCourse = Classroom.Courses.create(courseInfo);
      Logger.log('クラスを作成しました: ' + createdCourse.name + ' (ID: ' + createdCourse.id + ')');
    } catch (e) {
      Logger.log('エラーが発生しました: ' + e.toString());
    }
  }
}
```

---

## 4. Python による 外部 API 連携

Python から Google Classroom API を呼び出し、データの取得や生徒と先生のやり取りをシミュレートする実装例です。

### 🛠️ 事前準備
1. Google Cloud Console でプロジェクトを作成し、**Google Classroom API** を有効化。
2. 「認証情報」から **OAuth 2.0 クライアント ID** を作成し、`credentials.json` としてダウンロード。
3. 必要なライブラリのインストール:
   ```bash
   pip install google-auth-oauthlib google-api-python-client
   ```

### 📜 例1：所属するクラス一覧の取得（基本接続テスト）
```python
import os.path
from google.auth.transport.requests import Request
from google.oauth2.credentials import Credentials
from google_auth_oauthlib.flow import InstalledAppFlow
from googleapiclient.discovery import build
from googleapiclient.errors import HttpError

SCOPES = ["https://googleapis.com"]

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
        print("クラス一覧を取得中...")
        result = service.courses().list(pageSize=10).execute()
        courses = result.get("courses", [])

        if not courses:
            print("有効なクラスが見つかりませんでした。")
            return

        print("\n--- あなたのクラス一覧 ---")
        for course in courses:
            print(f"クラス名: {course.get('name')} (ID: {course.get('id')})")

    except HttpError as error:
        print(f"APIエラーが発生しました: {error}")

if __name__ == "__main__":
    main()
```

### 📜 例2：先生とのやり取り（生徒視点での課題確認・メッセージ・提出）
```python
import os.path
from google.auth.transport.requests import Request
from google.oauth2.credentials import Credentials
from google_auth_oauthlib.flow import InstalledAppFlow
from googleapiclient.discovery import build

# 課題の確認・コメント送信・提出に必要な権限（スコープの重複は修正済み）
SCOPES = ["https://googleapis.com"]

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

    service = build("classroom", "v1", credentials=creds)

    # 検証用の各種ID（実際の環境の値に書き換えてください）
    COURSE_ID = "123456789012"      
    COURSEWORK_ID = "987654321012"  

    try:
        # 1. 先生が出した課題（Coursework）の内容を確認する
        print("💡 1. 先生からの課題を取得しています...")
        assignment = service.courses().courseWork().get(
            courseId=COURSE_ID, 
            id=COURSEWORK_ID
        ).execute()
        print(f"【課題タイトル】: {assignment.get('title')}")
        print(f"【先生からの指示】: {assignment.get('description')}\n")

        # 2. 先生に非公開コメント（プライベートメッセージ）を送る
        print("💬 2. 先生に質問メッセージを送信します...")
        comment_body = {
            "text": "先生、課題のプログラムが完成しました。確認をお願いします！"
        }
        comment_response = service.courses().courseWork().studentSubmissions().comments().create(
            courseId=COURSE_ID,
            courseWorkId=COURSEWORK_ID,
            submissionId="me", # 'me' は実行中の生徒自身を指す
            body=comment_body
        ).execute()
        print(f"👉 メッセージを送信しました: \"{comment_response.get('text')}\"\n")

        # 3. 課題を「提出（TurnIn）」する
        print("🚀 3. 課題を提出（Turn In）します...")
        submissions_result = service.courses().courseWork().studentSubmissions().list(
            courseId=COURSE_ID,
            courseWorkId=COURSEWORK_ID,
            userId="me"
        ).execute()
        
        submissions = submissions_result.get("studentSubmissions", [])
        
        if submissions:
            submission_id = submissions[0].get("id")
            service.courses().courseWork().studentSubmissions().turnIn(
                courseId=COURSE_ID,
                courseWorkId=COURSEWORK_ID,
                id=submission_id,
                body={}
            ).execute()
            print("✅ 課題の提出が完了しました！")
        else:
            print("❌ 提出データが見つかりませんでした。")

    except Exception as e:
        print(f"エラーが発生しました: {e}")

if __name__ == "__main__":
    main()
```
