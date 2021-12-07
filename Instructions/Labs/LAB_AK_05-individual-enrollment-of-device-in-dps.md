---
lab:
    title: 'ラボ 05: DPS のデバイスの個別登録'
    module: 'モジュール 3: 大規模なデバイス プロビジョニング'
---

# DPS でのデバイスの個別登録

## ラボ シナリオ

Contoso の経営陣は、IoT デバイスを使用して現在のシステムで必要な手動のデータ入力作業を削減し、出荷プロセス時により高度な監視機能を提供する、自社の資産の監視および追跡ソリューションの更新を強く要求しています。このソリューションは、IoT デバイスのプロビジョニングとプロビジョニング解除を行う機能に依存しています。プロビジョニング要件を管理するための最良の選択肢は DSP のようです。

提案されたシステムは、輸送中の輸送コンテナーの場所、温度、圧力をトラッキングするために、統合されたセンサーを備えた IoT デバイスを使用します。デバイスは、Contoso がチーズの輸送に使用する既存の出荷コンテナー内に配置され、車両が提供する WiFi を使用して Azure IoT Hub に接続します。新しいシステムは、製品環境の継続的な監視を提供し、問題が検出されたときにさまざまな通知シナリオを可能にします。

Contoso のチーズ パッケージング施設では、空のコンテナーがシステムに入ると、新しい IoT デバイスが装備され、パッケージ化されたチーズ製品がロードされます。IoT デバイスは、デバイス プロビジョニング サービスを使用して IoT Hubに自動プロビジョニングする必要があります。コンテナーが宛先に到着すると、IoT デバイスが取得され、DPS を通じて "使用停止" されます。デバイスは、今後の出荷に再利用されます。

DPS を使用してデバイス プロビジョニングとプロビジョニング解除プロセスを検証する作業を担当します。プロセスの初期段階では、個別登録アプローチを使用します。

次のリソースが作成されます。

![ラボ 5 アーキテクチャ](media/LAB_AK_05-architecture.png)

## このラボでは

このラボでは、次のタスクを完了します。

* ラボの前提条件が満たされていることを確認する (必要な Azure リソースがあること)
* DPS での新しい個人登録の作成
* シミュレートされたデバイスの構成
* シミュレートされたデバイスのテスト
* デバイスの削除

## ラボの手順

### 演習 1: ラボの前提条件を確認する

このラボでは、次の Azure リソースが使用可能であることを前提としています。

| リソースの種類:  | リソース名 |
| :-- | :-- |
| リソース グループ | rg-az220 |
| IoT Hub | iot-az220-training-{your-id} |
| デバイス プロビジョニング サービス | dps-az220-training-{your-id} |

これらのリソースが利用できない場合は、演習 2 に進む前に、以下の手順に従って **lab05-setup.azcli** スクリプトを実行する必要があります。スクリプト ファイルは、開発環境構成 (ラボ 3) の一部としてローカルに複製した GitHub リポジトリに含まれています。

**lab05-setup.azcli** スクリプトは、**Bash** シェル環境で動作するように記述されています。これは、Azure Cloud Shell で実行するのが最も簡単です。

1. ブラウザーを使用して [Azure Cloud Shell](https://shell.azure.com/) を開き、このコースで使用している Azure サブスクリプションでログインします。

    Cloud Shell のストレージの設定に関するメッセージが表示された場合は、デフォルトをそのまま使用します。

1. Azure Cloud Shell が **Bash** を使用していることを確認 します。

    「Azure Cloud Shell」 ページの左上隅にあるドロップダウンは、環境を選択するために使用されます。選択されたドロップダウンの値が **Bash**であることを確認します。 

1. Azure Shell ツール バーで、「**ファイルのアップロード/ダウンロード**」 をクリックします (右から 4 番目のボタン)。

1. ドロップダウンで、「**アップロード**」 をクリックします。

1. ファイル選択ダイアログで、開発環境を構成したときにダウンロードした GitHub ラボ ファイルのフォルダーの場所に移動します。

    _ラボ 3: 開発環境の設定_:ZIP ファイルをダウンロードしてコンテンツをローカルに抽出することで、ラボ リソースを含む GitHub リポジトリを複製しました。抽出されたフォルダー構造には、次のフォルダー パスが含まれます。

    * Allfiles
      * Labs
          * 05-Individual Enrollment of a Device in DPS
            * Setup

    lab05-setup.azcli スクリプト ファイルは、ラボ 5 の設定 フォルダー内にあります。

1. **lab05-setup.azcli** ファイルを選択し、「**開く**」 をクリックします。

    ファイルのアップロードが完了すると、通知が表示されます。

1. 正しいファイルがアップロードされたことを確認するには、次のコマンドを入力します。

    ```bash
    ls
    ```

    `ls` コマンドを実行すると、現在のディレクトリの内容が一覧表示されます。lab05-setup.azcli ファイルが一覧表示されます。

1. セットアップ スクリプトを含むディレクトリをこのラボ用に作成し、そのディレクトリに移動するには、次の Bash コマンドを入力します。

    ```bash
    mkdir lab5
    mv lab05-setup.azcli lab5/
    cd lab5
    ```

    これらのコマンドは、このラボのディレクトリを作成し、そのディレクトリに **lab05-setup.azcli** ファイルを移動して、新しいディレクトリを現在の作業ディレクトリに変更します。

1. **lab05-setup.azcli** スクリプトに実行権限があることを確認するには、次のコマンドを入力します。

    ```bash
    chmod +x lab05-setup.azcli
    ```

1. Cloud Shell ツール バーで、lab05-setup.azcli ファイルを編集するには、「**エディターを開く**」 (右から 2 番目のボタン - **{ }**) をクリックします。 

1. 「**ファイル**」 の一覧で、lab6 フォルダーを展開してスクリプト ファイルを開き 、**lab5**、「**lab05-setup.azcli**」 の順にクリックします。

    エディターで **lab05-setup.azcli** ファイルの内容を表示します。

1. エディターで、`{YOUR-ID}` と `{YOUR-LOCATION}` 変数の値を更新します。

    サンプル例として、このコースの最初に作成した一意の ID 、つまり **CAH191211** に `{YOUR-ID}` を設定し、リソースにとって意味のある場所に `{YOUR-LOCATION}` を設定する必要があります。

    ```bash
    #!/bin/bash

    RGName="rg-az220"
    IoTHubName="iot-az220-training-{your-id}"

    Location="{YOUR-LOCATION}"
    ```

    > **注意**:  `{YOUR-LOCATION}` 変数は、リージョンの短い名前に設定する必要があります。次のコマンドを入力すると、使用可能な領域とその短い名前 (**名前** 列) の一覧を表示できます。
    >
    > ```bash
    > az account list-locations -o Table
    >
    > DisplayName           Latitude    Longitude    Name
    > --------------------  ----------  -----------  ------------------
    > East Asia             22.267      114.188      eastasia
    > Southeast Asia        1.283       103.833      southeastasia
    > Central US            41.5908     -93.6208     centralus
    > East US               37.3719     -79.8164     eastus
    > East US 2             36.6681     -78.3889     eastus2
    > ```

1. エディター画面の右上で、ファイルに加えた変更を保存してエディタを閉じるには、**..** をクリックし、**エディタを閉じる** をクリックします。 

    保存を求められたら、**保存** をクリックすると、エディタが閉じます。 

    > **注意**:  **CTRL+S** を使っていつでも保存でき、 **CTRL+Q** を押してエディターを閉じます。

1. この実習ラボに必要なリソースを作成するには、次のコマンドを入力します。

    ```bash
    ./lab05-setup.azcli
    ```

    これは、実行するのに数分かかります。各ステップが完了すると、JSON 出力が表示されます。

    スクリプトが完了したら、ラボを続行できます。

### 演習 2: DPS での新しい個別登録 (対称キー) の作成

この演習では、_対称キー構成証明_を使用して、デバイス プロビジョニング サービス (DPS) 内のデバイスに対して新しい個別登録を作成します。

#### タスク 1: 登録の作成

1. 必要に応じて、Azure アカウントの認証情報を使用して Azure portal にログインします。

    複数の Azure アカウントをお持ちの場合は、このコースで使用するサブスクリプションに関連付けられているアカウントでログインしていることを確認してください。

1. 前のタスクで作成した **AZ-220** ダッシュボードが読み込まれています。

    IoT Hubと DPS の両方のリソースが表示されます。

1. リソース グループ タイルで、「**dps-az220-training-{your-id}**」 をクリックします。

1. 左側のメニューから、「**登録を管理します**」 をクリックします。

1. ペインの上部にある 「**+ 個別登録の追加**」 をクリックします。

1. 「**登録の追加**」 ブレードの 「**メカニズム**」 ドロップダウンで、「**対称キー**」 をクリックします。

    これにより、対称キー認証を使用する構成証明方法が設定されます。

1. 「メカニズム」 設定のすぐ下で、「**キーの自動生成**」 オプションがオンになっていることを確認します。

    これにより、DPS は デバイス登録時に **主キー** と **セカンダリキー** の両方の値を自動的に生成するように設定されます。オプションで、このオプションのチェックを外して、カスタムキーを手動で入力できます。

1. 「**登録 ID**」 フィールドで、DPS 内のデバイス登録に使用する登録 ID を指定するには 「**sensor-thl-1000**」 と入力します 

    既定では、登録からデバイスがプロビジョニングされるときに、登録 ID が IoT Hubデバイス ID として使用されます。これらの値を異なる値にする必要がある場合は、そのフィールドに必要な IoT Hubデバイス ID を入力します。

1. 「**IoT Hub デバイス ID**」 フィールドは空白のままにします。 

    このフィールドを空白のままにすると、IoT Hubは登録 ID をデバイス ID として使用します。選択できないフィールドにデフォルトのテキスト値が表示される場合は、このテキストはプレースホルダー テキストであり、入力値として扱われるので、心配しないでください。

1. 「**IoT Edge デバイス**」 フィールドを **False** のままにします。   

   新しいデバイスはエッジ デバイスではありません。IoT Edge デバイスの操作については、このコースの後半で説明します。

1. 「**デバイスをハブに割り当てる方法を選択してください**」 フィールドを 「**加重が均等に分布**」 に設定したままにします。   

   登録に関連付けられている IoT Hubは 1 つしかないため、この設定は重要ではありません。  複数の分散ハブがある大規模な環境では、この設定によって、このデバイス登録を受信する IoT Hubの選択方法を制御できます。サポートされている割り当てポリシーは 4 つあります。

    * **最短待機時間**: デバイスは、デバイスに対する待ち時間が最も低いハブに基づいて IoT Hubにプロビジョニングされます。
    * **加重が均等に分布 (デフォルト)**: リンクされた IoT Hubは、デバイスがプロビジョニングされている可能性が同じになります。これがデフォルトの設定です。デバイスを 1 つの IoT Hubのみにプロビジョニングする場合、この設定を維持できます。 
    * **静的な構成**: 登録リストは、デバイス プロビジョニング サービス レベルの割り当てポリシーよりも優先されます。
    * **カスタム (Azure 関数を使用)**: デバイス プロビジョニング サービスは、デバイスと登録に関するすべての関連情報を提供する Azure 関数コードを呼び出します。関数コードが実行され、デバイスのプロビジョニングに使用される IoT Hub情報が返されます。

1. 「**このデバイスを割り当てることができる IoT Hubを選択する**」 ドロップダウンで 作成した **iot-az220-training-{your-id}** IoT Hubが指定されていることに注意してください。

   このフィールドはデバイスに割り当てられる IoT Hubを指定するために使用されます。

1. 「**再プロビジョニングするときのデバイス データの処理方法を選択する**」 フィールドを、既定値の 「**データの再プロビジョニングと移行**」 に設定したままにします。

    このフィールドによって、同じデバイス (同じ登録 ID で示される) が、少なくとも 1 回正常にプロビジョニングされた後に、プロビジョニング要求を後で送信する再プロビジョニング動作を高度に制御できます。次の 3 つのオプションがあります。

    * **データの再プロビジョニングと移行**: このポリシーは、新しい登録エントリの既定です。このポリシーは、登録エントリに関連付けられているデバイスが新しいプロビジョニング要求を送信するときにアクションを実行します。登録エントリの構成によっては、デバイスが別の IoT Hubに再割り当てされる場合があります。デバイスが IoT Hubを変更する場合は、最初の IoT Hubを使用したデバイスの登録が削除されます。その初期 IoT Hubからのすべてのデバイス状態の情報は、新しい IoT Hubに移行されます。
    * **再プロビジョニングして初期構成にリセットする**: このポリシーは、登録エントリに関連付けられているデバイスが新しいプロビジョニング要求を送信するときにアクションを実行します。登録エントリの構成によっては、デバイスが別の IoT Hubに再割り当てされる場合があります。デバイスが IoT Hubを変更する場合は、最初の IoT Hubを使用したデバイスの登録が削除されます。デバイスがプロビジョニングされたときにプロビジョニング サービス インスタンスが受信した初期構成データが、新しい IoT Hubに提供されます。
    * **再プロビジョニングを行わない**: デバイスが別のハブに再割り当てされることはありません。このポリシーは、下位互換性を管理するために用意されています。

1. 「**初期デバイス ツイン状態**」 フィールドで、`properties.desired` JSON オブジェクトを変更して、`telemetryDelay` という名前のプロパティを値 `2` で指定します。 

    最終的な JSON は次のようになります。

    ```json
    {
        "tags": {},
        "properties": {
            "desired": {
                "telemetryDelay": "2"
            }
        }
    }
    ```

    このフィールドには、デバイスの必要なプロパティの初期構成を表す JSON データが含まれます。入力したデータは、センサー テレメトリの読み取りと IoT Hubへのイベントの送信の遅延時間を設定するためにデバイスによって使用されます。

1. 「**エントリの有効化**」 フィールドは 「**有効**」 に設定したままにします。   

    一般的に、新しい登録エントリを有効にし、有効を維持する必要があります。

1. 「**登録の追加**」 ブレードの上部にある 「**保存**」 をクリック します。

#### タスク 2: 登録を確認して認証キーを取得する

1. 「**登録を管理します**」 ブレードで、個別のデバイス登録の一覧を表示するには、「**個々の登録**」 をクリックします。   

1. 「個々の登録」 で 「**sensor-thl-1000**」 をクリックします。

    これにより、作成した個々の登録の登録詳細を表示できます。

1. 「**認証の種類**」 セクションを見つけ、 **メカニズム**が**対称キー**に設定されていることに注意してください。     

1. このデバイスの登録の**主キー**と **セカンダリキー**の値をコピーし (この目的のために各テキスト ボックスの右側にボタンがあります)、後で参照できるように保存します。

    これらは、デバイスがサービスで認証するための認証キーです。

1. **初期デバイス ツインの状態**を見つけ、デバイス ツインの Desired State の JSON に、`telemetryDelay` プロパティが値 `2` に設定されていることに注意してください。 

1. sensor-thl-1000の登録の詳細ブレードを閉じます。

### 演習 3: シミュレートされたデバイスの構成

この演習では、前の単元で作成した個々の登録を使用して Azure IoT に接続するように、C# で記述されたシミュレートされたデバイスを構成します。また、Azure IoT Hub 内のデバイス ツインに基づくデバイス構成を読み取って更新するコードをシミュレートされたデバイスに追加します。

この演習で作成するシミュレートされたデバイスは、出荷コンテナーやボックス内に配置される IoT デバイスを表し、Contoso 製品の転送中の監視に使用されます。Azure IoT Hub に送信されるデバイスからのセンサー テレメトリには、コンテナーの温度、湿度、圧力、緯度/経度座標が含まれています。デバイスは、全体的な資産追跡ソリューションの一部です。

これは、シミュレートされたデバイスが Azure に接続された以前のラボとは異なります。そのラボでは認証に共有アクセス キーを使用してデバイスのプロビジョニングを必要とせず、プロビジョニング管理の利点 (デバイス ツインなど) も提供しないためです。また、共有キーのかなり大希望な配布と管理が必要です。  このラボでは、デバイス プロビジョニング サービスを使用して一意のデバイスをプロビジョニングします。

#### タスク 1: シミュレートされたデバイスを作成する

1. **dps-az220-training-{your-id}** ブレードで、「**概要**」 ペインに移動します。   

1. ブレードの右上で、ID スコープに割り当てられた値の上にマウス ポインタを置き、「**クリップボードにコピー**」 をクリックします。 

    この値はまもなく使用されるため、クリップボードを使用できない場合は値をメモしておきます。大文字の "O" と数字の "0" を必ず区別してください。

    **ID スコープ**は、次の値と似ています。  `0ne0004E52G`

1. **Visual Studio Code** を使用して、「ContainerDevice」フォルダーを開きます。

    ここでも、これはラボ 3 で開発環境を設定するときにダウンロードしたラボ リソース ファイルを指しています。フォルダ パスは次のとおりです。

    * Allfiles
      * Labs
          * 05-Individual Enrollment of a Device in DPS
            * Starter
               * ContainerDevice

1. 「**表示**」 メニューの 「**ターミナル**」 をクリックします。   

    選択したターミナル シェルが Windows コマンド プロンプトであることを確認します。

1. コマンド ラインを使用して NuGet パッケージのすべてのアプリケーションを復元するには、ターミナル ビューのコマンド プロンプトで次のコマンドを入力します。

    ```cmd/sh
    dotnet restore
    ```

1. 「**Program.cs**」 をクリックします。 

1. コード エディターで、プログラム クラスの上部にある `dpsIdScope` 変数を見つけて、Azure portal のデバイス プロビジョニング サービスからコピーした ID スコープ値を使用して割り当てられた値を更新します。

    > **注意**: ID スコープの値が使用できない場合は、DPS サービスの 「概要」 ブレード (Azure portal) で確認できます。

1. `registrationId` 変数を見つけて、値を **sensor-thl-1000** に置き換えます

    この変数は、デバイス プロビジョニング サービスで作成した個々の加入契約の **登録 ID** 値を表します。

1. `individualEnrollmentPrimaryKey` と `individualEnrollmentSecondaryKey` 変数を見つけて、それらの値をシミュレートされたデバイスの個々の登録を構成するときに保存した**主キー**と **セカンダリキー**の値に置き換えます。

    > **注意**: これらのキー値が利用できない場合は、次のように Azure portal からコピーできます。
    >
    > 「**登録を管理します**」 ブレードを開き、「**個々の登録**」 をクリックし、「**sensor-thl-1000**」 をクリックします。値をコピーし、上記の手順に示すように貼り付けます。

#### タスク 2: プロビジョニングコードの追加

このタスクでは、DPSを介してデバイスをプロビジョニングするコードを実装し、IoT Hubへの接続に使用できるDeviceClientインスタンスを作成します。

1. Program.csファイル内のコードを確認します。

    ContainerDeviceアプリケーションの全体的なレイアウトは、ラボ4で作成したCaveDeviceアプリケーションに似ています。両方のアプリケーションに以下が含まれていることに注意してください。

    * Usingステートメント
    * 名前空間の定義
        * Programクラス - Azure IoTへの接続とテレメトリの送信を担当します
        * EnvironmentSensorクラス - センサーデータの生成を担当

1. コードエディターで、`// INSERT Main method below here` を見つけます。

1. シミュレートされたデバイスアプリケーションのMainメソッドを作成するために、次のコードを入力します。

    ```csharp
    public static async Task Main(string[] args)
    {

        using (var security = new SecurityProviderSymmetricKey(registrationId,
                                                                individualEnrollmentPrimaryKey,
                                                                individualEnrollmentSecondaryKey))
        using (var transport = new ProvisioningTransportHandlerAmqp(TransportFallbackType.TcpOnly))
        {
            ProvisioningDeviceClient provClient =
                ProvisioningDeviceClient.Create(GlobalDeviceEndpoint, dpsIdScope, security, transport);

            using (deviceClient = await ProvisionDevice(provClient, security))
            {
                await deviceClient.OpenAsync().ConfigureAwait(false);

                // INSERT Setup OnDesiredPropertyChanged Event Handling below here

                // INSERT Load Device Twin Properties below here

                // Start reading and sending device telemetry
                Console.WriteLine("Start reading and sending device telemetry...");
                await SendDeviceToCloudMessagesAsync(deviceClient);

                await deviceClient.CloseAsync().ConfigureAwait(false);
            }
        }
    }
    ```

    このアプリケーションのMainメソッドは、前のラボで作成したCaveDeviceアプリケーションのMainメソッドと同様の目的を果たしますが、少し複雑です。 CaveDeviceアプリで、デバイス接続文字列を使用してIoT Hubに直接接続しました。今回は、最初にデバイスをプロビジョニングする（または、以降の接続では、デバイスがまだプロビジョニングされていることを確認する）必要があります。次に、適切なIoT Hub接続を取得します詳細。

    DPSに接続するには、**dpsScopeId**と**GlobalDeviceEndpoint**（変数で定義）が必要なだけでなく、次のものも指定する必要があります。

    * **security** - 登録の認証に使用される方法。以前に対称キーを使用するように個々の登録を構成したため、**SecurityProviderSymmetricKey**が論理的な選択です。ご想像のとおり、X.509およびTPMをサポートするプロバイダーもあります。

    * **transport** - プロビジョニングされたデバイスで使用されるトランスポートプロトコル。この例では、AMQPハンドラーが選択されました（**ProvisioningTransportHandlerAmqp**）。もちろん、HTTPおよびMQTTハンドラーも使用できます。

    **security**および**transport**変数が入力されたら、**ProvisioningDeviceClient**のインスタンスを作成します。このインスタンスを使用してデバイスを登録し、**ProvisionDevice**メソッドで**DeviceClient**を作成します。

    **Main**メソッドの残りの部分は、**CaveDevice**で行ったのとは少し異なる方法でデバイスクライアントを使用します - 今回は、デバイス接続を明示的に開いて、アプリがデバイスツインを使用できるようにします（詳細については、次の演習）、次に**SendDeviceToCloudMessagesAsync**メソッドを呼び出してテレメトリの送信を開始します。

    **SendDeviceToCloudMessagesAsync**メソッドは、**CaveDevice**アプリケーションで作成したものとよく似ています。 **EnvironmentSensor**クラスのインスタンスを作成し（これも圧力と位置のデータを返します）、メッセージを作成して送信します。メソッドループ内の固定遅延ではなく、**telemetryDelay**変数を使用して遅延が計算されることに注意してください： `await Task.Delay（telemetryDelay * 1000）;`。時間が許せば、自分自身をより深く見て、これを以前のラボで使用したクラスと比較してください。

    最後に、**Main**メソッドに戻り、デバイスクライアントを閉じます。
    
    > **Information**：**ProvisioningDeviceClient**のドキュメントは[こちら]（https://docs.microsoft.com/en-us/dotnet/api/microsoft.azure.devices.provisioning.client.provisioningdeviceclient?view=azure-dotnet） - そこから、他の関連クラスに簡単に移動できます。

1. `// INSERT ProvisionDevice method below here`コメントを見つけます。

1. **ProvisionDevice**メソッドを作成するために、次のコードを入力します。

    ```csharp
    private static async Task<DeviceClient> ProvisionDevice(ProvisioningDeviceClient provisioningDeviceClient, SecurityProviderSymmetricKey security)
    {
        var result = await provisioningDeviceClient.RegisterAsync().ConfigureAwait(false);
        Console.WriteLine($"ProvisioningClient AssignedHub: {result.AssignedHub}; DeviceID: {result.DeviceId}");
        if (result.Status != ProvisioningRegistrationStatusType.Assigned)
        {
            throw new Exception($"DeviceRegistrationResult.Status is NOT 'Assigned'");
        }

        var auth = new DeviceAuthenticationWithRegistrySymmetricKey(
            result.DeviceId,
            security.GetPrimaryKey());

        return DeviceClient.Create(result.AssignedHub, auth, TransportType.Amqp);
    }
    ```
    
    ご覧のとおり、このメソッドは、以前に作成したプロビジョニングデバイスクライアントとセキュリティインスタンスを受け取ります。 `provisioningDeviceClient.RegisterAsync（）`が呼び出され、**DeviceRegistrationResult**インスタンスが返されます。この結果には、**DeviceId**、**AssignedHub**、および**Status**を含む多数のプロパティが含まれています。

    > **Information**：**DeviceRegistrationResult**プロパティの詳細については[こちら](https://docs.microsoft.com/en-us/dotnet/api/microsoft.azure.devices.provisioning.client.deviceregistrationresult?view=azure-dotnet)を参照。

    次に、メソッドはプロビジョニングステータスが設定されていることを確認し、デバイスが*割り当てられていない*場合に例外をスローします。ここで考えられるその他の結果には、*Unassigned*、*Assigning*、*Failed*、*Disabled*があります。

    * **Program.ProvisionDevice**メソッドには、DPSを介してデバイスを登録するためのロジックが含まれています。
    * **Program.SendDeviceToCloudMessagesAsync**メソッドは、テレメトリをデバイスからクラウドへのメッセージとしてAzure IoT Hubに送信します。
    * **EnvironmentSensor**クラスには、温度、湿度、圧力、緯度、経度のセンサー読み取り値をシミュレートするためのロジックが含まれています。

1. **SendDeviceToCloudMessagesAsync**メソッドを見つけます。

1. **SendDeviceToCloudMessagesAsync**メソッドの下部で、 `Task.Delay（）`の呼び出しに注目してください。

    `Task.Delay（）`は、次のテレメトリメッセージを作成して送信する前に、しばらくの間「while」ループを「一時停止」するために使用されます。 **telemetryDelay**変数は、次のテレメトリメッセージを送信するまでに待機する秒数を定義するために使用されます。 Contosoでは、遅延時間を構成可能にする必要があります。

1. **Program**クラスの上部で、**telemetryDelay**変数宣言を見つけます。

    遅延のデフォルト値が**1**秒に設定されていることに注意してください。次のステップは、デバイスツイン値を使用して遅延時間を制御するコードを統合することです。


#### タスク 2: デバイス ツインのプロパティを統合する

デバイスで (Azure IoT Hub から) デバイス ツインのプロパティを使用するには、デバイス ツインのプロパティにアクセスして適用するコードを作成する必要があります。この場合、シミュレートされたデバイス コードを更新してデバイス ツインの必要なプロパティを読み取ってから、その値を `_telemetryDelay` 変数に割り当てます。また、現在デバイスに実装されている遅延値を示すために、デバイス ツインの報告されるプロパティを更新します。

1. Visual Studio Code エディターで、`Main` メソッドを見つけます。

    デバイス ツイン プロパティの統合を開始するには、デバイス ツイン プロパティが更新されたときに、シミュレートされたデバイスに通知を有効にするコードが必要です。

    これを実現するには、`DeviceClient.SetDesiredPropertyUpdateCallbackAsync` メソッドを使用し、`OnDesiredPropertyChanged` イベントを作成することによってイベント ハンドラーを設定します。

1. コードを確認してから、`// INSERT Setup OnDesiredPropertyChanged Event Handling below here` コメントを見つけます。

1. OnDesiredPropertyChanged イベントの DeviceClient を設定するために、次のコードを入力します。

    ```csharp
    await deviceClient.SetDesiredPropertyUpdateCallbackAsync(OnDesiredPropertyChanged, null).ConfigureAwait(false);
    ```
    
    **SetDesiredPropertyUpdateCallbackAsync**メソッドを使用して、**DesiredPropertyUpdateCallback**イベントハンドラーをセットアップし、デバイスツインの必要なプロパティの変更を受信します。 このコードは、デバイスツインのプロパティ変更イベントを受信したときに**OnDesiredPropertyChanged**という名前のメソッドを呼び出すように**deviceClient**を構成します。

     これで、**SetDesiredPropertyUpdateCallbackAsync**メソッドを使用してイベントハンドラーを設定できたので、それが呼び出す**OnDesiredPropertyChanged**メソッドを作成する必要があります。

1. `// INSERT OnDesiredPropertyChanged method below here`コメントを見つけます。

1. **OnDesiredPropertyChanged**メソッドを作成するために、次のコードを入力します。

    ```csharp
    private static async Task OnDesiredPropertyChanged(TwinCollection desiredProperties, object userContext)
    {
        Console.WriteLine("Desired Twin Property Changed:");
        Console.WriteLine($"{desiredProperties.ToJson()}");

        // Read the desired Twin Properties
        if (desiredProperties.Contains("telemetryDelay"))
        {
            string desiredTelemetryDelay = desiredProperties["telemetryDelay"];
            if (desiredTelemetryDelay != null)
            {
                telemetryDelay = int.Parse(desiredTelemetryDelay);
            }
            // if desired telemetryDelay is null or unspecified, don't change it
        }

        // Report Twin Properties
        var reportedProperties = new TwinCollection();
        reportedProperties["telemetryDelay"] = telemetryDelay.ToString();
        await deviceClient.UpdateReportedPropertiesAsync(reportedProperties).ConfigureAwait(false);
        Console.WriteLine("Reported Twin Properties:");
        Console.WriteLine($"{reportedProperties.ToJson()}");
    }
    ```

    **OnDesiredPropertyChanged**イベントハンドラーは、**TwinCollection**タイプの**desiredProperties**パラメーターを受け入れることに注意してください。

    **desiredProperties**パラメーターの値に**telemetryDelay**（デバイスツインの目的のプロパティ）が含まれている場合、コードはデバイスツインプロパティの値を**telemetryDelay**変数に割り当てます。 **SendDeviceToCloudMessagesAsync**メソッドには、**telemetryDelay **変数を使用してIoTハブに送信されるメッセージ間の遅延時間を設定する**Task.Delay**呼び出しが含まれていることを思い出してください。

    次のコードブロックを使用して、デバイスの現在の状態をAzure IoT Hubに報告します。このコードは、**DeviceClient.UpdateReportedPropertiesAsync**メソッドを呼び出し、デバイスプロパティの現在の状態を含む**TwinCollection**を渡します。これは、デバイスがデバイスツインの必要なプロパティ変更イベントを受信したことをIoT Hubに報告し、それに応じて構成を更新した方法です。これは、目的のプロパティのエコーではなく、プロパティの現在の設定を報告することに注意してください。デバイスから送信されたレポートされたプロパティがデバイスが受信した望ましい状態と異なる場合、IoT Hubはデバイスの状態を反映する正確なデバイスツインを維持します。

    デバイスがAzure IoT Hubからデバイスツインの必要なプロパティへの更新を受信できるようになったので、デバイスの起動時に初期セットアップを構成するようにコード化する必要もあります。これを行うには、デバイスは、Azure IoT Hubから現在のデバイスツインの必要なプロパティを読み込み、それに従って構成する必要があります。

1. **Main**メソッドで、 `// INSERT Load Device Twin Properties below here`コメントを見つけます。

1. デバイスツインの必要なプロパティを読み取り、デバイスの起動時に一致するようにデバイスを構成するには、次のコードを入力します。

    ```csharp
    var twin = await deviceClient.GetTwinAsync().ConfigureAwait(false);
    await OnDesiredPropertyChanged(twin.Properties.Desired, null);
    ```

    このコードは、 `DeviceTwin.GetTwinAsync`メソッドを呼び出して、シミュレートされたデバイスのデバイスツインを取得します。次に、 `Properties.Desired`プロパティオブジェクトにアクセスしてデバイスの現在のDesired Stateを取得し、それを**OnDesiredPropertyChanged**メソッドに渡して、シミュレートされたデバイスの**telemetryDelay**変数を構成します。

    このコードは、_OnDesiredPropertyChanged_イベントを処理するためにすでに作成されている**OnDesiredPropertyChanged**メソッドを再利用していることに注意してください。これにより、デバイスツインの望ましい状態プロパティを読み取り、起動時にデバイスを1か所で構成するコードを維持できます。結果のコードは、保守が簡単で簡単です。

1. Visual Studio Code [**ファイル**] メニューで、[**保存**]をクリックします。

    シミュレートされたデバイスは、Azure IoT Hubのデバイスツインプロパティを使用して、テレメトリメッセージ間の遅延を設定します。

### 演習 4: シミュレートされたデバイスのテスト

この演習では、シミュレートされたデバイスを実行し、センサー テレメトリを Azure IoT Hub に送信することを確認します。また、Azure IoT Hub 内のシミュレートされたデバイスのデバイス ツインを更新することで、テレメトリが Azure IoT Hub に送信される遅延も更新します。

#### タスク 1: デバイスをビルドおよび実行する

1. Visual Studio Code でコード プロジェクトを開いていることを確認します。

1. トップ メニューの 「**表示**」 をクリックし、「**ターミナル**」 をクリックします。

1. ターミナル ペインで、コマンド プロンプトに `Program.cs` ファイルのディレクトリ パスが表示されていることを確認します。

1. コマンド プロンプトで、シミュレートされたデバイス アプリケーションをビルドして実行するには、次のコマンドを入力します。

    ```cmd/sh
    dotnet run
    ```

    > **注意**: シミュレートされたデバイス アプリケーションを実行すると、まず、その状態に関する詳細がコンソール (ターミナル ペイン) に書き込まれます。

1. `Desired Twin Property Changed:`行の後の JSON 出力には、デバイスの `telemetryDelay` に必要な値が含まれていることに注意してください。

    ターミナル ペインを上にスクロールして、出力を確認できます。次のようになるはずです。

    ```bash
    ProvisioningClient AssignedHub: iot-az220-training-{your-id}.azure-devices.net; DeviceID: sensor-thl-1000
    Desired Twin Property Changed:
    {"telemetryDelay":"2","$version":1}
    Reported Twin Properties:
    {"telemetryDelay":"2"}
    Start reading and sending device telemetry...
    ```

1. シミュレートされたデバイス アプリケーションが Azure IoT Hub にテレメトリ イベントを送信し始めることに注意してください。

    テレメトリ イベントには、`temperature`、`humidity`、`pressure`、`latitude`、および`longitude`の値が含まれ、次のようになるはずです。

    ```bash
    11/6/2019 6:38:55 PM > Sending message: {"temperature":25.59094770373355,"humidity":71.17629229611545,"pressure":1019.9274696347665,"latitude":39.82133964767944,"longitude":-98.18181981142438}
    11/6/2019 6:38:57 PM > Sending message: {"temperature":24.68789062681044,"humidity":71.52098010830628,"pressure":1022.6521258267584,"latitude":40.05846882452387,"longitude":-98.08765031156229}
    11/6/2019 6:38:59 PM > Sending message: {"temperature":28.087463226675737,"humidity":74.76071353757787,"pressure":1017.614206096327,"latitude":40.269273772972454,"longitude":-98.28354453319591}
    11/6/2019 6:39:01 PM > Sending message: {"temperature":23.575667940813894,"humidity":77.66409506912534,"pressure":1017.0118147748344,"latitude":40.21020096551372,"longitude":-98.48636739129239}
    ```

    テレメトリの読み取り値のタイムスタンプの違いに注目してください。テレメトリ メッセージ間の遅延は、ソース コードでの規定値 `1` 秒ではなく、デバイス ツインを介して構成されるように `2` 秒となる必要があります。

1. シミュレートされたデバイス アプリを実行したままにします。

    デバイス コードが次のアクティビティで期待どおりに動作していることを確認します。

#### タスク 2: Azure IoT Hub に送信されるテレメトリ ストリームを確認する

このタスクでは、Azure CLI を使用して、シミュレートされたデバイスから送信されたテレメトリが Azure IoT Hub によって受信されていることを確認します。

1. ブラウザーを使用して [Azure Cloud Shell](https://shell.azure.com/) を開き、このコースで使用している Azure サブスクリプションでログインします。

1. Azure Cloud Shell で、次のコマンドを入力します。

    ```cmd/sh
    az iot hub monitor-events --hub-name {IoTHubName} --device-id sensor-thl-1000
    ```

    _**{IoTHubName}** プレースホルダーを Azure IoT Hub の名前に置き換えてください。_

1. IoT Hubが sensor-thl-1000 デバイスからテレメトリ メッセージを受信していることに注意してください。

    次のタスクのために、シミュレートされたデバイス アプリケーションを実行したままにします。

#### タスク 3: ツインを使用してデバイスの構成を変更する

シミュレートされたデバイスを実行すると、Azure IoT Hub 内でデバイス ツインの必要な状態を編集することで、`telemetryDelay` 構成を更新できます。これは、Azure portal 内の Azure IoT Hub でデバイスを構成することで実現できます。

1. **Azure portal** を開き、作成済みの**IoT Hub** リソースに移動します。

1. IoT Hub ブレードの左側にある 「**エクスプローラ**」 セクションで、「**IoT デバイス**」 をクリックします。   

1. IoT デバイスの一覧で、「**sensor-thl-1000**」 をクリックします。 

    > **重要**: このラボで使用しているデバイスを選択していることを確認します。

1. デバイス ブレードのブレードの上部にある 「**デバイス ツイン**」 をクリックします。

    「**デバイス ツイン**」 ブレードでは、デバイス ツインの完全な JSON が編集者に提供されます。これにより、Azure portal 内でデバイス ツインの状態を直接表示および/または編集できます。

1. `properties.desired` オブジェクトの JSON の場所を見つけます。

    これには、デバイスの望ましい状態が含まれています。`telemetryDelay` プロパティは既に存在し、DPS の個別登録に基づいてデバイスがプロビジョニングされたときに構成されたとおりに `"2"`に設定されていることに注意してください。

1. 目的の `telemetryDelay` プロパティに割り当てられた値を更新するには、値を`"5"`に変更します。

    値には引用符 ("") が含まれます。

1. ブレードの上部にある 「**保存**」 をクリックします。

    `OnDesiredPropertyChanged` イベントは、シミュレートされたデバイスのコード内で自動的にトリガーされ、デバイスは、デバイス ツインの必要な状態への変更を反映するように、その構成を更新します。

1. シミュレートされたデバイス アプリケーションを実行するために使用している Visual Studio Code ウィンドウに切り替えます。

1. Visual Studio Code で、ターミナル ウィンドウの下部までスクロールします。

1. デバイスが、デバイス ツインプロパティの変更を認識することに注意してください。

    出力には、新しい必要な `telemetryDelay` プロパティ値の JSON と共に、`Desired Twin Property Changed`というメッセージが表示されます。デバイスは、デバイス ツインの必要な状態の新しい構成を選択すると、現在構成されているように、5 秒ごとにセンサー テレメトリの送信を開始するよう自動的に更新されます。
    
    ```bash
    Desired Twin Property Changed:
    {"telemetryDelay":"5","$version":2}
    Reported Twin Properties:
    {"telemetryDelay":"5"}
    4/21/2020 1:20:16 PM > Sending message: {"temperature":34.417625961088405,"humidity":74.12403526442313,"pressure":1023.7792049974805,"latitude":40.172799921919186,"longitude":-98.28591913777421}
    4/21/2020 1:20:22 PM > Sending message: {"temperature":20.963297521678403,"humidity":68.36916032636965,"pressure":1023.7596862048422,"latitude":39.83252821949164,"longitude":-98.31669969393461}
    ```

1. Azure Cloud Shell で Azure CLI コマンドを実行しているブラウザー ページに切り替えます。

    `az iot hub monitor-events` コマンドがまだ実行されていることを確認します。実行されていない場合は、コマンドを再度開始します。

1. Azure IoT Hub に送信されたテレメトリ イベントが、新しい間隔 5 秒で受信されていることに注意してください。

1. **Ctrl+C** を使用して `az`  コマンドとシミュレートされたデバイス  アプリケーションの両方を停止します。 

1. Azure portal で、「**デバイス ツイン**」 ブレードに移動します。

1. Azure portal の 「sensor-thl-1000」 ブレードで、「**デバイス ツイン**」 をクリックします。

1. 今回は、`properties.reported` オブジェクトの JSON を探します。

    これには、デバイスによって報告された状態が含まれます。`telemetryDelay` プロパティもここに存在し、`5` に設定されていることに注意してください。  また、値が最後に更新された時点と、特定のレポート値が最後に更新された時刻を示す `$metadata` 値もあります。

1. もう一度**デバイス ツイン**ブレードを閉じます。

1. シミュレートされたデバイス ブレードを閉じ、IoT Hub ブレードを閉じます。

### 演習 5: デバイスのプロビジョニングを解除する

Contosoシナリオでは、出荷コンテナが最終目的地に到着すると、IoTデバイスがコンテナから削除され、Contosoの場所に戻されます。Contosoは、デバイスをテストしてインベントリに配置する前に、デバイスのプロビジョニングを解除する必要があります。将来的には、デバイスは同じIoTハブまたは異なる地域のIoTハブにプロビジョニングされる可能性があります。完全なデバイスのプロビジョニング解除は、IoTソリューション内のIoTデバイスのライフサイクルにおける重要なステップです。

この演習では、デバイスプロビジョニングサービス（DPS）とAzure IoTHubの両方からデバイスのプロビジョニングを解除するために必要なタスクを実行します。IoTデバイスをAzureIoTソリューションから完全にプロビジョニング解除するには、これらのサービスの両方から削除する必要があります。

#### タスク 1: デバイスをDPSから登録解除する

1. 必要に応じて、Azure アカウントの認証情報を使用して Azure portal にログインします。

    複数の Azure アカウントをお持ちの場合は、このコースで使用するサブスクリプションに関連付けられているアカウントでログインしていることを確認してください。

1.  **「dps-az220-training-{your-id}」** を開きます。 

1. デバイス プロビジョニング サービス ブレードの左側のメニューで、 **「登録を管理します」** をクリックします。  

1. 「登録を管理します」 ブレードで、個々のデバイス登録の一覧を表示するには、「**個々の登録**」をクリックします。  

1. sensor-thl-1000の左側にあるチェックボックスをクリックします。

    デバイス登録を開くのではなく、デバイスを選択するだけです。

1. ブレードの上部で、「**削除**」 をクリックします。

    > **注意**: DPS から個々の登録を削除すると、登録が恒久的に削除されます。一時的に登録を無効にするには、個々の登録の 「**登録の詳細**」 で 「**エントリを有効にする」**設定を 「**無効にする**」 に設定します。

1. 「**登録の削除**」 プロンプトで 「**はい**」 をクリックします。

    これで、個別登録がデバイス プロビジョニング サービス (DPS) から削除されます。デバイスの廃棄を完了するには、シミュレートされたデバイスの **デバイス ID** も **Azure IoT Hub** サービスから削除する必要があります。 

#### タスク 2: IoT Hub からデバイスを削除する

1. Azure portal で、ダッシュボードにもう一度アクセスしてください。

1. 「**iot-az220-training-{your-id}**」 に移動します。 

1. IoT Hub ブレードの左側にある**エクスプローラ** セクションで、「**IoT デバイス**」 をクリックします。

1. IoT デバイスのリスト内で、sensor-thl-1000 デバイス ID の左側にあるチェックボックスをオンにします。

    > **重要**: このラボで使用したシミュレートされたデバイスを表すデバイスを選択していることを確認します。

1. ブレードの上部で、「**削除**」 をクリックします。

1. 「**選択したデバイスを削除しますか**」 のプロンプトで、「**はい**」 をクリックします。   

    > **注意**:  IoT Hubからデバイス ID を削除すると、デバイスの登録が完全に削除されます。デバイスが IoT Hub に接続できないように一時的に無効にするには、 デバイスのプロパティ内で 「**IoT Hub への接続を有効にする**」 を 「**無効にする**」 に設定します。

これで、デバイス登録がデバイス プロビジョニング サービスから削除され、一致するデバイス ID が Azure IoT Hub から削除されたので、シミュレートされたデバイスがソリューションから完全に廃止されました。
