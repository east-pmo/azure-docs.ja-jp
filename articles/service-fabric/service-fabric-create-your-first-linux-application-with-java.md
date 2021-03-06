---
title: "Java を使用して Linux で初めての Azure マイクロサービス アプリを作成する | Microsoft Docs"
description: "Java を使用して Service Fabric アプリケーションを作成およびデプロイします"
services: service-fabric
documentationcenter: java
author: rwike77
manager: timlt
editor: 
ms.assetid: 02b51f11-5d78-4c54-bb68-8e128677783e
ms.service: service-fabric
ms.devlang: java
ms.topic: hero-article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 06/02/2017
ms.author: ryanwi
ms.translationtype: Human Translation
ms.sourcegitcommit: 43aab8d52e854636f7ea2ff3aae50d7827735cc7
ms.openlocfilehash: 4ffab9eb858b05d3f5894e8753fb0fd4f38c4087
ms.contentlocale: ja-jp
ms.lasthandoff: 06/03/2017


---
# <a name="create-your-first-service-fabric-java-application-on-linux"></a>Linux で初めての Service Fabric Java アプリケーションを作成する
> [!div class="op_single_selector"]
> * [C# - Windows](service-fabric-create-your-first-application-in-visual-studio.md)
> * [Java - Linux](service-fabric-create-your-first-linux-application-with-java.md)
> * [C# - Linux](service-fabric-create-your-first-linux-application-with-csharp.md)
>
>

このクイック スタートでは、Linux 開発環境で初めての Azure Service Fabric Java アプリケーションをほんの数分で作成できます。  作業が終了すると、ローカル開発クラスターで実行される単純な Java 単一サービス アプリケーションが作成されます。  

## <a name="prerequisites"></a>前提条件
作業を開始する前に、[Linux 開発環境](service-fabric-get-started-linux.md)に Service Fabric SDK と Azure CLI をインストールし、開発クラスターをセットアップします。 Mac OS X を使用している場合は、[Vagrant を使用して仮想マシンに Linux 開発環境をセットアップする](service-fabric-get-started-mac.md)ことができます。

## <a name="create-the-application"></a>アプリケーションを作成する
Service Fabric アプリケーションには、アプリケーションの機能を提供する際にそれぞれ特定の役割を果たすサービスが 1 つ以上含まれます。 Linux 用の Service Fabric SDK には、[Yeoman](http://yeoman.io/) ジェネレーターが含まれています。これを使用すると、初めてサービスを作成したり、後で追加したりする作業が簡単になります。  Eclipse 用のプラグインを使用して、Service Fabric Java アプリケーションを作成、ビルド、およびデプロイすることもできます。 [Eclipse を使用した初めての Java アプリケーションの作成とデプロイ](service-fabric-get-started-eclipse.md)に関するページを参照してください。 このクイック スタートでは、Yeoman を使用して、カウンター値の格納と取得という単一のサービスを行うアプリケーションを作成します。

1. ターミナルで、「``yo azuresfjava``」と入力します。
2. アプリケーションに名前を付けます。 
3. 最初のサービスの種類を選択し、名前を付けます。 このチュートリアルでは、[Reliable Actor サービス] を選択します。 他の種類のサービスの詳細については、「[Service Fabric プログラミング モデルの概要](service-fabric-choose-framework.md)」を参照してください。
   ![Java 用 Service Fabric Yeoman ジェネレーター][sf-yeoman]

## <a name="build-the-application"></a>アプリケーションのビルド
Service Fabric Yeoman テンプレートには、[Gradle](https://gradle.org/) のビルド スクリプトが含まれています。端末からアプリをビルドするために、これを使用することができます。 アプリをビルドしてパッケージ化するには、次のコマンドを実行します。

  ```bash
  cd myapp
  gradle
  ```

## <a name="deploy-the-application"></a>アプリケーションのデプロイ
アプリケーションがビルドされたら、Azure CLI を使用してローカル クラスターにデプロイできます。

1. ローカルの Service Fabric クラスターに接続します。

    ```bash
    azure servicefabric cluster connect
    ```

2. テンプレートに用意されているインストール スクリプトを実行してクラスターのイメージ ストアにアプリケーション パッケージをコピーし、アプリケーションの種類を登録して、アプリケーションのインスタンスを作成します。

    ```bash
    ./install.sh
    ```

3. ブラウザーを開き、http://localhost:19080/Explorer の [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) に移動します (Mac OS X で Vagrant を使用している場合は、localhost を VM のプライベート IP に置き換えます)。

4. **[アプリケーション]** ノードを展開し、アプリケーションの種類のエントリと、その種類の最初のインスタンスのエントリができたことを確認します。

## <a name="start-the-test-client-and-perform-a-failover"></a>テスト クライアントの起動と、フェールオーバーの実行
アクターは単独では何も実行しません。メッセージを送信するには、別のサービスまたはクライアントが必要です。 アクター テンプレートには、アクター サービスとの対話に使用できる簡単なテスト スクリプトが含まれています。

1. ウォッチ ユーティリティを使用してスクリプトを実行し、アクター サービスの出力を確認します。  テスト スクリプトは、アクターに対して `setCountAsync()` メソッドを呼び出してカウンターを増分させ、`getCountAsync()` メソッドを呼び出して新しいカウンター値を取得し、その値をコンソールに表示します。

    ```bash
    cd myactorsvcTestClient
    watch -n 1 ./testclient.sh
    ```

2. Service Fabric Explorer で、アクター サービスのプライマリ レプリカをホストしているノードを見つけます。 次のスクリーンショットでは、ノード 3 です。 プライマリ サービス レプリカは、読み取り操作と書き込み操作を処理します。  サービスの状態の変更がセカンダリ レプリカにレプリケートされます (次のスクリーンショットでは、ノード 0 のノード 1 で実行されています)。

    ![Finding the primary replica in Service Fabric Explorer][sfx-primary]

3. **[ノード]** で、前の手順で見つけたノードをクリックし、[アクション] メニューの **[非アクティブにする (再起動)]** を選択します。 この操作によって、プライマリ サービス レプリカを実行しているノードが再起動され、別のノードで実行されているセカンダリ レプリカのいずれかに強制的にフェールオーバーされます。  フェールオーバーされたセカンダリ レプリカがプライマリに昇格し、別のノードに別のセカンダリ レプリカが作成され、プライマリ レプリカによる読み取り/書き込み操作が開始されます。 ノードが再起動したら、テスト クライアントからの出力を監視して、フェールオーバーにかかわらず、カウンターが増加し続けることを確認してください。

## <a name="add-another-service-to-the-application"></a>別のサービスをアプリケーションに追加する
`yo` を使用して既存のアプリケーションにサービスを追加するには、次の手順を実行します。
1. ディレクトリを既存アプリケーションのルートに変更します。  たとえば、Yeoman で作成したアプリケーションが `MyApplication` の場合は、`cd ~/YeomanSamples/MyApplication` です。
2. `yo azuresfjava:AddService` を実行します。
3. 上記の手順に従って、アプリをビルドしてデプロイします。

## <a name="remove-the-application"></a>アプリケーションを削除する
テンプレートに用意されているアンインストール スクリプトを使用して、アプリ インスタンスを削除し、アプリケーション パッケージの登録を解除し、クラスターのイメージ ストアからアプリケーション パッケージを削除します。

```bash
./uninstall.sh
```

Service Fabric explorer で、アプリケーションとアプリケーションの種類が **[アプリケーション]** ノードに表示されていないことを確認します。

## <a name="next-steps"></a>次のステップ
* [Linux で初めての Service Fabric Java アプリケーションを作成する](service-fabric-get-started-eclipse.md)
* [Service Fabric Reliable Actors の概要](service-fabric-reliable-actors-introduction.md)
* [Azure CLI を使用した Service Fabric クラスターの対話操作](service-fabric-azure-cli.md)
* [デプロイのトラブルシューティング](service-fabric-azure-cli.md#troubleshooting)
* [Service Fabric のサポート オプション](service-fabric-support.md)について学びます。

<!-- Images -->
[sf-yeoman]: ./media/service-fabric-create-your-first-linux-application-with-java/sf-yeoman.png
[sfx-primary]: ./media/service-fabric-create-your-first-linux-application-with-java/sfx-primary.png
[sf-eclipse-templates]: ./media/service-fabric-create-your-first-linux-application-with-java/sf-eclipse-templates.png

