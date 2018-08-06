---
description: 版本：10
---

# 4.1. 語法結構

SQL 語法包含一連串的命令，命令是由一系列的指示記號所組合而成，以分號結尾。最後如果是串流輸入，也會結束一個命令。指示的合法性是由特別的命令語法所定義的。

指示記號可能是關鍵字、識別項、引號識別項、文字、或一個特別的字元符號。指示一般來說是以空白分隔（空白符號、定位符號、換行符號），但如果不會混淆的話，也不一定需要。（一般只出現在特殊字元用來調整了其他指示的型別）

舉個例子，下面就是一個合法（符合語法）的 SQL 輸入：

```text
SELECT * FROM MY_TABLE;
UPDATE MY_TABLE SET A = 5;
INSERT INTO MY_TABLE VALUES (3, 'hi there');
```

這個序列包含了 3 個命令，每行一個（然而這不是一定的，同一行可以超過一個命令，而一個命令也可以分解為多行使用）。

順帶一提的是，註解也是 SQL 輸入的一部份，但不屬於任何指示記號，他們等同於空白字元。

SQL 語法並不是很嚴格要求什麼樣的指示記號來識別命令，或是哪些是運算子或參數。通常最前面的指示記號是命令的名稱，以上面的例子來說，我們通常會說是一個「SELECT」、一個「UPDATE」、以及一個「INSERT」命令。但對於 UPDATE 命令而言，有一個 SET 指示記號出現在某個地方是必要的；同樣地，INSERT 也需要有 VALUES 來搭配。精確的語法規則都在[第 6 部份](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/vi-reference.md)中的章節進行說明。

## 4.1.1. 識別項（Identifier）和關鍵字 （Keyword）

在上面的例子中的 SELECT、UPDATE、或是 VALUES，都是屬於關鍵字的範圍。所謂關鍵字，意即在 SQL 語言中，其具有固定的意義。像指示記號 MY\_TABLE 則是屬於識別項。它識別表格的名稱，欄位名稱，或是其他的資料庫物件，端看命令如何看待該識別項。然而，有時候它們會被簡稱為「名稱」。關鍵字和識別項的文法結構是相同的，意即不看整個命令的話，是無法辨別到底是識別項還是關鍵字的。完整的關鍵字列表，收錄在附件 C 當中。

SQL 識別項與關鍵字必須以英文字母開頭（a - z，也可以是附加符號和非拉丁字母，中文沒問題）或是底線（\_）。剩餘的字元可以是字母、底線、數字（0 - 9）、或錢字號（$）。注意錢字號，在標準 SQL 語法中是不允許使用的，所以可能會降低一些應用程式的可攜性。標準 SQL 也沒有定義包含數字或是以底線起迄的關鍵字，所以識別項這樣的形式定義是安全的，不會和標準未來的修訂相衝突。

資料庫系統不能使用長度超過 NAMEDATALEN -1 的識別項；太長的名稱仍然可以在命令中被輸入，但會被截斷。預設上，NAMEDATALEN 的設定是 64，所以最長的識別項名稱長度是 63 位元組。如果這個限制會造成困擾的話，你也可以調整 NAMEDATALEN 的編譯值，它的設定在 src/include/pg\_config\_manual.h 檔案中。

關鍵字和無引號識別項都是不分大小寫的，所以：

```text
UPDATE MY_TABLE SET A = 5;
```

等同於：

```text
uPDaTE my_TabLE SeT a = 5;
```

有一種寫法很常使用，就是把關鍵字用大寫表示，而識別項名稱使用小寫，例如：

```text
UPDATE my_table SET a = 5;
```

第二種要介紹的識別項是，受限制的識別項，或是引號識別項。它的形式就是以雙引號括住的任何字串。受限制的識別項，就一定是識別項，不會是關鍵字。所以，「"select"」就會被識別為名稱為「select」的表格或欄位，而無引號的 select 就會被視為是關鍵字，也可能會產生解譯錯誤，如果剛好用在可能是表格或欄位名稱的位置上的話。使用引號識別項的例子如下：

```text
UPDATE "my_table" SET "a" = 5;
```

引號識別項可以包含任何字元，除了字元碼為 0 的字元以外。（要包含雙引號字元的話，請使用連續兩個雙引號。）這可以用來建立原來不能使用的表格或欄位名稱，甚至是包含空白或＂&＂。但長度的限制仍然要遵守。

還有一種變形的引號識別項，允許包含跳脫的形式來表現萬國碼（unicode）。這種變形會以「U&」開頭（U大小寫皆可）緊接在前面的雙引號的前面，不能有任何空白在它們之間，例如：U&"foo"。（注意，這可能會和運算子的 & 產生混淆，但可以在運算子的 & 前後都加上空白來避免這個問題。）在雙引號內，萬國碼字元以跳脫的形式表現，也就是以倒斜線再接 4 位數的 16 進位碼，或倒斜線接一個加號再串一組 6 位數的 16 進位碼。例如，識別項 "data" 可以寫成這樣：

```text
U&"d\0061t\+000061"
```

下面是稍微不簡明的例子是，俄文的＂slon＂（大象），以希伯萊文字母表現：

```text
U&"\0441\043B\043E\043D"
```

如果希望以不同的跳脫字元來代替倒斜線的話，那麼可以雙引號結束後使用 UESCAPE 子句來指定，舉例來說：

```text
U&"d!0061t!+000061" UESCAPE '!'
```

跳脫字元可以是任何的單一字元，除了 16 進位數字的字元、單引號、雙引號、或空白以外。注意指定的跳脫字元是以單引號括住，而不是雙引號。

內容要使用到跳脫字元的話，就重覆輸入 2 次。

萬國碼的跳脫語法，只能使用 UTF8 的編碼。如果有用到其他的編碼的話，只有在 ASCII 範圍（最大為 \007F）可以使用。4 位數及 6 位數的形式，可以組合配對用來指定 UTF-16 中，大於 U+FFFF 的字元，雖然 6 位數的形式單獨就可以解決這個問題（組合配對並不會直接被儲存起來，他們會被編碼成 UTF-8 再儲存。）

把識別項用引號括起來也可以用來保持它的大小寫狀態，沒有括起來的話，都會被轉成小寫字母。舉例來說，對 PostgreSQL 而言，FOO、foo、"foo"，三者都是一樣的，但 "Foo" 和 "FOO" 就彼此及前面三者都視為不同。（在 PostgreSQL 中，把未引號括起的名稱轉成小寫，並不是 SQL 的標準。SQL 標準反而是都轉成大寫。所以在 SQL 標準中，foo 應該是等同於 "FOO" 而不同於 "foo"。如果你要增加語法的可攜性的話，建議最好都使用引號括起特別的名稱，或者都不要使用引號。）

## 4.1.2. 常數

PostgreSQL 中有三種隱含型別的常數：字串、位元字串、和數值。常數也可以強制型別，有助於更精確的表達，也可以讓系統處理更有效率。接下來就開始進行相關的說明。

### 4.1.2.1. 字串常數

在 SQL 中，所謂的字串常數，指的是用單引號括住的任意字元串列，例如：'This is a string'。如果在字串常數內需要有單引號的話就使用連續兩個單引號，例如：'Dianne''s horse'。注意這不是雙引號，是兩個單引號。

兩個字串常數如果只用空白及至少一個換行符號所分隔的話，那個它們會被連在一起，和寫成一個字串是一樣的。舉例來說：

```text
SELECT 'foo'
'bar';
```

等同於：

```text
SELECT 'foobar';
```

但如果是這樣：

```text
SELECT 'foo'      'bar';
```

語法上就不正確了。（這是來自於 SQL 奇怪的常規，PostgreSQL 單純只是遵循。）

### 4.1.2.2. C 語言樣式的跳脫字串常數

PostgreSQL 也支援跳脫字串常數，這些是 SQL 標準的延伸。跳脫字串常數使用的是字母 E （大小寫皆可），緊接著單引號所組成，例如：E'foo'。（如果字串有超過一行的話，也只要在第一個單引號前有 E 就可以了。）在跳脫字串當中，使用倒斜線開頭，就可以使用 C 語言式的倒斜線跳脫字串，通常是一個倒斜線再接一個字元，對應到一個特殊位元組的值，如 Table 4.1 所示。

**Table 4.1. 倒斜線跳腳字串（Backslash Escape Sequence）**

| **倒斜線跳腳字串** | 字元意義 |
| :--- | :--- |
| `\b` | backspace（倒退） |
| `\f` | form feed（換頁） |
| `\n` | newline（換行） |
| `\r` | carriage return（回到行首） |
| `\t` | tab（定位符號） |
| `\o`,`\oo`,`\ooo`\(`o`= 0 - 7\) | octal byte value（8 進位值） |
| `\xh`,`\xhh`\(`h`= 0 - 9, A - F\) | hexadecimal byte value（16 進位值） |
| `\uxxxx`,`\Uxxxxxxxx`\(`x`= 0 - 9, A - F\) | 16 or 32-bit hexadecimal Unicode character value（16 位元或 32 位元的 16 進位萬國碼字元值） |

任何其他接在倒斜線後面的字元都僅以原樣呈現。而如果要包含一個倒斜線的話，就使用連續兩個倒斜線輸入。同樣地，要包含一個單引號的話，可以使用跳脫字串 \' 輸入，也可以用一般連續兩個單引號的方式輸入。

你需要確保你所使用的 8 進位或 16 進位創建的位元組序列，都是屬於資料庫中合法的字元集。當資料庫編輯是 UTF-8 時，就應該使用萬國碼跳脫寫法，或其他萬國碼的輸入方式，如前 4.1.2.3 中所述。（所謂其他的方式可能是自行組合每一個位元組，但這樣會是相當麻煩的事。）

萬國碼跳脫語法只有在 UTF8 的編碼下才完整支援。當有其他的字元編碼被使用時，就只能使用 ASCII 的範圍（最大值為 \u007F）中的值。4 位數及 6 位數的型式可以用來配對指定 UTF-16 超過 U+FFFF 的字元，即使 6 位數的型式就足以解決這個問題。（當使用配對語法，且字元編碼為 UTF8 時，他們會先被合併成單一字元，然後再編碼成 UTF-8。）

## 注意

如果設定檔參數 [standard\_conforming\_string](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/iii-server-administration/server-configuration/1913-version-and-platform-compatibility.md) 設定為 off，PostgreSQL 不論在一般字串還是跳脫字串常數，都會把倒斜線識別為跳脫符號。然而，在 PostgreSQL 9.1 之前，這個參數的預設值為 on，表示只在跳脫字串常數裡，才把倒斜線視為跳脫符號。這樣的模式是更與標準相容的，但可能會破壞默認舊有設定的應用程式，也就是總是把倒斜線視為跳脫符號。在這樣的背景之下，你可以把這個參數設為 off，但更好的是，修改程式不再使用倒斜線跳脫符號。如果你需要使用倒斜線跳脫符號來表示一個特殊字元，請使用 E 開頭的字串常數。

有關 [standard\_conforming\_string](https://www.postgresql.org/docs/10/static/runtime-config-compatible.html)，順帶一提的是，還有 [escape\_string\_warning](https://www.postgresql.org/docs/10/static/runtime-config-compatible.html) 和 [backslash\_quote](https://www.postgresql.org/docs/10/static/runtime-config-compatible.html) 兩個參數，也提供調整倒斜線在字串常數中的使用。

字元代碼 0 的字元不能使用在字串常數當中。

### 4.1.2.3. String Constants with Unicode Escapes

PostgreSQL 也支援其他跳脫字串的語法，可以用來直接輸入任意的萬國碼字元。萬國碼跳脫字串常數是以 U& （U& 或 u& 皆可）開頭，然後緊接著單引號括住的字串，記得中間不能有任何空白，例如：U&'foo'。（注意這可能會混淆到 & 的使用，最好在其他使用 & 作為運算子的指令中，在 & 前後 加上空白字元，以避免這個問題。）在括住的內容裡，萬國碼字元可以使用跳脫字元來指定，也就是使用倒斜線再接一組 4 位數的 16 進位值，或者以倒斜線加上加號再接一組 6 位數的 16 進位值。舉個例子，字串 'data' 也可以寫成：

```text
U&'d\0061t\+000061'
```

下面是稍微不簡明的例子是，俄文的＂slon＂（大象），以希伯萊文字母表現：

```text
U&'\0441\043B\043E\043D'
```

如果希望以不同的跳脫字元來代替倒斜線的話，那麼可以雙引號結束後使用 UESCAPE 子句來指定，舉例來說：

```text
U&'d!0061t!+000061' UESCAPE '!'
```

跳脫字元可以是任何的單一字元，除了 16 進位數字的字元、單引號、雙引號、或空白以外。

萬國碼跳脫語法只有在 UTF8 的編碼下才完整支援。當有其他的字元編碼被使用時，就只能使用 ASCII 的範圍（最大值為 \u007F）中的值。4 位數及 6 位數的型式可以用來配對指定 UTF-16 超過 U+FFFF 的字元，即使 6 位數的型式就足以解決這個問題。（當使用配對語法，且字元編碼為 UTF8 時，他們會先被合併成單一字元，然後再編碼成 UTF-8。）

然而，萬國碼的跳脫字串語法，只有在參數 [standard\_conforming\_strings](https://www.postgresql.org/docs/10/static/runtime-config-compatible.html) 設定為 on 時有效。這是因為這個語法可能會造成 SQL 指令在編譯時的困擾，造成 SQL 隱碼攻擊（SQL injection） 或其他安全性的問題。如果這個參數設定為 off，那麼這個語法就會被禁止，並且產生錯誤訊息。

內容要使用到跳脫字元的話，就重覆輸入 2 次。

### 4.1.2.4. 錢字引號字串常數

標準的語法用於字串常數的設定很方便的，但如果字串裡有很多單引號或倒斜線，可讀性就很低了，因為它們都必須再連續多一個符號輸入。像這樣的例子，要改善可讀性的話，PostgreSQL 提供了另一個方式，稱作「錢字引號」（dollar quoting），來描述字串常數。錢字引號字串常數包含一個錢字號（$），可省略或多個字元所組成的「標籤」，另一個錢字號，組成字川的任何序列文字，再一個錢字號，與起始的錢字引號同樣的標籤，再一個錢字號。舉例來說，這裡有兩個不同使用錢字引號的方式，但都是「Dianne's horse」

```text
$$Dianne's horse$$
$SomeTag$Dianne's horse$SomeTag$
```

注意在錢字引號字串中，單引號的使用就不需要跳脫處理了。實際上，在錢字引號字串中，沒有字元需要跳脫處理：字串內容就原樣輸出。倒斜錢並不特別，就算是錢字號也是，除非它們是引號標籤配對的一部份。

巢狀錢字字串常數是可以的，只要在不同層選擇不同的標籤就好。最常見的用途就是撰寫函數定義。舉例如下：

```text
$function$
BEGIN
    RETURN ($1 ~ $q$[\t\r\n\v\\]$q$);
END;
$function$
```

這裡，「$q$\[\t\r\n\v\\]$q$」以錢字引號字串輸出就是「\[\t\r\n\v\\]」，作為 PostgreSQL 的函數內容。但這個字串並不會和外層的 $function$ 配對。對外層的字串而言，它只是被包裏的一部份字元而已。

以錢字符作為標籤（如果有的話）的引號字串和無引號的識別項，遵循相同的規則，除了它無法包含錢字符號以外。標籤是區分大小寫的，所以 $tag$String content$tag$ 是正確的，而 $TAG$String content$tag$ 是不合法的。

錢字引號字串緊接著關鍵字或識別項的話，就必須以空白分隔；否則錢字號的終止符可能會被當作前面識別項的一部份。

錢字引號並不是標準 SQL 的用法，但當撰寫一些複雜字串的時候，會比標準語法更為便利。當字串常數內嵌於另一個常數時，也是很好用的情境，像自訂函數時就時常用到。使用單引號的語法時，前面例子中的每一個倒斜線，需要使用 4 個倒斜線才能表示（原來字串常數時需要雙倒斜線，然後在執行階段時也需要雙倒斜線，一共就是 4 倍）。

### 4.1.2.5. 位元字串常數（Bit-string Constants）

位元字串常數看起來就像是一般的字串常數，只是將 B（大小寫皆可）放在引號的前面（不能有空白），例如：B'1001'。而在位元字串當中，只能有 0 或 1 的存在。

另一方面，位元字串常數也可以表示一個 16 進位的值，使用的先導字為 X（大小寫皆可），例如：X'1FF'。這個撰寫方式與使用前段方式，以 4 位數 2 進位表示每一個 16 進位位數，是相同的結果。

這兩種位元字串常數的表達方式，都可以在字串中換行，如同一般的字串常數。錢字引號表示方式不能使用在位元字串常數上。

### 4.1.2.6. 數值常數（Numeric Constants）

數值常數可以以下列語法輸入：

```text
digits
digits.[digits][e[+-]digits]
[digits].digits[e[+-]digits]
[digits]e[+-]digits
```

這裡的 digits 指的是 0 到 9 的多位數十進位數字。如果有小數點的話，在小數點之前或之後要有數字。在指數標記 e 之前，也必須要有數字。字串中間不能再有其他字元或空白出現。注意，最前面正負號並不是數值常數的一部份，它是屬於運算子的概念。

下面是一些合法數值常數的例子：

42  
3.5  
4.  
.001  
5e2  
1.925e-3

數值常數如果沒有小數點或指數標記的話，預設就會被假定為整數，32 位元以內的為整數型別（interger），否則就會以 64 位元的大整數型別（bigint）來處理。其次就會宣告為數值型別（numeric）。只要包含小數點或指數標記的數值，都會預設使用數值型別。

預設數值常數的資料型別只是整個型別解析演算法的開端而已。在多數的情況下，各種常數會自動被轉換為最貼近內容的適當型別。不過，如果需要的話，你可以強制指定一個資料型別給該常數。舉例來說，你可以強制以實數型別（real 或 float4）來處理該數值：

```text
REAL '1.23'  -- string style
1.23::REAL   -- PostgreSQL (historical) style
```

實際上，在型別轉換上還有一些特殊的情況，留待後續探討。

### 4.1.2.7. 其他型別常數

任意型別的常數，可以使用下列的語法來表示：

```text
type 'string'
'string'::type
CAST ( 'string' AS type )
```

字串常數的內容會由型別轉換的程序 type 來處理，其結果就會得到該常數的專屬型別。明定型別轉換可以被省略，如果不會混淆的話（舉例來說，要輸入給特定的表格欄位的話，因為已有型別宣告，就不會混淆），那麼就會自動給定型別。

字串常數可以使用一般 SQL 標準寫法，或是錢字引號寫法。

還可以使用函數式的語法來撰寫：

```text
typename ( 'string' )
```

但並非所有的型別都可以使用這個方式，請參閱 [4.2.9 節](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/ii-the-sql-language/sql-syntax/42-value-expressions.md)取得詳細說明。

「::」、CAST\(\)、及函數式語法，也可以用來指定任何表示式在執行中的型別轉換，如同 [4.2.9 節](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/ii-the-sql-language/sql-syntax/42-value-expressions.md)中所描述的。要避免語法上的混淆，「type 'string'」這個語法，只能用在指定簡單的文字常數，另一個限制是，不能用於陣列型別。陣列常數的型別指定，請使用 :: 或 CAST\(\) 的語法。

## 4.1.3. 運算子（Operators）

一個運算子最長可以是 NAMEDATALEN - 1（預設為 63 個字元），除了以下的字元之外：

* * \* / &lt;&gt; = ~ ! @ \# % ^ & \| \` ?

還有一些運算子的限制：

* 「--」和「/\*」都不能出現在運算子裡，因為它們表示註解的開始。
* 多字元的運算子不能以 + 或 - 結尾，除非名稱裡也包含了下列字元：

  ~ ! @ \# % ^ & \| \` ?

舉個例子，@- 可以是合法的運算子，但 \*- 就不合法。這個限制是讓 PostgreSQL 解譯 SQL 語法時，可以不需要在不同的標記間使用空白分隔。

當使用非 SQL 標準的運算子時，你通常需要在相隣的運算子間使用空白以免混淆。舉例來說，如果你已經定義了一個左側單元運算子 @，你就不能使用 X\*@Y，必須寫成 X\* @Y，以確保 PostgreSQL 可以識別為兩個運算子，而不是一個。

## 4.1.4. 特殊字元

有一些字元並不是字母型態，而具有特殊意義，但並非運算子。詳細的說明請參閱相對應的語法說明。本節僅簡要描述這些特殊字元的使用情境。

* 錢字號（$）其後接著數字的話，用來表示函數宣告或預備指令的參數編號。其他的用法還有識別項的一部份，或是錢字引號常數。
* 小括號（\( \)）一般用來強調表示式並且優先運算。還有某些情況用於表示某些 SQL 指令的部份的必要性。
* 中括號（\[ \]）用於組成陣列的各個元素。詳情請參閱 [8.15 節](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/ii-the-sql-language/data-types/815-arrays.md)有關於陣列的內容。
* 逗號（,）用於一般語法上的結構需要，來分隔列表中的單元。
* 分號（;）表示 SQL 指令的結束。它不能出現在指令中的其他位置，除非是在字串常數當中，或是引號識別項。
* 冒號（:）用在取得陣列的小項。（參閱 [8.15 節](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/ii-the-sql-language/data-types/815-arrays.md)）在某些 SQL 分支（篏入式 SQL 之類的）中，冒號用來前置變數名稱。
* 米字號（\*）用來表示表格中所有的欄位，或複合性的內容。它也可以用於函數宣告時，不限制固定數量的參數。
* 頓號（.）用在數值常數之中，也用於區分結構、表格、及欄位名稱。

## 4.1.5. 註解（Comments）

註解是以連續兩個破折號開頭，一直到行結尾的字串。例如：

```text
-- This is a standard SQL comment
```

另外，C 語言的註解語法也可以使用：

```text
/* multiline comment
 * with nesting: /* nested block comment */
 */
```

這樣的註解，以「/\*」開頭，一直持續到對應的「\*/」出現才結束。這樣區塊式的註解可以巢狀使用，所以你可以一次註解掉一堆包含註解的指令。這點是 SQL 的標準，和 C 語言的使用不太一樣的地方。

註解會在進一步的語法分析前被消去，也可以方便地以空白字元替代。

## 4.1.6. 運算優先權（Operator Precedence）

Table 4.2 列出在 PostgreSQL 中，運算子的運算優先權及運算次序。大多數的運算子都是相同的運算優先權，並且是左側運算。這些優先權與次序是撰寫在解譯器的程式當中的。

你有時候需要加上括號，當遇到二元運算子與一元運算子一起出現時。舉個例子：

```text
SELECT 5 ! - 6;
```

會被解譯為：

```text
SELECT 5 ! (- 6);
```

因為解譯器並不知道實際的情況，所以它可能會搞錯。「!」是一個後置運算子，並非中置運算子。在這個例子中，要以想要的方式進行運算的話，你必須要改寫為：

```text
SELECT (5 !) - 6;
```

這是為了延展性而需要付出的代價。

**Table 4.2. Operator Precedence \(highest to lowest\)**

| Operator/Element | Associativity | Description |
| :--- | :--- | :--- |
| `.` | left | table/column name separator |
| `::` | left | PostgreSQL-style typecast |
| `[]` | left | array element selection |
| `+-` | right | unary plus, unary minus |
| `^` | left | exponentiation |
| `*/%` | left | multiplication, division, modulo |
| `+-` | left | addition, subtraction |
| \(any other operator\) | left | all other native and user-defined operators |
| `BETWEEN / IN / LIKE / ILIKE / SIMILAR` |  | range containment, set membership, string matching |
| `<>=<=>=<>` |  | comparison operators |
| `IS / ISNULL/ NOTNULL` |  | `IS TRUE`,`IS FALSE`,`IS NULL`,`IS DISTINCT FROM`, etc |
| `NOT` | right | logical negation |
| `AND` | left | logical conjunction |
| `OR` | left | logical disjunction |

注意，使用與內建運算子同名的自訂運算子，運算優先權的規則也會以原規則適用，如同上面的樣子。舉例來說，如果你定義了一個「+」的運算子，用於自訂的資料型態，那麼它就會和內建的「+」擁有相同的運算優先權，而與你的運算內容無關。

當某個結構操作的運算子用於 OPERATOR 語法之中時，如下所示：

```text
SELECT 3 OPERATOR(pg_catalog.+) 4;
```

OPERATOR 建構式被用來為任何運算子，取得如 Table 4.2 中所示的預設運算優先權。不論在 OPERATOR\(\) 中指定什麼運算子，都會回傳 true 的結果。

## 注意

PostgreSQL 在 9.5 之前的運算優先權有一些不同。比較特別的是，比較運算子「&lt;= &gt;= &lt;&gt;」是和一般其他運算子是相同等級的；「IS」先前的優先權較高；而「NOT BETWEEN」和相關的建構式行為不一致，使得在某些情況下，「NOT」和「BETWEEN」的優先權不同。這些規則的改變是為了與 SQL 標準有更好的相容性，減少因為等價轉換的不一致處理所造成的困擾。大多數的情況，這些改變並不需要使用習慣的改變，也不會產生沒有運算子的錯誤，而且都可以透過增加括號來解決。然而，有一些極端的情況可能會在沒有錯誤的情況改變其運算行為。如果你很關心這些變化，很擔心這些無聲的錯誤，你可以打開參數 [operator\_precedence\_warning ](https://www.postgresql.org/docs/10/static/runtime-config-compatible.html)來測試你的程式，然後檢查是否有警告被記錄下來。
