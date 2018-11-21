# 토스유저 생성 흐름

## 개요1. Admin 요청 및 Batch에서의 Queue 저장
Admin은 rmi를 통해 Batch에게 extract 요청만 하고, 이후부터는 Batch에서 관리하는 구조이다.
1. Admin에서 extract 요청을 하면 com.skt.moment.tos.rmi.TosServiceImpl 의 extract 메소드가 수행된다. 
2. 이때 `ExtractTosTask` 가 생성되어, `TosTaskQueue` 에 저장 된다.


## 개요2. Batch 서버 스케줄링
Batch 서버의 `TosScheduler` 에 의해 Queue 에 있는 TosTask 들을 수행한다.
* TOS USER 를 생성 할때에는 각 Task 들의 역할을 순서대로 수행하면서 진행된다.
* 순서는 `ExtractTosTask (Thread)` -> `GetDataTosTask (Thread)` -> `GenerateTosUserTosTask (Job)` 순 이다.
* 주기성 배치인 경우 `GenerateTosUserTosTask` 종료 시점에 `ExtractTosTask` 를 다시 생성하여 Queue 에 재등록 하여 반복할수 있게 한다.
* 모든 신규 `ExtractTosTask` 객체의 생성은 Admin 의 rmi 요청을 통해서만 가능하다. 

### TosScheduler
1초마다 `TosTaskQueue` 에 저장되어있는 `TosTask` 를 꺼내서 해당 작업들을 수행한다.


### TosTaskQueue
TosTask 들을 저장할수있는 자료구조 이다.



## TosTask 객체 설명 (최상위 super class)
TosTaskQueue 에 저장 가능한 객체 이다. Queue 에 있는 모든 객체는 `TosTask`를 상속 받아야만 한다. 상속 구조는 다음과 같다.  
```
TosTask
  L TosThreadTask
        L ExtractTosTask
        L GetDataTosTask
  L TosJobTask
        L GenerateTosUserTosTask
```


### ExtractTosTask 설명 (extends TosThreadTask)
Admin의 extract 요청이 추가 되었을때 생성된다.
1. tos user 배치가 필요한 캠페인들이 있는지 확인
2. insert TB_MMT_TOS_BATCH
3. TosExtractHttpRequest 수행. (TOS는 extract를 수행하게되고, 완료되면 callback을 준다)
4. tosBatch 상태값을 EXTRACT_REQUESTED 로 변경
5. `ExtractTosTask` 를 생성하여 큐에 저장하며 종료한다.


### GetDataTosTask (extends TosThreadTask)
TOS로 부터 extract가 완료 되었다고 callback 을 전달받는 시점에(Controller 에서 수신받음) 생성된다.
1. DB에 저장되어있는 tosBatch 를 조회
2. tosBatch 의 상태값을 EXTRACT_CALLBACK_RECEIVED 로 변경한다.
3. `GetDataTosTask` 를 생성하여 큐에 저장하며 종료한다.
   
   
### GenerateTosUserTosTask (extends TosJobTask)
GetDataTosTask 스레드가 run() 수행시에 생성된다. 
이녀석만 JOB인 이유는, read -> process -> write 가 수행되는 구조이기 때문이다. 
1. `JobParameters` 를 생성하여 return 한다.
2. 리턴받은 `JobParameters` 로 jobLauncher를 run 시킨다.
 ```java
JobExecution jobExecution = jobLauncher.run(context.getBean(tosJobTask.getJobName(), Job.class), tosJobTask.build());
```

