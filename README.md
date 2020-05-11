# SQL Session
通訊的會議期間

開發者從 pool 網路連線池拿取命名為 connection 的物件，
可呼叫其仲介軟體或是使用者端軟體中和 SQL 指令同名稱的方法如 

session.commit 
session.rollback 
session.close 

將連線物件放回 網路連線池子 pool 裡。

然而此時 connection 物件的交易 transaction 並沒有完成與完畢（rollback or commit)。 而不知什么原因（recyle 了，timeout 了），此時 connection 生命週期已盡，仲介軟體則會負責與 DB 重新生成連線，但是由于 transaction 沒有結束，故無法重新連線，產生新的 Session。
