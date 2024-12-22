# 27.3. Failover

如果主伺服器發生故障，則備用伺服器應該開始故障轉移程序。

如果備用伺服器發生故障，則毌須進行故障轉移。如果備用伺服器可以重新啟動，即使是在某個時間點之後，也可以利用可重新啟動的還原功能立即重新啟動還原程序。如果備用伺服器無法重新啟動，則應該重新建立一個完整的備用伺服器。

如果主伺服器發生故障，並且備用伺服器成為新的主伺服器，然後舊的主伺服器重新啟動，則必須具有一種機制，通知舊的主伺服器不再是主伺服器。這有時被稱為 STONITH（Shoot The Other Node In The Head），這是避免兩個系統都認為它們是主要系統所必須要做的事，這種情況會導致混亂並導致資料損毁。

許多故障轉移系統(failover system)僅使用兩個系統，即主系統和備用系統，透過某種心跳機制(heartbeat mechanism)連接，以不斷驗證兩個系統之間的連接性以及主系統的可行性。也可以使用第三個系統：見證伺服器(witness server)來防止某些不適當的故障轉移情況，但除非經過足夠的謹慎和嚴格的測試，否則額外的複雜性可能不值得。

PostgreSQL does not provide the system software required to identify a failure on the primary and notify the standby database server. Many such tools exist and are well integrated with the operating system facilities required for successful failover, such as IP address migration.

Once failover to the standby occurs, there is only a single server in operation. This is known as a degenerate state. The former standby is now the primary, but the former primary is down and might stay down. To return to normal operation, a standby server must be recreated, either on the former primary system when it comes up, or on a third, possibly new, system. The [pg\_rewind](https://www.postgresql.org/docs/12/app-pgrewind.html) utility can be used to speed up this process on large clusters. Once complete, the primary and standby can be considered to have switched roles. Some people choose to use a third server to provide backup for the new primary until the new standby server is recreated, though clearly this complicates the system configuration and operational processes.

So, switching from primary to standby server can be fast but requires some time to re-prepare the failover cluster. Regular switching from primary to standby is useful, since it allows regular downtime on each system for maintenance. This also serves as a test of the failover mechanism to ensure that it will really work when you need it. Written administration procedures are advised.

要觸發日誌傳送備用伺服器的故障轉移，請執行 `pg_ctl promote`、呼叫 `pg_promote` 或建立一個事件觸發的執行腳本檔案，該檔案名稱及路徑由 promot\_trigger\_file 指定。如果您打算使用 `pg_ctl promote` 或呼叫 `pg_promote` 進行故障轉移，則不需要 promote\_trigger\_file。 如果要設定僅用於從主伺服器唯讀查詢（而不是出於高可用性目的）的報表伺服器，則毌須進行故障轉移。
