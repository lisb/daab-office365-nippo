# 日報ボット for Office 365

## はじめに

このドキュメントは、direct と Office 365 を連携させた日報ボット(以下、ボット)について、各種設定から実行するまでの手順書となっています。そのため、direct および Office 365 の両サービスをご契約・ご利用中のものとしています。

まだ、ご利用でない方は、[direct](https://direct4b.com/ja/) および [Office 365](http://www.microsoft.com/ja-jp/office/365/) のそれぞれに無料トライアルがありますので、そちらをご参照ください。

## ボット用アカウントの取得

ボット用に、新しくメールアドレスを用意します。

### direct 

通常のユーザと同じように、ボット用アカウントを作成します。

組織の管理ツールから、ボット用メールアドレスに招待メールを送信します。
管理ツールのご利用方法については、[こちらの管理ツールマニュアル](https://direct4b.com/ja/manual_dl.html)をご参照ください。管理ツールのご利用には権限が必要です。お持ちでない方は、契約者もしくは管理者にご連絡下さい。

組織に招待されると、ボット用メールアドレスにメールが届きます。
メールに記載されているURLをクリックしてアカウント登録手続きをしてください。ご利用開始までの手順については、[こちらの導入マニュアル](https://direct4b.com/ja/manual_dl.html)をご参照ください。

[ログインページ](https://direct4b.com/signin)からボット用メールアドレスでログインします。
招待を承認する画面が開きますので、その画面で「承認」を選択してください。
次に、設定＞プロフィール編集より、表示名とプロフィール画像をボット用に変更します。

### Office 365

連携アプリケーションの開発のためには Office365 に Developer Site の追加が必要です。追加の方法は現在の契約状態によって異なりますので、[こちらのドキュメント](http://msdn.microsoft.com/en-us/office/office365/howto/setup-development-environment#bk_Office365Account)を参考に適切な方法を選択して追加してください。

連携アプリケーションの認証のためには Microsoft Azure Active Directory でのアプリケーション登録が必要です。[Microsoft Azure Management](https://manage.windowsazure.com/) を開き、Active Directory のアプリケーションタブを開き、下段のメニューからアプリケーションを追加します。Microsoft Azure Management でアプリケーションの追加方法については、[こちらのヘルプページ](http://msdn.microsoft.com/library/azure/dn151122.aspx)をご参照ください。

アプリケーション情報の指定では、「ネイティブ クライアント アプリケーション」を選択し、リダイレクト URI には ``http://localhost/`` を指定します。追加した後は構成の「他のアプリケーションに対するアクセス許可」で、「Office 365 SharePoint Online」を選択し、「デリゲートされたアクセス許可」のチェックをすべてオンにします。アプリケーションの構成方法については、[こちらのヘルプページ](http://msdn.microsoft.com/en-us/library/dn132599.aspx#BKMK_Native)をご参照ください。

## node のインストール

[http://nodejs.org/](http://nodejs.org/) から最新版をインストールします。v0.10.32 で動作確認しています。

## サンプルプログラムの設定

サンプルプログラムを[ダウンロード](office365-nippo-download.html)して展開します。以降は、この展開したディレクトリ(フォルダ)にて、コマンドライン(コマンドプロンプト)で作業することになります。

### direct

direct へのアクセスには、アクセストークンが利用されます。アクセストークンの取得には、アクセストークンを環境変数に設定していない状態で、以下のコマンドを実行し、ボット用のメールアドレスとパスワードを入力します。

	$ bin/hubot
	Email: loginid@bot.email.com
	Password: *****
	0123456789ABCDEF_your_direct_access_token

以下の環境変数に、アクセストークンを設定します。
	
	$ export HUBOT_DIRECT_TOKEN=0123456789ABCDEF_your_direct_access_token
	

### Office 365

Office 365 へのアクセスには、[OAuth2](http://msdn.microsoft.com/en-us/office/office365/howto/common-app-authentication-tasks)が利用されます。

以下の環境変数に、取得したアプリケーションIDおよびシークレットを設定します。

	export O365_CLIENT_ID=

ボットに初めて話しかけたとき、コンソールにURLが表示されて処理が停止します。

	$ bin/hubot
	...
	https://login.windows.net/common/oauth2/authorize?....
	redirect url? 
	
この URL をブラウザで開き、Office 365 の日報ボット用アカウントでログインします。ログインが成功すると認可画面が表示されますので承認してください。その後、http://localhost/?code=... というURLにリダイレクトされます。「ページの読み込みエラー」といった画面になるかもしれませんが、コマンドラインに戻ってそのURLを入力します。

	redirect url? http://localhost/?code=AAABAAAA...

トークンの情報は起動したディレクトリの ``.storage`` ファイルに保存されます。次回起動時はこの内容が読み込まれます。

### 日報フォルダとテンプレートファイル

日報はテンプレートファイルから生成され、日報フォルダにアップロードされます。

Office 365 の OneDrive for Business に``日報``という名前のフォルダを作成し、日報ボットの利用者が読み書きできるようにアクセス権限を設定します。

この日報フォルダに``日報テンプレ.docx``というファイルを置きます。テンプレートファイルのサンプルは、ダウンロードしたファイルに含まれています。

テンプレートファイル名とフォルダ名は、以下の環境変数で変更することができます。未設定の場合は以下の値になります。

	export NIPPO_FOLDER_NAME=日報
	export NIPPO_TEMPLATE_NAME=日報テンプレ.docx

## サンプルプログラムの実行

以下のコマンドを実行します。

	$ bin/hubot
