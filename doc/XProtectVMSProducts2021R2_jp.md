# XProtect VMS Products 2021 R2 のクラスタ化

## 評価環境
- Windows Server 2019
- XProtect VMS Product 2021 R2
- SQL Server 2019
- CLUSTERPRO X 4.3 
  ```
  +----------------------------------+ 
  | server1                          | 
  | C:                               |   +--------------------+
  |   - Windows Server 2019          |   | X:                 |
  |   - XProtect VMS Product 2021 R2 +---+   - Database files |
  |   - SQL Server 2019              |   |                    |
  |   - CLUSTERPRO X 4.3             |   +--+-----------------+
  +----------------------------------+      |
                                            | Mirroring
  +----------------------------------+      |
  | server2                          |      |
  | C:                               |   +--V-----------------+
  |   - Windows Server 2019          |   | X:                 |
  |   - XProtect VMS Product 2021 R2 +---+   - Database files |
  |   - SQL Server 2019              |   |                    |
  |   - CLUSTERPRO X 4.3             |   +--------------------+
  +----------------------------------+    
  ```

## SQL Server のクラスタ化
1. [ソフトウェア構築ガイド](https://jpn.nec.com/clusterpro/clpx/guide.html?#anc-win)に掲載されている手順に従い、SQL Server のクラスタを構築してください。
1. クラスタ構築後、SQL Server に接続できることを確認してください。
   1. server1 でフェイルオーバグループが起動していることを確認し、以下のコマンドを実行してください。
      1. SQL Server に接続してください。
         ```bat
         sqlcmd -U sa -P <password> -S <IP address>
         ```
      1. SQL Server のクラスタ化の際に作成したデータベースの状態を取得してください。
         ```sql
         SELECT name, physical_name AS CurrentLocation, state_desc
         FROM sys.master_files
         WHERE database_id = DB_ID('TESTDB');
         ```
      1. 上記 SQL 文を実行後、以下のような結果が得られます。ミラーディスク上に mdf, ldf ファイルがあること、ONLINE であることを確認してください。
         ```
         name           CurrentLocation              state_desc
         -------------- ---------------------------- -----------
         TESTDB_Data    X:\sql\data\TESTDB_Data.mdf  ONLINE
         TESTDB_Log     X:\sql\data\TESTDB_Log.ldf   ONLINE
         
         (2 rows affected)
         ```
      1. フェイルオーバグループを server2 に移動し、上記 SQL 文を実行し、同様にミラーディスク上に mdf, ldf ファイルがあること、ONLINE であることを確認してください。
1. クラスタのリソースが以下のように設定されていることを確認してください。
   |深度|リソース|用途|
   |-|-|-|
   |0|フローティング IP リソース|一意な IP アドレス|
   |1|ミラーディスクリソース|データベースファイルを保存|
   |2|サービスリソース|SQL Server サービスの制御|
   |3|スクリプトリソース|データベースの制御|
1. 以降の手順で TESTDB をクラスタの管理対象外にするため、SQL Server 監視リソースで TESTDB を監視している場合には、SQL Server 監視リソースを削除してください。

## XProtect VMS Products のインストール
1. server1 に XProtect VMS Products を Single Computer でインストールしてください。
1. インストール完了後、SQL Server に接続し、以下の mdf, ldf ファイルが作成されていることを確認してください。
   ```sql
   SELECT name, physical_name AS CurrentLocation, state_desc
   FROM sys.master_files
   WHERE database_id = DB_ID('Surveillance') or database_id = DB_ID('Surveillance_IDP') or database_id = DB_ID('SurveillanceLogServerV2');
   ```
   ```
   name                          CurrentLocation                                                                                        state_desc
   ----------------------------- ------------------------------------------------------------------------------------------------------ ------------------------------------------------------------
   Surveillance                  C:\Program Files\Microsoft SQL Server\MSSQL15.MSSQLSERVER\MSSQL\DATA\Surveillance.mdf                  ONLINE
   Surveillance_log              C:\Program Files\Microsoft SQL Server\MSSQL15.MSSQLSERVER\MSSQL\DATA\Surveillance_log.ldf              ONLINE
   Surveillance_IDP              C:\Program Files\Microsoft SQL Server\MSSQL15.MSSQLSERVER\MSSQL\DATA\Surveillance_IDP.mdf              ONLINE
   Surveillance_IDP_log          C:\Program Files\Microsoft SQL Server\MSSQL15.MSSQLSERVER\MSSQL\DATA\Surveillance_IDP_log.ldf          ONLINE
   SurveillanceLogServerV2       C:\Program Files\Microsoft SQL Server\MSSQL15.MSSQLSERVER\MSSQL\DATA\SurveillanceLogServerV2.mdf       ONLINE
   SurveillanceLogServerV2_log   C:\Program Files\Microsoft SQL Server\MSSQL15.MSSQLSERVER\MSSQL\DATA\SurveillanceLogServerV2_log.ldf   ONLINE
   
   (6 rows affected)
   ```
1. mdf, ldf ファイルをミラーディスク上に移動します。
   - 参考: https://docs.microsoft.com/ja-jp/sql/relational-databases/databases/move-user-databases?view=sql-server-ver15
   1. 以下の SQL 文を実行し、mdf, ldf ファイルのパスを、ミラーディスク上のディレクトリ (e.g. X:\sql\data) に変更してください。
      ```sql
      ALTER DATABASE Surveillance MODIFY FILE ( NAME = Surveillance, FILENAME = 'X:\sql\data\Surveillance.mdf' );
      ALTER DATABASE Surveillance MODIFY FILE ( NAME = Surveillance_log, FILENAME = 'X:\sql\data\Surveillance_log.ldf' );
      ALTER DATABASE Surveillance_IDP MODIFY FILE ( NAME = Surveillance_IDP, FILENAME = 'X:\sql\data\Surveillance_IDP.mdf' );
      ALTER DATABASE Surveillance_IDP MODIFY FILE ( NAME = Surveillance_IDP_log, FILENAME = 'X:\sql\data\Surveillance_IDP_log.ldf' );
      ALTER DATABASE SurveillanceLogServerV2 MODIFY FILE ( NAME = SurveillanceLogServerV2, FILENAME = 'X:\sql\data\SurveillanceLogServerV2.mdf' );
      ALTER DATABASE SurveillanceLogServerV2 MODIFY FILE ( NAME = SurveillanceLogServerV2_log, FILENAME = 'X:\sql\data\SurveillanceLogServerV2_log.ldf' );
      ALTER DATABASE Surveillance SET OFFLINE;
      ALTER DATABASE Surveillance_IDP SET OFFLINE;
      ALTER DATABASE SurveillanceLogServerV2 SET OFFLINE;
      ```
   1. 以下の mdf, ldf ファイルを Explorer などでミラーディスク上 (e.g. X:\sql\data) に、コピーしてください。
      ```
      Surveillance.mdf
      Surveillance_log.ldf
      Surveillance_IDP.mdf
      Surveillance_IDP_log.ldf
      SurveillanceLogServerV2.mdf
      SurveillanceLogServerV2_log.ldf
      ```
   1. Explorer などで上記 mdf, ldf に対して、NT Service\MSSQLSERVER にフルコントロールの権限を付与してください。また、icacls コマンドで付与することも可能です。
      ```bat
      icacls X:\sql\data /inheritance:e /grant "NT Service\MSSQLSERVER":(OI)(CI)F
      ```
1. 以下の SQL 文を実行し、全てのデータベースを ONLINE にしてください。
   ```sql
   ALTER DATABASE Surveillance SET ONLINE;
   ALTER DATABASE Surveillance_IDP SET ONLINE;
   ALTER DATABASE SurveillanceLogServerV2 SET ONLINE;
   ```
1. 全てのデータベースが ONLINE であることを確認してください。
   ```sql
   SELECT name, physical_name AS CurrentLocation, state_desc FROM sys.master_files
   WHERE database_id = DB_ID('Surveillance') or database_id = DB_ID('Surveillance_IDP') or database_id = DB_ID('SurveillanceLogServerV2');
   ```
1. 以下のコマンドを実行し、XProtect のサービスを停止してください。
   ```bat
   net stop "Milestone XProtect Data Collector Server"
   net stop "Milestone XProtect Log Server"
   net stop "Milestone XProtect Management Server"
   net stop "Milestone XProtect Mobile Server"
   net stop "Milestone XProtect Recording Server"
   net stop "MilestoneEventServerService"
   sc config "Milestone XProtect Data Collector Server" start= demand
   sc config "Milestone XProtect Log Server" start= demand
   sc config "Milestone XProtect Management Server" start= demand
   sc config "Milestone XProtect Mobile Server" start= demand
   sc config "Milestone XProtect Recording Server" start= demand
   sc config "MilestoneEventServerService" start= demand
   ```
1. 以下の SQL 文を実行し、データベースを OFFLINE にし、デタッチしてください。
   ```sql
   alter database [Surveillance] set offline with ROLLBACK IMMEDIATE
   exec sp_detach_db 'Surveillance',TRUE
   alter database [Surveillance_IDP] set offline with ROLLBACK IMMEDIATE
   exec sp_detach_db 'Surveillance_IDP',TRUE
   alter database [SurveillanceLogServerV2] set offline with ROLLBACK IMMEDIATE
   exec sp_detach_db 'SurveillanceLogServerV2',TRUE
   ```
1. Cluster WebUI または clpgrp コマンドでフェイルオーバグループを server2 に移動してください。
1. 以下のコマンドを実行し、データベースをアタッチし、ONLINE にしてください。
   ```sql
   create database [Surveillance] on
   (filename = 'X:\sql\data\Surveillance.mdf'),
   (filename = 'X:\sql\data\Surveillance_log.ldf')
   for attach
   create database [Surveillance_IDP] on
   (filename = 'X:\sql\data\Surveillance_IDP.mdf'),
   (filename = 'X:\sql\data\Surveillance_IDP_log.ldf')
   for attach
   create database [SurveillanceLogServerV2] on
   (filename = 'X:\sql\data\SurveillanceLogServerV2.mdf'),
   (filename = 'X:\sql\data\SurveillanceLogServerV2_log.ldf')
   for attach
   ```
1. XProtect VMS Products を server2 に Custom でインストールしてください。
   1. インストール時、[Select database] にて、[Use existing database] にチェックを入れてください。
1. インストール完了後、全てのデータベースが ONLINE であることを確認してください。
   ```sql
   SELECT name, physical_name AS CurrentLocation, state_desc FROM sys.master_files
   WHERE database_id = DB_ID('Surveillance') or database_id = DB_ID('Surveillance_IDP') or database_id = DB_ID('SurveillanceLogServerV2');
   ```
1. 以下のコマンドを実行し、XProtect のサービスを停止してください。
   ```bat
   net stop "Milestone XProtect Data Collector Server"
   net stop "Milestone XProtect Log Server"
   net stop "Milestone XProtect Management Server"
   net stop "Milestone XProtect Mobile Server"
   net stop "Milestone XProtect Recording Server"
   net stop "MilestoneEventServerService"
   sc config "Milestone XProtect Data Collector Server" start= demand
   sc config "Milestone XProtect Log Server" start= demand
   sc config "Milestone XProtect Management Server" start= demand
   sc config "Milestone XProtect Mobile Server" start= demand
   sc config "Milestone XProtect Recording Server" start= demand
   sc config "MilestoneEventServerService" start= demand
   ```
1. 以下の SQL 文を実行し、データベースを OFFLINE にし、デタッチしてください。
   ```sql
   alter database [Surveillance] set offline with ROLLBACK IMMEDIATE
   exec sp_detach_db 'Surveillance',TRUE
   alter database [Surveillance_IDP] set offline with ROLLBACK IMMEDIATE
   exec sp_detach_db 'Surveillance_IDP',TRUE
   alter database [SurveillanceLogServerV2] set offline with ROLLBACK IMMEDIATE
   exec sp_detach_db 'SurveillanceLogServerV2',TRUE
   ```
1. Cluster WebUI または clpgrp コマンドで、フェイルオーバグループを停止してください。

## XProtect VMS Products のクラスタ化
1. server1, server2 に SQL ファイルを保存するためのディレクトリを作成してください。
   ```bat
   C:
   cd \
   mkdir mssql
   ```
1. テキストエディタで ACT.SQL, DEACT.SQL ファイルを作成してください。
   ```sql
   /* ACT.SQL */
   create database [Surveillance] on
   (filename = 'X:\sql\data\Surveillance.mdf'),
   (filename = 'X:\sql\data\Surveillance_log.ldf')
   for attach
   create database [Surveillance_IDP] on
   (filename = 'X:\sql\data\Surveillance_IDP.mdf'),
   (filename = 'X:\sql\data\Surveillance_IDP_log.ldf')
   for attach
   create database [SurveillanceLogServerV2] on
   (filename = 'X:\sql\data\SurveillanceLogServerV2.mdf'),
   (filename = 'X:\sql\data\SurveillanceLogServerV2_log.ldf')
   for attach
   ```
   ```sql
   /* DEACT.SQL */
   alter database [Surveillance] set offline with ROLLBACK IMMEDIATE
   exec sp_detach_db 'Surveillance',TRUE
   alter database [Surveillance_IDP] set offline with ROLLBACK IMMEDIATE
   exec sp_detach_db 'Surveillance_IDP',TRUE
   alter database [SurveillanceLogServerV2] set offline with ROLLBACK IMMEDIATE
   exec sp_detach_db 'SurveillanceLogServerV2',TRUE
   ```
1. Cluster WebUI を起動し、設定モードに切り替えてください。
1. スクリプトリソースの start.bat, stop.bat を添付のように編集してください。
1. XProtect のサービスを制御するためのサービスリソースを追加してください。サービス名は以下になります。
   - Milestone XProtect Data Collector Server
   - Milestone XProtect Log Server
   - Milestone XProtect Management Server
   - Milestone XProtect Mobile Server
   - Milestone XProtect Recording Server
   - MilestoneEventServerService
     - 以下の事象を回避するため、サービス名での登録を推奨いたします。
       - https://www.support.nec.co.jp/View.aspx?id=3150114558
1. 追加後、依存関係が以下のようになるように設定してください。
   |深度|リソース|用途|
   |-|-|-|
   |0|フローティング IP リソース|一意な IP アドレス|
   |1|ミラーディスクリソース|データベースファイルを保存|
   |2|サービスリソース|SQL Server サービスの制御|
   |3|スクリプトリソース|データベースの制御|
   |4|サービスリソース|Milestone XProtect Data Collector Server の制御|
   |4|サービスリソース|Milestone XProtect Log Server の制御|
   |4|サービスリソース|Milestone XProtect Management Server の制御|
   |4|サービスリソース|Milestone XProtect Mobile Server の制御|
   |4|サービスリソース|Milestone XProtect Recording Server の制御|
   |4|サービスリソース|MilestoneEventServerService の制御|
1. クラスタの構成情報を反映してください。
1. Cluster WebUI または clpgrp コマンドで、フェイルオーバグループを起動してください。

## 参考
### XProtect のサービス一覧
```ps
Get-Service |Where DisplayName -like Mile*|Format-Table -AutoSize -Wrap -Property Name,DisplayName,RequiredServices,DependentServices

Name                                     DisplayName                              RequiredServices DependentServices
----                                     -----------                              ---------------- -----------------
Milestone XProtect Data Collector Server Milestone XProtect Data Collector Server {}               {}
Milestone XProtect Log Server            Milestone XProtect Log Server            {}               {}
Milestone XProtect Management Server     Milestone XProtect Management Server     {}               {}
Milestone XProtect Mobile Server         Milestone XProtect Mobile Server         {}               {}
Milestone XProtect Recording Server      Milestone XProtect Recording Server      {RpcSs}          {}
MilestoneEventServerService              Milestone XProtect Event Server          {RpcSs}          {}
```

### データベースのアクセス権限
- https://docs.microsoft.com/ja-jp/sql/database-engine/configure-windows/configure-file-system-permissions-for-database-engine-access?view=sql-server-ver15

### icacls コマンド
- https://docs.microsoft.com/ja-jp/windows-server/administration/windows-commands/icacls