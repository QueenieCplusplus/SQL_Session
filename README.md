# SQL Session
通訊的會議期間

# Connection Object

sessions 和 connections 並非指相同的東西， session 憑藉 connection 物件來操作資料庫，一旦任務完成後， session 會自動將 connection 物件交還给 pool。

        session 利用 connection 行使 transaction 

開發者從 pool 網路連線池拿取命名為 connection 的物件，
可呼叫其仲介軟體或是使用者端軟體中和 SQL 指令同名稱的方法如 

        session.commit 
        session.rollback 
        session.close 

將連線物件放回 網路連線池子 pool 裡。

# Timeout

然而此時 connection 物件的交易 transaction 並沒有完成與完畢（rollback or commit)。 而不知什么原因（recyle 了，timeout 了），此時 connection 生命週期已盡，仲介軟體則會負責與 DB 重新生成連線，但是由于 transaction 沒有結束，故無法重新連線，也就無法產生新的 Session。

# Driver

通常仲介軟體使用驅動物件或是特殊意義的物件產生來連結資料庫：

     object = method_called('agent://admin:password@IP:port/dbname')
     
# Pool 

預設上，session 由 QueuePool 自行管理和重複使用，如要關掉預設，可設定參數值為 NullPool !

關於連接池的相關參數：

- pool_recycle=n

     如果 connection 空閒了 n 秒，自動重新獲取，防止 connection 被 db server 關閉。
     
- pool_timeout=30

     獲取 session 的超时阈值，

- pool_size=5

     連接數大小，可根據實際情況調整

- max_overflow=10

     超出 pool_size 後可允許的最大連街數，這些連結使用完畢，不放回 pool 中，會被真正關閉的，不復使用。
     

        #!/usr/bin/env python
        #-*- coding: utf-8 -*-

        from sqlalchemy import create_engine
        from sqlalchemy.orm import sessionmaker
        from sqlalchemy.pool import NullPool

        engine = create_engine('mysql+mysqldb://root:password@127.0.0.1:3306/dbname', poolclass=NullPool) // 將預設設定 false
        Session = sessionmaker(bind=engine)
        session = Session()
        usr_obj_list = session.query(UsrObj).all()
        print usr_obj_list[0].id
        session.close() // 此時可斷開連接池
        
 # Thread & Task
 
實際上有多線程參與同一任務，這些線程之间共享 Session 及其對象；應用程式需要落實的 locking scheme，以便不會『同時訪問 Session 或其狀態』。

解決方法是為每個迸發 thread 維護一個 Session，並將對象從一 Session 複製到另一 Session，通常使用 Session.merge() 方法將對象的狀態复制到本地的新對象中。
        
 # flush & commit
 
-[x] flush 預先提交，但僅至資料庫內存，並為寫入資料庫文件內

-[x] commit 提交，把內存直接寫入，可以提供查询了
