# 31.6. 限制

邏輯複寫目前有以下限制或功能不足的地方。這些可能會在將來的版本中解決。

* 資料庫結構和 DDL 指令不會被複寫。初始模式可以使用 pg\_dump --schema-only 手動複寫。後續的結構變更需要手動保持同步。（但是，請注意，兩側的結構沒有必要完全相同。）當主要資料庫中的結構定義變更時，邏輯複寫是沒問題的：當發佈者上的結構産生變更並且複寫的資料開始到達訂閱戶但不符合資料表結構，複寫將産生錯誤，直到結構更新。在許多情況下，可以透過先將預定的架構變更套用於訂閱戶來避免間歇性的錯誤。
* 序列資料不會被複寫。序列支援的序列或識別欄位中的資料當然會作為資料表的一部分進行複寫，但序列本身仍然會顯示訂閱戶的初始值。如果使用者用作唯讀資料庫，那麼這通常不成問題。但是，如果打算對訂閱戶資料庫進行某種切換或故障轉移，則需要將序列更新為最新值，方法是複寫發佈者的目前資料（可能使用 pg\_dump）或設定對該資料表而言足夠高的值。
* 支援複寫 TRUNCATE 指令，但是在 TRUNCATE 由外部鍵連結的資料表群組時必須格外小心。當複寫 TRUNCATE 操作時，訂閱者將 TRUNCATE 在發佈者上被 TRUNCATE 的同一組資料表，這些資料表群組是明確指定的或透過 CASCADE 所收集的，再減去了不屬於發佈的資料表。如果所有受影響的資料表都屬於同一個發佈，則此方法將會正常運作。但是，如果某些要在訂閱者上被 TRUNCATE 的資料表具有指向不屬於同一（或任何）訂閱的資料表的外部鍵連結，則在訂閱伺服器上套用 TRUNCATE 操作將會失敗。
* Large objects（請參閱[第 34 章](https://github.com/pgsql-tw/gitbook-docs/tree/67cc71691219133f37b9a33df9c691a2dd9c2642/tw/client-interfaces/34.-large-objects)）不會被複寫。除了將資料儲存在普通資料表中之外，沒有其他解決方法。
* 僅 TABLE（包括 PARTITION TABLE）支援複寫。 嘗試製寫其他類型的關連物件，例如檢視表 VIEW，具體化檢視圖 MATERIALIZED VIEW 或外部資料表 FOREIGN TABLE，將會導致錯誤。
* 在分割資料表之間複寫時，預設情況下，會實際複寫來自發佈者上的子資料表，因此發佈者上的分割區也必須作為有效目標資料表存在於訂閱者上。 （它們本身可以是子資料表，也可以進一步再分割，甚至可以是獨立的資料表。）PUBLICATION 還可以指定要使用分割父資料表的 identity 和 schema 來複寫變更，取代實際要複寫的各個子資料表（請參閱 [CREATE PUBLICATION](../../reference/sql-commands/create-publication.md)）。